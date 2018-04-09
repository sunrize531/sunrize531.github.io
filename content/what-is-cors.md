Title: Three ways to allow CORS
Summary: How to make crossdomain calls from browser without breaking inner peace.
Date: 2018-02-28 12:00
Category: Guides
Tags: cors, http, guide

# What is CORS

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is
a browser feature. It won't let you send requests to other server(s) if
the server(s) doesn't allow it. I.e. if the page opened
from `https://google.com` and it wants to send the request to
`https://microsoft.com` - there are gonna be some restrictions.

More information:

* [Same-origin policy (wikipedia)](https://en.wikipedia.org/wiki/Same-origin_policy)
* [Cross-origin resource sharing (wikipedia)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)

# Why?

CORS is very basic protection against phishing and some other nasty stuff. 
It won't be possible to just copy your html page,
change it a little bit, put it somewhere and somehow get your users on it. It won't just work,
cause browser won't send the requests on your server.

# How it works?

Before sending the request to different domain using anything except GET method,
i.e. POST, PUT, DELETE etc browser would send the OPTIONS request first, without empty body.
It called "preflight" call. If our server responds on this call with a certain set of headers -
browser would send the actual call.

![OPTIONS CORS request in Chrome debug panel]({filename}/images/what-is-cors-001.png)

# Is it really secure?

Nah. Implementing proxy which would forward requests to your service with or without preflights
is easy-peasy. If they bothered to setup phishing page they can surely setup this kind of thing as well.
Very little effort needed for this, so in most cases I'd recommend...

# Just allow everything

Ok, we write public service, and we actually *want* people use it.
Although we can't really allow everything anyway, we got to explicitely enumerate request methods and
headers and things like that. Here's middleware.

```javascript
const {Router} = require('express');
const router = require('express').Router();
router.use(function (req, res) {
    res.set('Access-Control-Allow-Origin', '*');
    res.set('Access-Control-Allow-Methods',
        'OPTIONS,GET,POST');
    res.set('Access-Control-Allow-Headers',
        'Origin, X-Requested-With, Content-Type, Accept');
    res.set('Access-Control-Max-Age', 300);

    if (req.method === 'OPTIONS') {
        res.status(200);
        res.send();
    } else {
        next();
    }
});

router.use(yourMiddlewaresHere);
```

We might want to allow more request methods (`PUT`, `DELETE`, `UPDATE`,...) and/or more headers.
Just add them to corresponding lists.

# Allow specific domains

We know (and probably control) all domains using our service and might as well deny everything coming from outside.
We're going to return request's origin in `Access-Control-Allow-Origin` header, exactly as it comes in. After validation.

Also we check allowed methods here, specifically dissalowing GET request. This would prevent showing images from this
domain, for example.

```javascript
const router = require('express').Router();

const AllowedOrigins = [
    'https://www.bugger.online',
    /^https:\/\/(\w+\.)?sunrize531\.github\.io$/i
];

function checkOrigin(origin) {
    for (let o of AllowedOrigins) {
        if (o instanceof RegExp && o.test(origin)) {
            return true;
        } else if (origin === o) {
            return true;
        }
    }
    return false;
}

router.use(function (req, res, next) {
    let origin = req.get('origin');
    if (!checkOrigin(origin)) {
        // TODO: log this
        res.status(401).send();
        return;
    }

    res.set('Access-Control-Allow-Origin', origin);
    res.set('Access-Control-Allow-Methods',
        'POST,OPTIONS');
    res.set('Access-Control-Allow-Headers',
        'Origin, X-Requested-With, Content-Type, Accept');
    res.set('Access-Control-Max-Age', 300);

    if (req.method === 'OPTIONS') {
        res.status(200);
        res.send();
    } else {
        next();
    }
});

```

# Make sure all (well almost) requests come with valid preflight

In theory, only browsers need to run preflight requests.
So if there were no preflight request - then it's likely not a browser.
Or not well behaving browser. So we might as well give them 401 straight away.
100% paranoya mode. It won't safe us from malicious proxy though.
They can fake everything, including preflight calls.

But it would surely make their life little harder, cause they would need to guess
that we're doing this additional check and implement additional OPTIONS call.

Not a biggie though, just bit annoying. Perhaps there's even ~~porn~~ npm module of it,
i.e. running requests with preflight.

```javascript
const cache = require('lru-cache')({maxAge: 300000});
const router = require('express').Router();
const crypto = require('crypto');

function requestFingerPrint(req) {
    // Very basic client's fingerprint.
    // Could send some random identifier from client if we really need to,
    // or use some other techniques like cookies or even
    // Canvas Fingerprinting.
    //
    // Although the entire thing isn't super secure anyway, so this will do.

    let hash = crypto.createHash('sha256');
    hash.update(origin);
    hash.update(req.ip);
    hash.update(req.originalUrl);
    hash.update(req.get('user-agent'));
    return hash.digest('utf8');
}

function setCorsHeaders(res, origin, methods) {
    res.set('Access-Control-Allow-Origin', origin);
    res.set('Access-Control-Allow-Methods', 'POST,OPTIONS');
    res.set('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
    // Note shorter age then in lru-cache entries.
    res.set('Access-Control-Max-Age', 200);
}

router.options(function (req, res) {
    let origin = req.get('origin');
    if (!checkOrigin(origin))) {
        res.status(401).send();
        return;
    }

    let fingerPrint = requestFingerPrint(req);
    cache.set(fingerPring, (new Date()).valueOf());

    if (req.method === 'OPTIONS') {
        cache.set(requestFingerPrint(req), (new Date()).valueOf());
        setCorsHeaders(res, origin);
        res.status(200);
        res.send();
    } else {
        let preflightTime = cache.get(fingerPrint);
        let now = (new Date()).valueOf();
        if (preflightTime && now - preflightTime < 300000) {
            setCorsHeaders(res, origin);
            next();
        } else {
            res.status(401).send();
        }
    }
});

```

# Conclusion

In general CORS provides very basic protection and can be bypassed easy enough.
Also CORS can be used for things like disallowing using images from your resources.
Worth adding to the webservice anyway. Real easy, not worth adding dependencies.