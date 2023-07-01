---
title: "Locators"
weight: 1
---

The Locator is the central component of UI modeling with Qadenz. It is a key ingredient in an approach that seeks to improve the UI modeling by avoiding the `PageFactory` class and `@FindBy` annotation entirely. It's design sets out to accomplish several things. First, the Locator is a clean wrapper for both an element's selector and a display friendly name. Second, the Locator is a vehicle for parameterization of element selectors, leading to much more efficient UI models. And finally, the Locator carries attributes that assist with validations and element inspections.

In short, Qadenz is quite happy to follow [Simon Stewart's advice](https://www.youtube.com/watch?v=gyfUpOysIF8&t=1519s), and find a better way.

## Basics of a Locator

The Locator object simply carries the name of an element, the element’s selector.

```java
public Locator(String name, String selector)
```

An overloaded constructor allows one Locator instance to be passed to another, allowing a child relationship to be defined and to reduce repetitious selector segments. This approach will append the selector of the child to the selector of the parent, allowing the parent to be used as a reference point for one or more child elements.

```java
public Locator(String name, Locator parent, String selector)
```

### Display-friendly Element Names

Selenium does not readily (nor should it, being simply the tool that automates browsers) provide a meaningful context-friendly reference to element names, which can complicate debugging and troubleshooting when problems arise. Without additional logging in the test project, testers are generally only provided references to elements expressed as the given selectors. This has a very strong potential to slow down the resolution process as testers must first translate the selector in their stack trace to an actual element on the UI, which can then establish a point of orientation within the progression of test steps.

```java
Locator signInButton = new Locator("Sign In Button", ".btn-signIn");
```

By requiring a `name` value to be given in the Locator constructor, Qadenz refers to the display-friendly name of an element as the primary identifier in all logging and reporting output. This eliminates the time needed to perform any lookup or cross-referencing of selectors to elements in relation to test steps. By reviewing the default logging output or report content, the point at which a problem appears in a test is clearly marked and quickly identified.

### CSS Selectors

Many teams will choose either a default standard selector strategy (ID, CSS, XPath, etc.), or at least set an order of prioritization for the varying types. Qadenz has chosen CSS selectors for the superior performance, ease of use, and better browser compatibility when compared to XPath. The addition of the [SizzleJS](https://github.com/jquery/sizzle) library provides a number of pseudo-classes that make element selection more flexible and accurate.

The end result is a powerful and straightforward selector strategy that supports full parameterization capabilities. This also leads to a singular defined approach that an entire team can adopt and consistently apply across the entire project.

## Parameterization

Parameterization is a simple and effective means to reducing repeated code, and increasing the reusability of each component.

### Selector Parameters

Using list of search results as an example, testers would be forced to create a separate `@FindBy` annotated `WebElement` instance for each result element needed for a test. Alternately, a `@FindBy` could be used to initialize a single `List<WebElement>`, but additional logic would be necessary to identify a specific result element needed for a test step. In either case, the result is many extra lines of code.

Using a parameterized Locator (coupled with the benefits of Sizzle CSS Selectors), testers will be able to define a single Locator instance for a generic search result, and rely upon the parameterization to direct the test to choose the appropriate element instance.

```java
public Locator searchResultLink(String name) {
    return new Locator(name + " Search Result Link", ".search-result:contains(" + name + ")");
}
```

In this example, we also have a benefit of passing the parameter to the `name` field on the Locator, which increases clarity in the logs and reports by providing the exact instance of the element against which the interaction takes place.

### Parent Locators

The Locator can hold an optional instance of another Locator. This is intended to allow abstraction of selector segments by combining the selector of the parent Locator with that of the current Locator as a single selector value. This can be helpful in reducing repeated selector segments when creating Locators for closely related UI Elements.

Consider an e-commerce application wherein a list of catalog items are presented on the UI. Each item card contains the item name text, a ‘Cost’ value, a ‘Quantity’ field, and an ‘Add to Cart’ button.

As a very simple HTML representation:

```html
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

The selector for the ‘Add to Cart’ button could be:

```css
#item-list-section .item-card:contains(ACME Rocket Powered Roller Skates) .item-add button
```

While mapping the other elements on an item card, however, it would be discovered that the item card selector itself is repeated on each of the child elements. In this situation, a parent `Locator` could be created to abstract the repeated selector segments, especially if the abstracted selector can stand as it’s own element mapping.

With a parent `Locator` the resulting element mappings for the item card, ‘Quantity’ field, and ‘Add to Cart’ button could be:

```java
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



## Definable Element State Attributes

There are some cases where the UI under test experiences conditions where traditional element state inspections are unreliable due to styling, DOM structure, UI framework in use, or perhaps even unconventional UI development. This can sometimes produce incorrect results on inspections like visibility, selected, or enabled states. To mitigate this issue, the Locator allows three additional fields to be set to assist evaluating these element states.

These fields are available as setter methods on the Locator class, and each will define an HTML attribute and expected value that will be added to the corresponding element inspection methods on `WebInspector`. Each of these methods are overloaded to allow a tester to define either an attribute/value combo, or just an attribute name in the case of empty or boolean attributes.

When a `WebInspector` method that supports custom attribute checks runs, the attribute check will be appended to any default checks made to determine the given state of an element.

### Disabled Elements

The `WebInspector.getEnabledStateOfElement()` method checks `<input>` elements to determine whether the element is enabled for user input. This method presumes the element to be enabled, and performs each check in an attempt to prove the element is disabled.

Consider an example where a form is present on the UI that contains a checkbox that is only enabled for input under specific conditions. The form UI is heavily stylized, and the UI developer has chosen to create checkboxes using CSS with no underlying `<input>` element. Calls to `WebElement.isEnabled()` are not reliably returning an accurate result due to the UI design.

The tester has identified the CSS class that renders the checkbox inoperable, and will configure the `Locator` to provide this information to `WebInspector` to yield accurate inspections.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setDisabledByAttribute("class", "checkbox-disabled");
```

To specify a custom attribute that determines the element as disabled, the `setDisabledByAttribute()` method on the Locator must be called.

### Hidden Elements

The `WebInspector.getVisibilityOfElement()` method checks for elements that match the provided selector, dimensions of the element, and standard W3C defined means of rendering elements invisible. This method presumes the element to be visible, and performs each of the checks to attempt to prove the element is in fact hidden until a check proves the element hidden (and returns a result accordingly), or no additional checks can be made (in which case the element is determined to indeed be visible).

Consider an example of a “Confirm” button that is present on a form page, but is hidden from view until the form input has been completed by the user. Unfortunately, the UI developers have taken an unconventional approach to hiding this element, and normal visibility inspections are determining the element is visible. The tester has identified the CSS class that hides the element, and is able to pass this information to the WebInspector for a more accurate result.

```java
Locator confirmButton = new Locator("Confirm Button", ".customButton-confirm")
        .setHiddenByAttribute("class", "invisible");
```

To specify a custom attribute that defines the element as hidden, the `setHiddenByAttribute()` method must be called.

### Selected Elements

The `WebInspector.getSelectedStateOfElement()` method checks element such as checkboxes, options in a `<select>` menu, and radio buttons to determine whether the element is selected. This method presumes the element to be unselected and performs each check in an attempt to prove the element to be selected.

Revisiting the example above for the disabled checkbox, the UI developer has also created an animated interaction when the checkbox is selected. Calls to `WebElement.isSelected()` are not reliably returning an accurate result.

The tester has identified the resulting CSS class responsible for rendering the checkbox as checked, and will configure the `Locator` as required.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setSelectedByAttribute("class", "checkbox-checked");
```

To specify a custom attribute that defines the element as selected, the `setSelectedByAttribute()` method must be called.

### Fluent Design

The setters on the `Locator` for each of the element state attributes all return a self-reference. This allows the setter calls to be chained together. In the examples above where the “I Agree” checkbox has both disabled-by and selected-by attributes defined, the `Locator` can be instantiated and both configurations can be made in one chained method call.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setDisabledByAttribute("class", "checkbox-disabled")
        .setSelectedByAttribute("class", "checkbox-checked");
```

## The LocatorGroup

The `LocatorGroup` allows multiple `Locator` instances to be combined together on a `List`, and acted upon as a group. This is commonly applied to verify the visibility of UI component, or a set of default UI elements. Instead of passing multiple individual Conditions to a `.verify()` or `.check()` validation, a LocatorGroup can be verified with a single `Condition` call.

For example, a simple authentication form has several basic elements, the ‘Username’ field, the ‘Password’ field, a ‘Remember Me’ checkbox, and a ‘Sign In’ button. Each element would be mapped as an individual `Locator` instance for the purposes of input, but these same elements could also be included in a `LocatorGroup`, should the need arise to verify each element to be visible as part of the form.

```java
Locator usernameField = new Locator("Username Field", "#username");
Locator passwordField = new Locator("Password Field", "#password");
Locator rememberMeCheckbox = new Locator("Remember Me Checkbox", "#remember-me");
Locator signInButton = new Locator("Sign In Button", "#sign-in");

LocatorGroup signInForm = new LocatorGroup("Sign In Form",
        usernameField, passwordField, rememberMeCheckbox, signInButton);
```

By combining individual Locators onto a `LocatorGroup` instance, testers are able to identify and refer to collections of elements as the UI Components they represent.
