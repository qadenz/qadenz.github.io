---
title: Test Results
description: >
  Test Results
weight: 5
---

Logs tell a very important story during the lifecycle of any software application, and a testing library is no different. In fact, it could easily be argued that logging in a testing context is doubly important in order to capture all the details of what happened during the execution cycle. It's the step by step details that tell us what a test was doing when something goes wrong, which in turn helps us testers to more quickly understand and recreate these failure scenarios so that system defects can be reported earlier in the testing cycle.

Qadenz uses two primary loggers to tell the story of an execution run. The Suite Logger monitors major activity with the execution run itself and captures events outside of the individual tests. This includes setup and tear-down activities, configuration details, the starting and stopping of tests, and errors that are encountered during the reporting phase. The Test Logger captures the step by step details of each test that runs as part of a Suite. This includes every action, inspection, validation, and any errors that are encountered along the way.

Qadenz uses Logback to handle all logging within the library. These logs are presented on the console during a Suite execution, and are also used to generate content for the Qadenz HTML Reporter. Each log event will contain a timestamp, the logging level, the name of the logger, and the log message.

Logs are made available during and after an execution run in a variety of formats. The console logs will show the output of both the Suite and Test loggers in real time. The JSON report will show the compiled Test log output, combined with Base64 encoded screenshots, and sorted into Pass/Fail/Stop result sets. The HTML report will show the same data as the JSON report, but in an all-inclusive and sharable HTML file.
