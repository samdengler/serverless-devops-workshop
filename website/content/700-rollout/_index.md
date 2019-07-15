+++
title = "Lab 4: Canary Deployment"
chapter = true
weight = 700
+++

# Rolling out your changes to production

Pushing to production can be nerve-wracking even if you have 100% unit test coverage and a state-of-art full CD system. It is a good practice to expose your new code to a small percentage of production traffic, run tests, watch for alarms, and dial up traffic as you gain more confidence. The goal is to minimize production impact as much as possible.

To enable traffic shifting deployments for AWS Lambda functions, we will use AWS Lambda *aliases*, which can balance incoming traffic between two different versions of your function, based on preassigned weights. Before deployment, the alias sends 100% of invokes to the version used in production. During deployment, we will upload the code to AWS Lambda, publish a new version, send a small percentage of traffic to the new version, monitor, and validate before shifting 100% of traffic to the new version. You can do this manually by calling AWS Lambda APIs or let AWS CodeDeploy automate it for you. AWS CodeDeploy will shift traffic, monitor alarms, run validation logic, and even trigger an automatic rollback if something goes wrong.

{{% children showhidden="false" %}}