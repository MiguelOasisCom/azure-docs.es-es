---
title: "Creación de grupos de seguridad de red en el modo ARM mediante una plantilla | Microsoft Docs"
description: Aprenda a crear e implementar grupos de seguridad de red en ARM mediante una plantilla
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
tags: azure-resource-manager
ms.assetid: f3e7385d-717c-44ff-be20-f9aa450aa99b
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: jdial
translationtype: Human Translation
ms.sourcegitcommit: 3fe204c09eebf7d254a1bf2bb130e2d3498b6b45
ms.openlocfilehash: 5e5a0283fee79b9068784ad88017e96d3ab8e729


---
# <a name="how-to-create-nsgs-using-a-template"></a>Creación de grupos de seguridad de red mediante una plantilla
[!INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[!INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Este artículo trata sobre el modelo de implementación del Administrador de recursos. También puede [crear grupos de seguridad de red en el modelo de implementación clásica](virtual-networks-create-nsg-classic-ps.md).

[!INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

## <a name="nsg-resources-in-a-template-file"></a>Recursos de grupo de seguridad de red en un archivo de plantilla
Puede ver y descargar la [plantilla de ejemplo](https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/NSGs.json).

En la sección siguiente se muestra la definición del grupo de seguridad de red de front-end, según el escenario.

```json
"apiVersion": "2015-06-15",
"type": "Microsoft.Network/networkSecurityGroups",
"name": "[parameters('frontEndNSGName')]",
"location": "[resourceGroup().location]",
"tags": {
  "displayName": "NSG - Front End"
},
"properties": {
  "securityRules": [
    {
      "name": "rdp-rule",
      "properties": {
        "description": "Allow RDP",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "destinationPortRange": "3389",
        "sourceAddressPrefix": "Internet",
        "destinationAddressPrefix": "*",
        "access": "Allow",
        "priority": 100,
        "direction": "Inbound"
      }
    },
    {
      "name": "web-rule",
      "properties": {
        "description": "Allow WEB",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "destinationPortRange": "80",
        "sourceAddressPrefix": "Internet",
        "destinationAddressPrefix": "*",
        "access": "Allow",
        "priority": 101,
        "direction": "Inbound"
      }
    }
  ]
}
```
Para asociar el grupo de seguridad de red a la subred front-end, debe cambiar la definición de la subred en la plantilla y utilizar el identificador de referencia para el grupo de seguridad de red.

```json
"subnets": [
  {
    "name": "[parameters('frontEndSubnetName')]",
    "properties": {
      "addressPrefix": "[parameters('frontEndSubnetPrefix')]",
      "networkSecurityGroup": {
      "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('frontEndNSGName'))]"
      }
    }
  }, 
```

Se hace lo mismo para el grupo de seguridad de red de back-end y la subred back-end en la plantilla.

## <a name="deploy-the-arm-template-by-using-click-to-deploy"></a>Implementar la plantilla ARM por medio de un solo clic para implementar
La plantilla de ejemplo disponible en el repositorio público usa un archivo de parámetros que contiene los valores predeterminados utilizados para generar el escenario descrito anteriormente. Para implementar esta plantilla mediante el método de hacer clic para implementar, siga [esta plantilla](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd-NSG), haga clic en **Implementar en Azure**, reemplace los valores de parámetro predeterminados si es necesario y siga las instrucciones del portal.

## <a name="deploy-the-arm-template-by-using-powershell"></a>Implementar la plantilla ARM mediante PowerShell
Para implementar la plantilla ARM que descargó con PowerShell, siga estos pasos.

1. Si no ha usado nunca Azure PowerShell, consulte las instrucciones de [Cómo instalar y configurar Azure PowerShell](/powershell/azureps-cmdlets-docs) para instalarlo y configurarlo.
2. Ejecute el cmdlet **`New-AzureRmResourceGroup`** para crear un grupo de recursos con esta plantilla.

    ```powershell
    New-AzureRmResourceGroup -Name TestRG -Location uswest `
    -TemplateFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' `
    -TemplateParameterFile 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'
    ```

    Resultado esperado:

        ResourceGroupName : TestRG
        Location          : westus
        ProvisioningState : Succeeded
        Tags              :
        Permissions       :
                            Actions  NotActions
                            =======  ==========
                            *                  
   
        Resources         :
                            Name                Type                                     Location
                            ==================  =======================================  ========
                            sqlAvSet            Microsoft.Compute/availabilitySets       westus  
                            webAvSet            Microsoft.Compute/availabilitySets       westus  
                            SQL1                Microsoft.Compute/virtualMachines        westus  
                            SQL2                Microsoft.Compute/virtualMachines        westus  
                            Web1                Microsoft.Compute/virtualMachines        westus  
                            Web2                Microsoft.Compute/virtualMachines        westus  
                            TestNICSQL1         Microsoft.Network/networkInterfaces      westus  
                            TestNICSQL2         Microsoft.Network/networkInterfaces      westus  
                            TestNICWeb1         Microsoft.Network/networkInterfaces      westus  
                            TestNICWeb2         Microsoft.Network/networkInterfaces      westus  
                            NSG-BackEnd         Microsoft.Network/networkSecurityGroups  westus  
                            NSG-FrontEnd        Microsoft.Network/networkSecurityGroups  westus  
                            TestPIPSQL1         Microsoft.Network/publicIPAddresses      westus  
                            TestPIPSQL2         Microsoft.Network/publicIPAddresses      westus  
                            TestPIPWeb1         Microsoft.Network/publicIPAddresses      westus  
                            TestPIPWeb2         Microsoft.Network/publicIPAddresses      westus  
                            TestVNet            Microsoft.Network/virtualNetworks        westus  
                            testvnetstorageprm  Microsoft.Storage/storageAccounts        westus  
                            testvnetstoragestd  Microsoft.Storage/storageAccounts        westus  
   
        ResourceId        : /subscriptions/[Subscription Id]/resourceGroups/TestRG

## <a name="deploy-the-arm-template-by-using-the-azure-cli"></a>Implementar la plantilla ARM mediante la CLI de Azure
Para implementar la plantilla ARM mediante la CLI de Azure, siga estos pasos.

1. Si nunca ha usado la CLI de Azure, consulte [Instalación y configuración de la CLI de Azure](../xplat-cli-install.md) y siga las instrucciones hasta el punto donde deba seleccionar su cuenta y suscripción de Azure.
2. Ejecute el comando **`azure config mode`** para cambiar al modo de Administrador de recursos, como se muestra a continuación.

    ```azurecli
    azure config mode arm
    ```

    Se espera esta salida del comando:

        info:    New mode is arm

3. Ejecute el cmdlet **`azure group deployment create`** para implementar la nueva red virtual mediante la plantilla y los archivos de parámetros que descargó y modificó anteriormente. En la lista que se muestra en la salida se explican los parámetros utilizados.

    ```azurecli
    azure group create -n TestRG -l westus -f 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.json' -e 'https://raw.githubusercontent.com/telmosampaio/azure-templates/master/201-IaaS-WebFrontEnd-SQLBackEnd/azuredeploy.parameters.json'
    ```

    Resultado esperado:
   
        info:    Executing command group create
        info:    Getting resource group TestRG
        info:    Creating resource group TestRG
        info:    Created resource group TestRG
        info:    Initializing template configurations and parameters
        info:    Creating a deployment
        info:    Created template deployment "azuredeploy"
        data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG
        data:    Name:                TestRG
        data:    Location:            westus
        data:    Provisioning State:  Succeeded
        data:    Tags: null
        data:    
        info:    group create command OK
   
   * **-n (o --name)**. Nombre del grupo de recursos que se va a crear.
   * **-l (o --location)**. Región de Azure donde se creará el nuevo grupo de recursos.
   * **-f (o --template-file)**. Ruta de acceso al archivo de plantilla ARM.
   * **-e (o --parameters-file)**. Ruta de acceso al archivo de parámetros ARM.




<!--HONumber=Dec16_HO1-->


