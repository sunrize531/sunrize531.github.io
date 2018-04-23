Title: Regular expressions - guide (part 1)
Slug: regexp-guide-part-1
Date: 2018-04-18 20:00
Category: Guides
Summary: Just few common examples on how to use regexp. `RegExp.prototype.test` in this case. To be continued.
Tags: guide, regexp, javascript

This guide assumes that you're familiar with the subject. If not -
here's [the reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp).
And [a sandbox](https://regex101.com/).

In the part 1 we'll look into `RegExp.prototype.test` method. Here are some use cases:

1. User input validation (90% of all cases).
2. Switch-like checks control structures.
3. Exception handling, cause JavaScript doesn't have normal catch atm, and some errors.

## Single line mode, *without* \m switch

Single line mode means that `.` character class doesn't match with any end of line characters, and also `^` and `$`
will match the start and the end of the input respectively.

Here, how it works:

```javascript
const TEST_PIECE = /123/;
TEST_PIECE.test('123'); // true
TEST_PIECE.test('51234'); // true
TEST_PIECE.test('5\n 123 \t4783'); // true

const TEST_WORD = /\b123\b/;
TEST_WORD.test('123'); // true
TEST_WORD.test('51234'); // false
TEST_WORD.test('5,123-4'); // true
TEST_WORD.test('Look mum, I can count 123-4'); // true

const TEST_INPUT = /^12.*3$/;
TEST_INPUT.test(1243); // js, d'oh...
TEST_INPUT.test('51243'); // false
TEST_INPUT.test('124\n3'); // false, cause \n doesn't match with '.'

const TEST_STARTS_WITH = /^https?:\/\/[\w\d](?:[\w\d-]+[\w\d])?\.net/;
TEST_STARTS_WITH.test("https://a.net"); // true
TEST_STARTS_WITH.test("http://ab-c.net/test"); // true
TEST_STARTS_WITH.test("http://-c.com/test"); // false
TEST_STARTS_WITH.test("mongodb://-c.com/test"); //false

const TEST_ENDS_WITH = /[\/\\][\w\d](?:[\w\d-]+[\w\d])\.(bat|sh)$/
TEST_ENDS_WITH.test("/test/good-file-name.sh"); // true
TEST_ENDS_WITH.test("c:\\windows\\good-file-name.bat"); // true
TEST_ENDS_WITH.test("/test/.hidden-file.bat"); // false
TEST_ENDS_WITH.test("/test/no_exe.com"); // false
```

## Multiline mode, *with* \m switch

Well, I do believe that 90% of use cases for `RegExp.prototype.test` are validation.
For validation purposes, if we really need to make it strict, multiline isn't what we're looking for.
Just cause `^` and `$` won't work as we'd exepect.

Multiline is something we might need for parsers and full text search.
There's gonna be part 2 on that soon.

```javascript
const TEST_MULTILINE = /^12*.3$/m;
TEST_MULTILINE.test("12 3"); // true
TEST_MULTILINE.test("12\n3"); // false
// be careful now, ^$ doesn't mean the entire sting in multiline mode
TEST_MULTILINE.test("123\n43"); // true
```

## Links

* [repl with examples](https://repl.it/@sunrize531/regexp-part-1-test)
* [RegExp reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
* [RegExp sandbox (good one)](https://regex101.com/)
* [Puzzle](https://rampion.github.io/RegHex/)
