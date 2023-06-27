---
title: "Configuration"
weight: 2
description: >
  These objects, from the `dev.qadenz.automation.config` package, handle Suite and Test configuration tasks.
---

## AutomatedWebTest

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

## WebConfig

[`WebConfig`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebConfig.java) stores information that Qadenz uses at various points throughout the execution cycle. Primarily, these are the values given as parameters on the TestNG Suite XML file. While Qadenz calls these values during the setup and reporting phases they are made available on the chance these values are needed for any project-specific configuration.

Since these fields are all static, there really is no need to extend this class. Teams are encouraged to follow this pattern, though, and create a ProjectConfig class of their own should the need arise to track specific values or project-specific Suite parameters are in play.

## WebDriverProvider

The [`WebDriverProvider`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebDriverProvider.java) stores the `WebDriver` instance and makes it available to components where it is needed. Since the test classes work on an inheritance hierarchy, and with how TestNG manages threads, the `WebDriver` instance is required to be kept on a `ThreadLocal<WebDriver>` object. Rather than forcing testers to pass the `WebDriver` around to various library components, Qadenz has chosen to make the `ThreadLocal` static, and allow components that need the `WebDriver` to call a centralized location. On the (hopefully) rare occurrence where the `WebDriver` is needed directly, it can be invoked with the following call.

```
WebDriver webDriver = WebDriverProvider.getWebDriver();
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
