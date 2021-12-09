---
layout: page
title: Qadenz Documentation
description: Working with the Qadenz Automation Library
permalink: /docs/
---

## **Philosophy of Use**

The role in a test automation project of an intermediate API that wraps frameworks such as Selenium and TestNG is far from uncommon. Many teams realize the importance of an API layer for these underlying tools as a means of simply abstracting common code and building re-usable components for frequently utilized features of the base frameworks. Some teams will take this a step further and design a wrapper layer that works to unify the development approach and adopted design patterns within the test project, thus answering the question "What is the best way for **our team** to implement these tools?".

It's certainly not uncommon for teams to implement such a solution. We as testers frequently find ourselves in fast-paced and ever-evolving environments. Designing, building, and importantly, maintaining an API such as this can be as significant an effort as keeping up with the demands for expanding test coverage. 

At the end of the day, it becomes about development efficiency and code maintainability. How quickly can we automate this next block of tests? Is the tooling reliable enough to ensure repeatable executions? Are the implementation details tribal knowledge, or is the code clear enough that a new developer can easily and quickly ramp up for maintenance? If the answers to these questions leads to increased efficiency and improved maintainability, then the investment in these tools becomes quite worthwhile.

Qadenz is built to satisfy the need for such tool-set, and allow testers to focus on the more important task... build tests quickly.

## Objectives

To this end, Qadenz targets a number of core objectives...

- **Drive easy to read tests with a clearly worded API:** Simply, readable code is manageable code. Everything from the API syntax to the logging and reporting output has been carefully chosen to maximize readability and to be a simple to understand as possible.
- **Reduce coding overhead by doing the heavy lifting:** Selenium interactions with Qadenz can, with one method call, log the activity, initialize an element, perform an interaction against the element, handle exceptions with logging and reporting, and capture screenshots. By offloading this work to the library, users can focus on scripting their tests rather than sorting out all the supporting functionality. 
- **Encourage efficient UI Modeling:** Qadenz uses CSS Selectors exclusively due to maximized flexibility and easy of use. This is further enhanced by injecting SizzleJS to enable parameterized element selectors. CSS Selectors are versatile, simple to understand, and most importantly (well crafted selectors are) more resistant to change compared to XPath. Wrapping the element selectors on the Locator object also enhances reporting with friendly element names, and encourages teams to move beyond antiquated and inefficient patterns. Page Factory is old and busted. Simon even says so.
- **Provide flexible and precise validations:** The Conditions-based validations functionality is custom to Qadenz and enables high levels of precision using the What/How relationship of Conditions and Expectations. Additionally, the Verify/Check language in the API allows for hard and soft assertions, which enables the user to decide exactly when a test will be halted following a failure. Finally, the iterative nature of Qadenz validations allows for multiple validations to be made in groups, providing a more complete picture of test failures and maximized potential for quickly exposing system issues. 
- **Include detailed and easily shared reports:** The `emailable-report.html` provided by TestNG is nice for basic pass/fail results, but leaves much to be desired when it comes to detail and user experience. Other reporting libraries add a lot of noise to the code, and don't tend to coexist with standard logging frameworks. The Qadenz reporter is powered by Logback, is fully integrated with TestNG, and produces a singular HTML output file that contains full detailed results of each test and embedded screenshots of errors and failed validations.

## Opinionation

Qadenz can be considered an opinionated library. Some features and solutions have been designed specifically to guide users into a particular pattern of usage based on certain best practices, or simply to speed up the development process for testers by choosing a particular path over another. While each is explained in greater detail elsewhere in the documentation, these points are the major areas of opinionation within the Qadenz library:

- **The WebDriver is scoped to the test method:** By limiting the scope of the `WebDriver` session, Qadenz ensures that a fresh browser is used for each test case. This also has provided an avenue to more precisely define the execution workflow.
- **Explicit Waits over Implicit Waits:** Explicit Waits allow for precise conditions to be met within a given amount of time. Implicit waits are only time-based, and cause problems when used alongside Explicit Waits.
- **Sizzle CSS Selectors only:** CSS Selectors are fast, extremely versatile, and resistant to DOM change. Adding Sizzle increases their versatility and adds a number of pseudo-classes that improve the ability to parameterize element selectors. This approach is a unified strategy that ensures all contributors to a test project are following the same pattern, which further enhances the ease of code maintenance.
- **Conditions-powered validations and Explicit Waits:** The `Conditions` and `Expectations` consist of precise and clearly defined evaluations. These provide a unified alternative to both sorting through the variety of assertion types or `ExpectedConditions` for Explicit Waits. Additionally, these classes ensure consistent and detailed information are passed to the logs and report output.
- **Locator-based UI Modeling over Page Factory & @FindBy:** `PageFactory` is an overly complex form of instantiating page objects. `@FindBy` does not support parameterization and mitigating issues like the `StaleElementReferenceException` requires additional code in order to tune and make reliable for a given web application. Locators support parameterization, provide a context-friendly element name for the logs, and a reference to a UI element that is later used by the `WebCommander` or `WebInspector` to initialize an element at the appropriate time of use.

## **Execution Management**

The `AutomatedWebTest` is the base class for all test classes in a Qadenz-powered test project. This class is responsible for gathering parameter values from the TestNG Suite XML file, configuring and starting a `WebDriver` instance on a Selenium Grid for each test method, stopping the `WebDriver` after each test, and invoking the reporters after the Suite has completed.

While classes the hold `@Test` methods must inherit from this class, an intermediate class may be inserted into the inheritance hierarchy to allow for project-specific configurations to be performed during the setup or tear-down phases of the execution cycle. For example, a project may require that certain data items be in place as preconditions for tests, or other custom testing components would have to be started. These tasks would be well suited to be kept on a class that extends `AutomatedWebTest`, and is then extended by test classes.

## The Execution Cycle

**Before the Suite Begins**

The first task performed after launching a Suite execution is to capture a timestamp and save as the `suiteStartDate` value on the `WebConfig`. This will be used later by the reporter to calculate the duration of the execution. Next, the Suite-level parameters are retrieved from the `ITestContext`, which were specified by the user on the TestNG Suite XML file. The  `gridHost` is validated and saved.

**Before Each Test**

Repeated before each test method, the Test-level parameters are retrieved from the `ITestContext`, which are the `browser`, `browserVersion`, `browserConfigProfile`, `platform`, `timeout`, `appUrl`, `retryInterceptedClicks`. These parameters are validated and saved to the `WebConfig`.

Once all the parameters are processed, the `WebDriver` can be launched and execution of a test method can begin. The `CapabilityProvider` is invoked to configure the browser session. The `CapabilityProvider` calls values on the `WebConfig` and attempts to load any Browser Config Profiles declared in one of the browser configuration JSON files. This process chooses a browser, determines if a specific version of the browser is required if a version has been declared, ensures that the test will be run on the appropriate OS if one has been specified, and applies any arguments provided on the JSON file if a matching Browser Config Profile was found.

These options are then used to configure and initialize a `WebDriver` instance, which is in turn set on the `WebDriverProvider`.

Finally, the browser window is maximized, and the `appUrl` value is loaded.

**After Each Test**

At the end of each test method, the `WebDriver` is stopped.

**After the Suite Ends**

The final task of the execution cycle is to capture a second timestamp as the `suiteEndDate` on the `WebConfig`. 

**Reporting**

The default TestNG reporters are not disabled by default, so the standard HTML & XML reports will be generated, along with the `emailable-report.html`. Next, the `TestReporter` is invoked which generates the Qadenz Reports in JSON and HTML formats.

## WebConfig

`WebConfig` stores information that Qadenz uses at various points throughout the execution cycle. Primarily, these are the values given as parameters on the TestNG Suite XML file. While Qadenz calls these values during the setup and reporting phases they are made available on the chance these values are needed for any project-specific configuration.

Since these fields are all static, there really is no need to extend this class. Teams are encouraged to follow this pattern, though, and create a ProjectConfig class of their own should the need arise to track specific values or project-specific Suite parameters are in play.

## WebDriverProvider

The `WebDriverProvider` stores the `WebDriver` instance and makes it available to components where it is needed. Since the test classes work on an inheritance hierarchy, and with how TestNG manages threads, the `WebDriver` instance is required to be kept on a `ThreadLocal<WebDriver>` object. Rather than forcing testers to pass the `WebDriver` around to various library components, Qadenz has chosen to make the `ThreadLocal` static, and allow components that need the `WebDriver` to call a centralized location. On the (hopefully) rare occurrence where the `WebDriver` is needed directly, it can be invoked with the following call.

```
WebDriver webDriver = WebDriverProvider.getWebDriver();
```

## **UI Modeling**

Qadenz is a departure from the commonly practiced use of the Selenium Page Factory pattern when implementing the Page Object Model. The `@FindBy` annotation-based approach to mapping UI elements lacks the ability to parameterize selectors. Instead, testers are forced to create repetitive annotated `WebElement` instances for every needed permutation of a selector. Without adding additional calls to `WebDriverWait` in order to use an Explicit Wait to initialize the element (and dealing with the resulting added clutter), these selectors are notoriously susceptible to the dreaded `StaleElementReferenceException`.

In short, Qadenz is quite happy to follow Simon Stewart's [advice](https://www.youtube.com/watch?v=gyfUpOysIF8&t=1519s), and find a better way.

The UI Modeling strategy behind Qadenz centers around the [`Locator`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/Locator.java) object, and rather than mix between all the different selector types, Qadenz has chosen CSS selectors exclusively.

## Why only CSS Selectors?

Many teams will choose either a default standard selector strategy (ID, CSS, XPath, etc.), or at least prioritize one over others. Qadenz has chosen CSS selectors for the superior performance, ease of use, and better browser compatibility when compared to XPath. The addition of the [SizzleJS](https://github.com/jquery/sizzle) library provides a number of pseudo-classes that make element selection more flexible and accurate.

The end result is a powerful and straightforward selector strategy that supports full parameterization capabilities. This also leads to a singular defined approach that an entire team can adopt and consistently apply across the entire project.

## The Locator

The [`Locator`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/Locator.java) is the primary component of UI Modeling in the Qadenz environment. This object simply carries the name of an element, the element's CSS Selectors, and an optional parent Locator instance to assist with abstracting element relationships and re-using selector content. The `Locator` is passed to methods on classes such as the [`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) and [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java), which will then use the data within to initialize WebElements with the [`WebFinder`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/WebFinder.java).

The `Locator` provides several major advantages over the `@FindBy` and `PageFactory`. 

### Context-friendly Element Names

Selenium does not readily provide a meaningful friendly reference to element names, which can complicate debugging and troubleshooting when problems arise. Without additional logging in the test project, testers are generally only provided references to elements expressed as the given selectors. This has a very strong potential to slow down the resolution process as testers must first translate the selector in their stack trace to an actual element on the UI, which can then establish a point of orientation within the progression of test steps.

By requiring a `name` value to be given in the `Locator` constructor, Qadenz refers to the context-friendly name of an element as the primary identifier in all logging and reporting output. This eliminates the time needed to perform any lookup or cross-referencing of selectors to elements in relation to test steps. By reviewing the default logging output or report content, the point at which a problem appears in a test is clearly marked and quickly identified.

```
Locator signInButton = new Locator("Sign In Button", ".btn-signIn");
```

### Parameterized Selectors

Using list of search results as an example, testers would be forced to create a separate `@FindBy` annotated `WebElement` instance for each result element needed for a test. Alternately, a `@FindBy` could be used to initialize a single `List<WebElement>`, but additional logic would be necessary to identify a specific result element needed for a test step. In either case, the result is many extra lines of code. 

Using a parameterized Locator (coupled with the benefits of Sizzle CSS Selectors), testers will be able to define a single `Locator` instance for a generic search result, and rely upon the parameterization to direct the test to choose the appropriate element instance.

```
public Locator searchResultLink(String name) {
    return new Locator(name + " Search Result Link", ".search-result:contains(" + name + ")");
}
```

### On-the-fly Element Initialization

Using `PageFactory.initElements()` will instantiate the page class, and all `WebElement` instances annotated with `@FindBy` will be initialized. If the page on the UI is static and all mapped elements are already present, there should be no issues. Modern web-apps, though, use very dynamic interfaces. This frequently results in situations where some `@FindBy` mapped elements will change state at some point between the time `.initElements()` is called and when the element is actually used. To mitigate these issues, additional code in the form of Explicit Waits or other custom logic must be written to handle each element. 

With Qadenz, Locators are defined and passed to commands methods, which in turn initialize a `WebElement` prior to acting upon the element. Further, each command will initialize the target element each time the command is called. By initializing WebElements *at the time they are used*, this mitigates situations that might produce a `StaleElementReferenceException`, and when errors are encountered, the problem can be presented far more clearly.

## Using Locators

### Standard Locators

A `Locator` holds two required values, `name` and `selector`. The `name` field is completely user-definable, but is recommended to at least convey some level of meaningful context to the element being mapped. This value will appear in the logs, on the final report output, as well as within any error handling. 

### Parent Locators

The `Locator` can hold a third, optional value of another `Locator` instance. This is intended to allow abstraction of selector segments by combining the selector of the parent Locator with that of the current Locator as a single selector value. This can be helpful in reducing repeated selector segments when creating Locators for closely related UI Elements. 

Consider an e-commerce application wherein a list of catalog items are presented on the UI. Each item card contains the item name text, a 'Cost' value, a 'Quantity' field, and an 'Add to Cart' button. 

As a very simple HTML representation:

```
<div id="item-list-section" class="grid">
    <div class="item-row even">
        <div class="item-card">
            <div class="item-name">ACME Rocket Powered Roller Skates</div>
            <div class="item-cost">$99.99</div>
            <div class="item-qty input-field">
                <input type="text"></div>
            <div class="item-add action-button">
                <button type="submit">Add to Cart</button></div>
        </div>
    </div>
<div>
```

The selector for the 'Add to Cart' button could be:

```
#item-list-section .item-card:contains(ACME Rocket Powered Roller Skates) .item-add button
```

While mapping the other elements on an item card, however, it would be discovered that the item card selector itself is repeated on each of the child elements. In this situation, a parent `Locator` could be created to abstract the repeated selector segments, especially if the abstracted selector can stand as it's own element mapping.

With a parent `Locator` the resulting element mappings for the item card, 'Quantity' field, and 'Add to Cart' button could be:

```
public Locator itemCard(String itemName) {
    return new Locator(itemName + " Item Card", "#item-list-section .item-card:contains(" + itemName + ")");
}
public Locator itemQuantityField(String itemName) {
    return new Locator(itemName + " Quantity Field", itemCard(itemName), " .item-qty input");
}
public Locator itemAddToCartButton(String itemName) {
    return new Locator(itemName + " Add to Cart Button", itemCard(itemName), " .item-add button");
}
```

In the above example, the `itemCard()` Locator could have the added benefit of not only being an abstracted selector, but could also serve as the target element of a verification of a given item card element to be visible on the page.

Parent Locators are also not limited to a single layer. Locators can be passed as parent references as many times as needed.

## The LocatorGroup

The [`LocatorGroup`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/LocatorGroup.java) allows multiple `Locator` instances to be combined together on a `List`, and acted upon as a group. This is commonly applied to verify the visibility of UI component, or a set of default UI elements. Instead of passing multiple individual Conditions to a `.verify()` or `.check()` validation, a `LocatorGroup` can be verified with a single `Condition` call.

For example, a simple authentication form has several basic elements, the 'Username' field, the 'Password' field, a 'Remember Me' checkbox, and a 'Sign In' button. Each element would be mapped as an individual `Locator` instance for the purposes of input, but these same elements could also be included in a `LocatorGroup`, should the need arise to verify each element to be visible as part of the form.

```
Locator usernameField = new Locator("Username Field", "#username");
Locator passwordField = new Locator("Password Field", "#password");
Locator rememberMeCheckbox = new Locator("Remember Me Checkbox", "#remember-me");
Locator signInButton = new Locator("Sign In Button", "#sign-in");

LocatorGroup signInForm = new LocatorGroup("Sign In Form", usernameField, passwordField, 
                                           rememberMeCheckbox, signInButton);
```

By combining individual Locators onto a `LocatorGroup` instance, testers are able to identify and refer to collections of elements as the UI Components they represent.

## Page Objects

With Qadenz placing focus on the `Locator` component, this leaves the design of any Page Object implementation is left as open ended as possible. This is done to allow any given team to design and implement an UI Model layer that fits their needs and the modeling needs of their UI under test.

Qadenz does not force a particular means of initializing Page Objects like the `PageFactory`. In fact, unless a more sophisticated implementation is needed by a team, Locator-based UI Models can be initialized by simply calling a constructor.

### UI Models as Locator Maps

A team could choose simply to create Page classes that only declare and return `Locator` instances, then instantiate and call the Commands classes directly within their tests. The resulting tests would be slightly more verbose, but completely self-contained, requiring no stepping through code calls to follow the logic path. This does not allow for much abstraction of common test steps, but for simple and small test projects, it enables quick development.

### UI Models with Simple or Complex Actions

Alternately, a team could choose to create Page classes that both store `Locators` as well as instances of Commands classes, then expose individual methods that perform singular actions against mapped elements. The Page classes would be instantiated somewhere accessible to the test, and the resulting code calls in the test would be clean and concise. With descriptive method names on the Page classes, minimal stepping through code would be required in order to infer the logic path.

Should the tests be more complex, or the UI workflows containing repetitive tasks, a team could build upon the above design and add more complex procedure methods that call multiple individual actions. This lessens the code required directly in a test, but with descriptive method names the resulting code calls would remain clean and concise. More reusable code is created, but would require a moderate amount of stepping through code calls in order to follow the details of the logic path.

A secondary benefit of this and similar designs is the logger injection for the Commands classes. `WebCommander` and `WebInspector` both use overloaded constructors to allow users to choose either a default logger or custom logger. A default logger will result in logging and reporting output that shows events originating from the Commands classes. Using a custom logger, teams can pass the `.class` value of the Page class to the `WebCommander` and `WebInspector` instances that reside on the Page class. This results in logging and reporting output that shows events originating from the Page classes, adding better clarity to the reporting output. This is explained in more detail in the [Commands]() section.

Finally, if the Page classes are becoming too cumbersome due to a complex UI, the Locators could be separated into a Map class, allowing for some division in the UI Modeling components. A test would call a Page class in order to perform actions against elements, and the Page class would call the Map class in order to retrieve the element mappings.

## **Commands**

The Commands classes are a collection of functionality that execute actions against elements on the UI, or the browser itself. In short, these are test steps ranging from actions such as clicks and inputs, inspections such as reading text or determining the state of an element, browser actions such as navigation or window management, and validations.

The [`Commands`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Commands.java) class is abstract and sits atop the Commands Hierarchy (described in greater detail [here]({{ site.baseurl }}{% link _docs/extending-qadenz.md %})). This class provides common commands and is completely agnostic of any underlying automation frameworks. 

`Commands` is home to the validation functionality (explained [here]({{ site.baseurl }}{% link _docs/validations.md %})), as well as the `log()` and `annotate()` commands (explained [here]({{ site.baseurl }}{% link _docs/logging-and-reporting.md %})).

## The WebCommander

[`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) is a child of `Commands`, and wraps Selenium `WebElement` and `Actions` commands for use in tests and UI Models. Each of the methods on `WebCommander` provide a number of activities beyond simply performing Selenium actions. The workflow for `WebCommander` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Perform the `WebElement` or `Actions` command.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the application under test.
6. Re-throw the exception to stop execution of the test.

All `WebCommander` commands include an Explicit Wait during `WebElement` initialization. Commands that involve a Click action include a wait for the clickability of the target element. All other commands include a wait for the visibility of the target element to be `true`.

## The WebInspector

The [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java) class works alongside the `WebCommander`. Instead of performing actions on UI elements, `WebInspector` work to extract and return data from UI elements. This includes retrieving text or attribute values from elements, and discovering selected, enabled, or visible state of elements. The workflow for `WebInspector` methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Retrieve the required information from the `WebElement`.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the application under test.
6. Re-throw the exception to stop execution of the test.

All `WebInspector` commands that involve a single target element also include an Explicit Wait during `WebElement` initialization for the visibility of the target element to be `true`. Commands that involve a `List<WebElement>` do not include an Explicit Wait.

## Constructors & Loggers

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
09:55:02.583 | INFO | DashboardPage | Verifying Condition - Visibility of element [Company Logo Image] is TRUE.
09:55:02.998 | INFO | DashboardPage | Result - PASS
```

## Browser Commands

The [`Browser`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Browser.java) class manages the browser under test. This includes activities within the browser, but outside the rendered DOM of the application. Actions such as navigation, alert handling, cookie management, and switching between browser windows, are all handled by the `Browser`. The methods on the `Browser` class are static, and are able to be called from anywhere within the scope of the test, be it from the UI Modeling layer, or directly from the test itself.

## **Conditions & Expectations**

The pairing of [`Conditions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Conditions.java) and [`Expectations`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Expectations.java) becomes the core evaluative logic of Qadenz. These classes are used to determine if the state of the UI under test meets a given criteria.

When used with the Explicit Wait provided by the `pause()` command, or with the Assertions provided by the `verify()` and `check()` commands, a Condition is used to describe precisely what on the UI is being evaluated, and the accompanying Expectation describes exactly the expected outcome of the evaluation.

The goal of the Condition/Expectation pairing is to provide a unified structure for making evaluations throughout a testing project, rather than mixing Selenium `ExpectedConditions` calls for Explicit Waits, and various syntax patterns for test assertions. The resulting code is simple to read and quickly understand, and is easily maintained as the application under test evolves.

## How does it work?

Each Condition accepts a variety of Expectations based on the type of evaluation being performed. For example, Conditions pertaining to element state (visibility, selected, enabled) accept boolean-based Expectations, while Conditions evaluating text will require String-based Expectations.

From a technical perspective, Conditions and Expectations is a Hamcrest wrapper. The output of a Condition is based on whether the outcome of the evaluation matched the expected result. The Condition will return `true` is there was a match, `false` otherwise.

## Logging evaluations

Implementations of the [`Condition`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Condition.java) interface will provide a description of the Condition under evaluation to the logs and subsequent report output. Should the evaluation fail to match with the Expectation, the Condition will also provide a summary of the actual results.

Implementations of the [`Expectation`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/conditions/Expectation.java) interface will provide a description of the Expectation that will be added to the Condition logs, as well as the wrapped Hamcrest Matcher that will be used to compute the result of the Condition.

## **Validations**

Qadenz is built with a versatile and easy to use approach to performing validations against the UI under test. The key component of this approach is the inclusion of `Conditions` and `Expectations` (explained [here]({{ site.baseurl }}{% link _docs/conditions-and-expectations.md %})) as evaluation criteria for validations. These classes combine Selenium `WebElement` inspections to retrieve the 'actual' state of the UI, with Hamcrest Matchers to provide the 'expected' state, to return a result that represents a passing or failing outcome.

## Why not Assertions?

Unit testing frameworks such as TestNG or JUnit include assertion functionality as a core component, and are relatively simple to use. Being open-ended frameworks, however, individual users may tend to express very similar validations in a variety of different assertions. This leads to inconsistent coding patterns, and more difficult maintenance of test code.

Using `Conditions` and `Expectations` allows a team to ensure all contributors are following the same pattern for validations.

That said, Qadenz does employ a single TestNG assertion, the `assertTrue()` method, as a means of validating a Condition. The `result()` of a Condition is a simple representation of whether the state of the UI under test meets expectation. If the output of the Condition evaluation matches the Expectation, `result()` will return `true`. 

By passing this result to the `assertTrue()` method, Qadenz is ensuring that a passing result depends on the Condition evaluation meeting the Expectation. If not, the validation will fail.

## Assertion Types

The concept of Hard Assertions and Soft Assertions are not new in the test automation world. Qadenz implements both concepts by way of the `verify()` and `check()` methods.

`verify()` represents a Hard Assertion. If the validation fails, the test will be marked as failed and execution will be stopped.

`check()` represents a Soft Assertion. If the validation fails, the test will be marked as failed, but execution will be allowed to continue until a call to `Assertions.flush()` is made, which will stop execution of the test if any failures have been encountered.

The `verify()` or `check()` methods are available on the Commands Hierarchy and are callable on any descendant class of `Commands`. The mechanics of using these validations are the same, with the only difference being an additional step with `check()` required to call `Assertions.flush()` in order to handle any failed Soft Assertions and stop execution.

## Grouped Conditions

Validations in Qadenz are further enhanced with the ability to evaluate multiple Conditions as a group. In scenarios where a single UI action can trigger multiple verification points in a test, a tester may have to express multiple assert statements to ensure necessary coverage. If, for example, the first assertion were to fail, the remaining assertions would remain unchecked until either the UI under test is fixed, or the test scenario is executed manually. 

Using Qadenz, a tester is able to execute these same validations in one call to `verify()` or `check()`, and will receive results for each Condition evaluation regardless of individual results. If again, the first validation fails, Qadenz will perform handling tasks on the failure, then proceed to evaluate each of the other Conditions that were passed. In the case of a `verify()` with multiple Conditions where one or more have failed, halting of test execution will be delayed until all Conditions have been evaluated, which will ensure that the test step is completed in its entirety. 

## Managing Soft Assertions

The `check()` methods works alongside the static `Assertions.flush()` method to delay execution stoppages in the event of failed validations. As calls to `check()` are made and executed through the course of a test, the [`Assertions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/Assertions.java) class tracks whether any failures have been encountered. When the call to `Assertions.flush()` is made, this tracker is checked. If any failures are present, execution will be stopped. If no failures are found, execution continues.

Since the tracker is live for the entire duration of a test, there is no limit to how many calls to `Assertions.flush()` can be made throughout a test. It is possible then, to create a series of "checkpoints" in longer tests whenever it is deemed sensible to stop a test if failures have been found. This is especially convenient for smoke to end-to-end tests where a focus on completion of test is important for a full accounting of key validation points. 

Please note, however, that at least one call to `Assertions.flush()` is required in tests where only `check()` validations are made. If no call is made, the test will be allowed to continue to completion, and individual steps will be reported as failed (if validations have indeed failed), but the test as a whole will be reported as passing. Since the Qadenz reporter is integrated with TestNG, the `AssertionError` thrown by the `flush()` method in the event of individual failures is required to mark the test itself as failed.

One additional design consideration must be made when mixing `verify()` and `check()` validations within the same test. When a `check()` validation is made, and is followed by a `verify()` validation prior to calling `Assertions.flush()`, if the `verify()` validation fails, the test will be stopped at the failed `verify()` validation.

## Screenshots

Qadenz validations are built to capture screenshots whenever a Condition evaluation fails. If screenshots are desired for validation failures, no special action need be taken. Should screenshots *not* be needed for a validation, disabling is easy with the overloaded `verify()` and `check()` methods.

Adding a call to `Screenshot.SKIP` as the first argument in either `verify()` or `check()` will disable screenshots from being captured if the evaluations for any accompanying Conditions fail.

```
verify(Screenshot.SKIP, Conditions.visibilityOfElement(locator, Expectations.isTrue());
```

A `boolean` could also be passed to achieve the same outcome. The `Screenshot.SKIP` value is intended as a means to keep the resulting code easily readable at a glance.

## **Configuring Qadenz**

## TestNG Suite XML Parameters

Primary suite configuration in Qadenz is made via parameters on the TestNG Suite XML file.

- `gridHost`: Directs the tests to a Selenium Grid Hub. The expected value is simply the IP address of the Hub. The default port number (4444) is pre-configured.
- `browser`: Specifies which browser will run the test.
- `browserVersion`: An optional parameter that allows a specific version of the browser to be used for the test.
- `browserConfigProfile`: Provides arguments for the browser configuration. This is optional. If the parameter is not declared, or if the parameter value does not match an existing profile name, no arguments will be passed to the `WebDriver` for the execution cycle.
- `platform`: Directs the tests to a Node running a given OS. This parameter is optional.
- `timeout`: Sets the max allowable time for the Explicit Waits to run in Qadenz. This is an optional parameter, expressed in seconds. If this parameter is not given, a default of 30 seconds will be used.
- `appUrl`: The full URL of the application under test.
- `retryInterceptedClicks`: This works with the `click()` command. If a click throws an `ElementClickInterceptedException`, enabling this parameter will direct the `WebDriver` to retry the click using the `Actions` API. Do note, however, that this exception is typically a symptom of a selector issue. This parameter is optional, and defaults to enabled.

The `gridHost` is a Suite-level parameter, and is assigned only once during execution prior to any tests executing. The others are all Test-level parameters, and are assigned before each `<test>` node on the TestNG Suite XML file.

The Test-level parameters allow for configuration changes to take place during the execution cycle. For example, if the goal of the suite is to run the same set of tests in multiple browsers, rather than create 3 different Suite XML files, 3 `<test>` nodes can be added to the same Suite XML file with different configurations.

In the below example, both `<test>` nodes will use the same set of parameter values:

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    <parameter name="browser" value="chrome"/>
    
    <test name="Authentication Tests">
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Permissions Tests">
        <classes>
            <class name="com.acme.test.cases.PermissionsTest"/>        
        </classes>
    </test>
</suite>
```

Parameters on TestNG Suite XML files obey scoping rules. That is to say, a parameter declared within a `<test>` will take precedence over the same parameter as declared on the `<suite>`. 

In this next example, the second `<test>` will override the `browser` parameter, while the others will use follow what is declared at the `<suite>`.

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    <parameter name="browser" value="chrome"/>
    
    <test name="Authentication Tests">
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Permissions Tests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.acme.test.cases.PermissionsTest"/>        
        </classes>
    </test>

    <test name="Account Management Tests">
        <classes>
            <class name="com.acme.test.cases.AccountManagementTest"/>        
        </classes>
    </test>
</suite>
```

In another example, to run the same set of tests in three different browsers as part of the same execution, Test-level parameters can be used accordingly:

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    
    <test name="Authentication Tests">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Authentication Tests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>

    <test name="Authentication Tests">
        <parameter name="browser" value="edge"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
</suite>
```

## Browser Configuration Profiles

In some cases, extra configuration arguments need to be passed to the browser Options instance for `WebDriver` configuration. A common case for this is instructing a browser to run in headless mode. Qadenz provides a simple means to set these configurations, and allows for multiple "profiles" of these configurations to be set on the TestNG Suite XML file. For example, if a tester wishes to run in headless mode on the Selenium Grid, but run in a native browser when executing locally, the switch is as simple as altering a parameter value.

Configurations are stored in JSON files within the project's `resources` directory. To create a new configuration, within the `resources` directory, add a new folder called `config`, and create a new JSON file. This file must be named `{browser}-config.json`, where `{browser}` is the name of the browser being configured. A separate file must be created for each browser
requiring configuration.

This example configures Chrome to run in headless mode.

```
[
    {
        "profile": "headless",
        "args": [
            "--headless",
            "--window-size=1920,1080",
            "--disable-gpu"
        ]
    }
]
```

The JSON format is an array of configuration objects. Each object holds `profile` and `args` fields. The `profile` field will be matched to the `browserConfigProfile` parameter on the Suite XML. The `args` is an array of individual arguments to be passed to the `WebDriver` configuration.

## Custom Visibility Checks

The `WebInspector` provides a means to determine whether an element is visible. The inspection conducts a number of checks including matching selectors, element dimensions, attributes, and rudimentary CSS. There are many ways to hide or otherwise render an element invisible, so Qadenz offers a way for testers to insert additional checks into this inspection.

Configurations are stored in JSON files within the project's `resources` directory. To create a new configuration, within the `resources` directory, add a new folder called `config`, and create a new JSON file. This file must be named `visibility.json`.

In the above example, for any element where the `class` attribute contains `hidden`, the element will be determined to be invisible. This JSON configuration can contain as many entries on the array as needed to cover all scenarios in the UI under test where an element can become hidden. 

```
[
    {
        "attribute": "class",
        "value": "hidden"
    }
]
```

The JSON format is an array of configuration objects that hold an `attribute` and `value` fields. 

It is important to note that the `getVisibilityOfElement()` inspection presumes the element to be visible, and runs this series of checks until one check indicates that the element is not visible. With that in mind, the only items that should be added to this JSON configuration should items that define invisible or hidden elements.
