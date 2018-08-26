    debug = (require 'tangible') 'clever-note:axon-source'

    Axon = require '@shimaore/axon'

Axon Source
-----------

    module.exports = axon_source = (w) ->

Make sure you add this service in `cfg.axon.publish_to` on the FreeSwitch+Node.js side.

      source = Axon.socket 'sub'
      source.bind process.env.AXON_SOURCE
      w.once '__end', -> source.close()

      source.on 'message', (message) ->
        report = Object.assign {_key:message.key,_id:message.id}, message.value
        switch message.value?.report_type

          when 'call'
            type = message.key.split(':')[0]
            switch type
              when 'number_domain'
                w.emit 'report', report
              when 'agent'
                w.emit 'report:agent', report
              when 'endpoint'
                no

          when 'queuer'
            type = message.id.split(':')[0]
            switch type
              when 'agent'
                w.emit 'queuer', report
              when 'call'
                w.emit 'queuer:call', report

      source
