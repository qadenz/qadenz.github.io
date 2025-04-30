---
title: WebConfig
description: >
  Configuration
weight: 2
---

[`WebConfig`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebConfig.java) stores information that Qadenz uses at various points throughout the execution cycle. Primarily, these are the values given as parameters on the TestNG Suite XML file. While Qadenz calls these values during the setup and reporting phases they are made available on the chance these values are needed for any project-specific configuration.

Since these fields are all static, there really is no need to extend this class. Teams are encouraged to follow this pattern, though, and create a ProjectConfig class of their own should the need arise to track specific values or project-specific Suite parameters are in play.
