    PouchDB = require 'ccnq4-pouchdb'

    module.exports = pouchdb_changes = (w) ->

      source = new PouchDB process.env.POUCHDB_SOURCE

      stream = source.changes
        live: true
        include_docs: true
        since: 'now'

      stream.on 'change', (change) ->
        w.emit 'change', change
