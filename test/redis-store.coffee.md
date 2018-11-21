    ({expect} = require 'chai').should()
    EventEmitter = require 'events'

    sleep = (timeout) ->
      new Promise (resolve) ->
        setTimeout resolve, timeout

    describe 'redis-store', ->
      it 'should getset', ->
        {getset,get,store} = (require '../redis-store')()

        await getset 'fo:ba', 'hello'
        (await get 'fo:ba').should.equal 'hello'
        (await getset 'fo:ba', 'world').should.equal 'hello'
        (await get 'fo:ba').should.equal 'world'
        (await getset 'fo:ba', '!').should.equal 'world'
        (await get 'fo:ba').should.equal '!'
        await store.quit()

      it 'should getclear', ->
        {getclear,getset,get,store} = (require '../redis-store')()

        await getset 'ba:fo', 'hello'
        (await getclear 'ba:fo').should.equal 'hello'
        expect(await getset 'ba:fo', 'world').to.be.null
        (await get 'ba:fo').should.equal 'world'
        (await getclear 'ba:fo').should.equal 'world'
        expect(await get 'ba:fo').to.be.null
        await store.quit()

      after ->
