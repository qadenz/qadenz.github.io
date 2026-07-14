---
title: "AutomatedWebTest"
linkTitle: "AutomatedWebTest"
description: >
  The base class every test extends, and the owner of the execution cycle from suite start to final report.
weight: 1
---

[`AutomatedWebTest`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/AutomatedWebTest.java) is the base class for every test class in a Qadenz project. Extending it is what enrolls a class in the Qadenz execution cycle: the parameter reading, the driver lifecycle, and the reporting all hang off this class and its parent. A class that holds `@Test` methods must extend `AutomatedWebTest`, directly or through an intermediate class, or its tests run with none of that machinery in place.

## The class hierarchy

`AutomatedWebTest` extends `AutomatedTest`, and the two split the work by how specific it is to web testing.

`AutomatedTest` sits at the top of the hierarchy and holds configuration that any kind of automated test would share. It stamps the suite's start and end times onto the [`WebConfig`]({{< relref "webconfig.md" >}}), and it carries the `@Listeners` declaration that registers the Qadenz reporter. Nothing here touches a browser.

`AutomatedWebTest` adds everything specific to driving one: reading the Suite parameters, launching and quitting the driver, and loading the application. A project's test classes extend `AutomatedWebTest`, so they inherit both layers at once.

## The execution cycle

Qadenz hangs its setup and teardown on standard TestNG lifecycle annotations. Followed from the start of a suite to the end, the cycle runs in this order.

**Before the suite** (`@BeforeSuite`)

Inherited from `AutomatedTest`. Qadenz captures a timestamp as the `suiteStartDate` on the `WebConfig`, which the reporter later uses to calculate the run's duration.

**Before each `<test>`** (`@BeforeTest`)

Once for every `<test>` node on the Suite XML file, Qadenz reads the declared parameters, validates them through the `XmlParameterValidator`, and stores the results on the `WebConfig`. A missing required parameter, or an unrecognized `browser` or `platform`, fails the run here, before any browser launches. [Suite Parameters]({{< relref "suite-parameters.md" >}}) covers the full set and how each is validated.

**Before each test method** (`@BeforeMethod`)

Repeated ahead of every `@Test` method, this is where a test's browser comes to life. Qadenz initializes the assertion collector for the test, asks the `CapabilityProvider` to assemble the browser options (applying any [Browser Config Profile]({{< relref "browser-config-profiles.md" >}}) named for the run), launches a `RemoteWebDriver` against the Grid at `gridHost`, and hands it to the [`WebDriverProvider`]({{< relref "webdriverprovider.md" >}}). It then loads the `appUrl`. Because this runs per method, every test method starts on a driver of its own.

**After each test method** (`@AfterMethod`)

Qadenz quits the driver, ending the browser session and discarding its state. The next test method begins the sequence again with a fresh one.

**After the suite** (`@AfterSuite`)

Inherited from `AutomatedTest`. Qadenz captures a second timestamp as the `suiteEndDate`, closing the window the reporter measures.

**Reporting**

Qadenz does not disable the default TestNG reporters, so the standard HTML and XML reports and the `emailable-report.html` are still produced. Alongside them, the `TestReporter` registered on `AutomatedTest` generates the Qadenz reports in JSON and HTML. See [Test Results]({{< relref "/docs/Components/Test-Results/_index.md" >}}) for what those contain.

## Inserting custom configuration

A project often needs setup the framework cannot know about: seeding reference data, standing up a service, or signing in through an API before the browser opens. Rather than modify Qadenz, insert an intermediate class between `AutomatedWebTest` and the test classes, and give it the TestNG lifecycle methods the project needs.

```java
public class AcmeAutomatedWebTest extends AutomatedWebTest {

    @BeforeSuite
    public void seedReferenceData() {
        // project setup
    }
}

public class InventorySearchTest extends AcmeAutomatedWebTest {
    // @Test methods
}
```

Because Qadenz builds its own lifecycle on these same standard annotations, a project's `@BeforeSuite`, `@BeforeMethod`, and the rest join the same TestNG workflow rather than working around it. Test classes extend the intermediate class and inherit the project's configuration on top of the full Qadenz cycle beneath it. All of TestNG's tools for ordering and data, method dependencies, factories, and data providers, remain available to that intermediate class.
