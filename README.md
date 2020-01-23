# Signal Noise RFCs

[👉 Public site](https://signal-noise.github.io/rfcs) where all approved RFCs are visible.

Many aspects of the way we do things at Signal Noise merit being 
standardised. By documenting workflows, techniques and tools that we as a 
technology team decide to adopt, we save time when facing similar
challenges from project to project, making more time to focus on the 
interesting and/or difficult aspects of our work.

The "RFC" (request for comments) process is intended to provide a consistent
and controlled path for new decisions in any of the areas that could loosely
be described as "the way we do things".


## Table of Contents
[Table of Contents]: #table-of-contents

  - [When you need to follow this process]
  - [Before creating an RFC]
  - [What the process is]
  - [The RFC life-cycle]
  - [Reviewing RFCs]
  - [Help this is all too informal!]
  - [License]


## When you need to follow this process
[When you need to follow this process]: #when-you-need-to-follow-this-process

You need to follow this process if you intend to make a change to the way 
Signal Noise goes about making things, in any reusable way. Specific examples 
of things that merit going through this process would be the addition of, or 
change to the existing process for any of the following:

  - Any aspect of the standard workflow we use, including approach to git 
    branches, etc.
  - Guidelines for how to structure and write our code, including advice on
    e.g. methods for reuse.
  - Approaches and tooling that we use in general across project, including 
    rules and enforcement tooling for linting, for example.
  - Guidance on tooling to use for common functionality or operations, such as
    -- for example -- analytics, monitoring, state management and even charting
    etc.

This is not intended to be used for project learnings or specific pieces of 
advice, unless they are in a generalisable form that will be of benefit as a way 
to go about a new and different project. Some changes do not require an RFC:

  - Aspects of our processes that affect people not in the technology dept -
    for example team meeting content, structure or output.
  - Learnings from specific projects that are not in a general form; e.g. when
    working with a specific individual it's best to communicate in person.

## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

A hastily-proposed RFC can hurt its chances of acceptance. Low quality
proposals, proposals for previously-rejected changes or those that don't fit into
the criteria for an RFC may be quickly rejected, which can be demotivating
for the unprepared contributor. Laying some groundwork ahead of the RFC can
make the process smoother.

Although there is no single way to prepare for submitting an RFC, it is
generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the RFC may be desirable; having a consistent
impact the way we do things requires concerted effort toward consensus-building.

The most common preparations for writing and submitting an RFC include talking
the idea over on the `#development` channel on Slack. You may file issues on this 
repo for discussion, but these are not actively looked at by the team.

As a rule of thumb, receiving encouraging feedback from senior developers is a 
good indication that the RFC is worth pursuing.


## What the process is
[What the process is]: #what-the-process-is

In short, to change the way Signal Noise does something, one must first get the 
RFC merged into the RFC repository as a markdown file. At that point the RFC is
"active".

  - Fork the RFC repo [RFC repository]
  - Copy `0000-template.md` to `text/1234-my-feature.md` (where "my-feature" is
    descriptive. Assign an RFC number as the next available sequential number 
    and use that in the filename).
  - Fill in the RFC. Put care into the details: RFCs that do not present
    convincing motivation, demonstrate lack of understanding of the proposal's
    impact, or are disingenuous about the drawbacks or alternatives tend to
    be poorly-received.
  - Submit a pull request. As a pull request the RFC will receive feedback from 
    the larger community, and the author should be prepared to revise it in 
    response.
  - Build consensus and integrate feedback. RFCs that have broad support are
    much more likely to make progress than those that don't receive any
    comments. Feel free to reach out to the RFC assignee in particular to get
    help identifying stakeholders and obstacles.
  - The team will discuss the RFC pull request, as much as possible in the
    comment thread of the pull request itself. Offline discussion will be
    summarized on the pull request comment thread.
  - RFCs rarely go through this process unchanged, especially as alternatives
    and drawbacks are shown. You can make edits, big and small, to the RFC to
    clarify or change the design, but make changes as new commits to the pull
    request, and leave a comment on the pull request explaining your changes.
    Specifically, do not squash or rebase commits after they are visible on the
    pull request.
  - At some point, a tech lead will propose a "motion for final comment period" 
    (FCP), along with a *disposition* for the RFC (merge or close).  
    - This step is taken when enough of the tradeoffs have been discussed that
      the team is in a position to make a decision. That does not require
      consensus amongst all participants in the RFC thread (which is usually
      impossible). However, the argument supporting the disposition on the RFC
      needs to have already been clearly articulated, and there should not be a
      strong consensus *against* that position. Teamvmembers use their best 
      judgment in taking this step, and the FCP itself ensures there is ample 
      time and notification for stakeholders to push back if it is made prematurely.
    - For RFCs with lengthy discussion, the motion to FCP is usually preceded by
      a *summary comment* trying to lay out the current state of the discussion
      and major tradeoffs/points of disagreement.
  - The FCP usually lasts ten calendar days, so that it is open for at least 5 
    business days. It is also advertised on the `development` Slack channel. 
    This way all stakeholders have a chance to lodge any final objections before 
    a decision is reached.
  - In most cases, the FCP period is quiet, and the RFC is either merged or
    closed. However, sometimes substantial new arguments or ideas are raised,
    the FCP is canceled, and the RFC goes back into development mode.

## The RFC life-cycle
[The RFC life-cycle]: #the-rfc-life-cycle

Once an RFC becomes "active" then it is an official part of "the Signal Noise
approach" and all developers should abide by it wherever possible. In common
with the "defaults, not rules" core approach it will is permissable to run a
project against the mandate of one or more specific RFCs but the rationale 
for ignoring each RFC must be documented in the project's decision log.

Modifications to "active" RFCs can be done in follow-up pull requests. We
strive to write each RFC in a manner that it will reflect the final design of

In general, once accepted, RFCs should not be substantially changed. Only very
minor changes should be submitted as amendments. More substantial changes
should be new RFCs, with a note added to the original RFC. Exactly what counts
as a "very minor change" is up to the team to decide on an ad hoc basis.


## Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC pull request is up, the team may want to discuss the issues in 
greater detail, usually at the Dev Roundup. A summary from the meeting will 
be posted back to the RFC pull request.

The team makes final decisions about RFCs after the benefits and drawbacks
are well understood. If the reasoning for the decision is not clear from the 
discussion in thread, a senior team member will add a comment describing the
rationale for the decision.


### Help this is all too informal!
[Help this is all too informal!]: #help-this-is-all-too-informal

The process is intended to be as lightweight as reasonable for the present
circumstances. As usual, we are trying to let the process be driven by
consensus and community norms, not impose more structure than necessary.


## License
[License]: #license

This repository is licensed under the MIT license ([LICENSE](LICENSE) or http://opensource.org/licenses/MIT)

*With thanks to the RFCs process used by [React](https://github.com/reactjs/rfcs) and [Rust](https://github.com/rust-lang/rfcs).
