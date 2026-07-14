---
title: "Selectors and Sizzle"
weight: 2
description: >
  The CSS selector dialect Qadenz locators are written in, and the Sizzle engine that adds the pseudo-classes parameterization relies on.
---

Every `Locator` selector is a CSS selector. Qadenz commits to a single strategy across the whole suite, so a team writes and reads one kind of selector rather than switching between IDs, CSS, and XPath case by case.

## Why CSS, exclusively

Qadenz uses CSS selectors, and only CSS selectors. Every element in a suite is mapped the same way, so there is one convention to learn, read, and review, rather than a per-element choice among IDs, CSS, XPath, and name attributes.

That exclusivity works because CSS is versatile enough to absorb the alternatives. Anything the other strategies express, a CSS selector expresses too:

- An ID is `#elementId`.
- A name attribute is `[name="elementName"]`, and any other attribute follows the same form.
- XPath's parent-to-child traversal is a walk down the DOM tree with combinators: `.parent > .child` for a direct child, `.ancestor .descendant` for any depth.

Since CSS covers the same ground as the other strategies, no element forces a team off the standard, and there is no reason to mix approaches. Performance reinforces the choice, as browsers evaluate CSS selectors faster than XPath and support them more uniformly. The result is one selector language, applied everywhere.

The one thing native CSS lacks is a way to select on text content or positional index, the two things UI tests reach for constantly. That gap is what Sizzle fills.

## Sizzle

Qadenz resolves selectors through [Sizzle](https://github.com/jquery/sizzle), the selector engine originally built for jQuery, rather than the browser's native `querySelector`. On first use against a page, Qadenz injects the Sizzle script (version 2.3.10) onto the DOM, then runs every selector through it. Because of this, a Qadenz selector is a Sizzle selector: it accepts everything native CSS does, plus a set of pseudo-classes that native CSS does not have.

Sizzle and Selenium go back further than this. Before WebDriver, Selenium RC drove the browser by injecting JavaScript, and its `css=` locator strategy used Sizzle to evaluate CSS selectors, since native CSS querying was inconsistent across the browsers of the time. WebDriver later moved to the browser's own native selection, and the jQuery-era conveniences like `:contains()` and `:eq()` fell away with it, since they sit outside the CSS specification.

Injecting Sizzle brings those conveniences back without trading away what native selection does well. Sizzle hands standard selectors to the browser's native engine wherever it can, and turns to its own matching only for the extensions that native CSS cannot express. The extensions are a superset on top of native CSS, not a replacement for it: an everyday selector runs through the native engine just as it would without Sizzle, and the pseudo-classes are there for the moments a test genuinely needs them. Far from an exotic dependency, this restores a capability Selenium itself once shipped while keeping the performance and standards compliance of native CSS.

This all happens under `BySizzle`, the class that extends Selenium's `By` to run selectors through the injected engine. Nothing in a test or page object calls Sizzle directly. Writing the selector in a `Locator` is enough.

### Pseudo-classes worth knowing

These Sizzle extensions come up most often when mapping elements, and are the ones that make [parameterization]({{< relref "locators.md#parameterization" >}}) practical:

- **`:contains(text)`** matches elements whose text content includes `text`. This is the workhorse for selecting a row, card, or result by the content a user would actually read, rather than by a brittle positional path.
- **`:eq(n)`** matches the element at index `n` in the result set, **zero-based**. `:eq(0)` is the first match, `:eq(2)` the third.
- **`:first` / `:last`** match the first and last elements in the result set.
- **`:even` / `:odd`** match elements at even and odd indices, also zero-based, useful for striped rows.
- **`:has(selector)`** matches elements that contain a descendant matching `selector`.
- **`:not(selector)`** matches elements that do not match `selector`.

For example, selecting a search result by its visible text, then the third such result:

```java
".search-result:contains(Rocket Skates)"
".search-result:eq(2)"
```

Because `:contains()` takes an arbitrary string, a selector fragment can be built from a method argument, which is exactly what parameterized locators do. See [Parameterization]({{< relref "locators.md#parameterization" >}}) on the Locators page for how a single `Locator` method reaches a whole family of elements this way.

The full set of supported selectors is documented in the [Sizzle documentation](https://github.com/jquery/sizzle/wiki). Anything Sizzle accepts is a valid Qadenz selector.
