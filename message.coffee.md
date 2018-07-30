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
        if $ = report.number_domain.match /^number_domain:(.+)$/
          report.domain = $[1]
        else
          report.domain = report.number_domain
        return

    module.exports = {set_day,set_domain}
