---
title: Hangfire Usage
summary: Using of Hangfire to send messages from within an NServiceBus endpoint.
reviewed: 2017-09-20
component: Core
related:
- nservicebus/messaging/timeout-manager
---

This sample illustrates the use of [Hangfire](https://www.hangfire.io/) to send messages from within an NServiceBus endpoint.

> Hangfire - An easy way to perform background processing in .NET and .NET Core applications. Hangfire is an open-source framework that helps you to create, process and manage your background jobs.


Note: The approach used in this sample can mitigate some of the architectural drawbacks of the [NServiceBus Scheduler](/nservicebus/scheduling/). The NServiceBus scheduler is built on top of the [Timeout Manager](/nservicebus/messaging/timeout-manager.md) which leverages the queuing system to trigger scheduled actions. Under heavy load, there may be some disparity between the expected time of a scheduled action and actual execution time due to the delay between timeout messages being generated and processed.


## Running the project

 1. Start both the Scheduler and Receiver projects.
 1. At startup Scheduler will schedule a message send to Receiver every minute.
 1. Receiver will handle the message.


## Code Walk-through

### Endpoint Helper

A helper to make the `IEndpointInstance` created by NServiceBus available inside Hangfire jobs. Implemented as a static property in this sample.

snippet: EndpointInstance

Hangfire also supports Dependency Injection (DI) via the [JobActivator API](http://docs.hangfire.io/en/latest/background-methods/using-ioc-containers.html) for more advanced scenarios.


### Configure and start the scheduler

The endpoint is started and the `IEndpointInstance` is stored in the static endpoint helper.

This sample uses in-memory storage for the jobs. Production scenarios should use more robust alternatives: SqlServer, Msmq or Redis.

Hangfire calls their scheduler a [BackgroundJobServer](http://docs.hangfire.io/en/latest/background-processing/processing-background-jobs.html). It is started automatically when an instance of the `BackgroundJobServer` class is instantiated.

snippet: Configuration


### Job definition

This sample passes in an expression pointing to the static `Run` method in `SendMessageJob`.

snippet: SendMessageJob

Note `EndpointHelper` is used by the job to get access to the `IEndpointInstance` .

### Schedule a job

Hangfire will accept any lambda expression as a job definition. 

The expression is serialized, stored and scheduled for execution by the `BackgroundJobServer` in Hangfire.

The schedule is set using [Cron](https://en.wikipedia.org/wiki/Cron) syntax through the helper 'Cron' class. This sample is scheduled to run every minute.

snippet: scheduleJob


### Cleanup

The Hangfire server should be disposed when the endpoint is shut down.

snippet: shutdown

For cleanup purpose either a static variable may need to be kept or the shutdown done as part of the container cleanup.


### Logging

Hangfire uses [LibLog](https://github.com/damianh/LibLog). Since LibLog support the detection and utilization of [Serilog](https://serilog.net/), this sample use the [NServiceBus Serilog integration](/nservicebus/logging/serilog.md)

snippet: serilog

LibLog [supports many other common logging libraries](https://github.com/damianh/LibLog/wiki#transparent-logging-support). Or Hangfire can be configured to use a custom logger. See [Adding logging in Hangfire](http://docs.hangfire.io/en/latest/configuration/configuring-logging.html).


## Scale Out

When using the approach in the sample, it is important to note that there is an instance of the Hangfire scheduler running in every endpoint instance. So if an endpoint is [scaled out](/transports/scale-out.md) the configured jobs will be executed in each of those running instances. A persistent [job storage](http://docs.hangfire.io/en/latest/configuration/index.html) can help manage the Hangfire scheduler shared state including jobs, triggers, calendars, etc.


## Further information on Hangfire

 * [Hangfire Quick Start Guide](http://docs.hangfire.io/en/latest/quick-start.html)
 * [Hangfire Tutorials](http://docs.hangfire.io/en/latest/tutorials/index.html)