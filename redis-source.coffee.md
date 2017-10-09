    seem = require 'seem'

    debug = (require 'tangible') 'clever-note:redis-source'

    Redis = require 'ioredis'

Redis Source
------------

    module.exports = connect_to_redis_source = seem (w) ->

      source = new Redis process.env.REDIS_SOURCE

Subscribe to events defined in `needy-toothpaste`.

      yield source
        .subscribe 'huge-play:report', 'huge-play:queuer', 'huge-play:add'

Route events defined in `needy-toothpaste`.

      source.on 'message', (channel,message) ->
        if $ = channel.match /^huge-play:(.+)$/
          event = $[1]
          w.emit event, JSON.parse message

      source
