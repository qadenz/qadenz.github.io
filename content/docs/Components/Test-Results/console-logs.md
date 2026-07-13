---
title: "Console Logs"
description: >
  The raw, chronological, thread-tagged output shown while a run executes, ideal for local debugging.
weight: 1
---

The console shows the raw log output of a run as it happens, whether that run is launched from Maven, from an IDE such as IntelliJ IDEA, or from a build system such as Jenkins or TeamCity. Every event appears in the order it occurred, regardless of origin, which means the suite-level activity of the run and the step-by-step output of every test share a single stream. When tests run in parallel, their events interleave here as well.

To keep that stream readable, the console layout tags every event with the thread that produced it. Each line follows the same format:

```
HH:mm:ss:SSS | Thread: <id> | <LEVEL> | <Logger> | <message>
```

The five fields are the timestamp, the thread identifier, the log level, the name of the logger, and the message. A run of a small authentication suite executing two tests in parallel produces output like this:

```
14:22:07:918 | Thread: main | INFO | XmlParameterValidator | Using Selenium Grid at Host [127.0.0.1].
14:22:07:920 | Thread: main | INFO | XmlParameterValidator | Using Browser [chrome].
14:22:08:114 | Thread: 1 | INFO | AutomatedWebTest | Executing Method [signInSucceedsWithValidCredentials].
14:22:08:119 | Thread: 2 | INFO | AutomatedWebTest | Executing Method [signInRejectsInvalidPassword].
14:22:08:402 | Thread: 1 | INFO | LoginPage | Entering text [admin@qadenz.dev] into element [Username Field].
14:22:08:640 | Thread: 2 | INFO | LoginPage | Entering text [admin@qadenz.dev] into element [Username Field].
14:22:08:944 | Thread: 1 | INFO | LoginPage | Entering text [Test123$] into element [Password Field].
14:22:09:120 | Thread: 1 | INFO | LoginPage | Clicking element [Sign In Button].
14:22:10:233 | Thread: 1 | INFO | DashboardPage | Verifying Condition - Visibility of element [Welcome Banner] is TRUE.
14:22:10:641 | Thread: 1 | INFO | DashboardPage | Result - PASS
14:22:11:002 | Thread: 2 | INFO | LoginPage | Clicking element [Sign In Button].
14:22:12:118 | Thread: 2 | INFO | LoginPage | Verifying Condition - Visibility of element [Error Message] is TRUE.
14:22:12:530 | Thread: 2 | INFO | LoginPage | Result - PASS
```

The main thread of the run appears as `Thread: main` and carries the suite-level activity: reading and validating the Suite parameters, starting and stopping the driver, and the framing of each test. Each parallel test runs on its own worker thread and appears as a number, such as `Thread: 1`, so the two interleaved tests above can be read apart by following a single thread down the column. The logger name identifies where an event came from, and for command and validation output it reflects the [constructor a command was given]({{< relref "/docs/Components/Commands/logging.md" >}}), which is why the steps above are attributed to `LoginPage` and `DashboardPage` rather than to the command classes.

The console is the fastest way to debug a problematic test locally. It shows every event and every error in real time, in the order they happened, without waiting for the run to finish and the report to be written. For reading results after a run, and for anything worth sharing, reach for the [HTML Report]({{< relref "html-reports.md" >}}) instead: it is scoped to what happened inside each test and sorted by outcome, where the console is the unsorted superset of everything.

## Feeding log tooling

The console layout is a Logback layout, and the events behind it are ordinary SLF4J log events. Nothing about them is specific to a terminal. The same stream can be directed through Logback's appenders into the log viewers and analysis platforms a development team already runs, the systems it uses to watch application logs in the first place.

That puts test output next to application output in one place. A suite's results become one more signal in the tooling a team already trusts to read the health of a system, and the visibility a run provides reaches past the moment the run finishes.
