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
commit in the main branch per PR, ideally addressing one ticket. This means it's the 
place people in future will scan the code changes made when theyre trying to find a
bug, answer a question, or extend the functionality of the project.

Importantly, the PR is the place the rest of the team review the code going into the 
codebase, meaning it's the first and best opportunity to talk about stylistic or 
functional choices before they become difficult to untangle.


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

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
