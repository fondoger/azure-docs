---
title: Azure Functions on Azure Container Apps overview
description: Learn how Azure Functions works with autoscaling rules in Azure Container Apps.
services: container-apps
author: craigshoemaker
ms.service: azure-container-apps
ms.topic:  how-to
ms.date: 04/07/2025
ms.author: cshoe
---

# Azure Functions on Azure Container Apps overview

Azure Functions provides integrated support for developing, deploying, and managing containerized Functions apps on Azure Container Apps. Use Azure Container Apps for your Functions apps when you need to run in the same environment as other microservices, APIs, websites, workflows, or any container hosted programs.

Container Apps hosting lets you run your Functions in a fully supported and managed, container-based environment with built-in support for open-source monitoring, mTLS, Dapr, and Kubernetes Event-driven Autoscaling (KEDA).

As an integrated feature on Azure Container Apps,  you can  deploy Azure Functions images directly onto Azure Container Apps using the `Microsoft.App` resource provider by setting `kind=functionapp` when calling `az containerapp create`. Apps created this way have access to all Azure Container Apps features.

This article shows you how to create and deploy an Azure Functions app that runs within Azure Container Apps. You learn how to:

- Set up a containerized Functions app with preconfigured auto scaling rules
- Deploy your application using either the Azure portal or Azure CLI
- Verify your deployed Functions with an HTTP trigger

By running Functions in Container Apps, you benefit from automatic scaling, easy configuration, and a fully managed container environment—all without having to manage the underlying infrastructure yourself.

## Key benefits
The Container Apps hosting model builds on the flexibility of containerized workloads and the event-driven nature of Azure Functions. It offers the following key advantages:
- **Run Functions as containers** with custom dependencies and language stacks.
- **Scale in to zero and scale out to 1000 instances** using KEDA.
- **[Secure networking](../container-apps/networking.md)** with full [VNet integration](../container-apps/custom-virtual-networks.md).
- **Advanced [Container App features](../container-apps/overview.md#features)** like multi-revisions, traffic splitting, [Dapr integration](../container-apps/dapr-overview.md) and [observability components](../container-apps/observability.md).
- **[Serverless and Dedicated GPU](../container-apps/gpu-serverless-overview.md)** support for compute-intensive workloads.
- **Unified Container Apps environment** to run Functions alongside microservices, APIs, and background jobs.

The following table helps you compare the features of Functions on Container Apps with [Flex consumption plan](../azure-functions/flex-consumption-plan.md).

| Feature | Container Apps | Flex Consumption Plan |
| --- | --- | --- |
| Scale to zero | ✅ Yes (via KEDA) | ✅ Yes |
| Max scale-out | 1,000 (default 10, configurable) |	1,000 |
| Always-on instances | ✅ Yes (via `minReplicas`) | ✅ Yes (via always-ready instances) |
| VNet integration |	✅ Yes | ✅ Yes |
| Custom container support | ✅ Yes (bring your own image) | ❌ Limited (no bring your own container) |
| GPU support | ✅ Yes (via serverless GPU dedicated workload profile) | ❌ No |
| Built-in features | Container Apps feature support. For instance, KEDA, Dapr, multi-revisions, mTLS, sidecars, ingress control and more | Functions-only features |
| Billing model | Container Apps pricing: Consumption plan (vCPU, memory, requests) & Dedicated plan (workload profile based) | Execution-time + always-ready instances |

For a complete comparison of the Functions on Container Apps against Flex Consumption plan and all other plan and hosting types, see [Functions scale and hosting options](../azure-functions/functions-scale.md).

## Scenarios


Azure Functions on Azure Container Apps provide a versatile combination of services to meet the needs of your applications. The following scenarios are representative of the types of situations where paring Azure Container Apps with Azure Functions gives you the control and scaling features you need.

- **Line-of-business APIs**: Package custom libraries, packages, and APIs with Functions for line-of-business applications.

- **Migration support**: Migration of on-premises legacy and/or monolith applications to cloud native microservices on containers.

- **Event-driven architecture**: Supports event-driven applications for workloads already running on Azure Container Apps.

- **Serverless workloads**: Serverless workload processing of videos, images, transcripts, or any other processing intensive tasks that required  GPU compute resources.

- **Common Azure Functions scenarios**: All common Azure Functions scenarios like processing file uploads, running scheduled tasks, responding to database changes, machine learning/AI and others detailed in [Azure Functions scenarios](/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp).

## Pricing and billing
Azure Functions on Azure Container Apps follow the same pricing model as Azure Container Apps. Billing is based on the [plan type](../container-apps/plans.md) you select for your environment, which can be either Consumption or Dedicated.
- [Consumption plan](..//container-apps/billing.md#consumption-plan): This serverless compute option bills you only for the resources your apps use while they are running.
- [Dedicated plan](../container-apps/billing.md#consumption-dedicated): This option provides customized compute resources, billing you for the instances allocated to each workload profile.

Your choice of plan determines how billing calculations are made. Different applications within an environment can use different plans.

Key points to note:
- There are no additional charges for using the Azure Functions programming model within Container Apps.
- Durable Functions and other advanced patterns are supported and billed under the same Container Apps pricing model.
For detailed billing mechanics and examples, refer to the [Billing in Azure Container Apps](../container-apps/billing.md) documentation.

## Event-driven scaling

All Functions triggers are available in your containerized Functions app. However, only the following triggers can dynamically scale (from zero instances) based on received events when running in a Container Apps environment:

- Azure Event Grid
- Azure Event Hubs
- Azure Blob Storage (Event Grid based)
- Azure Queue Storage
- Azure Service Bus
- [Durable Functions (MSSQL storage provider)](../azure-functions/durable/durable-functions-mssql-container-apps-hosting.md)
- Durable Functions (DTS storage provider)
- HTTP
- Kafka
- Timer
- Azure Cosmos DB

Azure Functions on Azure Container Apps are designed to configure the scale parameters and rules as per the event target. You don't need to worry about configuring the KEDA scaled objects. You can still set minimum and maximum replica count when creating or modifying your Functions app.

You can write your Functions code in any [language stack supported](/azure/azure-functions/supported-languages?tabs=isolated-process%2Cv4&pivots=programming-language-csharp) by Azure Functions. You can use the same Functions triggers and bindings with event-driven scaling.

## Managed identity authorization

To adhere to security best practices, connect to remote services using Microsoft Entra authentication and managed identity authorization.

Managed identities are available for the following connections:

- [Default storage account](../azure-functions/functions-reference.md#connecting-to-host-storage-with-an-identity) (AzureWebJobsStorage)
- Azure Container Registry: When running in Container Apps, you can use Microsoft Entra ID with managed identities for all binding extensions that support managed identities. Currently, only these binding extensions support event-driven scaling when using managed identity authentication:
- Azure Event Hubs
- Azure Queue Storage
- Azure Service Bus

For other bindings, use fixed replicas when using managed identity authentication. For more information, see the [Functions developer guide](/azure/azure-functions/functions-reference).

## Scaling and performance

Azure Functions on Container Apps scale automatically based on events using KEDA, with no need to configure scale rules manually. You can still set min/max replicas to control scaling behavior.

- **Event-driven scaling**: Automatically scales based on triggers like Event Grid, Service Bus, or HTTP.
- **Scale to zero**: Idle apps scale-in to zero to save costs.
- **Cold start control**: Avoid cold starts by setting `minReplicas` ≥ 1.
- **Concurrency**: Each instance can process multiple events in parallel.
- **High scale**: Scale out to 1,000 instances per app (default is 10).
- **GPU support**: Run compute-heavy workloads like AI inference using GPU-backed nodes.

This makes Container Apps ideal for both bursty and steady-state workloads. To learn more, see [Set scaling rules in Azure Container Apps](../container-apps/scale-app.md)

## Networking and security

Azure Functions on Container Apps benefit from Container Apps’ robust [networking](../container-apps/networking.md) and [security features](../container-apps/security.md) for secure, scalable deployments:

- **VNet integration**: Access private resources securely via internal endpoints and private databases.
- **Managed identity**: Authenticate with Azure services using system/user-assigned identities—no secrets or connection strings needed.
- **Dapr support**: Enable pub/sub, state management, and secure service invocation via Dapr sidecars. For more information, see [Microservice APIs powered by Dapr](../container-apps/dapr-overview.md).
- **Ingress and TLS**: Expose secure HTTP endpoints with TLS/mTLS, custom domains, or keep them internal.
- **Environment Isolation**: Functions share Container Apps environment boundaries for secure, scoped communication.

These capabilities make Container Apps-hosted Functions ideal for enterprise-grade, secure serverless applications.

## Application logging

You can monitor your containerized Functions app hosted in Container Apps using Azure Monitor Application Insights in the same way you do with apps hosted by Azure Functions. For more information, see [Monitor Azure Functions](/azure/azure-functions/monitor-functions).

For bindings that support event-driven scaling, scale events are logged as `FunctionsScalerInfo` and `FunctionsScalerError` events in your Log Analytics workspace. For more information, see [Application Logging in Azure Container Apps](./logging.md).

## Submit Feedback

Submit an issue or a feature request to the [Azure Container Apps GitHub repo](https://github.com/microsoft/azure-container-apps/issues).

## Next steps

> [!div class="nextstepaction"]
> [Use Azure Functions in Azure Container Apps](functions-usage.md)
