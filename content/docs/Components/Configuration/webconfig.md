---
title: "WebConfig"
linkTitle: "WebConfig"
description: >
  The run's shared state: the validated parameter values and run timestamps that the rest of the framework reads from.
weight: 3
---

[`WebConfig`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebConfig.java) holds the state of a run. It is a single class of static fields where the validated Suite parameters land during setup, alongside a pair of timestamps that mark the run's boundaries. [`AutomatedWebTest`]({{< relref "automatedwebtest.md" >}}) writes to it as the execution cycle begins, and any component that needs a configuration value reads it back from here.

## What it holds

Most of the fields are the validated Suite parameters. Qadenz reads them from the TestNG XML before each `<test>`, validates them, and stores the typed result here.

| Field | Type | Value |
| --- | --- | --- |
| `gridHost` | `String` | Host of the Selenium Grid. |
| `browser` | `Browser` | The browser for the session. |
| `browserVersion` | `String` | Requested browser version, or `null` for any available. |
| `browserConfigProfile` | `String` | Named profile of browser arguments, or `null`. |
| `platform` | `Platform` | Requested operating system, or `null` for any available. |
| `timeout` | `int` | The explicit-wait limit, in seconds. |
| `appUrl` | `String` | The URL loaded at the start of each test. |
| `retryInterceptedClicks` | `boolean` | Whether `click()` retries an intercepted click. |

For what each parameter means and what the optional ones default to, see [Suite Parameters]({{< relref "suite-parameters.md" >}}).

Two more fields record the run rather than configure it. `suiteStartDate` and `suiteEndDate` are `LocalDateTime` stamps captured at the start and end of the suite by `AutomatedWebTest`'s parent class. The reporter reads the pair to calculate the run's duration.

## A shared, static read surface

Every field on `WebConfig` is static, and that is deliberate. A component that needs a configuration value reaches it through the class directly, with no `WebConfig` instance to construct or thread through a call. The value is set once during setup and stays put for the length of the run.

The framework reads it in exactly this way. The command and element-finding layers size every built-in wait from `WebConfig.timeout`, the `click()` command consults `WebConfig.retryInterceptedClicks` before it retries, and the reporter measures the run against the two timestamps. A project can read the same values wherever a helper or a custom command needs them.

```java
int timeout = WebConfig.timeout;
String appUrl = WebConfig.appUrl;
```

## Project-specific configuration

Because the fields are static, extending `WebConfig` gains nothing, and it is not meant to be extended. When a project has values of its own to carry across a run, whether its own Suite parameters or settings shared among tests, follow the same pattern with a small static holder of its own, a `ProjectConfig` kept alongside the tests and populated during setup. Qadenz keeps its configuration on `WebConfig`, and a project keeps its own beside it.