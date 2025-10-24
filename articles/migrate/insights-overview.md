---
title: Insights In Azure Migrate - Assess, Risk, Plan Cloud Migration
description: Discover how Azure Migrate's Insights (preview) feature helps identify vulnerabilities, end-of-support software, and missing security tools in your datacenter. Plan secure and efficient cloud migrations with early risk visibility and actionable insights.
author: habibaum
ms.author: v-uhabiba
ms.service: azure-migrate 
ms.topic: how-to 
ms.date: 09/19/2025
ms.custom: engagement-fy24 
::: moniker range="migrate"
# Customer intent: IT administrators and cloud architects use the Insights (preview) feature in Azure Migrate to identify and mitigate security risks in their datacenter during cloud migration planning. This helps them assess vulnerabilities, outdated software, and missing security tools to ensure a secure and efficient migration to Azure
---

# Insights in Azure Migrate (Preview)

This article describes the **Insights** (preview) feature in Azure Migrate, which provides a security assessment of the infrastructure and software inventory discovered in your datacenter.

## Key benefits of Insights: What users gain 

- Review security risks in your datacenter early during migration planning.
- Plan mitigation to fix security issues and make your migration to Azure smooth and secure.
- Identify and plan upgrade of Windows and Linux servers with end of support operating system, end of support software and pending updates. 
- Detect vulnerabilities in discovered software and take action to remediate risks.
- Identify servers without security or patch management software, and plan to configure [Microsoft Defender for Cloud](/azure/defender-for-cloud/) and [Azure Update Manager](../update-manager/overview.md).


## Security Insights data

Azure Migrate currently focuses on a core set of security risk areas. Each area corresponds to a specific security insight. The following table summarizes the available insights data: 

| Resource | Security Insight | Details | 
| --- | --- | --- | 
| Servers | With security risks </br> </br> </br> </br> OS end of support </br> </br> Software end of support </br> </br> With vulnerabilities </br> </br> </br> </br> Missing security software. </br> </br> </br> </br> Missing patch management software. </br></br>  </br> </br> Pending updates    | Servers are flagged if they have at least one of the following security risks: End-of-support operating system, End-of-support software, Known vulnerabilities (CVEs), Missing security or patch management software, Pending critical or security updates </br> </br> Servers with end of support Operating system. </br> </br> Servers with end of support Software discovered in Azure Migrate. </br> </br> </br> </br> Servers with known vulnerability (CVE) in OS and discovered Software. </br> </br> </br> Servers without any discovered software belonging to Security software category. </br> </br> </br> </br> Servers without any discovered patch management software. </br> </br> </br> </br> Servers with pending updates or patches.|
|  Software    | With security risks </br> </br> End of support </br> </br> With vulnerabilities  | Software with at least one of the security risks – end of support, vulnerabilities. </br> </br> Software declared end of support by vendor. </br> </br> Software with known vulnerability (CVE)  | 

### How are Insights derived

Azure Migrate identifies potential security risks in your datacenter using software inventory data collected through the [Azure Migrate appliance discovery process](how-to-review-discovered-inventory.md#deploy-and-configure-the-azure-migrate-appliance). When you run a discovery of your on-premises environment, you usually provide guest credentials for your Windows and Linux servers. This allows the tool to collect information about installed software, operating system configuration, and pending updates. Azure Migrate processes this data to generate key security insights without needing additional credentials or permissions.

>[!Note]
> Azure Migrate does not install additional agents or run a deep scan of your environment. Security insights are limited to software and operating system data discovered through the Azure Migrate appliance quick discovery. It analyzes the collected software inventory data and cross-references it with publicly available vulnerability and support lifecycle databases to highlight security risks in your datacenter.

Security risks are derived through a series of following analyses:

- **End-of-support software**: Azure Migrate checks the versions of discovered software against the publicly available [endoflife.date](https://endoflife.date/) repository. All end of life data is refreshed every 7 days. If software is found to be end of support (meaning the vendor no longer provides security updates), it flags it as a security risk. Identifying unsupported software early helps you plan upgrades or mitigations as part of your cloud migration.

- **Vulnerabilities**: Azure Migrate identifies installed software and operating system (OS) for each server. It maps the discovered software and OS to CPE nomenclature (Common Platform Enumeration) using AI model, which provides a unique identification for each software version. It stores only software metadata (name, publisher, version) and doesn't capture any organization-specific information. Azure Migrate correlates the CPE names with known CVE IDs (Common Vulnerabilities and Exposures). CVE IDs are unique identifiers assigned to publicly disclosed cybersecurity vulnerabilities and help organizations identify and track vulnerabilities in a standard way. Refer to [CVE](https://www.cve.org/) for more details. Information about CVE IDs and related software comes from the publicly available [National Vulnerability Database](https://nvd.nist.gov/) (NVD), managed by NIST. This helps identify vulnerabilities in the software. Each vulnerability is categorized by risk level (Critical, High, Medium, Low) based on the [CVSS](https://nvd.nist.gov/vuln-metrics/cvss) score provided by NVD. This feature uses the NVD API for all CVE IDs but is not endorsed or certified by the NVD. All CVE data from NVD is refreshed every 7 days. 

- **Pending Updates for servers**: Azure Migrate identifies machines that are not fully patched or updated based on Windows Update metadata for Windows servers and Linux package manager metadata for Linux servers. It also retrieves the classification of these updates (Critical, Security, Other updates) and shows them for further consideration. Azure Migrate refreshes data from Windows Updates and Linux package managers every 24 hours. This insight appears as Servers with pending security and critical updates, indicating that the server is not fully patched and should be updated.

- **Missing Security and Patch Management Software**: Azure Migrate also identifies unprotected machines by detecting the absence of software in Security & Compliance category.
For example, if the software inventory indicates that a server doesn't have software in categories such as, antivirus, threat detection, SIEM, IAM, or patch management, the system highlights it as a potential security risk.

Azure Migrate updates security insights whenever it refreshes discovered software inventory data. The platform updates insights when you run a new discovery or when the Azure Migrate appliance sends inventory updates. You usually run a full discovery at the start of a project and may do periodic re-scans before finalizing an assessment. Any system changes, such as, new patches or software reached end-of-life, will reflect in the updated security insights.

>[!Note]
> Security insights in Azure Migrate help guide and highlight potential security risks in the datacenter. They are not meant to be compared with specialized security tools. We recommend to adopt Azure services such as, [Microsoft Defender for Cloud](/azure/defender-for-cloud/) and [Azure Update Manager](../update-manager/overview.md) for comprehensive protection of your hybrid environment.

- **Determine security risk for a server**:  A server is considered at security risk if it meets any of the following conditions.
- **Calculate security risk score**: The security risk score for a server is calculated as: 
    OS end-of-support flag + Software end-of-support flag + Number of vulnerabilities + Number of pending critical and security updates + Security software flag + Patch management software flag.

    - **OS end-of-support flag** = 1 if the server operating system is at end of support; otherwise, 0.
    - **Software end-of-support flag** = 1 if the software is at end of support; otherwise, 0.
    - **Number of vulnerabilities** = Count of CVEs identified for the server.
    - **Number of pending critical and security updates** = Pending updates for Windows and Linux servers that are classified as Critical or Security.
    - **Security software flag** = 1 if no software belonging to the Security category was discovered on the server; otherwise, 0.
    - **Patch management software flag** = 1 if no software belonging to the Patch Management category was discovered on the server; otherwise, 0. 
 

## Prerequisites for reviewing Insights 

Ensure the following prerequisites are met for generating complete Insights:

- Use [appliance-based discovery in Azure Migrate](how-to-review-discovered-inventory.md) to review Insights. It might take up to 24 hours after discovery to generate Insights.[Import-based discovery](discovery-methods-modes.md) isn't supported.
- Use an existing project or create an [Azure Migrate project using portal](quickstart-create-project.md).
- Ensure guest discovery features are enabled on the appliance(s).
- Ensure there are no software discovery issues. Go to **Action Center** in Azure Migrate project to filter issues for software inventory. 
- Ensure Software inventory is gathered actively for all servers at least once in last 30 days

## Review Insights 

To review insights in Azure Migrate:

1. Go to the **[Azure Migrate](https://portal.azure.com)** portal.
1. Select your project from **All Projects**.
1. In the left menu, select **Explore inventory** > **Insights (preview)** to review security insights for the selected project. This page provides a summary of security risks across discovered servers and software. 

    :::image type="content" source="./media/security-insights-overview/insights-preview.png" alt-text="Screenshot shows the security risks across discovered servers and software." lightbox="./media/security-insights-overview/insights-preview.png":::

1. Select any insight to review detailed information. The **Summary card** highlights critical security risks in your datacenter that need immediate attention. It identifies:
    - Servers with critical vulnerabilities that benefit from enabling Microsoft Defender for Cloud after migration. 
    - Servers running end-of-support operating systems, recommending upgrades during migration.
    - The number of servers with pending critical and security updates, suggesting remediation using Azure Update Manager post-migration. 
    You can tag servers with critical risks to support effective planning and mitigation during modernization to Azure.

    :::image type="content" source="./media/security-insights-overview/summary-card.png" alt-text="Screenshot shows the summary of critical security risks in the datacenter that needs attention." lightbox="./media/security-insights-overview/summary-card.png":::

### Review Server risk assessment

The **Servers card** shows a summary of all discovered servers with security risks. A server is considered at risk if it has at least one of the following issues:

  - End-of-support operating system
  - End-of-support software
  - Known vulnerabilities (CVEs) in installed software or OS
  - Missing security or patch management software
  - Pending critical or security updates

    :::image type="content" source="./media/security-insights-overview/servers-card.png" alt-text="Screenshot shows the summarized view of all servers with security risks out of total discovered servers." lightbox="./media/security-insights-overview/servers-card.png":::

### Review Software risk assessment 

The **Software Card** shows a summary of all discovered software with security risks. Software is flagged as at risk if it is either end-of-support or has known vulnerabilities (CVEs). The card displays the number of end-of-support software and software with vulnerabilities as fractions of the total software with security risks.

:::image type="content" source="./media/security-insights-overview/software-card.png" alt-text="Screenshot provides aggregated view of all software with security risks out of total discovered software." lightbox="./media/security-insights-overview/software-card.png":::

## Review detailed Security Insights 

To review detailed security risks for Servers and Software, perform the following actions:

### Review Servers with security risks

To review detailed security risks for servers, follow these steps:

1. Open the **Insights** (preview) pane.
1. In the **Servers card**, select the link that shows the number of servers with security risks.

    :::image type="content" source="./media/security-insights-overview/servers-risk-type.png" alt-text="Screenshot shows the servers with security risks." lightbox="./media/security-insights-overview/servers-risk-type.png":::

1. You can view the detailed list of discovered servers, apply tags to support migration planning, and export the server data as a .csv file.

    :::image type="content" source="./media/security-insights-overview/servers-with-security-risks.png" alt-text="Screenshot shows the detailed list of discovered servers." lightbox="./media/security-insights-overview/servers-with-security-risks.png":::

### View Servers by security risk

To view servers with specific security risks, go to the **Insights (preview)** pane. On the **Servers card**, you see and select a detailed list of servers affected by the following issues:

 - End-of-support operating systems
 - End-of-support software
 - Known vulnerabilities (CVEs) in installed software or operating systems
 - Missing security or patch management tools

:::image type="content" source="./media/security-insights-overview/servers-impacted.png" alt-text="Screenshot shows the detailed list of servers impacted by each security risk." lightbox="./media/security-insights-overview/servers-impacted.png":::

Alternatively, you can filter servers with security risks from the **Explore inventory** > **All inventory** and **Explore inventory** > **Infrastructure** pane.

:::image type="content" source="./media/security-insights-overview/server-filters-with-security-risks.png" alt-text="Screenshot shows how to filter servers with security risks." lightbox="./media/security-insights-overview/server-filters-with-security-risks.png":::

### Review Software with security risks 

To review software with identified security risks, follow these steps:

1. Open the **Insights** (preview) pane.
1. In the **Software** card, select the link that shows the number of software items with security risks.

    :::image type="content" source="./media/security-insights-overview/software-with-security-risks.png" alt-text="Screenshot shows the number of software security risks." lightbox="./media/security-insights-overview/software-with-security-risks.png":::

1. You can view the detailed list of discovered software, examine associated metadata, and export the data as a .csv file.

    :::image type="content" source="./media/security-insights-overview/metadata-export-view.png" alt-text="Screenshot shows detailed list of discovered software and its metadata." lightbox="./media/security-insights-overview/metadata-export-view.png":::


1. To view software with specific security risks, go to the **Insights** (preview) pane. here, you see a detailed list of software affected due to the following issues:

    - End-of-support status
    - Known vulnerabilities (CVEs)

    :::image type="content" source="./media/security-insights-overview/software-impacted.png" alt-text="Screenshot shows detailed list of software impacted by each security risk." lightbox="./media/security-insights-overview/software-impacted.png":::

    Alternatively, you can filter end-of-support software and software with known vulnerabilities from the **Explore inventory** > **Software** pane.

    :::image type="content" source="./media/security-insights-overview/software-with-vulnerabilities.png" alt-text="Screenshot shows how to filter end of support software with vulnerabilities." lightbox="./media/security-insights-overview/software-with-vulnerabilities.png":::

### Review detailed Security Insights for a server 

To view detailed security insights for a specific server:

1. Go to the **Infrastructure** pane from the left menu and select the server you want to review.
1. Select the **Insights** (preview) tab.
    The tab displays security insights for the selected server, including: 
    - Operating system support status
    - Presence of security and patch management software
    - Pending critical and security updates
    - End-of-support software
    - Software with known vulnerabilities (CVEs)
    The summary of the top five pending updates and top five vulnerabilities is provided to help prioritize remediation.

    :::image type="content" source="./media/security-insights-overview/pending-updates.png" alt-text="Screenshot shows the pending updates and vulnerabilities." lightbox="./media/security-insights-overview/pending-updates.png":::

## Manage permissions for Security Insights 

Security insights are enabled by default for all users. To manage access, create [custom roles](/azure/role-based-access-control/custom-roles-portal#step-3-basics) and remove the following permissions:

| Resource | Permissions | Description | 
| --- | --- | --- | 
| Pending updates |`Microsoft.OffAzure/hypervSites/machines/inventoryinsights/pendingupdates/* ` </br> </br> </br> </br>`Microsoft.OffAzure/serverSites/machines/inventoryinsights/pendingupdates/*` </br> </br> </br> </br>`Microsoft.OffAzure/vmwareSites/machines/inventoryinsights/pendingupdates/*`   | Read pending updates of Hyper-V site </br> </br> Read pending updates of physical server site  </br> </br> Read pending updates of VMware machine  | 
| Vulnerabilities| `Microsoft.OffAzure/hypervSites/machines/inventoryinsights/vulnerabilities/* `</br> </br> </br> </br>`Microsoft.OffAzure/serverSites/machines/inventoryinsights/vulnerabilities/*` </br> </br> </br> </br>`Microsoft.OffAzure/vmwareSites/machines/inventoryinsights/vulnerabilities/*` |  Read vulnerabilities of Hyper-V site </br> </br> Read vulnerabilities of physical server site </br> </br> Read vulnerabilities of VMware machine   | 


:::image type="content" source="./media/security-insights-overview/permissions.png" alt-text="Screenshot shows that you don't have permissions to view." lightbox="./media/security-insights-overview/permissions.png":::

>[!Note]
> Support status for operating systems and software is a machine-level property. User access to this information is determined by the permissions assigned at the machine level.

## Next steps

- Learn more about [permissions in Azure Migrate](/azure/role-based-access-control/permissions/migration#microsoftmigrate).
- Learn more about [creating custom role](/azure/role-based-access-control/role-definitions).
- Learn more about [Security cost in Business case](concepts-business-case-calculation.md).
- Learn more about [Assessments](concepts-overview.md).