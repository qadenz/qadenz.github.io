---
title: "HTML Reports"
description: >
  The primary view of results: a single self-contained file with per-test logs, embedded screenshots, and results sorted by outcome.
weight: 2
---

The Qadenz HTML report is the primary view of test results, and it exists because the reports a Selenium team usually reaches for each force a compromise.

TestNG ships an emailable report, and its one real virtue is that it is a single self-contained file. Past that it is thin: a count of what passed and failed, and a little exception text on the ones that did not. It shows nothing of what a test actually did, and pairing it with screenshots means writing images to disk and shipping them beside the report in a zip file, which is the friction a self-contained file was meant to remove.

ExtentReports answers the detail problem from the other side and introduces a new one. Its reporting runs through an object model a team drives from its own code, so calls to the reporter end up threaded through tests and page objects, tying automation logic to the reporting tool it happens to use. Its more advanced capabilities sit behind a paid license as well.

The Qadenz report picks up where the TestNG report leaves off: a clear view of results, carrying the full log of every test and the screenshots that go with its failures, in one file with nothing to unpack. It reaches that without asking a team to write to it. The content comes from the [Logback output]({{< relref "console-logs.md" >}}) the commands already produce and from TestNG's own result data, so no reporting calls live in the tests or the page objects. A small amount of CSS and JavaScript carries the rest, giving the report collapsible result sections and click-to-open screenshots without pulling in a front-end framework or a single external asset.

The report is generated at the end of a run as `suite-results.html`, written to the TestNG output directory alongside the [JSON report]({{< relref "json-report.md" >}}).

The report opens collapsed: a summary across the top, and every result grouped by outcome below it.

![The Qadenz HTML report with its result sections collapsed, showing the summary header across the top and each class grouped under a color-coded result section.](/img/qadenz-report-main.png)

From there, expanding a section, then a class, then a method walks down to the full detail of a single test.

## The summary

The report opens with a summary header. It shows the Suite name as given on the Suite XML file, followed by:

- **Launched**, the date and time the run started.
- **Total Tests**, the count of all tests in the run.
- **Passed**, **Failed**, **Stopped**, and **Skipped**, the counts for each result type.
- **Execution Time**, the total duration of the run.

## Result sections

Below the summary, the results for each `<test>` node on the Suite XML appear in their own block, headed by the name given to that `<test>`. Within a block, results are grouped into sections and rendered in a fixed order, so the entries that most need attention come first:

1. Failed Configurations
2. Skipped Configurations
3. Failed Tests
4. Stopped Tests
5. Skipped Tests
6. Passed Tests

A section appears only when it has entries, and its header carries the count, such as `3 Failed Tests`. Configuration sections cover the setup and teardown methods (`@BeforeMethod`, `@AfterClass`, and the like); the rest cover the test methods themselves.

Each section lists the test classes that fall within it. The same class can appear in more than one section when its methods ended with different outcomes. Expanding a class exposes the methods from that class that belong to the section. When a test is driven by a `@DataProvider` or a factory, the parameters for the run are shown next to the method name, as `methodName | [parameters]`, so each parameterized run is identifiable on its own.

Expanding a method exposes its detail: the start time, the duration, and then the full log output for that method, the same step-by-step record seen on the console but scoped to this one test. A few log events are styled to stand out while scanning:

- Lines written with [`annotate`]({{< relref "/docs/Components/Commands/logging.md#adding-notes-to-the-report" >}}) (the `WARN` level) are shown in bold, which is what makes them useful for labeling phases of a test.
- A passing validation (`Result - PASS`) is shown in green.
- A failing validation (`Result - FAIL`) is shown in red, and carries a **View Screenshot** link.

![An expanded view of the report, with classes and methods opened to show each test's start time, duration, and full log output. Passing checks are green, failures are red, annotations are bold, and each failure carries a View Screenshot link.](/img/qadenz-report-expanded.png)

## Failed vs Stopped tests

By default, TestNG classifies any test that throws any exception as a failed test. Qadenz splits that single bucket in two, adding a result type called **Stopped**, to give a team more to work with when sorting and analyzing results.

Qadenz counts a **Failed** test as one that threw an `AssertionError`, meaning a validation ran and did not meet its expectation. It counts a **Stopped** test as one that threw anything else: a timing or sync issue, a missing element, or any other exception raised before a validation could reach its verdict.

Limiting the Failed category to validation failures lets a team point its post-run analysis at the results most likely to represent a real defect. The split is a triage aid, not a verdict on cause. A Stopped test can still be the first sign of a system defect, and a Failed test can still turn out to be a problem in the test rather than the application. What the distinction does is separate the two kinds of failure so the more telling one is not buried in the noise of the other.

## Screenshots

The report is completely inclusive of its detail and its screenshots, which is what keeps it as easy to distribute as the TestNG emailable report.

A screenshot is captured at the point of failure. When a command hits an error, or a `verify` or `check` validation fails, Qadenz takes a screenshot of the browser in that state and attaches it to the matching log event, where it surfaces as a **View Screenshot** link that opens the image in a modal. Passing steps do not capture screenshots by default, so the images in a report mark the moments that went wrong.

Each screenshot is encoded to a Base64 string the instant it is captured and embedded directly into the report as image data. There are no external image files, so the report never carries broken links and never needs to travel as a zip archive. One HTML file holds everything.

Screenshot capture can be tuned where needed. A validation takes an overload that disables its screenshot, `verify(Screenshot.SKIP, ...)` or `check(Screenshot.SKIP, ...)`, for cases where the image adds nothing. A test can also capture one on demand through the `captureScreenshot()` command, to record browser state at a point that is not a failure.
