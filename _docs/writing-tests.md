---
title: Writing Tests
tags: 
description: Writing Tests with Qadenz
--- 

# Writing Tests

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

AutomatedWebTest uses standard [TestNG Annotations](https://testng.org/doc/documentation-main.html#annotations) for configuration steps. As such, any intermediate configuration class that performs custom setup and tear-down tasks can utilize the same annotations to integrate seamlessly into the TestNG suite workflow.



- TestNG functionality is available with things like Groups, Data Providers and Factory classes (examples)

- Test dependencies are not recommended for @Test methods.

- Remove any overlap from Execution Management. In fact, rename Execution Management completely.
