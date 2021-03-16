---
title: CI/CD
description: Standardised approach to auto deployment
---

- Feature Name: `continuous-integration`
- Start Date: 2019-11-18
- RFC PR: [signal-noise/rfcs#0005](https://github.com/signal-noise/rfcs/pull/0005)

# Summary

[summary]: #summary

A standard approach to using CI/CD, including tooling, environment naming, hosting and configuration.

# Motivation

[motivation]: #motivation

Using CI/CD minimises stress and de-risks the deployment process. By standardising our approach, we make it feasible to have on all projects with reusable tools and configurations, leading to minimal setup time and low mental load when project switching.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Our approach to CI/CD is to try to standardise every aspect of the process while allowing for flexibility in both the project-specific hosting requirements, and the points at which different parts of the process are activated.

The process we follow is tightly integrated with our [Git Workflow](./0001-git-workflow), which you should probably understand before going further with this document.

It may be useful to start by reading about the standard [Environments] we support, as everything hinges off those.

## Day to day use

[day to day use]: #guide-use

If you're new to Signal Noise the chances are your project is already up and running with this process - if not you may want to jump ahead to the [Setup] section before continuing.

All environments built using our tooling will appear in the Github interface in the Code > Environments section.

### Pull Requests

The first point in the workflow that you'll encounter an automated deployment is during the Pull Request process. As soon as you create a pull request, or push new changes to a branch with an open PR, a new deployment will be created.

The deployment will create an environment called `prxxx`, where `xxx` is the PR number, for example `pr27`.

The deploybot agent `cimon-sn`'s activity will appear on the PR activity feed, with the most recent event having a button linking to the built environment, with the text `View Deployment`.

You'll also be able to go to any of the test devices and find the latest project build from the [Earl] dashboard, which should be set as the browser homepage.

### Merging to main

Once a PR has been merged to `main` (or in fact any time the main branch is updated), the [Preview] environment is rebuilt. The link to this environment should be pinned in the project's Slack channel as it's the best place for internal team members to review the project's latest state.

It's best practice to ensure that the Preview environment is password protected, with all Signal Noise email addresses allowed to view it.

In any case, please never share this link with clients.

### Starting the release process

Once a formal release is planned (i.e. where the end to end QA process takes place), an auto-built [Test] environment is useful.

In some cases with larger teams it can be useful for some of the team to continue to work on new features while others support the QA process, perhaps unblocking it with quick hotfixes. To support this process, the Test environment always builds from the most recent push to any branch starting with the word `release`.

The idea is that a new branch, e.g. `release/sprint-2` is created from the main branch, which automatically triggers a build of the Test environment. Should QA be blocked because of an easily-fixed error, a hotfix can be committed directly to this branch, which causes the Test environment to be automatically updated.

### Client review

Once the project is ready for UAT and client review (or really any time we'd like the client to review it), we can manually trigger a build to the [Staging] environment, from Slack. The command is simply:

`/cimon deploy release/sprint-1 to staging`

where `release/sprint-1` is the branch you'd like to deploy.

In order to allow the flexibility we often need, Staging builds can be triggered from any branch and are commonly from the main branch.

It should be noted that anyone with access to the relevant Slack channel can trigger a build to Staging.

Note that the URL for this environment should be the only place the client sees code before it's live, since it's the only place we have manual control and can make sure they see a stable version, or a version that has specific features enabled that they may require in order to update their own internal stakeholders.

### Deploying to production

Deploying to the [Production] environment (or creating a production build, which is sometimes not the same thing) is easy and stress-free with this setup. We recommend it's done by the project Producer.

It's possible to do this by creating a tag in the repo starting with `v`, but the best way is to create that tag via the Github Releases interface (just ensure you specify a `v` as the first character of the tag and ideally a semver number following that).

That process allows the deployer to focus on writing an explanation of what is contained in the release, ideally referencing documentation and/or ticket numbers.

Many project builds will automatically update the release packages to contain the built code, which is also good practice.

## Setup

[setup]: #setup

Setting up the tooling on a new project requires configuration of a few distinct areas.

### Hosting

The first step is to [configure the project infrastructure](./0004-hosting), allowing for (hopefully all) the standard [environments].

Note that with PR builds you may end up with hundreds of individual environments, and will need a way to manage this. For static sites, Google AppEngine is a good solution to this, while using subfolders of a single AWS S3 bucket will also often work, although HTTPS and password protection can be tricky. See the [Hosting RFC](./0004-hosting#aws-static-password) for more details.

### Build

As standard we use CircleCI to build our projects. Create the `.circleci/config.yml` file and add in all the instructions you'll need to build to your environments.

You'll need to ensure that you don't trigger any build jobs with the usual CircleCI triggers (e.g. workflows) but only use these for linting and testing.

Instead, all your build instructions must be contained in a single `build` job that can handle any environment, using the following environment variables that it will receive:

- `$ENVIRONMENT` (e.g. `preview` or `pr27`)
- `$VERSION` (e.g. `1.2.6` or `d70375e`)
- `$URL` (e.g. `https://preview-dot-myproject.appspot.com`)

You will also need to include a top level `notify` stanza in the config file, in order to update the tooling on the build status.

An example config file for a static project hosted on GAE, with linting on each push might be as follows:

```yaml
version: 2

jobs:
  test:
    docker:
      - image: eu.gcr.io/signalnoise/base:14
    steps:
      - checkout
      - run: npm ci
      - run: npm run lint

  build:
    docker:
      - image: eu.gcr.io/signalnoise/base:14
    steps:
      - checkout
      - run: npm ci
      - run:
          name: Build
          command: NODE_ENV=${ENVIRONMENT} npm run build
      - run:
          name: Creating Archive
          command: |
            echo $CIRCLE_SHA1 > ./public/commit.txt
            echo $VERSION > ./public/version.txt
            mkdir -p /tmp/build-artifacts && \
            tar -zcvf "/tmp/build-artifacts/${GOOGLE_PROJECT_ID}_${VERSION}.tgz" ./public
      - store_artifacts:
          path: "/tmp/build-artifacts/"
      - run:
          name: Store GCP Service Account
          command: echo $GCLOUD_SERVICE_KEY > ${HOME}/gcloud-service-key.json
      - run:
          name: Authenticate and set defaults to environment
          command: |
            gcloud auth activate-service-account --key-file=${HOME}/gcloud-service-key.json
            gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
            gcloud --quiet config set compute/zone ${GOOGLE_COMPUTE_ZONE}
      - run:
          name: Deploy to GAE
          command: |
            if [[ ${ENVIRONMENT} == "production" ]]; then
              gcloud app deploy
            else
              gcloud app deploy --quiet --no-promote --version ${SUBDOMAIN}
            fi
      - run:
          name: Notify Earl
          command: |
            curl -X POST -H "Content-Type: application/json" \
              -d '{"url": "'"$URL"'","project": "myproject","environment": "'"$ENVIRONMENT"'"}' \
              https://post-url-for-earl
notify:
  webhooks:
    - url: https://post-url-for-deploybot

workflows:
  version: 2
  test_and_build:
    jobs:
      - test
```

> Note that in this example the $GOOGLE_PROJECT_ID and $GCLOUD_SERVICE_KEY environment variables would be set in the CircleCI project settings

Some important pieces of the above to mention include

```yaml
- image: eu.gcr.io/signalnoise/base:14
```

Signal Noise run our own [generic CI containers](https://github.com/signal-noise/containers) that are lightweight but have the tooling required for most builds pre-installed.

```yaml
- run:
    name: Creating Archive
    command: |
      echo $CIRCLE_SHA1 > ./public/commit.txt
      echo $VERSION > ./public/version.txt
```

This is an important step; we almost always write static files containing just the commit hash and the version number (sometimes the same thing) to text files in the project's public root. This can be a lifesaver when debugging.

```yaml
            mkdir -p /tmp/build-artifacts && \
            tar -zcvf "/tmp/build-artifacts/${GOOGLE_PROJECT_ID}_${VERSION}.tgz" ./public
      - store_artifacts:
          path: "/tmp/build-artifacts/"
```

In addition, we're creating a tarball of the built code in this step and uploading it the the CircleCI archive space, which gives us a reference later should we need it.

```yaml
- run:
    name: Deploy to GAE
    command: |
      if [[ ${ENVIRONMENT} == "production" ]]; then
        gcloud app deploy
      else
        gcloud app deploy --quiet --no-promote --version ${SUBDOMAIN}
      fi
```

Aside from the deployment command, it's important to note that we're handling all different environments' requirements in this same build job. More complex builds may need to use bash scripts to do this effectively.

On some projects, our production "deployment" actually requires us to deliver a package of the code to the client, who then manually implements the deployment. The best way to achieve this is to use the GHR tool, which will add the archive GZIP bundle to Github's Releases page under the relevant release. An example Deploy step using that approahc might look be as follows:

```yaml
- run: 
    name: Deploy to S3 or Release to GitHub
    command: |
      if [ ${ENVIRONMENT} == "production" ]; then
        ghr -t ${GITHUB_TOKEN} -u ${CIRCLE_PROJECT_USERNAME} -r ${CIRCLE_PROJECT_REPONAME} -c ${CIRCLE_SHA1} -replace ${CIRCLE_TAG} /tmp/build-artifacts/
      else
        gcloud app deploy --quiet --no-promote --version ${SUBDOMAIN}
      fi
```


```yaml
- run:
    name: Notify Earl
    command: |
      curl -X POST -H "Content-Type: application/json" \
        -d '{"url": "'"$URL"'","project": "myproject","environment": "'"$ENVIRONMENT"'"}' \
        https://post-url-for-earl
```

Posting the build to [Earl] will create a link to this environment that's easy to get to from any of the Signal Noise test devices.

```yaml
notify:
  webhooks:
    - url: https://post-url-for-deploybot
```

This is crucial since without it [Deploybot] won't know the build status, and the Github interfaces will show the environment as unreachable, and won't publish any links.

```yaml
workflows:
  version: 2
  test_and_build:
    jobs:
      - test
```

The main thing to note here is that only the `test` job is triggered by internal CircleCI triggers - the `build` job is exclusively triggered by Deploybot, which is the next thing to set up.

### Orchestration

To set up deploybot, first identify a Slack channel to use for project devops - usually this is the project channel or a specific project/dev channel that has feeds from e.g. issues. You can only set up a single deploybot project in a single channel, and vice versa.

In that channel, type `/cimon setup signal-noise/reponame` for whatever your `reponame` is. If you intend to use a repo that isn't in the Signal Noise namespace you'll have to edit the GH bot's permissions.

The next commands to run depend on your environment; either way you need to set the domain that the environment URLs will be based from, by typing `/cimon set baseurl yourdomain`.

Deploybot expects to generate environment URLs from a main domain, for example https://preview.yourprojectdomain.com, but most aspects of this can be customised. By default deploybot uses the following pattern to generate environment URLs:

```
https://{environment}{url_separator}{baseurl}/
```

> HTTPS is required for these URLs by GitHub, which can cause issues for AWS S3 environments.

You can change the pattern by typing `/cimon set url_pattern your-pattern` but ensure you have all the above variables in your pattern or deployments will break. You can also override the pattern for specific environments with, for example, `/cimon set url production https://signal-noise.co.uk`. The `url_separator` defaults to `.` but can also be overridden to any string with, for example `/cimon set url_separator -dot-`. Note that the `environment` variable is derived by deploybot itself depending on the build trigger and can't be overridden in the settings.

Lastly, typing `/cimon help` in the Slack interface will give you a basic overview of the available commands.

From this point forwards you should be seeing automated builds of your environments in the Github interface, most easily under Code > Environments.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

This section covers our standard [Environments], the tool we use to trigger all the builds, integrate the services and manage the workflow — [Deploybot] — and the service that we use to run the builds themselves: [CircleCI].

## Environments

[environments]: #environments

We have several standard environments that we build to regularly. Not every project needs or uses each environment, so feel free to only trigger builds to the ones that make sense for the project you're working on.

Environment names are important; if you alter the naming you may run into issues with some of the automated tooling.

The environments our tooling caters for are as follows:

| Environment  | Trigger   | Branch/Tag                   | Best practice example                                                          |
| ------------ | --------- | ---------------------------- | ------------------------------------------------------------------------------ |
| [Production] | Manual    | Tag: `v*`                    | `v0.2.4` created with Github Release                                           |
| [Staging]    | Manual    | Any                          | From branch `release/0.2.4` at a point the team feels comfortable with sharing |
| [Test]       | Automatic | Branch: `release*`           | From branch `release/0.2.4` each time it is updated                            |
| [Preview]    | Automatic | Branch: `main`             | From branch `main` each time a PR is merged in                               |
| [PR]         | Automatic | Branch: Any involved in a PR | From branch `issue-65` with open PR to merge into `main`                     |

### Production

[production]: #production

**The live environment**

We always prefer to integrate deployment to the production environment with our tooling whenever possible, even if the hosting is managed by the client or another third party (many tech teams agree with this approach).

When the project delivery involves a build but not a deployment (e.g. we create a bespoke JS module for a client to manually add to their CMS) we still treat the production build process in the same way, for consistency.

Deployments to production should **always** come from a tag starting with `v`. The rest of the tag name should be a semantic version number. Ideally, the tag creation itself will trigger the production deployment (this is the default if you have [deploybot] set up).

The best way to deploy to production is from the GitHub interface. In the Code section is a Releases subsection. As well as creating a tag, a Github Release allows for documentation to be added that stays with the repo, as well as providing a place to keep persistent, unchanging, downloadable asset bundles.

Our preference is for the project managers to trigger the production deployments of our code, as inevitably they have to deal with the consequences first.

Note that technically speaking we don't wait for checks to pass, so it's currently possible for a production deployment to be triggered on code that fails the automated tests, although this won't happen if all processes are followed.

### Staging

[staging]: #staging

**The client review environment**

It goes without saying that this should be as close a mirror of the production environment as possible (including infrastructural optimisations like caching layers, autoscaling groups etc).

Clients should _not_ be looking at any other non-production environments. The builds to this environment are always manually triggered from Slack, which means that there ought to be no surprises in what the client is looking at.

Best practice is to trigger these builds from a `release/xxx` branch of the repository soon after reviewing the [Test] environment, which is automatically built from the same branch.

By triggering these deployments from the Slack channel where [deploybot] is configured, everyone with access to the channel is _de facto_ notified of the deployment.

### Test

[test]: #test

**The QA environment**

Ideally this environment is an exact mirror of the staging environment, and therefore very similar to the production environment.

Formal end to end testing and pre-release QA should take place on this environment.

In our process, this environment is automatically built when code is pushed to a branch in the repository whose name starts with `release`. Note that this means that hotfixes to a release branch will automatically trigger a rebuild.

### Preview

[preview]: #preview

**The internal review environment**

This environment may be reviewed at any time by any project stakeholder, or indeed anyone in the company. Every project should have this environment as it simplifies keeping people up to date and allows easy reviews in internal meetings.

This environment is automatically built every time code is pushed to the `main` branch of the repository, which should be when a PR is merged.

### PR

[pr]: #pr

**The PR review environment**

This environment is intended to be short-lived and act as a way to review the changes proposed in a specific PR. As such the environment name will have the PR number in it - e.g. `pr326`.

Our tooling builds/rebuilds this environment automatically each time a PR is updated, which is to say the trigger is a Github API action that maps to a push to a branch which has an open PR to another branch.

The tool will automatically post a link to the environment to the PR interface, making it easy for anyone with access to use this environment for review purposes.

In practice there is no current workflow for destroying environments, and so care should be taken that we don't waste resources on unnecessary, out-of-date PR environments.

## Deploybot

[deploybot]: #deploybot

Deploybot is an [open source project](https://github.com/signal-noise/deploybot) written at Signal Noise to support our workflow by triggering builds to the above environments. It is tightly integrated with Github, and loosely with [CircleCI] and Slack.

A single deploybot instance runs on AWS Lambda and DynamoDB and serves all Signal Noise's projects.

Deploybot is configured in Slack and associates a single Slack channel with a single Github repository. Anyone with access to the Slack channel is able to control and configure deploybot.

Deploybot subscribes to events for all repositories it tracks, and parses the event stream for triggers as laid out in the [Environments] section. Some builds can also be manually triggered from the Slack channel used for that repository.

When a trigger is received, deploybot uses some basic logic to determine the right environment to build, and registers a [deployment](https://help.github.com/en/github/administering-a-repository/viewing-deployment-activity-for-your-repository) with Github for that project. This causes Github to create a new tab in the interface where all deployments are listed with their statuses, and also to keep track of PR-specific deployments on the relevant PR page.

When the Github event feed acknowledges a deployment creation, deploybot uses the CircleCI API to trigger a build - when it finishes CircleCI calls back to deploybot which then updates the deployment status on Github.

## CircleCI

[circleci]: #circleci

[CircleCI](https://circleci.com/) is currently used as the CI provider on all Signal Noise's projects since its configuration stays in the repository, the pricing is fair and the developer experience is reasonable.

Developers with access to a Github repository can use their GH logins to access CircleCI, and their GH-registered SSH keys to tunnel into running containers in CircleCI.

Shortcomings in its API mean that each configuration must contain (possibly among other things) a single job named `build` - this is the only task that will be triggered by deploybot and needs to complete all tasks for any environment build.

Deploybot will send several parameters to the build job, including the environment name and version number, as well as the specific git ref to use.

## Earl

[earl]: #earl

Earl is another [open source project](https://github.com/signal-noise/earl) created at Signal Noise. It simply catches the notifications of all the builds and uses them to create a web dashboard.

This dashboard should be set as the homepage of the test devices, so that it's easy to go and preview any environment (often with long, complex and fiddly URLs) on any of our devices.

# Drawbacks

[drawbacks]: #drawbacks

There are drawbacks to this approach, and to the choice of tooling that we currently use.

By attempting to standardise all environments and workflows we sacrifice flexibility on a per-project basis in favour of a quick and easy setup in general. There may be occasions where this flexibility is missed, although those seem rare.

CircleCI has a poor API that imposes limitations on the way we can configure the builds, including meaning that it's not feasible to have a configuration for destroying environments.

The CircleCI default triggers are not nuanced enough to support our workflow (e.g. tag and PR creation don't always get caught), meaning that a new tool (Deploybot) has had to be built to support this. Deploybot is not flexible or especially robust and will have its own maintenance overhead.

Additionally, full deployments can be triggered despite checks (e.g. linting, testing) passing. It is possible to create a sequential stream of actions but this slows the CI/CD cycle down a lot. Ideally, if the CircleCI (or equivalent) API had greater granularity in terms of the 'jobs' that could be triggered, it would be possible to run tests and code builds concurrently, while holding off on deployments (perhaps on of some environments) until all checks pass.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

CI systems allow us to automate deployment of our code to different environments. This is important for a few different reasons.

First and foremost, deploying from a single developer's machine is error-prone and difficult to replicate; this can result in very stressful moments, especially when a new developer is doing it for the first time. Having an automated system do the work also frees up developer time while making it trivial to have many standard points at which code is deployed.

Lastly, as long as the automation is set up with pinned version numbers for all technologies, it gives future developers (i.e. you in a year) access to a reference working environment for the code.

## Alternatives

CircleCI has many competitors, but many of them are less easy to use and configure. A significant factor in choosing it was that the configuration lives in the codebase; also that the pricing was favourable. Certainly we would favour using a hosted service over e.g. Jenkins, which brings its own maintenance overhead.

Some potentially compelling alternatives include Github Actions (which is not yet fully stable) and Google Code Build (which has more expensive pricing).

Deploybot has a few competitors but none that simply deliver the workflow required, combined with the ease of Slack-based configuration.

# Prior art

[prior-art]: #prior-art

Most examples of CI/CD setups in use (and it is extremely common) have very different foundational requirements to those of Signal Noise - essentially working in an agency model with many projects (often with short turnarounds) we need a setup that combines some flexibility with a lot of standardisation. Examples of places with similar requirements putting a system such as this in place are thin on the ground.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

The greatest issue with the current setup is a lack of a way to destroy environments, e.g. when a PR is merged or closed, though it would also be good to parallelise only some of the build+deploy tasks (e.g. to prevent production deploying code that doesnt pass checks).

We've been using this system for almost year now and aside from the above, in general it works well.

# Future possibilities

[future-possibilities]: #future-possibilities

CircleCI has introduced a new pricing model which may precipitate exploring other CI hosted service providers. These should be assessed against the following criteria:

- Is configuration stored as code, in the repo? (This is a must have in order to keep the project easy to maintain)
- What is the pricing model? (per-seat pricing can end up extremely expensive for us)
- What is the developer experience (i.e. how easy is it to debug build issues?)
- What functionality does the API have? Can we trigger builds, specifying environment, version and URL, and get a status callback? If not, can we support all the triggers we currently have directly through the CI service?

There are also improvements that could be made to deploybot in the future, especially around:

- Increased flexibility (i.e. connecting to different CI providers, supporting different workflows)
- Increased maintainability (it's a complex set of Lambda functions with a difficult deployment path)
- Increased functionality (its communication with Slack is very basic; it could provide more complete and useful feedback for example)
