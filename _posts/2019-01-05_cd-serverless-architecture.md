--- 
title:  "Best Practices for Multi-Environment Serverless Architectures"
image: /images/rocket_cd.png
date:  2019-01-05 15:04:23
tags: [python, AWS, lambda, continuous delivery, bitbucket]
description: Serverless is the new Buzz word in town being promoted by cloud service providers designed to allow organizations to focus on developing applications, and not on managing infrastructure. However, the need for minimizing the risk in the software development cycle for these applications is not such a new concept. In most organizations, code delivered by developers is typically required to flow through multiple environments (test, staging, etc..) to ensure it works as expected before it is deployed to production. That said, Servless Architectures introduce a unique set of challenges that need to be considered when running a multi-environment setup. In this article, I'll go over a few best practices for managing multi-environment serverless architectures.
excerpt_separator: <!--more-->
---
*Serverless* is the new Buzz word in town being promoted by cloud service providers. It's designed to allow organizations to focus on **developing applications**, and not on **managing infrastructure**; however, the need for minimizing the risk in the software development cycle for these applications is not such a new concept. In most organizations, code delivered by developers typically flows through multiple environments (test, staging, etc..) to ensure it works as expected before it is deployed to production. That said, Servless Architectures introduce a unique set of challenges that need to be considered when running a multi-environment setup. In this article, I'll go over a few best practices for managing multi-environment serverless architectures.
<!--more-->

![cd_img](/images/Continuous-Delivery-and-Deployment.jpg)

## Multi-Stack vs Single Stack Approach
In order to minimize the risk introduced by any code integrations, organizations typically setup multiple environments (test, staging, etc..) designed to test the robustness of any delivered code before it is pushed into production. When configuring these environments in a serverless environment however, an important decision must be made as to whether a single stack or multi-stack strategy is most appropriate.

![staging_prod_architecture](/images/staging_prod.png)

The single stack approach shares its API Gateway and Lambda functions across all environments, and uses stages, environment variables and Lambda aliases to differentiate between environments. In contrast, a multi-stack approach uses a completely separate instance of each service for every environment and refrains from utilizing API stages or Lambda aliases to so. The main differences between the two approaches is that the multi-stack approach **minimizes risk**, while the single stack approach minimizes **configuration/management effort**.

![staging_prod_single_architecture](/images/staging_prod_single_stack.png)

In the event in which something goes wrong in a single stack approach, there is a much higher likelihood that your production systems are negatively impacted as well. This is due to this approach's reliance on environment variables and lambda aliases for indirection. In the multi-stack approach, this probability is greatly reduced as a result of the use of separate resources for each environment. 

## Adopt Continuous Delivery
The main idea behind Continuous delivery is to produce **production ready** artifacts from your code base frequently in an **automated fashion**. It  ensures that code can be rapdily and safely deployed to production by delivering every change to a production-like environment and that any business applications and services function as expected through rigorous automated testing. The most important point however, is that all of this must be automated.

Using their front end web application, Cloud providers such as Amazon Web Services make it easy to spin off a new lambda function for testing purposes or to update a function's application code. However, all aspects of your continous delivery strategy should be automated - the only manual step should be the push of the `deploy to production` button. This is vital for two reasons:

1. **Minimizing Risk** - Even with proper resource access controls in place, which is seldom the case, the risk of accidentally causing bad things to happen are higher than you think. You definitely don't want be the reason for a disruption to your service.
2. **Traceability** - When your continuous delivery strategy is an automated, often using an automated pipeline of some sort, the use of  deployments, builds and other components are better documented, making it easier to resolve issues and manage what has been used where.

![bitbucket_deploy](/images/deployments_video_edited.gif)

Using bitbucket pipelines, I usually use the following skeleton pipeline in multi-environment, continous delivery environments. Firstly, it checks and tests every Pull Request. Once the changes are deployed to master, it automatically updates our `STAGING` environment. Finally, our `PROD` environment is updated once the changes in `STAGING` are known to be safe using a manual trigger. If you are using AWS, checkout [this repo](https://bitbucket.org/awslabs/) for more information on the actual autoamted deployment of Lambda functions.

```yml
pipelines:
  default:
    - step:
        name: Run Static Analysis Tools
        script:
          ...
    - step:
        name: Run Automated Tests
        script:
          ...
  branches:
    master:
      - step:
          name: Run Static Analysis Tools
          script:
            ...
      - step:
          name: Run Automated Tests
          script:
            ...
      - step:
          name: Update STAGING
          deployment: staging
          script:
            (deploy to staging)...
      - step:
          name: Update PROD
          deployment: production
          trigger: manual
          script:
            (deploy to production)...
```


## Store App Config In the Environment
An app’s config is everything that is likely to vary between deploys (staging, production, developer environments, etc). This typically includes:

- Resource handles to the database, cache or any other attached resources
- Credentials to external services such as Amazon S3 or Twitter
- Per-deploy values such as the canonical hostname for the deploy

Apps sometimes store config variables as constants in source code which is problematic for several reasons. Firstly, any security sensitive related config data is visible to everyone who has access to the source code, which poses a serious security concern. Additionally, config varies substantially across deploys, whereas code does not. Therefore, developing code that dynamically detects the environment and sets the appropriate config data as required results in code that is uneccessarily complicated and difficult to maintain, especially as the number of environments/deploys increases. On top of this, any reqired changes to an environment config requires pushing a change through the code base and waiting for it to propagate through the software delivery process which is often tedious and slow.  A good test for determining whether an app has decoupled its config from the code is to see if the codebase could be made open source at any moment, without compromising any credentials.

Rather than keeping your config data in source code, consider using evironment variables. In multi-environment systems, environment variables can be leveraged to  generate flexible and infinite variations (test, staging, prod, etc..) of your environment template. More importantly, this can done without changing any code, which makes the code base much cleaner. They are also OS and language agnostic. In this method, applications are developed to simply refer to environment variables (possibly encrypted), which are dynamically set by a build system an a per-deploy basis. 

## Use A Serverless Framework

A serverless framework is a command line interface for building and deploying serverless applications. The main service provided by such frameworks is the ability to define entire Serverless applications using a simple yaml configuration file. Applications can be defined using using a front end console, but the amount of work to manage your application increases proportionally to the number of environments being managed. Serverless frameworks on the other hand offer a systematic and efficient approach to managing environments.


Cloud-agnostic
Allowing organizations to prevent data lock-in on a single vendor. Use the Serverless Framework CLI to build and deploy your application to any cloud provider with a consistent experience. The Framework automatically configures cloud vendor settings for you, based on the language you use and the provider you deploy to.


Componentized
So that developers don’t have to keep rebuilding the wheel. They can build once, use, and reuse Serverless Components to do things like create & manage DynamoDB tables or enable authentication with Auth0. Components are open source, and there is lots of pre-built functionality developers can tap into right away.


Code for your infrastructure
Because Serverless Applications require automation. If you're tying together multiple managed services and functions, you cannot rely on a checklist of manual steps. You should be able to recreate your entire application with a command.

