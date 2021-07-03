---
title: Enable tracing & metrics with opentelemetry javaagent automatically
layout: single
tags:
- open-telemetry
- tracing
- metrics
- prometheus
- jaeger
excerpt: A post about enabling traces and metrics in JAVA application using opentelemetry
  java instrumentation binary as javaagent, Prometheus to scrape the metrics, Jaeger
  to receive the traces.
category: development
---

A couple of years back, to debug an application logs were the only source of truth. But these days, logs alone are not enough. Sometimes we need to find why our application took a long time to complete a certain request. We may be asked to come up with the optimal min and max memory. Logs alone can't provide the answers for these. The solutions are tracing and metrics. Many of us don't know how to enable traces for our application nor how to collect and expose metrics. The thing is, it is a requirement for any service and is basically a duplication of work if done manually. To solve this opentelemetry helps us with auto instrumenting our code to enable tracing and also it exposes a default set of metrics for the application. In this post, I am going to use [opentelemetry-javaagent](https://github.com/open-telemetry/opentelemetry-java-instrumentation) in my spring-boot application to enable tracing and metrics without a single line of code change. 

Please refer to the below github repo to find the code discussed in this blog post.
[https://github.com/malathit/opentelemetry-example](https://github.com/malathit/opentelemetry-example)

### Overview
In this post I am going to use my spring boot application that has 2 APIs, one post method for creating an employee and another get request for listing al the employees using pagination.  The application uses H2 as database. To enable tracing and metrics, I need to run the opentelemetry instrumentation [binary](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent-all.jar) as javaagent with my spring-boot application. As of now the latest binary version 1.3.0 and I am going to use that. To receive the traces, I am going to run Jaeger as a docker container. To scrape and store the metrics, I will be running prometheus as a docker container as well.

#### Start the sample application
To start the application in the above Github url, run the below command
```sh
docker-compose up
```
If you notice, I am specifying the javaagent and the service name(needed for the traces) using the JAVA_TOOLS_OPTION env variable in the [dockerfile](https://github.com/malathit/opentelemetry-example/blob/main/Dockerfile#L13). Once started, you can reach the jaeger ui in url http://localhost:16686/ and prometheus server in url. 

#### Environment properties 
The javagent i.e the opentelemetry java instrumentation binary needs to be told where to send the traces and expose the metrics. This can be provided to the javaagent using environment variables. 

To send the traces to my jaeger instance, these are all the environment variables 
```sh
OTEL_TRACES_EXPORTER=jaeger
OTEL_EXPORTER_JAEGER_ENDPOINT=http://jaeger:14250
```

The first env variable mentions that we are going to use jaeger for sending the traces, so the javaagent can prepare the traces in a format specific to Jaeger. The second env variable tells where the jaeger collector service is running. In my case since I am using docker-compose the jaeger service is accessible with url http://jaeger
						
To expose the metrics in my application I use the below environment variables
```sh
OTEL_METRICS_EXPORTER=prometheus
OTEL_EXPORTER_PROMETHEUS_PORT=9464
```
The first varaible tells the javaagent to expose metrics in the format that can be scrapped by any prometheus server.		The second variable tells the port where the metrics are exposed. In my case, since I used port 9464, the metrics are available in the below url
http://localhost:9464

To see a complete list of environment variables allowed, please check [here](https://github.com/open-telemetry/opentelemetry-java/blob/main/sdk-extensions/autoconfigure/README.md#exporters).

#### Checking the metrics are recorded in prometheus server
Sample metrics available in the above url are as follows
```
# HELP runtime_jvm_memory_area Bytes of a given JVM memory area.
# TYPE runtime_jvm_memory_area gauge
runtime_jvm_memory_area{area="non_heap",type="committed",} 1.21208832E8
runtime_jvm_memory_area{area="heap",type="committed",} 3.93216E8
runtime_jvm_memory_area{area="heap",type="max",} 4.13138944E9
runtime_jvm_memory_area{area="non_heap",type="used",} 1.16545256E8
```

If you can see the prometheus config file, [prometheus.yml](https://github.com/malathit/opentelemetry-example/blob/main/prometheus.yml), I am adding our employee service metrics port as `9464` in the last 3 lines of the file.

Now go to http://localhost:9090 and see that our target(employee-service) is up. We should be able to query the metrics using Prometheus Query Language. We can extend this by connecting a grafana server and create custom dashboards for monitoring purposes.

### Checking the traces in the Jaeger server
Now call the api to create a new employee,
```sh
curl -X POST http://localhost:8111 -d '{"name": "abc"}' 
{"id": "1"}
```
Now go to the Jaeger ui on port 16686 and see that the service employee-api is available. Also under operations dropdown select the option named `/`. This operation is the request to api '/' which hosts the employee creation API.

Click on `Find Traces` and you should be able to see once trace. Click on that and you will be given further details of the trace like this,

![Tracing example](/images/opentelemetry/tracing-example.png)

Note that the trace has information about the database calls as well. In case of microservices with auto tracing enabled, you can also see those services in the same trace too

#### Other concepts
##### Sampling
Right now we have enabled all our traces to be sent to the Jaeger server, we can also configure only send a percentage of traces to the Jaeger server. To know more about sampling, check [here](https://github.com/open-telemetry/opentelemetry-java/blob/main/sdk-extensions/autoconfigure/README.md#sampler)

##### Adding trace id to logging
The trace id can also be used in logging. In case of microservices, the trace id is the same across services in a flow. So we can use the trace id in logs and use it for finding all the logs written by all the microservices involved in that flow. More details [here](https://github.com/open-telemetry/opentelemetry-java-instrumentation#logger-mdc-mapped-diagnostic-context-auto-instrumentation) on adding trace-id without any code changes.

#### Using spring-boot actuator for exposing the metrics
If you use spring boot actuator in code already, you may have the metrics exposed via actuator already, In that case, just tell the javagent to not expose any metrics using the environment variable,
```sh
OTEL_METRICS_EXPORTER=none
```
In the prometheus config, specify the port same as application port and specify the path where the metrics are available.


That's all for now folks!! Please reach out to me via comments if needed.
