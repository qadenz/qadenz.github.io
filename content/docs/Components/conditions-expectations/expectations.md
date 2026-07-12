---
title: "Expectations"
linkTitle: "Expectations"
description: >
  The full menu of Expectations, grouped by the value type each one evaluates.
weight: 2
---
An `Expectation` defines the expected value and how to compare against it. Each one wraps a Hamcrest `Matcher`. Choose an Expectation whose value type matches the Condition it pairs with. See [Conditions]({{< relref "/docs/Components/conditions-expectations/conditions.md" >}}) for the pairing table.

Many names are shared across types. `isEqualTo` exists for text, numbers, and dates, and the version in play is chosen by the type of the value passed. So the surface to learn is smaller than the method count suggests.

For exhaustive signatures, see the [`Expectations` API reference](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/expectations/Expectations.java). If none of them covers a comparison, [build a custom Expectation]({{< relref "/docs/Components/conditions-expectations/extensibility.md" >}}).

## Boolean

- `isTrue`
- `isFalse`

```java
Conditions.enabledStateOfElement(submitButton, Expectations.isFalse());
```

## Text

- `isEqualTo` / `isNotEqualTo`
- `contains` / `doesNotContain`
- `startsWith` / `doesNotStartWith`
- `endsWith` / `doesNotEndWith`
- `equalsIgnoringCase` / `containsIgnoringCase`
- `isEqualToOneOf` — matches any one of several values
- `isEmptyOrNull` / `isNotEmptyOrNull`

```java
Conditions.textOfElement(errorBanner, Expectations.contains("invalid credentials"));
```

## Numeric

- `isEqualTo` / `isNotEqualTo`
- `isGreaterThan` / `isGreaterThanOrEqualTo`
- `isLessThan` / `isLessThanOrEqualTo`

Each is overloaded for `Integer` and `Double`.

```java
Conditions.countOfElement(searchResults, Expectations.isGreaterThanOrEqualTo(1));
```

## Temporal

- `isBefore` / `isAfter`
- `isEqualTo` / `isNotEqualTo`
- `isEqualToOrBefore` / `isEqualToOrAfter`
- `isWithin` / `isNotWithin` — inside a span of time measured from a reference point
- `isDayOfWeekAsLocalDate` / `isDayOfWeekAsLocalDateTime`
- `isNotDayOfWeekAsLocalDate` / `isNotDayOfWeekAsLocalDateTime`

The comparison Expectations are overloaded for `LocalDate`, `LocalDateTime`, and `LocalTime`. `isWithin` and `isNotWithin` take a magnitude and a `ChronoUnit` alongside the reference value.

```java
Conditions.textOfElementAsDateTime(timestamp, DateTimeFormatter.ISO_DATE_TIME,
        Expectations.isWithin(5, ChronoUnit.MINUTES, LocalDateTime.now()));
```

## List

- `listContainsValues` — the collected values contain the given values

```java
Conditions.textOfListedElementsUnordered(cartItemNames,
        Expectations.listContainsValues(List.of("Backpack", "T-Shirt")));
```