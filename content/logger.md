Title: Adding a prefix to `console.info`
Slug: console-info-with-prefix
Date: 2018-05-21 21:00
Category: Snippets
Summary: How to add a prefix to logs without screwing the callstack
Tags: snippet, logging, console, client

So, we need to filter the massive log in the console. Off course, We could filter by filename, plain text, whatever,
but sometimes it would be easier just to add a prefix to all log entries in the module / lib / whatever. As long
as we're banned from using the debugger for some reason ;)

Most obvious implementation would be a simple wrapper, something like this:

```javascript
const PREFIX = "[the-lib]";
function log() {
    console.info(PREFIX, ... arguments);
}
```

Unfortunately this would screw the links to sources, that chrome console so helpfully adds.
All links would point to the place where `console.info` is called, and that would be always the same place, 
inside of `log` wrapper.

So, we need to call original `console.info` then. Here's how it may work:

```javascript
const LOG_METHODS = ['log', 'info', 'debug', 'warn', 'error'];

function noop() {}

class Logger {
    constructor(prefix = '[logger]') {
        // Check if console even exists
        if (window && window.console) {
            let console = window.console;
            for (let methodName of LOG_METHODS) {
                let method;
                if (console.hasOwnProperty(methodName)) {
                    method = console[methodName];
                } else {
                    method = console.log || noop;
                }
                this[methodName] = Function.prototype.bind.call(method, console, prefix);
            }
        } else {
            for (let method of LOG_METHODS) {
                this[method] = noop;
            }
        }
    }
}

let logger = new Logger('[bugger]');
logger.info('One, two', {check: true}, ['the', 'microphone']);
```

And [REPL](https://repl.it/@sunrize531/logs-with-prexis)
