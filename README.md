# fetch-mock-jest

Wrapper around [fetch-mock](http://www.wheresrhys.co.uk/fetch-mock) - a comprehensive, isomorphic mock for the [fetch api](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) - which provides an interface that is more idiomatic when working in [jest](https://jestjs.io).

The example at the bottom of this readme demonstrates the intuitive API, but shows off only a fraction of fetch-mock's functionality. Features include:

- helpers for all common http methods and for responding a limited number of times
- delayed responses
- spying on real network requests
- support for advanced fetch behaviours, such as streaming responses and aborting

# Installation

`npm install -D fetch-mock-jest`

## global fetch

`const fetchMock = require('fetch-mock-jest')`

## node-fetch

```
jest.mock('node-fetch', () => require('fetch-mock-jest').sandbox())
const fetchMock = require('node-fetch')
```

# API

## Setting up mocks

Please refer to the [fetch-mock documentation](http://wheresrhys.co.uk/fetch-mock)

All jest methods for configuring mock functions are disabled as fetch-mock's own methods should always be used

## Inspecting mocks

All the built in jest function inspection assertions can be used, e.g. `expect(fetchMock).toHaveBeenCalledWith('http://example.com')`.

`fetchMock.mock.calls` and `fetchMock.mock.results` are also exposed, giving access to manually inspect the calls.

The following custom jest expectation methods, proxying through to `fetch-mock`'s inspection methods are also available. They can all be prefixed with the `.not` helper for negative assertions.

- `expect(fetchMock).toHaveFetched(filter, options)`
- `expect(fetchMock).toHaveLastFetched(filter, options)`
- `expect(fetchMock).toHaveNthFetched(n, filter, options)`
- `expect(fetchMock).toHaveFetchedTimes(n, filter, options)`
- `expect(fetchMock).toBeDone(filter)`

`filter` and `options` are the same as those used by [`fetch-mock`'s inspection methods](http://www.wheresrhys.co.uk/fetch-mock/#api-inspectionfundamentals)

## Tearing down mocks

`fetchMock.mockClear()` can be used to reset the call history

`fetchMock.mockReset()` can be used to remove all configured mocks

# Example

```js
const fetchMock = require('fetch-mock-jest');
const userManager = require('../src/user-manager');

test(async () => {
  const users = [{name: 'bob'}];
  fetchMock
    .get('http://example.com/users', users)
    .post('http://example.com/user', (url, options) => {
      if (typeof options.body.name === 'string') {
        users.push(options.body)
        return 202
      }
      return 400
    })
    .patch({
      url: 'http://example.com/user'
    }, 405)
    
  expect(await userManager.getAll()).toEqual([{name: 'bob'}])
  await userManager.create({name: true})
  expect(await userManager.getAll()).toEqual([{name: 'bob'}])
  await userManager.create({name: 'sarah'})   
  expect(await userManager.getAll()).toEqual([{name: 'bob'}, {name: 'sarah'}])
  fetchMock.clear()
})

```
