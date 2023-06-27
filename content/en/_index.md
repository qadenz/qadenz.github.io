---
title: "Welcome to Test Automation with Qadenz!"
linkTitle: "Documentation"
type: "docs"
weight: 20

cascade:
- _target:
    path: "/blog/**"
  type: "blog"
  # set to false to include a blog section in the section nav along with docs
  toc_root: true
- _target:
    path: "/**"
    kind: "page"
  type: "docs"
- _target:
    path: "/**"
    kind: "section"
  type: "docs"
- _target:
    path: "/**"
    kind: "section"
  type: "docs"
---

> **This site is undergoing some upgrades and updates. Please pardon any broken links, unhandled variables, and other small issues while things are getting sorted out. Things will be back to normal just as quick as possible.**

Qadenz is a robust test automation library written in Java that wraps [Selenium](https://www.selenium.dev/) for browser automation, [TestNG](https://testng.org) for suite management, and [Hamcrest](http://hamcrest.org/JavaHamcrest/) for validations. Using a mildly opinionated approach, Qadenz solves the common issues encountered while implementing the wrapper layer between the tests and accompanying UI models, and the underlying frameworks (like Selenium and TestNG).

The goal of Qadenz is to take the heavy lifting out of developing a test automation project, and allow teams to focus instead on rapid scripting of tests. This is accomplished by using an easy-to-learn API that encourages simple to read tests without using Gherkin/BDD-based tools that add unnecessary complexity and maintenance overhead to the development workload.

Core Features of Qadenz:

- Boilerplate test setup and configuration is built and ready to use
- Selenium element interaction functionality is wrapped in a simple to use API
- A unique conditions-based approach to validations
- Custom built-in HTML reporting that provides detailed results data along with integrated screenshots

Qadenz is also extensible and stands ready to support custom functionality based on the needs of the tests to be automated. The design patterns that Qadenz provides can quickly and easily be followed for seamless integration of team-specific features.

# License

The Qadenz Library is made available under the [PolyForm Internal Use License](https://polyformproject.org/licenses/internal-use/1.0.0/) as a "Source Available" library. Teams are welcome to use Qadenz to power their internally managed test automation projects and modify as needed, but are prohibited from re-distributing the library or marketing/selling Qadenz (and derivative works of Qadenz) as a product for their customers.

# Getting Started

Getting started with Qadenz is as simple as importing the dependency from Maven. On the `pom.xml`, add a new entry to the `<dependencies>` section:

```xml
<dependency>
    <groupId>dev.qadenz</groupId>
    <artifactId>qadenz</artifactId>
    <version>1.0.0</version>
</dependency>
```

The code is also available at [GitHub](https://github.com/qadenz/qadenz).

This documentation on this site covers the general approach to automated test design from Qadenz, the major components and how they can be implemented, how test suites can be executed and results can be reviewed and shared. Improvements to the documentation are always in progress, but [questions and suggestions]({{ site.repo2 }}/issues/new?labels={% if page.editable %}{{ page.editable }}{% else %}question{% endif %}&title=Question:&body=Question on: {{ site.repo }}/tree/master/{{ page.path }}) are always welcome and will be a big help to understanding where to prioritize updates.

Happy automating :)
