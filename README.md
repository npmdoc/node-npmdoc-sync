# api documentation for  [sync (v0.2.5)](https://github.com/ybogdanov/node-sync)  [![npm package](https://img.shields.io/npm/v/npmdoc-sync.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-sync) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-sync.svg)](https://travis-ci.org/npmdoc/node-npmdoc-sync)
#### Library that makes simple to run asynchronous functions in synchronous manner, using node-fibers.

[![NPM](https://nodei.co/npm/sync.png?downloads=true)](https://www.npmjs.com/package/sync)

[![apidoc](https://npmdoc.github.io/node-npmdoc-sync/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-sync_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-sync/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-sync/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-sync/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Yuriy Bogdanov",
        "email": "chinsay@gmail.com"
    },
    "bugs": {
        "url": "https://github.com/ybogdanov/node-sync/issues"
    },
    "dependencies": {
        "fibers": ">=0.6"
    },
    "description": "Library that makes simple to run asynchronous functions in synchronous manner, using node-fibers.",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "3910bb9b66abee56542e2e70f0ce549031252df6",
        "tarball": "https://registry.npmjs.org/sync/-/sync-0.2.5.tgz"
    },
    "engines": {
        "node": ">=0.5.2"
    },
    "homepage": "https://github.com/ybogdanov/node-sync",
    "license": "MIT",
    "main": "lib/sync",
    "maintainers": [
        {
            "name": "octave",
            "email": "chinsay@gmail.com"
        }
    ],
    "name": "sync",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/ybogdanov/node-sync.git"
    },
    "url": "http://github.com/0ctave/node-sync",
    "version": "0.2.5"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module sync](#apidoc.module.sync)
1.  [function <span class="apidocSignatureSpan">sync.</span>Fiber (fn, callback)](#apidoc.element.sync.Fiber)
1.  [function <span class="apidocSignatureSpan">sync.</span>Fibers ()](#apidoc.element.sync.Fibers)
1.  [function <span class="apidocSignatureSpan">sync.</span>Future (timeout)](#apidoc.element.sync.Future)
1.  [function <span class="apidocSignatureSpan">sync.</span>log (err, result)](#apidoc.element.sync.log)
1.  [function <span class="apidocSignatureSpan">sync.</span>repl ()](#apidoc.element.sync.repl)
1.  [function <span class="apidocSignatureSpan">sync.</span>sleep (ms)](#apidoc.element.sync.sleep)
1.  [function <span class="apidocSignatureSpan">sync.</span>waitFutures ()](#apidoc.element.sync.waitFutures)
1.  object <span class="apidocSignatureSpan">sync.</span>Fibers.prototype
1.  object <span class="apidocSignatureSpan">sync.</span>Future.prototype
1.  object <span class="apidocSignatureSpan">sync.</span>stat

#### [module sync.Fibers](#apidoc.module.sync.Fibers)
1.  [function <span class="apidocSignatureSpan">sync.</span>Fibers ()](#apidoc.element.sync.Fibers.Fibers)
1.  [function <span class="apidocSignatureSpan">sync.Fibers.</span>yield ()](#apidoc.element.sync.Fibers.yield)
1.  number <span class="apidocSignatureSpan">sync.Fibers.</span>fibersCreated
1.  number <span class="apidocSignatureSpan">sync.Fibers.</span>poolSize

#### [module sync.Fibers.prototype](#apidoc.module.sync.Fibers.prototype)
1.  [function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>reset ()](#apidoc.element.sync.Fibers.prototype.reset)
1.  [function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>run ()](#apidoc.element.sync.Fibers.prototype.run)
1.  [function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>throwInto ()](#apidoc.element.sync.Fibers.prototype.throwInto)

#### [module sync.Future.prototype](#apidoc.module.sync.Future.prototype)



# <a name="apidoc.module.sync"></a>[module sync](#apidoc.module.sync)

#### <a name="apidoc.element.sync.Fiber"></a>[function <span class="apidocSignatureSpan">sync.</span>Fiber (fn, callback)](#apidoc.element.sync.Fiber)
- description and source-code
```javascript
function SyncFiber(fn, callback)
{
    var parent = Fiber.current;
    Sync.stat.totalFibers++;

    var traceError = new Error();
    if (parent) {
        traceError.__previous = parent.traceError;
    }

    var fiber = Fiber(function(){

        Sync.stat.activeFibers++;

        var fiber = Fiber.current,
            result,
            error;

        // Set id to fiber
        fiber.id = Sync.stat.totalFibers;

        // Save the callback to fiber
        fiber.callback = callback;

        // Register trace error to the fiber
        fiber.traceError = traceError;

        // Initialize scope
        fiber.scope = {};

        // Assign parent fiber
        fiber.parent = parent;

        // Fiber string representation
        fiber.toString = function() {
            return 'Fiber#' + fiber.id;
        }

        // Fiber path representation
        fiber.getPath = function() {
            return (fiber.parent ? fiber.parent.getPath() + ' > ' : '' )
                + fiber.toString();
        }

        // Inherit scope from parent fiber
        if (parent) {
            fiber.scope.__proto__ = parent.scope;
        }

        // Add futures support to a fiber
        fiber.futures = [];

        fiber.waitFutures = function() {
            var results = [];
            while (fiber.futures.length)
                results.push(fiber.futures.shift().result);
            return results;
        }

        fiber.removeFuture = function(ticket) {
            var index = fiber.futures.indexOf(ticket);
            if (~index)
                fiber.futures.splice(index, 1);
        }

        fiber.addFuture = function(ticket) {
            fiber.futures.push(ticket);
        }

        // Run body
        try {
            // call fn and wait for result
            result = fn(Fiber.current);
            // if there are some futures, wait for results
            fiber.waitFutures();
        }
        catch (e) {
            error = e;
        }

        Sync.stat.activeFibers--;

        // return result to the callback
        if (callback instanceof Function) {
            callback(error, result);
        }
        else if (error && parent && parent.callback) {
            parent.callback(error);
        }
        else if (error) {
            // TODO: what to do with such errors?
            // throw error;
        }

    });

    fiber.run();
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.Fibers"></a>[function <span class="apidocSignatureSpan">sync.</span>Fibers ()](#apidoc.element.sync.Fibers)
- description and source-code
```javascript
function Fiber() { [native code] }
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.Future"></a>[function <span class="apidocSignatureSpan">sync.</span>Future (timeout)](#apidoc.element.sync.Future)
- description and source-code
```javascript
function SyncFuture(timeout)
{
    var self = this;

    this.resolved = false;
    this.fiber = Fiber.current;
    this.yielding = false;
    this.timeout = timeout;
    this.time = null;

    this._timeoutId = null;
    this._result = undefined;
    this._error = null;
    this._start = +new Date;

    Sync.stat.totalFutures++;
    Sync.stat.activeFutures++;

    // Create timeout error to capture stack trace correctly
    self.timeoutError = new Error();
    Error.captureStackTrace(self.timeoutError, arguments.callee);

    this.ticket = function Future()
    {
        // clear timeout if present
        if (self._timeoutId) clearTimeout(self._timeoutId);
        // measure time
        self.time = new Date - self._start;

        // forbid to call twice
        if (self.resolved) return;
        self.resolved = true;

        // err returned as first argument
        var err = arguments[0];
        if (err) {
            self._error = err;
        }
        else {
            self._result = arguments[1];
        }

        // remove self from current fiber
        self.fiber.removeFuture(self.ticket);
        Sync.stat.activeFutures--;

        if (self.yielding && Fiber.current !== self.fiber) {
            self.yielding = false;
            self.fiber.run();
        }
        else if (self._error) {
            throw self._error;
        }
    }

    this.ticket.__proto__ = this;

    this.ticket.yield = function() {
        while (!self.resolved) {
            self.yielding = true;
            if (self.timeout) {
                self._timeoutId = setTimeout(function(){
                    self.timeoutError.message = 'Future function timed out at ' + self.timeout + ' ms';
                    self.ticket(self.timeoutError);
                }, self.timeout)
            }
            Fiber.yield();
        }
        if (self._error) throw self._error;
        return self._result;
    }

    this.ticket.__defineGetter__('result', function(){
        return this.yield();
    });

    this.ticket.__defineGetter__('error', function(){
        if (self._error) {
            return self._error;
        }
        try {
            this.result;
        }
        catch (e) {
            return e;
        }
        return null;
    });

    this.ticket.__defineGetter__('timeout', function(){
        return self.timeout;
    });

    this.ticket.__defineSetter__('timeout', function(value){
        self.timeout = value;
    });

    // append self to current fiber
    this.fiber.addFuture(this.ticket);

    return this.ticket;
}
```
- example usage
```shell
...
})
'''

Parallel execution:

'''javascript
var Sync = require('sync'),
	Future = Sync.Future();

// Run in a fiber
Sync(function(){
	try {
		// Three function calls in parallel
		var foo = asyncFunction.future(null, 2, 3);
		var bar = asyncFunction.future(null, 5, 5);
...
```

#### <a name="apidoc.element.sync.log"></a>[function <span class="apidocSignatureSpan">sync.</span>log (err, result)](#apidoc.element.sync.log)
- description and source-code
```javascript
log = function (err, result)
{
    if (err) return console.error(err.stack || err);
    if (arguments.length == 2) {
        if (result === undefined) return;
        return console.log(result);
    }
    console.log(Array.prototyle.slice.call(arguments, 1));
}
```
- example usage
```shell
...
}

// Run in a fiber
Sync(function(){
	
	// Function.prototype.sync() interface is same as Function.prototype.call() - first argument is 'this' context
	var result = asyncFunction.sync(null, 2, 3);
	console.log(result); // 5

	// Read file synchronously without blocking whole process? no problem
	var source = require('fs').readFile.sync(null, __filename);
    console.log(String(source)); // prints the source of this example itself
})
'''
...
```

#### <a name="apidoc.element.sync.repl"></a>[function <span class="apidocSignatureSpan">sync.</span>repl ()](#apidoc.element.sync.repl)
- description and source-code
```javascript
repl = function () {

    var repl = require('repl');

    // Start original repl
    var r = repl.start.apply(repl, arguments);

    // Wrap line watchers with Fiber
    var newLinsteners = []
    r.rli.listeners('line').map(function(f){
        newLinsteners.push(function(a){
            Sync(function(){
                require.cache[__filename] = module;
                f(a);
            }, Sync.log)
        })
    })
    r.rli.removeAllListeners('line');
    while (newLinsteners.length) {
        r.rli.on('line', newLinsteners.shift());
    }

    // Assign Sync to repl context
    r.context.Sync = Sync;

    return r;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.sleep"></a>[function <span class="apidocSignatureSpan">sync.</span>sleep (ms)](#apidoc.element.sync.sleep)
- description and source-code
```javascript
sleep = function (ms)
{
    var fiber = Fiber.current;
    if (!fiber) {
        throw new Error('Sync.sleep() can be called only inside of fiber');
    }

    setTimeout(function(){
        fiber.run();
    }, ms);

    Fiber.yield();
}
```
- example usage
```shell
...
	// we can use yield here
	// yield();
	
	// or throw an exception!
	// throw new Error('something went wrong');
	
	// or even sleep
	// Sync.sleep(200);
	
	// or turn fs.readFile to non-blocking synchronous function
	// var source = require('fs').readFile.sync(null, __filename)
	
	return a + b; // just return a value
	
}.async() // <-- here we make this function friendly with async environment
...
```

#### <a name="apidoc.element.sync.waitFutures"></a>[function <span class="apidocSignatureSpan">sync.</span>waitFutures ()](#apidoc.element.sync.waitFutures)
- description and source-code
```javascript
waitFutures = function () {
    if (Fiber.current) {
        Fiber.current.waitFutures();
    }
}
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.sync.Fibers"></a>[module sync.Fibers](#apidoc.module.sync.Fibers)

#### <a name="apidoc.element.sync.Fibers.Fibers"></a>[function <span class="apidocSignatureSpan">sync.</span>Fibers ()](#apidoc.element.sync.Fibers.Fibers)
- description and source-code
```javascript
function Fiber() { [native code] }
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.Fibers.yield"></a>[function <span class="apidocSignatureSpan">sync.Fibers.</span>yield ()](#apidoc.element.sync.Fibers.yield)
- description and source-code
```javascript
yield = function () { [native code] }
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.sync.Fibers.prototype"></a>[module sync.Fibers.prototype](#apidoc.module.sync.Fibers.prototype)

#### <a name="apidoc.element.sync.Fibers.prototype.reset"></a>[function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>reset ()](#apidoc.element.sync.Fibers.prototype.reset)
- description and source-code
```javascript
function reset() { [native code] }
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.Fibers.prototype.run"></a>[function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>run ()](#apidoc.element.sync.Fibers.prototype.run)
- description and source-code
```javascript
function run() { [native code] }
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.sync.Fibers.prototype.throwInto"></a>[function <span class="apidocSignatureSpan">sync.Fibers.prototype.</span>throwInto ()](#apidoc.element.sync.Fibers.prototype.throwInto)
- description and source-code
```javascript
function throwInto() { [native code] }
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.sync.Future.prototype"></a>[module sync.Future.prototype](#apidoc.module.sync.Future.prototype)



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
