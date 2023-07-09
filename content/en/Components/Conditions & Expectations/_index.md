---
title: Conditions & Expectations
description: >
  Conditions & Expectations
weight: 4
---

The pairing of Conditions and Expectations becomes the core evaluative logic of Qadenz. These classes are used to determine if the state of the UI under test meets a given criteria.

The goal of the Condition/Expectation pairing is to provide a unified structure for making evaluations throughout a testing project, rather than mixing Selenium `ExpectedConditions` calls for Explicit Waits, and various syntax patterns for test assertions. The resulting code is simple to read and quickly understand, and is easily maintained as the application under test evolves.

The evaluative logic behind Conditions and Expectations aims to answer the questions, "What is the current state of the UI?" and "What do I expect to see?". Conveyed in terms familiar to testers, "What is the ACTUAL outcome?" and "What is the EXPECTED outcome?".

A `Condition` describes a specific criteria to be evaluated on the UI. This could be the visibility of one or more elements, or the text shown in an element, for example. The `Condition` is used to establish the "actual outcome" portion of the evaluation. An `Expectation`, then, is required for each `Condition`. The `Expectation` will describe the "expected outcome" portion of the evaluation.

## How does it work?

Each `Condition` uses WebDriver commands to retrieve data from, or information about, elements on the UI. Each `Expectation` invokes a Hamcrest `Matcher` that is used for the evaluation, which is passed to the `Condition`. If the the value retrieved by the `Condition` matches the value given on the `Expectation`, the `Condition` result will return `TRUE`.
