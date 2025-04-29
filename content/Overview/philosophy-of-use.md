---
title: "Philosophy of Use"
linkTitle: "Philosophy of Use"
weight: 1
---
The role in a test automation project of an intermediate API that wraps frameworks such as Selenium and TestNG is far from uncommon. Many teams realize the importance of an API layer for these underlying tools as a means of simply abstracting common code and building re-usable components for frequently utilized features of the base frameworks. Some teams will take this a step further and design a wrapper layer that works to unify the development approach and adopted design patterns within the test project, thus answering the question "What is the best way for **our team** to implement these tools?"

Unfortunately, we as testers frequently find ourselves in fast-paced and ever-evolving environments. Designing, building, and importantly, maintaining an API such as this can be as significant an effort as keeping up with the demands for expanding test coverage.

## Objectives

To this end, Qadenz targets a number of core objectives...

- **Drive easy to read tests with a clearly worded API:** Simply, readable code is manageable code. Everything from the API syntax to the logging and reporting output has been carefully chosen to maximize readability and to be as simple to understand as possible.
- **Reduce coding overhead by doing the heavy lifting:** Selenium interactions with Qadenz can, with one method call, log the activity, initialize an element, perform an interaction against the element, handle exceptions with logging and reporting, and capture screenshots. By offloading this work to the library, users can focus on scripting their tests rather than sorting out all the supporting functionality.
- **Encourage efficient UI Modeling:** Qadenz uses CSS Selectors exclusively due to maximized flexibility and easy of use. This is further enhanced by injecting SizzleJS to enable parameterized element selectors. CSS Selectors are versatile, simple to understand, and most importantly (well crafted selectors are) more resistant to change compared to XPath. Wrapping the element selectors on the Locator object also enhances reporting with friendly element names, and encourages teams to move beyond antiquated and inefficient patterns. Page Factory is old and busted. Simon even says so.
- **Provide flexible and precise validations:** The Conditions-based validations functionality is custom to Qadenz and enables high levels of precision using the What/How relationship of Conditions and Expectations. Additionally, the Verify/Check language in the API allows for hard and soft assertions, which enables the user to decide exactly when a test will be halted following a failure. Finally, the iterative nature of Qadenz validations allows for multiple validations to be made in groups, providing a more complete picture of test failures and maximized potential for quickly exposing system issues.
- **Include detailed and easily shared reports:** The `emailable-report.html` provided by TestNG is nice for basic pass/fail results, but leaves much to be desired when it comes to detail and user experience. Other reporting libraries add a lot of noise to the code, and don't tend to coexist with standard logging frameworks. The Qadenz reporter is powered by Logback, is fully integrated with TestNG, and produces a singular HTML output file that contains full detailed results of each test and embedded screenshots of errors and failed validations.

## Opinionation

Qadenz can be considered an opinionated library. Some features and solutions have been designed specifically to guide users into a particular pattern of usage based on certain best practices, or simply to speed up the development process for testers by choosing a particular path over another. While each is explained in greater detail elsewhere in the documentation, these points are the major areas of opinionation within the Qadenz library:

- **The WebDriver is scoped to the test method:** By limiting the scope of the `WebDriver` session, Qadenz ensures that a fresh browser is used for each test case. This also has provided an avenue to tune the execution workflow.
- **Explicit Waits over Implicit Waits:** Explicit Waits allow for precise conditions to be met within a given amount of time. Implicit waits are only time-based, and cause problems when used alongside Explicit Waits.
- **Sizzle CSS Selectors only:** CSS Selectors are fast, extremely versatile, and resistant to DOM change. Adding Sizzle increases their versatility and adds a number of pseudo-classes that improve the ability to parameterize element selectors. This approach is a unified strategy that ensures all contributors to a test project are following the same pattern, which further enhances the ease of code maintenance.
- **Conditions-powered validations and Explicit Waits:** The `Conditions` and `Expectations` consist of precise and clearly defined evaluations. These provide a unified alternative to both sorting through the variety of assertion types or `ExpectedConditions` for Explicit Waits. Additionally, these classes ensure consistent and detailed information are passed to the logs and report output.
- **Locator-based UI Modeling over Page Factory & @FindBy:** `PageFactory` is an overly complex form of instantiating page objects. `@FindBy` does not support parameterization and mitigating issues like the `StaleElementReferenceException` requires additional code in order to tune and make reliable for a given web application. Locators support parameterization, provide a context-friendly element name for the logs, and a reference to a UI element that is later used by the `WebCommander` or `WebInspector` to initialize an element at the appropriate time of use.

## A Turn-Key Solution

In the end, development efficiency and code maintainability are major success factors for automation teams. How quickly can we automate this next block of tests? Is the tooling reliable enough to ensure repeatable executions? Are the implementation details tribal knowledge, or is the code clear enough that a new developer can easily and quickly ramp up for maintenance? If the answers to these questions leads to increased efficiency and improved maintainability, then the investment in these tools becomes quite worthwhile.

Qadenz is built to satisfy the need for such tool-set, and allow testers to focus on the more important task... build tests quickly.
