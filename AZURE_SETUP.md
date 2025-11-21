# Azure Setup Guide

This guide will help you set up Azure credentials for GitHub Actions deployment.

## Prerequisites

- Azure CLI installed
- An active Azure subscription
- GitHub repository with appropriate permissions


## Step 1: Create a Service Principal

Run the following command in Azure CLI to create a service principal with Contributor role:

```bash
az ad sp create-for-rbac \
  --name "github-actions-bicep-deploy" \
  --role contributor \
  --scopes /subscriptions/<YOUR_SUBSCRIPTION_ID> \
  --sdk-auth
```

Replace `<YOUR_SUBSCRIPTION_ID>` with your actual Azure subscription ID.

You can find your subscription ID by running:
```bash
az account show --query id -o tsv
```

## Step 2: Get the Required Values

The command will output JSON similar to this:

```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "your-client-secret",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Step 3: Configure Federated Credentials (Recommended for OIDC)

For better security, use OIDC instead of client secrets:

```bash
# Get your subscription ID
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Create service principal
APP_ID=$(az ad sp create-for-rbac \
  --name "github-actions-bicep-deploy" \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID \
  --query clientId -o tsv)

# Add federated credential for main branch
az ad app federated-credential create \
  --id $APP_ID \
  --parameters '{
    "name": "github-main",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:YOUR_GITHUB_USERNAME/bicep-ephemeral-env:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'

# Add federated credential for pull requests
az ad app federated-credential create \
  --id $APP_ID \
  --parameters '{
    "name": "github-pr",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:YOUR_GITHUB_USERNAME/bicep-ephemeral-env:pull_request",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username or organization name.

## Step 4: Add Secrets to GitHub

1. Go to your GitHub repository
2. Navigate to **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret** and add the following secrets:

- `AZURE_CLIENT_ID`: The clientId from the JSON output
- `AZURE_TENANT_ID`: The tenantId from the JSON output
- `AZURE_SUBSCRIPTION_ID`: The subscriptionId from the JSON output

**Note**: If you're using OIDC (federated credentials), you don't need to add `AZURE_CLIENT_SECRET`.

## Step 5: Verify the Setup

1. Push changes to the main branch or create a pull request
2. Check the Actions tab in your GitHub repository
3. The workflow should run and deploy your infrastructure to Azure

## Troubleshooting

### Permission Denied

If you get permission errors, ensure the service principal has the correct role:

```bash
az role assignment create \
  --assignee <CLIENT_ID> \
  --role Contributor \
  --scope /subscriptions/<SUBSCRIPTION_ID>
```

### Resource Provider Not Registered

Register required resource providers:

```bash
az provider register --namespace Microsoft.Web
az provider register --namespace Microsoft.Storage
```

## Cleanup

To delete the service principal when no longer needed:

```bash
az ad sp delete --id <CLIENT_ID>
```

## Additional Resources

- [Azure Login Action Documentation](https://github.com/Azure/login)
- [Azure ARM Deploy Action](https://github.com/Azure/arm-deploy)
- [Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
