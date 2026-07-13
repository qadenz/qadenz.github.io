---
title: "Locator Groups"
weight: 3
description: >
  Collect related Locators into one named unit and validate the whole group in a single call.
---

A `LocatorGroup` gathers several `Locator`s under one name and treats them as a unit. Where a `Locator` maps a single element, a `LocatorGroup` maps a component: the fields of a form, the controls of a toolbar, the default elements of a page. It exists so a whole component can be validated in one call rather than one `Condition` per element.

## Building a group

`LocatorGroup` takes a name and the `Locator`s that belong to it, either as varargs or as a `List`:

```java
Locator usernameField = new Locator("Username Field", "#username");
Locator passwordField = new Locator("Password Field", "#password");
Locator rememberMeCheckbox = new Locator("Remember Me Checkbox", "#remember-me");
Locator signInButton = new Locator("Sign In Button", "#sign-in");

LocatorGroup signInForm = new LocatorGroup("Sign In Form",
        usernameField, passwordField, rememberMeCheckbox, signInButton);
```

The name is what identifies the group in logs and reports, so it should read as the component it represents ("Sign In Form"), the same way a `Locator` name reads as its element.

An individual `Locator` can belong to a group and still be used on its own. The four fields above are each mapped for direct input, and also collected into `signInForm` for validation. Grouping does not consume them.

## Validating a group

Three Conditions accept a `LocatorGroup` and apply a single [Expectation]({{< relref "/docs/Components/conditions-expectations/_index.md" >}}) to every element in it:

- **`Conditions.visibilityOfElements(group, expectation)`**
- **`Conditions.presenceOfElements(group, expectation)`**
- **`Conditions.enabledStateOfElements(group, expectation)`**

Passing one of these to `verify()` or `check()` evaluates the entire group in a single validation. To confirm every element of the sign-in form is visible:

```java
verify(Conditions.visibilityOfElements(signInForm, Expectations.isTrue()));
```

The Condition walks each `Locator` in the group and tests it against the same expectation. If any element fails, the result reports which ones: the failure names each discrepant element and the state it was found in, so a group check does not just report that "the form" failed, it points at the specific field that was missing. This replaces a stack of individual visibility checks with one line that still pinpoints the culprit.
