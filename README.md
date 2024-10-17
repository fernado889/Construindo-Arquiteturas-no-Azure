# Construindo-Arquiteturas-no-Azure
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "[parameters('vnetName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[parameters('vnetAddressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[parameters('subnetName')]",
            "properties": {
              "addressPrefix": "[parameters('subnetAddressPrefix')]"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_DS1_v2"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('nicName'))]"
            }
          ]
        }
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-02-01",
      "name": "[parameters('nicName')]",
      "location": "[parameters('location')]",
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName')), '/subnets/', parameters('subnetName'))]"
              },
              "privateIPAllocationMethod": "Dynamic"
            }
          }
        ]
      }
    }
  ],
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "vnetName": {
      "type": "string",
      "metadata": {
        "description": "Name of the virtual network."
      }
    },
    "vnetAddressPrefix": {
      "type": "string",
      "metadata": {
        "description": "Address space for the virtual network."
      }
    },
    "subnetName": {
      "type": "string",
      "metadata": {
        "description": "Name of the subnet."
      }
    },
    "subnetAddressPrefix": {
      "type": "string",
      "metadata": {
        "description": "Address prefix for the subnet."
      }
    },
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "Name of the virtual machine."
      }
    },
    "nicName": {
      "type": "string",
      "metadata": {
        "description": "Name of the network interface."
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username for the virtual machine."
      }
    },
    "adminPassword": {
      "type": "secureString",
      "metadata": {
        "description": "Admin password for the virtual machine."
      }
    }
  }
}
