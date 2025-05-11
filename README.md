# [xssgame.com](http://xssgame.com/)

## Level 1

The input is embedded as-is into the page that'll load next.
high
```
<script>alert()</script>
```

## Level 2

The input becomes the argument of `onload="startTimer(...);"` like this: `onload="startTime(' + input + ');"`.

```
3');alert('
```

This works because `3');` correctly terminates the original function call, `alert('` starts a new call which ends up with "ending" of the original function call.

## Level 3

Entering the following URL into the search bar works.

```
http://www.xssgame.com/f/u0hrDTsXmyVJ/#'onerror='alert()'
```

Sometimes it told me for this and similar inputs that this is wrong, something about absolute URLs, etc., but it accepted it.
Try switching from single to double apostrophes, use `onload` instead and supply input for a working `src` attribute, or try to close of the `img` tag.
