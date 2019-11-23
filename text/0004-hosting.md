- Feature Name: `hosting`
- Start Date: 2019-11-23
- RFC PR: [signal-noise/rfcs#0000](https://github.com/signal-noise/rfcs/pull/0000)

# Summary

[summary]: #summary

A rundown of the common approaches to hosting used at Signal Noise, along with guidance about when to choose different setups and how to go about creating a new project.

We typically use one (or both) of [AWS](https://aws.amazon.com/) and [GCP](https://cloud.google.com/) for hosting when possible. We also have legacy projects on other platforms including Heroku but they are not discussed in this document.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

## Setting up a new project

### Secrets

.env

dashlane

### Setting up a new AWS account

### Setting up a new GCP project

### Static hosting on AWS

### Static hosting on GCP

### Dynamic hosting on AWS

### Dynamic hosting on GCP

## Accessing resources

### AWS

### GCP

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Domains

## AWS account structure

We have multiple AWS accounts set up as parts of an [AWS Organisation](https://aws.amazon.com/organizations/). This allows us to improve security for specific projects and across client hosting, as well as to make project billing simpler and clearer.

The AWS accounts we run are:

### Master

This is used for billing, internal team IAM and audit only. No resources should be created in this account.

### Sandbox

This is for experimentation. All developers have admin access to this account, but any resources they create should be terminated ASAP, and may be terminated at any time.

### Internal

For projects where we are the client - internal tooling, "FTP", internal projects etc.

### Legacy

(to be shut down)

### Development

All other environments for all other projects should go into the Development account. Deleting any resource in this account should have low impact.

### Specific Client accounts

Projects should go in a Specific Client AWS account if they are for a client with sensitivities around IP or data, _or_ if we are doing production hosting. All environments for all related projects should go into this account

### Specific Project or workstream accounts

A Specific Project or Stream AWS account is only merited in two cases:

- if we are doing production hosting for a client billing entity that's distinct from the rest of the client organisation. All related environments should go into this account
- If we have a project with a Specific Project Production account - in this case all nonproduction environments for that project should go here

### Specific Project Production accounts

​
Projects should get a Specific Project Production AWS account only if they are very sensitive (data or IP) _and_ if we are doing production hosting. Only a single project environment should be in this account, for security reasons.

## GCP account structure

With GCP each project should get a distinct Project, with all resources under that same entity, except for sensitive projects that may want the production environment split off.

# Drawbacks

[drawbacks]: #drawbacks

Why should we _not_ do this?

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
