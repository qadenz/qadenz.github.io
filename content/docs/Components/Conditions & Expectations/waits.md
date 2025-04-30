---
title: Waits
description: >
  Waits
weight: 2
---

Selenium provides both Implicit and Explicit Wait types, and Java provides the `Thread.sleep()`. While all technically valid, each have their own advantages and disadvantages. Qadenz does not implement the `WebDriver` Implicit Wait. The Implicit Wait can serve as a basic catch-all wait approach in simple projects, but the flexibility is limited, and more importantly, it tends to not pair well with Explicit Waits. Using both in conjunction can cause some very [unexpected side effects](https://www.selenium.dev/documentation/webdriver/waits/#implicit-wait).

Qadenz opts for the Explicit Wait as the primary UI synchronization approach, and pairs this concept with `Conditions` and `Expectations` to define the criteria for the syncronization.

## What about ExpectedConditions?

The `ExpectedConditions` class is well known among automation engineers and provides a wide variety of wait-conditions to handle timing and synchronization. In his [2017 Selenium State of the Union](https://youtu.be/gyfUpOysIF8?t=1773), Simon Stewart calls `ExpectedConditions` a "useful dumping ground for functionality" that "brutally violates this attempt to be concise". While there is no denying the usefulness of a class such as `ExpectedConditions`, it could also be said that the method options aren't always intuitive for choosing an ideal fit. In that same talk, Mr. Stewart uses the original intent and the evolution of `ExpectedConditions` as an example of why it's important that developers not punish themselves too harshly for code written in the past.

`Conditions` and `Expectations` are implemented in Qadenz for waits with the intent of making the invocations of wait-conditions much more concise and exact, thus enhancing the readability (and maintainability) in test code, as well as improving the clarity and precision of logging output that is captured and presented on the final reports.

## Invoking a Wait

Invoking a Wait is as simple as calling the `pause()` command, and passing an appropriate `Condition`/`Expectation` pairing.

If an application under test displays a confirmation banner when an item is added to a shopping cart that blocks access to the navigation menu, for example, the test would benefit from pausing execution until the banner confirmation disappears after a few seconds.

```java
commander.pause(Conditions.visibilityOfElement(addItemConfirmation, Expectations.isFalse()));
```

Unlike the validation functionality that consumes the `Condition` and `Expectation` pairing, the `pause()` method does not provide the ability to pass multiple Conditions as a group for a wait. This is done in the mindset that waits should be used as sparingly as possible to avoid unnecessary lenthening of execution times.

## I want my ~~MTV~~ ExpectedConditions

While Qadenz does implement certain functionality in an opinionated manner, there is no reason to prevent access to the underlying tools for use in a customized solution within a consuming test project. By [extending the `WebCommander`](/components/commands/extensibility), it would be very possible to create an instance of the `WebDriverWait` and pass an `ExpectedCondition` to execute a wait. The recommended practice would be to use this approach only if a suitable `Condition` does not exist for the needed wait.
