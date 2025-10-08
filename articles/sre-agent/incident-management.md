---
title: Incident Management in Azure SRE Agent Preview
description: Learn how the incident management capabilities in Azure SRE Agent help reduce manual intervention and accelerate resolution times for your Azure resources.
author: craigshoemaker
ms.topic: conceptual
ms.date: 09/23/2025
ms.author: cshoe
ms.service: azure-sre-agent
---

# Incident management in Azure SRE Agent Preview

Azure SRE Agent Preview streamlines incident management by automatically collecting, analyzing, and responding to alerts from various management platforms. This article explains how the agent processes incidents, evaluates their severity, and takes appropriate actions based on your configuration.

SRE Agent receives alerts from incident management platforms such as:

* [Azure Monitor alerts](/azure/azure-monitor/alerts/alerts-overview)
* [PagerDuty](https://www.pagerduty.com/)
* [ServiceNow](https://www.servicenow.com/)

When SRE Agent receives an alert from the management platform, the agent brings the incident into its context, analyzes the incident, and determines the next steps. This process mimics how a human SRE would acknowledge and investigate an incident.

After being notified of an issue, SRE Agent reviews logs, health probes, and other telemetry to assess the incident. During the assessment step, the agent summarizes findings, determines if the alert is a false positive, and decides whether action is needed.

When SRE Agent receives an alert from the management platform, the agent brings the incident into its context, analyzes the situation, and determines the next steps. This process mimics how a human site-reliability engineer would acknowledge and investigate an incident.

The agent reviews logs, health probes, and other data to assess the incident. During the assessment step, the agent summarizes findings, determines if the alert is a false positive, and decides whether action is needed.

## Agent responses

SRE Agent responds to incidents based on its configuration and operational mode:

When an incident is detected, a new thread appears in the chat history that includes the initial analysis done by the agent.

Depending on the [incident response plan](incident-response-plan.md), the agent can respond in a semi-autonomous, or fully autonomous manner.

* **Reader mode**: The agent provides recommendations and requires human intervention for resolution.

* **Autonomous mode**: The agent can automatically close incidents or take corrective actions, depending on your configuration settings. The agent can also update or close incidents in management platforms to maintain synchronization across platforms.

You can control what type of incidents SRE Agent handles by controlling the configuration settings of the management platform. For instance, you might decide that all low priority incidents are sent to the SRE Agent, while high priority incidents require complete human attention.

Once an incident is sent to the SRE Agent, you then can control how the agent responds by customizing incident handlers. Within an incident handler, you have control over:

* The agent's autonomy level
* Tools available to the agent for reporting and remediation
* Custom instructions further guiding the agent how to deal with incidents

For more information, see [Incident response plan](incident-response-plan.md).

## Platform integration

Minimal setup is required for Azure Monitor (default integration). Non-Microsoft systems like PagerDuty and ServiceNow require extra setup for incident handling preferences.

To access the incident management settings, open your agent in the Azure portal and select the **Incident platform** tab.

# [Azure Monitor alerts](#tab/azmon-alerts)

By default, Azure Monitor alerts are configured as the agent's incident management platform. As the agent encounters incidents, any instances of Azure Monitor in any resource groups managed by SRE Agent process incident data.

To use a different management platform, first disconnect Azure Monitor as the agent's incident management platform.

# [PagerDuty](#tab/pagerduty)

To set up PagerDuty:

1. Select the **Incident platform** tab and enter the following settings:

   | Setting | Value |
   |---|---|
   | **Incident platform** | Select **PagerDuty**. |
   | **REST API access key** | Enter your PagerDuty REST API access key. |
   | **Quickstart handler** | Keep the checkbox selected. |

1. Select **Save**. PagerDuty is now responsible for managing incidents for the agent.

You can choose to enable a series of tools that provide granular control over how PagerDuty manages incidents. To further refine the incident management process, you can also customize the instructions to better control how PagerDuty responds to incidents.

# [ServiceNow](#tab/servicenow)

To set up ServiceNow:

1. Select the **Incident platform** tab and enter the following settings:

1. From the Incident platform drop-down, select **ServiceNow**.

1. Enter the ServiceNow specific settings:

    | Property | Value |
    |---|---|
    | ServiceNow endpoint | Enter the ServiceNow integration endpoint address. |
    | Username | Enter your username. |
    | Password | Enter your password. |
    | Quickstart handler | Keep the box checked for the **default incident handler**. |

1. Select **Save**.

---

For more information on how to customize your incident response plans, see [Incident response plans in Azure SRE Agent](./incident-response-plan.md).

## Related content

* [Incident response plans in Azure SRE Agent](./incident-response-plan.md)
