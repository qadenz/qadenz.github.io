---
title: "Components"
linkTitle: "Components"
description: >
  The parts of Qadenz a test project works with directly, in the order a test uses them: configure the run, model the UI, act on it, assert against it, and read the results.
weight: 3
---
Every Qadenz project is built from the same set of parts, and this section is the reference for the ones we work with directly. It sets the framework's internals aside and covers what a consuming project actually writes against.

The parts follow the shape of a test. **Configuration** stands the project up, **UI Modeling** names the page, **Commands** act on it, **Conditions & Expectations** describe its state for both waiting and asserting, and **Test Results** record what the run leaves behind. Read in order, the five trace one test from setup to result. Reached for alone, each stands as the reference to its own part.