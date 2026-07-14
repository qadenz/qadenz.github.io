---
title: "Conditions"
linkTitle: "Conditions"
description: >
  The full menu of Conditions, grouped by the value type each one evaluates.
weight: 3
---
A `Condition` reads one aspect of the UI and reports its actual state. Choose a Condition by what needs to be read from the page, then pair it with an `Expectation` of the same value type.

Every Condition evaluates a value of one of five types. That type decides which Expectations it can pair with, and the compiler enforces the match.

| Value type | Conditions (examples) | Pairs with Expectations |
|---|---|---|
| **Boolean** | `visibilityOfElement`, `enabledStateOfElement`, `selectedStateOfElement`, `presenceOfElement` | `isTrue`, `isFalse` |
| **Text** | `textOfElement`, `textOfInputElement`, `attributeOfElement`, `selectedMenuOption` | `isEqualTo`, `contains`, `startsWith`, `equalsIgnoringCase`, `isEqualToOneOf`, ... |
| **Numeric** | `countOfElement`, `textOfElementAsInteger`, `textOfElementAsDouble` | `isGreaterThan`, `isLessThanOrEqualTo`, `isEqualTo`, ... |
| **Temporal** | `textOfElementAsDate`, `textOfElementAsDateTime`, `textOfElementAsTime` | `isBefore`, `isAfter`, `isWithin`, `isDayOfWeekAsLocalDate`, ... |
| **List** | `textOfListedElementsUnordered`, `textOfSelectMenuOptions`, `selectedMenuOptions` | `listContainsValues` |

The sections below list every Condition by family. For exhaustive signatures, see the [`Conditions` API reference](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/conditions/Conditions.java). If none of them fits a case, [build a custom Condition]({{< relref "/docs/Components/conditions-expectations/extensibility.md" >}}).

## Boolean state

Report whether something is true of an element: visible, enabled, selected, or present. Pair with `isTrue` or `isFalse`.

- `visibilityOfElement` / `visibilityOfElements`
- `enabledStateOfElement` / `enabledStateOfElements`
- `selectedStateOfElement`
- `presenceOfElement` / `presenceOfElements`
- `presenceOfAlert`

```java
commander.verify(Conditions.visibilityOfElement(loginButton, Expectations.isTrue()));
```

The plural forms take a `LocatorGroup` and apply the same expectation to every element in the group. Presence checks whether an element exists in the DOM at all, while visibility additionally requires it to be rendered.

## Text

Read a string from an element and compare it. Pair with the text Expectations (`isEqualTo`, `contains`, `startsWith`, and the rest).

- `textOfElement` / `textOfElements` — the visible text of an element
- `textOfInputElement` — the value held in an input field
- `directTextOfElement` — only the element's own text, excluding text from child elements
- `attributeOfElement` — the value of a named attribute
- `cssPropertyOfElement` — the value of a named CSS property
- `textOfAlert` — the message on a JavaScript alert
- `selectedMenuOption` — the chosen option in a `<select>` menu

```java
commander.verify(Conditions.textOfElement(greeting, Expectations.isEqualTo("Hello World!")));
```

Reach for `directTextOfElement` when an element holds child elements whose text should be excluded. `attributeOfElement` and `cssPropertyOfElement` take the attribute or property name as a second argument.

## Numeric

Evaluate a number. Pair with the numeric Expectations (`isGreaterThan`, `isLessThanOrEqualTo`, and the rest).

- `countOfElement` — how many elements match the locator
- `textOfElementAsInteger` / `textOfElementAsDouble` — the element's text parsed into a number

```java
commander.verify(Conditions.countOfElement(searchResults, Expectations.isGreaterThan(0)));
```

The `...As...` Conditions read the element's text as a string, then parse it into a number using a formatter passed as an argument. Parsing stays explicit, so a value like `"$1,299.00"` is read correctly. Each also has a `directText...As...` form (`directTextOfElementAsInteger`, `directTextOfElementAsDouble`) that excludes child element text before parsing, matching `directTextOfElement` in the text family.

```java
commander.verify(Conditions.textOfElementAsDouble(price, new DecimalFormat("$#,##0.00"),
        Expectations.isLessThan(1500.00)));
```

## Temporal

Evaluate a date, time, or date-time. Pair with the temporal Expectations (`isBefore`, `isWithin`, `isDayOfWeekAsLocalDate`, and the rest).

- `textOfElementAsDate` — parsed to a `LocalDate`
- `textOfElementAsDateTime` — parsed to a `LocalDateTime`
- `textOfElementAsTime` — parsed to a `LocalTime`

Each reads the element's text and parses it with a `DateTimeFormatter` passed as an argument. As with the numeric reads, each has a `directText...` counterpart (`directTextOfElementAsDate`, `directTextOfElementAsDateTime`, `directTextOfElementAsTime`) that excludes child element text.

```java
commander.verify(Conditions.textOfElementAsDate(dueDate, DateTimeFormatter.ofPattern("MM/dd/yyyy"),
        Expectations.isAfter(LocalDate.now())));
```

## Lists

Evaluate the text of several elements at once. Pair with `listContainsValues`.

- `textOfListedElementsInOrder` / `textOfListedElementsUnordered` — text of every element matching one locator
- `directTextOfListElementsInOrder` / `directTextOfListElementsUnordered` — the same, excluding child element text
- `textOfSelectMenuOptions` — every option in a `<select>` menu
- `selectedMenuOptions` — the chosen options in a multi-select menu

```java
commander.verify(Conditions.textOfListedElementsUnordered(cartItemNames,
        Expectations.listContainsValues(List.of("Backpack", "Bike Light", "T-Shirt"))));
```

Choose `InOrder` when the sequence matters, such as a sorted table column, and `Unordered` when only membership matters.