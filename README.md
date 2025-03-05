# delete-azure-machine-learning-endpoint

Github Action to delete an Azure Machine Learning Online Endpoint

Features:

* Delete an Azure Machine Learning Online Endpoint

For other Azure Machine Learning actions check out:

* [create-azure-machine-learning-online-endpoint](https://github.com/coding-kitties/create-azure-machine-learning-online-endpoint)
* [register-azure-machine-learning-model](https://github.com/coding-kitties/register-azure-machine-learning-model)
* [update-azure-machine-learning-online-deployment](https://github.com/coding-kitties/update-azure-machine-learning-online-deploymentl)
* [delete-azure-machine-learning-online-deployment](https://github.com/coding-kitties/delete-azure-machine-learning-online-deployment)

## Dependencies on other Github Actions

* Authenticate using [Azure Login](https://github.com/Azure/login)

## 🚀 Usage

### **1. Add to Your Workflow**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2.3.2

      - uses: Azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Delete AML Online Endpoint
        uses: coding-kitties/delete-azure-machine-learning-endpoint@v0.1.0
        with:
          resource_group: "my-resource-group"
          workspace_name: "my-aml-workspace"
          endpoint_name: "my-endpoint"
```

## Example deployment of an Azure Machine Learning Workflow with blue/green deployments

This example demonstrates an Azure Machine Learning Deployment with blue/green deployments for different environments. We use various Github Actions to create a complete workflow.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2.3.2

      - uses: Azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Move model into dev registry (Will be skipped if it already exists)
      - name: Register model in registry
        uses: coding-kitties/register-azure-machine-learning-model@v0.1.0
        with:
          model_name: 'model-name'
          model_version: '1'
          source_registry_name: 'playground-registry'
          source_registry_resource_group: 'my-registry-resource-group'
          destination_registry_name: 'playground-registry'
          destination_registry_resource_group: 'my-registry-resource-group'

      # Create AML Online Endpoint in DEV (Will be skipped if it already exists)
      - name: Create AML Online Endpoint DEV
        uses: coding-kitties/create-azure-machine-learning-online-endpoint@v0.3.0
        with:
          endpoint_name: 'dev-endpoint'
          resource_group: 'dev-group'
          workspace_name: 'dev-workspace'

      # Deploy the new green model to DEV
      - name: Create AML Online Endpoint Deployment DEV
        uses: coding-kitties/create-azure-machine-learning-online-deployment@v0.3.0
        with:
          endpoint_name: 'dev-endpoint'
          resource_group: 'dev-group'
          workspace_name: 'dev-workspace'
          deployment_yaml_file_path: 'path/to/deployment.yml'
          model_name: 'model-name'
          model_version: '1'
          traffic: '{ "green": 0, "blue": 100, mirror": {"green": 20} }'

      # Update green deployment traffic in DEV
      - name: Update AML Online Endpoint Deployment traffic
        uses: coding-kitties/update-azure-machine-learning-online-deployment@v0.1.0
        with:
          endpoint_name: 'my-endpoint'
          workspace_name: 'my-workspace'
          resource_group: 'my-resource-group'
          traffic: '{ "green": 100, "blue": 0, mirror": {"green": 0} }'

      - name: Delete AML Online Endpoint Deployment DEV
        uses: coding-kitties/delete-azure-machine-learning-online-deployment@v0.1.0
        with:
          endpoint_name: 'dev-endpoint'
          resource_group: 'dev-group'
          workspace_name: 'dev-workspace'
          deployment_name: 'blue'

      # Move model to production registy
      - name: Move model to production registry
        uses: coding-kitties/register-azure-machine-learning-model@v0.1.0
        with:
          model_name: 'model-name'
          model_version: '1'
          source_registry_name: 'playground-registry'
          source_registry_resource_group: 'my-registry-resource-group'
          destination_registry_name: 'production-registry'
          destination_registry_resource_group: 'my-registry-resource-group'

      # Create AML Online Endpoint in PROD (Will be skipped if it already exists)
      - name: Create AML Online Endpoint PROD
        uses: coding-kitties/create-azure-machine-learning-online-endpoint@v0.3.0
        with:
          endpoint_name: 'prod-endpoint'
          resource_group: 'prod-group'
          workspace_name: 'prod-workspace'

      # Deploy the new green model to PROD
      - name: Create AML Online Endpoint Deployment PROD
        uses: coding-kitties/create-azure-machine-learning-online-deployment@v0.3.0
        with:
          endpoint_name: 'prod-endpoint'
          resource_group: 'prod-group'
          workspace_name: 'prod-workspace'
          deployment_yaml_file_path: 'path/to/deployment.yml'
          model_name: 'model-name'
          model_version: '1'
          traffic: '{ "green": 0, "blue": 100, mirror": {"green": 20} }'
```
