---
title: New In Chrome 55
description: >
  With Chrome 55, you can write promise-based code as if it were synchronous,
  using async and await. PointerEvents provide a unified way of handling all
  input events. And <b>persistent storage</b> graduates from it's origin trial.
layout: 'layouts/blog-post.njk'
date: 2016-11-30
updated: 2018-04-16
authors:
  - petelepage
hero: 'image/0g2WvpbGRGdVs0aAPc6ObG7gkud2/mSTrfPSzkd9urYjiYguF.png'
alt: 'Cropped Chrome logo on the left, version number on the right.'
tags:
  - new-in-chrome
  - chrome-55
---

{% YouTube id="OC7tgJP1D4s" %}

[Watch on YouTube](https://www.youtube.com/watch?v=OC7tgJP1D4s)

* [`async` and `await`](#async-and-await) allows you to write promise-based
  code as if it were synchronous, but without blocking the main thread.
* [Pointer events](#pointer-events) provide a unified way of handling all
  input events.
* Sites added to the home screen, are automatically granted the [persistent
  storage](#persistent-storage) permission.

And there's plenty more.

I'm [Pete LePage](https://petelepage.com/), here's what's new for developers in Chrome 55!

## Pointer Events {: #pointer-events }

Pointing at things on the web used to be simple. You had a mouse, you moved
it around, sometimes you pushed buttons, and that was it. But this, doesn't
work so well on here.

Touch events are good, but to support
[touch](https://www.w3.org/TR/touch-events/) and
[mouse](https://developer.mozilla.org/docs/Web/API/MouseEvent), you had
to support two event models:

```js
elem.addEventListener('mousemove', mouseMoveEvent);
elem.addEventListener('touchmove', touchMoveEvent);
```

Chrome now enables unified input handling by dispatching
[PointerEvents](https://developer.mozilla.org/docs/Web/API/PointerEvent):

```js
elem.addEventListener('pointermove', pointerMoveEvent);
```

Pointer events unify the pointer input model for the browser, bringing
touch, pens, and mice together into a single set of events. They're supported
in [IE11, Edge, Chrome, Opera and partially supported in Firefox](https://goo.gl/znkJcj).

Check out  [Pointing the Way Forward](https://developers.google.com/web/updates/2016/10/pointer-events)
for more details.

## `async` and `await` {: #async-and-await }

Asynchronous JavaScript can be difficult to reason about.  Take this
function with all it's "lovely" callbacks:

```js
function logFetch(url) {
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function (e) {
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        console.log(xhr.responseText);
      } else {
        console.error('xhr failed', xhr.statusText);
      }
    }
  };
  xhr.onerror = function (e) {
    console.error(xhr.statusText);
  };
  xhr.open('GET', url);
  xhr.send();
}
```

Re-writing it with `promises` helps avoid the nesting problem:

```js
function logFetch(url) {
  return fetch(url)
    .then(response => response.text())
    .then(text => {
      console.log(text);
    }).catch(err => {
      console.error('fetch failed', err);
    });
}
```

But, Promise-based code can still be difficult to read when there are long
chains of asynchronous dependencies.

Chrome now supports the `async` and `await` JavaScript keywords, allowing you
to write Promise-based JavaScript that can be as structured and
readable as synchronous code.

Instead, our asynchronous function can be simplified to this:

```js
async function logFetch(url) {
  try {
    const response = await fetch(url);
    console.log(await response.text());
  }
  catch (err) {
    console.log('fetch failed', err);
  }
}
```

Jake has a great post:
[Async Functions - making promises friendly](https://developers.google.com/web/fundamentals/getting-started/primers/async-functions)
with all the details.

## Persistent Storage {: #persistent-storage }

The persistent storage origin trial is now over. You can now mark web
storage as persistent, preventing Chrome from automatically clearing the
storage for your site.

```js
if (navigator.storage && navigator.storage.persist) {
  navigator.storage.persist().then(granted => {
    if (granted)
      alert("Storage will not be cleared except by explicit user action");
    else
      alert("Storage may be cleared by the UA under storage pressure.");
  });
}
```

In addition, sites that have high engagement, have been added to the
home screen or have push notifications enabled are automatically
granted the persistence permission.

Check out Chris Wilson's [Persistent Storage](https://developers.google.com/web/updates/2016/06/persistent-storage)
post for full details and how you can request persistent storage for your site.

## CSS Automatic Hyphenation

[CSS automatic hyphenation](https://developers.google.com/web/updates/2016/10/css-hyphens), one of Chrome's
most frequently requested [layout features](https://googlechrome.github.io/samples/css-hyphens/)
is now supported on Android and Mac.

## Web Share API

And finally, it's now easier to invoke native sharing capabilities with the
new [Web Share API](https://github.com/mgiuca/web-share/blob/master/docs/interface.md),
which is available as an
[origin trial](https://github.com/GoogleChrome/OriginTrials/blob/gh-pages/developer-guide.md).
Paul (Mr. Web Intents) Kinlan has all the details in his
[Navigator Share](https://developers.google.com/web/updates/2016/10/navigator-share) post.

## Closing

These are just a few of the changes in Chrome 55 for developers.

If you want to stay up to date with Chrome and know what's coming, be sure to
[subscribe](https://goo.gl/6FP1a5), and be sure to check out the
[videos from the Chrome Dev Summit](https://www.youtube.com/playlist?list=PLNYkxOF6rcIBTs2KPy1E6tIYaWoFcG3uj)
for a deeper dive into some of the awesome things the Chrome team is working on.

I'm Pete LePage, and as soon as Chrome 56 is released, I'll be right here
to tell you what's new in Chrome!
