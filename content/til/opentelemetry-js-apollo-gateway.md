---
title: "Stuck in OpenTelemetry hell"
date: 2025-08-01T13:30:00+02:00
draft: false
tags: ["opentelemetry", "javascript", "apollo"]
---

I've been stuck multiple times in OpenTelemetry hell where things didn't work or suddenly break. At some point
our HTTP client instrumentation traces (which set the incoming root span on our GraphQL gateway) randomly did or did not include specific manually instrumented span
attributes on different environments. So it would work locally and on test, but sometimes fail on acceptance / production.

We use these for dashboards, triggers and general querying so we had to find a fix. 

## Attempt 1 - Apollo Plugins

We use Apollo Gateway and made a custom observability plugin. I tried several options like refactoring
our observability plugin or change the order of the plugins. None of this worked, and the [source code](https://github.com/apollographql/apollo-server/blob/d7e9b97595b063f1e796ec4449850a16d19e8b18/packages/server/src/requestPipeline.ts#L604)
shows:

```javascript
  async function invokeWillSendResponse() {
    await Promise.all(
      requestListeners.map((l) =>
        l.willSendResponse?.(
          requestContext as GraphQLRequestContextWillSendResponse<TContext>,
        )
      )
    );
  }
```

That we can't affect the order of plugins.

## Attempt 2 - Try AsyncLocalStorage

I've attempted to use AsyncLocalStorage to store the root span before. Accessing it then allows for much higher cardinality
on the root span, which is great for debugging/easy for developers.

Unfortunately, it didn't seem to work.

## Attempt 3 - Upgrade OpenTelemetry

After running out of options, we noticed OpenTelemetry JS SDK v2 was out, so we tried upgrading.
This also didn't resolve the issue.

## Attempt 4 - Express middleware

We figured out we can set the root span on the request in Express. To make sure nothing interferes
it's the very first middleware we setup and looks like this:

```javascript
app.use((req, _res, next) => {
	const rootSpan = getActiveSpan()
	if (rootSpan) {
		req.rootSpan = rootSpan
	}

    rootSpan.setAttribute('foo', 'bar')
	next()
})
```

## Conclusion

While we fixed the core issue, I think doing something similar with AsyncLocalStorage would be more
ergonomic if we want to set spans on the root span.
