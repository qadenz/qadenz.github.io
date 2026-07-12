---
title: "Browser"
linkTitle: "Browser"
description: >
  Static commands for controlling the browser around the DOM: navigation, alerts, cookies, and window focus.
weight: 3
---
The [`Browser`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/commands/Browser.java) class manages the browser itself, as opposed to the elements rendered inside it. Navigation, alert handling, cookie management, and switching between windows all happen at the browser level, outside the DOM of the application under test. Those actions live here rather than on the `WebCommander`.

Like the element commands, each `Browser` method logs the action it performs and, on failure, captures a screenshot before surfacing the exception. What these methods do not carry is an element wait, because they do not act on elements. There is no `Locator` to initialize and no visibility to wait on.

## Static by design

Every method on `Browser` is static. Call them directly on the class, from anywhere in the test:

```java
Browser.openUrl("https://qadenz.dev");
```

Browser-level actions are not tied to any element or page, so there is no reason to instantiate anything to reach them. A test navigates, a Page Object dismisses an alert, a setup method clears cookies, all through the same static entry point, without threading an instance through the layers that need it.

## Navigation

Open a URL, move through history, and refresh the current page.

- `openUrl(String url)` loads the given URL in the current window.
- `goBack()` navigates to the previous page in history.
- `goForward()` navigates to the next page in history, if one exists.
- `refreshPage()` reloads the current page.

## Alerts

Native JavaScript alerts sit outside the DOM and cannot be read or clicked as elements. The `Browser` handles them directly.

- `acceptAlert()` accepts the alert, the equivalent of clicking OK.
- `dismissAlert()` dismisses the alert, the equivalent of clicking Cancel.
- `enterTextOnAlert(String input)` types into a prompt-style alert that accepts input.

## Cookies

- `deleteCookies()` clears all cookies for the current session. This is a common way to reset authentication or application state between tests without restarting the browser.

## Windows

When an action opens a second window or tab, focus stays on the original until it is moved. These methods shift focus relative to the current window.

- `focusOnNextWindow()` moves focus to the next window.
- `focusOnPreviousWindow()` moves focus to the previous window.

Both resolve their target from the current window's position in the ordered list of open handles, so the test moves through windows in sequence rather than juggling handle strings by hand.

## Closing

- `closeBrowser()` closes the current browser window.
