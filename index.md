# Signal Noise coding guidelines

> This site hosts the RFCs for the Signal Noise organisation, that together define the approaches and tools that we take to create our work.

The aim with all the rules is to minimise the administration needed to stick to them, while saving time in the long term (i.e. thinking about returning to the project, bugfixing, etc). Where possible `all rules have tools` -- automation should support any new rule so that we're not adding to overheads, basically making implementation almost trivial.

> The first aim for all of our work is **quality**.

We're known for high quality visualisations -- but the builds that deliver those visuals, the interfaces that support the interaction, and the overall user experience all need to be top notch to have the greatest possible impact on users.

## Heuristics to follow

- Our priority is the best User Experience
- Complexity should be introduced only when it's inevitable
- Code should be easy to understand and reason about
- Code should be easy to change or delete
- Avoid abstracting too early
- Avoid thinking too far in the future

## Defaults, not rules

We have processes to keep the quality bar high, making it quicker and easier for people to start on projects, join them later, and reason about them after the event -- all while ensuring shortcuts are kept to a minimum.

But... we're a studio that works on a lot of different projects, using a mixture of permanent and freelance coders. Projects may be short, time-sensitive builds or long product builds; they may be targeting web users or controlled screens, or even AR, VR or other deliverables.

No one set of rules will cover all eventualities; and we want to be free to choose the best solution for the job while being conscious of the need to stick to our general approaches. You should think of everything here as *defaults* rather than *rules* -- except that the only rule is:

**If you break a rule, document your reasoning**

## Getting started

You should start by reading about our [Git Workflow](./text/0001-git-workflow) and the associated [CI/CD approach](./text/0003-continuous-integration.md). If you're setting up a new project you'll probably need to read about [hosting](./text/0004-hosting.md) as well.

Use a version manager for all your runtimes; examples include [nvm](https://github.com/creationix/nvm) and [pyenv](https://github.com/pyenv/pyenv).

Your project should say which version of each tool should be used; if it's a new project start off by specifying the version of everything you're using in the README. 

Anything involving back end code should be built inside one or more docker containers as that makes hosting -- and returning to the project -- much easier.

**Never** commit any tokens, passwords or similar to any repository, for any reason. Always introduce these as environment variables to your project. You can save an `.env` file for your project (or your project/environment combination) to Dashlane for other devs to access. We are also working on an internal `secrets` implementation but this is not yet ready.

## Documentation

Make sure there's enough for a stranger to get your project running, fix a bug and deploy it. That stranger might be you a year from now, or your successor who ends up badgering you for information.

Every project should have a decent README. Ideally there's a decision log in one form or another, and any detailed docs go in a `docs` folder.

## Linting

Always lint your code. Stylistic linting is important as it makes your code more shareable. Our rules will usually be the most widely-used community defaults where possible; any exceptions will be handled with automation. When you're working on a project with Javascript, please follow the [Linting and Formatting](./text/0002-tooling-linting-formatting) guide.

### Rationale

This is taken from [Python's PEP8 rule](https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds), but is generally relevant. 

TLDR; follow the guides but be sensible about it.

> One of Guido's key insights is that code is read much more often than it is written. The guidelines provided here are intended to improve the readability of code and make it consistent across the wide spectrum of Python code. As PEP 20 says, "Readability counts".
>
> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is the most important.
>
> However, know when to be inconsistent -- sometimes style guide recommendations just aren't applicable. When in doubt, use your best judgement. Look at other examples and decide what looks best. And don't hesitate to ask!

## Deploying

Always use CI deployment tools. Never deploy by hand: it's error-prone and stressful. CI also has the benefit of making a clean environment in which your code runs; this is handy when returning to a project as all system tools should be version pinned (never use `latest` version; always use a specific version) and encapsulated. Read the full guide in [the CI RFC](https://signal-noise.github.io/rfcs/text/0003-continuous-integration.html).

## Testing

We love automated testing, and everything we do goes through a QA process. 

As a studio that creates a wide range of outputs our testing philosophy changes somewhat between projects. 

Unit tests aren't required across the board, but should be easy to write and implement when necessary. That means please make sure there's a test harness that's easy to run for any new project.

A good place to add unit tests is during the QA process; before addressing a bug first replicate it with a test. This will prevent regressions (which are the bane of many a good team on a deadline).

As a general guide, follow the testing pyramid, and specifically for unit testing:

- Always ensure your code has a testing harness in place so it's easy to add a unit test
- Always ensure any data manipulation code has at least basic unit test coverage
- Generally try to write unit tests to replicate bugs found in QA

## Evolving the rules

We follow [an RFC process](https://en.wikipedia.org/wiki/Request_for_Comments) to write our team guidelines, managed via our [RFC Github repository](https://github.com/signal-noise/rfcs). If you would like to add a new guideline or change any of the existing rules, follow the instructions in the main README so the team gets to debate any changes.

Note that all debates and comments are kept private within the team, but that **the final ratified rule is [publicly accessible](https://signal-noise.github.io/rfcs/)**.

For reference, all the approved RFCs are listed here:

* [0001 Git Workflow](./text/0001-git-workflow)
* [0002 JS Tooling - Linting and Formatting](./text/0002-tooling-linting-formatting)
* [0003 Continuous Integration](./text/0003-continuous-integration.md)
* [0004 Hosting](./text/0004-hosting.md)
