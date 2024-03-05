# Notes for Cloud Direct deployment

## References

* <https://azuremarketplace.microsoft.com/en-us/marketplace/apps/fortinet.fortinet-fortigate?tab=PlansAndPrice>
* <https://github.com/fortinet/azure-templates/tree/main/FortiGate/Active-Passive-SDN>

## Accept Marketplace for PAYG

The terms for the FortiGate PAYG or BYOL image in the Azure Marketplace needs to be accepted once before usage. This is done automatically during deployment via the Azure Portal. For the Azure CLI the commands below need to be run before the first deployment in a subscription.
  - BYOL
    ```shell
    az vm image terms accept --publisher fortinet --offer fortinet_fortigate-vm_v5 --plan fortinet_fg-vm
    ```
  - PAYG
    ```shell
    az vm image terms accept --publisher fortinet --offer fortinet_fortigate-vm_v5 --plan fortinet_fg-vm_payg_2023
    ```

## Deploy

The deploy.sh has been customised. The original file is shown :

| **Parameter** | **Default Value** |
|---|---|
| location | AZURE_DEFAULTS_LOCATION or uksouth |
| rg | AZURE_DEFAULTS_GROUP or rg-fortigate |
| prefix | fortigate |
| adminUsername | fortigateadmin |
| adminPassword | needs setting |

I used novus for the prefix and '<Std>novus'

The deploy line has also been configured with the following additional parameters:

```shell
fortiGateImageSKU="fortinet_fg-vm_payg_2023"
availabilityOptions="Availability Zones"
```

## Post deployment outputs

```text
novus-FGT-A	4.250.49.184	172.16.136.4    WAN
novus-FGT-A	None	        172.16.136.68
novus-FGT-A	None	        172.16.136.132
novus-FGT-A	172.166.80.62	172.16.136.196  MGMT-A
novus-FGT-B	None	        172.16.136.5
novus-FGT-B	None	        172.16.136.69
novus-FGT-B	None	        172.16.136.133
novus-FGT-B	4.250.49.141	172.16.136.197  MGMT-B
```

## Post config

SSH

```shell
ssh fortigateadmin@172.166.80.62
```

GUI

<https://172.166.80.62>

Add role assignments to the system assigned managed identities

```shell
export AZURE_DEFAULTS_GROUP=novus
prefix=novus

scope=/subscriptions/$(az account show --query id -otsv)
identityA=$(az vm identity show --name ${prefix}-FGT-A --query principalId -otsv)
identityB=$(az vm identity show --name ${prefix}-FGT-B --query principalId -otsv)
az role assignment create --assignee-principal-type ServicePrincipal --role Reader --scope $scope --assignee-object-id $identityA
az role assignment create --assignee-principal-type ServicePrincipal --role Reader --scope $scope --assignee-object-id $identityB
```