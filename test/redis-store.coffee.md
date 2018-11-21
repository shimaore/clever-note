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

      it 'should delta', ->
        {delta,delta_clear,get,store} = (require '../redis-store')()

        await delta 'pandas', 42
        (await delta_clear 'pandas', 52).should.equal 10
        expect(await get 'pandas').to.be.null
        await store.quit()

      it 'should handle multiple delta/delta_clear', ->
        {delta,delta_clear,get,store} = (require '../redis-store')()

        expect(await delta 'pandas', 42).to.be.null
        (await delta 'pandas', 45).should.equal 3
        (await delta_clear 'pandas', 52).should.equal 7
        expect(await delta_clear 'pandas', 53).to.be.null
        await store.quit()

      after ->
