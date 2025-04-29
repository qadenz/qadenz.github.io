---
title: "Executing Tests"
weight: 3
description: >
    Launching test suites with Qadenz is designed to be as flexible as possible so as to integrate easily with the tooling and processes in use by the testing team.
---

## Execution Requirements

There are two requirements for running test suites with Qadenz.

First, a working Selenium Grid must be running. The Grid can be configured either remotely, or in Standlone Mode on a local machine. In the case of a remote Grid, at least one properly configured Node must be active and conencted to the Hub. In the case of a local machine running Selenium Grid in Standalone Mode, the OS must be properly configured with at least one browser driver.

Second, is the use of a TestNG Suite XML file. One XML file with one `<suite>` node must be provided in order to configure the Suite, and define the scope of included tests.

## TestNG XML Parameters

Suite configuration in Qadenz is made via parameters on the TestNG Suite XML file.

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

## Maven Surefire Plugin

Any means of invoking the TestNG XML file that is valid for TestNG should work without issue for a Qadenz-based project. Depending on the approach, however, some additional coding may be required. One simple method that is highly recommended is using the [Maven Surefire](https://maven.apache.org/surefire/maven-surefire-plugin/) plugin. This allows a Suite XML file to be stored within the project itself and invoked as part of the Maven Test command.

Start by adding the Surefire plugin to the project's `pom.xml`.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M4</version>
    <configuration>
        <suiteXmlFiles>
            <suiteXmlFile>TestNGSuite.xml</suiteXmlFile>
        </suiteXmlFiles>
    </configuration>
</plugin>
```

The example above assumes a TestNG Suite XML file called `TestNGSuite.xml` is located at the project root. Storing TestNG XML files in a subdirectory is also possible, but would require a reference to the directory in the `<suiteXmlFile>` node. Additionally, if multiple TestNG XML files are stored, it is possible to parameterize the file specification so that a command-line argument can be passed to Maven to specify which file to use for a given execution run.

```
<suiteXmlFiles>
    <suiteXmlFile>xml-suites/${suiteFile}</suiteXmlFile>
</suiteXmlFiles>
```

This example shows a folder at the project root called `xml-suites` which stores multiple suite files. The `${suiteFile}` property then becomes a command-line argument, allowing the TestNG XML file to be specified. It is worth noting that the above example may be interpreted as an error in some IDEs such as IntelliJ IDEA. This will not impact the ability to compile the project, nor will it impact test execution. This is an issue where the `pom.xml` does not have a value defined for the declared property. As long as the value is declared on the command line, the test run will launch.

## Command Line Execution

To run a TestNG XML file called `acme-tests.xml`, the following command would be used.

```
mvn clean test -DsuiteFile="acme-tests.xml"
```

This approach can be used on the local machine, or defined in a CI/CD tool like Jenkins or TeamCity as the build command.

Further information on how the Surefire plugin works and how it can be used for parameterizing and launching test suites can be found at the [Surefire](https://maven.apache.org/surefire/maven-surefire-plugin/) website. Additional information for integrating Maven builds into a CI/CD tool can be found in the documentation of the tool in use.

## Launching a Suite XML in the IDE

For local execution runs and debugging during the development phase, IDEs like Intellij IDEA support instant run of TestNG Suite XML files. With the TestNG plugin installed and enabled, simply right click a valid TestNG XML file, and click `Run`. If results are not generated in the `test-output` folder in the project root, go to the Run Configurations and (using IntelliJ IDEA as an example), open the configuration for the selected file, go to the `Listeners` tab, and enable the `Use default reporters` checkbox.
