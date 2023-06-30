---
title: Web Inspector
description: >
  Web Inspector Description
weight: 2
---

The `WebInspector` works alongside the `WebCommander`, but instead of acting upon UI elements, this class interrogates them for information. This includes retrieving inner text values, attribute values, element state, and instance counts. Most of the methods on `WebInspector` are used by Conditions and Expectations as part of evaluative logic for verifications and waits, but can be useful through the course of a test for retrieving data from the UI under test.

Each method on the `WebInspector` includes a number of activities beyond simply performing WebElement inspections. The workflow for these methods is as follows:

1. Log the action taking place and the name of the target element.
2. Initialize a `WebElement` using the provided `Locator` instance.
3. Retrieve the required information from the `WebElement`.
4. Catch and log any exceptions that are thrown.
5. If an exception is caught, capture a screenshot of the UI under test.
6. Throw the exception to stop execution of the test.

#### Element Attributes and Properties

The values assigned to attributes on elements can be retrieved via the `getAttributeOfElement()` method by passing the name of the attribute to be evaluated. Invoking `getAttributeOfElements()` will return the value for all instances of the matching `Locator` on a `List<String>`.

Similarly, the value of a CSS property can be retrieved from an element via the `getCssPropertyOfElement()` method.

#### Element States

Element states can be evaluated and will return a boolean value based on the result.

##### Enabled

The `getEnabledStateOfElement()` method evaluates if an element is enabled for interaction, and returns `true` if the element is in fact enabled. This is useful for determining if elements such as `<input>` elements or `<button>` elements are enabled. The state is determined by first invoking the `WebElement.isEnabled()` method. If the WebElement evaluates as enabled, a second check is performed against [user-defined attributes](/components/locators/#disabled-elements) on the element via the `Locator` instance.

This method presumes an element is enabled until one of the evaluations proves the element is disabled.

##### Selected

The `getSelectedStateOfElement()` method evaluates if an element is selected, and returns `true` if the element is in fact selected. This is useful for determining if elements such as checkboxes or radio buttons are selected. The state is determined by first invoking the `WebElement.isSelected()` method. If the WebElement evaluates as selected, a second check is performed against [user-defined attributes](/components/locators/#selected-elements) on the element via the `Locator` instance.

This method presumes an element is selected until one of the evaluations proves the element is not selected.

##### Visiblity

The `getVisibilityOfElement()` method evaluates if an element is visible on the UI, and returns `true` if the element is in fact visible. The state is determined by first finding all matching DOM nodes for the element selector on the given `Locator`. If more than zero matches are found, the method will then evaluate the dimensions of the first matching node. If the element has a height and width greater than zero, the method will then evaluate the styling and attributes of the element. The element will be checked for `display:none;` and then `visibility:hidden;`, and finally a check for the `hidden` attribute. If these checks result in no match, the method will then evaluate [user-defined attributes](/components/locators/#hidden-elements) on the element via the `Locator` instance. Additionally, the method will catch a `StaleElementReferenceException` throughout this series of evaluations and instantly return a `false` result.

This method presumes an element is visible until one of the evaluations proves the element is hidden.

#### Element Text

The `WebInspector` offers methods that retrieve text of UI elements in a variety of ways and a variety of formats. In addition to returning String values, methods exist to convert and return element text values as Temporals or Numbers as well. This allows a test to be designed where the UI data can be interacted with in a more flexible way, enjoying the ability to perform operations against, comparisons, and validations with the capabilities of these secondary object types.

General text inspection methods allow retrieval of basic element text, a `List` of text values from all matching nodes of the given `Locator`, the currently selected `<option>` value on a `<select>` menu, or the `List` of `<option>` values on a `<select>` menu.

##### Element Text vs Direct Element Text

The `WebElement.getText()` method returns the visible inner text of the given element along with the text of any child elements. In most use cases, this is perfectly fine. Some situations exist, however, when the DOM is constructed in such a way where a target element has child elements that also contain visible inner text that a tester may wish to ignore or avoid for an inspection or validation. For these scenarios, `WebInspector` employs a concept of 'direct text of element'. Methods with `directTextOfElement` in the name will retrieve element text, but prior to returning the text value to the calling method, will filter the visible inner text values of any child elements relative to the given target element.

##### Element Text as a non-String Object

The `WebInspector` includes the means to examine the text of an element, and with the aid of a formatter, parse and convert the text value into a `Number` or `Temporal`.

A `NumberFormat` can be used to drive the converstion of a text value to either an `Integer` or a `Double` value. A `DateTimeFormatter` can be used to drive the conversion to either a `LocalDate`, `LocalDateTime`, or a `LocalTime` object. By using these methods, the UI data can be used in mathematical operations directly without requiring the need to build any parsing or conversion logic into the test.

#### Element Instances

There are also methods on `WebInspector` that focus specifically on all instances of an element. The `getCountOfElement()` simply returns the number of instances on the DOM of a matching `Locator`. This can drive mathematical operations, looping logic, or precision validations of element groups. The `getInstanceOfElementText()` and `getInstanceOfElementAttribute()` will examine all instances of a matching `Locator` and return the index of the first node that contains a matching text or attribute value. This capability can provide an extra convenience in dynamically parameterizing `Locator` selector values based on the state if the UI.
