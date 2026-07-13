---
title: "Test Results"
linkTitle: "Test Results"
description: >
  A test's real output is the evidence it leaves behind. Qadenz records the full step-by-step story of every test, captures a screenshot at each failure, and sorts the results the way a team actually triages them.
weight: 5
---
When a run comes back red, someone has to answer a question before anything else can happen: is this a defect in the application, or a problem with the test? The default TestNG report answers it with a count and a stack trace, which is enough to know that something failed and almost never enough to know what. So the person triaging re-runs the test, watches it by hand, and reconstructs the story the first run already knew but did not keep.

Qadenz treats that story as the product of the run. Every command, inspection, and validation a test performs is logged as it happens, a screenshot is captured at the moment anything fails, and the whole record is compiled into a report sorted the way a team actually reads results. The verdict is one bit at the top; the evidence underneath it is the part that lets someone act without re-running a thing.

![The Qadenz HTML report with its result sections collapsed, showing the summary header across the top and each class grouped under a color-coded result section.](/img/qadenz-report-main.png)

## One stream, two destinations

Qadenz uses [Logback](https://logback.qos.ch/) for all logging, and every log event carries the same four fields: a timestamp, the level, the name of the logger, and the message. What changes is where an event goes and how it is dressed for the audience there.

The **console** is the live view. It receives every event, in the order it happened, tagged with the thread that produced it, so a run in progress can be watched and a problematic test debugged as it executes. Because parallel tests interleave here, the thread tag is what lets a reader follow a single test through the noise.

The **report** is the considered view. It receives only the events that happened inside a test, grouped by test and sorted by outcome, and it is what a person reads after the run to decide what needs attention. Suite-level activity, the driver lifecycle, parameter validation, the reporting phase itself, stays on the console and never reaches the report, because none of it belongs to a particular test.

That split is deliberate, not incidental. Two loggers write only to the console: a **Suite** logger for the run's own activity (setup, teardown, configuration, the starting and stopping of tests) and a **Reporter** logger for errors raised while results are compiled. Everything a test does, every command and validation, along with any notes a test [adds for itself]({{< relref "/docs/Components/Commands/logging.md#adding-notes-to-the-report" >}}), flows to both places at once. The console is therefore the superset, everything that happened; the report is the test-scoped subset, everything that happened inside a test.

## Failed and Stopped are two different problems

Every test framework tells a team that a test did not pass. Few tell it why, at the level triage actually turns on, because they file every non-passing test under a single heading. TestNG does exactly this: a test that asserted the wrong total and a test that never reached its assertion because an element was missing both land in the same failed pile, indistinguishable until someone opens each one and reads it.

Those two outcomes ask for different responses. A validation that ran and disagreed with the application is a candidate defect, the thing a suite exists to surface. A test that threw before it could validate, from a timeout or a missing element, is far more often a problem with the test or the environment than with the application under test. Sorting the two together buries the first kind under the second.

Qadenz draws the line TestNG leaves out. It reaches into TestNG's own result handling, [forking the reporter class TestNG ships](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/reporter/testng/TestResult.java) to re-read every failure and split it by cause. A test that threw an `AssertionError` is **Failed**: a validation ran and did not meet its expectation. A test that threw anything else is **Stopped**: it halted before it could reach a verdict. That single distinction turns a red run from one undifferentiated pile into a triage queue, where the Failed count is what a team acts on first and the Stopped count is what it works through separately, without either one hiding the other.

The split orders the work; it does not decide the outcome. A Failed test can still turn out to be a flaw in the test, and a Stopped test can still be the first sign of a real defect. The point of separating them is not to label cause in advance but to put the results likelier to matter at the front of the queue, so a team spends its first attention where it tends to pay off. [HTML Reports]({{< relref "html-reports.md" >}}) shows how the two appear in the report.

## The evidence travels with the result

Screenshots are part of the same record. Each one is captured at the point of failure and embedded directly into the report as encoded image data, so the whole thing is a single file that opens anywhere, with no broken links and no archive to unpack. A report shares as easily as an email attachment and reads the same on any machine.

## What a team gets

- **Every test keeps its story.** Every test carries its full step-by-step log and a screenshot at each failure, so triage happens by reading the record, not by re-running the test to rebuild it.
- **The right view for each moment.** A live, thread-tagged console for watching or debugging a run in progress, and a sorted, self-contained report for reading results after it finishes.
- **A result type TestNG does not have.** Qadenz forks TestNG's reporting to split failures into Failed (a validation the application did not meet) and Stopped (a test halted before it could validate), so a red run reads as a triage queue instead of one undifferentiated pile.
- **Results that are not locked in one file.** Screenshots embed inline for easy sharing, and the same run also emits structured JSON and standard Logback output, so results can flow into a team's own dashboards, log tooling, and cross-run trend analysis.

## The pieces

- **[Console Logs]({{< relref "console-logs.md" >}})** are the raw, chronological, thread-tagged output shown during a run, ideal for local debugging.
- **[HTML Reports]({{< relref "html-reports.md" >}})** are the primary view of results: a single self-contained file with per-test logs, embedded screenshots, and results sorted by outcome.
- **[The JSON Report]({{< relref "json-report.md" >}})** is the same compiled result data as a structured file, meant for export into external presentation systems.

Both files are written automatically at the end of a run by the reporter that [`AutomatedWebTest`]({{< relref "/docs/Components/Configuration/automatedwebtest.md" >}}) registers, and both land in the TestNG output directory as `suite-results.html` and `suite-results.json`.