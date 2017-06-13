Fluentd Integration    ![Build Status](https://travis-ci.org/kamon-io/kamon-fluentd.svg?branch=master)
==========================

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/kamon-io/Kamon?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.kamon/kamon-fluentd_2.11/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.kamon/kamon-fluentd_2.11)

Reporting Metrics to Fluentd
===========================
[Fluentd] is distributed, reliable, realtime and programmable data collector. Fluentd's 300+ plugins connect it to [many data sources](http://www.fluentd.org/datasources) and [data outputs](http://www.fluentd.org/dataoutputs) while keeping its core small and fast (30-40MB memory footprint). And, [2,000+ data-driven companies](http://www.fluentd.org/testimonials) rely on Fluentd to differentiate their products with better use of data.

`kamon-fluentd` module provides capabilities to send kamon metrics to fluentd server. The module provides kamon users huge flexibility on interacting with kamon metrics because user can use bunch of [fluentd-plugin ecosystem](http://www.fluentd.org/plugins) for reforming, filtering and forwarding kamon-metrics to anywhere.

Installation
------------

Add the `kamon-fluentd` dependency to your project and ensure that it is in your classpath at runtime, that's it.
Kamon's module loader will detect that the Fluentd module is in the classpath and automatically start it.

### Getting Started

Kamon kamon-fluentd module is currently available for Scala 2.11 and 2.12.

Supported releases and dependencies are shown below.

| kamon-fluentd  | status | jdk  | scala            |
|:------:|:------:|:----:|------------------|
|  0.6.7 | stable | 1.8+ |  2.11, 2.12  |

To get started with SBT, simply add the following to your `build.sbt`
file:

```scala
libraryDependencies += "io.kamon" %% "kamon-fluentd " % "0.6.7"
```

Configuration
-------------

By default, this module assumes that you have an instance of the Fluentd daemon running in localhost and listening on port 24224. If that is not the case the you can use the `kamon.fluentd.hostname` and `kamon.fluentd.port` configuration keys to point the module at your Fluentd daemon.

The Fluentd module subscribes itself to the entities included in the `kamon.fluentd.subscriptions` key. By default, the following subscriptions are included:

```typesafeconfig
kamon.fluentd {
  subscriptions {
    histogram       = [ "**" ]
    min-max-counter = [ "**" ]
    gauge           = [ "**" ]
    counter         = [ "**" ]
    trace           = [ "**" ]
    trace-segment   = [ "**" ]
    akka-actor      = [ "**" ]
    akka-dispatcher = [ "**" ]
    akka-router     = [ "**" ]
    system-metric   = [ "**" ]
    http-server     = [ "**" ]
  }
}
```

If you are interested in reporting additional entities to Fluentd please ensure that you include the categories and name patterns accordingly.

### Customizing Statistical Metrics over Histogram Snapshots ###
You can also configure statistical metrics over histogram snapshots via `kamon.fluentd.histogram-stats` property.

```typesafeconfig
kamon.fluentd {
  # statistic values to be reported for histogram type metrics
  # (i.e. Histogram, MinMaxCounter, Gauge).
  histogram-stats {
    # stats values:
    # "count", "min", "max", "average", "percentiles" are supported.
    # you can use "*" for wildcards.
    subscription = [ "count", "min", "max", "average", "percentiles" ],

    # percentile points:
    # this will be used when you set "percentiles" in "subscription" above.
    # In this example, kamon-fluentd reports 50th 90th, 99th and 99.9th percentiles.
    percentiles = [50.0, 90.0, 99.0, 99.9]
  }
}
```
### Json Formats passed to Fluentd ###

Fluentd's messages are represented by JSON format.  Sample Json Format generetad by the module is below:

```
{
  "time": 1445776940
  "tag": "kamon.fluentd.kamon-fluentd-example.akka-actor.kamon/user/metrics.mailbox-size",
  "app.name":"kamon-fluentd-example",
  "category.name":"akka-actor",
  "metric.name":"mailbox-size",
  "entity.name":"kamon/user/metrics",
  "stats.name":"average",
  "value": 2.0,
  "unit_of_measurement.name":"unknown",
  "unit_of_measurement.label":"unknown",
  "tags.some_tag_key":"some_tag_value",
  "canonical_metric.name":"kamon-fluentd-example.akka-actor.kamon/user/metrics.mailbox-size.average",
  "kamon-fluentd-example.akka-actor.kamon/user/metrics.mailbox-size.average": 2.0
}
```

Fluentd's JSON must have `time` and `tag` properties. `tag` is the property which is used for identifying data streams in Fluentd. `tag` which are generated by the module consists of tag prefix and canonical metric name. You can configure tag prefix via `kamon.fluentd.tag` property.

Naming convention of canonical metric name is as follows:

* For all single instrument entities (those tracking counters, histograms, gaugues and min-max-counters), tag name will be `<app.name>.<instrument.type>.<entity.name>.<stats.name>`. Instrument type will be `counter`, `histogram` and etc.. stats name will be `count`, `min`, `max`, `average`, `percentiles.50`, etc. depending on instrument type.

* For all other entities the pattern is a little different. the pattern will be `<app.name>.<category.name>.<entity.name>.<metric.name>.<stats_name>`.  For example, mailbox-size measurements of `actor:kamon/user/metrics` are reported under the `<app.name>.akka-actor.kamon/user/metrics.mailbox-size.<stats_name>` metrics.

You can configure `<app_name>` via `kamon.fluend.application-name`.

For all tags which are set in Kamon entities, please be noted that `tag` property in JSON message above is different from this, they will be embedded into Json messages as the form of `"tags.<tag_key>" : <tag_value>`.

"canonical_metric.name" and `<canonical_metric.name> : <value>` will be embedded too for user's convenience.

Integration Notes
-----------------

We don't flush the metrics data as soon as the measurements are taken but instead, all metrics data is buffered by the `kamon-fluentd` module and flushed periodically using the configured `kamon.fluentd.flush-interval`.

Examples
--------
Please look at [kamon-fluentd-example](https://github.com/kamon-io/Kamon/tree/master/kamon-examples/kamon-fluentd-example) for further details.

[Fluentd]: http://www.fluentd.org/
