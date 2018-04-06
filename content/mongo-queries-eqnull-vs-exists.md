Title: Mongo performance - finding documents with missing field.
Date: 2018-03-31 12:00
Category: MongoDB, Weirdness
Summary: Optimising mongo query for documents where certain indexed field doesn't exist or less then value.
Tags: mongodb, optimisation, guide

So, we need to find the documents with value in certain field less then given.
It's not a problem if the field exists in all the documents and it's indexed, of course.
I.e. our collection looks like:

```
{"_id" : "00001", "value": 1}
{"_id" : "00002", "value": 1}
{"_id" : "00003", "value": 1}
{"_id" : "00004", "value": 4}
```

And it has an index `{name: value, key: {value: 1}}`. Then we can just do:

```
db.getCollection('items').find({value: {$lt: 4}});
```

This would return documents with ids `['00001', '00002, '00003']`. Works like charm, super fast,
doesn't depend on number of documents, etc. But what if the value doesn't exist in all the documents?

# Including documents with non-existing values

If our collection looks like:

```
{"_id" : "00001", "value": 1}
{"_id" : "00002"}
{"_id" : "00003", "value": null}
{"_id" : "00004", "value": 4}
```

the query above would return the documents where value **does** exist, is number (or can be coerced to number) and less then 7.
And there would be just one matching document: `'00001'`. Which probably isn't what we wanted.

There're 2 ways to query that:

1\. Check for null

```
db.getCollection('items').find({$or: [{field: null}, {field: {$lt: 7}}]});
```

2\. Check for non-existing field

```
db.getCollection('items').find({$or: [{field: {$exists: false}}, {field: {$lt: 7}}]});
```

Query #1 would return all the documents we want, i.e. `['00001', '00002, '00003']`. But there's a caveat.
It appears that it does `COLSCAN` if there're lots of documents with missing `value` field.

Query #2 would **not** return `'00003'`, cause `value` field does exist.
But it apparently runs `FETCH` which is much much faster.

# Avoid using $ne, unless absolutely needed

And since we're on it anyway... Just to mention that mongo indexes don't quite work for `$ne` operator.

See [mongo documentation](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne):

> The inequality operator $ne is not very selective since it often matches a large portion of the index. As a result, in many cases, a $ne query with an index may perform no better than a $ne query that must scan all documents in a collection. See also [Query Selectivity](https://docs.mongodb.com/manual/core/query-optimization/#read-operations-query-selectivity).

Trying to trick mongo and doing `{value: {$gt: 5, $lt: 5}}` won't help either, cause in this case it looks like it applies two conditions to the index separately and
then intersects results. Something similar it does when it evaluates `$ne`. If you really need to do this - you'd need to narrow the query somehow.
Although this is actually the case for separate article.

# Conclusion

1. Try to pre-populate indexed fields, if it's possible
2. If you're going to use comparison operators - don't mix up the types, you might be surprised with results.
3. If you need to include the documents with missing indexed field - use `{value: {$exist: false}}` in favour of `{value: null}`.
