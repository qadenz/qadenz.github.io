---
title: Configuration Overview
tags: 
description: Understanding the default suite configuration of Qadenz.
--- 

# Configuration Components

These objects, from the `dev.qadenz.automation.config` package, handle Suite and Test configuration tasks.

## AutomatedWebTest

The `AutomatedWebTest` is the base class for all test classes in a Qadenz-powered test project. This class is responsible for gathering parameter values from the TestNG Suite XML file, configuring and starting a `WebDriver` instance on a Selenium Grid for each test method, stopping the `WebDriver` after each test, and invoking the reporters after the Suite has completed.

Classes that hold `@Test` methods must extend this class in order for tests to run using Qadenz configurations.

```
public class AuthenticationTest extends AutomatedWebTest {
    
    @Test
    public void verifySuccessfulAuthenticationWithValidCredentials() {
        // add test code
    }

    @Test
    public void verifyAuthenticationErrorWhenUsingInvalidCredentials() {
        // add test code
    }
}
```

Classes that hold `@Test` methods must inherit from this class in order for tests to run using Qadenz configurations. If project-specific configurations need to be performed during the setup or tear-down phases of the execution cycle, an intermediate class may be inserted into the inheritance hierarchy. For example, a project may require that certain data items be in place as preconditions for tests, or other custom testing components would have to be started. These tasks would be well suited to be kept on a class that extends `AutomatedWebTest`, and is in turn extended by classes that hold `@Test` methods.

```
public class AcmeAutomatedWebTest extends AutomatedWebTest {
    
    @BeforeSuite
    public void loadItemData() {
        // perform setup tasks
    }
}

public class ItemInventorySearchTest extends AcmeAutomatedWebTest {
    // add @Test methods
}
```

AutomatedWebTest uses standard [TestNG Annotations](https://testng.org/doc/documentation-main.html#annotations) for configuration steps. As such, any intermediate configuration class that performs custom setup and tear-down tasks can utilize the same annotations to integrate seamlessly into the TestNG suite workflow.

## WebConfig

`WebConfig` stores information that Qadenz uses at various points throughout the execution cycle. Primarily, these are the values given as parameters on the TestNG Suite XML file. While Qadenz calls these values during the setup and reporting phases they are made available on the chance these values are needed for any project-specific configuration.

Since these fields are all static, there really is no need to extend this class. Teams are encouraged to follow this pattern, though, and create a ProjectConfig class of their own should the need arise to track specific values or project-specific Suite parameters are in play.

## WebDriverProvider

The `WebDriverProvider` stores the `WebDriver` instance and makes it available to components where it is needed. Since the test classes work on an inheritance hierarchy, and with how TestNG manages threads, the `WebDriver` instance is required to be kept on a `ThreadLocal<WebDriver>` object. Rather than forcing testers to pass the `WebDriver` around to various library components, Qadenz has chosen to make the `ThreadLocal` static, and allow components that need the `WebDriver` to call a centralized location. On the (hopefully) rare occurrence where the `WebDriver` is needed directly, it can be invoked with the following call.

```
WebDriver webDriver = WebDriverProvider.getWebDriver();
```
