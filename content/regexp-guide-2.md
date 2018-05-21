Title: Regular expressions - guide (part 2)
Slug: regexp-guide-part-2
Date: 2018-05-19 12:00
Category: Guides
Summary: Just few common examples on how to use regexp. Now `RegExp.prototype.exec`. To be continued.
Tags: guide, regexp, javascript

This guide assumes that you're familiar with the subject. If not -
here's [the reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp).
And [a sandbox](https://regex101.com/).

Also have a look at [part 1](http://bugger.online/regexp-guide-part-1.html)

So, now let's see what we can do with `RegExp.prototype.exec`.
Mostly, it's the same as `String.prototype.match` but there're few differences.
`String.prototype.match` would also accept string as an argument, and will make a RegExp
of it under the hood. Javascript, d'oh. Better declare our pattern at start and run exec on it.
Also we can actually iterate through matches.

So, the common cases for using `RegExp.prototype.exec`

1. Parsing things, that are **not** HTML / XML. Seriously, don't parse HTML with RegExp, it's silly.
2. Finding things in texts.
3. Iterating through matching patterns in text.
4. Logs parsing (basically, all of 1-3)

## Finding subpatterns

So, the main difference of this method and `RegExp.prototype.test`
is that it returns little more the just true / false.

Also... Please note, that while I'm gonna do url parsing examples, using RegExp for
that is almost as silly as HTML parsing (instead of using DOM model), especially if you're on node.

Node has [builtin module](https://nodejs.org/dist/latest-v8.x/docs/api/url.html)
for that.

So, here, how it works. And don't forget to mark non-capturing groups with `?:`.

```javascript
// See if we can find a signature in the email
let [signature, firstName, lastName] = /(?:Kind regards|Bye|Cheers)\s+(\w+)(?:\s+(\w+))/m
    .exec('Hi, Annie.\nThanks for the quick responnse.\n\nKind regards\nIvan Filimonov');
console.info(firstName, lastName); // Ivan Filimonov
```

```javascript
// Get protocol and domain name from url
let [url, protocol, domain] = /^(https?):\/\/((?:[\w\d]+\.)+\w+)/
    .exec('http://bugger.online/test');
console.info(protocol, domain); // http bugger.online
```

```javascript
// Get domain name and path from url
let [url, domain, path] = /^https?:\/\/((?:[\w\d]+\.)+\w+)(?:\/((?:[\w\d]+\/?)+))/
    .exec('http://bugger.online/tags/test');
console.info(domain, path); // bugger.online tags/test
```

## Finding all matches

Another application for `RegExp.prototyp.exec` - finding all matches in the string / text
and iterating through them.

```javascript

// Iterating through matches
// Let's find markdown italic macros
// Note! JS sadly doesn't support lookbehind assertions so,
// escaping (`\*`) won't work like it should.
// Also note, that nested expression in second line doesn't match.
let PATTERN = /([_*])([^_*]+)\1/g;
let text = `Quick *brown* fox jumps over the _lazy_ dog
*Quick _brown_ fox jumps over* the lazy dog`
let matched;
while ((matched = PATTERN.exec(text))) {
    let [full, delimiter, content] = matched;
    console.info(full, content);
}
```

## REPL with all the examples

[Here](https://repl.it/@sunrize531/regexp-part-2-exec)

## Further reading

* [Advanced Regex Tutorial]([http://www.rexegg.com/regex-disambiguation.html]) - few things aren't widely supported in JS,
  but still good to learn, esp if you have to work with better regex implementation in other languages (python, ruby, java, perl)