# bicep-ephemeral-env

Azure infrastructure deployment using Bicep and GitHub Actions for ephemeral environments.

## Overview

This repository demonstrates how to deploy Azure infrastructure using Bicep templates with GitHub Actions. It includes support for ephemeral environments that are automatically created for pull requests and cleaned up when PRs are closed.

## Infrastructure

The Bicep template (`main.bicep`) deploys:

- **App Service Plan** - Linux-based plan for hosting web applications
- **Web App** - Node.js 18 LTS web application
- **Storage Account** - Standard LRS storage for application data

## Prerequisites

- Azure subscription
- Azure CLI installed locally
- GitHub account with repository access
- Proper Azure credentials configured (see [AZURE_SETUP.md](AZURE_SETUP.md))

## Quick Start

### 1. Configure Azure Credentials

Follow the detailed guide in [AZURE_SETUP.md](AZURE_SETUP.md) to:
- Create a service principal
- Configure OIDC authentication
- Add required secrets to GitHub

### 2. Deploy Infrastructure

The deployment happens automatically through GitHub Actions:

- **Push to main**: Deploys to the `dev` environment
- **Pull request**: Creates an ephemeral environment for testing
- **Manual trigger**: Deploy to any environment (dev/staging/prod)

### 3. Manual Deployment

You can also deploy manually using Azure CLI:

```bash
# Login to Azure
az login

# Create resource group
az group create --name rg-bicep-dev --location eastus

# Deploy Bicep template
az deployment group create \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev location=eastus
```

## GitHub Actions Workflows

### Azure Deploy Workflow

Located in `.github/workflows/azure-deploy.yml`, this workflow:

1. **Validates** the Bicep template
2. **Deploys** infrastructure to Azure
3. **Cleans up** ephemeral environments when PRs are closed

Trigger manually:
1. Go to the **Actions** tab
2. Select **Deploy to Azure**
3. Click **Run workflow**
4. Choose the environment (dev/staging/prod)

## Customization

### Modify Infrastructure

Edit `main.bicep` to add or change Azure resources:

```bicep
// Add new resources
resource newResource 'Microsoft.ResourceType@api-version' = {
  name: resourceName
  location: location
  properties: {
    // resource properties
  }
}
```

### Change Deployment Parameters

Update parameters in `.github/workflows/azure-deploy.yml`:

```yaml
parameters: environmentName=${{ steps.set-env.outputs.env_name }} location=eastus resourcePrefix=myapp
```

### Add Environments

Configure GitHub environments:
1. Go to **Settings** > **Environments**
2. Create new environment (e.g., `staging`, `prod`)
3. Add protection rules and secrets as needed

## Project Structure

```
.
├── .github/
│   └── workflows/
│       ├── azure-deploy.yml      # Azure deployment workflow
│       └── blog-post-action.yaml # Blog post workflow
├── main.bicep                    # Main Bicep template
├── AZURE_SETUP.md               # Azure credentials setup guide
└── README.md                    # This file
```

## Troubleshooting

### Deployment Fails

Check the following:
- Azure credentials are correctly configured in GitHub secrets
- Service principal has Contributor role
- Resource providers are registered
- Resource names are unique

### View Deployment Logs

1. Go to the **Actions** tab in GitHub
2. Click on the failed workflow run
3. Expand the failed step to view detailed logs

### Azure Resource Issues

View deployment details:

```bash
az deployment group show \
  --resource-group rg-bicep-dev \
  --name main
```

## Cleanup

### Delete Resources

```bash
# Delete resource group and all resources
az group delete --name rg-bicep-dev --yes
```

## Resources

- [Azure Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [GitHub Actions for Azure](https://github.com/Azure/actions)
- [Azure CLI Reference](https://learn.microsoft.com/cli/azure/)

## License

MIT
