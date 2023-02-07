# Private Preview Phase 1

## Limitations and Prerequists
* Compute Instance with Public IP is the only supported compute type.
* Make sure your subscription is allowlisted.

## What you will get

![](images/prprph1.png)

You can have an AzureML managed VNet with two configuration types.
* **AllowInternetOutbound**: Allow all internet oubound from AzureML managed VNet. You can have private endpoint connections to your private Azure resources.
* **AllowOnlyApprovedOutbound**: You can allow outbound only to the approved outbound. You can allow outbound using private endpoint, FQDN and service tag.

## CLI setup
1. Remove your Azure CLI AzureML extension if you have.

```python
az extension remove -n ml
```

2. Install CLI ML extension for private preview

```python
az extension add --source https://azuremlsdktestpypi.blob.core.windows.net/wheels/sdk-cli-v2/ml-0.0.82438729-py3-none-any.whl
```

## Create your new workspace (do not test this with your existing workspaces)

1. Create a workspace without managed network isolation

```python
az login
az account set -s <subscriptionId>
az group create -g <new_rg_name> 
az ml workspace create -n <new_ws_name> -g <rg_name> --location centraluseuap --managed-network Disabled
```

2. Enable manage network isolation

```python
az ml workspace update -n <ws_name> -g <rg_name> --managed-network AllowInternetOutbound
```
or
```python
az ml workspace update -n <ws_name> -g <rg_name> --managed-network AllowOnlyApprovedOutbound
```

## Create your compute intance

```python
az ml compute create --file computeInstance.yml --resource-group <rg_name> --workspace-name <ws_name> 
```

## Connect to the compute instance using SSH

```python
az ml compute connect-ssh --name <ci_name>--resource-group <rg_name> --workspace-name <ws_name> --private-key-file-path <your sshkey path>
```

## User jupyter notebook on the compute instance

TBU

## Create private endpoints to access your private storage

TBU

## Create FQDN outbound rule

TBU

## Clean up the workspace

```python
az ml workspace delete --resource-group <rg_name> --name <ws_name>
```
