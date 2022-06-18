# keylimepie.js

keylimepie.js is a JavaScript utility library. it provides utility functions, logging, and more.

keylimepie.js is primarily used in my personal projects, and isn't really meant to be used anywhere else. it contains functions and snippets of code i end up needing to rewrite. if you want a JavaScript utility library, please go use one of the many other better libraries, such as [lodash](https://lodash.com), [ramda](https://ramdajs.com/), or [underscore](https://underscorejs.org/).

# Installation

There are plenty of ways of using keylimepie.js in your project, as it's just a single file.

The most straightforward way is to just download the file and import it:
```sh
wget https://files.infinixi.us/keylimepie.min.js -P .
```
```js
import "./keylimepie.min.js"
// or
require("./keylimepie.min.js")
```

You can also use npm:
```sh
npm i keylimepie.js
```

```js
import "keylimepie.js"
// or
require("keylimepie.js")
```
Make sure to install `keylimepie.js`, and NOT `keylimepie`. [That package](https://www.npmjs.com/package/keylimepie) is completely unrelated.

You can also use this, although it is quite insecure and you should use one of the above methods instead:
```js
fetch("https://files.infinixi.us/keylimepie.min.js").then(r => r.text()).then(c => {eval(c)}).catch(e => {throw e})

// node specific:
require("https").get("https://files.infinixi.us/keylimepie.min.js", (r) => {var d = "";r.on("data", (c) => {d+=c});r.on("end", () => {eval(d)})}).end()
```

keylimepie.js will by default add all of it's methods to `global` if you're on Node.JS, `window` if you're on the browser, and `globalThis` anywhere else.

So:
```js
// after importing keylimepie.js

console.log(global.Lime.version)
// or
console.log(window.Lime.version)
// or
console.log(globalThis.Lime.version)
// or
console.log(Lime.version)
```

# Reference

## `Lime`

### `Lime.version`

String that contains the current version of keylimepie.

```js
const ver = Lime.version

console.log(`We're running v${ver}!`)
// We're running v1.0.0!
```

### `Lime.platform`

String that contains what JavaScript platform keylimepie is running on.

```js
const platform = Lime.platform

switch (platform) {
	case "node":
		console.log("We're running on Node.JS!")
		break
	case "browser":
		console.log("We're running on the browser!")
		break
	default:
		console.log("We're running JavaScript!")
		break
}
```

## `Logger`

A simple but easy to use logger intended to replace `console.log`. Available globally as `Logger`, like the `Lime` object.

### `Logger.init(fileStream: WritableStream)`

Initializes the logger with exclusive functions for each logging type (`Logger.info()`, `Logger.warn()`, etc), and optionally sets up the fileStream for logging to a file.

You can provide any WritableStream, and the log will log to it, without colors.

```js
import fs from "fs"
// or
const fs = require("fs")

Logger.init(fs.createWriteStream("log.txt", {flags: "a"}))

Logger.info("Hello!")
Logger.log("info", "Hello!") // these do the same thing, but Logger.info will only be accessible after Logger.init has been called.
```

### `Logger.log(type: string, message: string)`

Writes a log message with a type and a message. This method is a convenience, as you should use the type-specific logging functions as shown above.

```js
Logger.log("info", "Hello world!") // will run without Logger.init()
Logger.info("Hello world!") // Uncaught TypeError: Logger.info is not a function
```

### `Logger.config`

Configuration object for the logger. It can be changed via `Logger.config = {...}`. It must be changed before `Logger.init()`.

The config has 6 main options:
- **align** - A function that returns a string, for aligning the different types.
```js
// Without align:
[2022-06-10 15:45:26] [VERBOSE] : OK
[2022-06-10 15:45:26] [ERROR] : b

// With align:
[2022-06-10 15:45:26] [VERBOSE] : OK
[2022-06-10 15:45:26]   [ERROR] : b
```
- **format** - A function that takes the type, message, and a boolean determining color, and returns the log message to be used.
- **fileStream** - A `WritableStream` that the logger will write to with each log, without colors.
- **newline** - An object specifying the newline string to place at the end of each log message.
  - **newline.stdout** - The string to be placed at the end of each log to `stdout`. Contains a newline (`\n`) and a color reset (`\x1b[0m`) by default.
  - **newline.file** - The string to be placed at the end of each log the `fileStream`. Contains a newline (`\n`) by default.
- **timestamp** - A function that expects a `Date` object, and returns a string, to be used for a timestamp in each log message. Returns in `YYYY-MM-DD HH:MM:SS` format by default.
- **types** - An object of types to be used in the logger.
  - **type.stdout** - Boolean determining if this log type should go to `stdout` or not.
  - **type.file** - Boolean determining if this log type should go to the `fileStream` or not.
  - **type.color** - String which should be used to color the `type` when going to `stdout`.

This is the default configuration:
```js
{
	align: function(type) {return " ".repeat(Object.keys(this.types).reduce((a, b) => a.length > b.length ? a : b, "").length - type.length)},
	format: function (type, message, colored) { return `[${this.timestamp(new Date())}] ${colored ? this.types[type].color : ""}${this.align(type)}[${type.toUpperCase()}]${colored ? this.types["verbose"].color : ""} : ${message}` },
	fileStream: null,
	newline: {
		stdout: "\x1b[0m\n",
		file: "\n"
	},
	timestamp: function (date) { return `${date.getFullYear()}-${("0" + (date.getMonth() + 1)).slice(-2)}-${("0" + date.getDate()).slice(-2)} ${date.getHours()}:${date.getMinutes()}:${date.getSeconds()}`},
	types: {
		info: {stdout: true, file: true, color: "\x1b[34m"},
		warn: {stdout: true, file: true, color: "\x1b[33m"},
		error: {stdout: true, file: true, color: "\x1b[31m"},
		debug: {stdout: false, file: true, color: "\x1b[32m"},
		verbose: {stdout: false, file: true, color: "\x1b[37m"},
	}
}
```

## `Time`

### `new Time.Timer(interval: number, limit: number, callback: function) => Object`

Repeatedly runs `callback` every `interval`, eventually stopping at `limit`. This class is essentially just a fancy wrapper over the pre-existing `setInterval()` and `clearInterval()` methods.

```js
const timer = new Time.Timer(500, 5, (x) => {
	console.log(x)
})
timer.start()
// 1
// 2
// 3
// 4
// 5
```

Can also be paused:
```js
const timer = new Time.Timer(500, 5, (x) => {
	console.log(x)
})
timer.start()

await Time.wait(1000)
timer.pause()
await Time.wait(1000)
timer.resume()
```

### `Time.wait(milliseconds: number, callback: function) => Promise or void`

Waits for `milliseconds`. Upon finishing, if `callback` was provided, call it. If it wasn't, then `Time.wait()` will return a Promise, then resolve it once finished.

```js
// asynchronous

await Time.wait(5000)
console.log("Hello world, after 5 seconds!")

// synchronous

Time.wait(5000, () => {
	console.log("Hello world, after 5 seconds, synchronously!")
})
```

### `Time.waitUntil(func: function, callback: function) => Promise or void`

Similar to `Time.wait()`, except instead of waiting for a set amount of time to pass, `Time.waitUntil()` waits until `func` returns true. Upon finishing, if `callback` was provided, call it. If it wasn't, then `Time.wait()` will return a Promise, then resolve it once finished.

```js
var x = 0
setInterval(() => {
	x += 1
}, 500)

// asynchronous

await Time.waitUntil(() => { return x > 5 })
console.log("X is over 5!")

// synchronous

Time.waitUntil(() => { return x > 5 }, () => {
	console.log("X is over 5!")
})
```

You can also wait until a variable exists:
```js
var x

await Time.waitUntil(() => {return x != undefined})
console.log("x exists!")
```

Something like `Time.waitUntilExists(x)` wouldn't be possible, because JavaScript always passes variables by **value**, and not by **reference**. This means it isn't possible to dynamically check a variable's value inside of a function, only the value it was passed when the function was called.

## `Utility`

### `Utility.parseJSONSafe(value: string, reviver: function) => Promise`

Utility function for parsing a JSON string, but returns a promise that resolves or rejects, without throwing an error. Essentially identical to the standard `JSON.parse()` except for that.

```js
Utility.parseJSONSafe('{"part1":"Hello ","part2":"world!"}')
	.then(json => {
		console.log(json.part1 + json.part2)
		// Hello world!
	})
	.catch(err => {
		console.log(`JSON error! ${err}`)
	})
```
### `Utility.stringifyJSONSafe(value: object, replacer: string, space: string) => string`

Utility function for converting an object into a JSON string, but returns a promise that resolves or rejects, without throwing an error. Essentially identical to the standard `JSON.stringify()` except for that.

```js
const object = {
	"part1": "Hello ",
	"part2": " world!"
}

Utility.stringifyJSONSafe(object)
	.then(str => {
		console.log(`JSON string: ${str}`)
		// JSON string: {"part1":"Hello ","part2":" world!"}
	})
	.catch(err => {
		console.log(`JSON error! ${err}`)
	})
```

# Prototypes

## `Array`

### `Array.random() => Array`

Returns a random element from the array.

```js
const fruits = ["Apple", "Orange", "Banana", "Grape"]

console.log(`My favorite fruits are ${fruits.random()}s!`)
// My favorite fruits are Oranges!
```

## `Math`

### `Math.randInt(min: number, max: number) => number`

Function that returns a random number from min to max, with min and max included.

```js
setInterval(() => {
	console.log(`Random number: ${Math.randInt(0,100)}`)
}, 1000)
// Random number: 32
// Random number: 35
// Random number: 1
// Random number: 46
// Random number: 66
// Random number: 45
// Random number: 72
// ...
```

### `Math.range(amount: number) => Array`

Function that returns an array with elements up to amount, intended to be used in for loops. Essentially a port of Python's [range()](https://docs.python.org/3/library/stdtypes.html#range).

```js
for (const x of Math.range(15)) {
	console.log(`This code has been ran ${x} times!`)
}
```

## `Number`

### `Number.toStringWithSeparator() => string`

Returns the number in a string, with commas (or another optional string) separating it.

```js
const balance = 93841

console.log(`Your account currently has: ${balance.toStringWithSeparator()}`)
// Your account currently has: 93,841
```