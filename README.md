# AWSGreengrassV2Configs
AWSGreengrassV2 Working Config Example


# **COMPONENTS**

# If you run into any issues with the deployments, try to restore previous successful deployment or remove faulty components. Standard components are good to start debugging!
<br>

**Access Keys and GGC_USER / FOR TESTING PURPOSE ONLY!!!**
```
//Linux
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=

//Windows
set AWS_ACCESS_KEY_ID=
set AWS_SECRET_ACCESS_KEY=
set AWS_SESSION_TOKEN=
net user /add ggc_user (random password)
psexec -s cmd /c cmdkey /generic:ggc_user /user:ggc_user /pass:(random password)
```
_Usage of temporary IAM credentials is recommended by AWS_

**Greengrass nucleus - aws.greengrass.Nucleus (Linux)**

```
{
  "reset": [],
  "merge": {
    "iotRoleAlias": "GreengrassV2TokenExchangeRoleAlias",
    "mqtt": {
      "port": 443
    },
    "greengrassDataPlanePort": 443,
    "jvmOptions": "-Xmx512m",
    "runWithDefault": {
      "posixUser": "ggc_user:ggc_group"
    }
  }
}
```
**Greengrass nucleus - aws.greengrass.Nucleus (Windows)**

```
{
    "iotRoleAlias": "GreengrassV2TokenExchangeRoleAlias",
    "mqtt": {
      "port": 8883
    },
    "greengrassDataPlanePort": 443,
    "jvmOptions": "-Xmx512m",
    "runWithDefault": {
      "windowsUser": "ggc_user"
  }
}
```
_*Note: It looks that MQTT Broker needs port 8883 to work on Windows system_

**Client device auth - aws.greengrass.clientdevices.Auth**

```
{
  "deviceGroups": {
    "formatVersion": "2021-03-05",
    "definitions": {
      "MyPermissiveDeviceGroup": {
        "selectionRule": "thingName: *",
        "policyName": "MyPermissivePolicy"
      }
    },
    "policies": {
      "MyPermissivePolicy": {
        "AllowAll": {
          "statementDescription": "Allow client devices to perform all actions.",
          "operations": [
            "*"
          ],
          "resources": [
            "*"
          ]
        }
      }
    }
  }
}
```
_*Note: We need more restricted policy_

**MQTT 3.1.1 broker (Moquette) - aws.greengrass.clientdevices.mqtt.Moquette**


```
{
	"moquette": {
		"ssl_port": "443"
	}
}
```
_*Note: It looks that MQTT Broker needs port 8883 to work on Windows system_

**MQTT 5 broker (EMQX) - aws.greengrass.clientdevices.mqtt.EMQX**

```
{
  "emqx": {
    "mqtt.max_packet_size": "268435455",
    "log.to": "console",
    "listener.ssl.external.rate_limit": "",
    "listener.ssl.external.handshake_timeout": "15s",
    "listener.ssl.external.max_conn_rate": "500",
    "log.level": "warning",
    "listener.ssl.external.max_connections": "1024000",
    "listener.ssl.external": "443"
  },
  "accessControl": {
    "aws.greengrass.clientdevices.Auth": {
      "aws.greengrass.clientdevices.mqtt.EMQX:auth:1": {
        "policyDescription": "Access to client device auth operations.",
        "operations": [
          "aws.greengrass#SubscribeToCertificateUpdates",
          "aws.greengrass#VerifyClientDeviceIdentity",
          "aws.greengrass#GetClientDeviceAuthToken",
          "aws.greengrass#AuthorizeClientDeviceAction"
        ],
        "resources": [
          "*"
        ]
      }
    }
  },
  "crtLogLevel": "",
  "requiresPrivilege": "true",
  "restartIdentifier": "",
  "startupTimeoutSeconds": "90",
  "ipcTimeoutSeconds": "5"
}
```
_*Note: It looks that MQTT Broker needs port 8883 to work on Windows system_

**MQTT bridge - aws.greengrass.clientdevices.mqtt.Bridge**

```
{
  "reset": [],
  "merge": {
    "mqttTopicMapping": {
      "ClientSt099StData": {
        "topic": "st099/stationdata",
        "source": "LocalMqtt",
        "target": "IotCore"
      },
      "ClientSt099ShdMsg": {
        "topic": "st099/shadowmsg",
        "source": "IotCore",
        "target": "LocalMqtt"
      },
      "ClientSt100StData": {
        "topic": "st100/stationdata",
        "source": "LocalMqtt",
        "target": "IotCore"
      },
      "ClientSt100ShdMsg": {
        "topic": "st100/shadowmsg",
        "source": "IotCore",
        "target": "LocalMqtt"
      },
      "ClientXtsStData": {
        "topic": "xts/stationdata",
        "source": "LocalMqtt",
        "target": "IotCore"
      },
      "ClientXtsShdMsg": {
        "topic": "xts/shadowmsg",
        "source": "IotCore",
        "target": "LocalMqtt"
      },
      "ClientDeviceEvents": {
        "topic": "clients/+/detections",
        "targetTopicPrefix": "events/input/",
        "source": "LocalMqtt",
        "target": "Pubsub"
      },
      "ClientDeviceCloudStatusUpdate": {
        "topic": "clients/+/status",
        "targetTopicPrefix": "$aws/rules/StatusUpdateRule/",
        "source": "LocalMqtt",
        "target": "IotCore"
      },
      "GreengrassTelemetry": {
        "topic": "$local/greengrass/telemetry",
        "source": "LocalMqtt",
        "target": "IotCore"
      }
    },
    "brokerUri": "ssl://localhost:443"
  }
}
```
_*Note: It looks that MQTT Broker needs port 8883 to work on Windows system_

**IP detector - aws.greengrass.clientdevices.IPDetector**

```
{
    "defaultPort": "443",
    "includeIPv4LinkLocalAddrs": "false"
}
```
_*Note: It looks that MQTT Broker needs port 8883 to work on Windows system_

**Secret Manager - aws.greengrass.SecretManager **

```
{
  "reset": [],
  "merge": {
    "cloudSecrets": [
      {
        "arn": "..."
      }
    ]
  }
}
```

**Stream Manager- aws.greengrass.StreamManager  **

```
{
  "reset": [],
  "merge": {
    "STREAM_MANAGER_THREAD_POOL_SIZE": "4"
  }
}
```

**Log Manager - aws.greengrass.LogManager**

```
{
  "reset": [],
  "merge": {
    "logsUploaderConfiguration": {
      "systemLogsConfiguration": {
        "uploadToCloudWatch": "true",
        "minimumLogLevel": "INFO",
        "diskSpaceLimit": "100",
        "diskSpaceLimitUnit": "MB",
        "deleteLogFileAfterCloudUpload": "false"
      },
      "componentLogsConfigurationMap": {
        "aws.greengrass": {
          "logFileRegex": "aws.greengrass.\\w*.log",
          "deleteLogFileAfterCloudUpload": "false"
        },
        "aws.iot": {
          "logFileRegex": "aws.iot.\\w*.log",
          "deleteLogFileAfterCloudUpload": "false"
        }
      }
    },
    "periodicUploadIntervalSec": "30"
  }
}
```

**NucleusEmitter - aws.greengrass.telemetry.NucleusEmitter**

```
{
  "reset": [],
  "merge": {
    "pubSubPublish": "true",
    "mqttTopic": "pi/greengrasscore",
    "telemetryPublishIntervalMs": 5000
  }
}
```
**Secure Tunneling - aws.greengrass.SecureTunneling**

```
{
  "reset": [],
  "merge": {
    "accessControl": {
      "aws.greengrass.ipc.mqttproxy": {
        "aws.greengrass.SecureTunneling:mqttproxy:1": {
          "policyDescription": "Access to tunnel notification pubsub topic",
          "operations": [
            "aws.greengrass#SubscribeToIoTCore"
          ],
          "resources": [
            "$aws/things/+/tunnels/notify"
          ]
        }
      }
    },
    "OS_DIST_INFO": "auto"
  }
}
```


# **RULES & POLICIES**

**Service role needs to be created in CLI and attached in IoT Core settings...**
[https://docs.aws.amazon.com/greengrass/v1/developerguide/service-role.html](https://docs.aws.amazon.com/greengrass/v1/developerguide/service-role.html)

**GreengrassV2IoTThingPolicy in IoT Core Policy**


```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iot:Connect",
        "iot:Publish",
        "iot:Subscribe",
        "iot:Receive",
        "greengrass:*"
      ],
      "Resource": "*"
    }
  ]
}
```
_*Note: We need more restricted policy_

**GreengrassV2CertificatePolicyTokenExchangeRoleAlias in IoT Core Policy**

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:AssumeRoleWithCertificate",
      "Resource": "arn:aws:iam::602730180655:role/GreengrassV2TokenExchangeRole"
    }
  ]
}
```
**GreengrassV2CertificatePolicyTokenExchangeRoleAlias in IAM (Remember to add proper Alias in IoT Core)**

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:DescribeLogStreams",
                "s3:GetBucketLocation"
            ],
            "Resource": "*"
        }
    ]
}
```

[Greengrass Best Practices](https://docs.aws.amazon.com/greengrass/v2/developerguide/security-best-practices.html)
