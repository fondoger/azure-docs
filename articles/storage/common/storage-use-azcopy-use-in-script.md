---
title: Use AzCopy v10 in a script
description: Learn how to obtain a static link to a specific AzCopy version and use that link in your scripts.
author: normesta
ms.service: azure-storage
ms.topic: how-to
ms.date: 09/01/2025
ms.author: normesta
ms.subservice: storage-common-concepts
ms.custom: ai-video-demo
ai-usage: ai-assisted
# Customer intent: As a developer or system administrator, I want to incorporate AzCopy into automated scripts and scheduled tasks, so that I can reliably transfer data to and from Azure Storage without manual intervention.
---

# Use AzCopy v10 in a script

AzCopy is a command-line utility that you can use to copy blobs or files to or from Azure Storage accounts. While you can run AzCopy commands interactively, you'll often want to incorporate AzCopy into automated scripts for batch operations, scheduled data transfers, or continuous integration pipelines.

This article shows you how to use AzCopy effectively in scripts by obtaining static download links to ensure version consistency, creating scheduled tasks for automated data transfers, and handling special considerations like character escaping and Jenkins integration.

## Obtain a static download link

Over time, the AzCopy [download link](#download-and-install-azcopy) will point to new versions of AzCopy. If your script downloads AzCopy, the script might stop working if a newer version of AzCopy modifies features that your script depends upon.

To avoid these issues, obtain a static (unchanging) link to the current version of AzCopy. That way, your script downloads the same exact version of AzCopy each time that it runs.

> [!NOTE]
> The static link to AzCopy binaries is subject to change over time due to our content delivery infrastructure. If you must use a specific version of AzCopy for any reason, we recommend using AzCopy with an operating system that leverages a [published package](#install-azcopy-on-linux-by-using-a-package-manager). This method ensures that you can reliably install and maintain the desired version of AzCopy.

### [Linux](#tab/linux)

To obtain the link, run this command:

```bash
curl -s -D- https://aka.ms/downloadazcopy-v10-linux \| grep ^Location
```

The URL appears in the output of this command. Your script can then download AzCopy by using that URL.

You can also use the `--strip-components=1` on the `tar` command to remove the top-level folder that contains the version name, and instead extract the binary directly into the current folder. This allows the script to be updated with a new version of `azcopy` by only updating the `wget` URL.

```bash
wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1
```

### [Windows](#tab/windows)

To obtain the link, run one of these commands:

For Windows PowerShell run this command,To obtain the link, run this command:

```powershell
(Invoke-WebRequest -Uri https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction SilentlyContinue).headers.location
```

For PowerShell 6.1+, run this command:

```powershell
(Invoke-WebRequest -Uri https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction SilentlyContinue -SkipHttpErrorCheck).headers.location
```

The URL appears in the output of this command. Your script can then download AzCopy by using that URL.

For Windows PowerShell run this command,To obtain the link, run this command:

```PowerShell
Invoke-WebRequest -Uri <URL from the previous command> -OutFile 'azcopyv10.zip'
Expand-archive -Path '.\azcopyv10.zip' -Destinationpath '.\'
$AzCopy = (Get-ChildItem -path '.\' -Recurse -File -Filter 'azcopy.exe').FullName
# Invoke AzCopy 
& $AzCopy
```

For PowerShell 6.1+, run this command:

```PowerShell
Invoke-WebRequest -Uri <URL from the previous command> -OutFile 'azcopyv10.zip'
$AzCopy = (Expand-archive -Path '.\azcopyv10.zip' -Destinationpath '.\' -PassThru | where-object {$_.Name -eq 'azcopy.exe'}).FullName
# Invoke AzCopy
& $AzCopy
``` 

---

## Create a scheduled task

You can create a scheduled task or cron job that runs an AzCopy command script. The script identifies and uploads new on-premises data to cloud storage at a specific time interval.

The following examples assume that you have configured Microsoft Entra authentication by using the `AZCOPY_AUTO_LOGIN_TYPE` environment variable. To learn more, see [Authorize with Microsoft Entra ID](storage-use-azcopy-v10#authorize-with-microsoft-entra-id).

### [Linux](#tab/linux)

Copy the following AzCopy command to a text editor. Update the parameter values of the AzCopy command to the appropriate values. Save the file as `script.sh`.

```bash
azcopy sync "/mnt/myfiles" "https://mystorageaccount.blob.core.windows.net/mycontainer" --recursive=true
```

You can create a cron job by using the [Crontab](http://crontab.org/) command. The following example creates a cron job and specifies the cron expression `*/5 * * * *`  which indicates that the shell script `script.sh` should run every five minutes.

```bash
crontab -e
*/5 * * * * sh /path/to/script.sh
```

You can schedule the script to run at a specific time daily, monthly, or yearly. To learn more about setting the date and time for job execution, see [cron expressions](https://en.wikipedia.org/wiki/Cron#CRON_expression).

### [Windows](#tab/windows)

Copy the following AzCopy command to a text editor. Update the parameter values this command to the appropriate values. Save the file as `script.bat`.

```bash
azcopy sync "C:\myFolder" "https://mystorageaccount.blob.core.windows.net/mycontainer" --recursive=true
```

You can schedule a task by using the [Schtasks](/windows/win32/taskschd/schtasks) command.

To create a scheduled task on Windows, enter the following command at a command prompt or in PowerShell:

This example assumes that your script is located in the root drive of your computer, but your script can be anywhere that you want.

```cmd
schtasks /CREATE /SC minute /MO 5 /TN "AzCopy Script" /TR C:\script.bat
```

The command uses:
- The `/SC` parameter to specify a minute schedule.
- The `/MO` parameter to specify an interval of five minutes.
- The `/TN` parameter to specify the task name.
- The `/TR` parameter to specify the path to the `script.bat` file.

To learn more about creating a scheduled task on
Windows, see [Schtasks](/previous-versions/orphan-topics/ws.10/cc772785(v=ws.10)#BKMK_minutes).

---

## Escape special characters in SAS tokens

In batch files that have the `.cmd` extension, you'll have to escape the `%` characters that appear in SAS tokens. You can do that by adding an extra `%` character next to existing `%` characters in the SAS token string. The resulting character sequence appears as `%%`. Make sure to add an extra `^` before each `&` character to create the character sequence `^&`.

## Run scripts by using Jenkins

If you plan to use [Jenkins](https://jenkins.io/) to run scripts, make sure to place the following command at the beginning of the script.

```
/usr/bin/keyctl new_session
```

## Next steps

If you have questions, issues, or general feedback, submit them [on GitHub](https://github.com/Azure/azure-storage-azcopy) page.
