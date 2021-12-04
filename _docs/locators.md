---
title: Locators
tags: 
description: Element location
--- 

# UI Modeling with Qadenz

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

By requiring a `name` value to be given in the `Locator` constructor, Qadenz is able to refer to the context-friendly name of an element as the primary identifier in all logging and reporting output. This eliminates the time needed to perform any lookup or crossreferencing of selectors to elements in relation to test steps. By reviewing the default logging output or report content, the point at which a problem appears in a test is clearly marked and quickly identified.

```
public Locator signInButton() {
    return new Locator("Sign In Button", ".btn-signIn");
}
```

### Parameterized Selectors

Using list of search results as an example, testers would be forced to create a seperate `@FindBy` annotated `WebElement` instance for each result element needed for a test. Alternately, a `@FindBy` could be used to initialize a single `List<WebElement>`, but additional logic would be necessary to identify a specific result element needed for a test step. In either case, the result is many extra lines of code. 

Using a parameterized Locator (coupled with the benefits of Sizzle CSS Selectors), testers will be able to define a single `Locator` instance for a generic search result, and rely upon the parameterization to direct the test to choose the appropriate element instance.

```
public Locator searchResultLink(String name) {
    return new Locator(name + " Search Result Link", ".search-result:contains(" + name + ")");
}
```

### On-the-fly Element Initialization

