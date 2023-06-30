---
title: Commands
description: >
  Commands
weight: 1
---

Commands are a collection of classes that either act upon elements, interrogate elements, or control the browser under test. The methods on these classes represent steps in a test case. Commands can be used directly inside tests, or can be wrapped on Page Objects to create a contextually friendly reference to actions performed on the UI under test.

Methods on the Commands classes drive the supporting functionality that makes Qadenz useful. In addition to performing the surface actions, Commands methods handle WebElement initialization, catching and logging exceptions, and capturing screenshots.

Commands classes cover default Selenium interactions and cover some additional common actions, but are designed to be extensible so that specific needs of an automation project can be met.
