# Private Preview Phase 1

## Limitations
* Compute Instance **with Public IP** is the only supported compute type.
* Supported region is **eastus** only.
* allow_only_approved_outbound is not supported.
* FQDN outbound is not supported.
* Default PE to ACR/Monitor is not supported.
* Make sure your subscription is allowlisted.

## What you will get

![prprph1 network architecture](prprph1.png)

You can have an AzureML managed VNet with two configuration types.
* **allow_internet_outbound**: Allow all internet oubound from AzureML managed VNet. You can have private endpoint connections to your private Azure resources.
* **allow_only_approved_outbound**: You can allow outbound only to the approved outbound. You can allow outbound using private endpoint, FQDN(will be available) and service tag.

## CLI setup
1. Remove your Azure CLI AzureML extension if you have.

```Azure CLI
az extension remove -n ml
```

2. Install CLI ML extension for private preview

```Azure CLI
az extension add --source https://azuremlsdktestpypi.blob.core.windows.net/wheels/sdk-cli-v2/ml-0.0.88584999-py3-none-any.whl
```

## Create your new workspace (do not test this with your existing workspaces)

1. Create a workspace without managed network isolation

```Azure CLI
az login
az account set -s <subscriptionId>
az group create -g <new_rg_name>
az configure -d group=<new_rg_name> location=eastus
az ml workspace create -n <new_ws_name> -g <rg_name> --location eastus --managed-network disabled
```

2. Enable manage network isolation

**allow_internet_outbound**: Allow all internet oubound from AzureML managed VNet. You can have private endpoint connections to your private Azure resources.

```Azure CLI
az ml workspace update -n <ws_name> -g <rg_name> --managed-network allow_internet_outbound
```

<!---
or

**allow_only_approved_outbound**: You can allow outbound only to the approved outbound. You can allow outbound using private endpoint, FQDN(will be available) and service tag.

```Azure CLI
az ml workspace update -n <ws_name> -g <rg_name> --managed-network allow_only_approved_outbound
```
--->

## Create private endpoint outbound rule to access your private storage
You can create private endpoints to access your private resources. Below is an example to create a PE for an Azure storage.

```Azure CLI
az ml workspace update --file peoutbound.yml --resource-group MyGroup
```
You can find a sample [peoutbound.yml](peoutbound.yml)

```YAML
name: MyWorkspace
managed_network:
  outbound_rules:
    MyStorage:
      type: PrivateEndpoint
      destination:
        service_resource_id: "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/MyGroup/providers/Microsoft.Storage/storageAccounts/MyAccount"
        subresource_target: "blob"
```

## Create your compute intance

**Note that your first CI creation takes 10 mins because we need to initiate multiple private endpoints**

Use [computeinstance.yml](computeinstance.yml) with your compute instance name and SSH key.
```Azure CLI
az ml compute create --file computeInstance.yml --resource-group <rg_name> --workspace-name <ws_name> 
```

You can create and copy your SSH key if you do not have it.

```CLI
ssh-keygen -m PEM -t rsa -b 4096
cat ~/.ssh/id_rsa.pub
```

## Use Notebook of your Compute Instance

Go to your Workspace/Notebook or Workspace/Compute/Compute Instance/Jupyter to test python SDK using sample notebooks. We expect you testing below scenarios.

1. Run several example notebooks in https://github.com/Azure/azureml-examples/tree/main/sdk/python and if it works without issues.
2. Run your own jupyter notebook with your dataset in your private storage account. Note that you need to create a PE to your private storage mentioned in the above.

## Optional: Connect to the compute instance using SSH

```Azure CLI
az ml compute connect-ssh --name <ci_name>--resource-group <rg_name> --workspace-name <ws_name> --private-key-file-path <your sshkey path>
```

## Optional: Confirm private endpoint connection to your default resources (storage, KV)

You can check private endpoint connections on Azure portal. You can see private endpoints after your first compute creation. PE connection to ACR/Monitor will come.

![storage pe](storagepe.png)

<!---
## Optional: Create a service tag outbound

```Azure CLI
az ml workspace outbound-rule set --resource-group MyGroup --workspace-name MyWorkspace --rule MyAzureSerivce --type ServiceTag --service-tag DataFactory --port-ranges "80, 8080-8089" --protocol TCP
```
or
```Azure CLI
az ml workspace update --file servicetag.yml --resource-group MyGroup
```
You can find a sample [servicetag.yml](servicetag.yml).

```YAML
name: "MyWorkspace"
managed_network:
  outbound_rules:
    MyAzureSerivce:
      type: "ServiceTag"
      destination:
        service_tag: "DataFactory"
        port_ranges: "80, 8080-8089"
        protocol: "TCP"
```
--->

## File a bug if you have any problems
Submit a issue details via https://forms.office.com/r/5WpJGk9jK0.
AzureML team will set periodical meetings to discuss issues if necessaary.

## Clean up the environment

```python
az group delete -n <rg_name>
```
