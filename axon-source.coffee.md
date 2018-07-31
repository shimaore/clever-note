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
        report = Object.assign {_key:message.key,_id:message.id}, message.value
        type = message.key.split(':')[0]
        switch message.value.report_type
          when 'call'
            switch type
              when 'number_domain'
                w.emit 'report', report
              when 'agent'
                w.emit 'report:agent', report
              when 'endpoint'
                no
          when 'queuer'
            switch type
              when 'agent' # agent notification
                w.emit 'queuer', report
              when 'number_domain' # call notification
                w.emit 'queuer:call', report

      source
