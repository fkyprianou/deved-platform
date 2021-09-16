---
title: Using GitHub Repository Guidelines to Enhance Developer Experience
description: We care about Developer Experience, so we produced and shared the
  repository guidelines that we use to welcome developers to our GitHub repos.
thumbnail: /content/blog/using-github-repository-guidelines-to-enhance-developer-experience-dr/using-github-repository-guidelines-to-enhance-developer-experience.png
author: lornajane
published: true
published_at: 2020-01-17T13:08:45.000Z
updated_at: 2021-04-27T19:52:02.530Z
category: devlife
tags:
  - github
comments: true
redirect: ""
canonical: ""
---
At Nexmo, we love sharing code with our developer communities. Usually, this means publishing to a git repo on GitHub so that any developer that wants to use the code can go ahead and do so.

We realized though that now we have around 300 repositories between a few different GitHub organizations, it can be tricky to find what you need and also to understand how to use each project unless there are good instructions.

To make the Developer Experience even better for everyone, we created (and then publicly shared) [our Repository Standards](https://github.com/Nexmo/repo-standards).

## Appropriate Standards for Different Repo Types

We recognized three types of project that we publish often and adjusted our guidelines for each type.

### SDKs

Our SDKs are easily the most-used repositories and we're proud of them! Users of varying levels of experience need to be able to come to these projects and understand how they can use them in their own projects to perform particular Nexmo tasks. We took particular care with the installation instructions and making clear that the license is very clear so that our community can build on our SDKs with confidence.

### Demo Applications

We publish a lot of demo applications. These are standalone applications to demonstrate a particular Nexmo feature, and we make them available for others to copy and use. Having a clear statement of the features and purpose of the project is really important in a repository like this. We also worked on licensing and on including either docker setups or "click to deploy" buttons to allow users to try out what we had made for them.

The danger with standards of any kind is that the rules get out of hand! Plenty of our repositories are just a public version of a one-off standalone application that was created to illustrate a tutorial, blog post or video to show developers something in particular.

If we only published the ones that were perfectly described with deployability features and detailed usage instructions, there would be a lot fewer repositories on our GitHub account!

In fact, the only rule here is that the repository must have a `README` that includes a link to the thing it is in support of.

## Don't Make Me Think

We created Repository Standards in an attempt to capture a shared checklist of things we think are important when we publish a repository. Everyone had their own ideas, but as a team made up mostly of engineers, that didn't always translate into excellence when it came to things-that-are-not-code.

By creating checklists and templates of the most common things, we made "Do The Right Thing" into _"Do The Easy Thing"_, and improved the experience of developers finding our projects on GitHub.

We started with a [basic README template](https://github.com/Nexmo/repo-standards/blob/master/basic-readme-template.md) to give some simple structure and remind us to link to the developer documentation and tell users how to get in touch if they needed to. It's basic, but it's a LOT better than nothing.

Every repository needs a license, and we default to MIT. Having the guidelines, including a checklist, reminds us that's what we want to do.

We also have a basic `CONTRIBUTING` file—it's incomplete because it's hard to generalize, but it gives us a starting point and leaves just a little less work for repository creators to do when they have code to share.

## Developer Experience is About Detail

The details here are small things, and there are many more things in the complete guidelines - but details do matter. Developers may land on your GitHub Repositories, possibly unaware of what they have found or where to go next.
 Helping to orientate them on where they are, what this repository is about, and where they might go next are all great ingredients for a better Developer Experience.