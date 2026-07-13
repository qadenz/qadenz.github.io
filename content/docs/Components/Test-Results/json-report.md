---
title: "JSON Report"
linkTitle: "JSON Report"
description: >
  The same compiled result data as a structured file, meant for export into a team's own dashboards or presentation systems.
weight: 3
---

Every run writes a `suite-results.json` file to the TestNG output directory, next to the [HTML report]({{< relref "html-reports.md" >}}). The two are not independent products. Qadenz compiles the full results of a run into a single JSON model first, then renders the HTML from it, so the JSON holds everything the report shows, in a form built to be read by a program rather than a person.

That is its purpose. A team that keeps results in its own dashboard, feeds a quality metrics system, or presents outcomes somewhere other than the built-in report has a structured feed to consume, instead of a report to scrape. The HTML report is one way to present this data; the JSON is the data itself, left open for any other.

## Trending across runs

A single report answers what happened this run. The more valuable question on a mature suite is what happened across runs: the run where a test first went red, whether a failure is new or long-standing, which way a suite's health is moving over time. One report cannot answer that on its own, because it holds one run.

The JSON is what puts the answer in reach. Because every run emits its full results as structured data, those files can be collected and read together. A Qadenz report viewer is in development that will do exactly this, collating individual runs into trend lines and per-test history so a tester can find the run where a test started failing and trace it from there. Until it ships, the same JSON is already open to any tool a team uses for the purpose.

## Shape

The model nests the way results do. A report holds tests, a test holds its result groups, each group holds classes, a class holds methods, and a method holds the log events that make up its story.

```json
{
  "suiteName": "Regression",
  "suiteStartDate": "2026-07-13 14:22",
  "suiteExecutionTime": "00h 04m 18.42s",
  "browser": null,
  "browserVersion": null,
  "platform": null,
  "appUrl": null,
  "tests": [
    {
      "testName": "Authentication",
      "totalFailedTests": 1,
      "failedTests": [
        {
          "className": "com.example.tests.AuthenticationTest",
          "methods": [
            {
              "methodName": "signInSucceedsWithValidCredentials",
              "parameters": null,
              "testStartTime": "14:22:08:114",
              "testExecutionTime": "00m 03.91s",
              "logEvents": [
                {
                  "logMessage": "14:22:08:402 | INFO | LoginPage | Clicking element [Sign In Button].",
                  "screenshot": null
                },
                {
                  "logMessage": "14:22:10:641 | INFO | LoginPage | Result - FAIL :: Found [false].",
                  "screenshot": "iVBORw0KGgoAAAANSUhEUg..."
                }
              ]
            }
          ]
        }
      ],
      "totalPassedTests": 0,
      "passedTests": []
    }
  ]
}
```

The top level carries the run's header data: the Suite name, its start time, and its total duration. Each entry in `tests` corresponds to a `<test>` node on the Suite XML and holds a group for every result type, `failedConfigurations`, `skippedConfigurations`, `failedTests`, `stoppedTests`, `skippedTests`, and `passedTests`, each paired with a `total` count. The groups match the sections of the HTML report and follow the same [Failed versus Stopped]({{< relref "html-reports.md#failed-vs-stopped-tests" >}}) distinction.

A few fields are worth calling out:

- **`parameters`** carries the `@DataProvider` or factory arguments for a parameterized run, and is `null` otherwise.
- **`logEvents`** is the full log record for the method. Each event pairs its `logMessage` with a `screenshot`, a Base64-encoded PNG captured at a point of failure, or `null` when no image was taken for that event.
- **`stackTrace`** appears on a method only when it stopped on an exception other than an assertion failure, giving the short stack trace for the throwable.
- **`browser`, `browserVersion`, `platform`, and `appUrl`** are reserved on the header for future use and are not yet populated.

The JSON is written with pretty-printing, so the file is readable as-is when opening it to check a result by hand.
