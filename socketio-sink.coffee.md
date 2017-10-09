    debug = (require 'tangible') 'clever-note:socketio-sink'

    IO = require 'socket.io-emitter'
    Redis = require 'ioredis'


This bypasses / is faster than / duplicates the `register` code in `spicy-action/internal-message-broker`.
Note however how this code does NOT implement the default ("global") event sinks.

    notification_rooms = /^\w+:/

    module.exports = send_to_socketio = (w) ->
      return unless process.env.REDIS_IO
      redis = new Redis process.env.REDIS_IO

      io = IO redis

      make_emitter = (event) ->
        (report) ->
          return unless report._notify and report._in?

          destination = null
          report._in
          .filter (room) -> room.match notification_rooms
          .forEach (room) ->
            destination ?= io
            destination = destination.to room

          destination?.emit event, report

      w.on 'report', make_emitter 'report'
      w.on 'queuer', make_emitter 'queuer'
