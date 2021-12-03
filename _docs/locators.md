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

Many teams will choose either a default standard selector strategy (ID, CSS, XPath, etc.), or at least prioritize one over others. Qadenz has chosen CSS selectors for the superior performance, ease of use, and better browser compatibility when compared to XPath. The addition of the SizzleJS library provides a number of pseudo-classes that make element selection more flexible and accurate.

The end result is a powerful and straightforward selector strategy that supports full parameterization capabilities. This also leads to a singular defined approach that an entire team can adopt and consistently apply across the entire project.

# Enter the Locator

The [`Locator`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/Locator.java) captures the information required to map and later initialize individual UI elements. It provides friendly names of these elements to the logs and reports in order to drive better clarity in the output. It is far easier to read a friendly description of an element on a report than attempt to interpret context from only a selector.

Locators do not initialize WebElements. Instead, the `Locator` is part of a series of related classes that handle UI interactions. The `Locator` defines the element, the [`WebFinder`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/ui/WebFinder.java) generates WebElements using Locators, and the [`WebCommander`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebCommander.java) and [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/io/qadenz/automation/commands/WebInspector.java) act upon these WebElements.

The example below is a single method returning a `Locator` instance that defines a "Sign In" button.

```
public Locator signInButton() {
    return new Locator("Sign In Button", ".btn-signIn");
}
```
