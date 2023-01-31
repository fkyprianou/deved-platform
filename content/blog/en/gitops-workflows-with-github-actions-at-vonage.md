---
title: GitOps Workflows With GitHub Actions at Vonage
description: This article explains how we designed and implemented CI/CD
  workflows in GitHub Actions using a GitOps approach.
thumbnail: /content/blog/gitops-workflows-with-github-actions-at-vonage/gitops-workflows_github-actions.png
author: mark-tetlow
published: true
published_at: 2023-01-31T09:51:17.850Z
updated_at: 2023-01-31T09:51:19.286Z
category: inspiration
tags:
  - gitops
  - github-actions
  - devsecops
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
We recently redesigned a key build and deployment pipeline supporting our Vonage Contact Center platform. We took the opportunity to move this pipeline to a Security-compliant GitOps design. In this blog post, I will explain what is meant by GitOps, and how our pipeline was redesigned to benefit from this concept.

## Gitops Concepts: What is GitOps anyway?

Broadly speaking, GitOps is a collaboration of the best practices from DevOps and Security. The main difference between DevOps and GitOps is the addition of IaC (Infrastructure as Code) which is stored alongside project code in SCM (Source Code Management), usually Git.

GitOps is a framework rather than an individual software package or tool. In my view, this is one of its strengths. With GitOps you have the freedom and flexibility to tailor the implementation to best fit your organization and enhance your processes.

GitOps is comprised of three distinct concepts:

### 1. Infrastructure as Code (IaC)

IaC tools allow infrastructure to be designed and configured once and deployed repeatably, with state management features that prevent live resources from drifting from their original configuration. Once defined, this configuration data can be kept under source control, enabling a single source of truth for the infrastructure (as well as the associated permissions and security policies) to be maintained.

Additionally, with a well-designed repository structure, the Security concepts of Least Privilege and Separation of Duties are easily achieved for this data.

### 2. Pull Requests (PRs)

All code changes should be controlled by PRs. This protection applies to the application code as well as to the IaC code describing the supporting infrastructure

Developers should have permission to merge code via PRs into the application repository, but they should not have direct permissions to make updates to the GitOps pipeline or infrastructure repositories.

### 3. Continuous Integration / Continuous Delivery (CI/CD)

GitOps automates the CI/CD processes through continuous integration and delivery. CI is triggered by a merge to the main branch. Deployment is triggered either manually or automatically following the successful completion of quality checks.

Security testing can be integrated into the SDLC (Software Development Life Cycle) much more easily with GitOps.  Additionally, since the iteration cycle is much faster, this means more ease and less friction in the adoption of Security. This encourages a focus on prevention rather than detection, commonly referred to as 'shifting left'.

## How we implemented GitOps

### Context

Our goal was to build a GitOps CI/CD pipeline for the delivery of Micro-FrontEnd components into the Vonage Contact Centre cloud-hosted environments. The pipeline is a replacement for an earlier version, and as such a design was required that would significantly reduce development friction and deployment times, improve security and as a consequence, improve DevEx (Developer Experience).

### Research and Requirements

Since this was our first practical exposure to GitOps, we carried out some initial research into the GitOps framework and performed a spike to trial a few different approaches. Given the nature of our requirements and the inevitable constraints at work, we chose to adopt the event-driven model of GitHub Actions with a combination of cloud-native CLI tools, to allow us to control updates into the cloud-hosted infrastructure (in this case CDNs, serverless code functions, and a NoSQL DB). In our GitOps architechure we only need one event. Application deployment is triggered by a GitHub `repository_disptch` event originating from the Application repository and targetted to the GitOps repositoy, where it triggers the deployment automatically. 

Once we were confident GitOps was a viable solution, we refined our requirements:
- Improve security posture by adhering to the Least Privilege and Separation of Duties principles
- Provide timely deployment feedback to developers
- Reduce deployment friction and thereby improve deployment times and DevEx
- Support deployment metrics
- Use maintainable GitOps CI/CD pipelines
- Horizontal scalability (seamless integration of new applications into the GitOps pipeline)
- Leverage and enhance internal tooling where it makes sense

### GitOps Architecture

**Infrastructure repositories**

Terraform was our tool of choice for the rollout and management of the cloud infrastructure on which the MFE components would be hosted. This IaC approach is useful for controlling infrastructure that doesn't change often.

Each of the resources required to host the MFE components was captured in Terraform modules, along with the required supporting security and permissions objects. A pre-existing Terraform CI/CD pipeline is used to maintain these components.

**Application repositories**

Developers (and the MFE application repository itself) have narrow permissions to development tools and systems. Therefore a CI workflow defined using Github Actions takes care of these responsibilities, providing classic build and quality features in a templated, automated process. Build artifacts produced by this process are pushed to a package repository (jFrog Artifactory).

![Application CI Workflow](/content/blog/gitops-workflows-with-github-actions-at-vonage/app-ci-workflow.png "Application CI Workflow")

A CD workflow in the same repository provides the developer with the ability to deploy the application. In concrete terms, this is a GitHub `repository_dispatch` event sent to the target GitOps repository, followed by a wait-on-completion step.

Example GitHub Action triggering application deploy:

```yaml
- name: Deploy MFE ${{ env.MFE_NAME }} version ${{ env.MFE_VERSION }} to ${{ env.MFE_ENVIRONMENT }}
  uses: peter-evans/repository-dispatch@11ba7d3f32dc7cc919d1c43f1fec1c05260c26b5 # v2.0.0
  with:
    token: ${{ secrets.GITOPS_GITHUB_TOKEN }}
    repository: ${{ env.GITOPS_REPOSITORY }}
    event-type: deploy-mfe-trigger
    client-payload: '{
                        "type": "deploy-mfe-trigger",
                        "timestamp": "${{ env.DATEANDTIME }}",
                        "user": "${{ github.actor }}",
                        "mfe": "${{ env.MFE_NAME }}",
                        "version": "${{ env.MFE_VERSION }}",
                        "environment": "${{ env.ENVIRONMENT }}",
                        "cache_invalidation_keys": "${{ env.CACHE_INVALIDATION_KEYS }}",
                        "repository": "${{ github.repository }}",
                        "ref": "${{ github.ref }}",
                        "sha": "${{ github.sha }}"
                    }'
```

![Application CD Workflow](/content/blog/gitops-workflows-with-github-actions-at-vonage/app-cd-gitops-workflow.png "Application CD GitOps Workflow")

**GitOps IaC repository**

This repository forms the foundation of our GitOps framework. It has the permissions required to deploy to a given cloud provider. Note that it is separated from the MFE application repositories, thereby adhering to the Security principles of Least Privilege and Separation of Duties (as well as shifting Security left).

Deployment events are received from the application CD workflow and automatically processed. In essence, there are three distinct steps in the GitOps process:

1. Process GitHub `repository_dispatch` events
    1. accept/reject
    1. update IaC config based on event metadata
    1. raise PR
1. Merge PR automatically
    1. perform quality checks
    1. merge PR if quality checks pass
1. Deploy application
    1. copy build artifact into Cloud storage
    1. update DB Table
    1. invalidate CDN cache

![GitOps Ops Workflow](/content/blog/gitops-workflows-with-github-actions-at-vonage/gitops-ops-workflow.png "GitOps Operations Deployment Workflow")

**Cloud Infrastructure deployment tool**

We created a CLI tool that interfaces with our cloud provider and enables seamless deployments of applications.

Abstracting the deployment logic into this new CLI project allowed us to manage the fine-grained details of the deploy logic whilst ensuring new features are tested before being released to the production pipeline.

**Micro-FrontEnd project framework CLI**

This is an in-house tool that provides a CI/CD framework for developing in multiple languages. We use this tool to shortcut the "cruft" part of creating and updating our software projects - it provides such things as skeleton component code, common configuration options, and CI/CD pipeline templates. It supports a variety of language runtimes and component types.

We decided early on to enhance the tool to support the new GitOps framework for MFEs, to enhance the developer experience when creating new MFEs, and also to aid in migrating existing MFE components to the new CI/CD pipeline. Our changes to this tool mean that it will automatically create the required CI/CD GitOps Workflows for Micro-FrontEnd projects.

To adopt the GitOps pipeline in a new or already existing project, you would render the project with the CLI tool.

```bash
$ vng project render
  created .github/workflows/micro-frontend-ci.yml
  created .github/workflows/micro-frontend-cd.yml
```

## Where would we go next?

This project has given us a great chance to get to grips with the GitOps concepts and has allowed an insight into the areas where we could improve further. Here are some of the things we could take on next:

* Work closely with Vonage's Security teams to define Security Compliance requirements as configuration (YAML, Terraform, ...). When this is part of the SDLC (and is part of the continual cycles of review/refinement) then you are certain to meet compliance and have the ability to provide proof of compliance.
* Consider using GitHub Actions self-hosted runners, to give us more control over scaling, configuration, and costs. This step needs to be considered carefully as it introduces a trade-off between the security of the runners, cost, and maintainability.
* Create/adopt a GitOps GitHub App similar to the operations available for K8s.

## Final Thoughts

I hope this blog was informative and demonstrated some of the benefits of adopting GitOps into your development pipeline.

Make sure to follow us on [Twitter](https://twitter.com/VonageDev) and join our [Slack channel](https://developer.vonage.com/community/slack) for more information.

Thanks for reading!