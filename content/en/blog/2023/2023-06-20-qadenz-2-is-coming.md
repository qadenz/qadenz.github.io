---
date: 2023-06-20
title: "Qadenz 2.0.0 is coming"
linkTitle: "Qadenz 2.0.0 is coming"
description: >
    This is the first new functionality for Qadenz in a while, and with 2.0.0 comes setting the stage for more...
author: Tim Slifer ([tim-slifer](https://github.com/tim-slifer))
---

When I first set out to build the latest iteration of tooling that became Qadenz, I really didn't expect to hit a 2.0.0 milestone for a good long while, if ever. Here we are now, a couple years after the initial 1.0 release and the first functionality release since that time, and already we're bumping to 2.0.0. I guess I'll have my words on a platter with a side of potatoes.

## What's changing?

While adding the new Temporal and Numeric Conditions and Expectations, I made the decision to do some redesigning of how these components were built. I'll go into more detail in another post on the topic, but the short story is the evaluative logic in the library got repackaged, and now Conditions and Expectations both live on their own. Everything is still there... it's just presented a little differently. This represents a breaking change in consuming projects, and serves as the technical reason for a major version increment (following the Semantic Versioning model). Consuming projects will need only to update their imports.

As I mentioned, the Temporals and Numerics are the big new feature. These bring the ability for a test to parse the text of an element as either a Temporal (LocalDate, LocalDateTime, LocalTime) or a Number (Integer, Double), respectively, and perform validations using these APIs leveraging some of the capabilities and features of these classes.

Other changes in the release include an ability to retrieve text from an element directly while ignoring text shown in any child elements, removal of the auto-maximize browser logic in the WebDriver setup, and some added convenience for passing collections of Conditions to either the `verify()` or `check()` methods.

Full release notes will be published when it's all done, but the [Milestone](https://github.com/qadenz/qadenz/milestone/2) can be reviewed for a listing of all the Issues and Pull Requests that are in scope.

## A major version, though?

I said above that there was a technical reason for the major version increment. The other side of this has to do with maturity of the project and crossing that threshold from making something work, to getting it done right. A number of things in Qadenz were built in a way that was either what made sense at the time, or were the result of my own skill as a developer when each component was built. A conversation with [Matt](https://github.com/maurerit) really hit home when the topic turned to Qadenz-1.0.0 being an MVP of the potential in this project. Following that chat, I decided to make what I had originally planned as the 1.1.0 release the 2.0.0. There are still plenty of areas that I'd like to go back and make better. This release, as it happened, focused on Conditions and Expectations.

I'm viewing the 2.0.0 series of releases as the opprtunity to take this project from "just some automation code I've been tinkering with for years" to a more serious project that from which, as I would hope, some teams out there could really see some value.

So, will there be a 3.0.0? I've learned not to say no to something like that, but I think there's a good ways to go before that would happen.

## So where's the code already?

The Release PR is open at the time of this writing. I've frozen the feature scope, and any small changes made prior to release will be to address any major issues or other items that need to be done before the final merge. There isn't a preview release in Maven Central for this one like there was for 1.0.0, but the `release-2.0.0` branch can be pulled down and built locally for use as a dependency in a consuming test project, should anyone wish to tinker.

My focus at the moment is overhauling the documentation on this site to not only include the new features, but also to be a little more useful. In my first iteration, I think I was able to cover much of the "what" and "where", but perhaps maybe not enough "how" or "why".

As for the "when", the answer is "soon". I hope to be ready in the next few weeks.
