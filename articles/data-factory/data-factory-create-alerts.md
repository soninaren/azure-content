<properties 
	pageTitle="Create alerts on Azure Data Factory events" 
	description="Learn how you can create alerts on events raised by the Azure for data factory operations." 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/21/2015" 
	ms.author="spelluru"/>

# Creating alerts on Azure events
Azure events provide useful insights into what is happening in your Azure resources. Azure logs user events when an Azure resource (e.g. data factory) is created, updated or deleted. When using the Azure Data Factory, events are generated when:
 
1.	Azure Data Factory is created/updated/deleted.
2.	Data processing (called as Runs) has started/completed.
3.	When an on-demand HDInsight cluster is created and removed.

You can create alerts on these user events and configure them to send email notifications to the administrator and co-administrators of the subscription. In addition, you can specify additional email addresses of users who need to receive email notifications when the conditions are met.

## Specifying an alert definition
To specify an alert definition, you create a JSON file describing the operations that you want to be alerted on. In the example below, the alert will send an email notification for the **RunFinished** operation. To be specific, an email notification is sent when a run in the data factory has completed and the run has failed (Status = FailedExecution).

	{
    	"contentVersion": "1.0.0.0",
   		 "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    	"parameters": {},
    	"resources": 
		[
        	{
            	"name": "ADFAlertsSlice",
            	"type": "microsoft.insights/alertrules",
            	"apiVersion": "2014-04-01",
            	"location": "East US",
            	"properties": 
				{
                	"name": "ADFAlertsSlice",
                	"description": "One or more of the data slices for the Azure Data Factory has failed processing.",
                	"isEnabled": true,
                	"condition": 
					{
                    	"odata.type": "Microsoft.Azure.Management.Insights.Models.ManagementEventRuleCondition",
                    	"dataSource": 
						{
                        	"odata.type": "Microsoft.Azure.Management.Insights.Models.RuleManagementEventDataSource",
                        	"operationName": "RunFinished",
			         		"status": "Failed",
                        		"subStatus": "FailedExecution"   
                    	}
                	},
                	"action": 
					{
                    	"odata.type": "Microsoft.Azure.Management.Insights.Models.RuleEmailAction",
                    	"customEmails": [ "<your alias>@contoso.com" ]
                	}
            	}
        	}
    	]
	}

From the above JSON definition, **subStatus** can be removed if you don’t want to be alerted on a specific failure.

See [Available Operations and Statuses](#AvailableOperationsStatuses) for the list of operations and statuses (and sub-statuses). 

## Deploying the Alert
To deploy the alert, use the Azure PowerShell cmdlet: **New-AzureResourceGroupDeployment**, as shown in the following example:

	New-AzureResourceGroupDeployment -ResourceGroupName adf 	-TemplateFile .\ADFAlertFailedSlice.json -StorageAccountName testmetricsabc

The StorageAccountName specifies the storage account for storing the deployed alert JSON file.

Once the resource group deployment has completed successfully, you will see the following messages:
	
	VERBOSE: 7:00:48 PM - Template is valid.
	WARNING: 7:00:48 PM - The StorageAccountName parameter is no longer used and will be removed in a future release.
	Please update scripts to remove this parameter.
	VERBOSE: 7:00:49 PM - Create template deployment 'ADFAlertFailedSlice'.
	VERBOSE: 7:00:57 PM - Resource microsoft.insights/alertrules 'ADFAlertsSlice' provisioning status is succeeded

	DeploymentName    : ADFAlertFailedSlice
	ResourceGroupName : adf
	ProvisioningState : Succeeded
	Timestamp         : 10/11/2014 2:01:00 AM
	Mode              : Incremental
	TemplateLink      :
	Parameters        :
	Outputs           :

## Retrieving the list of Azure Resource Group Deployments
To retrieve the list of deployed Azure Resource Group deployments, use the cmdlet: **Get-AzureResourceGroupDeployment**, as shown in the following example:
	
	Get-AzureResourceGroupDeployment -ResourceGroupName adf
	
	DeploymentName    : ADFAlertFailedSlice
	ResourceGroupName : adf
	ProvisioningState : Succeeded
	Timestamp         : 10/11/2014 2:01:00 AM
	Mode              : Incremental
	TemplateLink      :
	Parameters        :
	Outputs           :

## <a name="AvailableOperationsStatuses"></a>Available Operation Names and Status Values

| Operation Name | Status | Sub Status |
| -------------- | ------ | ---------- |
| RunStarted | Started | Starting |
| RunFinished | Failed/Succeeded |	<p>FailedResourceAllocation </p><p>Succeeded</p><p>FailedExecution</p><p>TimedOut</p><p>Canceled</p><p>FailedValidation</p><p>Abandoned</p> | 
| SliceOnTime | In Progress | Ontime |
| SliceDelayed | In Progress | Late |
| OnDemandClusterCreateStarted | Started | |
| OnDemandClusterCreateSuccessful | Succeeded | | 
| OnDemandClusterDeleted | Succeeded | |


## Troubleshooting User Events
Run the following command to see the generated events:

	Get-AzureResourceGroupLog –Name $ResourceGroup -All | Where-Object EventSource -eq "Microsoft.DataFactory"
 