---
title: "Commands"
weight: 6
---

Commands are a collection of classes that either act upon elements, interrogate elements, or control the browser under test. The methods on these classes represent steps in a test case. Commands can be used directly inside tests, or can be wrapped on Page Objects to create a contextually friendly reference to actions performed on the UI under test.

Methods on the Commands classes drive the supporting functionality that makes Qadenz useful. In addition to performing the surface actions, Commands methods handle WebElement initialization, catching and logging exceptions, and capturing screenshots.

Commands classes cover default Selenium interactions and cover some additional common actions, but are designed to be extensible so that specific needs of an automation project can be met.

## Element Initialization

The methods on both the `WebCommander` and `WebInspector` require a `Locator` instance in order to identify the target element to receive the interaction or the focus of the inspection.

## Elemental Commands

### WebCommander Class

The `WebCommander` performs actions against WebElements. This class also extends the abstract class `Commands`, which is built to be agnostic of any automation tooling. From this class, functionality such as validations, time-based waits, and log wrappers are available to inheriting classes. This design was intended as a future-proofing measure should Qadenz at some point expand to include other underlying tool sets.

Each method on the `WebCommander` includes a number of activities beyond simply performing WebElement interactions. The workflow for these methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Perform the `WebElement` or `Actions` command.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the UI under test.
6. Throw the exception to stop execution of the test.

All `WebCommander` commands include an Explicit Wait during `WebElement` initialization. Commands that involve a Click action include a wait for the clickability of the target element. All other commands include a wait for the visibility of the target element to be `true`.

#### Basic Element Commands

The 4 basic [Selenium WebElement Interactions](https://www.selenium.dev/documentation/webdriver/elements/interactions/) are covered by the `WebCommander`. These are the `click()`, `sendKeys()`, `clear()`, and `select()` functions.

##### Clicks

The primary `click()` method includes a fallback intended for flexibility against tricky DOM configurations. On the chance that an element click would be intercepted by another element and an `ElementClickInterceptedException` Exception is caught, the `click()` method may reattempt the click using `Actions.click()`. This behavior is configurable and is enabled by default. To configure, simply add a parameter to the TestNG Suite XML file.

```xml
<parameter name="retryInterceptedClicks" value="false" />
```

It should be noted that the `ElementClickInterceptedException` can be a symptom of an element selector that needs to be optimized a little further. Before relying on the fallback, it is highly recommended to address the `Locator` first.

An overloaded `click()` method is also available that allows point-precision clicks on an element. Two `int` arguments represent X and Y offsets that allow the click to be placed precisely on an element. Since this method wraps the `Actions` class, the `ElementClickInterceptedException` fallback is not included.

##### Inputs

The `enterText()` method retains the flexibility of the underlying `WebElement.sendKeys()` method to accept both characters (Strings) and enumerated `Keys`. The logging in this method is configured to accurately represent both in a simple manner on the resulting reports.

The `clearAndEnterText()` method combines clearing a field and sending input as a convenience wrapper to save a second method call.

##### Selects

As the method name would imply, the `select()` method wraps the `Select` API to interact with menus constructed on the DOM as `<select>` elements. The `WebCommander` currently selects and deselects options only by visible text as a common use pattern. To gain access to other forms of interaction (by Index or by Value), the best solution is to extend `WebCommander` and create custom commands as needed. (More on this topic below)

#### WebDriver Actions

The `Actions` API is an extremely versatile interface for simulating keyboard, mouse, pen, and wheel actions. The Builder pattern invovled with the `Actions` API would make wrapping various combinations extremely difficult and extremely limited in actual value in a consuming project. While there are some more common and straightforward action sequences that are provided on the `WebCommander`, the `Actions` API is largely intended to build sequences custom to the needs of the UI under test.

The `WebCommander` currently provides `Actions` wrapping for a point-precise `click()` on an element, a `doubleClick()` on an element, a `controlClick()` series on multiple elements, and a `hover()` on an element.

More complex usage of the `Actions` API can be (and should be) wrapped as custom commands on a Project-level `WebCommander` sub-class. More on this in the 'Extensibility' section below.

#### Other Functions

In addition to `WebElement` interactions, the `WebCommander` provides additional functionality that helps to interact with the rendered DOM.

##### Frames

Switching between frames is one such function. The methods involved simply wrap the `WebDriver.switchTo().defaultContent()` and `WebDriver.switchTo().frame()`. To move focus to a child frame, simple invoke the `focusOnFrame(Locator locator)` method with a `Locator` instance that maps of the Frame node on the DOM. Invoking the `focusOnDefaultContent()` method will return focus to the primary frame on the page.

##### Waits

Inevitably, tests will need to work around some timing and synchronization issues. Implementing a time-based wait such as `Thread.sleep()` can work in a pinch, but best practices suggest something more flexible and performant. The `pause(Condition)` method was designed to function as an Explicit Wait, but uses Qadenz's [Conditions and Expectations](/components-old/conditions-and-expectations/) to express the type and criteria of the wait. As with Explicit Waits, the Condition will be evaluated repeatedly until either the Condition is satisfied, or a timeout occurs, at which point the test will be stopped.

##### Screenshots

While failed assertions and caught exceptions will trigger the capturing of a screenshot, there are other occasions where a team might need visual confirmation of the UI state at a given point in a test. This can also provide some additional insights while troubleshooting a troublesome test.

Invoking the `captureScreenshot()` method will save an image of the visible UI and embed the image into the final HTML report.

### WebInspector Class

The `WebInspector` works alongside the `WebCommander`, but instead of acting upon UI elements, this class interrogates them for information. This includes retrieving inner text values, attribute values, element state, and instance counts. Most of the methods on `WebInspector` are used by Conditions and Expectations as part of evaluative logic for verifications and waits, but can be useful through the course of a test for retrieving data from the UI under test.

Each method on the `WebInspector` includes a number of activities beyond simply performing WebElement inspections. The workflow for these methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Retrieve the required information from the `WebElement`.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the UI under test.
6. Throw the exception to stop execution of the test.

#### Element Attributes and Properties

The values assigned to attributes on elements can be retrieved via the `getAttributeOfElement()` method by passing the name of the attribute to be evaluated. Invoking `getAttributeOfElements()` will return the value for all instances of the matching `Locator` on a `List<String>`.

Similarly, the value of a CSS property can be retrieved from an element via the `getCssPropertyOfElement()` method.

#### Element States

Element states can be evaluated and will return a boolean value based on the result.

##### Enabled

The `getEnabledStateOfElement()` method evaluates if an element is enabled for interaction, and returns `true` if the element is in fact enabled. This is useful for determining if elements such as `<input>` elements or `<button>` elements are enabled. The state is determined by first invoking the `WebElement.isEnabled()` method. If the WebElement evaluates as enabled, a second check is performed against [user-defined attributes](/components/locators/#disabled-elements) on the element via the `Locator` instance.

This method presumes an element is enabled until one of the evaluations proves the element is disabled.

##### Selected

The `getSelectedStateOfElement()` method evaluates if an element is selected, and returns `true` if the element is in fact selected. This is useful for determining if elements such as checkboxes or radio buttons are selected. The state is determined by first invoking the `WebElement.isSelected()` method. If the WebElement evaluates as selected, a second check is performed against [user-defined attributes](/components/locators/#selected-elements) on the element via the `Locator` instance.

This method presumes an element is selected until one of the evaluations proves the element is not selected.

##### Visiblity

The `getVisibilityOfElement()` method evaluates if an element is visible on the UI, and returns `true` if the element is in fact visible. The state is determined by first finding all matching DOM nodes for the element selector on the given `Locator`. If more than zero matches are found, the method will then evaluate the dimensions of the first matching node. If the element has a height and width greater than zero, the method will then evaluate the styling and attributes of the element. The element will be checked for `display:none;` and then `visibility:hidden;`, and finally a check for the `hidden` attribute. If these checks result in no match, the method will then evaluate [user-defined attributes](/components/locators/#hidden-elements) on the element via the `Locator` instance. Additionally, the method will catch a `StaleElementReferenceException` throughout this series of evaluations and instantly return a `false` result.

This method presumes an element is visible until one of the evaluations proves the element is hidden.

#### Element Text

The `WebInspector` offers methods that retrieve text of UI elements in a variety of ways and a variety of formats. In addition to returning String values, methods exist to convert and return element text values as Temporals or Numbers as well. This allows a test to be designed where the UI data can be interacted with in a more flexible way, enjoying the ability to perform operations against, comparisons, and validations with the capabilities of these secondary object types.

General text inspection methods allow retrieval of basic element text, a `List` of text values from all matching nodes of the given `Locator`, the currently selected `<option>` value on a `<select>` menu, or the `List` of `<option>` values on a `<select>` menu.

##### Element Text vs Direct Element Text

The `WebElement.getText()` method returns the visible inner text of the given element along with the text of any child elements. In most use cases, this is perfectly fine. Some situations exist, however, when the DOM is constructed in such a way where a target element has child elements that also contain visible inner text that a tester may wish to ignore or avoid for an inspection or validation. For these scenarios, `WebInspector` employs a concept of 'direct text of element'. Methods with `directTextOfElement` in the name will retrieve element text, but prior to returning the text value to the calling method, will filter the visible inner text values of any child elements relative to the given target element.

##### Element Text as a non-String Object

The `WebInspector` includes the means to examine the text of an element, and with the aid of a formatter, parse and convert the text value into a `Number` or `Temporal`.

A `NumberFormat` can be used to drive the converstion of a text value to either an `Integer` or a `Double` value. A `DateTimeFormatter` can be used to drive the conversion to either a `LocalDate`, `LocalDateTime`, or a `LocalTime` object. By using these methods, the UI data can be used in mathematical operations directly without requiring the need to build any parsing or conversion logic into the test.

#### Element Instances

There are also methods on `WebInspector` that focus specifically on all instances of an element. The `getCountOfElement()` simply returns the number of instances on the DOM of a matching `Locator`. This can drive mathematical operations, looping logic, or precision validations of element groups. The `getInstanceOfElementText()` and `getInstanceOfElementAttribute()` will examine all instances of a matching `Locator` and return the index of the first node that contains a matching text or attribute value. This capability can provide an extra convenience in dynamically parameterizing `Locator` selector values based on the state if the UI.

### Logging

The `WebCommander` and `WebInspector` can be instantiated and used either from the tests directly, or from the UI Modeling layer, depending on the design of the test project. Both `WebCommander` and `WebInspector` have overloaded constructors that enable different types of logging to take place, and will both directly impact how the logs are presented on the report output.

The `WebCommander` constructor is used as an example, but the `WebInspector` shares this same pattern:

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

### Extensibility

Both the `WebCommander` and `WebInspector` classes are designed to be extended within automation projects to enable custom commands to be created as needed by the UI under test.

Starting at the class-level, the class will obviously need to extend either `WebCommander` or `WebInspector`. Then, a `Logger` instance and constructors need to be added.

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

Then, the anatomy of the command method is as follows:

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

The first operation in a command method is to log the command being executed (at the `INFO` level). Qadenz uses Logback, which provides the `{}` placeholder for additional values to be inserted. It is recommended to utilize this where possible to convey an appropriate level of detail in the logs and subsequent reports.

Next, the `try` block will initialize a `WebElement` using the inherited `WebFinder` instance, then perform any necessary actions. If the method is on a `WebInspector` sub-class, the `return` should take place within the `try` block.

The `catch` block should be set to catch the appropriate Exception, though a wide net may be cast by simply catching all descendents of `Expception`. A multi-catch or `finally` could be employed here if the use case is appropriate. Within the `catch` block, the Exception will be logged (at the `ERROR` level) and, if appropriate, a screenshot captured using the inherited `Screenshot` instance.

Finally, re-throw the Exception. Throwing a new instance of `RunTimeException` with a context-friendly message would be an appropriate alternative to allowing the Exception to surface to the test level, requiring a `try/catch` in the test itself or an Exception to be added to the method signature. This is obviously a preferential decision to be made within the automation project. Qadenz has simply been designed to avoid the need to do this.

Once complete, instantiating the new commands class in either the UI Models or directly in the test method will make functionality available for use.

## Browser Class

The `Browser` class manages the browser under test. This includes activities within the browser, but outside the rendered DOM of the application. Actions such as navigation, alert handling, cookie management, and switching between browser windows, are all handled by the `Browser`. The methods on the `Browser` class are static, and are able to be called from anywhere within the scope of the test, be it from the UI Modeling layer or directly from the test itself.
