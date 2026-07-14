---
title: "Philosophy of Use"
linkTitle: "Philosophy of Use"
weight: 1
description: >
    Why Qadenz is built the way it is, and the positions it takes.
---

Most teams that automate against a browser end up building an intermediate layer between their tests and the underlying tools. It abstracts the common code, unifies how the team works, and answers the question of how, for this team, these tools should be used. Designing and maintaining that layer is a real and ongoing effort, often as large as keeping up with the demand for new tests. Qadenz exists so a team does not have to take it on, and it takes deliberate positions along the way. This page makes the case for them.

## Turn-key on top of Selenium

Every architect weighing an automation stack asks the same thing: why this over the ready-made alternatives that install and run out of the box? It deserves a direct answer.

Those tools win teams over with exactly that readiness. Install, and write tests the same afternoon. A Selenium project has rarely been able to make that claim, and that ease, more than any feature list, is what draws teams to the newer options.

The reason is structural. Selenium is a browser automation tool, not a test framework. On its own it clicks, types, and reads the DOM, and does that well, but everything a real test needs around those actions, the synchronization, the logging, the reporting, the UI modeling, is left to the team to build. A Selenium team becomes the maintainer of an internal product: the wrapper layer that turns Selenium into something a suite can be written against. That layer costs real time to build before the first test, and real effort to maintain for the life of the project.

This is the ground Qadenz stands on. It is that wrapper layer, already built and maintained, so a Java team starts where a ready-made tool's users start, writing tests on day one, but on Selenium underneath.

And Selenium is the engine worth keeping. The WebDriver protocol is a [W3C standard](https://www.w3.org/TR/webdriver2/), implemented by the browser vendors themselves, so Selenium drives the real, shipping browser through the interface its own maker supports. Many of the newer tools reach their out-of-the-box ease partly by shipping and driving their own browser builds, where Selenium drives the stock browser a user actually runs. For a team that values standards compliance and tests that exercise the real browser, that is the foundation worth keeping. Qadenz keeps it, and delivers it ready to use.

None of the choices that follow are arbitrary. Qadenz did not arrive at its defaults on a whiteboard; each is a lesson carried out of years of building and maintaining real automation. The explicit waits are there because implicit waits fail quietly at scale. The fresh browser per test is there because shared sessions leak state. Elements resolve at the last moment because a held reference is the most common source of a flaky suite. The opinions Qadenz holds are the ones a team tends to reach the hard way, already made and built in.

## Where Qadenz goes further

Two capabilities set Qadenz apart from the frameworks a team might build or adopt, because both replace work a team would otherwise do by hand.

### One vocabulary for waiting and asserting

Selenium asks a team to learn two ways of describing the same UI. Waiting for an element uses `ExpectedConditions`; asserting on it uses a separate assertion library, with its own syntax and its own idea of what a value is. The two overlap constantly: a test waits for text to be present, then asserts the text is correct, in two different languages, and a team maintains both.

Qadenz collapses them into one. A Condition names what to look at, an Expectation names how to evaluate it, and the pair describes a piece of the UI's state a single way. That same pair drives an explicit wait through `pause()`, a hard assertion through `verify()`, and a soft assertion through `check()`. Learn the vocabulary once, and it reads the same whether a test is waiting or judging. A grouped `check()` evaluates every Condition and reports all the failures in a step rather than stopping at the first, so one run surfaces the full picture of what broke. Because the vocabulary is precise, every evaluation narrates itself on the report in plain language. See [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}).

### The report a team would otherwise build by hand

Most reporting comes in one of two disappointing shapes. The runner's own report gives a pass/fail count and a stack trace, enough to know something broke, not enough to know what happened, and any screenshots arrive as a folder to zip and pass around. Or a team adopts a reporting library and pays for the detail in coupling: reporting calls threaded through the tests and the page objects, automation logic tangled with the tool that describes it, and the richest features behind a license.

Qadenz treats a test's real output as the evidence it leaves behind. Every step a test takes is already logged, and the report is built from that same stream, so the full story of every test, with a screenshot captured at each failure, lands in one self-contained HTML file with nothing to instrument and nothing to unpack. Results sort the way a team triages them, and Qadenz goes a step past the runner by separating a genuine assertion failure from an unexpected error, because the two are different problems and a team reads them differently. See [Test Results]({{< relref "/docs/Components/Test-Results/_index.md" >}}).

## Opinionation

Qadenz is opinionated on purpose. Some of its choices exist to steer a team onto one path rather than leave the decision open, because a single consistent approach across a project is easier to read, review, and maintain than a mix of styles. A few of those positions are worth stating plainly.

**CSS selectors, and only CSS.** Every element in a Qadenz project is mapped with a CSS selector, and nothing else. One selector language expresses what IDs, names, and DOM traversal would, so a team reads and reviews the same syntax everywhere instead of choosing a style per element. Sizzle, injected into the page, adds the pseudo-classes that make selectors parameterizable, so a single mapping can reach a whole family of elements. See [Selectors and Sizzle]({{< relref "/docs/Components/ui-modeling/selectors-and-sizzle.md" >}}).

**Locators over PageFactory and `@FindBy`.** Qadenz treats these as patterns to leave behind, not options to offer. `PageFactory` is an overly complex way to instantiate page objects. `@FindBy` cannot parameterize, and keeping it reliable against a moving DOM means writing extra code to handle the `StaleElementReferenceException` it invites. A `Locator` is a plain value, a name and a selector, that holds no element at all: it parameterizes, it carries a friendly name into the logs, and because it holds nothing live, it never goes stale. This is a pattern the people who built WebDriver have themselves urged teams to move past. See [UI Modeling]({{< relref "/docs/Components/ui-modeling/_index.md" >}}).

**No Gherkin, no BDD layer.** Tools like Cucumber insert a translation layer between plain-language steps and the code that runs them, and that layer is overhead: glue code to write and maintain, a level of indirection to debug through, and a vocabulary that rarely earns its keep once the demo is over. Qadenz keeps tests in Java, readable on their own terms, so the thing a team writes is the thing that runs. Readable tests come from a clear API, not from a second language layered on top of the first.

**One browser session per test.** The `WebDriver` is scoped to the test method, so every test runs on a browser of its own. It costs a little startup time and buys isolation: tests do not inherit each other's state, failures do not cascade from a poisoned session, and a suite parallelizes safely. See [Configuration]({{< relref "/docs/Components/Configuration/_index.md" >}}).

## What it comes down to

Efficiency and maintainability decide whether an automation effort succeeds. How fast can the next block of tests be written? Does the tooling run the same way every time? Can a new engineer read the suite and maintain it, or is it tribal knowledge locked in the heads of the people who wrote it? Qadenz is built so those answers come out in a team's favor. The framework is the part already solved, so the work that remains is the work that matters: writing tests.
