---
title: "OpenTelemetry non compliant trace ids"
date: 2026-06-04T13:30:00+02:00
draft: false
tags: ["opentelemetry", "javascript", "honeycomb", "honeycomb refinery"]
---

I recently worked on a new ecommerce project which was using my favorite OpenTelemetry vendor Honeycomb.io, and the stack running in NodeJS/NextJS on a combination of Azure Containers Apps and Azure Functions. Unfortunately, when inspecting the state of the traces
I found the following issues:

- A mix of Sentry tracing/samplers and default OpenTelemetry setup
- Frontend and backend traces are not linked
- Traces are missing parent spans/traces
- None of the Azure Functions are instrumented

Luckily I've now done multiple projects where I fixed most of these issues, or know how to solve them (or so I thought).

A lot of issues can be fixed by at least being consistent on the OpenTelemetry settings. The first thing I did was make sure all
the OpenTelemetry dependencies are set to latest, including @vercel/otel for the NextJS app. I also updated Honeycomb Refinery
from 2.8 to 3.2. Unfortunately, I then figured out the backend used I also updated Honeycomb Refinery from 2.8 to 3.2. B3 propagation (which makes sense in GCP so you actually get traces there, but not in Azure). After some more debugging
I also figured out the backend uses http/json. @vercel/otel did not have an override, which defaults to http/protobuf (this will be important later on).

We still didn't get fully connected traces, and after more debugging I figured out Caddy is being used as a sidecar on both the NextJS and Node backend service.
Caddy does have tracing, but it can only do Grpc, and wasn't configured at all. After disabling the Caddy tracing I managed to get some frontend to backend
connected traces. I also noticed the traces had a sampling decision, but it came from Sentry. This project does use Sentry, but not for tracing. I had to remove all the samplers
in the OpenTelemetry SDK setup to make sure the sampling is left to Refinery.

For Azure Functions, I already blogged about how you need to use an alternative setting to set NODE_OPTIONS to properly load OpenTelemetry on boot in Azure Functions.
Adding an observability wrapper with forceFlush was enough to get this working:

```typescript
export const withObservability = <T extends FunctionHandler>(handler: T): T => {
 const wrappedHandler = async (
  triggerInput: Parameters<T>[0],
  context: Parameters<T>[1],
 ): Promise<Awaited<ReturnType<T>>> => {
  try {
   return await withSpan(
    context.functionName,
    { attributes: getSpanAttributes(triggerInput, context) },
    async (span) => {
     const result = await handler(triggerInput, context);

     if (result?.status) {
      span.setAttribute(ATTR_HTTP_RESPONSE_STATUS_CODE, result.status);
      if (result.status >= 500) {
       span.setStatus({ code: SpanStatusCode.ERROR });
      }
     }

     return result;
    },
   );
  } finally {
   await forceFlush();
  }
 };
```

Finally, I found some code that started a trace but did a span.end() before a call that throws an exception, which also
leads to missing parent spans if it throws that exception.

# Non-compliant trace ids

I did most of this work in parallel, and at some point traces became completely unconnected between frontend and backend. I added logging to log the trace ids and
unfortunately the traces I saw in the logs did not match what I saw in Honeycomb.

Logs:

- trace ID: `adb975f0528562d20350d5f81f13549d`
- span ID: normal 16 hex chars

After ingest, Honeycomb showed:

- trace ID: `69d6fdef97f4e76f39eb6776d37e747797fcd5fd77e78f5d`
- span ID length: 24 hex chars

This transform matches exactly:

`hex(base64_decode(original_trace_id))`

In the end it was correlated to when I did the Honeycomb Refinery upgrade from 2.8 to 3.2. It turns out, a regression was introduced in 3.x, but only for http/json payloads.
After reporting the bug it was fixed very quickly: <https://github.com/honeycombio/husky/pull/355> They also suggested for performance, you should prefer http/protobuf anyway over http/json.
