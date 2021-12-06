---
title: Qadenz Documentation
description: Working with the Qadenz Automation Library
--- 

# Philosophy of Use

Very seldom will teams forge forward with writing tests using raw Selenium, joined with a unit testing framework and perhaps a data library (JDBC, Apache POI, etc). They might even have a basic Page Object model in place to abstract repeated code. At best, and most rarely, this type of team is typically running like an extremely well oiled machine with highly disciplined developers writing detailed code with a consistent implementation. Usually, this type of team may be in a Wild West environment where anything goes as far as design, and little attention is paid to things like maintenance. More than likely, a this type of team may very well be plowing full steam toward a brick wall, and may not realize it.

The concept of a wrapper-layer to unify the implementation of underlying tools within a test automation environment is nothing new. Fortunately, most teams come to realize that frameworks like Selenium, TestNG or even JUnit, are indeed open ended frameworks designed as such very intentionally to allow it's users to tailor their implementation of these tools to their own needs. 

This commonly leads many teams to design even a basic wrapper-layer for the sake of convenience alone, simply to reduce the coding effort. Some will take the time to answer the question "What is the best way for **our team** to implement these tools?" It's not uncommon for teams to build out a custom toolset at this layer and over time see a varied degree of success. It's certainly not uncommon for consulting companies and managed services vendors to offer their own "framework" to their clients as either a value-add or a means of retaining clients for a longer term via maintenance contracts, again with varied degrees of success on the long term.

At the end of the day, it becomes about development efficiency and code maintainability. How quickly can we automate this next block of tests? Is the tooling reliable enough to ensure repeatable executions? Are the implementation details tribal knowledge, or is the code clear enough that a new developer can easily and quickly ramp up for maintenance? If the answers to these questions leads to increased efficiency and improved maintainability, then the investment in these tools becomes quite worthwhile.

## Objectives

To these ends, Qadenz targets a number of core objectives...

### Decouple Tests from Configuration

By separating the actual tests from the setup and tear-down, tests will receive the appropriate focus. The configuration can then be abstracted and maintained from a central location. Tests are more easily organized, and are not obfuscated by boilerplate.

### Reduce coding overhead by doing the heavy lifting

Selenium interactions can, with one method call, log the activity, initialize an element, perform an interaction against the element, handle exceptions with logging and reporting, and capture screenshots. By offloading this work to the library, users can focus on scripting their tests rather than sorting out all the supporting functionality. 

### Encourage efficient UI Modeling

Qadenz uses CSS Selectors exclusively due to maximized flexibility and easy of use. This is further enhanced by injecting SizzleJS to enable parameterized element selectors. CSS Selectors are versatile, simple to understand, and most importantly (well crafted selectors are) more resistant to change compared to XPath. Wrapping the element selectors on the Locator object also enhances reporting with friendly element names, and encourages teams to move beyond antiquated and inefficient patterns. Page Factory is old and busted. Simon even says so.

### Provide flexible and precise validations

The Conditions-based validations functionality is custom to Qadenz and enables high levels of precision using the What/How relationship of Conditions and Expectations. Additionally, the Verify/Check language in the API allows for hard and soft assertions, which enables the user to decide exactly when a test will be halted following a failure. Finally, the iterative nature of Qadenz validations allows for multiple assertions to be made in groups, ensuring all validations are made regardless of those that fail.

### Include detailed and easily shared reports

TestNG provides basic reporting out of the the box. The `emailable-report.html` is nice for basic pass/fail results, but leaves much to be desired when it comes to detail. Other reporting libraries add a lot of noise to the code, and don't tend to coexist with standard logging frameworks. The Qadenz reporter is powered by Logback, is fully integrated with TestNG, and produces a singular HTML output file that contains full detailed results of each test and embedded screenshots of errors and failed validations.

## Opinionation

Qadenz tries to keep a balance between opinionated design, and remaining flexible for user implementation.

# UI Modeling

Qadenz is a departure from the commonly practiced use of the Selenium Page Factory pattern when implementing the Page Object Model. The `@FindBy` annotation-based approach to mapping UI elements lacks the ability to parameterize selectors. Instead, testers are forced to create repetitive annotated `WebElement` instances for every needed permutation of a selector. Without adding additional calls to `WebDriverWait` in order to use an Explicit Wait to initialize the element (and dealing with the resulting added clutter), these selectors are notoriously susceptible to the dreaded `StaleElementReferenceException`.

In short, Qadenz is quite happy to follow Simon Stewart's [advice](https://www.youtube.com/watch?v=gyfUpOysIF8&t=1519s), and find a better way.

The UI Modeling strategy behind Qadenz centers around the [`Locator`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/Locator.java) object, and rather than mix between all the different selector types, Qadenz has chosen CSS selectors exclusively.

# Why only CSS Selectors?

Many teams will choose either a default standard selector strategy (ID, CSS, XPath, etc.), or at least prioritize one over others. Qadenz has chosen CSS selectors for the superior performance, ease of use, and better browser compatibility when compared to XPath. The addition of the [SizzleJS](https://github.com/jquery/sizzle) library provides a number of pseudo-classes that make element selection more flexible and accurate.

The end result is a powerful and straightforward selector strategy that supports full parameterization capabilities. This also leads to a singular defined approach that an entire team can adopt and consistently apply across the entire project.

# Enter the Locator

The [`Locator`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/Locator.java) is the primary component of UI Modeling in the Qadenz environment. This object simply carries the name of an element, the element's CSS Selectorm, and an optional parent Locator instance to assist with abstracting element relationships and re-using selector content. The `Locator` is passed to methods on classes such as the [`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) and [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java), which will then use the data within to initialize WebElements with the [`WebFinder`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/WebFinder.java).

## Locator vs @FindBy

The `Locator` provides several major advantages over the `@FindBy` and `PageFactory`. 

### Context-friendly Element Names

Selenium does not readily provide a meaningful friendly reference to element names, which can complicate debugging and troubleshooting when problems arise. Without additional logging in the test project, testers are generally only provided references to elements expressed as the given selectors. This has a very strong potential to slow down the resolution process as testers must first translate the selector in their stack trace to an actual element on the UI, which can then establish a point of orientation within the progression of test steps.

By requiring a `name` value to be given in the `Locator` constructor, Qadenz refers to the context-friendly name of an element as the primary identifier in all logging and reporting output. This eliminates the time needed to perform any lookup or cross-referencing of selectors to elements in relation to test steps. By reviewing the default logging output or report content, the point at which a problem appears in a test is clearly marked and quickly identified.

```JAVA
Locator signInButton = new Locator("Sign In Button", ".btn-signIn");
```

### Parameterized Selectors

Using list of search results as an example, testers would be forced to create a seperate `@FindBy` annotated `WebElement` instance for each result element needed for a test. Alternately, a `@FindBy` could be used to initialize a single `List<WebElement>`, but additional logic would be necessary to identify a specific result element needed for a test step. In either case, the result is many extra lines of code. 

Using a parameterized Locator (coupled with the benefits of Sizzle CSS Selectors), testers will be able to define a single `Locator` instance for a generic search result, and rely upon the parameterization to direct the test to choose the appropriate element instance.

```JAVA
public Locator searchResultLink(String name) {
    return new Locator(name + " Search Result Link", ".search-result:contains(" + name + ")");
}
```

### On-the-fly Element Initialization

Using `PageFactory.initElements()` will instantiate the page class, and all `WebElement` instances annotated with `@FindBy` will be initialized. If the page on the UI is static and all mapped elements are already present, there should be no issues. Modern webapps, though, use very dynamic interfaces. This frequently results in situations where some `@FindBy` mapped elements will change state at some point between the time `.initElements()` is called and when the element is actually used. To mitigate these issues, additional code in the form of Explicit Waits or other custom logic must be written to handle each element. 

With Qadenz, Locators are defined and passed to commands methods, which in turn initialize a `WebElement` prior to acting upon the element. Further, each command will initialize the target element each time the command is called. By initializing WebElements *at the time they are used*, this mitigates situations that might produce a `StaleElementReferenceException`, and when errors are encountered, the problem can be presented far more clearly.

# Using Locators

## Standard Locators

A `Locator` holds two required values, `name` and `selector`. The `name` field is completely user-definable, but is recommended to at least convey some level of meaningful context to the element being mapped. This value will appear in the logs, on the final report output, as well as within any error handling. 

## Parent Locators

The `Locator` can hold a third, optional value of another `Locator` instance. This is intended to allow abstraction of selector segments by combining the selector of the parent Locator with that of the current Locator as a single selector value. This can be helpful in reducing repeated selector segments when creating Locators for closely related UI Elements. 

Consider an e-commerce application wherein a list of catalog items are presented on the UI. Each item card contains the item name text, a 'Cost' value, a 'Quantity' field, and an 'Add to Cart' button. 

As a very simple HTML representation:

```HTML
<div id="item-list-section" class="grid">
    <div class="item-row even">
        <div class="item-card">
            <div class="item-name">ACME Rocket Powered Roller Skates</div>
            <div class="item-cost">$99.99</div>
            <div class="item-qty input-field">
                <input type="text"></div>
            <div class="item-add action-button">
                <button type="submit">Add to Cart</button></div>
        </div>
    </div>
<div>
```

The selector for the 'Add to Cart' button could be:

```
#item-list-section .item-card:contains(ACME Rocket Powered Roller Skates) .item-add button
```

While mapping the other elements on an item card, however, it would be discovered that the item card selector itself is repeated on each of the child elements. In this situation, a parent `Locator` could be created to abstract the repeated selector segments, especially if the abstracted selector can stand as it's own element mapping.

With a parent `Locator` the resulting element mappings for the item card, 'Quantity' field, and 'Add to Cart' button could be:

```JAVA
public Locator itemCard(String itemName) {
    return new Locator(itemName + " Item Card", "#item-list-section .item-card:contains(" + itemName + ")");
}
public Locator itemQuantityField(String itemName) {
    return new Locator(itemName + " Quantity Field", itemCard(itemName), " .item-qty input");
}
public Locator itemAddToCartButton(String itemName) {
    return new Locator(itemName + " Add to Cart Button", itemCard(itemName), " .item-add button");
}
```

In the above example, the `itemCard()` Locator could have the added benefit of not only being an abstracted selector, but could also serve as the target element of a verification of a given item card element to be visible on the page.

Parent Locators are also not limited to a single layer. Locators can be passed as parent references as many times as needed.

# LocatorGroup

The [`LocatorGroup`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/LocatorGroup.java) allows multiple `Locator` instances to be combined together on a `List`, and acted upon as a group. This is commonly applied to verify the visibility of UI component, or a set of default UI elements. Instead of passing multiple individual Conditions to a `.verify()` or `.check()` validation, a `LocatorGroup` can be verified with a single `Condition` call.

For example, a simple authentication form has several basic elements, the 'Username' field, the 'Password' field, a 'Remember Me' checkbox, and a 'Sign In' button. Each element would be mapped as an individual `Locator` instance for the purposes of input, but these same elements could also be included in a `LocatorGroup`, should the need arise to verify each element to be visible as part of the form.

```JAVA
Locator usernameField = new Locator("Username Field", "#username");
Locator passwordField = new Locator("Password Field", "#password");
Locator rememberMeCheckbox = new Locator("Remember Me Checkbox", "#remember-me");
Locator signInButton = new Locator("Sign In Button", "#sign-in");

LocatorGroup signInForm = new LocatorGroup("Sign In Form", usernameField, passwordField, 
                                           rememberMeCheckbox, signInButton);
```

By combining individual Locators onto a `LocatorGroup` instance, testers are able to identify and refer to collections of elements as the UI Components they represent.
