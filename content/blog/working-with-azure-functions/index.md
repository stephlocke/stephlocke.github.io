---
title: Working with Azure Functions
date: 2023-03-21T09:00:00Z
tags:
  - Azure Functions
  - Azure
  - Serverless
---

This post is an overview of Azure Functions based on the session "Working with data using Azure Functions" that was first delivered at SQLBits with Liam Moat. See the slides for the session at [Talks/Working with data using Azure Functions](../../presenting/talks/working_with_data_using_azure_functions/).

## What are Azure Functions?

Azure Functions are one of the latest abstractions away from needing to define everything yourself when performing activities using software.

First we abstracted away from the physical server to the virtual, then started using Platform as a Service or containerising code to run in the cloud. Now we have the ability to run code without needing to worry about the underlying infrastructure.

The current level of abstraction is called serverless. Serverless means no infrastructure management, event-driven scalabity, and a pay-per-use model. Azure Functions, AWS Lambda, and similar are part of Functions as a Service (FaaS). These Functions usually have a single responsibility, are short-lived, stateless, and, most commonly, are event-driven.

Azure Functions include an integrated programming model that handles things like triggers, has robust tooling support, can be hosted in many ways, and is an incredibly cost-efficient choice for many workloads.

## Anatomy of a Function

An Azure Function has a few key components:

- A trigger that causes the Function to execute. This could be a timer, an event, or a remote request for it run
- 0 or more inputs that fetch data from different sources like CosmosDB or Azure Table storage
- 0 or more outputs that send data to different destinations like Azure EventHubs or Azure SQL

{{< amp-image src="AnatomyOfAFunction.png"
layout="responsive"
height=150
width=300 >}}

Behind the scenes, there will be helper files that include dependencies, configuration settings, and other things that are needed to run the Function. Multiple functions can be helped in a single project making it easy to deploy and manage related Functions.

## Streamlining connections and improving security

Managing connection strings and keeping passwords is both a pain and a risk! Instead we can use Managed Identities (basically service principals) to authenticate securely against downstream connections and even to communicate with the Functions themselves.

For working with data sources that don't have Managed Identity, you can store secrets in Azure Key Vault, include a reference to the Key Vault secret in app settings, and then have the Function communicate with Key Vault and authenticating with it's Managed Identity.

To leverage Managed Identity connectivity to different data sources like CosmosDB you will need to use v4 versions of the associated packages. Note that at this point in time (21st March 2023) that VS Code and Azure Functions Core Tools heavily preference making v4 a little awkward to initially code if you're not the most comfortable programmer.

I firmly believe in CODING ALL THE THINGS! To that point, I recommend you use Infrastructre as Code (IaC) to create the resources of your Azure envirnment and even assign access rights to the Azure Function.

To do this, as well as resource creation, we can perform role assignments for the Azure Function app's Managed Identity.

```bicep
param principalId string

param roleDefinitionIds array

var roleAssignmentsToCreate = [for roleDefinitionId in roleDefinitionIds: {
  name: guid(principalId, roleDefinitionId)
  roleDefinitionId: roleDefinitionId
}]

resource roleAssignment 'Microsoft.Authorization/roleAssignments@2020-04-01-preview' = [for roleAssignmentToCreate in roleAssignmentsToCreate: {
  name: roleAssignmentToCreate.name
  scope: resourceGroup()
  properties: {
    description: roleAssignmentToCreate.name
    principalId: principalId
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', roleAssignmentToCreate.roleDefinitionId)
    principalType: 'ServicePrincipal'
  }
}]
```

(src: [roleAssignments.bicep](https://github.com/stephlocke/working-with-data-using-azure-functions/blob/main/bicep/modules/roleAssignments.bicep))

```bicep
module roleAssignments 'roleAssignments.bicep' = {
  name: 'roleAssignments'
  params: {
    principalId: functionApp.identity.principalId
    roleDefinitionIds: [
      'b7e6dc6d-f1e8-4753-8033-0f276bb0955b' // Storage Blob Data Owner
      'f526a384-b230-433a-b45c-95f59c4a2dec' // Azure Event Hubs Data Owner 
    ]
  }
}
```

(src: [functionApp.bicep](https://github.com/stephlocke/working-with-data-using-azure-functions/blob/main/bicep/modules/functionApp.bicep))

We can use the v4 connection string approach to reference the different resources in the app settings.

```bicep
resource functionApp 'Microsoft.Web/sites@2021-03-01' = {
  name: functionAppName
  location: location
  kind: 'functionapp'
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: hostingPlan.id
    siteConfig: {
      appSettings: [
        {
          name: 'AzureWebJobsStorage__accountname'
          value: storageAccountName
        }
        {
          name: 'WEBSITE_CONTENTAZUREFILECONNECTIONSTRING'
          value: 'DefaultEndpointsProtocol=https;AccountName=${storageAccountName};EndpointSuffix=${environment().suffixes.storage};AccountKey=${storageAccount.listKeys().keys[0].value}'
        }
        {
          name: 'CorpDB__accountEndpoint'
          value: databaseAccount.properties.documentEndpoint
        }
        {
          name: 'SQLConnection'
          value: 'Server=${sqlServer.properties.fullyQualifiedDomainName}; Authentication=Active Directory Managed Identity; Database=${sqlDatabaseName}'
        }
        {
          name: 'EventHubConnection__fullyQualifiedNamespace'
          value: '${eventHubNamespaceName}.servicebus.windows.net'
        }
        {
          name: 'MICROSOFT_PROVIDER_AUTHENTICATION_SECRET'
          value: authProviderSecret
        }
        {
          name: 'WEBSITE_CONTENTSHARE'
          value: toLower(functionAppName)
        }
        {
          name: 'FUNCTIONS_EXTENSION_VERSION'
          value: '~4'
        }
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: instrumentationKey
        }
        {
          name: 'FUNCTIONS_WORKER_RUNTIME'
          value: 'dotnet'
        }
      ]
      ftpsState: 'FtpsOnly'
      minTlsVersion: '1.2'
    }
    httpsOnly: true
  }
}
```

(src: [functionApp.bicep](https://github.com/stephlocke/working-with-data-using-azure-functions/blob/main/bicep/modules/functionApp.bicep))

The specific functions then refer to simplified connections when mapping to triggers, inputs, and outputs. Inb the example below "CorpDB" is used instead of "CorpDB__accountEndpoint".

```csharp
namespace SQLBitsDemo.Functions.Cosmos
{
    public static class CosmosOutputTest
    {
        [FunctionName("CosmosOutputTest")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            [CosmosDB(
                databaseName: "CorpDB",
                containerName: "People",
                Connection = "CorpDB")]IAsyncCollector<Person> people,
            ILogger log)
        {
```

## Getting started with Azure Functions

Before the end of April 2023, you can [enter our Azure Functions Cloud Skills Challenge](https://aka.ms/sqlbits-dwf) to work through great Microsoft content on this topic. After April, you will be able to access the [learning path](https://learn.microsoft.com/en-us/users/stephlocke-1889/collections/3gm0ij7jznwpk2) but you won't get as much kudos. 😉
