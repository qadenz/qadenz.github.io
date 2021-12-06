---
title: Commands
tags: 
description: Element commands as the building block of test steps.
--- 

# Commands

The Commands classes are a collection of functionality that execute actions against elements on the UI, or the browser itself. In short, these are test steps ranging from actions such as clicks and inputs, inspections such as reading text or determining the state of an element, browser actions such as navigation or window management, and validations.

# Browser Commands

The [`Browser`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Browser.java) class manages the browser under test. This includes activities within the browser, but outside the rendered DOM of the application. Actions such as navigation, alert handling, cookie management, and switching between browser windows, are all handled by the `Browser`. The methods on the `Browser` class are static, and are able to be called from anywhere within the scope of the test, be it from the UI Modeling layer, or directly from the test itself.

# The Commands class

The [`Commands`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Commands.java) class is abstract and sits atop the Commands Hierarchy (described in greater detail [here]({{ site.baseurl }}{% link _docs/extending-qadenz.md %})). This class provides common commands and is completely agnostic of any underlying automation frameworks. 

`Commands` is home to the validation functionality (explained [here]({{ site.baseurl }}{% link _docs/validations.md %})), as well as the `log()` and `annotate()` commands (explained [here]({{ site.baseurl }}{% link _docs/logging-and-reporting.md %})).

## The WebCommander

[`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) is a child of `Commands`, and wraps Selenium `WebElement` and `Actions` commands for use in tests and UI Models. Each of the methods on `WebCommander` provide a number of acitivities beyond simply performing Selenium actions. The workflow for `WebCommander` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Perform the `WebElement` or `Actions` command.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the application under test.
6. Re-throw the exception to stop execution of the test.

## The WebInspector

The [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java) class works alongside the `WebCommander`. Instead of performing actions on UI elements, `WebInspector` work to extract and return data from UI elements. This includes retriving text or attribute values from elements, and discovering selected, enabled, or visible state of elements. The workflow for `WebInspector` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Retrieve the required information from the `WebElement`.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the application under test.
6. Re-throw the exception to stop execution of the test.

## Using WebCommander and WebInspector

The `WebCommander` and `WebInspector` can be instantiated and used from either the tests directly, or from the UI Modeling layer, depending on the design of the test project.

### Constructors & Loggers

Both `WebCommander` and `WebInspector` have overloaded constructors that enable different types of logging to take place, and will both directly impact how the logs are presented on the report output.

The `WebCommander` constructor is used as an example, but the `WebInspector` shares this same pattern:

```Java

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

```TXT
09:55:00.939 | INFO | WebCommander | Entering text [admin@qadenz.io] into element [Username Field].
09:55:01.308 | INFO | WebCommander | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | WebCommander | Clicking element [Sign In Button].
09:55:02.583 | INFO | Commands | Verifying Condition - Visibility of element [Qadenz Logo Image] is TRUE.
09:55:02.998 | INFO | Commands | Result - PASS
```

The overloaded constructor requires a `Class<?>` argument, and allows for another class reference to be injected as the logger for the `WebCommander` instance. If `WebCommander` is being instantiated from a Page Object, and the Page Object class is passed to the constructor, the logs and reporting output will be shown to originate from the Page Object itself, resulting in a greater level of detail in the logs and reports. By using the class injection for the logger, commands will be logged in the context of the page where the command was executed.

```
09:55:00.939 | INFO | LoginPage | Entering text [admin@qadenz.io] into element [Username Field].
09:55:01.308 | INFO | LoginPage | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | LoginPage | Clicking element [Sign In Button].
09:55:02.583 | INFO | DashboardPage | Verifying Condition - Visibility of element [Company Logo Image] is TRUE.
09:55:02.998 | INFO | DashboardPage | Result - PASS
```