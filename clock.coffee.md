    moment = require 'moment-timezone'

    sleep = (timeout) -> new Promise (resolve) -> setTimeout resolve, timeout

    module.exports = (w) ->

      timezone  = process.env.DAILY_TIMEZONE
      timezone ?= process.env.REPORTING_TIMEZONE
      timezone ?= 'UTC'

      running = true

      clock = (period,event,delay_range = 0) ->

        while running

          try
            now = moment().tz timezone

When the event is created we are _inside_ the period we want to notify for…

            this_period = now.clone()
              .startOf period

… and we send the event at the end of that period.
We might add a random delay so that not all processes run at the same time on all servers. (This avoids e.g. oversubscribing an upstream server.)

            delay = delay_range*Math.random()

            alarm = now.clone()
              .endOf period
              .add delay, 'm'

            await sleep alarm.diff now

            w.emit event, this_period.clone()

            await sleep 30*1000

          catch error
            console.error "Alarm issue: #{error.stack ? JSON.stringify error}"

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
