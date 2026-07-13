---
title: "UI Modeling"
linkTitle: "UI Modeling"
description: >
  Name the UI once, in the language of the application, and let tests read against the names instead of the selectors.
weight: 2
---
A test knows the application in human terms. It signs in, it searches for an item, it adds the item to a cart. The browser knows the application as a tree of nodes addressed by CSS. UI modeling is the layer that translates between the two: it names each part of the page once, so the rest of the suite can speak in the language of the application and never in raw selectors.

Selenium already has an answer here, the `PageFactory` and its `@FindBy` annotation. Qadenz takes a different one, and it is worth being clear about why.

## What a mapped element has to carry

Give a name to the "Sign In" button and something has to hold onto it. Consider what that something is asked to do over the life of a suite:

- **Read back as the "Sign In" button**, in every log line and every report, not as `.btn-signIn`. When a step fails, the report should name the element a human would recognize, so nobody has to translate a selector back into a place on the page.
- **Stretch to cover more than one element.** The third search result and the tenth are the same kind of thing at different coordinates. One mapping should reach any of them, rather than a separate field per instance.
- **Reuse the selector fragments it shares** with its neighbors. Every control inside an item card sits under the same card selector. That shared prefix should be written once.
- **Stay fresh.** The DOM re-renders, and any element reference captured ahead of time goes stale. Whatever holds the mapping should not bind to a live node until the moment of use.

`@FindBy` binds a field to a selector and stops there. It has no display name, no room for a runtime parameter, no way to share a fragment with the next field, and it resolves eagerly into an element that the next re-render can invalidate. It answers the first question, "which node," and none of the others.

## The Locator is a value, not a binding

Qadenz maps an element with a [`Locator`]({{< relref "locators.md" >}}): a small value that pairs a display name with a selector.

```java
Locator signInButton = new Locator("Sign In Button", ".btn-signIn");
```

Because it is a plain value and not a bound element, the same four demands fall out of it naturally. The name rides along into every log line, so the report reads in the application's own words. A method can build a `Locator` from an argument, so one mapping covers a whole family of elements. A `Locator` can name a parent, so a shared selector fragment is written once and reused. And nothing is located yet: a `Locator` is a description of how to find an element, resolved into an actual node only when a command acts on it. Qadenz follows [Simon Stewart's advice](https://www.youtube.com/watch?v=gyfUpOysIF8&t=1519s) and leaves the `PageFactory` behind to find a better way.

That last point is the hinge between this section and [Commands]({{< relref "/docs/Components/Commands/_index.md" >}}). A `Locator` never holds a `WebElement`. When a command receives one, the [`WebFinder`]({{< relref "webfinder.md" >}}) resolves it against the live DOM at that instant, waits for it if the command calls for a wait, and discards it after. The model is a set of descriptions; the elements are momentary. This is why a Qadenz page object does not go stale between steps: there is nothing held to go stale.

## A page is a collection of Locators

A page object in Qadenz gathers the `Locator`s for one screen and exposes the actions a test takes there. It holds no `WebElement` fields and does its own waiting through the commands, so a page class is mostly a map of names to selectors plus a handful of methods.

```java
public class SignInPage {

    private WebCommander commander = new WebCommander(getClass());

    private Locator usernameField = new Locator("Username Field", "#username");
    private Locator passwordField = new Locator("Password Field", "#password");
    private Locator signInButton = new Locator("Sign In Button", ".btn-signIn");

    public void signIn(String username, String password) {
        commander.enterText(usernameField, username);
        commander.enterText(passwordField, password);
        commander.click(signInButton);
    }
}
```

The test that drives this page never sees a selector. It calls `signIn(...)`, the page speaks in `Locator` names, and the commands turn those names into waited, logged, screenshot-on-failure interactions. How to structure these classes at scale, one command per method, a shared base page, and where the `WebCommander` lives, is a topic for a Guides walkthrough rather than this reference.

## The pieces

- **[Locators]({{< relref "locators.md" >}})** map a single element: display name, selector, runtime parameters, parent chaining, and the state attributes that harden inspections against unconventional UIs.
- **[Selectors and Sizzle]({{< relref "selectors-and-sizzle.md" >}})** covers the CSS dialect Qadenz selectors are written in, including the Sizzle pseudo-classes like `:contains()` and `:eq()` that make parameterization work.
- **[Locator Groups]({{< relref "locator-groups.md" >}})** collect several `Locator`s into one named unit, so a whole component can be verified in a single call.
- **[WebFinder]({{< relref "webfinder.md" >}})** is the class that turns a `Locator` into a `WebElement` at the moment of use, and the reason the model stays fresh.