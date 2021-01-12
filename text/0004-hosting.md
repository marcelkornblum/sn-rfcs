---
title: Hosting
description: How we approach hosting on AWS and GCP
---
- Feature Name: `hosting`
- Start Date: 2019-11-23
- RFC PR: [signal-noise/rfcs#0006](https://github.com/signal-noise/rfcs/pull/0006)

# Summary
[summary]: #summary

A rundown of the common approaches to hosting used at Signal Noise, along with guidance about when to choose different setups and how to go about creating a new project. Note that this is not exhaustive and shouldn't be seen as a replacement for choosing the right setup and/or settings for your project's needs; it's more of a guide to our default approach.

We typically use one (or both) of [AWS](https://aws.amazon.com/) and [GCP](https://cloud.google.com/) for hosting when possible. We also have legacy projects on other platforms including Heroku but they are not discussed in this document.

If you're interested in the general rules please jump straight to the **[Reference-level explanation](#reference-level-explanation)**; otherwise read on for guidance for specific situations you may be in.

# Guide-level explanation
[Guide-level explanation]: #guide-level-explanation

Jump straight to the section that is most relevant, or (preferably) read through them all first:

* [Choosing between AWS and GCP](#choosing-between-aws-and-gcp)
* [Setting up a new project](#setting-up-a-new-project)
* [Static Hosting on AWS](#static-hosting-on-aws)
* [Static Hosting on GCP](#static-hosting-on-gcp)
* [Dynamic Hosting on AWS](#dynamic-hosting-on-aws)
* [Dynamic Hosting on GCP](#dynamic-hosting-on-gcp)
* [Setting up a new AWS account](#setting-up-a-new-aws-account)
* [Setting up a new GCP project](#setting-up-a-new-gcp-project)
* [Accessing resources](#accessing-resources)


## Choosing between AWS and GCP
[Choosing between AWS and GCP]: #choosing-between-aws-and-gcp

In general terms, AWS and GCP have near parity in terms of features they offer, and usually the choice of which to take has no serious technical weight. That said, there are some circumstances which make one or the other a better choice for your project, should the client not be mandating which platform you use.

Ideally, we write code that is able to painlessly be hosted on services provided by any major hosting platform, but in real life there are reasons to create projects that are opinionated about where they are hosted. 

If you're building a project which will run as a static site (or you're confident that it'll be easily hosted elsewhere; perhaps you're using Kubernetes), there's no reason not to use GCP for pre-production environments and AWS for production (and some reasons that is a good idea, see below).

You might choose AWS for your project / environment if:
* Your production environment has to be on AWS (perhaps it's client mandated)
* You are using a service provided by AWS without a close analogue on GCP (especially e.g. an Alexa service)
* You are using a tool or toolchain that is AWS-specific
* The people involved in the day to day on the project are more comfortable with AWS

You might choose GCP for your project / environment if:
* Your production environment has to be on GCP (perhaps it's client mandated)
* Your project hosting requirement is [static files](#static-hosting-gcp) only (*static works better on GCP than AWS*)
* You need basic password protection on your static site, perhaps on non-production environments (this is easy in GCP and complex in AWS)
* You are using a service provided by GCP without a close analogue on AWS (especially e.g. an ML service)
* You are using a tool or toolchain that is GCP-specific
* The people involved in the day to day on the project are more comfortable with GCP

All else being equal, we usually prefer GCP over AWS since it involves less management overhead:
* The management of projects and billing is clearer, easier and takes less time
* The management of project resources is simpler, taking into account pricing structures and security optimisation

## Setting up a new project
[Setting up a new project]: #setting-up-a-new-project

When setting up a new project, the first thing to consider is the hosting provider or combination of providers. A common scenario is to use GCP for non-production hosting, with AWS for production and staging environments.

If using AWS, the second step is to check whether an existing AWS account would be appropriate for your project, or if you need to set up a new account before proceeding.

Next, you'll need to set up your project environments. Usually you'll benefit by setting them all up at the same time (generally you'll want one for each of the environments set out in our [CI/CD approach](./0003-continuous-integration), including a way to have unlimited PR environments).

You'll probably be setting up your CI/CD pipeline at the same time as your hosting; if not make sure you take notes of secret keys and credentials as you go, but that you **[manage them properly](#secrets)**.

## Static hosting on AWS
[Static Hosting on AWS]: #static-hosting-on-aws

To set up hosting a static project, or the static part of a larger project on AWS, you will need to follow this section. It seems daunting at first but once you know what you are doing it should take no more than half an hour. If you want to skip to a specific section in this topic use this submenu:

* [Get your environment name](#get-your-environment-name)
* [Set up your S3 Bucket](#set-up-your-s3-bucket)
* [Set up your CloudFront Distribution](#set-up-your-cloudfront-distribution)
* [Password protection of a static site on AWS](#password-protection-of-a-static-site-on-aws)
* [Set up your custom domain](#set-up-your-custom-domain)
* [Set up the credentials for deploying to your environment](#set-up-the-credentials-for-deploying-to-your-environment)

There are a few ways of achieving a static hosting setup and we usually do slightly different things for different environments...

For _Production_ we use S3 for file hosting, with CloudFront doing all the jobs of a webserver. We keep all credentials and permissions for the production environment separate to other environments.

For a _Staging_ environment we use the same S3 & CloudFront setup as production, but we generally make a single set of credentials and permissions to cover all non-production environments, for the sake of quicker setup and lower maintenance overhead.

_Test_ environments are usually set up using an S3 bucket with website hosting enabled but no CloudFront, as CF adds an extra layer of management as well as a longer wait (and extra complication) during deployments. 

_Preview_ and _PR_ environments can all be hosted in subfolders of a single S3 bucket (to avoid making potentially hundreds of buckets), again with S3 website hosting and no CloudFront setup. You can also setup _Test_ in the same way as these environments, to further reduce overhead. 

| Environment | S3 Bucket | Bucket website hosting | CloudFront setup | Security setup |
| --- | --- | --- | --- | --- |
| _Production_ | Dedicated | No | Yes | Dedicated |
| _Staging_ | Dedicated | No | Yes | Shared 'non-production' |
| _Test_ | Optional | Yes | No | Shared 'non-production' |
| _Preview_ | Shared | Yes | No | Shared 'non-production' |
| _PR_ | Shared | Yes | No | Shared 'non-production' |

Then, making sure you [choose the right AWS account](#account-folder-structure) for your project and environment, you need to follow the below instructions for each environment you'll need.

**NB. For Wordpress hosting, you may find it easiest / best to use AWS Lightsail which is cheap and easy to get going with.**

### Get your environment name

You need a unique name (which we'll refer to in this document as `<name>`) that will identify resources for this environment.  
 
We use the format `<projectnumber>-<projectname>-<environment>` if you're on a single-client AWS account, and making a single-environment resource. An example might be `1234-explainingdatawebsite-production`. 

On a [shared AWS account](#account-folder-structure) the format should be `<projectnumber>-<clientname>-<projectname>-<environment>`, for example `1234-teg-explainingdatawebsite-production`.

If you're making a shared resource (e.g. a security policy covering more than one environment) you may end up with a name along the lines of `1234-teg-explainingdatawebsite-nonproduction`.

### Set up your S3 Bucket

Create an S3 bucket called whichever `<name>` you are using for this environment, described above.

First, you can simply follow the bucket creation wizard, accepting all the default options, except that you should make sure not to block all public access as you're going to be hosting directly from the bucket (just uncheck the box in the creation wizard). You may want to avoid this in a few cases (e.g. if you are setting up [OAI access via CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)) but usually you'll want the content publicly accessible.

You may want to upload a file, just to have something to test your setup on.

Once you have a bucket, check if your site requires CORS (if you're loading files at runtime there's a decent chance you'll end up needing it) - if so, now is the time to set it up. In the bucket Permissions > CORS configuration, add the following (tweak the specific settings to be less permissive in production, by e.g. specifying domain names):

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```

If you **are using Cloudfront** for this environment, you are done for this step and can move on. Otherwise, there are a few more things to do.

Next, enable S3 website hosting in the bucket Properties > Static website hosting dialog, by selecting Use this bucket to host a website, and specifying `index.html` as the Index document (which will enable the save button). 

Make a note of the endpoint published on the following screen, and note that if you are sharing this bucket between environments your site root will be a folder inside the bucket (e.g. `http://<name>.s3-website.eu-west-1.amazonaws.com/<environment>/`).

In the bucket Permissions > Bucket Policy, you may want to add the following to automatically make all content public. If you don't do this you'll need to make sure each file is made public individually:

```json
{
    "Version": "2008-10-17",
    "Id": "Policy1397632521960",
    "Statement": [
        {
            "Sid": "Stmt1397633323327",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<name>/*"
        }
    ]
}
```

Lastly, go to the website endpoint you noted earlier and check that you can see the file you uploaded.

### Set up your CloudFront Distribution

Skip this step if you aren't making a production or staging environment. Otherwise, use the following settings unless you know what you're doing (anything that isn't mentioned is usually best left untouched).

1. Your Origin should be the bucket **website** url, especially if you're using Gatsby, as explained in [this page](https://www.gatsbyjs.org/docs/deploying-to-s3-cloudfront/#setting-up-cloudfront).
2. You should probably set _Viewer Protocol Policy_ to `Redirect HTTP to HTTPS`
3. If your site loads data at runtime you may need CORS set up. If so, you need to do [a few extra things](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/header-caching.html#header-caching-web-cors):
  * In the Behaviours section, select `Use legacy cache settings` (this is most likely to "just work" for you, but the new settings can be more optimised for better overall performance)
  * In the Behaviours section, select `GET, HEAD, OPTIONS` under _Allowed HTTP Methods_
  * In the Behaviours section, set _Cache Based on Selected Request Headers_ to `Whitelist`, and then select the following headers:
    * `Origin`
    * `Access-Control-Request-Headers`
    * `Access-Control-Request-Method`
  * Ensure you have CORS set up on the bucket (see above)
4. Optionally change the default cache times
5. Optionally set _Price Class_ to something cheaper than `All Edge Locations`

**NB.** It's really important to make sure you use the comment field to put the project name and ENVIRONMENT; when scanning a list of CloudFront distributions this is by far the easiest way to see which is which. It's also searchable.

Lastly, go to the Cloudfront distribution URL (https://xxxx.cloudfront.net) and check you can see your content.

### Password protection of a static site on AWS
[Password protecting a static site on AWS]: #aws-static-password

If you are hosting your static site on AWS as described in this guide, and you do need some sort of password protection, you will have to do a couple of extra things:
* Ensure your S3 bucket does **NOT** serve content as a website, and that content is not publicly accessible directly from it (NB this breaks Gatsby's deep linking functionality)
* Ensure you have a CloudFront distribution, accessing the S3 bucket content via the API (the default in the CF setup console wizard), using [Origin Access Identity](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-access-to-amazon-s3/)
* [Setup a Lambda@Edge function](https://medium.com/hackernoon/serverless-password-protecting-a-static-website-in-an-aws-s3-bucket-bfaaa01b8666) connected to your CF distribution to enforce HTTP Basic password protection

### Set up your custom domain

This only works if you've set up CloudFront - if you are only using S3 website hosting you can't have a custom domain. To set your domain up to work with your CloudFront instance, you will need access to the DNS records, most easily configured via Route53.

**NB.** Do not buy a domain for your project until you've read the [Domains] section of this document.

The first step is setting up a certificate to enable HTTPS on your domain. Go to the `ACM` service , and make sure you are in the **N Virginia `us-east-1`** region or your certificate won't work on CloudFront. Create a new certificate, attaching all the domains and subdomains you'll need for the project. You should avoid wildcard (*) subdomains if possible.

Verify the domains and subdomains on your certificate (using DNS is usually simpler, and with Route53 it's painless). Once it's verified, go back to the CloudFront distribution, where you can add the SSL certificate (it will autocomplete) and specify your CNAMEs for the custom domains against this domain.

Lastly, you need to add the CF's own domain as a CNAME record against each supported subdomain, which will route the traffic to the CloudFront instance.

### Set up the credentials for deploying to your environment

We're going to set up an AWS IAM Policy with limited credentials; just enough to deploy to our new static environment. For the sake of pragmatism we usually make a single policy for all non-production environments and a second set of credentials for production. Some projects require a stronger security posture and a separate set of credentials should be made for each.

The Policy defines a set of API calls that can be made (either via the CLI or the Console); we'll associate our policy with a new IAM User created specifically for that purpose, and also with the IAM Role that developers use to access resources, coming from the Master AWS account.

The end result will be a custom user that can be invoked by the CI system which has a very limited set of permissions, and also those exact same permissions extended to perm developers on the team where appropriate, along with the permissions from the other environments. Developers accessing resources in this way must do so via secure 2FA logins, and this gives us a balance between pragmatic access to resources and security. For some projects and environments we will not assign the policy to the main developer access role, but keep it separate as a single use role to be assigned to specific individuals.

In IAM create a Policy called `<name>` and paste the following JSON into the editor (note the `<variables>` you need to switch out for real values). 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObjectAcl",
                "s3:PutObject",
                "s3:GetObjectAcl",
                "s3:GetObject",
                "cloudfront:GetDistribution",
                "s3:GetBucketWebsite",
                "s3:DeleteObject",
                "s3:GetBucketAcl",
                "s3:GetBucketPolicy",
                "cloudfront:CreateInvalidation"
            ],
            "Resource": [
                "arn:aws:s3:::<name>/*",
                "arn:aws:s3:::<name>",
                "arn:aws:cloudfront::<AccountId>:distribution/<CloudFrontDistributionId>"

            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "cloudfront:ListDistributions"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**NB.** If you're not using CloudFront for this environment you can safely delete those lines. If you are using the same credentials for more than one environment, simply add the extra bucket and/or CloudFront lines to the resources section (making sure to have a line for each bucket and another line for `/*` - all the items inside the bucket).

Next, still in IAM, create a user also called `<name>`, and assign it the policy you just made. Give it programmatic access and make a note of the key and secret (but make sure the note can be deleted later; you don't want this in a repo, a chat message or anything persistent). This is the user the CI system will use to deploy your front end.

The next step is making it easy for perm Signal Noise devs to perform the same operations as the CI user you've just made (ie deploying). Simply attach the new Policy to the DeveloperAccessFromMaster Role that you should find in IAM if the account was set up according to these instructions. Note that for high security environments you may not want to perform this step, or you may want to attach the policy to a custom role not available to all developers. 

Lastly, [set up CI](./0003-continuous-integration) using the key and secrets you noted above (you should have at least two: production and non-production, and perhaps more per-environment). Test a CI build and make sure it puts the right code in the right place and that it's accessible via the CF endpoint.

## Static hosting on GCP
[Static Hosting on GCP]: #static-hosting-on-gcp

We often use GCP for hosting static projects, or the non-production environments of static projects, since the [AppEngine](https://cloud.google.com/appengine/) service gives the following easy-to-setup benefits over e.g. AWS S3:
* Free HTTPS endpoint with no setup (for a non-custom domain)
* Access controls, allowing only specified people, or e.g. only Signal Noise employees, to access the project, with no extra code
* Easy concurrent version hosting, making CI integration simple and painless
* Route mapping rules, for example making bare folders return index files (e.g. so `/about` maps to `/about/index.html`)

If you want a high performance static front end you should follow the [official tutorial](https://cloud.google.com/storage/docs/hosting-static-website), but to use our common GAE setup you need to follow these instructions. If you want to skip to a specific section in this topic use this submenu:

* [Set up your project](#set-up-your-project)
* [Set up your configuration file](#set-up-your-configuration-file)
* [Restricted access](#restricted-access)
* [CORS](#cors)
* [Integrating with CI/CD](#integrating-with-cicd)

### Set up your project

First of all [set up your GCP project and enable GAE](https://cloud.google.com/appengine/docs/standard/nodejs/building-app/creating-project) ensuring you add it to the Signal Noise organisation (you may need a tech lead to do this for you, and to enable billing via Signal Noise). Please also make sure it's in the right [organisation folder](#account-folder-structure).

You also need to [set up a Service Account](https://cloud.google.com/sdk/docs/authorizing#authorizing_with_a_service_account) for CircleCi to use for deployments. You'll need to assign the Service Account the following permissions:
   - `App Engine Deployer`
   - `App Engine Service Admin`
   - `Storage Object Creator`
   - `Storage Object Viewer`
   - `Cloud Build Service Account`
   - `Service Account User`
   
Download a JSON key for the user, and paste the whole JSON into the CircleCI > Project Settings > Environment Variables page as a var named `GCLOUD_SERVICE_KEY`, also setting the `GOOGLE_PROJECT_ID` and `GOOGLE_COMPUTE_ZONE` variables.

Lastly, enable the [Cloud Build API](https://console.cloud.google.com/marketplace/details/google/cloudbuild.googleapis.com), and the [App Engine API](https://console.developers.google.com/apis/api/appengine.googleapis.com/overview) needed for auto-deployments to GAE now.

### Set up your configuration file

You need to create a configuration file in your project next - the below works well for the static use case (save it in your repository root as `app.yaml`):

```yaml
runtime: python27
api_version: 1
threadsafe: true

handlers:
  - url: /
    static_files: public/index.html
    upload: public/index.html
    secure: always
    login: required

  - url: /(.*\..*)
    static_files: public/\1
    upload: public/(.*)
    secure: always
    http_headers:
      Access-Control-Allow-Origin: "*"

  - url: /(.+)/
    static_files: public/\1/index.html
    upload: public/(.+)/index.html

  - url: /(.*)
    static_files: public/\1/index.html
    upload: public/(.*)
    secure: always

skip_files:
  - ^(?!.*public).*$
```

Note that this assumes that your built code lives in a folder called `public`; you may need to change this for your project. 

### Restricted access 

In the above config, we have set it to not be publicly accessible, using the following line:

```yaml
    login: required
```

This enables auth in front of your project, but note that by default it will still be open to anyone with a Gmail account; you need to [set up IAP](https://cloud.google.com/iap/docs/app-engine-quickstart#enabling_iap) next. In order to set up IAP, you may first need to configure the oAuth credentials screen - just do the minimum required. 

We generally set IAP to the whole `signal-noise.co.uk` domain so that internal employees can view projects (make sure the `IAP` toggle is on, and assign the `IAP-secured Web App User` permission to a 'Member' set as the bare domain `signal-noise.co.uk`), sometimes adding specific clients later in the project lifecycle.

If you want to remove this restriction just on some environments, you'll need to edit your build configuration file `.circleci/config.yml` to remove the above declaration from the app.yaml at deploy time. The following bash snippet will do that for just `staging` and `production` environments:

```bash
if [[ ${ENVIRONMENT} =~ ^(production|staging)$ ]]; then
  sed -i "s/login: required//" app.yaml
fi
```

### CORS

Another line above worth noting is the following, which enables unrestricted CORS access. 

```yaml
    http_headers:
      Access-Control-Allow-Origin: "*"
```

Most projects won't need CORS at all, and obviously some will need better restrictions, so feel free to edit or delete this.

### Integrating with CI/CD

The first deployment to GAE must be manually triggered, so ensure you have the [`gcloud` CLI installed](https://cloud.google.com/sdk/install) on your machine. 

Run a build locally, then deploy it manually by running the following locally, and answering the interactive questions.

```bash
gcloud app deploy
```

After this [complete the CI setup](./0003-continuous-integration#build).

## Dynamic hosting on AWS
[Dynamic Hosting on AWS]: #dynamic-hosting-on-aws

The main rule about dynamic back ends on AWS or GCP is write your code within a docker container, since it reduces maintenance overhead and increases portability, even if you code specifically for AWS. Since ElasticBeanstalk has very poor support for containers (it uses ECS in a very inflexible way that causes a lot of headaches), the best options are ECS or EKS directly, or for small projects to use Lambdas.

These are all outside the scope of this document since they become very involved and project-specific, but if using containers please consider the Fargate runtime for jobs that need to remain available but get very low traffic (such as non-production environments).

## Dynamic hosting on GCP
[Dynamic Hosting on GCP]: #dynamic-hosting-on-gcp

The main rule about dynamic back ends on AWS or GCP is write your code within a docker container, since it reduces maintenance overhead and increases portability, even if you code specifically for GCP. This rules out AppEngine as a general approach and ideally [GKE](https://cloud.google.com/kubernetes-engine) would be the service of choice, but we've also had some success using Cloud Run for jobs that need to remain available but get very low traffic (such as non-production environments).

## Setting up a new AWS account
[Setting up a new AWS account]: #setting-up-a-new-aws-account

* Log into the Master AWS account as an administrator
* Go to [Organization](https://console.aws.amazon.com/organizations/home) page
* Add Account > Create account (**Use a group email - should be developers+something@**)
* Wait a minute until it's created, then switch to the new account using the OrganizationAccountAccessRole which gives admin privileges on that account
* Go to IAM > Roles > Create a Role
* Trusted entity > Another AWS Account > ID: (the master account), Require MFA [tick]
* Add policy ViewOnlyAccess
* Call it (the same role as this one on the other accounts) so all devs automatically have access to it
* Post the switch role URL on the #perm-tech channel (using the developer access role you just made)

## Setting up a new GCP project
[Setting up a new GCP project]: #setting-up-a-new-gcp-project

First of all [set up your GCP project and enable GAE](https://cloud.google.com/appengine/docs/standard/nodejs/building-app/creating-project) ensuring you add it to the Signal Noise organisation (you may need a tech lead to do this for you, and to enable billing via Signal Noise). Please also make sure it's in the right [organisation folder](#account-folder-structure).

## Accessing resources
[Accessing resources]: #accessing-resources

By default, we use the following access rules:

* Freelancers have no direct resource access
* Permanent coders get view access to all AWS accounts
* Permanent coders also get the same permissions as the CI users, so they ought to be able to write files etc and manually deploy resources if needed
* Permanent coders get admin access to a Sandbox AWS account for experimentation / debugging etc.
* Project Technical Leads (either a Tech Lead or occasionally a Lead Developer) will get admin access to the relevant AWS account, though this should be revoked when needed
* Technical Director has admin access to all AWS accounts via a secondary role (on AWS).

On GCP switching projects is trivial and all projects should be on the same overall account; if you can't see a project you need to ask a Tech Lead to add you.

On AWS in the console, to access any project resources you will need to first login to the master account with your username (generally your email address) and your password + MFA device. Note the master account has user access and audit records but no hosting resources. From there you will need to [Switch Role](https://signin.aws.amazon.com/switchrole) in the top right menu. You'll need to enter the relevant AWS account number (to be found pinned in the #perm-tech Slack channel) as well as the Role name and a name + colour for your reference. Note that this is stored in a cookie so that the last 5 Roles you access are retained in that browser. The standard Role name used for perm coders is the same across all accounts and is also pinned in the same channel.

To access AWS resources via the CLI, in an account you access via Role Switching, follow this simple guide: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-cli.html

# Reference-level explanation
[Reference-level explanation]: #reference-level-explanation

* [Secrets](#secrets)
* [Domains](#domains)
* [Account or folder structure](#account-folder-structure)

## Secrets
[Secrets]: #secrets

**Never commit your secrets to the repo** (except perhaps ones that are only valid on a dev environment, but even then ideally don't commit them). 

On most projects the easiest thing to do is to keep them in an `.env` file and use code in the project to make them available. If you do this ensure that the file does not get committed, though it's best practice to make and version a `.env.sample` file that has all the required keys in it, to make life easier for people. 

You also need to keep all the secrets you manage this way in Dashlane: create a secure note and copy the env file contents in along with any other bits and pieces needed. Name the note with the project name and ideally number, and share it with the Technology group.

It's OK to keep secrets in CircleCI's Environment variables area (you'll need to anyway) or in GCP's or AWS's Secret Manager modules.

## Domains
[Domains]: #domains

Project-specific bought domains should be managed in Route53, in the same AWS account as the project. You can [buy a domain on AWS](https://aws.amazon.com/getting-started/tutorials/get-a-domain/) or [move the DNS to AWS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html) if you already own the domain. If the name is not available in Route53 please use name.com using the company account, and manage the DNS on Route53.

It's bad practice to speculatively buy a domain on behalf of a client.

## Account or folder structure
[Account or folder structure]: #account-folder-structure

On GCP we use folders to group projects for security access reasons, on AWS we use multiple accounts set up as parts of an [AWS Organisation](https://aws.amazon.com/organizations/). This allows us to improve security for specific projects and across client hosting, as well as to make project billing simpler and clearer.

In either case the rationale for where a project should go is the same, and the folder or account should be named the same way. We use the shorthand of account below but on GCP we're referring to Folders under the IAM Managing Resources section:

### Master

This is used for billing, internal team IAM and audit only. No resources should be created in this account.

### Sandbox

This is for experimentation. All developers have admin access to this account, but any resources they create should be terminated ASAP, and may be terminated at any time.

### Internal

For projects where we are the client - internal tooling, "FTP", internal projects etc.

### Legacy

This is to be shut down after all current projects are migrated away.

### Development

All other environments for all other projects should go into the Development account. Deleting any resource in this account should have a low impact.

### Specific Client accounts

Projects should go in a Specific Client AWS account if they are for a client with sensitivities around IP or data, _or_ if we are doing production hosting. All environments for all related projects should go into this account

### Specific Project or workstream accounts

A Specific Project or Stream AWS account is only merited in two cases:

- if we are doing production hosting for a client billing entity that's distinct from the rest of the client organisation. All related environments should go into this account
- If we have a project with a Specific Project Production account - in this case all non-production environments for that project should go here

### Specific Project Production accounts

Projects should get a Specific Project Production AWS account only if they are very sensitive (data or IP) _and_ if we are doing production hosting. Only a single project environment should be in this account, for security reasons.

# Drawbacks
[Drawbacks]: #drawbacks

This guide only really covers static hosting, and doesn't go into all the settings for secure production hosting even then. Some independent knowledge would be required to be confident that something has been set up correctly, but this guide should at least allow anyone with credentials to set up a project's non-production hosting in a way that will cause the fewest headaches.

# Rationale and alternatives
[Rationale and alternatives]: #rationale-and-alternatives

These approaches have been tried and tested across many projects. It's also not really a single approach so much as a framework and guidance for choosing the right setup for your circumstances. The overall idea is to reduce our dependence on a wide variety of hosting providers that may make some processes easier but add to overall admin and management overheads - for example such as using Heroku or Zeit.

# Unresolved questions
[Unresolved questions]: #unresolved-questions

This RFC does not really cover dynamic or complex hosting requirements, nor does it discuss choices such as whether to use AWS services or self-hosted servers on AWS. All of these merit discussion but would complicate this document and are better suited to a dedicated RFC regarding infrastructural choices. 

Security concerns and the tradeoffs regarding e.g. CSP headers with Styled Components are also not discussed, despite touching on the hosting solution in place.

# Future possibilities
[Future possibilities]: #future-possibilities

As AWS and GCP expand their offering this guide will have to change.
