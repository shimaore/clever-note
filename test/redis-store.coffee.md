    seem = require 'seem'
    (require 'chai').should()
    EventEmitter = require 'events'

    sleep = (timeout) ->
      new Promise (resolve) ->
        setTimeout resolve, timeout

    describe 'redis-store', ->
      it 'should getset', seem ->
        {getset,get,store} = (require '../redis-store')()

        yield getset 'fo:ba', 'hello'
        (yield get 'fo:ba').should.equal 'hello'
        (yield getset 'fo:ba', 'world').should.equal 'hello'
        (yield get 'fo:ba').should.equal 'world'
        (yield getset 'fo:ba', '!').should.equal 'world'
        (yield get 'fo:ba').should.equal '!'

        yield store.quit()
