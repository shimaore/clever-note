    moment = require 'moment-timezone'
    {debug} = (require 'tangible') 'clever-note:start'

    module.exports = (w) ->

      timezone  = process.env.DAILY_TIMEZONE
      timezone ?= process.env.REPORTING_TIMEZONE
      timezone ?= 'UTC'

      now = moment().tz timezone

      w.emit '__start', now
      return
