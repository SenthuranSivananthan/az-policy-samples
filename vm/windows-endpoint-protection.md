## Audit for virtual machines that doesn't either have Endpoint Protection or Endpoint Protection is in a non-healthy status

**Notes**:

1.  Policy will only scan for virtual machines that are running.
2.  The signature for VM image can change over time as new operating systems are released.
3.  Endpoint assessment is executed by [Azure Security Center based on these rules](https://docs.microsoft.com/en-us/azure/security-center/security-center-endpoint-protection).

```json
{
    "mode": "All",
    "policyRule": {
        "if": {
            "allOf": [
                {
                    "field": "type",
                    "equals": "Microsoft.Compute/virtualMachines"
                },
                {
                    "field": "Microsoft.Compute/virtualMachines/instanceView.statuses[*].code",
                    "notEquals": "PowerState/deallocated"
                },
                {
                    "anyOf": [
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftWindowsServer"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "WindowsServer"
                                },
                                {
                                    "field": "Microsoft.Compute/imageSKU",
                                    "in": [
                                        "2008-R2-SP1",
                                        "2008-R2-SP1-smalldisk",
                                        "2012-Datacenter",
                                        "2012-Datacenter-smalldisk",
                                        "2012-R2-Datacenter",
                                        "2012-R2-Datacenter-smalldisk",
                                        "2016-Datacenter",
                                        "2016-Datacenter-Server-Core",
                                        "2016-Datacenter-Server-Core-smalldisk",
                                        "2016-Datacenter-smalldisk",
                                        "2016-Datacenter-with-Containers",
                                        "2016-Datacenter-with-RDSH",
                                        "2019-Datacenter",
                                        "2019-Datacenter-Core",
                                        "2019-Datacenter-Core-smalldisk",
                                        "2019-Datacenter-Core-with-Containers",
                                        "2019-Datacenter-Core-with-Containers-smalldisk",
                                        "2019-Datacenter-smalldisk",
                                        "2019-Datacenter-with-Containers",
                                        "2019-Datacenter-with-Containers-smalldisk",
                                        "2019-Datacenter-zhcn"
                                    ]
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftWindowsServer"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "WindowsServerSemiAnnual"
                                },
                                {
                                    "field": "Microsoft.Compute/imageSKU",
                                    "in": [
                                        "Datacenter-Core-1709-smalldisk",
                                        "Datacenter-Core-1709-with-Containers-smalldisk",
                                        "Datacenter-Core-1803-with-Containers-smalldisk"
                                    ]
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftWindowsServerHPCPack"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "WindowsServerHPCPack"
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftSQLServer"
                                },
                                {
                                    "anyOf": [
                                        {
                                            "field": "Microsoft.Compute/imageOffer",
                                            "like": "*-WS2016"
                                        },
                                        {
                                            "field": "Microsoft.Compute/imageOffer",
                                            "like": "*-WS2016-BYOL"
                                        },
                                        {
                                            "field": "Microsoft.Compute/imageOffer",
                                            "like": "*-WS2012R2"
                                        },
                                        {
                                            "field": "Microsoft.Compute/imageOffer",
                                            "like": "*-WS2012R2-BYOL"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftRServer"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "MLServer-WS2016"
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftVisualStudio"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "in": [
                                        "VisualStudio",
                                        "Windows"
                                    ]
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftDynamicsAX"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "Dynamics"
                                },
                                {
                                    "field": "Microsoft.Compute/imageSKU",
                                    "equals": "Pre-Req-AX7-Onebox-U8"
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "microsoft-ads"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "windows-data-science-vm"
                                }
                            ]
                        },
                        {
                            "allOf": [
                                {
                                    "field": "Microsoft.Compute/imagePublisher",
                                    "equals": "MicrosoftWindowsDesktop"
                                },
                                {
                                    "field": "Microsoft.Compute/imageOffer",
                                    "equals": "Windows-10"
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        "then": {
            "effect": "[parameters('effect')]",
            "details": {
                "type": "Microsoft.Security/complianceResults",
                "name": "endpointProtection",
                "existenceCondition": {
                    "field": "Microsoft.Security/complianceResults/resourceStatus",
                    "in": [
                        "OffByPolicy",
                        "Healthy"
                    ]
                }
            }
        }
    },
    "parameters": {
        "effect": {
            "type": "String",
            "metadata": {
                "displayName": "Effect",
                "description": "Enable or disable the execution of the policy"
            },
            "allowedValues": [
                "AuditIfNotExists",
                "Disabled"
            ],
            "defaultValue": "AuditIfNotExists"
        }
    }
}
```
