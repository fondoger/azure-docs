---
 title: Description of Azure Storage read-access geo-redundant storage secondary region access
 description: Description of Azure Storage read-access geo-redundant storage secondary region access
 author: anaharris-ms
 ms.service: azure
 ms.topic: include
 ms.date: 07/02/2024
 ms.author: anaharris
 ms.custom: include file
---

- **Secondary region access**: With GRS and GZRS configurations, the secondary region isn't accessible for reads until a failover occurs.

    RA-GRS and RA-GZRS configurations provide read access to the secondary region during normal operations, but because of the asynchronous replication latency, might return slightly outdated data.
