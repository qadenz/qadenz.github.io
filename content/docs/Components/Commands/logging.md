---
title: "Logging"
description: >
  How the constructor choice attributes each logged step, and how a test adds its own notes to the report.
weight: 4
---

The two constructors on `WebCommander` and `WebInspector` are not just a way to instantiate the classes. They decide how every logged step is attributed on the report. The [choice between them]({{< relref "/docs/Components/Commands/webcommander.md#creating-a-webcommander" >}}) tracks a design decision: whether commands are called directly from the test, or from a UI-modeling layer such as a page object. This page shows what each choice produces.

Both classes share the same constructor pattern. The `WebCommander` is used here as the example:

```java
private Logger LOG;

public WebCommander() {
    super();
    LOG = LoggerFactory.getLogger(WebCommander.class);
}

public WebCommander(Class<?> logger) {
    super(logger);
    LOG = LoggerFactory.getLogger(logger);
}
```

The no-args constructor is generic and can be used regardless of how the classes are consumed. This constructor assigns the `WebCommander.class` as the `Logger`, and all logging output for method calls on this class will be shown to originate from the `WebCommander` class. If the UI of the application under test is very simple, or the team simply does not require a level of detail in the logging that ties actions to specific UI Models, then this constructor will provide an ideal configuration.

```
09:55:00.939 | INFO | WebCommander | Entering text [admin@qadenz.dev] into element [Username Field].
09:55:01.308 | INFO | WebCommander | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | WebCommander | Clicking element [Sign In Button].
09:55:02.583 | INFO | Commands | Verifying Condition - Visibility of element [Qadenz Logo Image] is TRUE.
09:55:02.998 | INFO | Commands | Result - PASS
```

The overloaded constructor requires a `Class<?>` argument, and allows for another class reference to be injected as the logger for the `WebCommander` instance. If `WebCommander` is being instantiated from a Page Object, and the Page Object class is passed to the constructor, the logs and reporting output will be shown to originate from the Page Object itself, resulting in a greater level of detail in the logs and reports. By using the class injection for the logger, commands will be logged in the context of the page where the command was executed.

```
09:55:00.939 | INFO | LoginPage | Entering text [admin@qadenz.dev] into element [Username Field].
09:55:01.308 | INFO | LoginPage | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | LoginPage | Clicking element [Sign In Button].
09:55:02.583 | INFO | UserProfilePage | Verifying Condition - Visibility of element [Company Logo Image] is TRUE.
09:55:02.998 | INFO | UserProfilePage | Result - PASS
```

## Adding notes to the report

Beyond the automatic per-command logging, a test can write its own entries into the report. These are useful for labeling sections of a test, recording system state, or leaving context that helps interpret a failure.

Two of these are instance methods inherited from the [`Commands`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/Commands.java) base, so they are available on any `WebCommander` or `WebInspector` and are attributed to the same logger the instance was constructed with:

- `log(String)` writes an entry at the `INFO` level, alongside the regular command output.
- `annotate(String)` writes an entry at the `WARN` level, which stands out on the report for higher-level notes such as labeling a phase of the test.

```java
commander.annotate("Verifying the checkout totals.");
commander.log("Cart contained " + itemCount + " items.");
```

For a note that belongs to the test itself rather than to a command or page, the static [`Log`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/Log.java) class provides `Log.annotate(String)`. It writes at the `WARN` level under a `Test` logger, so the entry reads as a test-level annotation regardless of which commands are in play.

```java
Log.annotate("Starting the returns workflow.");
```
