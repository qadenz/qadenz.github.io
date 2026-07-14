---
title: "Test Automation with Qadenz"
linkTitle: "Documentation"
type: "docs"
---

Qadenz is a Java test automation library. It wraps [Selenium](https://www.selenium.dev/) for browser control, [TestNG](https://testng.org) for suite management, and [Hamcrest](http://hamcrest.org/JavaHamcrest/) for validations behind one small, opinionated API.

Any team automating against a browser needs a layer between its tests and those tools: synchronization so an interaction does not race the page, logging and reporting so a run is legible, a way to model the UI that does not go stale. Building that layer, and then maintaining it, is often as much work as the tests it supports.

## Turn-key on top of Selenium

Teams automating a browser tend to land in one of two places. The ready-made tools install and run, and a team writes tests the same day. Selenium asks more: a project builds its own wrapper layer around it first, then maintains that layer as an internal product, paying for both long before a test ships.

Qadenz is a third option. It is the wrapper layer, already built, maintained, and shaped by years of real automation work, so a Java team gets the ready-made start without giving up Selenium underneath: the W3C-standard engine, the real browser, and the accuracy that comes with them.

The [Philosophy of Use]({{< relref "/docs/philosophy-of-use.md" >}}) makes the full case.

## What sets Qadenz apart

Most of what a framework does, other frameworks also do in some form. Two things Qadenz does that they generally do not:

- **One vocabulary for waiting and asserting.** Selenium hands a team two separate languages: `ExpectedConditions` for waits, and a separate assertion library for checks. Qadenz replaces both with a single vocabulary of Conditions and Expectations that describes the state of the UI once, then drives waits, hard assertions, and soft checks alike. A grouped check reports every failure in a step instead of stopping at the first, and each one reads in plain language on the report. See [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}).
- **Reports that are the evidence, not the score.** Qadenz records the full step-by-step story of every test and embeds a screenshot at each failure, in one self-contained HTML file that shares without a zip. It does this with no reporting code in the tests or page objects: the report is built from the same log output the commands already produce. Most tools give a pass/fail count and a stack trace, or ask a team to thread reporting calls through its own code to get more. See [Test Results]({{< relref "/docs/Components/Test-Results/_index.md" >}}).

## The fundamentals, handled

The everyday parts are here too, built to the same standard:

- **Commands** wrap every Selenium interaction in the wait, the logging, and the screenshot-on-failure a real test needs, so a click is never just a click. See [Commands]({{< relref "/docs/Components/Commands/_index.md" >}}).
- **UI Modeling** names each element once, in the language of the application, with a `Locator` that holds no live element and never goes stale. See [UI Modeling]({{< relref "/docs/Components/ui-modeling/_index.md" >}}).
- **Configuration** stands the project up from a base class and a few Suite parameters, with a fresh browser per test and safe parallelism. See [Configuration]({{< relref "/docs/Components/Configuration/_index.md" >}}).

## Getting started

[Getting Started]({{< relref "/docs/getting-started.md" >}}) builds a first test from an empty Maven project through a passing run and its report. For the reasoning behind the design, and the positions Qadenz takes, read the [Philosophy of Use]({{< relref "/docs/philosophy-of-use.md" >}}).

The source is on [GitHub](https://github.com/qadenz/qadenz).

## License

The Qadenz library is made available under the [PolyForm Internal Use License](https://polyformproject.org/licenses/internal-use/1.0.0/) as a Source Available library. Teams are welcome to use Qadenz to power their internally managed test automation projects and to modify it as needed, but may not redistribute the library or market or sell Qadenz (or derivative works) as a product for their own customers.

Happy automating :)