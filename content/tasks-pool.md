Title: Promises - Tasks pool
Date: 2018-04-16 20:00
Category: Patterns
Tags: promise, javascript, node, pattern
Summary: Running tasks asyncronously in small batches.

# Running tasks in small batches

So, we have a pool of tasks, but we don't want to launch them simulatneously.
Instead, we're going to run them in small batches, like this:

```javascript
// This is our task
function wait(n) {
    return new Promise(function (r) {
       setTimeout(r, Math.random() * 1000, n);
    });
}

// And this is an iterator
let queue = new Set();
let results = [];
const LIMIT = 3;
function drainPool(pool) {
    while (pool.length && queue.size < limit) {
        let p = wait(pool.pop())
            .then(function (n) {
                results.push(n);
                queue.delete(p);
                return drainPool(pool);
            });
        queue.add(p);
    }
    return Promise.all([... queue]);
}

let n = 25;
let pool = [];
for (let i = 0; i < n; i++) { pool.push(i); }

drainPool(pool).then(function () {
    console.info('done');
    console.info(results);
});
```

[Check it out at repl.it](https://repl.it/@sunrize531/async-pool)

# We can do this with async iterator too

Example with mongo cursor:

```javascript
const db = connectDB();
const cursor = db.collection('items').find().sort({time: -1}).limit(100);
const LIMIT = 6;
const queue = new Set();
const results = [];
function drainPool() {
    if (queue.size() < limit) {
        return cursor.next()
            .then(function (item) {
                if (item) {
                    let p = drainPool(item).then(function (r) {
                        queue.remove(p);
                        resulst.push(r);
                        return process();
                    });
                    queue.add(p);
                    return p;
                }
            })
    } else {
        return Promise.all([... queue])
    }
}

drainPool();
```

Not too sure this one works though... Gonna need to check it at some point.