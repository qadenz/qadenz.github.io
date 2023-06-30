---
title: WebDriverProvider
description: >
  Configuration
weight: 3
---

The [`WebDriverProvider`](https://github.com/qadenz/qadenz/blob/master/src/main/java/dev/qadenz/automation/config/WebDriverProvider.java) stores the `WebDriver` instance and makes it available to components where it is needed. Since the test classes work on an inheritance hierarchy, and with how TestNG manages threads, the `WebDriver` instance is required to be kept on a `ThreadLocal<WebDriver>` object. Rather than forcing testers to pass the `WebDriver` around to various library components, Qadenz has chosen to make the `ThreadLocal` static, and allow components that need the `WebDriver` to call a centralized location. On the (hopefully) rare occurrence where the `WebDriver` is needed directly, it can be invoked with the following call.

```
WebDriver webDriver = WebDriverProvider.getWebDriver();
```
