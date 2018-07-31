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

    module.exports = {set_day,set_domain}
