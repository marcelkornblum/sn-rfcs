---
title: Pull Requests
description: The how and why of Pull Requests
---
- Feature Name: `pull_requests`
- Start Date: 2021-03-11
- RFC PR: [signal-noise/rfcs#0019](https://github.com/signal-noise/rfcs/pull/0019)

# Summary
[summary]: #summary

Pull Requests are important at Signal Noise. This RFC sets out recommendations for how
to create a PR and how to review one, as well as the surrounding processes.

# Motivation
[motivation]: #motivation

Well formed, well reviewed Pull Requests are arguablty the most important aspect of 
coding at Signal Noise.

Because [we use the `Squash and Merge` strategy](./0001-git-workflow#reference-level-explanation),
the PR is the unit of code that is preserved in the record of the project, with one 
commit in the main branch per PR, ideally addressing one ticket. 

This means it's the place people in future will scan the code changes made when they're 
trying to find a bug, answer a question, or extend the functionality of the project.

Importantly, the PR is the place the rest of the team review the code going into the 
codebase, meaning it's the first and best opportunity to talk about stylistic or 
functional choices before they become difficult to untangle.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation


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




Explain the proposal as if it were already included in the Signal Noise way of doing things and you were teaching it to another programmer. That generally means:

- Introducing new named concepts.
- Explaining the technique or tool largely in terms of examples.
- Explaining how programmers should *think* about the workflow, tool or process, and how it should impact the way they work. It should explain the impact as concretely as possible.

This section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the proposal in sufficient detail that its interaction with other tools or approaches is clear.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this approach the best in the space of possible approaches?
- What other approaches have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other organisations, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other companies or teams is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that Signal Noise sometimes intentionally diverges from common approaches.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the approach do you expect to resolve through the RFC process before this gets merged?
- What parts of the approach do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the company as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the company and team in your proposal.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
