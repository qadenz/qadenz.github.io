---
title: "UI Modeling"
weight: 7
description: >
    Modeling the UI Under Test
---

Qadenz is a departure from the commonly practiced use of the Selenium Page Factory pattern when implementing the Page Object Model. The `@FindBy` annotation-based approach to mapping UI elements lacks the ability to parameterize selectors. Instead, testers are forced to create repetitive annotated `WebElement` instances for every needed permutation of a selector. Without adding additional calls to `WebDriverWait` in order to use an Explicit Wait to initialize the element (and dealing with the resulting added clutter), these selectors are notoriously susceptible to the dreaded `StaleElementReferenceException`.

### On-the-fly Element Initialization

Using `PageFactory.initElements()` will instantiate the page class, and all `WebElement` instances annotated with `@FindBy` will be initialized. If the page on the UI is static and all mapped elements are already present, there should be no issues. Modern web-apps, though, use very dynamic interfaces. This frequently results in situations where some `@FindBy` mapped elements will change state at some point between the time `.initElements()` is called and when the element is actually used. To mitigate these issues, additional code in the form of Explicit Waits or other custom logic must be written to handle each element.

With Qadenz, Locators are defined and passed to commands methods, which in turn initialize a `WebElement` prior to acting upon the element. Further, each command will initialize the target element each time the command is called. By initializing WebElements *at the time they are used*, this mitigates situations that might produce a `StaleElementReferenceException`, and when errors are encountered, the problem can be presented far more clearly.

## Page Objects

With Qadenz placing focus on the `Locator` component, this leaves the design of any Page Object implementation is left as open ended as possible. This is done to allow any given team to design and implement an UI Model layer that fits their needs and the modeling needs of their UI under test.

Qadenz does not force a particular means of initializing Page Objects like the `PageFactory`. In fact, unless a more sophisticated implementation is needed by a team, Locator-based UI Models can be initialized by simply calling a constructor.

### UI Models as Locator Maps

A team could choose simply to create Page classes that only declare and return `Locator` instances, then instantiate and call the Commands classes directly within their tests. The resulting tests would be slightly more verbose, but completely self-contained, requiring no stepping through code calls to follow the logic path. This does not allow for much abstraction of common test steps, but for simple and small test projects, it enables quick development.

### UI Models with Simple or Complex Actions

Alternately, a team could choose to create Page classes that both store `Locators` as well as instances of Commands classes, then expose individual methods that perform singular actions against mapped elements. The Page classes would be instantiated somewhere accessible to the test, and the resulting code calls in the test would be clean and concise. With descriptive method names on the Page classes, minimal stepping through code would be required in order to infer the logic path.

Should the tests be more complex, or the UI workflows containing repetitive tasks, a team could build upon the above design and add more complex procedure methods that call multiple individual actions. This lessens the code required directly in a test, but with descriptive method names the resulting code calls would remain clean and concise. More reusable code is created, but would require a moderate amount of stepping through code calls in order to follow the details of the logic path.

A secondary benefit of this and similar designs is the logger injection for the Commands classes. `WebCommander` and `WebInspector` both use overloaded constructors to allow users to choose either a default logger or custom logger. A default logger will result in logging and reporting output that shows events originating from the Commands classes. Using a custom logger, teams can pass the `.class` value of the Page class to the `WebCommander` and `WebInspector` instances that reside on the Page class. This results in logging and reporting output that shows events originating from the Page classes, adding better clarity to the reporting output. This is explained in more detail in the [Commands](#commands) section.

Finally, if the Page classes are becoming too cumbersome due to a complex UI, the Locators could be separated into a Map class, allowing for some division in the UI Modeling components. A test would call a Page class in order to perform actions against elements, and the Page class would call the Map class in order to retrieve the element mappings.
