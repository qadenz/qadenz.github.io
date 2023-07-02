---
title: Console Logs
description: >
  Console Logs
weight: 1
---

The console logs will show output from both the Suite Logger and the Test Logger. These will be present when running locally either from Maven or within an IDE like IntelliJ IDEA, as well as in build systems like Jenkins or TeamCity. The console logs are the "raw" log output, showing all events in chronological order, regardless of origin. This means events reported inside test methods that are run in parallel will be shown in the same output. To facilitate better tracing of these events, the console logger is configured to add a thread identifier to each event.

```
>>> Add log output example
```

The Suite Logger output can be identified by `Thread: main`, and the Test Logger output for each test can be identified by a numbered idntifier, such as `Thread: 1`,  based on how many tests are executed in parallel.

The console logs are beneficial for local debugging of tests, providing a fast and easy view into the events and any errors encountered during the execution of a problematic test.
