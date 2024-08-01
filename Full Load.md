
# Full Data Pipeline

## Overview

This document outlines the design and implementation of an Full data pipeline. The goal is to process only the new or modified data since the last run, thus improving efficiency and reducing processing time.

![Image Description](https://github.com/heyrobin/Microsoft-Fabric/blob/main/Images/Full%20Load.png)

*Figure 1: Full load pipeline.*

## Lookup
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
                "value": "SELECT * FROM dbo.watermarktable",
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
                "name": "Full Load",
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
                            "value": "@concat('SELECT * FROM ','dbo','.',item().TableName)",
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
                        "type": "ParquetSink",
                        "storeSettings": {
                            "type": "LakehouseWriteSettings"
                        },
                        "formatSettings": {
                            "type": "ParquetWriteSettings",
                            "enableVertiParquet": true
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
                            "type": "Parquet",
                            "typeProperties": {
                                "location": {
                                    "type": "LakehouseLocation",
                                    "fileName": {
                                        "value": "@concat(item().TableName,'.parquet')",
                                        "type": "Expression"
                                    },
                                    "folderPath": "Ecom/Full"
                                },
                                "compressionCodec": "snappy"
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
            }
        ]
    }
}
```

## WaterMark Table


![Image Description](https://github.com/heyrobin/Microsoft-Fabric/blob/main/Images/Full%20Load%20Water%20Mark%20Table.png)

*Figure 2: WaterMark Table.*
