    debug = (require 'tangible') 'clever-note/call-events'
    {set_day,set_domain} = require './message'

Incoming events
---------------

    module.exports = (w) ->

Simple report (via `@notify` in `huge-play/middleware/setup`).

      w.on 'report', (report) ->

        set_day report
        set_domain report

        if report.state?
          w.emit "state:#{report.state}", report
        if report.event?
          w.emit "event:#{report.event}", report
        if report.old_state?
          w.emit "leave:#{report.old_state}", report

Queuer report (via `agent.notify` in `huge-play/middleware/client/queuer`).

      w.on 'queuer', (report) ->

        set_day report
        set_domain report

        if report.state?
          w.emit "state:#{report.state}", report
        if report.event?
          w.emit "event:#{report.event}", report
        if report.old_state?
          w.emit "leave:#{report.old_state}", report

Call start and end
------------------

in huge-play/middleware/client/setup

      w.on 'state:incoming-call-client-side', (report) ->
        w.emit "new-#{report.direction}-call", report

in huge-play/middleware/carrier/setup

      w.on 'state:incoming-call-carrier-side', (report) ->
        w.emit "new-#{report.direction}-call", report

in huge-play/middleware/cdr

      w.on 'state:end', (report) ->
        w.emit "end-#{report.cdr_report.direction}-call", report
