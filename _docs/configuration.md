---
title: Configuration
tags: 
description: Setup and teardown of a Suite
--- 

## **Configuring Qadenz**

## TestNG Suite XML Parameters

Primary suite configuration in Qadenz is made via parameters on the TestNG Suite XML file.

- `gridHost`: Directs the tests to a Selenium Grid Hub. The expected value is simply the IP address of the Hub. The default port number (4444) is pre-configured.
- `browser`: Specifies which browser will run the test.
- `browserVersion`: An optional parameter that allows a specific version of the browser to be used for the test.
- `browserConfigProfile`: Provides arguments for the browser configuration. This is optional. If the parameter is not declared, or if the parameter value does not match an existing profile name, no arguments will be passed to the `WebDriver` for the execution cycle.
- `platform`: Directs the tests to a Node running a given OS. This parameter is optional.
- `timeout`: Sets the max allowable time for the Explicit Waits to run in Qadenz. This is an optional parameter, expressed in seconds. If this parameter is not given, a default of 30 seconds will be used.
- `appUrl`: The full URL of the application under test.
- `retryInterceptedClicks`: This works with the `click()` command. If a click throws an `ElementClickInterceptedException`, enabling this parameter will direct the `WebDriver` to retry the click using the `Actions` API. Do note, however, that this exception is typically a symptom of a selector issue. This parameter is optional, and defaults to enabled.

The `gridHost` is a Suite-level parameter, and is assigned only once during execution prior to any tests executing. The others are all Test-level parameters, and are assigned before each `<test>` node on the TestNG Suite XML file.

The Test-level parameters allow for configuration changes to take place during the execution cycle. For example, if the goal of the suite is to run the same set of tests in multiple browsers, rather than create 3 different Suite XML files, 3 `<test>` nodes can be added to the same Suite XML file with different configurations.

In the below example, both `<test>` nodes will use the same set of parameter values:

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    <parameter name="browser" value="chrome"/>
    
    <test name="Authentication Tests">
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Permissions Tests">
        <classes>
            <class name="com.acme.test.cases.PermissionsTest"/>        
        </classes>
    </test>
</suite>
```

Parameters on TestNG Suite XML files obey scoping rules. That is to say, a parameter declared within a `<test>` will take precedence over the same parameter as declared on the `<suite>`. 

In this next example, the second `<test>` will override the `browser` parameter, while the others will use follow what is declared at the `<suite>`.

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    <parameter name="browser" value="chrome"/>
    
    <test name="Authentication Tests">
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Permissions Tests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.acme.test.cases.PermissionsTest"/>        
        </classes>
    </test>

    <test name="Account Management Tests">
        <classes>
            <class name="com.acme.test.cases.AccountManagementTest"/>        
        </classes>
    </test>
</suite>
```

In another example, to run the same set of tests in three different browsers as part of the same execution, Test-level parameters can be used accordingly:

```
<suite name="Automated Tests" parallel="methods" thread-count="6">
    
    <parameter name="gridHost" value="10.1.10.10"/>
    <parameter name="appUrl" value="http://qa.acmecorp.com"/>
    
    <test name="Authentication Tests">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
    
    <test name="Authentication Tests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>

    <test name="Authentication Tests">
        <parameter name="browser" value="edge"/>
        <classes>
            <class name="com.acme.test.cases.AuthenticationTest"/>        
        </classes>
    </test>
</suite>
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
