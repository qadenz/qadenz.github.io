---
title: Logging
description: >
  Web Inspector Description
weight: 3
---

The `WebCommander` and `WebInspector` can be instantiated and used either from the tests directly, or from the UI Modeling layer, depending on the design of the test project. Both `WebCommander` and `WebInspector` have overloaded constructors that enable different types of logging to take place, and will both directly impact how the logs are presented on the report output.

The `WebCommander` constructor is used as an example, but the `WebInspector` shares this same pattern:

```java
private Logger LOG;

public WebCommander() {
    super();
    LOG = LoggerFactory.getLogger(WebCommander.class);
}

public WebCommander(Class<?> logger) {
    super(logger);
    LOG = LoggerFactory.getLogger(logger);
}
```

The no-args constructor is generic and can be used regardless of how the classes are consumed. This constructor assigns the `WebCommander.class` as the `Logger`, and all logging output for method calls on this class will be shown to originate from the `WebCommander` class. If the UI of the application under test is very simple, or the team simply does not require a level of detail in the logging that ties actions to specific UI Models, then this constructor will provide an ideal configuration.

```
09:55:00.939 | INFO | WebCommander | Entering text [admin@qadenz.dev] into element [Username Field].
09:55:01.308 | INFO | WebCommander | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | WebCommander | Clicking element [Sign In Button].
09:55:02.583 | INFO | Commands | Verifying Condition - Visibility of element [Qadenz Logo Image] is TRUE.
09:55:02.998 | INFO | Commands | Result - PASS
```

The overloaded constructor requires a `Class<?>` argument, and allows for another class reference to be injected as the logger for the `WebCommander` instance. If `WebCommander` is being instantiated from a Page Object, and the Page Object class is passed to the constructor, the logs and reporting output will be shown to originate from the Page Object itself, resulting in a greater level of detail in the logs and reports. By using the class injection for the logger, commands will be logged in the context of the page where the command was executed.

```
09:55:00.939 | INFO | LoginPage | Entering text [admin@qadenz.dev] into element [Username Field].
09:55:01.308 | INFO | LoginPage | Entering text [Test123$] into element [Password Field].
09:55:01.472 | INFO | LoginPage | Clicking element [Sign In Button].
09:55:02.583 | INFO | UserProfilePage | Verifying Condition - Visibility of element [Company Logo Image] is TRUE.
09:55:02.998 | INFO | UserProfilePage | Result - PASS
```
