+++
title = "Lab 2: CD Pipeline"
chapter = true
weight = 400
+++

# Deliver your Application in an AWS Continuous Deployment pipeline

Now it's time to setup automatic code deployments of our Unicorn API. Here we will be using a set of AWS Code tools, such as [AWS CodePipeline](https://aws.amazon.com/codepipeline/) and [AWS CodeBuild](https://aws.amazon.com/codebuild/) to automate these deployments. 

The goal of this part is to enable, automatic - but at the same time, tested - deployments of our Code to production. We will be using a nifty tool called `mocha` to test our NodeJS code. 


{{% children showhidden="false" %}}
