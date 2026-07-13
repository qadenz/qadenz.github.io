---
title: "Suite Parameters"
linkTitle: "Suite Parameters"
description: >
  The TestNG XML parameters that configure a Qadenz run: which are required, what the optional ones default to, and how their scope works.
weight: 2
---

Qadenz configures a run entirely through parameters on the TestNG Suite XML file. Before each `<test>` runs, [`AutomatedWebTest`]({{< relref "automatedwebtest.md" >}}) reads these values, validates them, and stores them on the [`WebConfig`]({{< relref "webconfig.md" >}}) for the rest of the framework to use. Nothing here is configured in code.

## The parameters

| Parameter | Required | Default | Effect |
| --- | --- | --- | --- |
| `gridHost` | Yes | none | Host or IP address of the Selenium Grid. Qadenz connects at `http://{gridHost}:4444/wd/hub`; the port `4444` and the `/wd/hub` path are fixed. |
| `browser` | Yes | none | The browser for the session. One of `Chrome`, `Edge`, or `Firefox`, matched without regard to case. |
| `appUrl` | Yes | none | Full URL of the application under test. Qadenz loads it at the start of each test. |
| `browserVersion` | No | any available | Pins a specific browser version. When omitted, the Grid provides any available version. |
| `platform` | No | any available | Directs the session to a node running a given operating system. Validated against Selenium's `Platform` values. |
| `timeout` | No | `30` | Maximum time, in seconds, that the built-in explicit waits run before failing a command. |
| `browserConfigProfile` | No | none | Names a [Browser Config Profile]({{< relref "browser-config-profiles.md" >}}) whose arguments are applied to the browser, such as running headless. |
| `retryInterceptedClicks` | No | `false` | When `true`, a `click()` that throws `ElementClickInterceptedException` is retried using the `Actions` API. |

### Required parameters

`gridHost`, `browser`, and `appUrl` have no defaults. A run that omits any of the three fails immediately with an `IllegalArgumentException` before any test executes. A `browser` or `platform` value that Qadenz does not recognize fails the same way, rather than falling back to a default.

### A note on retryInterceptedClicks

An `ElementClickInterceptedException` is usually a symptom, not the problem. It most often means a selector is resolving to an element that another node is covering, and the fix belongs in the selector or the wait, not in a retry. Leave `retryInterceptedClicks` at its default of `false` unless a specific, understood case calls for it.

## Scope and precedence

Parameters follow TestNG's standard scoping. A `<parameter>` declared on the `<suite>` applies to every `<test>` beneath it. The same parameter declared on a `<test>` overrides the suite value for that test alone. Because Qadenz reads the parameters before each `<test>`, a test-level override takes effect for exactly that test.

In practice, the values that hold steady across a run, `gridHost` and `appUrl`, sit on the `<suite>`, while anything that varies per test moves down to the `<test>` node.

```xml
<suite name="Regression" parallel="methods" thread-count="6">

    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="https://app.example.com"/>
    <parameter name="browser" value="chrome"/>

    <test name="Authentication">
        <classes>
            <class name="com.example.tests.AuthenticationTest"/>
        </classes>
    </test>

    <test name="Permissions">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.example.tests.PermissionsTest"/>
        </classes>
    </test>
</suite>
```

Here the Permissions test overrides `browser` to Firefox, while Authentication uses the Chrome value declared at the suite level.

The same mechanism runs one set of tests across several browsers from a single file. Leave `browser` off the `<suite>`, repeat the `<test>` with the same classes, and set a different `browser` on each.

```xml
<suite name="Cross-Browser" parallel="methods" thread-count="6">

    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="https://app.example.com"/>

    <test name="Authentication - Chrome">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.example.tests.AuthenticationTest"/>
        </classes>
    </test>

    <test name="Authentication - Firefox">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.example.tests.AuthenticationTest"/>
        </classes>
    </test>

    <test name="Authentication - Edge">
        <parameter name="browser" value="edge"/>
        <classes>
            <class name="com.example.tests.AuthenticationTest"/>
        </classes>
    </test>
</suite>
```