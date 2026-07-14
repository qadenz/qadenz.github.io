---
title: "Locators"
description: >
  Map a single element: display name, selector, runtime parameters, parent chaining, and the state attributes that harden inspections against unconventional UIs.
weight: 1
---

A `Locator` is the central piece of UI modeling in Qadenz. It pairs a display-friendly name with a CSS selector, and that small pairing does three jobs: it names the element in every log and report, it carries parameters that let one mapping reach a whole family of elements, and it holds optional attributes that sharpen element-state inspections.

## Basics of a Locator

A `Locator` carries the name of an element and its selector.

```java
public Locator(String name, String selector)
```

An overloaded constructor accepts a parent `Locator`, prepending the parent's selector to this one. This defines a child relationship and removes repeated selector segments, letting one parent stand as the reference point for several child elements.

```java
public Locator(String name, Locator parent, String selector)
```

### Display-friendly Element Names

Selenium works in selectors, not names. It is the tool that drives the browser, so when a step fails, the reference a tester gets back is the selector rather than a recognizable element. Reading a stack trace then begins with translating that selector into an actual place on the page before the failure can be located in the sequence of steps.

A `Locator` closes that gap by requiring a name.

```java
Locator signInButton = new Locator("Sign In Button", ".btn-signIn");
```

Qadenz uses that name as the element's primary identifier in all logging and reporting. A report reads in the application's own words, and the step where a problem appears is marked plainly, with no selector-to-element lookup to perform first.

### CSS Selectors

Every `Locator` selector is a CSS selector, and Qadenz uses CSS exclusively. See [Selectors and Sizzle]({{< relref "selectors-and-sizzle.md" >}}) for why CSS is the single standard, and for the Sizzle pseudo-classes that make parameterization possible.

## Parameterization

Parameterization cuts repeated code and makes a single mapping reusable across many elements.

### Selector Parameters

Take a list of search results. With `@FindBy`, each result needs its own annotated `WebElement`, or a single `List<WebElement>` plus extra logic to pick the instance a step needs. Either way costs lines.

A parameterized `Locator` replaces them with one method. The argument feeds both the selector, through a Sizzle `:contains()` match, and the name.

```java
public Locator searchResultLink(String name) {
    return new Locator(name + " Search Result Link", ".search-result:contains(" + name + ")");
}
```

Passing the argument into the `name` field pays off in the logs: each interaction reports the exact element instance it acted on.

### Parent Locators

A `Locator` can be built from another `Locator`, combining the parent's selector with this one into a single value. Only the parent's selector is folded in, and only at construction: the child does not keep a reference to the parent or inherit its name or state attributes. This abstracts repeated selector segments across closely related elements.

Consider an e-commerce page that lists catalog items. Each item card holds the item name, a cost, a quantity field, and an "Add to Cart" button.

As a simple HTML representation:

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
</div>
```

The selector for the "Add to Cart" button could be:

```css
#item-list-section .item-card:contains(ACME Rocket Powered Roller Skates) .item-add button
```

Mapping the other elements on the card repeats the item-card segment every time. Pull it into a parent `Locator`, and the mappings for the card, the quantity field, and the button become:

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

The `itemCard()` `Locator` does double duty here: it abstracts the shared selector, and it stands on its own as the target for verifying an item card is visible.

Parent chaining is not limited to one layer. A `Locator` can serve as a parent as many times as needed.

## Definable Element State Attributes

Some UIs defeat the standard element-state inspections. Styling, DOM structure, the UI framework in play, or unconventional markup can make visibility, selected, or enabled states report incorrectly. To keep those inspections accurate, a `Locator` accepts three optional attributes that feed extra checks into the matching `WebInspector` methods.

Each attribute is set through a method on the `Locator` and names an HTML attribute, optionally with an expected value. Each setter is overloaded: pass an attribute and a value, or an attribute name alone for empty or boolean attributes. When a `WebInspector` method that supports custom attribute checks runs, the custom check is appended to its default checks for that state.

### Disabled Elements

`WebInspector.getEnabledStateOfElement()` inspects `<input>` elements for whether they accept user input. It assumes the element is enabled and runs each check in an attempt to prove it disabled.

Take a form with a checkbox that is only enabled under certain conditions. The form is heavily styled, and the developer built the checkbox in CSS with no underlying `<input>`, so `WebElement.isEnabled()` does not report reliably. Identify the CSS class that renders the checkbox inoperable and hand it to the `Locator`.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setDisabledByAttribute("class", "checkbox-disabled");
```

`setDisabledByAttribute()` registers the attribute that marks the element disabled.

### Hidden Elements

`WebInspector.getVisibilityOfElement()` checks for a matching element, its dimensions, and the standard W3C means of rendering an element invisible. It assumes the element is visible and runs each check in an attempt to prove it hidden, until one check succeeds and reports the element hidden, or the checks run out and it reports the element visible.

Take a "Confirm" button that stays hidden until a form is complete, hidden by an unconventional approach that standard inspection reads as visible. Identify the CSS class that hides the element and pass it along.

```java
Locator confirmButton = new Locator("Confirm Button", ".customButton-confirm")
        .setHiddenByAttribute("class", "invisible");
```

`setHiddenByAttribute()` registers the attribute that marks the element hidden.

### Selected Elements

`WebInspector.getSelectedStateOfElement()` inspects checkboxes, `<select>` options, and radio buttons for whether they are selected. It assumes the element is unselected and runs each check in an attempt to prove it selected.

Back on the checkbox above, the developer also animates the selected state, so `WebElement.isSelected()` does not report reliably. Identify the CSS class applied when the box is checked and configure the `Locator`.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setSelectedByAttribute("class", "checkbox-checked");
```

`setSelectedByAttribute()` registers the attribute that marks the element selected.

### Fluent Design

Every state-attribute setter returns the `Locator`, so the calls chain. The "I Agree" checkbox above needs both a disabled-by and a selected-by attribute, and both are set in one expression.

```java
Locator iAgreeCheckbox = new Locator("I Agree Checkbox", "#i-agree")
        .setDisabledByAttribute("class", "checkbox-disabled")
        .setSelectedByAttribute("class", "checkbox-checked");
```
