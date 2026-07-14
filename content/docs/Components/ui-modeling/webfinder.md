---
title: "WebFinder"
weight: 4
description: >
  The class that turns a Locator into a live WebElement at the moment of use, and the reason a Qadenz UI model never goes stale.
---

`WebFinder` is where a `Locator` becomes a `WebElement`. It is the one piece that resolves a mapping (a name and a selector, holding no live element) into an actual node on the page. Tests and page objects rarely call it directly, but it is worth understanding, because it is what makes the [model-is-data]({{< relref "_index.md" >}}) approach hold together, and it is the class a custom command reaches for when extending the command set.

## On-the-fly element initialization

When a `Locator` is passed to a command, `WebFinder` locates the element from the `Locator`'s selector immediately before the command acts on it. This happens for every interaction, inspection, and validation, each time, against the live DOM at that instant. Nothing is located ahead of time and held.

This late resolution is the defense against `StaleElementReferenceException`. A `WebElement` captured earlier goes stale the moment the surrounding DOM re-renders; a `Locator` cannot, because it is only a description until the command runs. Resolving at the last possible moment also sharpens failure reporting: when a lookup or interaction fails, the problem is caught around a single, named element and surfaced in the logs and reports with that name attached.

## To wait, or not to wait

`WebFinder` is the only class in Qadenz that uses Selenium's `ExpectedConditions`. Its methods resolve a `Locator` in one of two modes: immediately, or after waiting for a specific state.

| Method | Resolves | Wait |
|--------|----------|------|
| `find` | first match | none, immediate |
| `findAll` | all matches | none, immediate |
| `findWhenVisible` | first match | until visible |
| `findAllWhenVisible` | all matches | until all visible |
| `findWhenClickable` | first match | until visible and enabled to receive a click |
| `findWhenPresent` | first match | until present on the DOM |

The waiting methods poll for the element roughly every 500 milliseconds up to the configured [timeout]({{< relref "/docs/Components/Configuration/_index.md" >}}). If the state is never reached before the timeout, the method throws and the test stops.

## Which wait the commands use

The command classes pick a `WebFinder` method to match what the interaction needs, so a test never chooses a wait itself:

- **`WebCommander`** resolves through `findWhenClickable` for actions that perform a click, since a click needs an element that is both visible and enabled. Every other interaction resolves through `findWhenVisible`.
- **`WebInspector`** resolves through `findWhenVisible` before reading a single element. The exception is methods that inspect multiple instances of a `Locator`, which resolve through `findAll` with no wait, since they measure whatever is present at that moment.

A [custom command]({{< relref "/docs/Components/Commands/extensibility.md" >}}) chooses among these same methods when it needs to locate its target, inheriting the same synchronization behavior as the built-in commands.