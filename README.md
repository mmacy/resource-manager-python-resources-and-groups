---
services: azure-resource-manager
platforms: python
author: lmazuel
---

# Manage Azure resources and resource groups with Python

This sample explains how to manage your
[resources and resource groups in Azure](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-groups)
using the Azure Python SDK.

**On this page**

- [Run this sample](#run)
- [What is example.py doing?](#example)
    - [List resource groups](#list-groups)
    - [Create a resource group](#create-group)
    - [Update a resource group](#update-group)
    - [Create a key vault in the resource group](#create-resource)
    - [List resources within the group](#list-resources)
    - [Export the resource group template](#export)
    - [Delete a resource group](#delete-group)

<a id="run"></a>
## Run this sample

1. If you don't already have it, [install Python](https://www.python.org/downloads/).

1. We recommend to use a [virtual environment](https://docs.python.org/3/tutorial/venv.html) to run this example, but it's not mandatory. You can initialize a virtualenv this way:

    ```
    pip install virtualenv
    virtualenv mytestenv
    cd mytestenv
    source bin/activate
    ```

1. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/resource-manager-python-resources-and-groups.git
    ```

1. Install the dependencies using pip.

    ```
    cd resource-manager-python-resources-and-groups
    pip install -r requirements.txt
    ```

1. Create an Azure service principal either through
[Azure CLI](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal-cli/),
[PowerShell](https://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal/)
or [the portal](https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/).

1. Export these environment variables into your current shell. 

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    ```

1. Run the sample.

    ```
    python example.py
    ```

<a id="example"></a>
## What is example.py doing?

The sample walks you through several resource and resource group management operations.
It starts by setting up a ResourceManagementClient object using your subscription and credentials.

```python
import os
from azure.common.credentials import ServicePrincipalCredentials
from azure.mgmt.resource import ResourceManagementClient

subscription_id = os.environ.get(
    'AZURE_SUBSCRIPTION_ID',
    '11111111-1111-1111-1111-111111111111') # your Azure Subscription Id
credentials = ServicePrincipalCredentials(
    client_id=os.environ['AZURE_CLIENT_ID'],
    secret=os.environ['AZURE_CLIENT_SECRET'],
    tenant=os.environ['AZURE_TENANT_ID']
)
client = ResourceManagementClient(credentials, subscription_id)
```

It also sets up a ResourceGroup object (resource_group_params) to be used as a parameter in some of the API calls.

```python
resource_group_params = {'location':'westus'}
```

There are a couple of supporting functions (`print_item` and `print_properties`) that print a resource group and its properties.
With that set up, the sample lists all resource groups for your subscription, it performs these operations.

<a id="list-groups"></a>
### List resource groups

List the resource groups in your subscription.

```python
for item in client.resource_groups.list():
    print_item(item)
```

<a id="create-group"></a>
### Create a resource group

```python
client.resource_groups.create_or_update('azure-sample-group', resource_group_params)
```

<a id="update-group"></a>
### Update a resource group

The sample adds a tag to the resource group.

```python
resource_group_params.update(tags={'hello': 'world'})
client.resource_groups.create_or_update('azure-sample-group', resource_group_params)
```

<a id="create-resource"></a>
### Create a key vault in the resource group

```python
key_vault_params = {
    'location': 'westus',
    'properties':  {
        'sku': {'family': 'A', 'name': 'standard'},
        'tenantId': os.environ['AZURE_TENANT_ID'],
        'accessPolicies': [],
        'enabledForDeployment': True,
        'enabledForTemplateDeployment': True,
        'enabledForDiskEncryption': True
    }
}
client.resources.create_or_update(GROUP_NAME,
                                  'Microsoft.KeyVault',
                                  '',
                                  'vaults',
                                  'azureSampleVault',
                                  '2015-06-01',
                                  key_vault_params)
```

<a id="list-resources"></a>
### List resources within the group

```python
for item in client.resource_groups.list_resources(GROUP_NAME):
    print_item(item)
```

<a id="export"></a>
### Export the resource group template

```python
client.resource_groups.export_template(GROUP_NAME, ['*'])
```

<a id="delete-group"></a>
### Delete a resource group

```python
delete_async_operation = client.resource_groups.delete('azure-sample-group')
delete_async_operation.wait()
```
