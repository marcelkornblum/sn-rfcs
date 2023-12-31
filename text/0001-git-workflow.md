---
title: Git Workflow
description: Structuring commits and branches
---

- Feature Name: `git_workflow`
- Start Date: 2019-10-16
- RFC PR: [signal-noise/rfcs#0002](https://github.com/signal-noise/rfcs/pull/0002)

# Summary
[summary]: #summary

A consistent way to approach version control, allowing the team to easily move between projects and reducing the time for implementation of CI/CD. This approach allows easy, thoughtless committing often, while preserving a neat and navigable history of changes.

Essentially, we follow Github Flow but with extra details and principles.

# Motivation
[motivation]: #motivation

Without a consistent workflow for making changes to codebases, each codebase follows the whim of the developer involved. Teams need to discuss this at the beginning of each project and onboarding takes more time. Having a single company-wide approach minimises the time spent on this in favour of an approach that balances the needs of developers with those of the organisation.

The specific workflow outlined here allows developers to commit and push often, using the remote git repository as a backup, without having to consider unitary functionality and isolation at that stage. This in turn is helpful for quick onboarding of freelancers and juniors, many of whom struggle with onerous "cleanliness" requirements at this stage.

The requirement for a project to retain a useful, clear history in order to facilitate post-hoc rationalisation about the code and the work is met at the Pull Request level; the main branch has a clean history and the PRs explain the details of each code change.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

We assume you already have a local copy of the repo and can use basic git commands. There are a few parts to this:

* [Day to day process]
  * [Update main]
  * [Create a feature branch]
  * [Write some code]
  * [Open a Pull Request]
  * [Review the PR]
  * [Merge the PR to main]
* [Preparing for a release]
  * [Create a release branch]
  * [Make hotfixes]
  * [Create a release for production]

... or jump to the [Reference-level explanation]

## Day to day process
[Day to day process]: #day-to-day-process

This covers the essential process of writing some code to add a feature or fix a bug in the normal course of things.

Start off by deciding which feature or bug you'll be working on. Usually this will have an issue with a number associated.

### Update main
[Update main]: #update-main

*NB. We use `master` and `main` interchangeably here since we have repos with different main branch names*

Update your local `main` branch so you're working on the latest code, to minimise your merge issues.

`git checkout main && git pull` or `git checkout main && git fetch && git rebase`

### Create a feature branch
[Create a feature branch]: #create-a-feature-branch

Create a new branch based on `main`, just for this specific piece of functionality (it's important that you don't make changes to other things on this branch, and that you keep the changeset a reasonable size so that other people can review your changes effectively). It doesn't matter what you call this branch - let's call it `new-branch` in this example. 

`git checkout -b new-branch` 

Note that many people like to name their branches with decriptive names and / or names that include issue numbers. We applaud and encourage that, but it's not required and you won't be judged if you don't do it. 

### Write some code
[Write some code]: #write-some-code

Make some changes to the code. Commit often. Don't worry overly much about the commit messages or format, or what's in each commit. Nobody is expected to look at your code on the level of individual commits, and we won't be cherry-picking them.

`git commit -am 'just fiddling with some stuff'`

When you have some changes that you either think may be ready for review, or you'd like to preview them on the CI/CD environment, or simply that you'd like them somewhere central and off your computer, push your changes. The first push requires creation of a remote branch to track against your local, which you can do in one step with the below command:

`git push -u origin HEAD`

### Open a Pull Request
[Open a Pull Request]: #open-a-pull-request

Feel free to make more changes, and commit and push some more, but at some point you'll want to open a new PR. You can do that in the Github interface (there's a custom link in the response to your push, if you're looking for shortcuts).

The Pull Request is the place to stop and consider the code that's going into the project in detail. This is what will go onto the main branch, and that needs to provide others with the information they need to understand what happened and why. 

Sometimes it can take a while to get everything together for a complete PR and many people like to work with an open PR that they can slowly build up as they go. This is fine, just please remember to label the PR as `WIP` or `DO NOT MERGE`.

PRs will generally also have a custom CI/CD environment so that anyone on the project can review the proposed changes in their browser. If it's your PR it's your responsiblity to check that everything looks the way it should on that environment. PRs should also trigger **status checks** as part of the CI/CD process - at a minimum a linting check and also a full test run if tests are written for the project.

The PR will usually have a template explaining the steps to take; please follow these, including the below. 

* Provide a concise, clear title
* Link to any relevant issues that may have more information about the work (with a [keyword to close the issue](https://help.github.com/en/github/managing-your-work-on-github/closing-issues-using-keywords), if relevant)
* Write a description of the changes you've made
* Include any instructions that other developers will need to take to stay up-to-date; i.e. installing new requirements or running DB migrations
* Make sure that any relevant documentation is also updated and included in the same PR - check the README and any other docs you may have
* Make sure to assign it to relevant (or all) developers on your project, including the lead

### Review the PR
[Review the PR]: #review-the-pr

If you're assigned to a PR please find time to take a look when you get to a natural break in your work. No PR should go through without being approved by the tech lead or lead developer on the project, or at least one other developer if that isn't possible.

Developers should inspect code for **style, clarity and overall approach**. 

It's important to ensure that the code is understandable, which may require some extra commenting for example. 

It's also often the case that coders may miss work that others have done, resulting in duplication or different approaches to the same problem in the same project, which is always worth avoiding.

Wherever possible, code should adhere to the relevant RFCs regarding style and code approaches, and the PR is a good place to catch errors.

No PR should go through without its status checks passing. If there is testing on the project, the review process should include checking that there are enough tests for the new functionality.

When reviewing someone else's PR, please be thoughtful about your comments. In particular, do not close the PR without discussion with them as this can result in repeated work and is very discouraging. 

In a similar vein, when posting your review think carefully about the type of response you post. _Request Changes_ puts you in the middle of the process and is comparatively discouraging to other developers, as you are removing their ability to judge the seriousness of the impact of your comments. If you're not ready to approve please only use Request Changes if:
* You have specific changes to request, not general comments, and
* The PR should not be merged without them; i.e. they are serious enough to prevent 'fix later' or 'do better next time' to be an option, and
* You are available to re-review the PR at short notice. 

### Merge the PR to main
[Merge the PR to main]: #merge-the-pr-to-main

Please only use the `Squash and Merge` merge strategy on PRs. This ensures that the main branch's history remains clean and easy to browse, which in turn lets us commit early and often. Restricting this is configurable in the project's Github settings page.

Once a PR has been approved, it's best if **the approver merges the PR**, just to keep everything ticking over, though note that they will then share credit in the github history. The exceptions are either if the PR is out of date and can't be merged, or if a comment was made about something that won't block the merge but that the author ought to read before merging.

After merging, please **delete the branch** to keep the repo tidy. There's a setting in the projects Girhub settings page to automate this.

## Preparing for a release
[Preparing for a release]: #preparing-for-a-release

When your code is going to be deployed, or needs to be reviewed outside the organisation, you should start a release process.

Following this process will allow the standard CI/CD tools to automate much of the fiddly stuff and everything process-wise should _just work_.

### Create a release branch
[Create a release branch]: #create-a-release-branch

First of all create a release branch. This should be based on the branch that contains the code to be pushed out - usually this will be on `main`.

`git checkout main && git pull`

`git checkout -b release/sprint-3`

Pushing this branch should trigger a rebuild of the `test` environment, which is where end to end QA should be taking place. 

In general the `staging` environment should also be built from this branch as it's what is going to be on production next. This process is manual however, so that it's more tightly controlled since it's where the client reviews the work.

**NB.** This branch *must* be called `release/xxx` where xxx is some version code or name (e.g. `release/v0.1` or `release/sprint2`), otherwise the CI/CD systems won't recognise it.

### Make hotfixes
[Make hotfixes]: #make-hotfixes

Should issues arise during QA that need small fixes, these are most easily done by committing directly to the release branch. 

As with commits to a feature branch, don't worry about the formatting of commit messages etc at this point, but do note that pushing these commits will trigger a rebuild of the `test` environment, that may affect the QA process.

Larger fixes or issues are better addressed via the normal process of creating a feature or bug branch and sending it via the PR process to main, then updating the release branch from there.

### Create a release for production
[Create a release for production]: #create-a-release-for-production

Once QA has passed successfully, and the client has reviewed the up-to-date staging environment (both should build from the release branch), it's time to deploy to production.

The best way to deploy to production is from the GitHub interface. In the Code section is a Releases subsection. Releases are a layer on top of tags, allowing for tag creation to be associated with a text field that documents the features present in the deployment.

Create a new Release, and specify a new tag is created form the release branch you recently created (e.g. `releases/sprint-3`). Make sure that the release is documented and link to relevant materials.

**NB** It is critical that this tag starts with a `v` so that the CI/CD tool recognises it as a release to deploy to production.

Our preference is for the project managers to trigger the production deployments of our code, as inevitably they have to deal with the consequences first.

Once the release has been tagged and triggered, the release branch should be merged back into main via a PR in the usual way, to capture and retain any hotfix changes made during QA.

# Reference-level explanation
[Reference-level explanation]: #reference-level-explanation

## Branching model

We use the [GitHub Flow](https://guides.github.com/introduction/flow/) model of when to create branches - please take the time to read that guide; it's based on the well-known Git Flow model but is simpler. 

Note that we add extra requirements as laid out below.

**NB** In this model the `main` branch represents the furthest development of the code - it is not QAd and safe to deploy. All deployments will have tags in the repo.

## Squash and Merge

We use `squash merging` to merge code from a Pull Request to its target (usually `main`, though some repos still use `master`). This means our main branch always has a neat history with each commit being a unit of work. The GitHub interface links each commit to its related Pull Request, so that it's easy to review everything that went into the PR and any discussion around it.

It also means **you can commit and push code safely at any time**. We encourage you to think of the repo as a backup; commit frequently without worrying about everything working, and push often. When you get to the Pull Request stage is when you have to be tidy.

## Rebasing

We never rebase as standard and with our model you should never need to. If you do want to rebase for some reason it's not acceptable to rebase anything another developer has pulled locally; this means **it's only safe to rebase commits you haven't yet pushed**. Rebasing as a pull strategy for example, is fine and many of us use it by default.

Force pushing should never be necessary for any reason, and should be disabled on `main`.

## Specific branch naming

The below approach allows us to tightly integrate CI systems with our workflow to automate some of the more stressful moments in the workflow as well as give us a decent history to review.

Importantly, it also gives us the capability to separate QA and release preparation from ongoing feature development in a standard way; this is very important on some projects.

1. The main branch of the repo is always called `main`. Nothing should be committed directly to main; it should always be updated via a PR.
2. Feature branches, generally each relating to a specific issue / ticket, can be named anything. These are usually based on the main branch and the PR generally targets the main branch.
3. Once a milestone is reached and code is ready for QA and review before deployment, a new branch should be made from main - this **must** be called `release/xxx` where xxx is some version code or name (e.g. `release/v0.1` or `release/sprint2`).
4. Bug fixes (hotfixes) should be based on this branch and can be committed directly back to this branch; no PR necessary. No features should ever be committed to a release branch; they should come via main.
5. Deployments to production should **always** be tagged in the format `vxxx` where xxx is the semantic version number or, possibly, a made-up version name (e.g. `v1.0.3`). The tag should generally be made from the `HEAD` of the release branch, which should then be merged back into main via a PR.

## Deployments to production

Deployments to production should **always** come from a tag starting with `v`. The rest of the tag name should be a semantic version number. Ideally, the tag creation itself will trigger the production deployment (this is the default if you have [deploybot](https://github.com/signal-noise/deploybot) set up).

The deployment tag should reference the last fully tested code, which ought to be a release branch. The branch should be merged back in to main (to capture any hotfixes) immediately after release.

## Large or on-going features

If a feature is very large, complex or long running it's tempting to start a main branch for it, and then create new branches from that, merging back into it.

If possible, **this should be avoided** as it adds a lot of complexity and risks merge errors and losing history.

If required, some important points to take into account when doing this are:
* Keep the main feature branch updated from main on a regular (daily) schedule
* Each sub-branch should be merged into the main one via the standard PR process, while also mentioning that they are part of the larger feature
* When it's time to merge the main branch back to main, you should use `merge commits` - NOT `squash and merge`. This means all the PRs from the sub-branches will appear as individual PRs on the main branch history.

# Drawbacks
[drawbacks]: #drawbacks

Having any systems and processes in place can impact the speed of onboarding of new team members; we've aimed to make this as low impact as possible. Another drawback of this approach is that the cleanliness of the history is reliant on Github or similar, where Pull Requests are a first class feature of the system; using a self-hosted Git server this approach would not work, and therefore portability has been reduced.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

With no common system we find ourselves doing a lot of repetitive configuration work, and it becomes much harder to reason about projects that you weren't personally part of since different workfows result in different levels of historical information, and captured in different places.

Git Flow is perhaps the most commonly understood and talked-about methodology for using Git in multi-person teams, but it has a relatively steep learning curve and a fairly high overhead. Unless the project is a product with a very formal process, the main branch becomes almost unused with almost all attention on the develop branch, which leads to extra merging and unnecessary maintenance.

Github Flow is fairly commonly used amongst organisations that adopt a formal scheme. The day to day workflow is easily understandable both by newbies and those used to Git Flow, and the release process is similar to Git Flow's.

The day to day overhead is low in comparison and there's enough flexibility to allow for quick deployments or more formal ones.

Perhaps most importantly, compared to most formal approaches this workflow allows developers to think about the formal, clean and tidy approach in a separate time and headspace to the actual writing of code, by allowing commits to be loose and unstructured. This allows the organisation to more easily work with developers of varying styles and skillsets, without adding a burden of process to the lowest level of the day-to-day method.

# Prior art
[prior-art]: #prior-art

While Github Flow is commonly used I'm not sure of examples of this exact setup in use anywhere, but really the exact setup is no more than a little bit of (thought out) standard configuration on top of the standard process.

One observation I've had is that the release process is different enough from the day-to-day that people tend to forget it and not follow it, which can sometimes lead to issues, especially if people take a very unstructured approach and e.g. build to staging from the main branch.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

I think since we all already use this process it should be pretty straightforward to process this RFC. I've left the CI/CD process and details of the environments etc to a separate RFC which I intend to submit shortly.

# Future possibilities
[future-possibilities]: #future-possibilities

It would be good to come up with any improvements to the release process that made it easier for people to understand and follow naturally, while maintaining the discipline and flexibility of the current approach.
