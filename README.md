# sst

Super Simple Test Format

This a definition/spec for very simple and portable JavaScript tests.

The goal of this spec is to separate the test format from the runner/framework.

# Basics

* A given JavaScript module (ESM or CommonjS depending on your environment) is the top level container for tests.
* All tests are functions.
* All exported functions are tests.
* A module may export multiple test functions or a single default export function as its only test.

## Advanced

* If the exports contain a `parallel` property set to `true` then the tests can be running in parallel.
* Otherwise, test ordering must be preserved from the original definition.
* A test function *may* have a "testName" property. This name will be used instead of the exported function name
  during reporting.
* That's it!

## Test Examples

Single exports

```javascript
/* commonjs */
const assert = require('assert')
module.exports = () => assert(true, true)

/* esm */
export default () => assert(true, true)
```

Multiple exports

```javascript
/* commonjs */
exports.testPass = async () => { /* noop */ }
exports.testFail = async () => throw new Error('fail please')

/* esm */
export async () => { /* noop */ } as testPass
export async () => throw new Error('fail please') as testFail
```

Name overwrites

```javascript
exports.testHttp = async () => {
  /* some http test code */
}
exports.testHttp.testName = 'http 200 works properly'

/* or, an easier api would be */

const test = (name, t) => {
  t.testName = name
  return t
}

exports.testHttp = test('http 200 works property', async () => {
  /* some http test code */
}
```

## How to implement more advanced features.

Common operations like `setup` and `teardown` can be wrapped around each test function by
any number of test frameworks.

```javascript
/* some-test-framwork */
module.exports = obj => async () => {
  if (obj.setup) await obj.setup()
  await obj.run()
  if (obj.teardown) await obj.teardown()
}
    
/* test.js */
const test = require('some-test-framework')
const assert = require('assert')

let global
exports.myTest = test({setup: () => { global = 'run' }, test: () => assert(global, 'run')})
```
