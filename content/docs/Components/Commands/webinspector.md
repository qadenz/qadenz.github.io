---
title: "WebInspector"
linkTitle: "WebInspector"
description: >
  The commands that interrogate elements: text, attributes, CSS, element state, and instance counts.
weight: 2
---
The [`WebInspector`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/WebInspector.java) works alongside the `WebCommander`, but instead of acting on elements it interrogates them for information: inner text, attribute and CSS values, element state, and instance counts. Most of these methods are the evaluative logic behind [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}), and they are equally useful on their own for pulling data out of the UI during a test.

Each method follows the same [four-step anatomy]({{< relref "/docs/Components/Commands/_index.md" >}}) as the rest of the commands. It logs the inspection and target, initializes the element through an explicit wait, and on failure captures a screenshot before surfacing the exception. The difference is the third step: rather than performing an action, an inspection reads a value and returns it.

## Creating a WebInspector

The `WebInspector` shares the same two constructors as the [`WebCommander`]({{< relref "/docs/Components/Commands/webcommander.md#creating-a-webcommander" >}}), and the choice works the same way. The no-argument constructor attributes logged inspections to the `WebInspector`; the `Class<?>` constructor attributes them to the page or class passed in.

```java
WebInspector inspector = new WebInspector(getClass());
```

See [Logging]({{< relref "/docs/Components/Commands/logging.md" >}}) for what each constructor produces on the report.

## Attributes and CSS properties

`getAttributeOfElement(Locator, String attributeName)` returns the value of the named attribute. `getAttributeOfElements(Locator, String attributeName)` returns those values for every element matching the `Locator`, as a `List<String>`.

`getCssPropertyOfElement(Locator, String cssProperty)` returns the value of the named CSS property.

## Element state

Each of these methods evaluates a state and returns a `boolean`.

### Enabled

`getEnabledStateOfElement(Locator)` returns `true` when an element is enabled for interaction, useful for `<input>` and `<button>` elements. It first calls `WebElement.isEnabled()`, and if that reports enabled, checks the [user-defined disabled attribute]({{< relref "/docs/Components/ui-modeling/locators.md#disabled-elements" >}}) on the `Locator`. The method presumes an element is enabled until one of the checks proves otherwise.

### Selected

`getSelectedStateOfElement(Locator)` returns `true` when an element is selected, useful for checkboxes and radio buttons. It first calls `WebElement.isSelected()`, and if that reports selected, checks the [user-defined selected attribute]({{< relref "/docs/Components/ui-modeling/locators.md#selected-elements" >}}) on the `Locator`. The method presumes an element is selected until one of the checks proves otherwise.

### Visibility

`getVisibilityOfElement(Locator)` returns `true` when an element is visible on the UI. It finds all nodes matching the selector, then evaluates the first match through a series of checks: the element's dimensions must be greater than zero, and it must not carry `display: none;`, `visibility: hidden;`, or the `hidden` attribute. If those pass, it checks the [user-defined hidden attribute]({{< relref "/docs/Components/ui-modeling/locators.md#hidden-elements" >}}) on the `Locator`. A `StaleElementReferenceException` at any point returns `false`. The method presumes an element is visible until one of the checks proves otherwise.

## Element text

The `WebInspector` retrieves element text in several forms.

- `getTextOfElement(Locator)` returns the visible inner text of an element.
- `getTextOfElements(Locator)` returns the text of every element matching the `Locator`, as a `List<String>`.
- `getSelectedMenuOption(Locator)` returns the text of the currently selected option on a `<select>` menu.
- `getSelectedMenuOptions(Locator)` returns the text of all selected options on a multi-select `<select>` menu.
- `getTextOfOptions(Locator)` returns the text of every option on a `<select>` menu, selected or not.

### Element text vs. direct text

`WebElement.getText()` returns an element's inner text along with the text of any child elements, which is usually what a test wants. Some DOM structures nest child elements whose text should be ignored for an inspection. For those cases, the `WebInspector` offers a "direct text" variant of each text method. Methods with `directTextOfElement` in the name filter out the visible text of child elements before returning the value of the target element alone.

### Text as a number or date

The `WebInspector` can parse element text into a typed value instead of a raw `String`. Supplied with a `NumberFormat`, it converts text to an `Integer` or `Double`; supplied with a `DateTimeFormatter`, it converts to a `LocalDate`, `LocalDateTime`, or `LocalTime`. This lets a test do arithmetic, comparisons, and validations against UI data directly, without hand-rolling parsing logic.

```java
Integer itemsInCart = inspector.getTextOfElementAsInteger(cartBadge, NumberFormat.getInstance());
if (itemsInCart > 0) {
    commander.click(checkoutButton);
}
```

Each conversion is available in both standard and direct-text forms, for example `getTextOfElementAsInteger(Locator, NumberFormat)` and `getDirectTextOfElementAsDate(Locator, DateTimeFormatter)`.

## Element instances

Some methods operate across every instance of a `Locator`.

- `getCountOfElement(Locator)` returns the number of matching nodes on the DOM. This can drive arithmetic, looping logic, or precise validations of element groups.
- `getPositionOfElementWithText(Locator, String expectedText)` returns the index of the first matching element whose text contains the expected value.
- `getPositionOfElementWithAttribute(Locator, String attributeName, String expectedValue)` returns the index of the first matching element whose named attribute contains the expected value.

The two `getPositionOf...` methods are handy for dynamically parameterizing a `Locator` based on the current state of the UI: find the position of the element that contains a value, then target it by index. Because Qadenz uses Sizzle selectors, the returned position drops straight into a zero-based `:eq()` selector.

```java
int position = inspector.getPositionOfElementWithText(searchResult, "ACME Rocket Powered Roller Skates");
Locator matchedResult = new Locator("Matched Search Result", ".search-result:eq(" + position + ")");
commander.click(matchedResult);
```