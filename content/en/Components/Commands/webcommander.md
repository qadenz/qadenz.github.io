---
title: Web Commander
description: >
  Web Commander Description
weight: 1
---

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
