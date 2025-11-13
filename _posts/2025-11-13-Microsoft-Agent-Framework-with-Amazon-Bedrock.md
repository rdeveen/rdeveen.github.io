---
title: Using Amazon Bedrock models with Microsoft Agent Framework
date: 2025-11-13 00:00:00 +1000
categories: AWS
description: 
tagline: 
tags:
  - AWS
  - Amazon Bedrock
  - Microsoft Agent Framework
  - AI
  - LLM
excerpt_separator: <!--more-->
# header:
#   overlay_image: ./assets/images/Angry-Blue-Tit-2025-Robert-de-Veen.jpg
#   caption: "Photo credit: [**Robert de Veen**](https://www.robertdeveen.com)"
#   show_overlay_excerpt: false
---

With the [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview) you can create AI Agents on my favorite programming language .NET but also in Python. The SDK provide various types of agents with, off-course, the out-of-the-box support for Azure AI Foundry. In the successor [Symantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/) there was a [specific agent](https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/agent-types/bedrock-agent?pivots=programming-language-csharp) type for Amazon Bedrock’s Agent service. But in the new Microsoft Agent Framework, this specific agent type is not yet available.

<!--more-->

The Microsoft Agent Framework support creating agents thats uses the generic [OpenAI Chat Completion](https://platform.openai.com/docs/api-reference/chat/create) services. This means that you can use the Microsoft Agent Framework to create agents that uses Amazon Bedrock models by using the [OpenAI compatible API provided by Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-chat-completions.html).

## Creating a Bedrock Agent with Microsoft Agent Framework

To create a Bedrock Agent with the Microsoft Agent Framework, you need to configure the `OpenAIClient` to use the Bedrock endpoint and provide the necessary Api Key. Below is an example of how to set up a Bedrock Agent using C#:

```csharp
#:package Microsoft.Agents.AI.OpenAI@1.0.0-preview.251110.2

using System.ClientModel;
using OpenAI;
using OpenAI.Chat;

OpenAIClient client = new(
    new ApiKeyCredential("<your_aws_bedrock_api_key>"),
    new OpenAIClientOptions()
    {
        Endpoint = new Uri("https://bedrock-runtime.<your_aws_region>.amazonaws.com/openai/v1"),
    });

var chatCompletionClient = client.GetChatClient("openai.gpt-oss-120b-1:0");

ChatCompletion chatCompletion = await chatCompletionClient.CompleteChatAsync("Hello, how can I use AWS Bedrock with Microsoft Agent Framework?");

Console.WriteLine(chatCompletion.Content[0].Text.Trim());
```

Happy building! 🚀
