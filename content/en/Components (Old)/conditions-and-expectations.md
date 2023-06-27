---
title: "Conditions & Expectations"
weight: 4
description: >
    The pairing of [`Conditions`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/conditions/Conditions.java) and [`Expectations`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/conditions/Expectations.java) becomes the core evaluative logic of Qadenz. These classes are used to determine if the state of the UI under test meets a given criteria.
---

When used with the Explicit Wait provided by the `pause()` command, or with the Assertions provided by the `verify()` and `check()` commands, a Condition is used to describe precisely what on the UI is being evaluated, and the accompanying Expectation describes exactly the expected outcome of the evaluation.

The goal of the Condition/Expectation pairing is to provide a unified structure for making evaluations throughout a testing project, rather than mixing Selenium `ExpectedConditions` calls for Explicit Waits, and various syntax patterns for test assertions. The resulting code is simple to read and quickly understand, and is easily maintained as the application under test evolves.

## How does it work?

Each Condition accepts a variety of Expectations based on the type of evaluation being performed. For example, Conditions pertaining to element state (visibility, selected, enabled) accept boolean-based Expectations, while Conditions evaluating text will require String-based Expectations.

Conditions and Expectations are powered by Hamcrest. The return value of a Condition is based on whether the outcome of the evaluation matched the expected result. The Condition will return `true` is there was a match, `false` otherwise.

## Logging evaluations

Implementations of the [`Condition`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/conditions/Condition.java) interface will provide a description of the Condition under evaluation to the logs and subsequent report output. Should the evaluation fail to match with the Expectation, the Condition will also provide a summary of the actual results.

Implementations of the [`Expectation`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/conditions/Expectation.java) interface will provide a description of the Expectation that will be added to the Condition logs, as well as the wrapped Hamcrest Matcher that will be used to compute the result of the Condition.
