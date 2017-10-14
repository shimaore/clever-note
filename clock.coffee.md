    moment = require 'moment-timezone'
    {debug} = (require 'tangible') 'clever-note:clock'

    module.exports = (w) ->

      timezone  = process.env.DAILY_TIMEZONE
      timezone ?= process.env.REPORTING_TIMEZONE
      timezone ?= 'UTC'

      clock = (period,event,delay_range = 0) ->
        debug 'clock', {period,event,delay_range}

        restart = ->
          clock period, event, delay_range
          return

        now = moment().tz timezone

        this_period = now.clone()
          .startOf period

We might add a random delay so that not all processes run at the same time on all servers. (This avoids e.g. oversubscribing an upstream server.)

        delay = delay_range*Math.random()

        alarm = now.clone()
          .endOf period
          .add delay, 'm'

        send = ->
          debug 'send', {event,this_period}
          w.emit event, this_period
          setTimeout restart, 30*1000
          return

        setTimeout send, alarm.diff now
        return

Daily
=====

This sends a `daily` message (with the date, including timezone, as string) early in the morning so that handlers can generate daily reports for the previous day. The event is generated within 30 minutes of the end of day.

      clock 'day', 'daily', 30

Hourly
======

This sends a 'hourly' message (with the date and hour, including timezone, as string) within one minute of the end of the hour.

      clock 'hour', 'hourly', 1
      return
