    PouchDB = require 'pouchdb-core'
      .plugin require 'pouchdb-adapter-http'
      .plugin require 'pouchdb-mapreduce'

    module.exports = pouchdb_changes = (w) ->

      source = new PouchDB process.env.POUCHDB_SOURCE

      stream = source.changes
        live: true
        include_docs: true
        since: 'now'

      stream.on 'change', (change) ->
        w.emit 'change', change
