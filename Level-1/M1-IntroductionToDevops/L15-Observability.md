# Invest in application and infrastructure monitoring.( Prometheus, Loki, Grafana, CloudWatch, Nagios, Datadog, New Relic, Sentry )

In DevOps, observability is referred to the software tools and methodologies that help Dev and Ops teams to log, collect, correlate, and analyze massive amounts of performance data from a distributed application and glean real-time insights. This empowers teams to effectively monitor, revamp, and enhance the application to deliver a better customer experience.

## Why DevOps observability is the future and why your organization needs it

In the past, IT businesses have used Application Performance Monitoring (APM) to monitor and enhance application performance. It collects and analyzes telemetry data of the applications and systems and provides valuable insights for teams to address and prevent abnormal conditions. However, APM is the right fit only for monolithic applications or traditional distributed applications. It is because in those applications, the new code is released regularly and workflows and dependencies are well-known and easy to identify.

But in the present world, the highly distributed nature of the applications has made APM obsolete. Organizations today are leveraging DevOps practices, including agile development, continuous integration & continuous deployment (CI/CD), to deliver applications faster than ever. And, APM can't stay abreast. The DevOps ecosystem required high-quality telemetry data to create accurate, context-rich, fully-correlated information of every application. Therefore, organizations need DevOps observability to gain high visibility of their complex application ecosystem, understand any change (planned or unplanned), and stay ahead of the curve.

![](Images/monitoring.png)

## To do a good job with monitoring and observability, your should have the following:

- Reporting on the overall health of systems (Are my systems functioning? Do my systems have sufficient resources available?).
- Reporting on system state as experienced by customers (Do my customers know if my system is down and have a bad experience?).
- Monitoring for key business and systems metrics.
- Tooling to help you understand and debug your systems in production.
- Tooling to find information about things you did not previously know (that is, you can identify unknown unknowns).
- Access to tools and data that help trace, understand, and diagnose infrastructure problems in your production environment, including interactions between services.

## How to implement monitoring and observability

Monitoring and observability solutions are designed to do the following:

- Provide leading indicators of an outage or service degradation.
- Detect outages, service degradations, bugs, and unauthorized activity.
- Help debug outages, service degradations, bugs, and unauthorized activity.
- Identify long-term trends for capacity planning and business purposes.
- Expose unexpected side effects of changes or added functionality.

**Below we have mentioned few Logging and Monitoring tools, please go through the following:**

#### What is Grafana and Prometheus?

Prometheus is a monitoring solution for storing time series data like metrics. Grafana allows to visualize the data stored in Prometheus (and other sources). 

#### Is CloudWatch a monitoring tool?

Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and the applications you run on AWS in real time. You can use CloudWatch to collect and track metrics, which are variables you can measure for your resources and applications.

- [Installation and usage of cloudWatch agent](https://docs.google.com/document/d/1Ty2-g3fjQtML_mbf5IkXUXUyj13uXN34/edit)

#### What is Loki and Promtail?

Promtail is an agent which ships the contents of local logs to a private Grafana Loki instance or Grafana Cloud. It is usually deployed to every machine that has applications needed to be monitored. 

#### What is Nagios tool used for?

Nagios is an open source monitoring system for computer systems. It was designed to run on the Linux operating system and can monitor devices running Linux, Windows and Unix operating systems (OSes). Nagios software runs periodic checks on critical parameters of application, network and server resources.

#### What is Datadog tool used for?
Datadog is a monitoring and analytics tool for information technology (IT) and DevOps teams that can be used to determine performance metrics as well as event monitoring for infrastructure and cloud services. The software can monitor services such as servers, databases and tools.

#### What is New Relic used for ?

New Relic is a Software as a Service offering that focuses on performance and availability monitoring. It uses a standardized Apdex (application performance index) score to set and rate application performance across the environment in a unified manner.

#### What is Sentry ?

Sentry is a crash reporting platform that provides you with "real-time insight into production deployments with info to reproduce and fix crashes". It notifies you of exceptions or errors that your users run into while using your app, and organizes them for you on a web dashboard.

- It is good to know and have done hands on Grafana, Prometheus, Promtail agent, Loki 
- Must have experience with CloudWatch 
- Optionally as per project or client requirement we can work on Nagios, Datadog, New Relic, Sentry and other monitoring tools 