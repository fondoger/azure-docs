---
title: Mount an NFS Azure file share on Linux
description: Learn how to mount a Network File System (NFS) Azure file share on Linux, including configuring network security and mount options.
author: khdownie
ms.service: azure-file-storage
ms.custom: linux-related-content, references_regions
ms.topic: how-to
ms.date: 10/14/2024
ms.author: kendownie
---

# Mount NFS Azure file shares on Linux

Azure file shares can be mounted in Linux distributions using either the Server Message Block (SMB) protocol or the Network File System (NFS) protocol. This article is focused on mounting with NFS. For details on mounting SMB Azure file shares, see [Use Azure Files with Linux](storage-how-to-use-files-linux.md). For details on each of the available protocols, see [Azure file share protocols](storage-files-planning.md#available-protocols).

## Applies to

| File share type | SMB | NFS |
|-|:-:|:-:|
| Standard file shares (GPv2), LRS/ZRS | ![No](../media/icons/no-icon.png) | ![No](../media/icons/no-icon.png) |
| Standard file shares (GPv2), GRS/GZRS | ![No](../media/icons/no-icon.png) | ![No](../media/icons/no-icon.png) |
| Premium file shares (FileStorage), LRS/ZRS | ![No](../media/icons/no-icon.png) | ![Yes](../media/icons/yes-icon.png) |

## Support

[!INCLUDE [files-nfs-limitations](../../../includes/files-nfs-limitations.md)]

## Regional availability

NFS Azure file shares are supported in all the same regions that support premium file storage. See [Azure products available by region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/?products=storage&regions=all).

## Step 1: Create an NFS Azure file share

If you haven't already done so, [create an NFS Azure file share](storage-files-how-to-create-nfs-shares.md).

## Step 2: Configure network security

NFS shares can only be accessed from trusted networks. Currently, the only way to secure the data in your storage account is by using a virtual network and other network security settings. Any other tools used to secure data, including account key authorization, Microsoft Entra security, and access control lists (ACLs) can't be used to authorize an NFSv4.1 request.

> [!IMPORTANT]
> The NFSv4.1 protocol runs on port 2049. If you're connecting from an on-premises network, make sure that your client allows outgoing communication through port 2049. If you've granted access to specific VNets, make sure that any network security groups associated with those VNets don't contain security rules that block incoming communication through port 2049.

### Create a private endpoint or service endpoint

To use NFS Azure file shares, you must either [create a private endpoint](storage-files-networking-endpoints.md#create-a-private-endpoint) (recommended) or [restrict access to your public endpoint](storage-files-networking-endpoints.md#restrict-public-endpoint-access).

### Disable secure transfer

Azure Files doesn't currently support encryption-in-transit with the NFS protocol and relies instead on network-level security. Therefore, you'll need to disable secure transfer on your storage account.

1. Sign in to the [Azure portal](https://portal.azure.com/) and access the storage account containing the NFS share you created.
1. Select **Configuration**.
1. Select **Disabled** for **Secure transfer required**.
1. Select **Save**.

    :::image type="content" source="media/storage-files-how-to-mount-nfs-shares/disable-secure-transfer.png" alt-text="Screenshot of storage account configuration screen with secure transfer disabled." lightbox="media/storage-files-how-to-mount-nfs-shares/disable-secure-transfer.png":::

### Enable hybrid access through VPN or ExpressRoute (optional)

To enable hybrid access to an NFS Azure file share, use one of the following networking solutions:

- [Configure a Point-to-Site (P2S) VPN](storage-files-configure-p2s-vpn-linux.md).
- [Configure a Site-to-Site (S2S) VPN](storage-files-configure-s2s-vpn.md).
- Configure [ExpressRoute](../../expressroute/expressroute-introduction.md).

## Step 3: Mount an NFS Azure file share

You can mount the share using the Azure portal. You can also create a record in the **/etc/fstab** file to automatically mount the share every time the Linux server or VM boots.

### Mount an NFS share using the Azure portal

You can use the `nconnect` Linux mount option to improve performance for NFS Azure file shares at scale. For more information, see [Improve NFS Azure file share performance](nfs-performance.md#nconnect).

1. Once the file share is created, select the share and select **Connect from Linux**.
1. Enter the mount path you'd like to use, then copy the script.
1. Connect to your client and use the provided mounting script. Only the required mount options are included in the script, but you can add other [recommended mount options](#mount-options).

    :::image type="content" source="media/storage-files-how-to-create-mount-nfs-shares/mount-nfs-file-share-script.png" alt-text="Screenshot of file share connect blade.":::

You have now mounted your NFS share.

### Mount an NFS share using /etc/fstab

If you want the NFS file share to automatically mount every time the Linux server or VM boots, create a record in the **/etc/fstab** file for your Azure file share. Replace `YourStorageAccountName` and `FileShareName` with your information.

```bash
<YourStorageAccountName>.file.core.windows.net:/<YourStorageAccountName>/<FileShareName> /media/<YourStorageAccountName>/<FileShareName> nfs vers=4,minorversion=1,_netdev,nofail,sec=sys 0 0
```

For more information, enter the command `man fstab` from the Linux command line.

### Mount options

The following mount options are recommended or required when mounting NFS Azure file shares.

| **Mount option** | **Recommended value** | **Description** |
|******************|***********************|*****************|
| `vers` | 4 | Required. Specifies which version of the NFS protocol to use. Azure Files only supports NFSv4.1. |
| `minorversion` | 1 | Required. Specifies the minor version of the NFS protocol. Some Linux distros don't recognize minor versions on the `vers` parameter. So instead of `vers=4.1`, use `vers=4,minorversion=1`. |
| `sec` | sys | Required. Specifies the type of security to use when authenticating an NFS connection. Setting `sec=sys` uses the local UNIX UIDs and GIDs that use AUTH_SYS to authenticate NFS operations. |
| `rsize` | 1048576 | Recommended. Sets the maximum number of bytes to be transferred in a single NFS read operation. Specifying the maximum level of 1048576 bytes will usually result in the best performance. |
| `wsize` | 1048576 | Recommended. Sets the maximum number of bytes to be transferred in a single NFS write operation. Specifying the maximum level of 1048576 bytes will usually result in the best performance. |
| `noresvport` | n/a | Recommended. Tells the NFS client to use a non-privileged source port when communicating with an NFS server for the mount point. Using the `noresvport` mount option helps ensure that your NFS share has uninterrupted availability after a reconnection. Using this option is strongly recommended for achieving high availability. |
| `actimeo` | 30-60 | Recommended. Specifying `actimeo` sets all of `acregmin`, `acregmax`, `acdirmin`, and `acdirmax` to the same value. Using a value lower than 30 seconds can cause performance degradation because attribute caches for files and directories expire too quickly. We recommend setting `actimeo` between 30 and 60 seconds. |

## Step 4: Validate connectivity

If your mount fails, it's possible that your private endpoint wasn't set up correctly or isn't accessible. For details on confirming connectivity, see [Verify connectivity](storage-files-networking-endpoints.md#verify-connectivity).

## NFS file share snapshots

Customers using NFS Azure file shares can create, list, and delete NFS Azure file share snapshots. This capability allows users to roll back entire file systems or recover files that were accidentally deleted or corrupted.

> [!IMPORTANT]
> You should mount your file share before creating snapshots. If you create a new NFS file share and take snapshots before mounting the share, attempting to list the snapshots for the share will return an empty list. We recommend deleting any snapshots taken before the first mount and re-creating them after you've mounted the share.

### Limitations

Only file management APIs (`AzRmStorageShare`) are supported for NFS Azure file share snapshots. File data plane APIs (`AzStorageShare`) aren't supported.

Azure Backup isn't currently supported for NFS file shares.

AzCopy isn't currently supported for NFS file shares. To copy data from an NFS Azure file share or share snapshot, use file system copy tools such as rsync or fpsync.

NFS Azure file share snapshots are available in all Azure public cloud regions.

### Create a snapshot

You can create a snapshot of an NFS Azure file share using the Azure portal, Azure PowerShell, or Azure CLI. A share can support the creation of up to 200 share snapshots.

# [Azure portal](#tab/portal)

To create a snapshot of an existing file share, sign in to the Azure portal and follow these steps.

1. In the search box at the top of the Azure portal, type and select *storage accounts*.

1. Select the FileStorage storage account that contains the NFS Azure file share that you want to take a snapshot of.

1. Select **Data storage** > **File shares**.

1. Select the file share that you want to snapshot, then select **Operations** > **Snapshots**.

1. Select **+ Add snapshot**. Add an optional comment, and select **OK**.

  :::image type="content" source="media/storage-files-how-to-mount-nfs-shares/add-file-share-snapshot.png" alt-text="Screenshot of adding a file share snapshot.":::

# [Azure PowerShell](#tab/powershell)

To create a snapshot of an existing file share, run the following PowerShell command. Replace `<resource-group-name>`, `<storage-account-name>`, and `<file-share-name>` with your own values.

```azurepowershell
New-AzRmStorageShare -ResourceGroupName "<resource-group-name>" -StorageAccountName "<storage-account-name>" -Name "<file-share-name>" -Snapshot
```

# [Azure CLI](#tab/cli)
To create a snapshot of an existing file share, run the following Azure CLI command. Replace `<file-share-name>` and `<storage-account-name>` with your own values.

```azurecli
az storage share snapshot --name <file-share-name> --account-name <storage-account-name>
```
---

### List file share snapshots

You can list all the snapshots for a file share using the Azure portal, Azure PowerShell, or Azure CLI.

# [Azure portal](#tab/portal)

To list all the snapshots for an existing file share, sign in to the Azure portal and follow these steps.

1. In the search box at the top of the Azure portal, type and select *storage accounts*.

1. Select the FileStorage storage account that contains the NFS Azure file share that you want to list the snapshots of.

1. Select **Data storage** > **File shares**.

1. Select the file share for which you want to list the snapshots.

1. Select **Operations** > **Snapshots**, and any existing snapshots for the file share will be listed.

# [Azure PowerShell](#tab/powershell)

To list all file shares and snapshots in a storage account, run the following PowerShell command. Replace `<resource-group-name>` and `<storage-account-name>` with your own values.

```azurepowershell
Get-AzRmStorageShare -ResourceGroupName "<resource-group-name>" -StorageAccountName "<storage-account-name>" -IncludeSnapshot
```

# [Azure CLI](#tab/cli)
To list all file shares and snapshots in a storage account, run the following Azure CLI command. Replace `<storage-account-name>` with your own value.

```azurecli
az storage share list --account-name <storage-account-name> --include-snapshots
```
---

### Delete snapshots

Existing share snapshots are never overwritten. They must be deleted explicitly. You can delete share snapshots using the Azure portal, Azure PowerShell, or Azure CLI.

# [Azure portal](#tab/portal)

To delete a snapshot of an existing file share, sign in to the Azure portal and follow these steps.

1. In the search box at the top of the Azure portal, type and select *storage accounts*.

1. Select the FileStorage storage account that contains the NFS Azure file share for which you want to delete snapshots.

1. Select **Data storage** > **File shares**.

1. Select the file share for which you want to delete one or more snapshots, then select **Operations** > **Snapshots**. Any existing snapshots for the file share will be listed.

1. Select the snapshot(s) that you want to delete, and then select **Delete**.

  :::image type="content" source="media/storage-files-how-to-mount-nfs-shares/delete-file-share-snapshot.png" alt-text="Screenshot of deleting file share snapshots.":::

# [Azure PowerShell](#tab/powershell)

To delete a file share snapshot, run the following PowerShell command. Replace `<resource-group-name>`, `<storage-account-name>`, and `<file-share-name>` with your own values. The `SnapshotTime` parameter must follow the correct name format, such as `2021-05-10T08:04:08Z`.

```azurepowershell
Remove-AzRmStorageShare -ResourceGroupName "<resource-group-name>" -StorageAccountName "<storage-account-name>" -Name "<file-share-name>" -SnapshotTime "<snapshot-time>"
```

To delete a file share and all its snapshots, run the following PowerShell command. Replace `<resource-group-name>`, `<storage-account-name>`, and `<file-share-name>` with your own values.

```azurepowershell
Remove-AzRmStorageShare "<resource-group-name>" -StorageAccountName "<storage-account-name>" -Name "<file-share-name>" -Include Snapshots
```

# [Azure CLI](#tab/cli)

To delete a file share snapshot, run the following Azure CLI command. Replace `<storage-account-name>` and `<file-share-name>` with your own values. The `--snapshot` parameter must follow the correct name format, such as `2021-05-10T08:04:08Z`.

```azurecli
az storage share delete --account-name <storage-account-name> --name <file-share-name> --snapshot <snapshot-time>
```

To delete a file share and all its snapshots, run the following Azure CLI command. Replace `<storage-account-name>` and `<file-share-name>` with your own values.

```azurecli
az storage share delete --account-name <storage-account-name> --name <file-share-name> --delete-snapshots include
```
---

### Mount an NFS Azure file share snapshot

To mount an NFS Azure file share snapshot to a Linux VM (NFS client) and restore files, follow these steps.

1. Run the following command in a console. See [Mount options](#mount-options) for other recommended mount options. To improve copy performance, mount the snapshot with [nconnect](nfs-performance.md#nconnect) to use multiple TCP channels.
   
   ```bash
   sudo mount -o vers=4,minorversion=1,proto=tcp,sec=sys $server:/nfs4account/share /media/nfs
   ```
   
1. Change the directory to `/media/nfs/.snapshots` so you can view the available snapshots. The `.snapshots` directory is hidden by default, but you can access and read from it like any directory.
   
   ```bash
   cd /media/nfs/.snapshots
   ```
   
1. List the contents of the `.snapshots` folder.
   
   ```bash
   ls
   ```
   
1. Each snapshot has its own directory that serves as a recovery point. Change to the snapshot directory for which you want to restore files.
   
   ```bash
   cd <snapshot-name>
   ```
   
1. List the contents of the directory to view a list of files and directories that can be recovered.
   
   ```bash
   ls
   ```
   
1. Copy all files and directories from the snapshot to a *restore* directory to complete the restore.
   
   ```bash
   cp -r <snapshot-name> ../restore
   ```
   
The files and directories from the snapshot should now be available in the `/media/nfs/restore` directory.

## Next step

- If you experience any issues, see [Troubleshoot NFS Azure file shares](/troubleshoot/azure/azure-storage/files-troubleshoot-linux-nfs?toc=/azure/storage/files/toc.json).