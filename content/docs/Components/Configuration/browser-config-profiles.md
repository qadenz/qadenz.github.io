---
title: Browser Config Profiles
description: >
  Configuration
weight: 4
---

In some cases, extra configuration arguments need to be passed to the browser Options instance for `WebDriver` configuration. A common case for this is instructing a browser to run in headless mode. Qadenz provides a simple means to set these configurations, and allows for multiple "profiles" of these configurations to be set on the TestNG Suite XML file. For example, if a tester wishes to run in headless mode on the Selenium Grid, but run in a native browser when executing locally, the switch is as simple as altering a parameter value.

Configurations are stored in JSON files within the project's `resources` directory. To create a new configuration, within the `resources` directory, add a new folder called `config`, and create a new JSON file. This file must be named `{browser}-config.json`, where `{browser}` is the name of the browser being configured. A separate file must be created for each browser
requiring configuration.

This example configures Chrome to run in headless mode.

```
[
    {
        "profile": "headless",
        "args": [
            "--headless",
            "--window-size=1920,1080",
            "--disable-gpu"
        ]
    }
]
```

The JSON format is an array of configuration objects. Each object holds `profile` and `args` fields. The `profile` field will be matched to the `browserConfigProfile` parameter on the Suite XML. The `args` is an array of individual arguments to be passed to the `WebDriver` configuration.
