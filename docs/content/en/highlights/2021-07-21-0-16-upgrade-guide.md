---
date: "2021-07-28"
title: "0.16 Upgrade Guide"
description: "An upgrade guide that addresses breaking changes in 0.16.0"
authors: ["jszwedko", "JeanMertz"]
pr_numbers: []
release: "0.16.0"
hide_on_release_notes: false
badges:
  type: breaking change
---

Vector's 0.16.0 release includes one breaking change:

1. [Datadog Log sink encoding option removed](#encoding)

And one notable change:

1. [Vector source/sink version 2 released](#vector-source-sink)

We cover them below to help you upgrade quickly:

[##](##) Upgrade Guide

### Datadog Log sink encoding option removed {#encoding}

In previous versions of vector it was possible to configure the Datadog logs
sink to send in 'text' or 'json' encoding. While the logs ingest API does accept
text format the native format for that API is json. Sending text comes with
limitations and is only useful for backward compatability with older clients.

We no longer allow you to set the encoding of the payloads in the Datadog logs
sink. For instance, if your configuration looks like so:

```toml
[sinks.dd_logs_egress]
type = "datadog_logs"
inputs = ["datadog_agent"]
encoding.codec = "json"
```

You should remove `encoding.codec` entirely, leaving you with:

```toml
[sinks.dd_logs_egress]
type = "datadog_logs"
inputs = ["datadog_agent"]
```

Encoding fields other than `codec` are still valid.

### Vector source/sink version 2 released {#vector-source-sink}

We've released a new major version (`v2`) of our `vector` [source][]/[sink][]
components. This release resolves a lot of issues we saw with our previous
(`v1`) TCP-based implementation of these two components.

The new version transitions to using gRPC over HTTP as its communication
protocol, which resolves those issues.

To allow operators to transition at their leisure, this new release of Vector
still defaults to `v1`. In the next release (`0.17.0`) we'll require operators
to explicitly state which version they want to use, but continue to support
`v1`. The release after that (`0.18.0`) we'll drop `v1` completely, and default
to `v2`, we also no longer require you to explicitly set the version since there
will only be one supported going forward.

If you want to opt in to the new (stable!) `v2` version, you can do so as
follows:

```diff
[sinks.vector]
  type = "vector"
+ version = "v2"

[sources.vector]
  type = "vector"
+ version = "v2"
```

There are a couple of things to be aware of:

#### Upgrade both the source _and_ sink

You **have** to upgrade **both** the source and sink to `v2`, or none at all,
you cannot update one without updating the other. Doing so will result in a loss
of events.

#### Zero-downtime deployment

If you want to do a zero-downtime upgrade to `v2`, you'll have to introduce the
new source/sink versions next to the existing versions, before removing the
existing one.

First, deploy the configuration that defines the source:

```diff
  [sources.vector]
    address = "0.0.0.0:9000"
    type = "vector"
+   version = "v1"

+ [sources.vector]
+   address = "0.0.0.0:5000"
+   type = "vector"
+   version = "v2"
```

Then, deploy the sink configuration, switching it over to the new version:

```diff
  [sinks.vector]
-   address = "127.0.1.2:9000"
+   address = "127.0.1.2:5000"
    type = "vector"
+   version = "v2"
```

Once the sink is deployed, you can do another deploy of the source, removing the
old version:

```diff
- [sources.vector]
-   address = "0.0.0.0:9000"
-   type = "vector"
-   version = "v1"
-
  [sources.vector]
    address = "0.0.0.0:5000"
    type = "vector"
    version = "v2"
```

[source]: https://vector.dev/docs/reference/configuration/sources/vector/
[sink]: https://vector.dev/docs/reference/configuration/sinks/vector/
