---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
description: >
    From an empty Maven project to a passing test and its report.
---

Standing up a Qadenz project takes three things: a new Java/Maven project, the Qadenz dependency, and a test. This guide covers the prerequisites and then builds a first test that signs into an authentication form and verifies the result.

## Prerequisites

Qadenz is a Java library, so a **JDK 11 or newer** is required. Any IDE works, and **IntelliJ IDEA** is the common choice, with its free Community Edition enough for automation work. IntelliJ bundles Maven, which is functional, though installing a standalone **Maven** and pointing the IDE at it avoids the occasional difference in behavior. IDE support for **TestNG** and **Maven** makes the tooling easier to work with.

Qadenz drives browsers through a `RemoteWebDriver`, so a running **[Selenium Server](https://www.selenium.dev/downloads/)** is required, along with the target browser and its driver. This guide uses **Chrome** and **Chromedriver**, though any [supported browser]({{< relref "/docs/Components/Configuration/suite-parameters.md" >}}) works.

A package manager keeps these tools current: **Chocolatey** on Windows, **Homebrew** on macOS, and the native package manager on Linux.

## Create the project

Start a new Maven project in the IDE, open its `pom.xml`, and add the Qadenz dependency.

```xml
<dependency>
    <groupId>dev.qadenz</groupId>
    <artifactId>qadenz</artifactId>
    <version>2.1.10</version>
</dependency>
```

Create a package and a class to hold the first test. The class must extend [`AutomatedWebTest`]({{< relref "/docs/Components/Configuration/automatedwebtest.md" >}}), which wires it into the Qadenz execution cycle: a fresh browser per test, the configured Suite parameters, and the reporter.

```java
package com.herokuapp.theinternet;

public class FormAuthenticationTest extends AutomatedWebTest {

}
```

## Map the elements

The test exercises the [Form Authentication](https://the-internet.herokuapp.com/login) page: enter a username and password, click Login, and verify the confirmation banner in the secure area.

Each element the test touches is mapped with a [`Locator`]({{< relref "/docs/Components/ui-modeling/_index.md" >}}), which pairs a readable name with a CSS selector. Map the username and password fields and the login button from the sign-in form...

![The Form Authentication sign-in page](/img/demo-login-page.png)

...then map the banner from the secure area so the test can read its message.

![The secure area confirmation banner](/img/demo-login-secure.png)

Declare the Locators on the `FormAuthenticationTest` class.

```java
private Locator usernameField = new Locator("Username Field", "#username");
private Locator passwordField = new Locator("Password Field", "#password");
private Locator loginButton = new Locator("Login Button", "#login .fa-sign-in");

private Locator bannerMessage = new Locator("Banner Message", "#flash");
```

{{% alert title="" color="primary" %}}
A real project models the UI in page objects rather than declaring Locators on the test class. This demo keeps everything on one class to stay focused on the mechanics. See [UI Modeling]({{< relref "/docs/Components/ui-modeling/_index.md" >}}) for the pattern.
{{% /alert %}}

## Write the test

Add the test method. A [`WebCommander`]({{< relref "/docs/Components/Commands/_index.md" >}}) drives the form, and a [Condition paired with an Expectation]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}) verifies the banner text.

```java
@Test
public void testAuthenticationSucceedsWithValidCredentials() {
    WebCommander commander = new WebCommander();

    commander.enterText(usernameField, "tomsmith");
    commander.enterText(passwordField, "SuperSecretPassword!");
    commander.click(loginButton);
    commander.verify(Conditions.directTextOfElement(bannerMessage,
            Expectations.isEqualTo("You logged into a secure area!")));
}
```

Each command waits for its element, logs the step, and captures a screenshot if it fails. The `verify` call is a hard assertion: if the banner text does not match, the test fails at that line.

Add a TestNG Suite XML file at the project root. Its [parameters]({{< relref "/docs/Components/Configuration/suite-parameters.md" >}}) tell Qadenz which browser to run, the URL of the application, and where to reach the Selenium Server.

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >

<suite name="Qadenz Quickstart" verbose="1" thread-count="1">

    <parameter name="browser" value="chrome"/>
    <parameter name="appUrl" value="https://the-internet.herokuapp.com/login"/>
    <parameter name="gridHost" value="127.0.0.1"/>

    <test name="Login Test">
        <classes>
            <class name="com.herokuapp.theinternet.FormAuthenticationTest">
                <methods>
                    <include name="testAuthenticationSucceedsWithValidCredentials"/>
                </methods>
            </class>
        </classes>
    </test>

</suite>
```

## Run it

Start a Selenium Server before launching the test. Running locally, launch it in standalone mode: open a terminal, change to the directory holding the Selenium Server jar, and run it. The filename matches the version downloaded.

```shell
java -jar selenium-server-4.45.0.jar standalone
```

Back in the IDE, right-click the Suite XML file and select `Run`. TestNG launches the suite, a Chrome window opens, and the test runs. When it finishes, the console logs and the HTML report are ready to review.

### Console logs

The console shows each step as it happens, one line per event.

```console
21:06:57:620 | Thread: main | INFO | WebCommander | Entering text [tomsmith] into element [Username Field].
21:06:57:798 | Thread: main | INFO | WebCommander | Entering text [SuperSecretPassword!] into element [Password Field].
21:06:57:893 | Thread: main | INFO | WebCommander | Clicking element [Login Button].
21:06:58:149 | Thread: main | INFO | Commands | Verifying Condition - Direct text of element [Banner Message] is equal to [You logged into a secure area!].
21:06:58:274 | Thread: main | INFO | Commands | Result - PASS
```

Each line carries a timestamp, the thread, the logging level, the logger name, and the message. See [Test Results]({{< relref "/docs/Components/Test-Results/console-logs.md" >}}) for how the console relates to the report.

### HTML report

Qadenz writes a self-contained HTML report, `suite-results.html`, to the `test-output` directory alongside the standard TestNG output. It carries the full log of every test, sorted by result and grouped by class, with a screenshot embedded at each failure.

![The Qadenz HTML report](/img/demo-html-report.png)

See [Test Results]({{< relref "/docs/Components/Test-Results/_index.md" >}}) for the full report and the JSON export that accompanies it.
