Title: Asyncronous Iterator
Date: 2018-03-03 12:00
Category: Patterns
Summary: I needed to run something async on mongo cursor and I didn't want to read everything into memory, for obvious reasons

# How to asyncroniously iterate through asyncronous iterator

I needed to run something async on mongo cursor and I didn't want to read everything into memory, for
obvious reasons. So here it is.

## Iterator itself

Well, we don't really need to implement mongo cursor to iterate over mongo cursor, but what if it's not mongo cursor.
Here's very basic example:

```javascript
const DONE = {};
let count = -1;

function asyncIterator() {
    if (++count > 10) {
        return Promise.resolve(count);
    } else {
        return Promise.resolve(DONE);
    }
}
```

Or it can be class, like mongo cursor, even...

```javascript
const DONE = {};

class AsyncIterator() {
    constructor(n) {
        this._count = -1;
    }

    next() {
        if (++this._count > 10) {
            return Promise.resolve(this._count);
        } else {
            return Promise.resolve(DONE);
        }
    }
}
```

This thing isn't really async, I know... But we got an idea. For example, if we wanted to walk the directory,
it would be something like this:

```javascript
```