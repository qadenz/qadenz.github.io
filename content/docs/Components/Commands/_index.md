---
title: "Commands"
linkTitle: "Commands"
description: >
  Browser interactions with synchronization, logging, and failure diagnostics built into every call.
weight: 3
---
Selenium drives a browser. It clicks, types, selects, switches frames, and reads the DOM, and it does those things well. What Selenium does not do is test.

A test needs more than the raw action. Before it acts it has to know the element is actually ready, or the click lands on nothing. After it acts it needs a record of what happened, so a passing run is legible and a failing one is diagnosable. Selenium supplies none of that on its own. It gives us the verb and leaves the rest to us.

Commands are that rest, supplied once. Every command wraps a browser interaction in the work a real test needs around it.

## The anatomy of a command

Every method on a Commands class does the same four things, in the same order, whether it clicks a button or reads a date off a table:

1. **Log the action** and the name of the target element, so the step names itself on the report.
2. **Initialize the element** through an explicit wait, so the interaction happens only once the element is ready.
3. **Perform the action or inspection** against the freshly located element.
4. **On failure, capture a screenshot and surface the exception**, so a broken step arrives with evidence attached and stops the test rather than limping onward.

Call `click(loginButton)` and all four happen. The wait, the log line, and the screenshot-on-failure come with the click, not as extra lines we remember to write around it.

## The layer someone has to build

None of this is exotic. Any team can wait before interacting, log each step, and screenshot on failure. The catch is that someone has to build that layer, and then keep building it, consistently, on every interaction, for the life of the project.

In practice that goes one of a few ways. A team maintains its own wrapper layer, essentially its own Qadenz, and pays to design, test, and maintain it. Or the features get applied unevenly: waits on the flaky interactions but not the rest, logging where someone remembered, screenshots on some failures and not others. Or they are skipped entirely, and every failure is a stack trace with no picture of the page. Worst of all, they get built wrong, and a subtly broken wait masks real defects or invents flaky ones.

Qadenz has already done that work. The underlying tools are welded together behind a small coding interface, with the heavy lifting finished and refined: the synchronization tuned, the logging shaped into readable output, the failure handling consistent across every command. We call the method and get the behavior.

## Waits are part of the command

Synchronization is the largest single source of flakiness in UI automation, and it is where the built-in behavior earns its keep. Every command initializes its target through an explicit wait before acting. A click waits for the element to be clickable; other commands wait for visibility. There is no separate wait step to remember and no arbitrary `Thread.sleep()` to tune, because the wait is not a thing we add to a command. It is part of the command.

For the cases where a test still needs to wait on a specific state before proceeding, `pause(Condition)` exposes the same wait mechanism directly, expressed in the [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}) vocabulary.

## Fresh elements, fewer stale references

The order of those four steps matters. Each command locates its element immediately before interacting with it, rather than holding a reference captured earlier. A `WebElement` found and stored ahead of time goes stale the moment the DOM around it re-renders, and the next interaction throws `StaleElementReferenceException`. By finding the element at the last possible moment, Commands sidesteps most of that class of failure without the test having to think about it.

## Three surfaces, one foundation

The interactions split by what they touch, and all three sit on a shared, tool-agnostic `Commands` base that supplies the validation and wait methods.

- **[WebCommander]({{< relref "/docs/Components/Commands/webcommander.md" >}})** acts on elements: clicks, text entry, selects, frame switching, and `Actions`-based sequences like hover and double-click.
- **[WebInspector]({{< relref "/docs/Components/Commands/webinspector.md" >}})** interrogates elements: text, attributes, CSS properties, element state, and instance counts, with the ability to return text as numbers or dates.
- **[Browser]({{< relref "/docs/Components/Commands/browser-commands.md" >}})** controls the browser around the DOM: navigation, alerts, cookies, and window focus. Its methods are static and callable from anywhere in the test.

## Extend and build, never clone and change

The commands cover default Selenium interactions plus a set of common conveniences, but no fixed set can anticipate every UI. So `WebCommander` and `WebInspector` are built to be extended. A project subclasses them and adds its own commands, inheriting the same four-step anatomy for free. The [Extensibility]({{< relref "/docs/Components/Commands/extensibility.md" >}}) page walks through the pattern.

Two things make this extend-and-build model work rather than fight it. First, custom commands get the built-in wait, logging, and screenshot behavior automatically, so an extension looks and behaves like a first-class command. Second, extending never walls off the raw tooling: the `WebDriver` remains directly accessible for the rare interaction that needs to go lower. There is no need to fork Qadenz or edit its source to add behavior. We build on top of it and leave it intact.

## What a team gets

- **Interactions that self-synchronize.** The wait is inside the command, so the most common source of flakiness is handled by default rather than by discipline.
- **Reports that read themselves.** Every command logs its action and its target, and every failure carries a screenshot, with no logging code in the test.
- **One consistent behavior.** Every interaction across the suite waits, logs, and fails the same way, because they all flow through the same anatomy.
- **Room to grow.** Custom commands inherit all of it, and the underlying `WebDriver` stays within reach.

## Where to go next

- **[WebCommander]({{< relref "/docs/Components/Commands/webcommander.md" >}})** and **[WebInspector]({{< relref "/docs/Components/Commands/webinspector.md" >}})** catalog the interactions available to act on and read from the UI.
- **[Browser]({{< relref "/docs/Components/Commands/browser-commands.md" >}})** covers browser-level control outside the DOM.
- **[Logging]({{< relref "/docs/Components/Commands/logging.md" >}})** shows how the constructor choice ties each logged step to a page or class.
- **[Extensibility]({{< relref "/docs/Components/Commands/extensibility.md" >}})** builds a custom command that inherits the full anatomy.