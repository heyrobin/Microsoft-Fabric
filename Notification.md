![image](https://github.com/user-attachments/assets/454285f3-1f11-4119-bbcb-d108e8dddd29)

## if succeed

```json
{
    "name": "Teams1_copy1",
    "type": "Teams",
    "dependsOn": [
        {
            "activity": "Incremental",
            "dependencyConditions": [
                "Succeeded"
            ]
        }
    ],
    "typeProperties": {
        "inputs": {
            "method": "post",
            "path": "/beta/teams/conversation/message/poster/User/location/Channel",
            "body": {
                "recipient": {
                    "groupId": "463aee24-c0c9-4b35-a198-4c9083ecbb1c",
                    "channelId": "19:f9eeba40d8974b6ba9823a0db7c107a4@thread.tacv2"
                },
                "messageBody": "@concat('Copy Activity Passed, Details:',activity('Incremental').output)"
            }
        }
    }
}
```

## if Failed

```json

{
    "name": "Teams1",
    "type": "Teams",
    "dependsOn": [
        {
            "activity": "Incremental",
            "dependencyConditions": [
                "Failed"
            ]
        }
    ],
    "typeProperties": {
        "inputs": {
            "method": "post",
            "path": "/beta/teams/conversation/message/poster/User/location/Channel",
            "body": {
                "recipient": {
                    "groupId": "463aee24-c0c9-4b35-a198-4c9083ecbb1c",
                    "channelId": "19:f9eeba40d8974b6ba9823a0db7c107a4@thread.tacv2"
                },
                "messageBody": "@concat('Copy Activity Failed, Details:',activity('Incremental').output)",
                "subject": "Pipeline Fail"
            }
        }
    }
}

```

