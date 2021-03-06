---
id: documentationOverview
title: Introduction
---

QuestDB is a relational column-oriented database which can handle transactions and real-time analytics.
It uses the SQL language with a few extensions for time-series. 
The below will help you get familiar with QuestDB and get started. 

### Concepts
This section describes the architecture of QuestDB, how it stores and queries data, and introduces
features and capabilities specific to QuestDB. 
 
As a start, we suggest you read about the [storage model](storageModel.md) and 
about the [designated timestamp](designatedTimestamp.md). To make the most of QuestDB, you should also get 
familiar with our [SQL extensions](sqlOverview.md) which allow to make the most of time-series capabilities with 
an efficient non-verbose syntax. You will also find the [symbol](symbol.md) concept interesting 
to store and retrieve repetitive strings efficiently.

### Guides
Setup guides are available for [Docker](guideDocker.md), the [binaries](guideBinaries.md) or [Homebrew](guideHomebrew.md).

There are guides to get started with the [Web console](consoleGuide.md), with the Postgres Wire Protocol (beta), for example with [PSQL](guidePSQL.md), 
the [HTTP REST API](guideREST.md) or the [Java API](embeddedJavaAPI.md)

### Reference
This section lists reference about configuration, interfaces, APIs, functions, and SQL statements. 

### Need help? Have questions?
We are happy to help with any question you may have, particularly to help 
you optimise the performance of your application. Feel free to reach out using the following channels: 
- [Ask a question on Slack](https://join.slack.com/t/questdb/shared_invite/enQtNzk4Nzg4Mjc2MTE2LTEzZThjMzliMjUzMTBmYzVjYWNmM2UyNWJmNDdkMDYyZmE0ZDliZTQxN2EzNzk5MDE3Zjc1ZmJiZmFiZTIwMGY>)
- [Raise an issue on Github](https://github.com/questdb/questdb/issues)
- or send us an email at [hello@questdb.io](mailto:hello@questdb.io)

