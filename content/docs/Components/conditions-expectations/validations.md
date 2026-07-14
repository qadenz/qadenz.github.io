---
title: "Validations"
linkTitle: "Validations"
description: >
  Asserting the state of the UI with verify and check, hard and soft, one check or many.
weight: 1
---
A validation asks whether the UI meets expectation and records the answer. Every validation is a `Condition` paired with an `Expectation`, evaluated through one of two methods. Two design ideas shape how they behave.

The first is that hard and soft asserts are first-class. `verify()` is a hard assert: a failure stops the test. `check()` is a soft assert: a failure is recorded, and the test continues until a later checkpoint decides whether to stop. Both take the same Conditions and Expectations used everywhere else, so choosing how strict a validation should be never changes how it is written.

The second is that a single validation can cover several checks at once and report all of them, so a failing step gives a complete picture instead of stopping at the first problem.

## Choosing verify or check

`verify()` is the primary assertion, used in most tests. It evaluates the validation, and on failure it stops the test then and there.

```java
commander.verify(Conditions.visibilityOfElement(loginButton, Expectations.isTrue()));
```

`check()` is the soft variant. On failure it records the result and lets the test continue, so later steps still run. A recorded failure is made final by a later call to `Assertions.flush()`, covered below.

```java
commander.check(Conditions.textOfElement(greeting, Expectations.isEqualTo("Hello World!")));
```

Both methods live on the `Commands` hierarchy, so they are callable from any descendant such as the `WebCommander`. The only difference in use is that `check()` needs a `flush()` at some later point to turn recorded failures into a stopped test.

## A complete picture when a step fails

Most steps check more than one thing. After adding an item to the cart, a step might confirm the notification message, the cart quantity, and that checkout is now enabled. Written as three separate assertions, the first failure stops the step and the other two never run. The report shows one problem when there might be three, and the rest stay hidden until the bug is fixed and the test runs again.

Pass several Conditions to a single `verify()` or `check()`, and every one is evaluated before the call decides what to do.

```java
commander.verify(
        Conditions.textOfElement(snackbarNotification, Expectations.isEqualTo("Items added successfully!")),
        Conditions.textOfElement(cartQuantity, Expectations.isEqualTo("1")),
        Conditions.enabledStateOfElement(checkoutButton, Expectations.isTrue()));
```

Each Condition is evaluated and reported on its own. With `verify()`, a failure in the group still stops the test, but only after all three have run, so a single execution surfaces every problem in the step. It behaves like a soft assertion wrapped in a hard one: the completeness of a soft assert, with the firm stop of a hard assert.

## Managing soft assertions

`check()` works alongside the static `Assertions.flush()` method. As a test runs, each failed `check()` sets a per-test failure flag on the [`Assertions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/Assertions.java) tracker. Calling `Assertions.flush()` inspects that flag: if any failure has been recorded, it throws and the test stops; if not, execution continues.

```java
commander.check(Conditions.textOfElement(firstName, Expectations.isEqualTo("Ada")));
commander.check(Conditions.textOfElement(lastName, Expectations.isEqualTo("Lovelace")));
Assertions.flush();
```

Place `flush()` wherever it makes sense to stop if failures have piled up. Because the flag lives for the whole test, a test can hold several `flush()` calls, each a checkpoint that halts if anything has failed up to that point. This suits long smoke or end-to-end tests, where finishing the run matters for a full accounting of the important validations.

One rule to keep in mind: a test that uses only `check()` needs at least one `flush()`. The `AssertionError` that marks a test failed is thrown only by `flush()`, so leaving it out has a quiet but serious consequence. Each failed `check()` still logs its failure and captures a screenshot on its own step, so the report looks like it caught the problem. But with nothing to throw the error, the test's overall result is `passed`, and it is filed with the passing tests. The failure sits in plain view on a step while the suite reports green, and a real defect can slip by unnoticed.

Nothing in the compiler or the IDE enforces this. A test can call `check()` and never call `flush()` and still compile and run. Treat the pairing as a habit: any test that uses `check()` ends its relevant checkpoints with a `flush()`, so recorded failures actually decide the test's result.

Mixing the two methods is fine, with one consequence to expect. If a `verify()` fails after some `check()` calls but before a `flush()`, the test stops at that `verify()`, and the earlier recorded failures never reach a `flush()`.

## Screenshots

By default, a validation captures a screenshot whenever a Condition fails and embeds it in the report. Nothing extra is needed to get this.

To turn it off for a validation, pass `Screenshot.SKIP` as the first argument to `verify()` or `check()`.

```java
commander.verify(Screenshot.SKIP, Conditions.visibilityOfElement(spinner, Expectations.isFalse()));
```

`Screenshot.SKIP` is a convenience constant that resolves to `false`. Passing `false` directly works the same way, but the named constant states the intent at a glance. Screenshots are captured per failed Condition, so skipping applies to every Condition in a grouped call.