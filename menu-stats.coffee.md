    seem = require 'seem'

    {debug,hand,heal} = (require 'tangible') 'clever-note:menu-stats'

    redis_hour = 3600
    TWO_DAYS = 2*24*redis_hour

    module.exports = (w) ->

      {store,sum,count,save,reset,clear,get,getset,delta} = (require './redis-store')()

      w.once '__end', -> store.quit()

      save_agent_name = (agent,agent_name) ->
        save "an-#{agent}", agent_name

      get_agent_name = seem (agent) ->
        (yield get "an-#{agent}") ? agent

      set_menu = ({menu,reference}) ->
        getset "rm-#{reference}", menu

      get_menu = seem ({menu,reference}) ->
        return Promise.resolve menu if menu?
        yield get "rm-#{reference}"

Call progress reporting
-----------------------

      call_step = seem ({reference,now},text) ->
        time_key = "rT-#{reference}"

        ms = yield delta time_key, now

        if ms? and (s = ms // 1000) > 0
          w.emit 'call-step', reference, text, s
        else
          w.emit 'call-step', reference, text

Storage for later stats
-----------------------

### Menu traversal

      menu_start = (report,name) ->
        {reference,now} = report
        key = "#{name}-#{reference}"
        heal delta key, now

      menu_stop = seem (report,name) ->
        {day,domain,reference,now} = report

        key = "#{name}-#{reference}"
        value = yield delta key, now
        if value?
          value //= 1000

        heal clear key

        call_stats report, name, value

### Agent

      agent_start = (report,name) ->
        {agent,now} = report
        agent ?= ''
        key = "#{name}-#{agent}"
        heal delta key, now

      agent_stop = seem (report,name) ->
        {agent,now} = report
        agent ?= ''

        key = "#{name}-#{agent}"
        value = yield delta key, now
        if value?
          value //= 1000

        heal clear key

        call_stats report, name, value

      call_stats = seem (report,name,value) ->
        {day,domain,agent} = report
        agent ?= ''
        menu = yield get_menu report
        menu ?= ''

Detail per menu/group and agent

        w.emit 'stats:menu+agent', {name,day,domain,menu,agent,value,stats: yield stats "S.#{name}:#{day}:#{domain}:#{menu}:#{agent}", value}

Summary per agent

        w.emit 'stats:agent', {name,day,domain,agent,value,stats: yield stats "S.#{name}:#{day}:#{domain}:*:#{agent}", value}

Summary per menu/group

        w.emit 'stats:menu', {name,day,domain,menu,value,stats: yield stats "S.#{name}:#{day}:#{domain}:#{menu}", value}

Summary per domain

        w.emit 'stats:domain', {name,day,domain,value,stats: yield stats "S.#{name}:#{day}:#{domain}", value}

        return

      get_agent_stats = ({day,domain,menu,agent},name) ->
        agent ?= ''
        menu ?= ''
        store.hgetall "S.#{name}:#{day}:#{domain}:#{menu}:#{agent}"

      get_menu_stats = ({day,domain,menu},name) ->
        menu ?= ''
        store.hgetall "S.#{name}:#{day}:#{domain}:#{menu}"

      get_domain_stats = ({day,domain},name) ->
        store.hgetall "S.#{name}:#{day}:#{domain}"

Define a new 'stats' function on the redis store.

      lua_stats_code = '''
        local key = KEYS[1]
        local v = tonumber(ARGV[1])
        local min = redis.call('hget',key,'min')
        if min == false then
          min = v
        else
          min = tonumber(min)
          if v < min then
            min = v
          end
        end
        local max = redis.call('hget',key,'max')
        if max == false then
          max = v
        else
          max = tonumber(max)
          if v > max then
            max = v
          end
        end
        redis.call('hset',key,'min',min)
        redis.call('hset',key,'max',max)
        local count = redis.call('hincrby',key,'count',1)
        local sum   = redis.call('hincrby',key,'sum',v)
        return {key,count,sum,max,min}
      '''

      store.defineCommand 'stats', numberOfKeys: 1, lua: lua_stats_code

      stats = seem (key,value) ->

        data = yield store.stats key, value ? 0

        key: data[0]
        count: data[1]
        sum: data[2]
        max: data[3]
        min: data[4]

      scan = (pattern,cb) ->
        new Promise (resolve,reject) ->
          stream = store.scanStream
            match: pattern
            count: 100

          result = {}

          stream.on 'data', hand (keys) ->
            return unless keys.length > 0
            values = yield store.mget keys...
            for key, i in keys
              cb key, values[i]
            return

          stream.on 'end', ->
            resolve result

          stream.on 'error', reject

      lex_insert = seem (key,value) ->
        yield store.zadd key, 0, value
        yield store.expire key, TWO_DAYS

      lex_scan = seem (key,cb) ->
        last = ''
        while true
          res = yield store.zrangebylex key, "(#{last}", '[zzzz', 'limit', 0, 100
          return unless res.length > 0
          for value in res
            yield cb value
          last = res[res.length-1]

        return

      enumerate_and_sum = seem (pattern,field) ->
        new Promise (resolve,reject) ->
          stream = store.scanStream
            match: pattern
            count: 100

          sum = 0

          stream.on 'data', hand (keys) ->
            return unless keys.length > 0
            for key in keys
              sum += parseInt yield store.hget key, field
            return

          stream.on 'end', ->
            resolve sum

          return

Try: `SORT #{list} ALPHA NOSORT GET *->#{fieldname}`

      {save_agent_name,get_agent_name,set_menu,get_menu,call_step,menu_start,menu_stop,agent_start,agent_stop,call_stats,get_agent_stats,get_menu_stats,get_domain_stats,lex_insert,lex_scan,enumerate_and_sum}
