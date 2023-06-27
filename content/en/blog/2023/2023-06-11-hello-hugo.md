---
date: 2023-06-11
title: "Hello, Hugo"
linkTitle: "Hello, Hugo"
description: >
    Saying goodbye to Jekyll, Qadenz is now using the Hugo static site generator. Here's how it's going...
author: Tim Slifer ([tim-slifer](https://github.com/tim-slifer))
---

I'm not an expert on static site generators. What sparked my initial interest in the beginning was really just the idea that I could stand up a decent looking site in little time and host for free on Github instead of paying for a bunch of hosting features I neither need nor want. Fortunately, the community around tools like Jekyll and the availability of open source themes made the job of getting a personal profile page up and running quite simple, and the learning process was very satisfying. When the time came to release Qadenz, I knew I needed some place to host some documentation, else I'd be sealing it's fate that it would never see any use beyond myself and my teams.

Github wikis seemed a logical solution for this at first, but I wanted something different (having worked with GH wikis at a number jobs), something that I could make a better presentation of the content and maybe engage just a bit with any users that found their way to my work. I had already used Jekyll to build a couple iterations of a personal site, so after settling on a name for the library (which had up to that point simply been called "Automation") and registering some domains and finding a really nice theme called Docsy, I got to work.

## Transitioning from Jekyll to Hugo

My interest in Hugo came along after discovering that the Selenium project was also using Docsy to organize their documentation. I had only heard of Hugo at the time, but if the pros on the Selenium project were using it, it was probably worth a deeper look.

Installing Go and setting up Hugo was way faster than getting a Ruby environment stood up and installing Jekyll. Configuring the site and running locally were very easy, and it wasn't much of a chore to track down the spots where I wanted to make some small UI tweaks. I also opted for a "[mostly-docs](https://github.com/gwatts/mostlydocs)" configuration as I really didn't feel the need to have much more than the documentation and a blog on the site.

Getting the site published to Github Pages was the trickiest part, but admittedly a lot of that came down to me learning my way through Github Actions at the same time. One advantage the Jekyll version of the site had was deploying the site to the live environment was baked right into the repo features. I ultimately settled on a first workflow that would set up the build environment, run NPM and Hugo, then publish the site to the `gh-pages` branch of the repo. I configured the Pages options to auto-depoy on any push to the `gh-pages` branch, so the baked-in workflow to deploy the site works it's magic, and out comes a nice looking site full of documentation.

One item of note that I'm particularly happy about is that the Hugo version of the site was much easier to get Google Analytics up and running. I'm not sure why, but I was never able to get it working with the Jekyll version. That's hardly a knock on the [docsy-jekyll](https://github.com/vsoch/docsy-jekyll) project though. I very likely missed a setting along the way that would have made it work. I never saw anyone else complaining that it didn't work, so it couldn't have been a problem in Docsy itself. [vsoch](https://github.com/vsoch) and her contributors have built an impressive implementation of Docsy for the Jekyll community.

## Onward...

So at this point, the site as it sits today is more or less a direct port of the content from the Jekyll site. I've done a number of UI tweaks to match the look and feel of the old site, but more work yet remains. As well, there are a number of instances in the site content where placeholders and such are no longer parsed in the markdown. I'm cleaning these up as I see them, but my focus for the moment is mostly the UI details of the site. I'm planning a bigger overhaul of the documentation that will take care of remaining content issues.

I'll talk more about it in another post, but I've been working on Release-2.0.0 of Qadenz. It's only fitting that a new version of the code comes with a new version of the docs.
