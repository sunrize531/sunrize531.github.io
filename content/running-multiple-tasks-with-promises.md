Title: Promises - Running multiple tasks
Date: 2018-04-06 20:00
Category: Patterns
Tags: promise, javascript, node, pattern
Summary: Very simple way to run multiple asyncronous tasks in parallel or in series.

# Run asyncronous functions in parallel

Just as simple as it can be.

```javascript
Promise.all([task1(), task2(1, 2, 3), task4('firstArgument')])
    .then(function ([result1, result2, result3]) {
        // This will be called when all the tasks resolved.
    })
    .catch(function (err) {
        // This will be called if at list one of the tasks was rejected.
    });
```

# Run asyncronous functions in series

Little bit trickier here, still simple enough.

```javascript
let results = [];
Promise.resolve()
    .then(function () {
        return task1().then(function (r) {
            results.push(r);
            return r;
        });
    })
    .then(function () {
        return task2(1, 2, 3).then(function(r) {
            results.push(r);
            return r;
        });
    })
    .then(function () {
        return task4('firstArgument').then(function (r) {
            results.push(r);
            return r;
        });
    })
    .then(function () {
        // Triggers when all the tasks finished
        let [result1, result2, result3] = results;
    })
    .catch(function (err) {
        // Triggers if any of previous things was rejected.
    });
```

# Applying some async method to iterable in series

```javascript
function mapSeries(iterable, method) {
    let p = Promise.resolve();
    let r = [];
    for (let item in iterable) {
        p = p.then(function () {
            return method(item)
                .then(function (result) {
                    r.push(result);
                });
        });
    }

    return p.then(function () {
        return r;
    });
}

eachSerise([1, 2, 3], someFunction)
    .then(function (results) {
        // Triggers when all the tasks finished
    })
    .catch(function (err) {
        // Triggers when any of tasks or previous handler was rejected.
    })
```