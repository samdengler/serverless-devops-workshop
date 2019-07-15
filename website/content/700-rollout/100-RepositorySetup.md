+++
title = "Repository Setup"
weight = 100
+++

### Seed the `uni-api` CodeCommit Git repository

Each module has corresponding source code used to seed the AWS CodeCommit Git repository to support the workshop. The following seed is the same as for Lab 1. To seed the AWS CodeCommit Git repository, click on the **Launch Stack** button for your region below:

Region| Launch
------|-----
EU (Ireland) | [![Launch Module 1 in eu-west-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/create/review?stackName=Seed-4-Canary&templateURL=https://s3.amazonaws.com/fsd-aws-wildrydes-eu-west-1/codecommit-template.yml&param_sourceUrl=https://demo-dub.s3-eu-west-1.amazonaws.com/cdonaws/uni-api-1-v5.zip&param_targetRepositoryName=uni-api&param_targetRepositoryRegion=eu-west-1)

1. The CloudFormation template has been prepopulated with the necessary fields for this module. No changes are necessary.

1. Select the **I acknowledge that AWS CloudFormation might create IAM resources.** checkbox to grant CloudFormation permission to create IAM resources on your behalf

    ![Seed Repository CloudFormation Stack Review](images/seed-repository-1.png)

1. Click the **Create stack** button in the lower right corner of the browser window to create the AWS CloudFormation stack and seed the AWS CodeCommit repository.

1. There will be a short delay as the Git repository seeded with the new source code. Upon successful completion, the AWS CloudFormation console will show the status ``CREATE_COMPLETE``.

    ![CloudFormation Stack Creation Complete](images/seed-repository-2.png)

### Fetch CodeCommit Git Repository

Now that the CodeCommit Git repository has been seeded with new source code, you will need to fetch the changes locally so that you may modify the code.  Typically, this is accomplished using the `git pull` command, however for the workshop we have replaced the repository with a new history and different Git commands will be used.

Go back to the Cloud9 tab in your browser and run the following commands in a terminal tab in your local `uni-api` Git repository:

```bash
git fetch --all
git reset --hard origin/master
git clean -fd
```

![Fetch updates in Cloud9](images/fetch-in-cloud9.png)

After the Git reset to the Seed, the commit message should reflect the module/ lab name.

It may be required to refresh the local file tree in your Cloud9 IDE to have it reflect the updated code repository. If you have any file open in an editor tab that has disappeared due to the repository update, you will be prompted if Cloud9 should close the tab or not (you can close the tab).
![Cloud9 refresh file tree](images/cloud9.refresh-file-tree.png)
