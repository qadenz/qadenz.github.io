---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
---

Bootstrapping an automation project with Qadenz is as simple as starting a new Java/Maven project, adding a dependency, and building a simple test. This guide will run through the pre-requisites, and show how to build a simple first automated test that verifies the functionality of an authentication form.

## The Foundation

Before we dive into a fresh project, let us first make sure we have the necessary tooling in place to do so.

Being a Java project, we'll need a JDK. Qadenz is currently built with **JDK 11**, so an equivalent or newer version must be installed. Next is an IDE. This is completely a preference-based decision, but with the prevalence of **IntelliJ IDEA**, it is strongly recommended. The Community Edition is free to use and will be sufficient for automation development with Qadenz. Most IDEs come with **Maven** bundled, and IntelliJ is no different. This is perfectly functional, but can be known to exhibit some different behavior in certain situations than a native installation. It is recommended to install the latest Maven and configure the IDE to use the native instance instead of the bundled version. Version control is also an important part of collaborating with other team members. While there are a number of tools available to suit this need, **Git** is very commonly adopted by teams. Finally, ensuring IDE support for things like **TestNG**, **Maven**, and **Git** can make working with these tools very convenient.

Next, since this is a test automation project, we'll need to make sure some related tooling is in place. Qadenz is built to use the `RemoteWebDriver`, so the **[Selenium Server](https://www.selenium.dev/downloads/)** is a necessity. **Chromedriver** and the **Chrome** browser itself will also need to be in place. Other browsers are certainly supported, Chrome is simply a good starting point due to it's dominant market share.

Package managers are a great way to install and update development tools. In a Windows environment, **Chocolatey** is a good option. For Mac, **Homebrew** is a great choice. Linux distros will have their own package systems baked in, and these tools are typically available through official repos.

## The Framing

((Standing up a Java project, adding Qadenz, first classes))

In the IDE, start a fresh Maven project. Open the `pom.xml` file and add the dependency information for Qadenz.

```
<dependency>
    <groupId>dev.qadenz</groupId>
    <artifactId>qadenz</artifactId>
    <version>2.0.0</version>
</dependency>
```

## The Fixtures

(( First UI models, Write and run a first test ))

## The Finishing

(( Results viewing ))
