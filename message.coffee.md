Message expansion
-----------------

    Moment = require 'moment-timezone'

    reporting_timezone  = process.env.REPORTING_TIMEZONE
    reporting_timezone ?= 'UTC'

    set_day = (report) ->
      stamp = Moment report.now
        .tz reporting_timezone

      report.day = stamp
        .format 'YYYY-MM-DD'
      report.time = stamp
        .format 'HH:mm'

      report

    set_domain = (report) ->
      return if report.domain?

      if report.number_domain?
        report.domain = report.number_domain
        return

The number-domain for a call is stored as a tag, currently.


      if report.tags?
        domains = report.tags
          .map (n) -> n.match(/^number_domain:(.+)$/)?[1]
          .filter (n) -> n?

        if domains.length > 0
          report.domain = domains[0]
          return

      if report._in?
        domains = report._in
          .map (n) -> n.match(/^number_domain:(.+)$/)?[1]
          .filter (n) -> n?

        if domains.length > 0
          report.domain = domains[0]
          return

    module.exports = {set_day,set_domain}
