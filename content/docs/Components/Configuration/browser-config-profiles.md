---
title: "Browser Config Profiles"
linkTitle: "Browser Config Profiles"
description: >
  Named sets of browser launch arguments, kept in JSON and selected per run by a Suite parameter, so switching between headless and headed is a value change rather than a code change.
weight: 5
---

Some runs need extra arguments passed to the browser at launch. Running headless is the common case, but window sizing and other browser flags fall in the same category. Qadenz keeps these arguments out of the test code and in external JSON files, grouped into named profiles that a run selects with the [`browserConfigProfile`]({{< relref "suite-parameters.md" >}}) parameter. Switching between headless on the Grid and a headed browser on a local machine becomes a matter of changing one parameter value, with nothing recompiled.

## Where the file goes

Configurations live in JSON files under the project's `resources` directory. Add a folder named `config`, and inside it a file named `{browser}-config.json`, where `{browser}` is the lowercased browser name, for example `chrome-config.json`. Each browser that needs configuring gets its own file.

## The file format

The file is an array of profile objects. Each object holds a `profile` name and an `args` array of individual arguments.

```json
[
    {
        "profile": "headless",
        "args": [
            "--headless",
            "--window-size=1920,1080",
            "--disable-gpu"
        ]
    },
    {
        "profile": "maximized",
        "args": [
            "--start-maximized"
        ]
    }
]
```

The `profile` name is what the `browserConfigProfile` parameter matches against. The `args` are added to the browser's `Options` when the driver launches, so they are the same command-line arguments the browser itself accepts, and they are browser-specific.

## How a profile is selected

Before each test, Qadenz reads the `browserConfigProfile` parameter and looks through the matching browser's config file for a `profile` whose name equals it, ignoring case. The arguments from the matched profile are applied to the session.

If the parameter is absent, or if its value matches no profile in the file, Qadenz applies no arguments and the browser launches with its defaults. A misspelled profile name therefore fails quietly, as a browser that comes up without the flags rather than an error, so confirm the parameter value against the profile names when a configuration does not take effect.

This selection is what makes the same tests run differently per environment. Point `browserConfigProfile` at `headless` for a CI run on the Grid and leave it off, or point it at a headed profile, for local debugging. Because it is a Suite parameter, it also obeys the usual scoping: set it once on the `<suite>`, or override it on an individual `<test>`.