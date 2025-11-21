# Azure CLI Setup and Usage Guide

## Current Status

Azure CLI is already installed on your system!
- **Version**: 2.14.2 (Outdated - updates available)
- **Python Version**: 3.6.8
- **Location**: `C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\python.exe`

## Step 1: Update Azure CLI (Recommended)

Your version is outdated. Update to the latest version:

### Method 1: Using Azure CLI (Easiest)
```bash
az upgrade
```

### Method 2: Download Latest Installer
1. Download the latest MSI installer from: https://aka.ms/installazurecliwindows
2. Run the installer
3. Follow the installation wizard
4. Restart your terminal after installation

### Verify Installation
```bash
az --version
```

## Step 2: Login to Azure

### Interactive Browser Login (Recommended)
```bash
az login
```

This will:
1. Open your default web browser
2. Redirect to Azure login page
3. Ask you to sign in with your Microsoft account
4. Return you to the CLI with confirmation

### Alternative: Login with Credentials
```bash
az login -u <username> -p <password>
```

### Login to Specific Tenant
```bash
az login --tenant <tenant-id>
```

### Login with Service Principal
```bash
az login --service-principal \
  -u <client-id> \
  -p <client-secret> \
  --tenant <tenant-id>
```

### Verify Login
```bash
# List all subscriptions
az account list --output table

# Show current subscription
az account show
```

## Step 3: Set Default Subscription

If you have multiple subscriptions:

```bash
# List all subscriptions
az account list --output table

# Set default subscription
az account set --subscription "<subscription-name-or-id>"

# Verify
az account show --query name -o tsv
```

## Step 4: Basic Azure CLI Commands

### Resource Groups

```bash
# List all resource groups
az group list --output table

# Create a resource group
az group create \
  --name rg-bicep-dev \
  --location eastus

# Show resource group details
az group show --name rg-bicep-dev

# Delete resource group
az group delete --name rg-bicep-dev --yes --no-wait
```

### Locations

```bash
# List all available locations
az account list-locations --output table

# List locations in specific region
az account list-locations --query "[?metadata.regionType=='Physical']" --output table
```

### Resources

```bash
# List all resources in subscription
az resource list --output table

# List resources in a resource group
az resource list \
  --resource-group rg-bicep-dev \
  --output table

# Show specific resource
az resource show \
  --resource-group rg-bicep-dev \
  --name myapp \
  --resource-type "Microsoft.Web/sites"
```

### Bicep Operations

```bash
# Install/Upgrade Bicep CLI
az bicep install
az bicep upgrade

# Check Bicep version
az bicep version

# Build Bicep file (convert to ARM template)
az bicep build --file main.bicep

# Validate Bicep file
az bicep build --file main.bicep --stdout

# Deploy Bicep template
az deployment group create \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev location=eastus

# Show deployment details
az deployment group show \
  --resource-group rg-bicep-dev \
  --name main

# List all deployments
az deployment group list \
  --resource-group rg-bicep-dev \
  --output table
```

## Step 5: Common Azure CLI Patterns

### Output Formats

```bash
# Table format (human readable)
az group list --output table

# JSON format (default)
az group list --output json

# TSV format (tab-separated)
az group list --output tsv

# YAML format
az group list --output yaml

# JSON with JMESPath query
az group list --query "[].{Name:name, Location:location}" --output table
```

### Query with JMESPath

```bash
# Get subscription ID
az account show --query id -o tsv

# Get all resource group names
az group list --query "[].name" -o tsv

# Filter resources by location
az group list --query "[?location=='eastus'].name" -o table

# Get multiple fields
az account list --query "[].{Name:name, ID:id, State:state}" -o table
```

### Working with Variables

```bash
# Store output in variable (Windows CMD)
for /f "tokens=*" %i in ('az account show --query id -o tsv') do set SUBSCRIPTION_ID=%i

# Store output in variable (PowerShell)
$SUBSCRIPTION_ID = az account show --query id -o tsv

# Store output in variable (Bash/Git Bash)
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
echo $SUBSCRIPTION_ID
```

## Step 6: Setup for This Project

### Complete Setup Script

For **Git Bash** or **WSL**:

```bash
# Login to Azure
az login

# Get subscription ID
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
echo "Subscription ID: $SUBSCRIPTION_ID"

# Create service principal with OIDC
APP_ID=$(az ad sp create-for-rbac \
  --name "github-actions-bicep-deploy" \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID \
  --query clientId -o tsv)

echo "Client ID: $APP_ID"

# Get tenant ID
TENANT_ID=$(az account show --query tenantId -o tsv)
echo "Tenant ID: $TENANT_ID"

# Create resource group
az group create \
  --name rg-bicep-dev \
  --location eastus

# Deploy Bicep template
az deployment group create \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev location=eastus

echo "Deployment complete!"
```

For **PowerShell**:

```powershell
# Login to Azure
az login

# Get subscription ID
$SUBSCRIPTION_ID = az account show --query id -o tsv
Write-Host "Subscription ID: $SUBSCRIPTION_ID"

# Create service principal
$APP_ID = az ad sp create-for-rbac `
  --name "github-actions-bicep-deploy" `
  --role contributor `
  --scopes /subscriptions/$SUBSCRIPTION_ID `
  --query clientId -o tsv

Write-Host "Client ID: $APP_ID"

# Get tenant ID
$TENANT_ID = az account show --query tenantId -o tsv
Write-Host "Tenant ID: $TENANT_ID"

# Create resource group
az group create `
  --name rg-bicep-dev `
  --location eastus

# Deploy Bicep template
az deployment group create `
  --resource-group rg-bicep-dev `
  --template-file main.bicep `
  --parameters environmentName=dev location=eastus

Write-Host "Deployment complete!"
```

## Troubleshooting

### Issue: "az: command not found"

**Solution**: Add Azure CLI to PATH or restart terminal
```bash
# Windows: Add to PATH
setx PATH "%PATH%;C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin"
```

### Issue: Login fails

**Solution**: Clear credentials and retry
```bash
az account clear
az login
```

### Issue: Permission denied

**Solution**: Ensure you have proper role assignments
```bash
# Check current user permissions
az role assignment list --assignee <your-email> --output table

# Assign role (requires admin)
az role assignment create \
  --assignee <your-email> \
  --role Contributor \
  --scope /subscriptions/<subscription-id>
```

### Issue: Bicep not found

**Solution**: Install Bicep CLI
```bash
az bicep install
```

### Issue: Old Python version

Your current Python is 3.6.8 (outdated). Update Azure CLI to get the latest Python version.

## Useful Resources

- **Azure CLI Documentation**: https://learn.microsoft.com/cli/azure/
- **Azure CLI Reference**: https://learn.microsoft.com/cli/azure/reference-index
- **JMESPath Tutorial**: https://jmespath.org/tutorial.html
- **Bicep Documentation**: https://learn.microsoft.com/azure/azure-resource-manager/bicep/

## Quick Reference Card

| Task | Command |
|------|---------|
| Login | `az login` |
| Show subscription | `az account show` |
| List subscriptions | `az account list --output table` |
| Set subscription | `az account set --subscription <name>` |
| Create resource group | `az group create --name <name> --location <location>` |
| Deploy Bicep | `az deployment group create --resource-group <rg> --template-file <file>` |
| List resources | `az resource list --output table` |
| Get help | `az --help` or `az <command> --help` |
| Logout | `az logout` |

## Next Steps

1. **Update Azure CLI**: Run `az upgrade`
2. **Login to Azure**: Run `az login`
3. **Set subscription**: Run `az account set --subscription "<your-subscription>"`
4. **Install Bicep**: Run `az bicep install`
5. **Deploy your infrastructure**: Follow the scripts in Step 6
