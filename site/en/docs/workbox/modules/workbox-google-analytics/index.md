---
layout: 'layouts/doc-post.njk'
title: workbox-google-analytics
date: 2017-11-27
updated: 2021-04-23
description: >
  Support for replaying offline Google Analytics interactions.
---

If you're building an application that works offline, then understanding
how users are interacting with your app when they don't have connectivity
is crucial to optimizing that experience.

Analytics providers like
[Google Analytics](https://www.google.com/analytics) require a network
connection to send data to their servers, which means if connectivity is
unavailable, those requests will fail and those interactions will be
missing from your analytics reports. It'll be like they never happened.

Workbox Google Analytics solves this problem for Google Analytics users by
leveraging Service Worker's ability to detect failed requests.

Google Analytics receives all data via HTTP requests to the
[Measurement Protocol](https://developers.google.com/analytics/devguides/collection/protocol/v1/),
which means a Service Worker script can add a fetch handler to detect
failed requests sent to the Measurement Protocol. It can store these
requests in IndexedDB and then retry them later once connectivity is
restored.

Workbox Google Analytics does exactly this. It also adds fetch
handlers to cache the
[analytics.js](https://developers.google.com/analytics/devguides/collection/analyticsjs/) and
[gtag.js](https://developers.google.com/analytics/devguides/collection/gtagjs/)
scripts, so they can also be run offline. Lastly, when failed requests are
retried, Workbox Google Analytics also automatically sets (or updates) the
[`qt`](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters#qt)
in the request payload to ensure timestamps in Google Analytics reflect the
time of the original user interaction.

## Enabling Workbox Google Analytics

To enable Workbox Google Analytics, call the `initialize()` method:

```js
import * as googleAnalytics from 'workbox-google-analytics';

googleAnalytics.initialize();
```

This is the only code that's required to queue and retry failed requests to
Google Analytics, and it's the simplest way to get Google Analytics working
offline.

However, if using only the code above, the retried requests are
indistinguishable from requests that succeed on the first try. This means
you'll receive all the interaction data from offline users, but you won't
be able to tell which interactions occurred while the user was offline.

To address this concern, you can use one of the configuration options
described below to modify or annotate the data that gets sent in the
retried request.

### Modifying what data gets sent

If you want to be able to differentiate retried requests from non-retried
requests, you can specify either the `parameterOverrides` or `hitFilter`
[configuration options](/docs/workbox/reference/workbox-google-analytics/#method-initialize).

These options let you modify the
[Measurement Protocol parameters](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters)
that get sent in the retried request. The `parameterOverrides` option
should be used when you want to set the same value for a particular
parameter for every retried request. The `hitFilter` option should be used
in cases where the value of a particular parameter needs to be computed at
runtime or derived from the value of another parameter.

The examples below show how you'd use both options.

## Examples

### Using a custom dimension to track online vs. offline interactions

Google Analytics does not have a built-in dimension for online vs. offline
interactions. However, you can create your own dimension for exactly this
purpose using a feature called
[custom dimensions](https://support.google.com/analytics/answer/2709828).

To track requests that were replayed by the service worker using a custom
dimension with Workbox Google Analytics, follow these steps:

1. [Create a new custom dimension](https://support.google.com/analytics/answer/2709829)
   in Google Analytics. Give it a name like "Network Status" and set its
   [scope to "hit"](https://support.google.com/analytics/answer/2709828#example-hit)
   (since any interaction can be offline).
2. Take note of the index assigned for the newly created dimension and pass
   that as the parameter name to the `parameterOverrides` configuration option
   in your Workbox Google Analytics code.  

   For example, if this is your first custom dimension, its index would be `1`,
   and the parameter name would be `cd1` (if the index were `8` it would be
   `cd8`):

    ```js
    import * as googleAnalytics from 'workbox-google-analytics';

    googleAnalytics.initialize({
      parameterOverrides: {
        cd1: 'offline',
      },
    });
    ```

3. _(Optional)_ Since values in `parameterOverrides` are only applied
   to retried ("offline") requests, you may also want to set a default value
   of "online" for all other requests. While this isn't strictly necessary,
   it'll make your reports easier to read.<br><br>
   For example, if you used the default analytics.js tracking snippet to install
   Google Analytics, you could add the line `ga('set', 'dimension1', 'online')`
   to use a default value of `'online'` for your "Network Status" custom dimension
   for all requests not replayed by the service worker.

    ```html
    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

      ga('create', 'UA-XXXXX-Y', 'auto');

      // Set default value of custom dimension 1 to 'online'
      ga('set', 'dimension1', 'online');

      ga('send', 'pageview');
    </script>
    ```

{% Aside %}
`workbox-google-analytics` uses Measurement Protocol
[parameter names](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters),
which are different from the
[field names](https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#dimension)
used by analytics.js. For example, with custom dimensions the Measurement
Protocol uses
[`cdXX`](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters#cd_)
(e.g. `cd1`) whereas analytics.js uses
[`dimensionXX`](https://developers.google.com/analytics/devguides/collection/analyticsjs/field-reference#dimension) (e.g. `dimension1`).
{% endAside %}

### Using a custom metric to track time requests spent in the queue

If you were curious to know how much time had passed between when an offline
interaction took place and when connectivity was restored and the request was
successfully retried, you could track this using a
[custom metric](https://support.google.com/analytics/answer/2709828) and
the `hitFilter` configuration option:

1. [Create a new custom metric](https://support.google.com/analytics/answer/2709829)
   in Google Analytics. Give it a name like "Offline Queue Time", set its
   [scope to "hit"](https://support.google.com/analytics/answer/2709828#example-hit),
   and set its formatting type to "Time" (in seconds).
1. Use the `hitFilter` option to get the value of the
   [`qt`](https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters#qt)
   param and divide it by 1000 (to convert it to seconds). Then set that value
   as a param with the index of the newly created metric. If this is your
   first custom metric, the parameter name would be `'cm1'`:  

    ```js
    import * as googleAnalytics from 'workbox-google-analytics';

    googleAnalytics.initialize({
      hitFilter: (params) => {
        const queueTimeInSeconds = Math.round(params.get('qt') / 1000);
        params.set('cm1', queueTimeInSeconds);
      },
    });
    ```

## Testing Workbox Google Analytics

As Workbox Google Analytics uses Background Sync to replay events, it can
be unintuitive to test. Read more at
[Testing Workbox Background Sync](/docs/workbox/modules/workbox-background-sync#testing_workbox_background_sync).
