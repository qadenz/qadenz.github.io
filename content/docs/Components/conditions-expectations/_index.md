---
title: "Conditions & Expectations"
linkTitle: "Conditions & Expectations"
description: >
  One vocabulary for describing the state of the UI, used for both waits and validations.
weight: 4
---
Every UI test asks the same handful of questions over and over. Is this element visible? Does this field hold the expected text? Is this button enabled? Qadenz gives us exactly one way to ask, whether we are waiting for the answer or asserting on it.

That single vocabulary is the pairing of `Conditions` and `Expectations`, and it is the core evaluative logic of the framework.

## The two-vocabulary tax

Raw Selenium and TestNG make us learn the same question twice.

To wait for an element to become visible, we reach for Selenium's `ExpectedConditions`. To assert that the same element is visible, we reach for a TestNG assertion, or a Hamcrest matcher, or some combination of the two. Two unrelated APIs, two syntaxes, two mental models, all for one question about the state of the page.

Multiply that across a suite and across a team, and every contributor settles on a personal blend. The code drifts. Reading someone else's test starts with decoding which dialect they happened to choose that day.

## One question, one vocabulary

A `Condition` describes what to look at on the page and reports its actual state. An `Expectation` describes what that state should be. Pair them, and the evaluation is complete.

```java
Conditions.textOfElement(greeting, Expectations.isEqualTo("Hello World!"));
```

Read it left to right and it says what it does: the text of the greeting element is equal to `Hello World!`. The `Condition` supplies the What, the actual outcome. The `Expectation` supplies the measure, the expected outcome. In terms every tester already knows, ACTUAL against EXPECTED.

The result is that a tester only has to remember one way of expressing an evaluation. That one habit covers every check in the suite.

## The same Condition waits and asserts

Here is where the pairing earns its place. The exact same `Condition` object drives both synchronization and assertion.

```java
// Wait until the login button is visible.
commander.pause(Conditions.visibilityOfElement(loginButton, Expectations.isTrue()));

// Assert the login button is visible.
commander.verify(Conditions.visibilityOfElement(loginButton, Expectations.isTrue()));
```

`pause()` hands the `Condition` to a WebDriver wait and polls it until it is satisfied or the timeout expires. `verify()` evaluates the `Condition` once and asserts on the result. Same Condition, same Expectation, same underlying evaluation. There is no `ExpectedConditions` for the wait and a separate assertion library for the check. There is one vocabulary to learn, and it works everywhere.

This unification is the largest single reason Qadenz test code stays compact. One vocabulary, applied to waiting and validating alike.

Validations build on that vocabulary with hard and soft asserts, and grouped checks that report every failure in a step rather than stopping at the first. See [Validations]({{< relref "/docs/Components/conditions-expectations/validations.md" >}}).

## Hamcrest does the matching

An `Expectation` is a thin wrapper over a Hamcrest [`Matcher`](https://hamcrest.org/JavaHamcrest/). `Expectations.isEqualTo(...)` produces an equality matcher, `Expectations.contains(...)` a substring matcher, and so on down the list.

Qadenz leans on Hamcrest rather than reinventing comparison logic. Expectations stay predictable to anyone who has used matchers before, and the matching itself is handled by a library built for exactly that job.

## Paired by value type

A Condition reports a value of one type: a boolean, a piece of text, a number, a date or time, or a list. Every Expectation is built for one of those same types. Pairing a Condition with an Expectation means matching the types, and the compiler enforces it. A text Condition will not accept a numeric Expectation, so a mismatched pair never reaches a running test.

This is why the same names can be reused across types. `isEqualTo` reads naturally for text, numbers, and dates alike, and the right one is chosen by the value passed. The menu we actually have to learn stays small.

The [Conditions]({{< relref "/docs/Components/conditions-expectations/conditions.md" >}}) and [Expectations]({{< relref "/docs/Components/conditions-expectations/expectations.md" >}}) catalogs group everything by these types and show the pairings.

## What a team gets

Because every evaluation flows through the same two types, a few things follow without any extra effort:

- **Every contributor writes checks the same way.** There is no house style to negotiate, because the API offers one path.
- **The logs describe themselves.** A Condition and its Expectation each know how to state themselves in words. The example above reports as `Text of element [Greeting Message] is equal to [Hello World!].` without a line of logging code.
- **Maintenance stays cheap.** When the application changes, we adjust a Locator or an expected value, not a scattered mix of assertion styles.

## Where to go next

- **[Validations]({{< relref "/docs/Components/conditions-expectations/validations.md" >}})** covers the mechanics: `verify` against `check`, grouping several Conditions in one call, flushing soft assertions, and screenshots on failure.
- **[Conditions]({{< relref "/docs/Components/conditions-expectations/conditions.md" >}})** and **[Expectations]({{< relref "/docs/Components/conditions-expectations/expectations.md" >}})** catalog what is available to pair, grouped by value type.