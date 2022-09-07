# Create a Container App On Azure
Azure Container Apps enables you to run microservices and containerized applications on a serverless platform. 
Common use cases include: Deploying API endpoints, Hosting background processing applications and more.

# Azure CLI Login
First we need to check you are logged in to the Azure in the CLI. The following command will check to see if you are logged in. 
If not it will open a browser and take you through the login steps.

To Login to Az CLI and select a subscription 
`az login` followed by `az account list --output table` and `az account set --subscription "name of subscription to use"`

To Install Az CLI
If you need to install Azure CLI run the following command: curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Step 1 - Install Azure CLI Extension
The Azure CLI offers the capability to load extensions. 
Extensions allow you gain access to experimental and pre-release commands.

Currently, Container App is in preview so it requries an extension.
```bash
az extension add --name containerapp
```

# Step 2 - Register Resource Providers
Resources are manageable items available through Azure like virtual machines or storage accounts. Resource providers supply Azure resources. 
Microsoft.App is a resource provider for Contianer Apps.
Microsoft.OperationalInsights is a resource for Azure Monitor.

The `--wait` parameter delays the next instruction until the command is completed.
```bash
az provider register --namespace Microsoft.App --wait
```
```bash
az provider register --namespace Microsoft.OperationalInsights --wait
```

# Step 3 - Create a resource group
An Azure resource group is a logical group in which Azure resources are deployed and managed. When you create a resource group, you are prompted to specify a location. This location is:
  - The storage location of your resource group metadata.
  - Where your resources will run in Azure if you don't specify another region during resource creation. 

This command uses two environment variables, `RESOURCE_GROUP` is the name of the resource group and will be commonly using in other commands.
`LOCATION` is the data center that the resource group will be created in. 
When this command has completed it will return a JSON file. 

You can see what the variables are set at for this tutorial in that output.
If you want to change them press `b` and run the command export `VARIABLE_NAME="new variable value"`
```bash
echo $LOCATION
```
```bash
echo $RESOURCE_GROUP
```

Validate Resource Group does not already exist. If it does, select a new resource group name by running the following:

```
if [ "$(az group exists --name $RESOURCE_GROUP)" = 'true' ]; then export RAND=$RANDOM; export RESOURCE_GROUP="$RESOURCE_GROUP$RAND"; echo "Your new Resource Group Name is $RESOURCE_GROUP"; fi
```

```bash
az group create --name $RESOURCE_GROUP --location $LOCATION
```
Results:

```expected_similarity=0.5
{
  "id": "/subscriptions/bb318642-28fd-482d-8d07-79182df07999/resourceGroups/testResourceGroup24763",
  "location": "eastus",
  "managedBy": null,
  "name": "testResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```
# Step 4 - Create Azure Container Apps Environment
Individual container apps are deployed to a single Container Apps environment, which acts as a secure boundary around groups of container apps.
Container Apps in the same environment are deployed in the same virtual network and write logs to the same Log Analytics workspace. 
This next command will create a Container App Environment in the Resource Group created in `Step 3`.

**Command will take ~3 minutes to complete.**

You can see what the variables are set at for this tutorial in that output.
If you want to change them press `b` and run the command export `VARIABLE_NAME="new variable value"`

```bash
echo $CONTAINERAPPS_ENVIRONMENT
```
```bash
az containerapp env create --name $CONTAINERAPPS_ENVIRONMENT --resource-group $RESOURCE_GROUP --location $LOCATION
```

# Step 5 - Create Container App with a Public Image
Now that you have an environment created, you can deploy your first container app. 
With the containerapp create command, deploy a container image to Azure Container Apps.
*NOTE: Make sure the value for the --image parameter is in lower case.*
By setting `--ingress` to external, you make the container app available to public requests.

**Command will take ~3 minutes to complete.**

You can see what the variables are set at for this tutorial in that output.
If you want to change them press `b` and run the command export `VARIABLE_NAME="new variable value"`
```bash
echo $GITHUB_USERNAME
```

Update the `GITHUB_USERNAME` environment variable below with your GitHub username or organization name.
    * **Press `b` and run the command `GITHUB_USERNAME="username"` to set variable.**
```bash
echo $GITHUB_USERNAME
```
```bash
CONTAINER_IMAGE=ghcr.io/$GITHUB_USERNAME/serverless-python:release
```
```bash
echo $CONTAINER_APP_NAME
```
```bash
echo $CONTAINER_IMAGE
```
```bash
az containerapp create --name $CONTAINER_APP_NAME --resource-group $RESOURCE_GROUP --environment $CONTAINERAPPS_ENVIRONMENT --image "$CONTAINER_IMAGE" --target-port 80 --ingress 'external'
```

# Step 6 - Test Container App with curl
The `az containerapp show` command returns the fully qualified domain name of a container app.
In the next command we are setting the domain name to the variable `CONTAINERAPP_FQDN`
```bash
CONTAINERAPP_FQDN=$(az containerapp show --resource-group $RESOURCE_GROUP --name $CONTAINER_APP_NAME --query "properties.configuration.ingress.fqdn" --out tsv)
```
```bash
echo "https://$CONTAINERAPP_FQDN"
```
```bash
curl "https://$CONTAINERAPP_FQDN"
```

# Success! You now have scccessfully created a Container Apps image in Azure. 
If you would like to delete the resources created push any button.
If you want to keep the resources created, push `b` and `CTRL + C` to exit the program.

# Step 7 - Delete Resource Group
The Container App and Container App Environment will be deleted with command below.

```bash
az group delete --name $RESOURCE_GROUP --yes
```
