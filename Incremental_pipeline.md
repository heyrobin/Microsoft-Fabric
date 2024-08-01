
# Incremental Data Pipeline

## Overview

This document outlines the design and implementation of an incremental data pipeline. The goal is to process only the new or modified data since the last run, thus improving efficiency and reducing processing time.

## Table of Contents
1. [Section 1](#section-1)
2. [Section 2](#section-2)
3. [Section 3](#section-3)
4. [Conclusion](#conclusion)
5. [References](#references)

## Lookup Code


```json
{
    "name": "Lookup Watermark",
    "type": "Lookup",
    "dependsOn": [],
    "policy": {
        "timeout": "0.12:00:00",
        "retry": 0,
        "retryIntervalInSeconds": 30,
        "secureOutput": false,
        "secureInput": false
    },
    "typeProperties": {
        "source": {
            "type": "DataWarehouseSource",
            "sqlReaderQuery": {
                "value": "SELECT * FROM dbo.watermark_table",
                "type": "Expression"
            },
            "queryTimeout": "02:00:00",
            "partitionOption": "None"
        },
        "firstRowOnly": false,
        "datasetSettings": {
            "annotations": [],
            "linkedService": {
                "name": "Silver",
                "properties": {
                    "annotations": [],
                    "type": "DataWarehouse",
                    "typeProperties": {
                        "endpoint": "kxqgblqraptexcbhphhmdszpea-66wsnzcjsywexovrmukrugjisu.datawarehouse.fabric.microsoft.com",
                        "artifactId": "8100ddec-e0bc-4d40-bcbd-b80fbacb8048",
                        "workspaceId": "e426adf7-9649-4b2c-bab1-65151a192895"
                    }
                }
            },
            "type": "DataWarehouseTable",
            "schema": [],
            "typeProperties": {
                "schema": "dbo",
                "table": "watermarktable"
            }
        }
    }
}
```


## For Each

```json
{
    "name": "looping tables",
    "type": "ForEach",
    "dependsOn": [
        {
            "activity": "Lookup Watermark",
            "dependencyConditions": [
                "Succeeded"
            ]
        }
    ],
    "typeProperties": {
        "items": {
            "value": "@activity('Lookup Watermark').output.value",
            "type": "Expression"
        },
        "isSequential": true,
        "activities": [
            {
                "name": "Incremental Load",
                "type": "Copy",
                "dependsOn": [],
                "policy": {
                    "timeout": "0.12:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "typeProperties": {
                    "source": {
                        "type": "SqlServerSource",
                        "sqlReaderQuery": {
                            "value": "@concat('SELECT * FROM ','dbo','.',item().table_name,\n' Where ', item().column_name,'>',item().watermark_value)",
                            "type": "Expression"
                        },
                        "queryTimeout": "02:00:00",
                        "partitionOption": "None",
                        "datasetSettings": {
                            "annotations": [],
                            "type": "SqlServerTable",
                            "schema": [],
                            "typeProperties": {
                                "schema": "dbo",
                                "table": "food",
                                "database": "ecom"
                            },
                            "externalReferences": {
                                "connection": "2023d801-e0c7-4429-94c4-bb942a8fbe63"
                            }
                        }
                    },
                    "sink": {
                        "type": "DelimitedTextSink",
                        "storeSettings": {
                            "type": "LakehouseWriteSettings"
                        },
                        "formatSettings": {
                            "type": "DelimitedTextWriteSettings",
                            "quoteAllText": true,
                            "fileExtension": ".txt"
                        },
                        "datasetSettings": {
                            "annotations": [],
                            "linkedService": {
                                "name": "Bronze",
                                "properties": {
                                    "annotations": [],
                                    "type": "Lakehouse",
                                    "typeProperties": {
                                        "workspaceId": "e426adf7-9649-4b2c-bab1-65151a192895",
                                        "artifactId": "42edde57-70cc-4c1c-9070-ae40cd055976",
                                        "rootFolder": "Files"
                                    }
                                }
                            },
                            "type": "DelimitedText",
                            "typeProperties": {
                                "location": {
                                    "type": "LakehouseLocation",
                                    "fileName": {
                                        "value": "@concat(item().table_name,'.txt')",
                                        "type": "Expression"
                                    },
                                    "folderPath": "Ecom/incremental"
                                },
                                "columnDelimiter": ",",
                                "escapeChar": "\\",
                                "firstRowAsHeader": true,
                                "quoteChar": "\""
                            },
                            "schema": []
                        }
                    },
                    "enableStaging": false,
                    "translator": {
                        "type": "TabularTranslator",
                        "typeConversion": true,
                        "typeConversionSettings": {
                            "allowDataTruncation": true,
                            "treatBooleanAsNumber": false
                        }
                    }
                }
            },
            {
                "name": "Lookup1",
                "type": "Lookup",
                "dependsOn": [
                    {
                        "activity": "Incremental Load",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "policy": {
                    "timeout": "0.12:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "typeProperties": {
                    "source": {
                        "type": "SqlServerSource",
                        "sqlReaderQuery": {
                            "value": "@concat('SELECT max(',item().column_name,') as max_value FROM ','dbo','.',item().table_name)",
                            "type": "Expression"
                        },
                        "queryTimeout": "02:00:00",
                        "partitionOption": "None"
                    },
                    "datasetSettings": {
                        "annotations": [],
                        "type": "SqlServerTable",
                        "schema": [],
                        "typeProperties": {
                            "database": "ecom"
                        },
                        "externalReferences": {
                            "connection": "2023d801-e0c7-4429-94c4-bb942a8fbe63"
                        }
                    }
                }
            },
            {
                "name": "Stored procedure1",
                "type": "SqlServerStoredProcedure",
                "dependsOn": [
                    {
                        "activity": "Lookup1",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "policy": {
                    "timeout": "0.12:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "typeProperties": {
                    "storedProcedureName": "[dbo].[usp_write_watermark]",
                    "storedProcedureParameters": {
                        "update_table_name": {
                            "value": {
                                "value": "@item().table_name",
                                "type": "Expression"
                            },
                            "type": "String"
                        },
                        "update_value": {
                            "value": {
                                "value": "@activity('Lookup1').output.firstRow.max_value",
                                "type": "Expression"
                            },
                            "type": "String"
                        }
                    }
                },
                "linkedService": {
                    "name": "Silver",
                    "objectId": "8100ddec-e0bc-4d40-bcbd-b80fbacb8048",
                    "properties": {
                        "annotations": [],
                        "type": "DataWarehouse",
                        "typeProperties": {
                            "endpoint": "kxqgblqraptexcbhphhmdszpea-66wsnzcjsywexovrmukrugjisu.datawarehouse.fabric.microsoft.com",
                            "artifactId": "8100ddec-e0bc-4d40-bcbd-b80fbacb8048",
                            "workspaceId": "e426adf7-9649-4b2c-bab1-65151a192895"
                        }
                    }
                }
            }
        ]
    }
}
```
