---
title: How to use managed identities with Azure File Sync
description: Learn how to configure Azure File Sync to use system-assigned managed identities provided by Microsoft Entra ID. 
author: khdownie
ms.service: azure-file-storage
ms.topic: concept-article
ms.date: 05/01/2025
ms.author: kendownie
# Customer intent: "As an IT administrator, I want to configure Azure File Sync to use system-assigned managed identities, so that I can enhance security by eliminating the need for shared keys and streamline authentication to Azure file shares."
---

# How to use managed identities with Azure File Sync

Managed Identity support eliminates the need for shared keys as a method of authentication by utilizing a system-assigned managed identity provided by Microsoft Entra ID. 

When you enable this configuration, the system-assigned managed identities will be used for the following scenarios:
-	Storage Sync Service authentication to Azure file share
-	Registered server authentication to Azure file share
-	Registered server authentication to Storage Sync Service 

To learn more about the benefits of using managed identities, see [Managed identities for Azure resources](/entra/identity/managed-identities-azure-resources/overview).

To configure your Azure File Sync deployment to utilize system-assigned managed identities, follow the guidance in the subsequent sections.

## Prerequisites
# [Portal](#tab/azure-portal) 
- **Azure File Sync agent version 20.0.0.0 or later** must be installed on the registered server. 

- On your **storage accounts** used by Azure File Sync, you must be a **member of the Azure File Sync Administrator or Owner management role** or have “Microsoft.Authorization/roleassignments/write” permissions. 

# [PowerShell](#tab/azure-powershell) 
- **Azure File Sync agent version 20.0.0.0 or later** must be installed on the registered server. 

- On your **storage accounts** used by Azure File Sync, you must be a **member of the Azure File Sync Administrator or Owner management role** or have “Microsoft.Authorization/roleassignments/write” permissions. 

 
  
-	**Az.StorageSync [PowerShell module](https://www.powershellgallery.com/packages/Az.StorageSync) version 2.5.0 or later** must be installed on the machine that will be used to configure Azure File Sync to use managed identities. To install the latest Az.StorageSync PowerShell module, run the following command from an elevated PowerShell window:

    ```powershell
    Install-Module Az.StorageSync -Force
    ```
---
## Regional availability

Azure File Sync support for system-assigned managed identities  is available in [all Azure Public and Gov regions](https://azure.microsoft.com/global-infrastructure/locations/) that support Azure File Sync.

## Enable a system-assigned managed identity on your registered servers
Before you can configure Azure File Sync to use managed identities, your registered servers must have a system-assigned managed identity that will be used to authenticate to the Azure File Sync service and Azure file shares. 

To enable a system-assigned managed identity on a registered server that has the Azure File Sync v20 agent installed, perform the following steps:

- If the server is hosted outside of Azure, it must be an **Azure Arc-enabled server** to have a system-assigned managed identity. For more information on Azure Arc-enabled servers and how to install the Azure Connected Machine agent, see: [Azure Arc-enabled servers Overview](/azure/azure-arc/servers/overview).
- If the server is an Azure virtual machine, **enable the system-assigned managed identity setting on the VM**. For more information, see: [Configure managed identities on Azure virtual machines](/entra/identity/managed-identities-azure-resources/how-to-configure-managed-identities?pivots=qs-configure-portal-windows-vm#enable-system-assigned-managed-identity-on-an-existing-vm).

> [!NOTE]
> Once the Storage Sync Service is configured to use managed identities, registered servers that don't have a system-assigned managed identity will continue to use a shared key to authenticate to your Azure file shares.
 
### How to check if your registered servers have a system-assigned managed identity
# [Portal](#tab/azure-portal) 
To check if your registered servers have a system-assigned managed identity, perform the following steps using the Azure portal:  

1. Go to your **Storage Sync Service** in the Azure portal, expand **Settings** and select **Managed identity**. 

2. In the **Registered Servers** section, select the **Ready to use Managed ID** tile. This tile displays a list of servers that have a system-assigned managed identity. If your server isn't listed, perform the steps to [Enable a system-assigned managed identity on your registered servers]( #enable-a-system-assigned-managed-identity-on-your-registered-servers).  

# [PowerShell](#tab/azure-powershell) 
To check if your registered servers have a system-assigned managed identity, run the following PowerShell command:

```powershell
Get-AzStorageSyncServer -ResourceGroupName <string> -StorageSyncServiceName <string>
```

Verify the **LatestApplicationId** property has a GUID which indicates the server has a system-assigned managed identity but isn't currently configured to use the managed identity.  

If the value for the **ActiveAuthType** property is **Certificate** and the **LatestApplicationId** doesn't have a GUID, the server doesn't have a system-assigned managed identity and will use shared keys to authenticate to the Azure file share.

> [!NOTE]
> Once a server is configured to use the system-assigned managed identity by following the steps in the following section, the **LatestApplicationId** property is no longer used (will be empty), the **ActiveAuthType** property value will be changed to **ManagedIdentity**, and the **ApplicationId** property will have a GUID which is the system-assigned managed identity.

---
## Configure your Azure File Sync deployment to use system-assigned managed identities
# [Portal](#tab/azure-portal) 

To configure the Storage Sync Service and registered servers to use system-assigned managed identities, perform the following steps in the Azure portal: 

1. Go to your **Storage Sync Service** in the Azure portal, expand **Settings** and select **Managed identity**. 

2. Select **Turn on Managed identity** to begin setup. 

The following steps are performed and will take several minutes (or longer for large topologies) to complete: 

- Enables a system-assigned managed identity for Storage Sync Service resource. 

- Grants the Storage Sync Service system-assigned managed identity access to your Storage Accounts (Storage Account Contributor role). 

- Grants the Storage Sync Service system-assigned managed identity access to your Azure file shares (Storage File Data Privileged Contributor role). 

- Grants the registered server(s) system-assigned managed identity access to the Azure file shares (Storage File Data Privileged Contributor role). 

- Configures the Storage Sync Service to use system-assigned managed identity. 

- Configures registered server(s) to use system-assigned managed identity.  

> [!NOTE] 
> Once the registered server(s) are configured to use a system-assigned managed identity, it can take up to 15 minutes before the server uses the system-assigned managed identity to authenticate to the Storage Sync Service and file shares.


# [PowerShell](#tab/azure-powershell) 
To configure the Storage Sync Service and registered servers to use system-assigned managed identities, run the following command from an elevated PowerShell window:

```powershell
Set-AzStorageSyncServiceIdentity -ResourceGroupName <string> -StorageSyncServiceName <string> -Verbose
```

The **Set-AzStorageSyncServiceIdentity** cmdlet performs the following steps for you and will take several minutes (or longer for large topologies) to complete:
-	Enables a system-assigned managed identity for Storage Sync Service resource.
- Grants the Storage Sync Service system-assigned managed identity access to your Storage Accounts (Storage Account Contributor role).
-	Grants the Storage Sync Service system-assigned managed identity access to your Azure file shares (Storage File Data Privileged Contributor role).
-	Grants the registered server(s) system-assigned managed identity access to the Azure file shares (Storage File Data Privileged Contributor role).
-	Configures the Storage Sync Service to use system-assigned managed identity.
- Configures registered server(s) to use system-assigned managed identity. 

Use the **Set-AzStorageSyncServiceIdentity** cmdlet anytime you need to configure additional registered servers to use managed identities.

> [!NOTE]
> Once the registered server(s) are configured to use a system-assigned managed identity, it can take up to 15 minutes before the server uses the system-assigned managed identity to authenticate to the Storage Sync Service and file shares.

---
### How to check if the Storage Sync Service is using a system-assigned managed identity

# [Portal](#tab/azure-portal)  

To check if the Storage Sync Service is using a system-assigned managed identity, perform the following steps in the Azure portal: 

1. Go to your **Storage Sync Service** in the Azure portal, expand **Settings**, and select **Managed identity**. 

2. In the **Registered Servers** section, if you have at least one server listed in the **Using Managed ID** tile, your service is configured to use managed identities.  

# [PowerShell](#tab/azure-powershell) 
To check if the Storage Sync Service is using a system-assigned managed identity, run the following command from an elevated PowerShell window:

```powershell
Get-AzStorageSyncService -ResourceGroupName <string> -StorageSyncServiceName <string>
```

Verify the value for the **UseIdentity** property is **True**. If the value is **False**, the Storage Sync Service is using shared keys to authenticate to the Azure file shares.

---
### How to check if a registered server is configured to use a system-assigned managed identity

# [Portal](#tab/azure-portal) 

To check if a registered server is configured to use a system-assigned managed identity, perform the following steps in the Azure portal: 

1. Go to your **Storage Sync Service** in the Azure portal, expand **Settings**, and select **Managed identity**. 

2. In the **Registered Servers** section, select the **Using Managed ID** tile and verify the server is listed. 

# [PowerShell](#tab/azure-powershell) 
To check if a registered server is configured to use a system-assigned managed identity, run the following command from an elevated PowerShell window:

```powershell
Get-AzStorageSyncServer -ResourceGroupName <string> -StorageSyncServiceName <string>
```

Verify the **ApplicationId** property has a GUID which indicates the server is configured to use the managed identity. The value for the **ActiveAuthType** property will be updated to **ManagedIdentity** once the server is using the system-assigned managed identity.

> [!NOTE]
> Once the registered server(s) are configured to use a system-assigned managed identity, it can take up to 15 minutes before the server uses the system-assigned managed identity to authenticate to the Storage Sync Service and Azure file shares.

---
## More information
Once the Storage Sync Service and registered server(s) are configured to use a system-assigned managed identity:
  - New endpoints (cloud or server) that are created will use a system-assigned managed identity to authenticate to the Azure file share.
  - When you need to configure additional registered servers to use managed identities, go to the Managed identity blade in the portal and select **Turn on Managed identity**, or use the `Set-AzStorageSyncServiceIdentity` PowerShell cmdlet.

If you experience issues, see: [Troubleshoot Azure File Sync managed identity issues](/troubleshoot/azure/azure-storage/files/file-sync/file-sync-troubleshoot-managed-identities).
