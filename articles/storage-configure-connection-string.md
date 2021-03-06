<properties 
	pageTitle="Configure a Connection String to Azure Storage | Microsoft Azure" 
	description="Learn how to configure a connection string to an Azure storage account. A connection string includes the information needed to authenticate programmatic access to resources in a storage account. The connection string may encapsulate your account access key for an account that you own, or it may include a shared access signature for accessing resources in an account without the access key." 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/28/2015" 
	ms.author="tamram"/>

# Configure Azure Storage Connection Strings

## Overview

A connection string includes the information necessary to access Azure Storage resources programmatically. Your application uses the connection string to provide to Azure Storage the information needed to authenticate access.

You can configure a connection string to:

- Connect to the Azure storage emulator while you are locally testing your service or application.

- Connect to a storage account in Azure by using the default endpoints for the storage services.

- Connect to a storage account in Azure by using explicit endpoints for the storage services.

- Access resources in a storage account via a shared access signature (SAS).

In your application, you'll use the connection string to encapsulate your credentials for accessing storage resources. 


Once you have done so using the [ServiceCredentials](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.auth.storagecredentials.aspx) type, you can use those credentials for subsequent requests. A common way to do this is via a service client object. A service client object represents one of the storage services, and can be used to access the functionality of that service from your code. The service client types are:

- [CloudBlobClient](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobclient.aspx)
- [CloudQueueClient](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.queue.cloudqueueclient.aspx)
- [CloudTableClient](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.table.cloudtableclient.aspx)
- [CloudFileClient](https://msdn.microsoft.com/library/microsoft.windowsazure.storage.file.cloudfileclient.aspx)

## Storing your connection string

Your application will need to store the connection string in order to authenticate access to Azure Storage when it is running. You have a few different options for storing your connection string:

- For an application running on the desktop or on a device, you can store the connection string in an app.config file or another configuration file.
- For an application running in an Azure cloud service, you can store your connection string in the [Azure service configuration schema (.cscfg) file](https://msdn.microsoft.com/library/ee758710.aspx). 

You can use the Azure [CloudConfigurationManager](https://msdn.microsoft.com/library/microsoft.windowsazure.cloudconfigurationmanager.aspx) class to access your connection string at runtime regardless of where your application is running.

## Create a connection string to the storage emulator

The storage emulator is a local account with a well-known name and key. Since the account name and key are the same for all users, you can use a shortcut string format to refer to the storage emulator within a connection string. Set the value of the connection string to `UseDevelopmentStorage=true`.

You can also specify an HTTP proxy to use when you're testing your service against the storage emulator. This can be useful for observing HTTP requests and responses while you're debugging operations against the storage services. To specify a proxy, add the `DevelopmentStorageProxyUri` option to the connection string, and set its value to the proxy URI. For example, here is a connection string that points to the storage emulator and configures an HTTP proxy:

    UseDevelopmentStorage=true;DevelopmentStorageProxyUri=http://myProxyUri

## Create a connection string to an Azure storage account

To create a connection string to your Azure storage account, use the connection string format below. Indicate whether you want to connect to the storage account through HTTP or HTTPS (recommended), replace `myAccountName` with the name of your storage account, and replace `myAccountKey` with your account access key:

    DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey

For example, your connection string will look similar to the following sample connection string:

```        DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=KWPLd0rpW2T0U7K2pVpF8rYr1BgYtB7wYQw33AYiXeUoquiaY6o0TWqduxmPHlqeCNZ3LU0DHptbeIAy5l/Yhg==
```

> [AZURE.NOTE] Azure Storage supports both HTTP and HTTPS in a connection string; however, using HTTPS is highly recommended.
    
## Creating a connection string to an explicit Storage endpoint

You can to explicitly specify the service endpoints in your connection string if:

- You have mapped a custom domain name for your storage account with the Blob service.
- You possess a shared access signature for accessing storage resources in a storage account.

To create a connection string that specifies an explicit Blob endpoint, specify the complete service endpoint for each service, including the protocol specification (HTTP or HTTPS), in the following format:

``` 
BlobEndpoint=myBlobEndpoint;QueueEndpoint=myQueueEndpoint;TableEndpoint=myTableEndpoint;FileEndpoint=myFileEndpoint;[credentials]
```

You must specify at least one service endpoint, but you do not need to specify all of them. For example, if you're creating a connection string for use with a custom blob endpoint, specifying the queue and table endpoints is optional. Note that if you choose to omit the queue and table endpoints from the connection string, then you will not be able to access the Queue and Table services from your code by using that connection string.

When you explicitly specify service endpoints in the connection string, you have two options for specifying `credentials` in the string above:

- You can specify the account name and key: `AccountName=myAccountName;AccountKey=myAccountKey` 
- You can specify a shared access signature: `SharedAccessSignature=base64Signature`

### Specifying a Blob endpoint with a custom domain name 

If you have registered a custom domain name for use with the Blob service, you may want to explicitly configure the blob endpoint in your connection string. The endpoint value that is listed in the connection string is used to construct the request URIs to the Blob service, and it dictates the form of any URIs that are returned to your code. 

For example, a connection string to a Blob endpoint on a custom domain may be similar to:

```
DefaultEndpointsProtocol=https;BlobEndpoint=www.mydomain.com;AccountName=storagesample;AccountKey=KWPLd0rpW2T0U7K2pVpF8rYr1BgYtB7wYQw33AYiXeUoquiaY6o0TWqduxmPHlqeCNZ3LU0DHptbeIAy5l/Yhg== 
```

### Specifying a Blob endpoint with a shared access signature 

You can create a connection string with explicit endpoints to access storage resources via a shared access signature. In this case, you can specify the shared access signature for as part of the connection string, rather than the account name and key credentials. The shared access signature token encapsulates information about the resource to be accessed, the period of time for which it is available, and the permissions being granted. For more information about shared access signatures, see [Delegating Access with a Shared Access Signature](https://msdn.microsoft.com/library/ee395415.aspx).

To create a connection string that includes a shared access signature, specify the string in the following format:

    BlobEndpoint=myBlobEndpoint; QueueEndpoint=myQueueEndpoint;TableEndpoint=myTableEndpoint;SharedAccessSignature=base64Signature

The endpoint can be either the default service endpoint or a custom endpoint. The `base64Signature` corresponds to the signature portion of a shared access signature. The signature is an HMAC computed over a valid string-to-sign and key using the SHA256 algorithm, that is then Base64-encoded.

### Creating a connection string with an endpoint suffix

To create a connection string for storage service in regions or instances with different endpoint suffixes, such as for Azure China or Azure Governance, use the following connection string format. Indicate whether you want to connect to the storage account through HTTP or HTTPS, replace `myAccountName` with the name of your storage account, replace `myAccountKey` with your account access key, and replace `mySuffix` with the URI suffix:


	DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=mySuffix;


For example, your connection string should look similar to the following sample connection string:

	DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=KWPLd0rpW2T0U7K2pVpF8rYr1BgYtR7wYQk33AYiXeUoquiaY6o0TWqduxmPHlqeCNZ3LU0DHptbeIHy5l/Yhg==;EndpointSuffix=core.chinacloudapi.cn;

