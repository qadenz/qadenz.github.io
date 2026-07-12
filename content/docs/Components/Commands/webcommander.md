---
title: "WebCommander"
linkTitle: "WebCommander"
description: >
  The commands that act on elements: clicks, text entry, selects, frames, and Actions-based sequences.
weight: 1
---
The `WebCommander` performs actions against elements: clicking, entering text, selecting options, switching frames, and driving `Actions`-based sequences. It is the class a test reaches for whenever it needs to *do* something to the UI.

Every method follows the [four-step anatomy]({{< relref "/docs/Components/Commands/_index.md" >}}) shared by all commands: it logs the action and the target element, initializes the element through an explicit wait, performs the interaction, and on failure captures a screenshot before surfacing the exception to stop the test. The rest of this page covers what each command does, not that boilerplate, because the boilerplate is the same everywhere.

`WebCommander` extends the tool-agnostic `Commands` base, which is where the validation methods (`verify`, `check`) and the `pause` wait live. Those are documented alongside the vocabulary they use, under [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}).

## Creating a WebCommander

A `WebCommander` is instantiated where it is used, whether that is directly in a test or inside a UI-modeling layer such as a page object. Which of its two constructors to reach for is the first decision, and it comes down to how much detail the logs should carry.

Use the no-argument constructor when commands are called straight from the test, or when the UI is simple enough that tracing each action to a specific page is not worth the ceremony. Every logged step is attributed to the `WebCommander`.

```java
WebCommander commander = new WebCommander();
```

Use the `Class<?>` constructor when a page object or other UI-modeling layer is in place and each step should be attributed to the page it happened on. Pass the consuming class, and its name stands in as the source of every logged step.

```java
WebCommander commander = new WebCommander(getClass());
```

However it is instantiated, each command reads as the step it performs:

```java
commander.enterText(usernameField, "admin@qadenz.dev");
commander.enterText(passwordField, "Test123$");
commander.click(signInButton);
```

Where those calls live, whether directly in a test or wrapped in page-object methods, is a design choice for the consuming project rather than a constraint of the API.

The two constructors produce visibly different reports. See [Logging]({{< relref "/docs/Components/Commands/logging.md" >}}) for a side-by-side comparison of the output.

## How the wait is chosen

The explicit wait built into element initialization is not identical for every command. It is matched to the interaction:

- Commands that click wait for the target element to be **clickable**.
- Commands that upload a file wait for the input to be **present**, since file inputs are frequently hidden.
- All other commands wait for the target element to be **visible**.

## Element commands

The four standard [Selenium element interactions](https://www.selenium.dev/documentation/webdriver/elements/interactions/) are covered directly: `click`, `enterText`, `clear`, and `select`.

### Clicks

The primary `click(Locator)` method includes a fallback for tricky DOM configurations. If a click is intercepted by another element and Selenium throws `ElementClickInterceptedException`, the method re-locates the element and reattempts the click through the `Actions` API. This behavior is enabled by default and is controlled by a TestNG Suite XML parameter:

```xml
<parameter name="retryInterceptedClicks" value="false" />
```

An intercepted click is often a symptom of a `Locator` that needs tightening, so address the selector before relying on the fallback.

An overloaded `click(Locator, int xOffset, int yOffset)` places a point-precise click on an element using X and Y offsets. Because this variant wraps the `Actions` class, it does not include the interception fallback.

### Inputs

`enterText(Locator, CharSequence...)` retains the flexibility of the underlying `WebElement.sendKeys()`, accepting both text and enumerated `Keys` in a single call. Its logging is shaped to render both cleanly on the report.

```java
commander.enterText(searchField, "qadenz", Keys.ENTER);
```

`clearAndEnterText(Locator, String)` clears a field and enters text in a single call, saving the separate `clear` step.

`clear(Locator)` empties an input field.

### Selects

`select(Locator, String)` wraps Selenium's `Select` API to interact with menus built as `<select>` elements, and `deselect(Locator, String)` reverses it. Both operate by the visible text of the option, and both offer a varargs overload for acting on several options at once. To select by index or by value, [extend the `WebCommander`]({{< relref "/docs/Components/Commands/extensibility.md" >}}) with a custom command.

### Files

`uploadFile(Locator fileInput, String fileName)` uploads a file to a file `<input>`. The file is resolved from the project resources by name, and a `LocalFileDetector` is set so the upload works against a remote or grid-hosted browser as well as a local one.

## Actions sequences

The `Actions` API simulates keyboard, mouse, pen, and wheel input. Its builder pattern makes wholesale wrapping impractical and of little value, so complex sequences are best built as [custom commands]({{< relref "/docs/Components/Commands/extensibility.md" >}}) tailored to the UI under test. The `WebCommander` provides wrappers for the most common sequences:

- `doubleClick(Locator)` double-clicks an element.
- `controlClick(Locator...)` clicks each of several elements while holding the CTRL key, for multi-select interactions.
- `hover(Locator)` moves the pointer over an element, for menus and tooltips that respond to hover.

## Frames

Content inside an `<iframe>` is only reachable once focus moves into that frame.

- `focusOnFrame(Locator)` moves focus into the frame mapped by the given `Locator`.
- `focusOnDefaultContent()` returns focus to the main document.

## Waits

Beyond the waits already built into each command, `pause(Condition)` waits for a specific state to be reached before the test proceeds. It functions as an explicit wait, but expresses the wait using the [Conditions & Expectations]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}) vocabulary, evaluating the `Condition` repeatedly until it is satisfied or the timeout expires.

## Screenshots

Failed validations and caught exceptions capture a screenshot automatically. When a test needs visual confirmation of the UI at a specific point, or an extra frame while troubleshooting, `captureScreenshot()` saves an image of the visible UI and embeds it in the HTML report.