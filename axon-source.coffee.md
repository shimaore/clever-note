    debug = (require 'tangible') 'clever-note:axon-source'

    Axon = require 'axon'

Axon Source
-----------

    module.exports = axon_source = (w) ->

Make sure you add this service in `cfg.axon.publish_to` on the FreeSwitch+Node.js side.

      source = Axon.socket 'sub'
      source.bind process.env.AXON_SOURCE
      w.once '__end', -> source.close()

      source.on 'message', (message) ->
        w.emit 'report', message.value

      source
