---
title: Component Overview
tags: 
description: Looking at the nuts and bolts of Qadenz.
--- 

# Qadenz Components

This is an overview of the primary components that user will encounter while automating tests with Qadenz. These components will be referenced freqently throughout the documentation. This guide will not cover every component within the the Qadenz library, but rather will focus on components with which users will interact directly within consuming test projects.

# Configuration Components

These objects, from the `dev.qadenz.automation.config` package, handle Suite and Test configuration tasks.

## AutomatedWebTest

The [`AutomatedWebTest`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/config/AutomatedWebTest.java) is the base class for all test classes in a Qadenz-powered test project. This class is responsible for gathering parameter values from the TestNG Suite XML file, configuring and starting a `WebDriver` instance on a Selenium Grid for each test method, stopping the `WebDriver` after each test, and invoking the reporters after the Suite has completed.

Classes that hold `@Test` methods must extend this class in order for tests to run using Qadenz configurations.

```
public class AuthenticationTest extends AutomatedWebTest {
    
    @Test
    public void verifySuccessfulAuthenticationWithValidCredentials() {
        // add test code
    }

    @Test
    public void verifyAuthenticationErrorWhenUsingInvalidCredentials() {
        // add test code
    }
}
```

Classes that hold `@Test` methods must inherit from this class in order for tests to run using Qadenz configurations. If project-specific configurations need to be performed during the setup or tear-down phases of the execution cycle, an intermediate class may be inserted into the inheritance hierarchy. For example, a project may require that certain data items be in place as preconditions for tests, or other custom testing components would have to be started. These tasks would be well suited to be kept on a class that extends `AutomatedWebTest`, and is in turn extended by classes that hold `@Test` methods.

```
public class AcmeAutomatedWebTest extends AutomatedWebTest {
    
    @BeforeSuite
    public void loadItemData() {
        // perform setup tasks
    }
}

public class ItemInventorySearchTest extends AcmeAutomatedWebTest {
    // add @Test methods
}
```

AutomatedWebTest uses standard [TestNG Annotations](https://testng.org/doc/documentation-main.html#annotations) for configuration steps. As such, any intermediate configuration class that performs custom setup and tear-down tasks can utilize the same annotations to integrate seamlessly into the TestNG suite workflow.

## WebConfig

[`WebConfig`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/config/WebConfig.java) stores information that Qadenz uses at various points throughout the execution cycle. Primarily, these are the values given as parameters on the TestNG Suite XML file. While Qadenz calls these values during the setup and reporting phases they are made available on the chance these values are needed for any project-specific configuration.

Since these fields are all static, there really is no need to extend this class. Teams are encouraged to follow this pattern, though, and create a ProjectConfig class of their own should the need arise to track specific values or project-specific Suite parameters are in play.

## WebDriverProvider

The [`WebDriverProvider`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/config/WebDriverProvider.java) stores the `WebDriver` instance and makes it available to components where it is needed. Since the test classes work on an inheritance hierarchy, and with how TestNG manages threads, the `WebDriver` instance is required to be kept on a `ThreadLocal<WebDriver>` object. Rather than forcing testers to pass the `WebDriver` around to various library components, Qadenz has chosen to make the `ThreadLocal` static, and allow components that need the `WebDriver` to call a centralized location. On the (hopefully) rare occurrence where the `WebDriver` is needed directly, it can be invoked with the following call.

```
WebDriver webDriver = WebDriverProvider.getWebDriver();
```

# Commands Components

The classes from the `dev.qadenz.automation.commands` package are a collection of functionality that execute actions against elements on the UI, or the browser itself. In short, these are test steps ranging from actions such as clicks and inputs, inspections such as reading text or determining the state of an element, browser actions such as navigation or window management, and validations.

## Commands

The [`Commands`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Commands.java) class is abstract and sits atop the Commands Hierarchy (described in greater detail [here]({{ site.baseurl }}{% link _docs/extending-qadenz.md %})). This class provides common commands and is completely agnostic of any underlying automation frameworks. 

`Commands` is home to the validation functionality (explained [here]({{ site.baseurl }}{% link _docs/validations.md %})), as well as the `log()` and `annotate()` commands (explained [here]({{ site.baseurl }}{% link _docs/logging-and-reporting.md %})).

## The WebCommander

[`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) is a child of `Commands`, and wraps Selenium `WebElement` and `Actions` commands for use in tests and UI Models. Each of the methods on `WebCommander` provide a number of activities beyond simply performing Selenium actions. The workflow for `WebCommander` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Perform the `WebElement` or `Actions` command.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the UI under test.
6. Throw the exception to stop execution of the test.

All `WebCommander` commands include an Explicit Wait during `WebElement` initialization. Commands that involve a Click action include a wait for the clickability of the target element. All other commands include a wait for the visibility of the target element to be `true`.

## The WebInspector

The [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java) class works alongside the `WebCommander`. Instead of performing actions on UI elements, `WebInspector` work to extract and return data from UI elements. This includes retrieving text or attribute values from elements, and discovering selected, enabled, or visible state of elements. The workflow for `WebInspector` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Retrieve the required information from the `WebElement`.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the UI under test.
6. Throw the exception to stop execution of the test.

All `WebInspector` commands that involve a single target element also include an Explicit Wait during `WebElement` initialization for the visibility of the target element to be `true`. Commands that involve a `List<WebElement>` do not include an Explicit Wait.

### Constructors & Loggers

The `WebCommander` and `WebInspector` can be instantiated and used either from the tests directly, or from the UI Modeling layer, depending on the design of the test project. Both `WebCommander` and `WebInspector` have overloaded constructors that enable different types of logging to take place, and will both directly impact how the logs are presented on the report output.

The `WebCommander` constructor is used as an example, but the `WebInspector` shares this same pattern:

```
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
09:55:02.583 | INFO | UserProfilePage | Verifying Condition - Visibility of element [Company Logo Image] is TRUE.
09:55:02.998 | INFO | UserProfilePage | Result - PASS
```

## Browser Commands

The [`Browser`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Browser.java) class manages the browser under test. This includes activities within the browser, but outside the rendered DOM of the application. Actions such as navigation, alert handling, cookie management, and switching between browser windows, are all handled by the `Browser`. The methods on the `Browser` class are static, and are able to be called from anywhere within the scope of the test, be it from the UI Modeling layer, or directly from the test itself.

# Conditions & Expectations

The pairing of [`Conditions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Conditions.java) and [`Expectations`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Expectations.java) becomes the core evaluative logic of Qadenz. These classes are used to determine if the state of the UI under test meets a given criteria.

When used with the Explicit Wait provided by the `pause()` command, or with the Assertions provided by the `verify()` and `check()` commands, a Condition is used to describe precisely what on the UI is being evaluated, and the accompanying Expectation describes exactly the expected outcome of the evaluation.

The goal of the Condition/Expectation pairing is to provide a unified structure for making evaluations throughout a testing project, rather than mixing Selenium `ExpectedConditions` calls for Explicit Waits, and various syntax patterns for test assertions. The resulting code is simple to read and quickly understand, and is easily maintained as the application under test evolves.

## How does it work?

Each Condition accepts a variety of Expectations based on the type of evaluation being performed. For example, Conditions pertaining to element state (visibility, selected, enabled) accept boolean-based Expectations, while Conditions evaluating text will require String-based Expectations.

From a technical perspective, Conditions and Expectations is a Hamcrest wrapper. The output of a Condition is based on whether the outcome of the evaluation matched the expected result. The Condition will return `true` is there was a match, `false` otherwise.

## Logging evaluations

Implementations of the [`Condition`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Condition.java) interface will provide a description of the Condition under evaluation to the logs and subsequent report output. Should the evaluation fail to match with the Expectation, the Condition will also provide a summary of the actual results.

Implementations of the [`Expectation`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Expectation.java) interface will provide a description of the Expectation that will be added to the Condition logs, as well as the wrapped Hamcrest Matcher that will be used to compute the result of the Condition.
