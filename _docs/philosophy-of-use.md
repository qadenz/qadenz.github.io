---
title: The Qadenz Approach
tags: 
description: Understanding Qadenz
--- 

# That space between...

Very seldom will teams forge forward with writing tests using raw Selenium, joined with a unit testing framework and perhaps a data library (JDBC, Apache POI, etc). They might even have a basic Page Object model in place to abstract repeated code. At best, and most rarely, this type of team is typically running like an extremely well oiled machine with highly disciplined developers writing detailed code with a consistent implementation. Usually, this type of team may be in a Wild West environment where anything goes as far as design, and little attention is paid to things like maintenance. More than likely, a this type of team may very well be plowing full steam toward a brick wall, and may not realize it.

The concept of a wrapper-layer to unify the implementation of underlying tools within a test automation environment is nothing new. Fortunately, most teams come to realize that frameworks like Selenium, TestNG or even JUnit, are indeed open ended frameworks designed as such very intentionally to allow it's users to tailor their implementation of these tools to their own needs. 

This commonly leads many teams to design even a basic wrapper-layer for the sake of convenience alone, simply to reduce the coding effort. Some will take the time to answer the question "What is the best way for **our team** to implement these tools?" It's not uncommon for teams to build out a custom toolset at this layer and over time see a varied degree of success. It's certainly not uncommon for consulting companies and managed services vendors to offer their own "framework" to their clients as either a value-add or a means of retaining clients for a longer term via maintenance contracts, again with varied degrees of success on the long term.

At the end of the day, it becomes about development efficiency and code maintainability. How quickly can we automate this next block of tests? Is the tooling reliable enough to ensure repeatable executions? Are the implementation details tribal knowledge, or is the code clear enough that a new developer can easily and quickly ramp up for maintenance? If the answers to these questions leads to increased efficiency and improved maintainability, then the investment in these tools becomes quite worthwhile.

# The Qadenz Philosophy of Use

Qadenz was designed to exist on top of tools like Java, Selenium, TestNG, and Hamcrest, to provide one way of implementing these tools in a meaningful and beneficial way to the test project it powers. This ensures all the contributors to a test project are making the same design considerations in their tests, following the same coding patterns, and ultimately writing cleaner and more manageable tests.

To these ends, Qadenz targets a number of core objectives...

## Decouple Tests from Configuration

By separating the actual tests from the setup and tear-down, tests will receive the appropriate focus. The configuration can then be abstracted and maintained from a central location. Tests are more easily organized, and are not obfuscated by boilerplate.

## Reduce coding overhead by doing the heavy lifting

Selenium interactions can, with one method call, log the activity, initialize an element, perform an interaction against the element, handle exceptions with logging and reporting, and capture screenshots. By offloading this work to the library, users can focus on scripting their tests rather than sorting out all the supporting functionality. 

## Encourage efficient UI Modeling

Qadenz uses CSS Selectors exclusively due to maximized flexibility and easy of use. This is further enhanced by injecting SizzleJS to enable parameterized element selectors. CSS Selectors are versatile, simple to understand, and most importantly (well crafted selectors are) more resistant to change compared to XPath. Wrapping the element selectors on the Locator object also enhances reporting with friendly element names, and encourages teams to move beyond antiquated and inefficient patterns. Page Factory is old and busted. Simon even says so.

## Provide flexible and precise validations

The Conditions-based validations functionality is custom to Qadenz and enables high levels of precision using the What/How relationship of Conditions and Expectations. Additionally, the Verify/Check language in the API allows for hard and soft assertions, which enables the user to decide exactly when a test will be halted following a failure. Finally, the iterative nature of Qadenz validations allows for multiple assertions to be made in groups, ensuring all validations are made regardless of those that fail.

## Include detailed and easily shared reports

TestNG provides basic reporting out of the the box. The `emailable-report.html` is nice for basic pass/fail results, but leaves much to be desired when it comes to detail. Other reporting libraries add a lot of noise to the code, and don't tend to coexist with standard logging frameworks. The Qadenz reporter is powered by Logback, is fully integrated with TestNG, and produces a singular HTML output file that contains full detailed results of each test and embedded screenshots of errors and failed validations.
