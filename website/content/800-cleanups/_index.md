+++
title = "Clean up"
weight = 800
chapter = true
+++

# Clean up your stack

To avoid unexpected chargers to your account, make sure you clean up your account.

1. Go to [AWS Cloud9](https://eu-west-1.console.aws.amazon.com/cloud9/home?region=eu-west-1#) console and delete all IDEs.
1. Detach the `AWSLambdaFullAccess` policy from each `CodeStarWorker-uni-api-*CloudFormation` role attached in Lab 0.
1. Go to the [AWS CloudFormation console](https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1):
    - Remove all awscodestar-* stacks.
    - Remove all aws-cloud9-* stacks.
    - Remove all stacks used for seeding the repository. They are called like "Seed-0-\*", "Seed-1-\*" etc.
