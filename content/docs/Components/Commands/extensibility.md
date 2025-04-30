---
title: Extensibility
description: >
  Web Inspector Description
weight: 4
---

Both the `WebCommander` and `WebInspector` classes are designed to be extended within automation projects to enable custom commands to be created as needed by the UI under test.

Starting at the class-level, the class will obviously need to extend either `WebCommander` or `WebInspector`. Then, a `Logger` instance and constructors need to be added.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AcmeWebCommander extends WebCommander {

    private Logger LOG;

    public AcmeWebCommander() {
        super();
        LOG = LoggerFactory.getLogger(AcmeWebCommander.class);
    }

    public AcmeWebCommander(Class<?> logger) {
        super(logger);
        LOG = LoggerFactory.getLogger(logger);
    }
}
```

Then, the anatomy of the command method is as follows:

```java

public void doSomething(Locator locator) {
    LOG.info("Doing something with element [{}].", locator.getName());
    try {
        WebElement webElement = webFinder.findWhenVisible(locator);

        // add logic as needed to complete the custom command
    }
    catch (Exception exception) {
        LOG.error("Error doing something :: {}: {}", exception.getClass().getSimpleName(), exception.getMessage());
        screenshot.capture();

        throw exception;
    }
}
```

The first operation in a command method is to log the command being executed (at the `INFO` level). Qadenz uses Logback, which provides the `{}` placeholder for additional values to be inserted. It is recommended to utilize this where possible to convey an appropriate level of detail in the logs and subsequent reports.

Next, the `try` block will initialize a `WebElement` using the inherited `WebFinder` instance, then perform any necessary actions. If the method is on a `WebInspector` sub-class, the `return` should take place within the `try` block.

The `catch` block should be set to catch the appropriate Exception, though a wide net may be cast by simply catching all descendents of `Expception`. A multi-catch or `finally` could be employed here if the use case is appropriate. Within the `catch` block, the Exception will be logged (at the `ERROR` level) and, if appropriate, a screenshot captured using the inherited `Screenshot` instance.

Finally, re-throw the Exception. Throwing a new instance of `RunTimeException` with a context-friendly message would be an appropriate alternative to allowing the Exception to surface to the test level, requiring a `try/catch` in the test itself or an Exception to be added to the method signature. This is obviously a preferential decision to be made within the automation project. Qadenz has simply been designed to avoid the need to do this.

Once complete, instantiating the new commands class in either the UI Models or directly in the test method will make functionality available for use.
