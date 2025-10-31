---
title: Run Apache Hive queries using PowerShell on Entra enabled HDInsight Cluster
description: Learn how to run Apache Hive queries using PowerShell on Entra enabled HDInsight Cluster
ms.service: azure-hdinsight
ms.topic: how-to
author: apurbasroy
ms.author: apsinhar
ms.reviewer: nijelsf
ms.date: 08/20/2025
---

# Run Apache Hive queries using PowerShell on Entra ID-enabled HDInsight clusters

Apache Hive provides a powerful SQL-like interface for querying and analyzing data stored in Azure HDInsight. With Microsoft Entra ID-enabled HDInsight clusters, you can securely authenticate and run Hive queries using your organizational identity, ensuring centralized access control and compliance.

This guide walks you through how to connect to an Entra-enabled HDInsight cluster from PowerShell, authenticate with Entra ID, and run Hive queries to analyze your data. By leveraging PowerShell, you can automate Hive operations, integrate with scripts, and manage workloads more efficiently.

> [!NOTE]  
>This document does not provide a detailed description of what the HiveQL statements that are used in the examples do. For information on the HiveQL that is used in this example, see [Use Apache Hive with Apache Hadoop on HDInsight](../hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md).

 ## Prerequisites

- An Entra enabled Apache Hadoop cluster on HDInsight. See [Get Started with HDInsight on Linux](../hadoop/apache-hadoop-linux-tutorial-get-started.md).
- The PowerShell [Az Module](/powershell/azure/) installed.

## Run a Hive query
- Azure PowerShell provides *cmdlets* that allow you to remotely run Hive queries on HDInsight. Internally, the cmdlets make REST calls to [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) on the HDInsight cluster.

The following cmdlets are used when running Hive queries in a remote HDInsight cluster:
-  `Connect-AzAccount`: Authenticates Azure PowerShell to your Azure subscription.
- `New-AzHDInsightHiveJobDefinition`: Creates a *job definition* by using the specified HiveQL statements.
- `Start-AzHDInsightJob`: Sends the job definition to HDInsight and starts the job. A *job* object is returned.
- `Wait-AzHDInsightJob`: Uses the job object to check the status of the job. It waits until the job completes, or the wait time is exceeded.
- `Get-AzHDInsightJobOutput`: Used to retrieve the output of the job.
- `Invoke-AzHDInsightHiveJob`: Used to run HiveQL statements. This cmdlet blocks the query completes, then returns the results.
- `Use-AzHDInsightCluster`: Sets the current cluster to use for the `Invoke-AzHDInsightHiveJob` command.

## Setup (Secure Bearer Access Token)

Bearer Token is needed to send the cURL or any REST communication. You can follow the below mentioned step to get the token:

Execute an HTTP GET request to the OAuth 2.0 token endpoint with the following specifications:

### URL

```bash
https://login.microsoftonline.com/{Tenant_ID}/oauth2/v2.0/token
```

### Body

| Parameter | Description | Required |
| --- | --- | --- |
| grant_type    | Must be set to "client_credentials"                 | Yes |
| client_id     | Application (client) ID from Entra App registration | Yes |
| client_secret | Generated client secret or certificate              | Yes |
| scope         | Resource URL with .default suffix                   | Yes |


### cURL Request

```bash
curl --request GET \
  --url https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token \
  --header 'Content-Type: multipart/form-data' \
  --form grant_type=client_credentials \
  --form client_id={app_id} \
  --form client_secret={client_secret} \
  --form scope=https://{clustername}.clusteraccess.azurehdinsight.net/.default \
```

### Response

```powershell
{
	"token_type": "Bearer",
	"expires_in": 3599,
	"ext_expires_in": 3599,
	"access_token": "eyJ0eXAiOiJKV1iLCJub25jZSI6IkhaZ3lqQ2MxSkxzaXRSbmxzT1FTSHV0bEtBeXhhMU1JTzdyWmluLWF6LUEiLCJhbGciOiJSUzI1NiIsIng1dCI6ImltaTBZMnowZFlLeEJ0dEFxS19UdDVoWUJUayIsImtpZCI6ImltaTBZMnowZFlLeEJ0dEFxS19UdDVoWUJUayJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8wY2QzZGY5OS1lMDJmLTRmZDgtYTdkOC0zYjE5ZWVhZGFiYTUvIiwiaWF0IjoxNzQxMjgzMzUzLCJuYmYiOjE3NDEyODMzNTMsImV4cCI6MTc0MTI4NzI1MywiYWlvIjoiazJSZ1lIRDF1U1R4NGx2bjdmMTdGcXlkZUdwWlBnQT0iLCJhcHBfZGlzcGxheW5hbWUiOiJBenVyZSBIREkgTVNGVCBDbGllbnQiLCJhcHBpZCI6IjAzZDNiNTg5LWFjM2MtNDE4NC1iY2EyLTQ3ZWRiN2Q2ZmVjNiIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzBjZDNkZjk5LWUwMmYtNGZkOC1hN2Q4LTNiMTllZWFkYWJhNS8iLCJpZHR5cCI6ImFwcCIsIm9pZCI6ImQ0NDA3YjQ4LWZmZTctNDJjNS04ZDIwLTdiMTTgwNWE4NCIsInJoIjoiMS5BUnNBbWRfVERDX2cyRS1uMkRzWjdxMnJwUU1BQUFBQUFBQUF3QUFBQUFBQUFBRFlBQUFiQUEuIiwic3ViIjoiZDQ0MDdiNDgtZmZlNy00MmM1LThkMjAtN2IxMzU5ODA1YTg0IiwidGVuYW50X3JlZ2lvbl9zY29wZSI6Ik5BIiwidGlkIjoiMGNkM2RmOTktZTAyZi00ZmQ4LWE3ZDgtM2IxOWVlYWRhYmE1IiwidXRpIjoiLVA1T3JPWGpJVWk0VE12dElTYWRBQSIsInZlciI6IjEuMCIsIndpZHMiOlsiMDk5N2ExZDAtMGQxZC00YWNiLWI0MDgtZDVjYTczMTIxZTkwIl0sInhtc19pZHJlbCI6IjI4IDciLCJ4bXNfdGNkdCI6MTQ4NjM3NDQ2MH0.a9z3ZYyMTRQCoY7dzPYE55DmpNAxqo4a4rrt80A-RpK0NDDAftNkc2hafbLl6gdwEzqRyKc1HExUggFUpKxaLUXc62-u-9emxC12EsNlQYd-ZzG_GRDNoTYrro4RDRL-_gDo2lgBNOi5ZZ4a9UI_pYVvV1b0SBRpgd5bmIV4kI2tDfAVZ1-HMpGscuVkQIy45Tqt4c3gXPoMEZ3UYikbCpErbTNfUFqngE3sARXRV-rB1OMu6ZbN32ijjL-rD8593-IfSpmVDUfE5CMGc-7FuWGOYyUUJmp5AQ1yFpJzqaDBEdPT8kKync1o7eplWXCsPWOnVvAKNf7BuWCRRedBWg"
}
```

The following steps demonstrate how to use these cmdlets to run a job in your HDInsight cluster:

1. Using an editor, save the following code as `hivejob.ps1`.
    
   
   ```powershell
    # Login to your Azure subscription
    $context = Get-AzContext
    if ($context -eq $null) 
    {
        Connect-AzAccount
    }
    $context

    # Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $bearerToken = Read-Host -Prompt "Enter the bearer token"

    #HiveQL
    #Note: set hive.execution.engine=tez; is not required for
    #      Linux-based HDInsight
    $queryString = "set hive.execution.engine=tez;" +
            "DROP TABLE log4jLogs;" +
            "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';" +
            "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"

    #Create an HDInsight Hive job definition
    $hiveJobDefinition = New-AzHDInsightHiveJobDefinition -Query $queryString

    #Submit the job to the cluster
    Write-Host "Start the Hive job..." -ForegroundColor Green

    $hiveJob = Start-AzHDInsightJob -ClusterName $clusterName -JobDefinition $hiveJobDefinition -Headers @{ "Authorization" = "Bearer $token" }

    #Wait for the Hive job to complete
    Write-Host "Wait for the job to complete..." -ForegroundColor Green
    Wait-AzHDInsightJob -ClusterName $clusterName -JobId $hiveJob.JobId -Headers @{ "Authorization" = "Bearer $token" }
    # Print the output
    Write-Host "Display the standard output..." -ForegroundColor Green
    Get-AzHDInsightJobOutput `
    -Clustername $clusterName `
    -JobId $hiveJob.JobId `
    -Headers @{ "Authorization" = "Bearer $token" }
   ```

1. Open a new **Azure PowerShell** command prompt. Change directories to the location of the `hivejob.ps1` file, then use the following command to run the script:
    

    ```powershell
      .\hivejob.ps1
    ```

    When the script runs, you're prompted to enter the cluster name and the HTTPS/Cluster Admin account credentials. You may also be prompted to sign in to your Azure subscription.


1. When the job completes, it returns information similar to the following text:
    
    Output

   ```json
      Display the standard output...
      2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
      2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
      2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id

   ```


1. As mentioned earlier, `Invoke-Hive` can be used to run a query and wait for the response. Use the following script to see how Invoke-Hive works:
    


     ```powershell
        # Login to your Azure subscription
        $context = Get-AzContext
        if ($context -eq $null) 
        {
            Connect-AzAccount
        }
          $context

        # Get cluster info
        $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
        $bearerToken = Read-Host -Prompt "Enter the bearer token"

        # Set the cluster to use
        Use-AzHDInsightCluster -ClusterName $clusterName -Headers @{ "Authorization" = "Bearer $token" }

          $queryString = "set hive.execution.engine=tez;" +
            "DROP TABLE log4jLogs;" +
            "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)                 ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/';" +
            "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"
          Invoke-AzHDInsightHiveJob `
            -StatusFolder "statusout" `
            -Query $queryString

     ```
1. The output looks like the following text:

   Output
     ```json
        2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
        2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
        2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id

     ```

> [!NOTE]  
>For longer HiveQL queries, you can use the Azure PowerShell **Here-Strings** cmdlet or HiveQL script files. The following snippet shows how to use the `Invoke-Hive` cmdlet to run a HiveQL script file. The HiveQL script file must be uploaded to wasbs://.`Invoke-AzHDInsightHiveJob -File "wasbs://<ContainerName>@<StorageAccountName>/<Path>/query.hql"` For more information about **Here-Strings**, see [**HERE-STRINGS**](/powershell/module/microsoft.powershell.core/about/about_quoting_rules#here-strings).

### Troubleshooting

If no information is returned when the job completes, view the error logs. To view error information for this job, add the following to the end of the `hivejob.ps1` file, save it, and then run it again.


```powershell
  # Print the output of the Hive job
Get-AzHDInsightJobOutput `
    -Clustername $clusterName `
    -Headers @{ "Authorization" = "Bearer $token" }
    -JobId $job.JobId `
    -DisplayOutputType StandardError `

```

This cmdlet returns the information that is written to STDERR during job processing.

## Summary

As you can see, Azure PowerShell provides an easy way to run Hive queries in an HDInsight cluster, monitor the job status, and retrieve the output.




