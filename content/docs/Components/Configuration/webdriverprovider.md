---
title: "WebDriverProvider"
linkTitle: "WebDriverProvider"
description: >
  The single place the active WebDriver lives, held per thread so parallel tests never reach into each other's browser.
weight: 4
---

[`WebDriverProvider`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebDriverProvider.java) is where the active `WebDriver` lives. [`AutomatedWebTest`]({{< relref "automatedwebtest.md" >}}) sets the driver here when it launches one before each test, and every component that needs to touch the browser, the commands and the element finder among them, reads it back from the same place rather than being handed a driver of its own.

## One place to reach the driver

A `WebDriver` has to be available to a lot of code: every command that acts on the page, the finder that resolves elements, and the base class that opens and closes the session. The alternative to a central holder is passing the driver into all of it, through command constructors, into page objects, and down into helpers, so that every signature carries a `WebDriver` whether it uses one directly or only forwards it along.

Qadenz keeps the driver in one static location instead. A component that needs the browser asks `WebDriverProvider` for it at the moment of use, and nothing has to route a driver through code that only meant to pass it on.

## Held per thread

The catch with a single static driver is parallel execution. With `parallel="methods"`, several tests run at once, each in its own browser, and a plain static field would hold one driver for all of them to share and collide over.

`WebDriverProvider` holds the driver on a `ThreadLocal` for exactly this reason. Each thread gets its own slot, so `getWebDriver()` returns the driver belonging to the calling thread's test and no other. The static field makes the driver reachable from anywhere; the `ThreadLocal` makes "anywhere" still resolve to the one correct browser for the test in hand. Together they are what let the per-test isolation the framework enforces hold up under parallel load.

The slot's contents track the test lifecycle. `AutomatedWebTest` installs a fresh driver in its `@BeforeMethod` and quits it in the `@AfterMethod`, so a thread's slot holds a live driver only for the span of its current test.

## Reaching the driver directly

The command and inspector layers cover the great majority of interactions, but a project occasionally needs the raw `WebDriver` for a low-level Selenium call that has no command in front of it. Retrieve it with the same call the framework makes internally.

```java
WebDriver webDriver = WebDriverProvider.getWebDriver();
```

Because the driver is held per thread, this returns the browser for the current test and stays correct under parallel execution. Setting the driver is the base class's job, done once as each test starts, so a project reads from `WebDriverProvider` but does not write to it.