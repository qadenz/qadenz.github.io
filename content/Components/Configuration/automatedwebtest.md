---
title: AutomatedWebTest
description: >
  Configuration
weight: 1
---

The [`AutomatedWebTest`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/AutomatedWebTest.java) is the base class for all test classes in a Qadenz-powered test project. This class is responsible for gathering parameter values from the TestNG Suite XML file, configuring and starting a `WebDriver` instance on a Selenium Grid for each test method, stopping the `WebDriver` after each test, and invoking the reporters after the Suite has completed.

Classes that hold `@Test` methods must extend this class in order for tests to run using Qadenz configurations.

### The Execution Cycle

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
