---
title: Run Apache Hive queries in HDInsight using REST API
description: Learn how to run Apache Hive queries in HDInsight using REST API
ms.service: azure-hdinsight
ms.topic: how-to
author: apurbasroy
ms.author: apsinhar
ms.reviewer: nijelsf
ms.date: 08/20/2025
---

# Run Apache Hive queries in HDInsight using the REST API

Apache Hive enables you to query and analyze large datasets in Azure HDInsight using a familiar SQL-like language. The HDInsight REST API provides a programmatic way to submit Hive queries, monitor execution, and retrieve results without requiring direct cluster login or manual tools.

With the REST API, you can integrate Hive query execution into applications, scripts, and automation pipelines. This approach is especially useful for scenarios where you need to run queries from external systems, enforce secure access through Microsoft Entra ID, or manage workloads at scale.

## Prerequisites

- An Entra Enabled Apache Hadoop cluster on HDInsight. See [Get Started with HDInsight on Linux](../hadoop/apache-hadoop-linux-tutorial-get-started.md).
- A REST client. This document uses [Curl](https://curl.haxx.se/).
- If you use Bash, use jq, a command-line JSON processor. See [Download jq](https://jqlang.org/download/)

### Base URI for REST API

The base Uniform Resource Identifier(URI) for the REST API on HDInsight is `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME`, where `CLUSTERNAME` is the name of your cluster. Cluster names in URIs are **case-sensitive**. While the cluster name in the fully qualified domain name (FQDN) part of the URI (`CLUSTERNAME.azurehdinsight.net`) is case-insensitive, other occurrences in the URI are case-sensitive.

### Authentication

When using cURL or any other REST communication with WebHCat, you must authenticate the requests by providing the Bearer Token for the HDInsight cluster administrator. The REST API is secured via OAuth 2.0. To help ensure that your credentials are securely sent to the server, always make requests by using Secure HTTP (HTTPS).

### Setup (Secure Bearer Access Token)

Bearer Token is needed to send the cURL or any REST communication. Follow the mentioned steps to get the token:

Execute an HTTP GET request to the OAuth 2.0 token endpoint with the following specifications:

### URL
  ```json
    https://login.microsoftonline.com/{Tenant_ID}/oauth2/v2.0/token
  ```


### Body
  ```bash
      curl --request GET \
    --url https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token \
    --header 'Content-Type: multipart/form-data' \
    --form grant_type=client_credentials \
    --form client_id={app_id} \
    --form client_secret={client_secret} \
    --form scope=https://{clustername}.clusteraccess.azurehdinsight.net/.default \
  ```

## Response

A successful request returns a JSON object containing:

- token_type: Always "Bearer"
- expires_in: Token validity duration in seconds
- ext_expires_in: Extended expiration time in seconds
- access_token: The Bearer token for authentication

  ```bash
      {
	"token_type": "Bearer",
	"expires_in": 3599,
	"ext_expires_in": 3599,
	"access_token": "eyJ0eXAiOiJKV1iLCJub25jZSI6IkhaZ3lqQ2MxSkxzaXRSbmxzT1FTSHV0bEtBeXhhMU1JTzdyWmluLWF6LUEiLCJhbGciOiJSUzI1NiIsIng1dCI6ImltaTBZMnowZFlLeEJ0dEFxS19UdDVoWUJUayIsImtpZCI6ImltaTBZMnowZFlLeEJ0dEFxS19UdDVoWUJUayJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8wY2QzZGY5OS1lMDJmLTRmZDgtYTdkOC0zYjE5ZWVhZGFiYTUvIiwiaWF0IjoxNzQxMjgzMzUzLCJuYmYiOjE3NDEyODMzNTMsImV4cCI6MTc0MTI4NzI1MywiYWlvIjoiazJSZ1lIRDF1U1R4NGx2bjdmMTdGcXlkZUdwWlBnQT0iLCJhcHBfZGlzcGxheW5hbWUiOiJBenVyZSBIREkgTVNGVCBDbGllbnQiLCJhcHBpZCI6IjAzZDNiNTg5LWFjM2MtNDE4NC1iY2EyLTQ3ZWRiN2Q2ZmVjNiIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzBjZDNkZjk5LWUwMmYtNGZkOC1hN2Q4LTNiMTllZWFkYWJhNS8iLCJpZHR5cCI6ImFwcCIsIm9pZCI6ImQ0NDA3YjQ4LWZmZTctNDJjNS04ZDIwLTdiMTTgwNWE4NCIsInJoIjoiMS5BUnNBbWRfVERDX2cyRS1uMkRzWjdxMnJwUU1BQUFBQUFBQUF3QUFBQUFBQUFBRFlBQUFiQUEuIiwic3ViIjoiZDQ0MDdiNDgtZmZlNy00MmM1LThkMjAtN2IxMzU5ODA1YTg0IiwidGVuYW50X3JlZ2lvbl9zY29wZSI6Ik5BIiwidGlkIjoiMGNkM2RmOTktZTAyZi00ZmQ4LWE3ZDgtM2IxOWVlYWRhYmE1IiwidXRpIjoiLVA1T3JPWGpJVWk0VE12dElTYWRBQSIsInZlciI6IjEuMCIsIndpZHMiOlsiMDk5N2ExZDAtMGQxZC00YWNiLWI0MDgtZDVjYTczMTIxZTkwIl0sInhtc19pZHJlbCI6IjI4IDciLCJ4bXNfdGNkdCI6MTQ4NjM3NDQ2MH0.a9z3ZYyMTRQCoY7dzPYE55DmpNAxqo4a4rrt80A-RpK0NDDAftNkc2hafbLl6gdwEzqRyKc1HExUggFUpKxaLUXc62-u-9emxC12EsNlQYd-ZzG_GRDNoTYrro4RDRL-_gDo2lgBNOi5ZZ4a9UI_pYVvV1b0SBRpgd5bmIV4kI2tDfAVZ1-HMpGscuVkQIy45Tqt4c3gXPoMEZ3UYikbCpErbTNfUFqngE3sARXRV-rB1OMu6ZbN32ijjL-rD8593-IfSpmVDUfE5CMGc-7FuWGOYyUUJmp5AQ1yFpJzqaDBEdPT8kKync1o7eplWXCsPWOnVvAKNf7BuWCRRedBWg"
      }
  ```

### Run a Hive query

1. To verify that you can connect to your HDInsight cluster, use one of the following commands:

      ```bash
        curl -H "Authorization: Bearer $TOKEN" -G https://$CLUSTER_NAME.azurehdinsight.net/templeton/v1/status

      ```

      You receive a response similar to the following text:

      ```json
        {"status":"ok","version":"v1"}
      ```
    The parameters used in this command are as follows:

  
 	 - 'u' The username and password used to authenticate the request.
     - 'G' Indicates that this request is a GET operation.
1. The beginning of the URL, `https://$CLUSTERNAME.azurehdinsight.net/templeton/v1`, is the same for all requests. The path, `/status`, indicates that the request is to return a status of WebHCat (also known as Templeton) for the server. You can also request the version of Hive by using the following command:

    ```bash
      curl -H "Authorization: Bearer $TOKEN" -G https://$CLUSTER_NAME.azurehdinsight.net/templeton/v1/version/hive
    ```

    This request returns a response similar to the following text:
    
    ```json
      {"module":"hive","version":"1.2.1000.2.6.5.3008-11"}
    ```

1. Use the following to create a table named **log4jLogs**:
    
    
    ```bash
      curl -s -H "Authorization: Bearer $TOKEN" -d user.name=admin -d execute="DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'/example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="/example/rest" https://$CLUSTER_NAME.azurehdinsight.net/templeton/v1/hive | jq -r .id
    ```
    This request uses the POST method, which sends data as part of the request to the REST API. The following data values are sent with the request:

      - `user.name` - The user that's running the command.
      - `execute` - The HiveQL statements to execute.
      - `statusdir` - The directory that the status for this job is written to.

    These statements perform the following actions:

     - `DROP TABLE` - If the table already exists, it is deleted.
     - `CREATE EXTERNAL TABLE` - Creates a new 'external' table in Hive. External tables store only the table definition in Hive. The data is left in the original location.


  > [!NOTE]  
  > Use external tables when an external source updates the underlying data. For example, an automated data upload process or another MapReduce operation.


Dropping an external table does **not** delete the data, only the table definition.


      

-`ROW FORMAT` - How the data is formatted. The fields in each log are separated by a space.
-`STORED AS TEXTFILE LOCATION` - Where the data is stored (the example/data directory) and that it is stored as text.
-`SELECT` - Selects a count of all rows where column **t4** contains the value **[ERROR]**. This statement returns a value of **3** as there are three rows that contain this value.


  > [!NOTE]  
  > Curl replaces the spaces between HiveQL statements with the + character,'+' should not replace quoted values that contain a space, such as the delimiter.
    
    
 This command returns a job ID that can be used to check the status of the job.



### To check the status of the job, use the following command:
         
 ```bash
 	curl -H "Authorization: Bearer $TOKEN" -d user.name=admin -G 		     https://$CLUSTER_NAME.azurehdinsight.net/templeton/v1/jobs/$jobid | jq .status.state
 ```
If the job finishes, the state is **SUCCEEDED**.

 Once the state of the job changes to **SUCCEEDED**, you can retrieve the results of the job from Azure Blob storage. The `statusdir` parameter passed with the query contains the location of the output file; in this case, `/example/rest`. This address stores the output in the `example/curl` directory in the clusters default storage.

