---
title: "Extensibility"
linkTitle: "Extensibility"
description: >
  Add project-specific commands by subclassing WebCommander or WebInspector, inheriting the full command anatomy.
weight: 5
---
No fixed set of commands can anticipate every UI. When a project needs an interaction the built-in commands do not cover, extend [`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/WebCommander.java) or [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/WebInspector.java) and add it, rather than forking Qadenz or reaching around it. A custom command written this way inherits the same [four-step anatomy]({{< relref "/docs/Components/Commands/_index.md" >}}) as every built-in command: the wait, the logging, and the screenshot-on-failure all come for free.

## Extend the class

Start by subclassing either `WebCommander` or `WebInspector`. Add a `Logger`, and carry over both constructors so the subclass preserves the [logging choice]({{< relref "/docs/Components/Commands/webcommander.md#creating-a-webcommander" >}}) for its consumers.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AcmeWebCommander extends WebCommander {

    private Logger LOG;

    public AcmeWebCommander() {
        super();
        LOG = LoggerFactory.getLogger(AcmeWebCommander.class);
    }

    public AcmeWebCommander(Class<?> logger) {
        super(logger);
        LOG = LoggerFactory.getLogger(logger);
    }
}
```

## Write a command

A command method follows the same anatomy as the built-in commands:

```java
public void doSomething(Locator locator) {
    LOG.info("Doing something with element [{}].", locator.getName());
    try {
        WebElement webElement = webFinder.findWhenVisible(locator);

        // add logic as needed to complete the custom command
    }
    catch (Exception exception) {
        LOG.error("Error doing something :: {}: {}", exception.getClass().getSimpleName(), exception.getMessage());
        screenshot.capture();

        throw exception;
    }
}
```

Build it in the same order every command follows:

1. **Log the action** at the `INFO` level. Qadenz uses Logback, so use the `{}` placeholder to fold element names and values into the message for readable reports.
2. **Initialize the element** inside the `try` block using the inherited `WebFinder`, then perform the interaction. On a `WebInspector` subclass, return the value from within the `try` block.
3. **Handle failure** in the `catch` block: log the exception at the `ERROR` level and capture a screenshot with the inherited `Screenshot`. Catch the specific exception where that helps, or cast a wide net by catching all descendants of `Exception`. A multi-catch or a `finally` fits here when the use case calls for it.
4. **Surface the exception** so it stops the test. Re-throw it as shown, or throw a new `RuntimeException` with a context-friendly message. Throwing an unchecked exception keeps the command free of a checked-exception signature, so tests that call it need no `try`/`catch` of their own. Qadenz is built to keep that burden off the test.

## Reaching the WebDriver directly

Most custom commands build on the inherited `WebFinder` and `Screenshot`, and on the existing commands. For the rare interaction that needs to go lower than the command layer, the raw `WebDriver` is always available through `WebDriverProvider.getWebDriver()`. Extending Qadenz adds to the tooling without walling off what sits underneath it.

## Use the custom command

Instantiate the subclass wherever commands are used, whether in the UI-modeling layer or directly in a test, and the new command is available alongside every inherited one.