---
title: "Extensibility"
linkTitle: "Extensibility"
description: >
  Writing custom Conditions and Expectations when the built-in vocabulary does not cover a case.
weight: 5
---
The built-in [Conditions]({{< relref "/docs/Components/conditions-expectations/conditions.md" >}}) and [Expectations]({{< relref "/docs/Components/conditions-expectations/expectations.md" >}}) cover the cases most tests need. When one falls outside them, extend the vocabulary rather than reaching for raw Selenium. A custom Condition or Expectation plugs into `pause()`, `verify()`, and `check()` exactly like a built-in one, and it logs to the report the same way. The test code stays in one vocabulary, and nothing about reading or maintaining it changes.

## A custom Condition

A `Condition` implements two methods: `result()`, which performs the evaluation and returns whether it passed, and `actual()`, which reports the actual value for the logs. Overriding `toString()` supplies the description that appears on the report.

There is no built-in Condition for the browser's page title. Here is one.

```java
public class TitleOfPage implements Condition {

    private Expectation<String> expectation;
    private String title;

    public TitleOfPage(Expectation<String> expectation) {
        this.expectation = expectation;
    }

    @Override
    public Boolean result() {
        title = WebDriverProvider.getWebDriver().getTitle();
        return expectation.matcher().matches(title);
    }

    @Override
    public String actual() {
        return title;
    }

    @Override
    public String toString() {
        return "Title of page " + expectation + ".";
    }
}
```

It takes an `Expectation<String>`, so it reuses every text Expectation already available. And because it is an ordinary `Condition`, the same instance serves a wait or a validation.

```java
commander.pause(new TitleOfPage(Expectations.contains("Dashboard")));
commander.verify(new TitleOfPage(Expectations.isEqualTo("Dashboard - Acme")));
```

## A custom Expectation

An `Expectation<T>` implements a single method, `matcher()`, which returns the Hamcrest `Matcher` that performs the comparison. As with a Condition, `toString()` supplies the description for the report.

Custom Expectations are needed far less often, since the built-in set covers the common comparisons and Hamcrest supplies the matching. Reach for one when a comparison has no existing Expectation, wrapping any Hamcrest `Matcher`.

```java
public class MatchesRegex implements Expectation<String> {

    private String regex;

    public MatchesRegex(String regex) {
        this.regex = regex;
    }

    @Override
    public Matcher<String> matcher() {
        return org.hamcrest.Matchers.matchesPattern(regex);
    }

    @Override
    public String toString() {
        return "matches pattern [" + regex + "]";
    }
}
```

Pair it with any text Condition, the same as a built-in Expectation.

```java
commander.verify(Conditions.textOfElement(orderNumber, new MatchesRegex("ORD-\\d{6}")));
```

## How the descriptions combine

A Condition and its Expectation contribute their `toString()` output to a single log line. The Condition names what it evaluated and appends the Expectation, which completes the sentence. Take the built-in text check:

```java
commander.verify(Conditions.textOfElement(greeting, Expectations.isEqualTo("Hello World!")));
```

The Condition's `toString()` produces `Text of element [Greeting Message] `, the Expectation's produces `is equal to [Hello World!]`, and concatenation joins them. The report reads:

```
Text of element [Greeting Message] is equal to [Hello World!].
```

Structure a custom `toString()` to fit that sentence:

- **A Condition is the subject, and closes the line.** Name what is evaluated, put any element name or value in square brackets, and end with a space before the Expectation and a period after it: `"Title of page " + expectation + "."`.
- **An Expectation is the predicate.** Write a lowercase phrase that completes the sentence, with no leading capital and no trailing period, so the Condition supplies the closing punctuation: `"matches pattern [" + regex + "]"`.

Following the pattern keeps custom evaluations reading the same as the built-ins on the report, so the logs stay consistent and easy to scan.

## Dropping to WebDriver directly

For the rare wait or check that no Condition can model, even with a custom implementation, the underlying tools stay reachable. [Extend the `WebCommander`]({{< relref "/docs/Components/Commands/extensibility.md" >}}) to create a `WebDriverWait` and pass a Selenium `ExpectedCondition` directly. Treat this as the last option, well after a custom Condition, since it steps outside the vocabulary and the reporting that comes with it.