[![Build Status](https://travis-ci.org/storehouse/engen.svg?branch=master)](http://travis-ci.org/storehouse/engen)

engen
=====

Async control flow using pure ES6 generators.

Requires ES6 generator support. Tested on Node 0.11 with the `--harmony` flag.

API
---

### engen.run()

Run generator based code from callback-land.

```javascript
var g = require('engen');

function *f() {
  yield g.wait(2000);
  return 'done';
}

g.run(f(), function(err, result) {
  console.log(err); // null
  console.log(result); // 'done'
});

```

### engen.wrap()

Wrap callback-style code and call it from generator-land.

```javascript
var g = require('engen');
var readFile = g.wrap(require('fs').readFile);

function *f() {
  yield g.wait(1000);
  return yield readFile('test.js');
}

g.run(f(), function(err, res) {
  console.log(err); // null
  console.log(res); // <Buffer ...>
});
```

By default, `wrap()` expects callbacks to have the standard Node signature:

```javascript
function(error, result) {}
```

To wrap an unusual callback, you can pass in a function as a second parameter
to `wrap()` to perform custom handling of the callback parameters.

To handle multiple return values and return them as an array:

```javascript
var g = require('engen');

var f = g.wrap(function(cb) {
  cb(err, 12, 34, 56);
}, function(err, a, b, c) {
  if (err) throw err;
  return [a, b, c];
});
```

To handle callbacks without an `err` parameter:

```javascript
var g = require('engen');

var f = g.wrap(function(cb) {
  cb(12, 34, 56);
}, function(a, b, c) {
  return [a, b, c];
});
```

For your convenience, these two handlers are built in, as `g.multipleReturnCallback` and `g.noErrorCallback`.

### engen.wait()

Generator-based version of `setTimeout`, provided for convenience.

Yields
------

Yielding an array of generators will run them in parallel and return an array
of ordered results.

```javascript
var g = require('engen');

function *a() {
  yield g.wait(2000);
  return 'a';
}

function *b() {
  yield g.wait(2000);
  return 'b';
}

function *f() {
  return yield [a(), b()]; // takes 2000ms
}

g.run(f(), function(err, res) {
  console.log(err); // null
  console.log(res); // ['a', 'b']
});
```

Yielding an object of generators behaves identically to yielding an array, but
the resulting value is an object.

```javascript
var g = require('engen');

function *a() {
  yield g.wait(2000);
  return 'a';
}

function *b() {
  yield g.wait(2000);
  return 'b';
}

function *f() {
  return yield {a: a(), b: b()}; // takes 2000ms
}

g.run(f(), function(err, res) {
  console.log(err); // null
  console.log(res); // {a: 'a', b: 'b'}
});
```

Yielding anything other than a generator will return that value.

```javascript
var g = require('engen');

function *a() {
  yield g.wait(1000);
  return 'a';
}

function *f() {
  return yield [a(), false, null, {}];
}

g.run(f(), function(err, res) {
  console.log(err); // null
  console.log(res); // ['a', false, null, {}];
});
```

License
-------

MIT
