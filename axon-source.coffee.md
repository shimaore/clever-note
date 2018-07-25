    debug = (require 'tangible') 'clever-note:axon-source'

    Axon = require 'axon'

Axon Source
-----------

    module.exports = connect_to_redis_source = seem (w) ->

Make sure you add this service in `cfg.axon.publish_to` on the FreeSwitch+Node.js side.

      source = Axon.socket 'pub'
      source.bind process.env.AXON_SOURCE
      w.once '__end', -> source.close()

      source.on 'message', (message) ->
        w.emit 'report', message

      source
