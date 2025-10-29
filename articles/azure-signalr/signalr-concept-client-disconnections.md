---
title: Understanding Client Disconnections and Reconnection in Azure SignalR service
description: This article provides information about nature of client disconnections and how to handle client re-connections effectively
author: kevinguo-ed
ms.author: kevinguo
ms.service: azure-signalr-service
ms.topic: conceptual
ms.date: 10/29/2025
---

# Understanding Client Disconnections and Reconnection

Persistent connections, such as those maintained through WebSockets, are inherently subject to interruptions. While Azure SignalR Service is designed for high reliability and scalability, occasional client disconnections are a reality that all internet-connected applications must account for.

This article explains the nature of client disconnections, how Azure SignalR Service handles them — especially during service maintenance — and how to design your application to minimize the impact of reconnections.


## High-level guidance

### 1. Connection interruptions are inevitable

Client disconnections can occur for a variety of reasons and sources — network instability, browser tab suspensions, load balancer re-configurations, or routine service maintenance. These are normal occurrences across all real-time applications.

Developers should design applications with the expectation that disconnections **will** happen, and implement **retry and reconnection logic** as a standard part of the connection lifecycle.

### 2. Make reconnections lightweight

Reconnection should be as lightweight and end users shouldn’t notice that a reconnection even happened whenever possible. From user's perspective, the app continues to function smoothly. 

Review your current flow to determine whether all operations tied to an active WebSocket connection are truly dependent on it. In many applications — such as notifications, auctions, chat, dashboards, or collaborative experiences— reconnections are quick *(typically within a few seconds)* and low-impact. For applications involving **sensitive or transactional data**, the reconnection flow might be heavier, but consider whether certain operations can be **decoupled** and performed via **REST APIs** instead of through the persistent connection.

### 3. Handle reconnections effectively

When a connection drops, clients typically establish a new connection and resume communication. How you handle this reconnection process greatly affects user experience and system reliability.

#### Use stateful reconnect when applicable

Azure SignalR Service [supports stateful reconnect](https://learn.microsoft.com/aspnet/core/signalr/configuration?view=aspnetcore-9.0&tabs=dotnet#configure-stateful-reconnect) that allows clients to resume their previous connection without losing their state. This works when the client reconnects using **the same connection ID** — for example, when a temporary network glitch occurs and the client recovers quickly.

With stateful reconnect enabled, the client can continue receiving messages missed during the brief disconnection window *(30 seconds)*, reducing data loss and minimizing the disruption perceived by end users.


#### When reconnecting with a new connection ID

In cases where a reconnection results in a different connection ID — such as after a full application restart — the client is treated as a new connection by Azure SignalR Service. Developers should plan for this scenario explicitly. 

Consider:
- Rejoining groups that the client was previously in,
- Restoring session state *(e.g., subscriptions)* from a reliable store,
- Handling missed messages by retrieving them from an application-level store or event log, if message delivery guarantee is important.

### 4. Manage load during large-scale reconnections

When a large number of clients disconnect and attempt to reconnect simultaneously, this can place additional load on downstream components *(e.g., authentication, state stores, or application servers)*. We recommend identifying these components and implementing **auto-scaling policies** to absorb temporary spikes in reconnection traffic.

If many clients try to negotiate connections at once, some of those requests may be throttled by the service. The limit is proportional to the number of units — more units give you a higher threshold. [You can learn more about how to add more units to your SignalR resource here.](https://learn.microsoft.com/azure/azure-signalr/signalr-howto-scale-signalr) Additionally, to reduce the chance of throttling, introduce a small random delay before each reconnect attempt. You can find [retry examples here](https://learn.microsoft.com/azure/azure-signalr/signalr-howto-troubleshoot-guide#how-to-restart-client-connection). 


### 5. Consider transport options and hosting models carefully

* **Long Polling** and **Server-Sent Events (SSE)** are fallbacks for environments without WebSocket support, but they typically **do not increase reliability**.
* **Self-hosting SignalR** gives you full control over maintenance timing and lifecycle management but also shifts operational and scaling responsibilities to your team.
* **Serverless mode** of Azure SignalR Service does not reduce the likelihood or impact of connection drops, since disconnections are a network-level behavior on the client side.


## Disconnections during service maintenance

The Azure SignalR Service team regularly performs maintenance for improvements in performance, reliability, security, and features. 

During planned maintenance, Azure SignalR Service employs a **graceful shutdown strategy** to minimize the impact on connected clients: connections are **gradually disconnected** over a set time window, allowing clients to reconnect progressively.
  
This approach provides a good balance between operational efficiency and user experience for most customers.
