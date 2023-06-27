---
title: "Logging & Reporting"
weight: 5
description: >
    Logs tell a very important story during the lifecycle of any software application, and a testing library is no different.
---

In fact, it could easily be argued that logging in a testing context is doubly important in order to capture all the details of what happened during the execution cycle. It's the step by step details that tell us what a test was doing when something goes wrong, which in turn helps us testers to more quickly understand and recreate these failure scenarios so that system defects can be reported earlier in the testing cycle.

Qadenz uses two primary loggers to tell the story of an execution run. The Suite Logger monitors major activity with the execution run itself and captures events outside of the individual tests. This includes setup and tear-down activities, configuration details, the starting and stopping of tests, and errors that are encountered during the reporting phase. The Test Logger captures the step by step details of each test that runs as part of a Suite. This includes every action, inspection, validation, and any errors that are encountered along the way.

Qadenz uses Logback to handle all logging within the library. These logs are presented on the console during a Suite execution, and are also used to generate content for the Qadenz HTML Reporter.

Each log event will contain a timestamp, the logging level, the name of the logger,

## Console Logging

The console logs will show output from both the Suite Logger and the Test Logger. These will be present when running locally either from Maven or within an IDE like IntelliJ IDEA, as well as in build systems like Jenkins or TeamCity. The console logs are the "raw" log output, showing all events in chronological order, regardless of origin. This means events reported inside test methods that are run in parallel will be shown in the same output. To facilitate better tracing of these events, the console logger is configured to add a thread identifier to each event.

```
>>> Add log output example
```

The Suite Logger output can be identified by `Thread: main`, and the Test Logger output for each test can be identified by a numbered idntifier, such as `Thread: 1`,  based on how many tests are executed in parallel.

The console logs are beneficial for local debugging of tests, providing a fast and easy view into the events and any errors encountered during the execution of a problematic test.

## The Qadenz HTML Reporter

The Qadenz HTML Reporter is the primary view of test results. This report was originally inspired by the TestNG `emailable-report.html`, but sought to make some key improvements to both the level of detail in the report, and general user experience.

The TestNG emailable report is sufficient for a count and listing of tests that passed and failed, and some basic insight on any exceptions that were caught during a test, but is lacking for any additional detail. There are other libraries that generate HTML reports, but can be cumbersome to use and may introduce additional code management overhead in order to integrate with tests and UI models.

Qadenz solves this problem by leveraging the Logback loggers to generate the detailed content for each test, and integrating directly with TestNG to generate and calculate results.

### Report Output

The Qadenz HTML report header shows the Suite Name as given on the Suite XML file, the date and time the execution run was launched, test counts (total, passed, failed, stopped, skipped), and an overall duration for the run.

Next, the classes and methods invoked by each `<test>` node on the Suite XML will be present in blocks. The name given in the `<test>` node will be present, followed by an expandable list of test classes. Failed tests will be listed first, followed by Stopped tests, then Skipped, followed by Passed.

Expanding a class entry on the list will expose a list of methods from the same class that were included in the run and that fall within the current result section. The same class may be listed in multiple sections depending on the final outcome of the included tests. If a `@DataProvider` has been used with the tests, the parameters from the Data Provider will be listed next to the name of the test method.

Next, expanding a test entry will expose the individual logs for the test. The report will show the start time and duration for each test method, followed by the detailed logging output from the test. In Failed and Stopped tests, each failure and error will be accompanied by a "View Screenshot" link. Clicking this link will open the screenshot image in a modal for viewing.

### Failed vs Stopped Tests

Default TestNG behavior is to classify any test that throws any exception as a failed test. Qadenz introduces a new result type called "Stopped" to allow some granularity in how test results can be sorted and analyzed.

Qadenz considers a "Failed" test to be a test that contains one or more failed validations, and considers a "Stopped" test to be a test that has thrown any other exception. This includes timing/sync issues, missing elements, etc.

By limiting the "Failed" category to validation failures, this allows teams to focus their post-execution results analysis on the tests that could potentially be more likely to produce a defect. This does not preclude Stopped tests from being so due to system defects, nor does it preclude Failed tests from being so due to issues in the tests. This is simply a means to draw attention specifically to tests that have failed due to validations that did not meed expectations, and those that have failed due to other reasons.

### Screenshots

The Qadenz HTML report is completely inclusive of all detailed reporting content and screenshots. The ease of distribution is the same as the TestNG emailable report.

Screenshots in the Qadenz library are captured and immediately encoded to a Base64 String, which is then added to the report. This allows the images to be embedded directly into the report and to be shared without having to work with cumbersome zip files or deal with broken image links.
