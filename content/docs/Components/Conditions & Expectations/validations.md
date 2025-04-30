---
title: Validations
description: >
  Validations
weight: 1
---

Unit testing frameworks such as TestNG or JUnit include assertion functionality as a core component, and are relatively simple to use. Being open-ended frameworks, however, individual users may tend to express very similar validations in a variety of different assertions. This leads to inconsistent coding patterns, and more difficult maintenance of test code.

Using `Conditions` and `Expectations` allows a team to ensure all contributors are following the same pattern for validations.

```
Conditions.textOfElement(greetingText, Expectations.isEqualTo("Hello World!");
```

That said, Qadenz does employ a single TestNG assertion, the `assertTrue()` method, as a means of validating a Condition. The `result()` of a Condition is a simple representation of whether the state of the UI under test meets expectation. If the output of the Condition evaluation matches the Expectation, `result()` will return `true`.

By passing this result to the `assertTrue()` method, Qadenz is ensuring that a passing result depends on the Condition evaluation meeting the Expectation. If not, the validation will fail.

## Assertion Types

The concept of Hard Assertions and Soft Assertions are not new in the test automation world. Qadenz implements both concepts by way of the `verify()` and `check()` methods.

`verify()` represents a Hard Assertion. If the validation fails, the test will be marked as failed and execution will be stopped.

`check()` represents a Soft Assertion. If the validation fails, the test will be marked as failed, but execution will be allowed to continue until a call to `Assertions.flush()` is made, which will stop execution of the test if any failures have been encountered.

The `verify()` or `check()` methods are available on the Commands Hierarchy and are callable on any descendant class of `Commands`. The mechanics of using these validations are the same, with the only difference being an additional step with `check()` required to call `Assertions.flush()` in order to handle any failed Soft Assertions and stop execution.

## Grouped Conditions

Validations in Qadenz are further enhanced with the ability to evaluate multiple Conditions as a group. In scenarios where a single UI action can trigger multiple verification points in a test, a tester may have to express multiple assert statements to ensure necessary coverage. If, for example, the first assertion were to fail, the remaining assertions would remain unchecked until either the UI under test is fixed, or the test scenario is executed manually.

Using Qadenz, a tester is able to execute these same validations in one call to `verify()` or `check()`, and will receive results for each Condition evaluation regardless of individual results. If again, the first validation fails, Qadenz will perform handling tasks on the failure, then proceed to evaluate each of the other Conditions that were passed. In the case of a `verify()` with multiple Conditions where one or more have failed, halting of test execution will be delayed until all Conditions have been evaluated, which will ensure that the test step is completed in its entirety.

In the example below, a user has added an item to the shopping cart, and the next step will verify a snackbar notification is displayed with a confirmation message, the item quantity is shown on the shopping cart icon, and the 'Checkout Now' button is enabled.

```
commander.verify(
        Conditions.textOfElement(snackBarNotification, Expectations.isEqualTo("Items added successfully!")),
        Conditions.textOfElement(quantityInCartIndicator, Expectations.isEqualTo("1")),
        Conditions.enabledStateOfElement(checkOutNowButton, Expectations.isTrue()));
```

By grouping these verifications together, even if one (or more) Conditions fail, all will be evaluated and reported individually.

## Managing Soft Assertions

The `check()` methods works alongside the static `Assertions.flush()` method to delay execution stoppages in the event of failed validations. As calls to `check()` are made and executed through the course of a test, the [`Assertions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/Assertions.java) class tracks whether any failures have been encountered. When the call to `Assertions.flush()` is made, this tracker is checked. If any failures are present, execution will be stopped. If no failures are found, execution continues.

Since the tracker is live for the entire duration of a test, there is no limit to how many calls to `Assertions.flush()` can be made throughout a test. It is possible then, to create a series of "checkpoints" in longer tests whenever it is deemed sensible to stop a test if failures have been found. This is especially convenient for smoke to end-to-end tests where a focus on completion of test is important for a full accounting of key validation points.

Please note, however, that at least one call to `Assertions.flush()` is required in tests where only `check()` validations are made. If no call is made, the test will be allowed to continue to completion, and individual steps will be reported as failed (if validations have indeed failed), but the test as a whole will be reported as passing. Since the Qadenz reporter is integrated with TestNG, the `AssertionError` thrown by the `flush()` method in the event of individual failures is required to mark the test itself as failed.

One additional design consideration must be made when mixing `verify()` and `check()` validations within the same test. When a `check()` validation is made, and is followed by a `verify()` validation prior to calling `Assertions.flush()`, if the `verify()` validation fails, the test will be stopped at the failed `verify()` validation.

## Screenshots

Qadenz validations are built to capture screenshots whenever a Condition evaluation fails. If screenshots are desired for validation failures, no special action need be taken. Should screenshots *not* be needed for a validation, disabling is easy with the overloaded `verify()` and `check()` methods.

Adding a call to `Screenshot.SKIP` as the first argument in either `verify()` or `check()` will disable screenshots from being captured if the evaluations for any accompanying Conditions fail.

```
verify(Screenshot.SKIP, Conditions.visibilityOfElement(locator, Expectations.isTrue());
```

A `boolean` could also be passed to achieve the same outcome. The `Screenshot.SKIP` value is intended as a means to keep the resulting code easily readable at a glance.
