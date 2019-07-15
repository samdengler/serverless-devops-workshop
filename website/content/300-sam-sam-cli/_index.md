+++
title = "Lab 1: Run and test locally"
chapter = true
weight = 300
+++

# Run and test locally

You have noticed earlier, that it takes some time till your code changes run through the pipeline after you have pushed them to the origin. That's normal and expected, however it is not in the interest of developers to wait to see the outcome of their changes on some remote enviroment, but they rather want to execute them locally.

In this part of the workshop, we will have a look at the [AWS Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model) that we have seen before, but not explained so far. We will see how we can use it to define a serverless application with its core components.  We will also use [AWS SAM CLI](http://docs.aws.amazon.com/lambda/latest/dg/test-sam-local.html) to locally develop and rapidly test an API, in order to fulfil the need to not always wait for a remote deployment of the pipeline after each code change.

{{% children showhidden="false" %}}
