# Week 0 — Billing and Architecture

Draw the application architecture on Lucid charts

## Install AWS CLI in the gitpod environment

The Gitpod environment is a Linux environment.

To install AWS CLI on Gitpod:
```
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```

Then install the aws user credentials by setting and persist the environment variables on Gitpod

```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=""
```

At this point, we might loose the above configuration when we close and relaunch our gitpod environment.

To make sure that every time, we start our Gitpod environment, it installs the environment variables we should do the following.

Update the `.gitpod.yml` file with the following content:

```
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

## Create budget with AWS CLI

[Here's the link to the official AWS documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/budgets/create-budget.html)

Get our AWS account ID

`aws sts get-caller-identity --query Account --output text`

As per the documentation example, we'll need two json files ` budget.json` and `notifications-with-subscribers.json` which will be stored in aws/json folder.

- Update the json files
- Create the budget with this aws cli command

```
aws budgets create-budget \
    --account-id AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/notifications-with-subscribers.json
```

## Enable Billing

To receive alerts, we need to turn on Billing Alerts.

- We go the [Billing page](https://us-east-1.console.aws.amazon.com/billing/home) in the root account
- Under `Billing preferences` we choose `Receive Billing Alerts`
- Save preferences

## Create a billing alarm

### Create SNS Topic

- We need an SNS topic before we create the alarm
- The SNS topic will deliver an alert when we get overbilled
- [aws sns create-topic](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sns/create-topic.html)

We run the following command to create the topic

`aws sns create-topic --name billing-alarm`

which will return a TopicARN

We then create a subscription supply to the TopicARN and our email

```
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint my-email@example.com
```

We check our email and confirm the subscription

### Create Alarm

- [aws cloudwatch put-metric-alarm](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudwatch/put-metric-alarm.html?highlight=cloudwatch%20put%20metric%20alarm)
- [Create an alarm via AWS CLI](https://repost.aws/knowledge-center/cloudwatch-estimatedcharges-alarm)
- We update the configuration json script with the TopicARN we generated earlier

Then run the following command:

`aws cloudwatch put-metric-alarm --cli-input-json file://alarm_config.json`

We are storing the abo
