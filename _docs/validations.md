---
title: Validations
tags: 
description: Making assertions with Conditions and Expectations
--- 

# Test Validations

Qadenz is built with a versatile and easy to use approach to performing validations against the UI under test. The key component of this approach is the inclusion of `Conditions` and `Expectations` (explained [here]({{ site.baseurl }}{% link _docs/conditions-and-expectations.md %})) as evaluation criteria for validations. These classes combine Selenium `WebElement` inspections to retrieve the 'actual' state of the UI, with Hamcrest Matchers to provide the 'expected' state, to return a result that represents a passing or failing outcome.

# Why not Assertions?

Unit testing frameworks such as TestNG or JUnit include assertion functionality as a core component, and are relatively simple to use. Being open-ended frameworks, however, indiviudal users may tend to express very similar validations in a variety of different assertions. This leads to inconsistent coding patterns, and more difficult maintenance of test code.

Using `Conditions` and `Expectations` allows a team to ensure all contributors are following the same pattern for validations.

That said, Qadenz does employ a single TestNG assertion, the `assertTrue()` method, as a means of validating a Condition. The `result()` of a Condition is a simple representation of whether the state of the UI under test meets expectation. If the output of the Condition evaluation matches the Expectation, `result()` will return `true`. 

By passing this result to the `assertTrue()` method, Qadenz is ensuring that a passing result depends on the Condition evaluation meeting the Expectation. If not, the validation will fail.

# Assertion Types

The concept of Hard Asserts and Soft Asserts are not new in the test auomtion world. Qadenz implements both concepts by way of the `verify()` and `check()` methods.

`verify()` represents a Hard Assertion. If the validation fails, the test will be marked as failed and execution will be stopped.

`check()` represents a Soft Assertion. If the validation fails, the test will be marked as failed, but execution will be allowed to continue until a