Title: WTF is CORS
Date: 2018-02-24 12:00
Category: Article

# What is CORS

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is
a browser feature. It won't let you send requests to other server(s) if
these server(s) doesn't implement CORS support. I.e. if the page opened
from ()[https://google.com] and it wants to send the request to
()[https://microsoft.com] - there are gonna be some restriction.
It won't `just work` in all cases.

# Why care

Even now days in most cases everything is hosted on the same domain.
Even if it's not hosted on the same server there are many ways to
setup everything so it can be accessed using the same domain name. CORS is
always on in browser, but it doesn't do anything whe everything is in
the same domain. But...

Here, short list of cases when you might need to know what is it:

1. Development. If you write client side web application, test it locally and
   want some data from remote server, dev, stage or even production version.
1. Your web application uses multiple web services and they can't be hosted on
   the same domain for one of many possible reasons.
1. You implement web service used by other web applications, web sites or even
   3rd parties.
1. You're phisher and trying to steal other people's personal data (seriously,
   go find better job and also fuck you).

```python
def test():
    t = {}
    t["key"] = 14
    return t
```
