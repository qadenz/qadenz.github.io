---
title: "WebFinder"
weight: 2
---

`WebFinder` initializes WebElements. This is not a class that will typically be invoked directly in day-to-day use of Qadenz, but it's a class worth familiarity in terms of how Qadenz initializes WebElements for use in Commands classes, especially if the need to create custom commands should arise.

## On-the-fly Element Initialization

When `Locator` objects are passed to a Command, the `WebFinder` is called to initialize a `WebElement` using the CSS selector from the `Locator`. This process is performed for each interaction, inspection, and validation method, immediately prior to the `WebElement` being acted upon. The intent of this approach is to mitigate situations that might produce a `StaleElementReferenceException`. Additionally, when errors are encountered while initializing or interacting with a `WebElement`, the problem can be captured and presented in the logs/reports far more clearly.

## To Wait, or Not To Wait...

The `WebFinder` is the only class where the `ExpectedConditions` class is used within Qadenz. Methods are avaialable to initialize either a singe `WebElement` or a `List<WebElement>` using no explicit wait at all (immediate initialization), with a wait for the element(s) to become visible, the element(s) to become present on the DOM, or a wait for the element(s) to become clickable.

These initializer methods will attempt to initialize the given element every 500 milliseconds up to the limit of time expressed by the timeout value. If the attempt reaches the timeout, an appropriate exception will be thrown and the test will be stopped.

In the case of `WebCommander`, methods that perform a click action will initialize the target `WebElement` after it becomes clickable, and all other interactions will initialize the `WebElement` after it becomes visible.

On the `WebInspector`, methods wait until the target element is visible prior to acting, with the exception of methods that interact with multiple instances of a `Locator`. In these cases, the `List<WebElement>` is initialized with no wait.
