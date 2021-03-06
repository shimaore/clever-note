    {debug,foot} = (require 'tangible') 'clever-note:menu-stats'

    redis_hour = 3600
    TWO_DAYS = 2*24*redis_hour
    TWO_MONTHS = 2*31*24*redis_hour

    module.exports = (w) ->

      {store,sum,count,save,reset,get,getset,delta,delta_clear} = (require './redis-store')()

      w.once '__end', -> store.quit()

      save_agent_name = (agent,agent_name) ->
        save "an-#{agent}", agent_name

      get_agent_name = (agent) ->
        get "an-#{agent}"

      get_menu = ({menu,reference}) ->
        return Promise.resolve menu if menu?
        await get "rm-#{reference}"

Call progress reporting
-----------------------

      call_step = ({reference,now},text) ->
        time_key = "rT-#{reference}"

        ms = await delta time_key, now

        if ms? and (s = ms // 1000) > 0
          w.emit 'call-step', reference, text, s,    now
        else
          w.emit 'call-step', reference, text, null, now

        debug "#{reference} #{now} call_step #{s} #{text}"
        return

Storage for later stats
-----------------------

### Menu traversal

      menu_start = (report,name) ->
        {reference,menu,now} = report

        debug "#{reference} #{now} menu_start #{name} #{menu}"

        key = "#{name}-#{reference}"
        await delta key, now
        await getset "rm-#{reference}", menu

      menu_stop = (report,name) ->
        {day,domain,reference,now} = report

        debug "#{reference} #{now} menu_stop  #{name}"

        key = "#{name}-#{reference}"
        value = await delta_clear key, now

Avoid double-counts

        return if not value?

        value //= 1000

        await call_stats report, name, value
        return

### Agent

      agent_start = (report,name) ->
        {agent,now} = report
        agent ?= ''

        debug "#{report.reference} #{now} agent_start #{name} #{agent}"

        key = "#{name}-#{agent}"
        await delta key, now
        return

      agent_stop = (report,name) ->
        {agent,now} = report
        agent ?= ''

        debug "#{report.reference} #{now} agent_stop  #{name} #{agent}"

        key = "#{name}-#{agent}"
        value = await delta_clear key, now

        if value?
          value //= 1000

        await call_stats report, name, value
        return

      call_stats = (report,name,value) ->
        {day,domain,agent} = report
        agent ?= ''
        menu = await get_menu report
        menu ?= ''

        debug "#{report.reference} #{report.now} call_stats #{value} #{name}:#{day}:#{domain}:#{menu}:#{agent}"

Detail per menu/group and agent

        w.emit 'stats:menu+agent', {name,day,domain,menu,agent,value,stats: await stats "S.#{name}:#{day}:#{domain}:#{menu}:#{agent}", value}

Summary per agent

        w.emit 'stats:agent', {name,day,domain,agent,value,stats: await stats "S.#{name}:#{day}:#{domain}:*:#{agent}", value}

Summary per menu/group

        w.emit 'stats:menu', {name,day,domain,menu,value,stats: await stats "S.#{name}:#{day}:#{domain}:#{menu}", value}

Summary per domain

        w.emit 'stats:domain', {name,day,domain,value,stats: await stats "S.#{name}:#{day}:#{domain}", value}

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
        redis.call('expire',key, TWO_MONTHS)
        local count = redis.call('hincrby',key,'count',1)
        local sum   = redis.call('hincrby',key,'sum',v)
        return {key,count,sum,max,min}
      '''.replace 'TWO_MONTHS', TWO_MONTHS

      store.defineCommand 'stats', numberOfKeys: 1, lua: lua_stats_code

      stats = (key,value) ->

        data = await store.stats key, value ? 0

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

          stream.on 'data', foot (keys) ->
            return unless keys.length > 0
            values = await store.mget keys...
            for key, i in keys
              cb key, values[i]
            return

          stream.on 'end', ->
            resolve result

          stream.on 'error', reject

      lex_insert = (key,value) ->
        await store.zadd key, 0, value
        await store.expire key, TWO_DAYS

      lex_scan = (key,cb) ->
        last = ''
        while true
          res = await store.zrangebylex key, "(#{last}", '[zzzz', 'limit', 0, 100
          return unless res.length > 0
          for value in res
            await cb value
          last = res[res.length-1]

        return

      enumerate_and_sum = (pattern,field) ->
        new Promise (resolve,reject) ->
          stream = store.scanStream
            match: pattern
            count: 1000

          sum = 0

          stream.on 'data', foot (keys) ->
            return unless keys.length > 0
            for key in keys
              sum += parseInt await store.hget key, field
            return

          stream.on 'end', ->
            resolve sum

          return

Try: `SORT #{list} ALPHA NOSORT GET *->#{fieldname}`

      {save_agent_name,get_agent_name,get_menu,call_step,menu_start,menu_stop,agent_start,agent_stop,call_stats,get_agent_stats,get_menu_stats,get_domain_stats,lex_insert,lex_scan,enumerate_and_sum}
