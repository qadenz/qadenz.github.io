---
title: "Writing Tests"
weight: 2
description: >
    Writing Tests with Qadenz
---

Adding new tests to a Qadenz-powered project is as simple as extending the `AutomatedWebTest` class, and adding `@Test` annotated methods. This inheritance is required in order to utilize the Qadenz suite configuration features.

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

## Custom Configurations

If project-specific configurations need to be performed during the setup or tear-down phases of the execution cycle, an intermediate class may be inserted into the inheritance hierarchy. For example, a project may require that certain data items be in place as preconditions for tests, or other custom testing components would have to be started. These tasks would be well suited to be kept on a class that extends `AutomatedWebTest`, and is in turn extended by classes that hold `@Test` methods.

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

## Integrating with TestNG

AutomatedWebTest uses standard [TestNG Annotations](https://testng.org/doc/documentation-main.html#annotations) for configuration steps. As such, any intermediate configuration class that performs custom setup and tear-down tasks can utilize the same annotations to integrate seamlessly into the TestNG suite workflow.

Accessibility to TestNG features is generally open-ended with Qadenz. This was done intentionally so as to minimize restrictions on how users could configure their suites. Data handling features such as DataProviders and Parameters are usable, as are other features like Test Factories and disabling tests via the `@Ignore` annotation.

Parallelism can be set as desired, but do keep in mind that Qadenz executes WebDriver startup and shutdown in the `@BeforeMethod` and `@AfterMethod`. As such, for most projects, specifying `parallel="methods"` on the TestNG Suite XML file is recommended. Deviating from this approach should only be done if there is a specific need to do so within the design of the test project.

Qadenz does make use of dependencies within the scope of configuration methods to ensure that methods that take place at the same point in the workflow (for example, there are two `@BeforeSuite` methods) are executed in a certain order. This pattern can be repeated as needed should custom configurations need also to be performed in order. Given the focus on parallel execution with Qadenz, however, it is not recommended that `@Test` methods utilize any dependencies among one another.
