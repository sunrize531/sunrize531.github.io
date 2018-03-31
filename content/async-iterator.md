Title: Asyncronous Iterator
Date: 2018-03-03 12:00
Category: Patterns
Summary: I needed to run something async on mongo cursor and I didn't want to read everything into memory, for obvious reasons

# How to asyncroniously iterate through asyncronous iterator

I needed to run something async on mongo cursor and I didn't want to read everything into memory, for
obvious reasons. So here it is.

## Iterator itself

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

Or it can be class.

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

Not really async, I know... But we got an idea.

## Iteratee

So, this is how we iterate:

```javascript
function action(item)  {
    return doSomethingAsyncWithItem(item)
        .then(function (result) {
            console.info(result);
            return;
        });
}

function iterate(iterator, iteratee) {
    return iterator().then(function (item) {
        if (item !== DONE) {
            return iteratee(item).then(function () {
                iterate(iterator, iteratee);
            });
        } else {
            return DONE;
        }
    });
}

iterate(new AsyncIterator(5), action)
    .catch(function (err) {
        console.error(err.stack);
        process.exit(1);
    });
```

## MongoDB example

```javascript
const {MongoClient} = require('mongodb');
const db = MongoClient.connect('mongodb://localhost')
    .then(function (client) {
        return client.db('items');
    })
    .then(function (db) {
        let byCategory = new Map();
        function readItems(category) {
            return db.collection('items').find({category: category._id}).toArray()
                .then(function (items) {
                    byCategory.set(category, items);
                });
        }

        function iterate(cursor, iteratee) {
            return cursor.next()
                .then(function (i) {
                    if (i) {
                        return iteratee(i)
                            .then(function () {
                                return iterate(cursor, iteratee);
                            });
                    }
                    return byCategory;
                });
        }

        // We don't need to implement iterator here, just cursor will do
        return iterate(db.collection('categories').find(), readItems);
    })
    .catch(function (err) {
        console.error(err.stack);
        process.exit(1);
    });
```