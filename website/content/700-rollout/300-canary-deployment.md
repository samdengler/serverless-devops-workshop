+++
title = "Canary Deployment"
weight = 300
+++

## Traffic shifting using AWS CodeDeploy

For production deployments, you want more controlled traffic shifting from an old version to a new version which monitors alarms and triggers a rollback if necessary.

Alarms are bases on metrics. For rollbacks you should take any metric into account that defines the quality of your product. This can be a technical metric like the error rate of AWS Lambda function calls. Even better is a business metric. That can be captured only within your business logic, such as usage, sold items, etc. To demonstrate the capturing of logs, we will log the HTTP status in our application and expose an Amazon CloudWatch metric for it.

1. Change the `list.js` code to always log the HTTP status code and the function version. 

    {{< highlight js "hl_lines=9-16" >}}
'use strict';

const AWS = require('aws-sdk');

exports.lambda_handler = (event, context, callback) => {
  console.log('Received event:', JSON.stringify(event, null, 2));

  var message = "Hello World!";
  var res = {
    statusCode: 200,
    body: JSON.stringify({
      Output: message
    })
  };
  console.log(`{"log": "response", "function": "${context.functionName}:${context.functionVersion}", "statusCode": ${res.statusCode}}`);
  callback(null, res);
};
{{< /highlight >}}

2. Expose the information in the AWS Lambda log group as an Amazon CloudWatch Alarm. You need to define a *filter metric* based on your logs and define an alarm that goes to the *Alarm* on errors. Append the highlighted lines to the end of `template.yml`. In YAML indentation is important. Add the code **just a single** indentation level deeper than the parent element `Resources:`.

    {{< highlight yml "hl_lines=3-34" >}}
Resources:
    [...]
  # If you've never run your AWS Lambda function before, you need to create the log group
  # ListFunctionLogGroup:
  #   Type: AWS::Logs::LogGroup
  #   Properties:
  #     LogGroupName: !Sub '/aws/lambda/${ListFunction}'

  LambdaDeploymentErrorMetric:
    Type: AWS::Logs::MetricFilter
    Properties:
      FilterPattern: !Sub '{ $.log = "response" &&  $.function = "${ListFunction}:${ListFunction.Version.Version}" && $.statusCode > 400 }'
      LogGroupName: !Sub '/aws/lambda/${ListFunction}'
      MetricTransformations:
      - MetricValue: "1"
        MetricNamespace: "ListFunction"
        MetricName: !Sub "Errors-Version-${ListFunction.Version.Version}"

  LambdaDeploymentErrorGreaterThanZeroAlarm:
    Type: "AWS::CloudWatch::Alarm"
    Properties:
      AlarmDescription: !Sub "Errors-${ListFunction}:${ListFunction.Version.Version} > 0"
      ComparisonOperator: GreaterThanThreshold
      Threshold: 0
      EvaluationPeriods: 1
      MetricName: !Sub "Errors-Version-${ListFunction.Version.Version}"
      Namespace: ListFunction
      Period: 60
      Statistic: Sum
      TreatMissingData: notBreaching
{{< /highlight >}}

    You see that the definition of the metric includes the version. Thus there will be a separate metric for each deployment.

3. Finally, you define a deployment that takes this alarm into account. AWS CodeDeploy is a service which does the traffic shifting for you. It uses AWS Lambda alias' ability to route a percentage of traffic to two different AWS Lambda versions. To use this feature, set the ``DeploymentPreference`` property of ``AWS::Serverless::Function`` resource. And add the alarm as highlighted here:

    {{< highlight yml "hl_lines=6-18" >}}
Resources:
  ListFunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      AutoPublishAlias: live
      DeploymentPreference:
        # Traffic is shifted in two increments. You can
        # choose from predefined canary options. The
        # options specify the percentage of traffic
        # that's shifted to your updated Lambda function
        # version in the first increment, and the
        # interval, in minutes, before the remaining
        # traffic is shifted in the second increment.
        Role: !Ref CodeDeployRole
        Type: Canary10Percent5Minutes
        Alarms:
        # A list of alarms that you want to monitor
        - !Ref LambdaDeploymentErrorGreaterThanZeroAlarm
    {{< /highlight >}}


    When you update your function code and deploy the SAM template using
    AWS CloudFormation, the following will happen:

    - AWS CloudFormation publishes a new AWS Lambda version from the new code.
    - AWS CloudFormation adds a metric that captures the errors in the Amazon CloudWatch log group for the latest AWS Lambda function version.
    - The Alarm points to the new metric. As no data exists for the metric the Alarm will get the *OK* state (see the `TreatMissingData` property above).
    - Since a deployment preference is set, AWS CodeDeploy takes the job of actually shifting traffic from the old version to the new version.
    - ``Role: !Ref CodeDeployRole`` determines the ARN of the role that AWS CodeDeploy assumes during AWS Lambda deployment. This role has been created by AWS CodeStar on project creation. AWS CodePipeline passes the `CodeDeployRole` parameter in the AWS CloudFormation action.
    - ``Type: Canary10Percent5Minutes`` instructs AWS CodeDeploy to start with 10% traffic on new version and after that shift the remaining traffic.
    - During traffic shifting, if any assigned Amazon CloudWatch Alarms go to *Alarm* state, AWS CodeDeploy will immediately flip the alias back to old version and report a failure to AWS CloudFormation which will rollback the remaining part of deployment.
    - If everything went well, the alias will be pointing to the new AWS Lambda version.

4. Please add, commit, and push all the changes:

    ```bash
    git add .
    git commit -m "deploy canary style"
    git push
    ```

    This will run a new pipeline, deploying your code. Test it!

## Test a Rollback

1. Open a new AWS Cloud9 terminal and run a bash script that periodically pings your applications endpoint and outputs the HTTP response code. Replace `YOUR_ENDPOINT` with your individual endpoint in the AWS CodeStar console:

    {{< highlight bash >}}
while sleep 1; do
    code=$(curl -s -o /dev/null -w "%{http_code}" \
    https://YOUR_ENDPOINT.execute-api.eu-west-1.amazonaws.com/Prod/unicorns)
	if [ "$code" -ge 400 ]; then
		printf '\e[41m'
	fi
	echo -e "$code\e[0m"
done{{< /highlight >}}

    This should output the HTTP code **200** every second. Keep this script running.

2. Change your application code in `list.js` from code **200** to **503** so it always responds with an HTTP error.

3. Try to get the new faulty implementation into production. Add, commit, and push the change to run the new pipeline:

    ```bash
    git add list.js
    git commit -m "faulty version"
    git push
    ```

4. Open the [AWS CodePipeline console](https://eu-west-1.console.aws.amazon.com/codesuite/codepipeline/pipelines?region=eu-west-1). When the **ExecuteChangeSet** Action for your last push is running, click on the detail to see the **events** of the AWS CloudFormation stack. When the AWS CodeDeploy deployment is ongoing you will see it here. Continue to watch the events until the deployment is rolled back.

5. Now undo Step 2 above and run the pipeline again with your changes.

## Traffic Shifting Configurations

In the above example ``Canary10Percent5Minutes`` is one of several preselected traffic shifting configurations 
available in CodeDeploy. You can pick the configuration that best suits your application. See [docs](https://github.com/awslabs/serverless-application-model/blob/master/docs/safe_lambda_deployments.rst#traffic-shifting-configurations) for the complete list:

- Canary10Percent30Minutes
- Canary10Percent5Minutes
- Canary10Percent10Minutes
- Canary10Percent15Minutes
- AllAtOnce
- Linear10PercentEvery10Minutes
- Linear10PercentEvery1Minute
- Linear10PercentEvery2Minutes
- Linear10PercentEvery3Minutes

They work as follows:

- **LinearXPercentYMinutes**: Traffic to new version will linearly increase in steps of X percentage every Y minutes.

    Example: ``Linear10PercentEvery10Minutes`` will add 10 percentage of traffic every 10 minute to complete in 100 minutes.

- **CanaryXPercentYMinutes**: X percent of traffic will be routed to new version for Y minutes. After Y minutes, 100 percent of traffic will be sent to new version. Some people call this as Blue/Green deployment.

    Example: ``Canary10Percent15Minutes`` will send 10 percent traffic to new version and 15 minutes later complete deployment by sending all traffic to new version.

- **AllAtOnce**: This is an instant shifting of 100% of traffic to new version. This is useful if you want to run
  run pre/post hooks but don't want a gradual deployment. If you have a pipeline, you can set Beta/Gamma stages to 
  deploy instantly because the speed of deployments matter more than safety here.

## Bonus 1: Add further Amazon CloudWatch Alarms

Add a further alarm that prevents new code to be introduced to production that breaks the SLA of 1 second for response times.

Create an Amazon CloudWatch Alarm that takes the duration of the AWS Lambda function into account. Try adding an alarm in the AWS Management Console console first for the duration metric of your AWS Lambda function. Then add it to the AWS CloudFormation template.

To test a slow function, add these two lines to your `list.js` code:

{{< highlight js "hl_lines=6-7" >}}
'use strict';

const AWS = require('aws-sdk');

exports.lambda_handler = (event, context, callback) => {
  var startDate = new Date();
  while ((new Date()) - startDate <= 1500) {}
{{< /highlight>}}

<details>
<summary><strong>⬇️ Solution Hints (click for details)</strong></summary><p>

Add this Alarm and now refer to it in the deployment preference:

{{< highlight yaml "hl_lines=9 13-32" >}}
Resources:
  ListFunction:
    ...
    Properties:
      ...
      DeploymentPreference:
        Alarms:
        - !Ref LambdaDeploymentErrorGreaterThanZeroAlarm
        - !Ref LambdaDeploymentSlaBreachAlarm

  ...

  LambdaDeploymentSlaBreachAlarm:
    Type: "AWS::CloudWatch::Alarm"
    Properties:
      AlarmDescription: !Sub "${ListFunction}:${ListFunction.Version.Version} duration > 1s"
      ComparisonOperator: GreaterThanThreshold
      Threshold: 1000
      EvaluationPeriods: 1
      MetricName: Duration
      Dimensions:
      - Name: FunctionName
        Value: !Ref ListFunction
      - Name: ExecutedVersion
        Value: !GetAtt ListFunction.Version.Version
      - Name: Resource
        Value: !Sub "${ListFunction}:live"
      Namespace: AWS/Lambda
      Period: 60
      Statistic: Maximum
      TreatMissingData: notBreaching
{{< /highlight>}}

</p></details>

## Bonus 2: Verification AWS Lambda functions with Hooks

Sometimes it might be a good idea to verify that our deployment is working as expected before shifting the traffic. This is what hooks are for in the `DeploymentPreference` section:

```yaml
Hooks:
    # Validation Lambda functions that are run before & after traffic shifting
    PreTraffic: !Ref PreTrafficLambdaFunction
    PostTraffic: !Ref PostTrafficLambdaFunction
```

Before traffic shifting starts, AWS CodeDeploy will invoke the PreTraffic Hook AWS Lambda function. This function must call back to AWS CodeDeploy with an explicit status of Success or Failure, via the [`PutLifecycleEventHookExecutionStatus`](https://docs.aws.amazon.com/codedeploy/latest/APIReference/API_PutLifecycleEventHookExecutionStatus.html) API. On Failure, AWS CodeDeploy will abort and report a failure back to AWS CloudFormation. On Success, AWS CodeDeploy will proceed with the specified traffic shifting.

If you want to implement this feature, you can create a Lambda function based on [this one](https://github.com/awslabs/serverless-application-model/blob/master/examples/2016-10-31/lambda_safe_deployments/src/preTrafficHook.js). For example, create Lambda function that executes the latest version of your deployment to find out if it works as intended (kind of an acceptance test).
