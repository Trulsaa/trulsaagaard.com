---
layout: post
title: "Convert your feedback loop into a continuous discussion with review environments"
categories: DevOps
---

Wouldn't it be fantastic if everyone on the team, with all of their diverse skills and knowledge on different parts of an issue, could work on issues at the same time? All of the skills are needed to get tasks from idea to production. This means that splitting the work over time only makes it take longer. Review environments makes it possible for everyone on the team to work on the task without impeeding each other's work, speeding up throughput significantly. It this sounds interesting, I try to highlight the benefits and how to in this article.

## Precisely what is a review environment?

Review environments, also known as ephemeral environments or preview environments. A review environment is a full deployment of an application stack, representing the latest changes made to a branch connected to a Merge Request (MR). It lives alongside the MR and provides a way to see the changes made in that MR from the perspective of the end user. The deployment of the review environments is done using the same automation that is used when deploying to production.

![](/media/Review%20app%20branch%20strategy.excalidraw.png)

## Prerequisites

Creating review environments means that N number of unique environments are going to be created and updated hundreds of times a day. It goes without saying that build, test and deployment need to be robust. This is a list of requirements that need to be in place when setting up review environments for a web app.

-  Fully automated build and deployment
-  Baseline database with test data
-  Automated database migrations
-  Fast deployments
-  Idempotent deployment
-  Automated destruction of environments
-  DNS zone with a * wild card or dynamic creation of DNS subzones
-  Automated creation of site certificates to enable HTTPS (letsencrypt)
-  Environment configuration properties need to be configurable during deployment

## Creating a review environment

A review environment is created by deploying an environment with a name unique to the MR. This environment can be reached on a unique URL that is then made readily available as a button in the MR. The deployment is triggered in the pipeline that runs when pushing code to the branch connected to the MR. The unique name is provided in the pipeline as an environment variable called CI_ENVIRONMENT_SLUG. This value will be the same for the lifetime of the MR. And can therefore be used as input to the Idempotent deployment script, which should create a new review environment if it does not exist and upgrade the environment if it exists. The following code is an example of a pipeline job in Gitlab that deploys a review environment on every push to an MR that is set to merge with the master branch.

```yml
deploy_review:
  image: $CI_REGISTRY_IMAGE/az-helm-kubelogin
  stage: review
  script:
    - ./run ci_deploy review $CI_ENVIRONMENT_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_ENVIRONMENT_SLUG.review.example.com
    on_stop: stop_review
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: never
    - when: on_success
```

## Stopping a review environment

It is good practice to stop a review environment when it is no longer needed. Because these environments are created for each MR, the number of environments can accumulate fast and consume significant resources.

Stopping a review environment is done by calling a script that takes the CI_ENVIRONMENT_SLUG environment variable available in the pipeline and uninstalls the review environment unique to that MR. The following code shows a job that is triggered when the MR is merged.

```yml
stop_review:
  image: $CI_REGISTRY_IMAGE/az-helm-kubelogin
  stage: review
  script:
    - ./run ci_un_deploy review $CI_ENVIRONMENT_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: never
    - when: manual
```

## The benefit

The main benefit of review environments is team member happiness. I think this comes from a team dynamic where all the members of the team can collaborate on the same future at the same time. Greatly reducing cognitive load by reducing context switching. Resulting in faster development. And a team that gets a sense of accomplishment together.

Review environments enable **parallelizing development and feedback**. We can fix bugs while the issue is fresh in our minds. When the we think a solution has been reached we push the code and create an MR with a review environment. Then while the we write some tests, a tester checks if the issue is solved in the review environment while someone reviews the code. The feedback is then given while the developer is still working on the code. This has in several instances resulted in feedback that has been, discussed and fixed within minutes of the last code push.

Getting several pairs of eyes to **test** new code is essential to get another perspective and get it right before merging. Normally this is done when the code has been merged into the test environment. This creates a time gap between when the developer writes the code and when it is tested and new tasks are created to fix the newly introduced bugs. More than 2 weeks can go by before the developer gets feedback on whether the future works or not. At this time, all of the developer's cognitive capacity is occupied with a new task. Using review environments, this feedback can be given while the code is being code reviewed. Testers, functional architects, security testers, product owners and even super users can give feedback and have an opportunity to discuss if this code solves the issue. This gives the added benefit of transparency and stakeholder involvement. But the most important benefit is that the developer can fix the feedback while having the code fresh in mind.

Code reviewers can **contextualize the changes in the code**. Before we had review environments, reviewers had to stop what they were doing, clone the branch they needed to review and run this branch locally to look at the result of the changes. This normally meant shutting down a running backend and frontend development server. Sometimes the local database had to be flushed and re-seeded. Before starting it all up again on the branch that needed to be reviewed. This could take around 20 minutes depending on what changes had been done in the branch. There are too many parts of the process where irrelevant bugs can be introduced. And even if everything went well you still have to do the whole process again to get your work back up and running. The result more often than not is that this was not done at all and the code was approved without seeing the result. With a review environment, this process is as simple as clicking a button.

### Bonus benefit

Doing **demos** is important to give stakeholders insights into the state and progress of the application development. But it might also become a blocker when we are unable to merge into the dev environment because it is used to show a demo. Of course, you could restrict demos to using the test environment. But it is always cooler to show off the latest and greatest futures than an old and boring test environment. This whole problem is completely removed when you can spin up review environments based on any commit in any branch whenever you want. Review environments are isolated and do not block any other development. Demos in nature are also focused on showing the happy path. So it is possible to combine several future branches into one review environment that is a perfect glimpse into the near future of the application. Just make sure to stay on the happy path during the demo

## Summary

Using review environments has enabled the team to accelerate development. The whole team is able to contribute with their expertise to every change to the source code. This creates an environment that enables more correct feedback at the right time. Being part of the review process is engaging and promotes a sense of responsibility. The ability to engage often facilitates more consistent involvement with a project which leads to a better understanding of changes for all involved.

## Sources

- [https://docs.gitlab.com/ee/ci/review_apps/](https://docs.gitlab.com/ee/ci/review_apps/)
- [https://docs.gitlab.com/ee/ci/environments/index.html](https://docs.gitlab.com/ee/ci/environments/index.html)
