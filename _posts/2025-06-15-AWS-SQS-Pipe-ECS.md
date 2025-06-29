---
title: Using AWS SQS and Event Bridge Pipes to send messages to ECS
date: 2025-06-15 00:00:00 +1000
categories: AWS
description: "Learn how to use AWS SQS and EventBridge Pipes to send messages to ECS."
tagline: "Are AWS SQS Pipes reliably to send messages to ECS tasks?"
tags:
  - AWS
  - SQS
  - Event Bridge
  - Pipes
  - ECS
excerpt_separator: <!--more-->
header:
  overlay_image: ./assets/images/Angry-Blue-Tit-2025-Robert-de-Veen.jpg
  caption: "Photo credit: [**Robert de Veen**](https://www.robertdeveen.com)"
  show_overlay_excerpt: false
---

Today I was working on a integration to process message from an AWS Simple Queueing Services (SQS) Queue to a backend services. The specifics of this backend services doesn't really matter for this post, the only thing relevant is that it using the old-school Windows Communication Framework (WCF). With a custom .NET application written in the beloved C# language, we can communicate with the backend. 

<!--more-->

This integration start with a third party putting a message on our queue. Because the amount of message we think this integration would process we would like to make this integration serverless. This means no service running 24-7 that is constantly polling for work to do, but a process that is triggered when a new message has arrived. 

## Integration overview

![Integration using SQS, Event Bridge Pipe and ECS](/assets/images/SQS-Pipe-ECS.drawio.svg)

In this post I will show you how to use AWS SQS and EventBridge Pipes to send messages to ECS. The integration consists of the following components:
- **SQS Queue**: The queue where the third party put the messages. Nothing special about this queue, just a standard SQS queue.
- **EventBridge Pipe**: The pipe that connects the SQS queue to the ECS task.
- **ECS Task**: The task that processes the messages from the SQS queue. This task is running a custom .NET application that communicates with the backend service using WCF.

### EventBridge Pipe

![AWS Event Bridge Pipe, Source is a SQS queue, the target an ECS Cluster Task definition](/assets/images/EventBridge-Pipe-SQS-ECS.png)

Creating an EventBridge Pipe is a [straightforward process](https://docs.aws.amazon.com/eventbridge/latest/userguide/pipes-get-started.html). The pipe will listen to the SQS queue and trigger the ECS task when a new message arrives. When creating the pipe, you need to specify the source (SQS queue) and the target (ECS task). You can also configure the pipe to filter messages or enrich messages based on certain criteria, but for this example, we will keep it simple and process all messages as they arrive.

![AWS Event Bridge Pipe, Source is a SQS queue, the target an ECS Cluster Task definition, no filter or enrichment configured.](/assets/images/EventBridge-Pipe-SQS-Filtering-Enrichment-ECS.png)

### CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template for EventBridge PipeSQS-Pipe-ECS-Integration
Resources:
  MySQSQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: MyQueue

  MyECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: MyECSCluster

  MyECSTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: MyTaskDefinition
      ContainerDefinitions:
        - Name: MyContainer
          Image: my-docker-image
          Memory: 512
          Cpu: 256
          Essential: true

  MyEventBridgePipe:
    Type: AWS::Events::Pipe
    Properties:
      Name: MyQueueToECSPipe
      DesiredState: RUNNING
      Source: !GetAtt MySQSQueue.Arn
      Target: !GetAtt MyECSCluster.Arn
      TargetParameters:
        EcsTaskParameters:
          TaskDefinitionArn: MyECSTaskDefinition.Arn
          TaskCount: 1
          NetworkConfiguration:
            awsvpcConfiguration: {}
          EnableECSManagedTags: true
          EnableExecuteCommand: false
```

This (partial complete) CloudFormation template creates the necessary resources for the integration. It creates an SQS queue, an ECS cluster, an ECS task definition, and an EventBridge Pipe that connects the SQS queue to the ECS task.

This Pipe will automatically trigger the ECS task when a new message arrives in the SQS queue. The ECS task will then start up. It doesn't receive anything from the SQS queue, it will just start up and run the code in the container.

If the Pipe is configured correctly, it will start the ECS task when a new message arrives in the SQS queue. The message is not passed to the ECS task, but the task will start up and run the code in the container. When the container is started, the message is already removed from the SQS queue. This means that the ECS task will not receive the message automatically. 

You can pass the message to the ECS task by using `$.body` in the template. This will pass the message body to the ECS task as an environment variable. You can then read this environment variable in your code and process the message accordingly. 

```yaml  
  MyEventBridgePipe:
    Type: AWS::Events::Pipe
    Properties:
      TargetParameters:
        EcsTaskParameters:
          ... # other parameters
          Overrides:
            ContainerOverrides:
              - Name: MyContainer
                Environment:
                  - Name: MESSAGE_BODY
                    Value: "$.body"
```

It is also possible to pass the message body as a startup argument to the ECS task. In the `ContainerOverrides` section of the `EcsTaskParameters`, you can specify the command to run when the container starts. This allows you to pass the message body as an argument to your application.

```yaml
  MyEventBridgePipe:
    Type: AWS::Events::Pipe
    Properties:
      TargetParameters:
        EcsTaskParameters:
          ... # other parameters
          Overrides:
            ContainerOverrides:
              - Name: !Ref ContainerName
                Command: 
                  - "dotnet"
                  - "/app/Processor.Host.dll"
                  - "--body"
                  - "$.body"
```

### No resilience integration

When the Pipe can not start the ECS task because, for example: incorrect configuration, unknown task definition, or not the right permission to do so, it will not delete the message from the SQS queue. This means that if the ECS task fails to start, the message will remain in the SQS queue and can be processed again later. However, if the Pipe successfully starts the ECS task, it will delete the message from the SQS queue.

> **Warning**: When the Pipe has successfully started the ECS task, it will not wait for the task to complete but it will delete the message from the SQS queue. This means that if the ECS task fails to process the message, the message will be lost. This will make this integration not resilient, a failure in the ECS task will not result in the message being retried.

## Conclusion

This is not a resilient integration, but a simple way to start a process based on a message that is send to an SQS queue using EventBridge Pipes and ECS. The integration is serverless, meaning that there is no need to run a service 24-7 that is constantly polling for work to do. Instead, the ECS task is triggered when a new message arrives in the SQS queue, making it a cost-effective solution for processing messages.
