    {heal} = (require 'tangible') 'clever-note:redis-upstream'
    seem = require 'seem'

    redis_hour = 3600
    redis_day = 24*redis_hour
    EXPIRE = 32*redis_day

    Redis = require 'ioredis'

    ABANDONNED = 'ab'
    ANSWERED = 'an'
    EGRESS = 'eg'
    INGRESS = 'ig'
    IN_CALL = 'ic'
    LCR = 'lc'
    MENU = 'mn'
    MISSED = 'mi'
    PRESENTED = 'pr'
    WRAP_UP = 'wu'

Redis Store
-----------

    module.exports = redis_upstream = (w) ->

      return unless process.env.REDIS_UPSTREAM?

      store = new Redis process.env.REDIS_UPSTREAM

      desired_names = [
        ANSWERED
        MISSED
        INGRESS
        LCR
      ]

      w.on 'stats:agent', (stat) ->
        {name,day,agent} = stat
        if name in desired_names
          key = "Se:#{agent}"
          heal store
            .multi()
            .hset key, "#{day}:#{name}", stat
            .expire key, EXPIRE
            .exec()
        return
