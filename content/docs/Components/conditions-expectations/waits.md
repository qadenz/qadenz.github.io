---
title: "Waits"
linkTitle: "Waits"
description: >
  Synchronizing on the UI with Explicit Waits, driven by the same Conditions and Expectations.
weight: 2
---
Tests move faster than the applications they drive. A wait is how a test stays in step with the UI, holding until the page reaches the state the next action needs.

Selenium offers Explicit Waits and Implicit Waits, and Java offers `Thread.sleep()`. Each works, and each carries tradeoffs. Qadenz commits to the Explicit Wait as its one synchronization approach, and drives it with the same `Conditions` and `Expectations` used for validations. The Condition that asserts a state can also be waited on, so there is one vocabulary for both.

Qadenz does not use the WebDriver Implicit Wait. An Implicit Wait is a blunt, time-based catch-all, and it pairs badly with Explicit Waits. Combining the two produces [unexpected side effects](https://www.selenium.dev/documentation/webdriver/waits/#implicit-wait). Committing to Explicit Waits alone keeps synchronization precise and predictable.

## What about ExpectedConditions?

Selenium's `ExpectedConditions` is the familiar way to express a wait, and it covers a lot of ground. In his [2017 Selenium State of the Union](https://youtu.be/gyfUpOysIF8?t=1773), Simon Stewart calls it a "useful dumping ground for functionality" that "brutally violates this attempt to be concise." The usefulness is real, but the sprawl means the right method is not always the obvious one.

Driving waits with `Conditions` and `Expectations` instead keeps a wait concise and exact, reads the same as every validation elsewhere in the test, and produces the same clear logging on the report.

## Invoking a wait

Call `pause()`, passing a `Condition` and `Expectation` pairing. The wait holds until the Condition is met, or until the configured timeout elapses.

Suppose the application shows a confirmation banner when an item is added to the cart, and the banner covers the navigation menu until it fades. Wait for it to clear before moving on.

```java
commander.pause(Conditions.visibilityOfElement(addItemConfirmation, Expectations.isFalse()));
```

The timeout that bounds every wait is set once on the suite (see [Configuration]({{< relref "/docs/Components/Configuration/_index.md" >}})).

Unlike a validation, `pause()` takes a single Condition rather than a group. Waits should be used as sparingly as possible, since each one adds to execution time, and one precise wait usually states exactly what the test is waiting for.

## When no built-in Condition fits

The built-in Conditions cover the common cases, but not all of them. When none expresses the wait a test needs, build a custom Condition rather than dropping out of the vocabulary. A custom Condition works with `pause()` exactly like a built-in one, and logs the same way on the report. See [Extensibility]({{< relref "/docs/Components/conditions-expectations/extensibility.md" >}}).

## A fixed pause, as a last resort

`pause()` is also overloaded to accept a number of seconds, stopping the test for exactly that long with a plain `Thread.sleep()`.

```java
commander.pause(3);
```

This waits blindly, with no knowledge of the UI, so it either burns time or risks being too short. Prefer a Condition-based wait in nearly every case. Keep the fixed pause for the rare spot where nothing observable marks the state the test needs, such as waiting out a fixed animation or a third-party process that gives no visible signal.