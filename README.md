## About

This module integrates [Dropwizard Metrics library](http://metrics.dropwizard.io/) with Spring, and provides Java annotation based configuration.

## Working

* Creates metrics and proxies beans which contain methods annotated with `@Timed`, `@Metered`, `@ExceptionMetered`, and `@Counted`
* Registers a `Gauge` for beans which have members annotated with `@Gauge` and `@CachedGauge`
* Autowires Timers, Meters, Counters and Histograms into fields annotated with `@Metric`
* Registers with the `HealthCheckRegistry` any beans which extend the class `HealthCheck`
* Registers and Creates reporters, those reporters are used to report the monitored data.
* Registers Jvm metrics and metric sets.


## Usage

```java
import com.metrics.configuration.annotation.AbstractMetricConfiguration;
import com.metrics.configuration.annotation.MetricsSupport;
import org.springframework.context.annotation.Configuration;

@Configuration
@MetricsSupport
public class DropwizardMetricsConfiguration extends AbstractMetricConfiguration {

    @Override
    public void configureReporters() {
        registerReporter("es", "elastic-search", "/mnt1/config/elasticsearch.xml");
        registerReporter("console", "console", "");
    }
    
}
```

A @Configuration class annotated with @MetricsSupport is necessary to use all annotation bean post processors. 
* `registerReporter(reporter_name, reporter_type, reporter_properties_file_path)` - registers reporters


## Dropwizard Metrics

### Meters

A meter measures the rate of events over time (e.g., “requests per second”). In addition to the mean rate, meters also track 1-, 5-, and 15-minute moving averages.

```java
@Metered(name = "meter_name", reporterNames = {reporter_names})
public void handleRequest(Request request, Response response) {
    // etc
}
```

This meter will measure the rate of requests in requests per second.


### Gauges

A gauge is an instantaneous measurement of a value.

```java
public class QueueManager {
    private final Queue queue;
    
    public QueueManager(String name) {
        this.queue = new Queue();
        //etc; 
        
        @Gauge(name = "gauge_name", reporterNames = {reporter_names}
        int len = queue.length;
    }
    
}
```

When this gauge is measured, it will return the number of jobs in the queue.


### Counters

A counter is just a gauge for an AtomicLong instance. You can increment or decrement its value.


### Histograms

A histogram measures the statistical distribution of values in a stream of data. In addition to minimum, maximum, mean, etc., it also measures median, 75th, 90th, 95th, 98th, 99th, and 99.9th percentiles.


### Timers

A timer measures both the rate that a particular piece of code is called and the distribution of its duration.

```java
@Timed(name = "timer_name", reporterNames = {reporter_names})
public String handleRequest(Request request, Response response) {
    // etc;
    return "OK";
}
```

This timer will measure the amount of time it takes to process each request in nanoseconds and provide a rate of requests in requests per second.


### Health Checks

Metrics also has the ability to centralize your service’s health checks with the metrics-healthchecks module.

Implement a HealthCheck subclass:

```java
public class DatabaseHealthCheck extends HealthCheck {
    private final Database database;
    
    public DatabaseHealthCheck(Database database) {
        this.database = database;
    }
    
    @Override
    public HealthCheck.Result check() throws Exception {
        if (database.isConnected()) {
            return HealthCheck.Result.healthy();
        } else {
            return HealthCheck.Result.unhealthy("Cannot connect to " + database.getUrl());
        }
    }
}
```


## Reporters

### ElasticSearch Reporter

#### Configuration

Define elastic search reporter in configuration class.

```java
public void configureReporters() {
        registerReporter("es", "elastic-search", "/mnt1/config/elasticsearch.xml");
```

elasticsearch.xml will contain:

```<reporter period="1m" hosts="localhost:9200"/>```

##### Options

    hosts: A list of hosts used to connect to, must be in the format hostname:port, default is localhost:9200
    timeout: Milliseconds to wait for an established connections, before the next host in the list is tried. Defaults to 1000
    bulkSize: Defines how many metrics are sent per bulk requests, defaults to 2500
    index: The name of the index to write to, defaults to metrics
    indexDateFormat: The date format to make sure to rotate to a new index, defaults to yyyy-MM
    timestampFieldname: The field name of the timestamp, defaults to @timestamp, which makes it easy to use with kibana

#### JSON Format of metrics

This is how the serialized metrics looks like in elasticsearch

##### Counter

```
{
  "name": "counter",
  "@timestamp": "2018-07-040T09:29:58.000+0000",
  "count": 18
}
```

##### Timer

```
{
  "name" : "timer",
  "@timestamp" : "2018-07-04T09:43:58.000+0000",
  "count" : 114,
  "max" : 109.681,
  "mean" : 5.439666666666667,
  "min" : 2.457,
  "p50" : 4.3389999999999995,
  "p75" : 5.0169999999999995,
  "p95" : 8.37175,
  "p98" : 9.6832,
  "p99" : 94.68429999999942,
  "p999" : 109.681,
  "stddev" : 9.956913151098842,
  "m15_rate" : 0.10779994503690074,
  "m1_rate" : 0.07283351433589833,
  "m5_rate" : 0.10101298115113727,
  "mean_rate" : 0.08251056571678642,
  "duration_units" : "milliseconds",
  "rate_units" : "calls/second"
}
```

##### Meter

```
{
  "name" : "meter",
  "@timestamp" : "2018-07-04T09:29:58.000+0000",
  "count" : 224,
  "m1_rate" : 0.3236309568191993,
  "m5_rate" : 0.45207208204948995,
  "m15_rate" : 0.5014348927301423,
  "mean_rate" : 0.4135529888278531,
  "units" : "events/second"
}
```

##### Histogram

```
{
  "name" : "histogram",
  "@timestamp" : "2018-07-04T09:29:58.000+0000",
  "count" : 114,
  "max" : 109.681,
  "mean" : 5.439666666666667,
  "min" : 2.457,
  "p50" : 4.3389999999999995,
  "p75" : 5.0169999999999995,
  "p95" : 8.37175,
  "p98" : 9.6832,
  "p99" : 94.68429999999942,
  "p999" : 109.681,
  "stddev" : 9.956913151098842,}
}
```

##### Gauge

```
{
  "name" : "gauge",
  "@timestamp" : "2018-07-04T09:29:58.000+0000",
  "value" : 123
}
```
