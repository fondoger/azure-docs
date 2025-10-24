---
title: SAP Testing Automation Framework High Availability Testing
description: Learn about High Availability testing capabilities in the SAP Testing Automation Framework
author: devanshjain
ms.author: devanshjain
ms.reviewer: depadia
ms.topic: how-to
ms.service: sap-on-azure
ms.subservice: sap-automation
ms.date: 10/23/2025
ms.custom: devx-track-azurecli, devx-track-azurepowershell, linux-related-content
---

# SAP Testing Automation Framework: High Availability Testing

High Availability (HA) is essential for maintaining business continuity in SAP landscapes. The SAP Testing Automation Framework provides a structured, automated approach to validating HA configuration and resilience for SAP HANA (scale-up) and SAP Central Services. It executes configuration validation checks and orchestrates controlled failure simulations to ensure that recovery and failover mechanisms comply with SAP on Azure best practices.

The framework uses Ansible to coordinate test execution, collect telemetry, capture logs, and generate detailed outcome reports. Tests cover scenarios such as resource migration, process crashes, node failures, fencing events, and network partitions helping teams confidently assess readiness before go‑live or during lifecycle operations.

## SAP HANA Scale-Up High Availability

Validates the failover mechanism of SAP HANA in a scale-up configuration, ensuring that the database can recover from node failures without data loss or significant downtime. The following test cases are available to validate SAP HANA high availability:

| Test Case | Description | Applicable |
|-----------|-------------|-------------|
| **High Availability Parameters Validation** | Checks high availability configuration including Corosync settings, Pacemaker resources, SBD devices, and HANA system replication setup. |  |
| **Azure Load Balancer** | The Azure LB configuration test validates Azure Load Balancer setup including health probe configuration, backend pool settings, load balancing rules, and frontend IP configuration. |  |
| **Resource Migration** | The Resource Migration test validates planned failover scenarios by executing controlled resource movement between HANA nodes. It performs a graceful migration of the primary HANA resources to the secondary node, verifies proper role changes, ensures cluster maintains stability throughout the transition, and validates complete data synchronization after migration. |  |
| **HANA Stop on Primary** | The HANA stop on primary test simulates cluster behavior when HANA database is stopped manually. The monitoring of SAP HANA resource agent would identify that SAP HANA database has been stopped, and would promote the database to secondary node.|  |
| **Block Network** | The Block Network test validates cluster behavior during network partition scenarios by implementing iptables rules to block communication between primary and secondary HANA nodes. It verifies split-brain prevention mechanisms, validates proper failover execution when nodes become isolated, and ensures cluster stability and data consistency after network connectivity is restored. |  |
| **Primary Index Server Crash** | The Primary Index Server Crash test validates high availability behavior by forcefully terminating the HANA index server process on the primary node. This simulates a critical service failure, triggering automatic failover to the secondary node. The test verifies proper failover execution, ensures data consistency, and validates service restoration after recovery. |            |
| **Primary Node Kill** | The Primary Node Kill test validates cluster behavior by forcefully terminating all HANA processes on the primary node using SIGKILL signal. This simulates an abrupt service failure, triggering automatic failover to the secondary node. The test verifies proper promotion of secondary to primary, ensures data consistency, and validates complete cluster recovery. |  |
| **Primary Echo B** | The Primary Echo B test simulates an immediate system crash on the primary HANA node by executing the 'echo b' command to trigger an abrupt reboot without proper shutdown. This tests the cluster's ability to handle unexpected primary node failures, validates proper failover execution, and verifies data consistency after recovery. |  |
| **Secondary Index Server Crash** | The Secondary Index Server Crash test simulates failure of the HANA index server process on the secondary node. It validates that the primary node continues normal operation while verifying the cluster's ability to handle secondary failures, tests automatic recovery mechanisms, and ensures system replication resumes properly after service restoration. |  |
| **Secondary Node Kill** | The Secondary Node Kill test examines cluster resilience by forcefully terminating HANA processes on the secondary node using the kill -9 signal. The test validates that the primary node maintains normal operation while the secondary node undergoes recovery, ensuring cluster stability and proper data synchronization after the recovery process completes. |  |
| **Secondary Echo B** | The Secondary Echo B test simulates an uncontrolled system crash on the secondary HANA node by executing the 'echo b' command, triggering an immediate reboot without proper shutdown procedures. The test validates that the primary node maintains operation, verifies cluster stability, and ensures system replication resumes correctly after the secondary node recovers. |  |
| **Filesystem Freeze** | The Filesystem Freeze test validates cluster behavior when the primary node's filesystem becomes unresponsive. It simulates a storage issue by freezing the filesystem on the primary node running HANA database, which triggers automatic failover to the secondary node. The test verifies proper cluster reaction, resource migration, and data consistency after recovery. This check would only execute when SAP HANA is configured with Azure NetApp Files (ANF). |  |
| **SBD Fencing** | Validates cluster fencing mechanism by killing the SBD inquisitor process on the primary node. Tests proper fence detection, node isolation, and automated failover to ensure cluster integrity during hardware or communication failures. This check would only execute when SAP HANA is configured with SBD Stonith mechanism. |  |

## SAP Central Services High Availability

Validates the failover mechanism of SAP Central Services, ensuring that the system can recover from node failures without impacting the availability of critical services. The following test cases are available to validate SAP Central Services high availability:

| Test Case | Description | Applicable |
|-----------|-------------|-------------|
| **HA Parameters Validation** | The HA parameter validation test validates HA configuration including Corosync settings, Pacemaker resources, SBD device configuration, and SCS system replication setup. |  |
| **Azure Load Balancer** | The Azure LB configuration test validates Azure Load Balancer setup including health probe configuration, backend pool settings, load balancing rules, and frontend IP configuration. |  |
| **SAP Control Config Validation** | The SAPControl Config Validation test runs multiple sapcontrol commands to validate the SCS configuration. It executes commands like HAGetFailoverConfig, HACheckFailoverConfig, and HACheckConfig, capturing their outputs and statuses to ensure proper configuration and functionality. This check is only valid for SAP System running on SUSE Linux Enterprise Server (SLES). |  |
| **Resource Migration** | The Resource Migration test validates planned failover scenarios by controlling resource movement between SCS nodes, ensuring proper role changes. |  |
| **ASCS Node Crash** | The ASCS Node Crash test simulates cluster behavior when the ASCS node crashes. It simulates an ASCS node failure by forcefully terminating the process, then verifies automatic failover to the ERS node, monitors system replication status, and confirms service recovery. |  |
| **Block Network Communication** | The Block Network test validates cluster behavior during network partition scenarios by implementing iptables rules to block communication between ASCS and ERS nodes. It verifies split-brain prevention mechanisms, validates proper failover execution when nodes become isolated, and ensures cluster stability after network connectivity is restored. |  |
| **Kill Message Server Process** | The Message Server Process Kill test simulates failure of the message server process on the ASCS node by forcefully terminating it using the kill -9 signal. It verifies proper cluster reaction, automatic failover to the ERS node, and ensures service continuity after the process failure. |  |
| **Kill Enqueue Server Process** | The Enqueue Server Process Kill test simulates failure of the enqueue server process on the ASCS node by forcefully terminating it using the kill -9 signal. It validates proper cluster behavior, automatic failover execution. |  |
| **Kill Enqueue Replication Server Process** | The Enqueue Replication Server Process Kill test simulates failure of the replication server process on the ERS node by forcefully terminating it using the kill -9 signal. This test handles both ENSA1 and ENSA2 architectures. It validates the automatic restart of the process. |  |
| **Kill sapstartsrv Process for ASCS** | The sapstartsrv Process Kill test simulates failure of the SAP Start Service for the ASCS instance by forcefully terminating it using the kill -9 signal. It validates proper cluster reaction, automatic failover to the ERS node, and verifies service restoration after the process failure. This check is only valid for SAP Central Services configured using simple mount on SUSE Linux Enterprise Server (SLES). |  |
| **Manual Restart of ASCS Instance** | The Manual Restart test validates cluster behavior when the ASCS instance is manually stopped using sapcontrol. It verifies proper cluster reaction to a controlled instance shutdown, ensures automatic failover to the ERS node, and confirms service continuity throughout the operation. |  |
| **HAFailoverToNode Test** | The HAFailoverToNode test validates SAP's built-in high availability functionality by using the sapcontrol command to trigger a controlled failover. It executes 'HAFailoverToNode' as the SAP administrator user, which initiates a clean migration of the ASCS instance to another node. This check is only valid for SAP System running on SUSE Linux Enterprise Server (SLES). |  |

## Offline Validation of High Availability Configuration

Offline validation is a mode of the SAP Testing Automation Framework that validates SAP HANA and SAP Central Services high availability cluster configurations without establishing a live SSH connection to the production cluster. Instead, it analyzes captured cluster information base (CIB) XML files exported from each cluster node. This approach enables repeatable and non-intrusive assessment of HA configuration, ideal for compliance audits, pre change reviews, and air-gapped analysis. For more details on how to run offline validation, see [high availability configuration offline validation]([sap-automation-qa/docs/HA_OFFLINE_VALIDATION.md at main · Azure/sap-automation-qa](https://github.com/Azure/sap-automation-qa/blob/main/docs/HA_OFFLINE_VALIDATION.md)).

> [!NOTE]
>
> The offline validation does not run any functional tests.

## Next Steps

- [Understand configuration validation checks](testing-framework-configuration-checks.md)
- [Review the framework architecture](testing-framework-architecture.md)
- [Learn about supported platforms](testing-framework-supportability.md)