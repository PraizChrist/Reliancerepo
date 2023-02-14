Analytic Cloud Deployment Flow


Connect to acount/subsccription

```
$subs = Get-AzSubscription | Select-Object -ExpandProperty Name
if($subs.GetType().IsArray -and $subs.length -gt 1){
        $subOptions = [System.Collections.ArrayList]::new()
        for($subIdx=0; $subIdx -lt $subs.length; $subIdx++){
                $opt = New-Object System.Management.Automation.Host.ChoiceDescription "$($subs[$subIdx])", "Selects the $($subs[$subIdx]) subscription."   
                $subOptions.Add($opt)
        }
        $selectedSubIdx = $host.ui.PromptForChoice('Enter the desired Azure Subscription for this lab','Copy and paste the name of the subscription to make your choice.', $subOptions.ToArray(),0)
        $selectedSubName = $subs[$selectedSubIdx]
        Write-Information "Selecting the $selectedSubName subscription"
        Select-AzSubscription -SubscriptionName $selectedSubName
}
```
2. Linkservice

```
{
    "name": {{name}}, 
    "type": "Microsoft.Synapse/workspaces/linkedservices",
    "properties": {
        "annotations": [],
        "type": "AzureBlobStorage|CosmosDb|PowerBIWorkspace",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=#STORAGE_ACCOUNT_NAME#;AccountKey=#STORAGE_ACCOUNT_KEY#;EndpointSuffix=core.windows.net;"
        }
    }
}

```



2 dataset
```
{
    "name": {{name]},
    "properties": {
        "linkedServiceName": {
            "referenceName": "#LINKED_SERVICE_NAME#",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "AzureSqlDWTable",
        "schema": [
            {
                "name": "",
                "type": ""
            },
         
        ],
        "typeProperties": {
            "schema": {{scheama_name}},
            "table": {{table_name}}"
        }
    },
    "type": "Microsoft.Synapse/workspaces/datasets"
}
```



** Pipelinee
```
{
    "name": "ASAMCW - Exercise 2 - Copy Product Information",
    "properties": {
        "activities": [
            {
                "name": "Copy Product Information",
                "type": "Copy",
                "dependsOn": [],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "source": {
                        "type": "DelimitedTextSource",
                        "storeSettings": {
                            "type": "AzureBlobFSReadSettings",
                            "recursive": true
                        },
                        "formatSettings": {
                            "type": "DelimitedTextReadSettings"
                        }
                    },
                    "sink": {
                        "type": "SqlDWSink",
                        "preCopyScript": "truncate table wwi_mcw.Product",
                        "allowPolyBase": true,
                        "polyBaseSettings": {
                            "rejectValue": 0,
                            "rejectType": "value",
                            "useTypeDefault": true
                        },
                        "disableMetricsCollection": false
                    },
                    "enableStaging": true,
                    "stagingSettings": {
                        "linkedServiceName": {
                            "referenceName": "asastoredemoman",
                            "type": "LinkedServiceReference"
                        },
                        "path": "staging"
                    },
                    "translator": {
                        "type": "TabularTranslator",
                        "mappings": [
                            {
                                "source": {
                                    "type": "String",
                                    "ordinal": 1
                                },
                                "sink": {
                                    "name": "ProductId",
                                    "type": "Int16"
                                }
                            },
                            {
                                "source": {
                                    "type": "String",
                                    "ordinal": 2
                                },
                                "sink": {
                                    "name": "Seasonality",
                                    "type": "Byte"
                                }
                            },
                            {
                                "source": {
                                    "type": "String",
                                    "ordinal": 3
                                },
                                "sink": {
                                    "name": "Price",
                                    "type": "Decimal"
                                }
                            },
                            {
                                "source": {
                                    "type": "String",
                                    "ordinal": 4
                                },
                                "sink": {
                                    "name": "Profit",
                                    "type": "Decimal"
                                }
                            }
                        ]
                    }
                },
                "inputs": [
                    {
                        "referenceName": "asamcw_product_csv",
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "asamcw_product_asa",
                        "type": "DatasetReference"
                    }
                ]
            }
        ],
        "annotations": [],
        "lastPublishTime": "2022-09-13T09:48:56Z"
    },
    "type": "Microsoft.Synapse/workspaces/pipelines"
}

```