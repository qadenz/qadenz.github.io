---
title: "Configuration"
linkTitle: "Configuration"
description: >
  The driver lifecycle, parallelism, and reporting that most teams build and maintain by hand are the framework's job. A project extends one class and declares a few values.
weight: 1
---
Every Selenium project eventually grows a harness around the raw driver. A factory to build browsers, thread handling so tests can run in parallel, teardown to clean up after each one, and a reporter to turn results into something a person can read. That harness is code a team writes once and then owns forever. It has to keep pace with Selenium, every new engineer has to learn it before they can be trusted with it, and when it breaks, tests that passed yesterday fail today for reasons that have nothing to do with the application under test.

Qadenz starts from the position that a team should not have to build or maintain that layer at all. Configuration in a Qadenz project is a matter of declaring, not coding. A test class extends one base class, a TestNG Suite XML file names a few values, and the driver launches, the application loads, tests run in parallel, and the reports are generated, with none of that machinery living in the project to be maintained.

## The lifecycle belongs to the framework, not the project

Every test class extends [`AutomatedWebTest`]({{< relref "automatedwebtest.md" >}}), and that single inheritance is the whole of the wiring. The base class reads and validates the Suite parameters before the tests begin, launches a driver and opens the application before each test method, quits the driver after each one, and registers the reporter that turns the run into logs and HTML. A test class inherits all of it and adds only `@Test` methods.

```java
public class AuthenticationTest extends AutomatedWebTest {

    @Test
    public void signInSucceedsWithValidCredentials() {
        // test steps
    }
}
```

That inheritance also settles two costs a homegrown harness never stops charging. Onboarding an engineer means teaching them to extend a class, not walking them through bespoke infrastructure before they can be trusted with a test. And when Selenium moves, keeping current is the framework's problem to solve, not a task sitting on the team's backlog.

## A run is described, not compiled

The decisions that change from one run to the next, which browser, where the Grid lives, which application, how long to wait, live on the Suite XML file rather than in code. A run is described by its parameters, so the same compiled tests execute against a different browser or a different environment by changing values, never by editing and recompiling.

```xml
<suite name="Regression" parallel="methods" thread-count="6">

    <parameter name="gridHost" value="127.0.0.1"/>
    <parameter name="browser" value="chrome"/>
    <parameter name="appUrl" value="https://app.example.com"/>

    <test name="Authentication">
        <classes>
            <class name="com.example.tests.AuthenticationTest"/>
        </classes>
    </test>
</suite>
```

Because the configuration is data, the person running a suite does not have to be the person who wrote it. A pipeline points the tests at a staging or a production environment by setting `appUrl`. A lead reruns the full regression pack against a different browser without touching a line of Java. The tests are a fixed asset, and the run is a set of choices made at launch. Three parameters are required, `gridHost`, `browser`, and `appUrl`, and the rest carry sensible defaults. [Suite Parameters]({{< relref "suite-parameters.md" >}}) documents the full set.

## Parallelism that stays honest

Qadenz starts a new driver in the `@BeforeMethod` and quits it in the `@AfterMethod`, so every `@Test` method runs in its own browser session and throws it away at the end. No cookies, no logged-in state, and no leftover navigation carry from one test into the next. Each test begins from the application URL and nothing else.

That isolation is what makes running tests in parallel safe rather than merely fast. Because a test never shares a browser with its neighbors, there is no shared state to corrupt and no execution order to protect, so the flaky failures that appear only under load simply have no place to originate. A team can drive a suite's runtime down by adding threads and trust the result, which is the difference between a suite that scales and one that has to run serially to stay believable. It is also why `parallel="methods"` is the recommended setting: the per-method isolation the framework already enforces is exactly what parallelism needs.

The same reset nudges a suite toward atomic tests, long held up across the industry as the mark of a healthy one. When a test cannot inherit a logged-in session or a half-filled form from the one before it, the natural way to write it is to set up what it needs and stand on its own. Qadenz does not make that discipline complete, though. A suite can still couple its tests through shared fixtures, data seeded in a fixed order, or an external system left in a particular state, and none of that lives in the browser session the framework resets. Test independence remains a philosophy a team has to adopt and hold on its own. What the framework does is start from the atomic default rather than work against it, so the path of least resistance and the correct one run together.

## Local and CI run the same code

Qadenz builds every session as a `RemoteWebDriver` pointed at a Selenium Grid. There is no separate local-driver path. Running on a developer's laptop means running a Selenium Server in standalone mode on that laptop and pointing `gridHost` at `127.0.0.1`; running in CI means pointing it at a full Grid. The port and endpoint are fixed, so the only value that moves between the two is `gridHost`.

This closes one of the more corrosive gaps on an automation team, the case where a test passes locally and fails in the pipeline because the two ran through different driver stacks. Under Qadenz they run through the same one, so "works on my machine" stops being a category of failure and the results the pipeline reports are the results an engineer can reproduce at their desk. Qadenz gives up the small convenience of a bundled local driver to buy that trust, and on a shared suite the trust is worth far more than the convenience.

## Where the run's state lives

Two small classes hold the state the rest of the framework reads. [`WebConfig`]({{< relref "webconfig.md" >}}) is where the validated parameter values land, available to any component that needs to know the timeout, the application URL, or the browser in play. [`WebDriverProvider`]({{< relref "webdriverprovider.md" >}}) holds the active driver on a `ThreadLocal`, so the command layer can reach the right browser for the current test without a project ever passing a driver around by hand. Both follow from the same idea: the framework owns the moving parts, and the project reads from them when it needs to.

## What a team gets

- **Less infrastructure to own.** The driver lifecycle, thread handling, teardown, and reporter wiring arrive with the dependency and are maintained upstream, not homegrown code the team carries and updates.
- **Runs configured by data, not code.** A pipeline or a lead retargets the browser and the environment by changing Suite XML values, so the same compiled tests run anywhere without a rebuild.
- **Trustworthy parallelism.** A fresh browser per test removes shared state, so adding threads drives the suite's runtime down without breeding the flaky failures that only appear under load.
- **One driver path everywhere.** Local and CI both run the same `RemoteWebDriver` against a Grid, so a result in the pipeline reproduces at an engineer's desk and "works on my machine" stops being a category of failure.

## The pieces

- **[AutomatedWebTest]({{< relref "automatedwebtest.md" >}})** is the base class every test extends, and the owner of the execution cycle from suite start to final report.
- **[Suite Parameters]({{< relref "suite-parameters.md" >}})** is the full reference for the TestNG XML parameters that configure a run, including which are required and what the optional ones default to.
- **[WebConfig]({{< relref "webconfig.md" >}})** stores the validated parameter values and run metadata for the length of the execution.
- **[WebDriverProvider]({{< relref "webdriverprovider.md" >}})** holds the active `WebDriver` on a `ThreadLocal` and hands it to the components that need it.
- **[Browser Config Profiles]({{< relref "browser-config-profiles.md" >}})** pass extra arguments to the browser, such as headless mode, and let a run switch between them by naming a profile on the Suite XML.
