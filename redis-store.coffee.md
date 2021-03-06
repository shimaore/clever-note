    redis_hour = 3600
    TWO_DAYS = 2*24*redis_hour
    ONE_CALL = 8*redis_hour

    Redis = require 'ioredis'

    decimal = (x) -> parseInt x, 10

Redis Store
-----------

    module.exports = redis_store = ->

      store = new Redis process.env.REDIS_STORE

Sum values (integers, 64 bits signed)

      sum = (key,value,timeout) ->
        timeout ?= TWO_DAYS
        decimal (await store
          .multi()
          .incrby key, value
          .expire key, timeout
          .exec())[0][1]

Count values, up to one day or the specified interval

      count = (key,timeout) ->
        timeout ?= TWO_DAYS
        decimal (await store
          .multi()
          .incr key
          .expire key, timeout
          .exec())[0][1]

Save a value, for up to one call or the specified interval

      save = (key,value,timeout) ->
        timeout ?= ONE_CALL
        store.setex key, timeout, value

Reset a value, returns the previous value

      reset = (key,timeout) ->
        getset key, 0, timeout

Clear a value.

      clear = (key) ->
        store.del key

Get a value

      get = (key) ->
        store.get key

Get a value and set a new one

      getset = (key,value,timeout) ->
        timeout ?= TWO_DAYS
        (await store
          .multi()
          .getset key, value
          .expire key, timeout
          .exec())[0][1]

      getclear = (key,value) ->
        (await store
          .multi()
          .get key
          .del key
          .exec())[0][1]

Compute the difference between the value previously stored and the new value, or null if no value was previously stored.

      delta = (key,value,timeout) ->
        previous = await getset key, value, timeout
        if previous?
          value - decimal previous
        else
          null

      delta_clear = (key,value,timeout) ->
        previous = await getclear key, timeout
        if previous?
          value - decimal previous
        else
          null

      {store,sum,count,save,reset,clear,get,getset,getclear,delta,delta_clear}
