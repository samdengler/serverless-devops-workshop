+++
title = "Lab 3: AWS X-Ray Integration"
chapter = true
weight = 500
+++

# AWS X-Ray Integration

In this module, you'll use [AWS X-Ray](https://aws.amazon.com/xray/) to analyze and debug the Unicorn API.

The work you will do is divided up into 3 phases:  


{{% children showhidden="false" %}}

But first, a quick intro to AWS X-Ray...



## AWS X-Ray Overview

[AWS X-Ray](https://aws.amazon.com/xray/) helps you analyze and debug production, distributed applications. With X-Ray, you can understand how your application and its underlying services are performing to identify and troubleshoot the root cause of performance issues and errors. X-Ray provides an end-to-end view of requests as they travel through your application, and shows a map of your application's underlying components. You can use X-Ray to analyze both applications in development and in production.  Next, we'll look at how to integrate X-Ray with Lambda. the following:


### AWS X-Ray Integration with AWS Lambda

Using AWS X-Ray to trace requests enables you to gain insights into the performance of serverless applications, allowing you to pinpoint the root cause of issues so that you can address them.

To integrate X-Ray with Lambda, a few changes are required to the Unicorn API from the previous Module.  These changes are already included in the uni-api in this Module (Seed-3-XRay), but we will review them so that you are familiar with the modifications.

