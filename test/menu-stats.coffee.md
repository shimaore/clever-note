    ({expect} = require 'chai').should()
    EventEmitter = require 'events'

    sleep = (timeout) ->
      new Promise (resolve) ->
        setTimeout resolve, timeout

    describe 'menu-stats', ->
      it 'should menu_stop / menu_start', ->
        {get,store} = (require '../redis-store')()
        w = new EventEmitter
        {menu_start,menu_stop} = (require '../menu-stats') w

        our_menu = "hello-#{Math.random()}"

        report =
          reference: 'cat'
          menu: our_menu
          now: 42000

        await menu_start report, 'count'
        (await get 'count-cat').should.equal '42000'

        report =
          reference: 'cat'
          menu: our_menu
          now: 45000
        await menu_start report, 'count'
        (await get 'count-cat').should.equal '45000'

        new_stats = null

        w.on 'stats:menu', ({name,menu,stats}) ->
          menu.should.equal our_menu
          name.should.equal 'count'
          new_stats = stats

        report =
          reference: 'cat'
          now: 72000
        await menu_stop report, 'count'
        expect(await get 'count-cat').to.be.null

        await sleep 1000

        new_stats.should.have.property 'count', 1
        new_stats.should.have.property 'sum', 27

        await store.quit()
        w.emit '__end'
        return
