---
title: Copy or move data to Azure Storage by using AzCopy v10
description: AzCopy is a command-line utility that you can use to copy data to, from, or between storage accounts. This article helps you download AzCopy, connect to your storage account, and then transfer data.
author: normesta
ms.service: azure-storage
ms.topic: how-to
ms.date: 10/02/2025
ms.author: normesta
ms.subservice: storage-common-concepts
ms.custom: ai-video-demo
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to use AzCopy to transfer data to and from Azure Storage, so that I can efficiently manage storage operations across my cloud environment.
---

# Get started with AzCopy

AzCopy is a command-line utility that you can use to copy data to, from, or between storage accounts. This article helps you download AzCopy, connect to your storage account, and then transfer data.<br><br>

> [!VIDEO 4238a2be-881a-4aaa-8ccd-07a6557a05ef]

AzCopy **V10** is the currently supported version of AzCopy and it is supported only on versions of Windows, Linux, or macOS that are officially maintained. If you need to use a previous version of AzCopy, see the [Use the previous version of AzCopy](#previous-version) section of this article.

## AzCopy use cases

Use AzCopy to accomplish these tasks:

> [!div class="checklist"]
> - Download data from an on-premises source
> - Upload data to an on-premises source
> - Copy data between accounts
> - Synchronize data with an on-premises source or with another account
> - Copy data from Amazon S3 or Google Cloud Storage buckets
> - Modify properties and metadata
> - Create new containers or file shares, list accounts, and delete objects
> - Manage AzCopy jobs
> - Run performance benchmark tests

The following table shows the all supported sources and destinations for transferring data.

| Source | Destination |
|---|---|
| On-premises | Azure Blob Storage containers |
| On-premises | SMB Azure file shares |
| On-premises | NFS Azure file shares |
| On-premises | Azure Data Lake Storage containers |
| Azure Blob Storage containers | Azure Blob Storage containers |
| Data Lake Storage containers | Data Lake Storage containers |
| SMB Azure file shares | SMB Azure file shares |
| NFS Azure file shares | NFS Azure file shares |
| Azure Blob containers | Data Lake Storage containers |
| Data Lake Storage containers | Azure Blob containers |
| Azure Blob containers | SMB Azure file shares |
| SMB Azure file shares | Azure Blob containers |
| AWS S3 | Azure Block Blob |
| Google Cloud Storage | Azure Block Blob |

[!INCLUDE [storage-azcopy-change-support](includes/storage-azcopy-change-support.md)]

<a id="download-and-install-azcopy"></a>
<a id="run-azcopy"></a>

## Get AzCopy

If you are using AzCopy on a linux machine, you can use a package manager. For all other operating systems, download a portable binary file. For detailed information on AzCopy releases, see the [AzCopy release page](https://github.com/Azure/azure-storage-azcopy/releases).

### Use a package manager (Linux only)

Installing AzCopy through your Linux distribution's package manager is the most convenient and maintainable way to get this tool. Package manager installation includes automatic dependency resolution, simplified updates, and integration with your system's software management. For step-by-step guidance, see [Install AzCopy on Linux by using a package manager](storage-use-azcopy-install-linux-package.md).

<a id="download-azcopy"></a>

### Download a portable binary

An installation package is available only for Linux. For all other operating systems, you can download the AzCopy V10 executable file to any directory on your computer.

- [Windows 64-bit](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Windows 32-bit](https://aka.ms/downloadazcopy-v10-windows-32bit) (zip)
- [Windows ARM64](https://aka.ms/downloadazcopy-v10-windows-arm64) (zip)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [Linux ARM64](https://aka.ms/downloadazcopy-v10-linux-arm64) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (zip)
- [macOS ARM64](https://aka.ms/downloadazcopy-v10-mac-arm64) (zip)

These files are compressed as a zip file (Windows and Mac) or a tar file (Linux). To download and decompress the tar file on Linux, see the documentation for your Linux distribution.

> [!NOTE]
> If you want to copy data to and from your [Azure Table storage](../tables/table-storage-overview.md) service, then install [AzCopy version 7.3](/previous-versions/azure/storage/storage-use-azcopy#azcopy-with-table-support-v73).

### Add AzCopy to your system path

For convenience, consider adding the directory location of the AzCopy executable to your system path for ease of use. That way you can type `azcopy` from any directory on your system. If you choose not to add the AzCopy directory to your path, you'll have to change directories to the location of your AzCopy executable and type `azcopy` or `.\azcopy` in a command shell.

<a id="choose-how-youll-provide-authorization-credentials"></a>

## Authorize AzCopy

As an owner of your Azure Storage account, you aren't automatically assigned permissions to access data. Before you can do anything meaningful with AzCopy, you need to decide how you'll provide authorization credentials to the storage service. You can provide credentials by using Microsoft Entra ID or by using a SAS token.

<a name='option-1-use-azure-active-directory'></a>

### Authorize with Microsoft Entra ID

By using Microsoft Entra ID, you can provide credentials once instead of having to append a SAS token to each command. Start by choosing what type of *security principal* you want to authorize. Use the following table as a guide.

| Security principal | Guidance |
|-----|-----|
| User identity | [Authorize access for AzCopy with a user identity](storage-use-azcopy-authorize-managed-identity.md) |
| Managed identity | [Authorize access for AzCopy with a managed identity](storage-use-azcopy-authorize-managed-identity.md) |
| Service principal | [Authorize access for AzCopy with a service principal](storage-use-azcopy-authorize-service-principal.md) |

<a id="option-2-use-a-sas-token"></a>

### Authorize with a SAS token

You can append a SAS token to each source or destination URL that use in your AzCopy commands. This example command recursively copies data from a local directory to a blob container. A fictitious SAS token is appended to the end of the container URL.

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

To learn more about SAS tokens and how to obtain one, see [Using shared access signatures (SAS)](./storage-sas-overview.md).

<a id="transfer-data"></a>

## Transfer data

After you've authorized your identity or obtained a SAS token, you can begin transferring data.

To find example commands, see any of these articles.

| Service | Article |
|--------|-----------|
|Azure Blob Storage|[Upload files to Azure Blob Storage](storage-use-azcopy-blobs-upload.md) |
|Azure Blob Storage|[Download blobs from Azure Blob Storage](storage-use-azcopy-blobs-download.md)|
|Azure Blob Storage|[Copy blobs between Azure storage accounts](storage-use-azcopy-blobs-copy.md)|
|Azure Blob Storage|[Synchronize with Azure Blob Storage](storage-use-azcopy-blobs-synchronize.md)|
|Azure Files |[Transfer data with AzCopy and file storage](storage-use-azcopy-files.md)|
|Amazon S3|[Copy data from Amazon S3 to Azure Storage](storage-use-azcopy-s3.md)|
|Google Cloud Storage|[Copy data from Google Cloud Storage to Azure Storage (preview)](storage-use-azcopy-google-cloud.md)|
|Azure Stack storage|[Transfer data with AzCopy and Azure Stack storage](/azure-stack/user/azure-stack-storage-transfer#azcopy)|

> [!TIP]
> While you can run AzCopy commands interactively, you can also incorporate AzCopy into automated scripts for batch operations, scheduled data transfers, or continuous integration pipelines. For more information, see [Use AzCopy in scripts and scheduled transfers](storage-use-azcopy-in-script.md).

## Get command help

To see a list of commands, type `azcopy -h` and then press the ENTER key.

To learn about a specific command, just include the name of the command (For example: `azcopy list -h`).

> [!div class="mx-imgBorder"]
> ![Inline help](media/storage-use-azcopy-v10/azcopy-inline-help.png)

### List of commands

The following table lists all AzCopy v10 commands. Each command links to a reference article.

|Command|Description|
|---|---|
|[azcopy bench](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_bench)|Runs a performance benchmark by uploading or downloading test data to or from a specified location.|
|[azcopy copy](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_copy)|Copies source data to a destination location|
|[azcopy doc](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_doc)|Generates documentation for the tool in Markdown format.|
|[azcopy env](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_env)|Shows the environment variables that can configure AzCopy's behavior.|
|[azcopy jobs](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs)|Subcommands related to managing jobs.|
|[azcopy jobs clean](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs_clean)|Remove all log and plan files for all jobs.|
|[azcopy jobs list](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs_list)|Displays information on all jobs.|
|[azcopy jobs remove](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs_remove)|Remove all files associated with the given job ID.|
|[azcopy jobs resume](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs_resume)|Resumes the existing job with the given job ID. |
|[azcopy jobs show](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_jobs_show)|Shows detailed information for the given job ID.|
|[azcopy list](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_list)|Lists the entities in a given resource.|
|[azcopy login](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_login)|Logs in to Microsoft Entra ID to access Azure Storage resources.|
|[azcopy login status](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_login_status)|Lists the entities in a given resource.|
|[azcopy logout](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_logout)|Logs the user out and terminates access to Azure Storage resources.|
|[azcopy make](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_make)|Creates a container or file share.|
|[azcopy remove](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_remove)|Delete blobs or files from an Azure storage account.|
|[azcopy sync](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_sync)|Replicates the source location to the destination location.|
|[azcopy set-properties](https://github.com/Azure/azure-storage-azcopy/wiki/azcopy_set-properties)|Change the access tier of one or more blobs and replace (overwrite) the metadata, and index tags of one or more blobs.|

> [!NOTE]
> AzCopy does not have a command to rename files.

## Configure, optimize, and fix

See any of the following resources:

- [AzCopy configuration settings](storage-ref-azcopy-configuration-settings.md)

- [Optimize the performance of AzCopy](storage-use-azcopy-optimize.md)

- [Find errors and resume jobs by using log and plan files in AzCopy](storage-use-azcopy-configure.md)

- [Troubleshoot problems with AzCopy v10](storage-use-azcopy-troubleshoot.md)

<a id="previous-version"></a>

## Use a previous version (deprecated)

If you need to use the previous version of AzCopy, see either of the following links:

- [AzCopy on Windows (v8)](/previous-versions/azure/storage/storage-use-azcopy)

- [AzCopy on Linux (v7)](/previous-versions/azure/storage/storage-use-azcopy-linux)

> [!NOTE]
> These versions AzCopy are been deprecated. Microsoft recommends using AzCopy v10.

## Next steps

If you have questions, issues, or general feedback, submit them [on GitHub](https://github.com/Azure/azure-storage-azcopy) page.
