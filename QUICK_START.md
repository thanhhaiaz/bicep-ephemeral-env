# Quick Start Guide - Azure CLI

## Your Current Status

✅ **Azure CLI Installed**: Version 2.14.2 (needs update)
✅ **Logged In**: admin@hainguyenstepmedia.onmicrosoft.com
⚠️ **Token Expired**: Need to re-authenticate
📦 **Subscription**: NFR-StepMedia (9f1e96f9-cdf7-4df1-94a5-55c9640ecc0a)

---

## Step-by-Step: Run Azure CLI Commands

### Step 1: Update Azure CLI (Recommended First)

Your version is from 2021 and needs updating:

```bash
az upgrade
```

**Alternative**: Download latest installer from https://aka.ms/installazurecliwindows

---

### Step 2: Re-Login to Azure

Your token has expired. Login again:

```bash
az login
```

**What happens:**
1. Browser opens automatically
2. Sign in with: `admin@hainguyenstepmedia.onmicrosoft.com`
3. After successful login, browser shows success message
4. Return to terminal - you're ready!

**Expected output:**
```json
[
  {
    "cloudName": "AzureCloud",
    "id": "9f1e96f9-cdf7-4df1-94a5-55c9640ecc0a",
    "isDefault": true,
    "name": "NFR-StepMedia",
    "state": "Enabled",
    "tenantId": "2e239550-d962-4aee-8547-7bb43a4ccedc"
  }
]
```

---

### Step 3: Verify Login

```bash
# Show current account
az account show

# List all subscriptions
az account list --output table
```

---

### Step 4: Install Bicep CLI

Required for deploying our Bicep templates:

```bash
az bicep install
```

Check version:
```bash
az bicep version
```

---

### Step 5: Create Resource Group

```bash
az group create \
  --name rg-bicep-dev \
  --location eastus
```

**Expected output:**
```json
{
  "id": "/subscriptions/9f1e96f9-cdf7-4df1-94a5-55c9640ecc0a/resourceGroups/rg-bicep-dev",
  "location": "eastus",
  "name": "rg-bicep-dev",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

---

### Step 6: Deploy Bicep Template

```bash
az deployment group create \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev location=eastus
```

This will:
1. Validate the Bicep template
2. Create App Service Plan
3. Create Web App
4. Create Storage Account
5. Return deployment outputs

**Watch the deployment:**
```bash
# The deployment takes 2-5 minutes
# You'll see progress messages
```

---

### Step 7: Verify Deployment

```bash
# List all resources in the resource group
az resource list \
  --resource-group rg-bicep-dev \
  --output table

# Show deployment details
az deployment group show \
  --resource-group rg-bicep-dev \
  --name main
```

---

### Step 8: Get Deployment Outputs

```bash
# Get the Web App URL
az deployment group show \
  --resource-group rg-bicep-dev \
  --name main \
  --query properties.outputs.webAppUrl.value \
  -o tsv
```

---

## Common Commands You'll Use

### Resource Groups

```bash
# List all resource groups
az group list --output table

# Show specific resource group
az group show --name rg-bicep-dev

# Delete resource group (careful!)
az group delete --name rg-bicep-dev --yes
```

### Deployments

```bash
# List all deployments
az deployment group list \
  --resource-group rg-bicep-dev \
  --output table

# Show deployment
az deployment group show \
  --resource-group rg-bicep-dev \
  --name main

# What-if deployment (preview changes)
az deployment group what-if \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev
```

### Resources

```bash
# List all resources
az resource list --output table

# List resources in resource group
az resource list \
  --resource-group rg-bicep-dev \
  --output table

# Show specific resource
az webapp show \
  --resource-group rg-bicep-dev \
  --name <your-web-app-name>
```

---

## Setup GitHub Actions (After Manual Deployment Works)

Once you've successfully deployed manually, set up GitHub Actions:

### 1. Create Service Principal

```bash
# Get your subscription ID
az account show --query id -o tsv

# Create service principal
az ad sp create-for-rbac \
  --name "github-actions-bicep-deploy" \
  --role contributor \
  --scopes /subscriptions/9f1e96f9-cdf7-4df1-94a5-55c9640ecc0a \
  --sdk-auth
```

**Save the output!** You'll need it for GitHub secrets.

### 2. Create Federated Credentials (OIDC - More Secure)

```bash
# First, create the service principal and get client ID
APP_ID=$(az ad sp create-for-rbac \
  --name "github-actions-bicep-deploy" \
  --role contributor \
  --scopes /subscriptions/9f1e96f9-cdf7-4df1-94a5-55c9640ecc0a \
  --query clientId -o tsv)

echo "Client ID: $APP_ID"

# Add federated credential for main branch
# Replace YOUR_GITHUB_USERNAME with your actual GitHub username
az ad app federated-credential create \
  --id $APP_ID \
  --parameters '{
    "name": "github-main",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:YOUR_GITHUB_USERNAME/bicep-ephemeral-env:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

### 3. Get Required Values for GitHub Secrets

```bash
# Subscription ID
az account show --query id -o tsv

# Tenant ID
az account show --query tenantId -o tsv

# Client ID (from step 2)
echo $APP_ID
```

Add these to GitHub:
- Repository Settings → Secrets and variables → Actions → New repository secret
- Add: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`

---

## Troubleshooting

### Token Expired (like now)
```bash
az logout
az login
```

### Need to clear all credentials
```bash
az account clear
az login
```

### Wrong subscription selected
```bash
az account set --subscription "NFR-StepMedia"
```

### Bicep errors
```bash
# Reinstall Bicep
az bicep uninstall
az bicep install

# Validate Bicep file
az bicep build --file main.bicep
```

---

## Ready to Start?

Run these commands in order:

```bash
# 1. Update CLI
az upgrade

# 2. Login
az login

# 3. Install Bicep
az bicep install

# 4. Validate Bicep template
az bicep build --file main.bicep

# 5. Create resource group
az group create --name rg-bicep-dev --location eastus

# 6. Deploy!
az deployment group create \
  --resource-group rg-bicep-dev \
  --template-file main.bicep \
  --parameters environmentName=dev location=eastus
```

---

## Cleanup When Done Testing

```bash
# Delete everything
az group delete --name rg-bicep-dev --yes --no-wait
```

The `--no-wait` flag allows the command to return immediately while deletion happens in the background.

---

## Next Steps

1. ✅ Run the commands above to deploy manually
2. ✅ Verify resources are created in Azure Portal
3. ✅ Set up service principal for GitHub Actions
4. ✅ Add GitHub secrets
5. ✅ Push code to GitHub - automatic deployment!

Need help? Check:
- **AZURE_CLI_GUIDE.md** - Comprehensive guide
- **AZURE_SETUP.md** - GitHub Actions setup
- **README.md** - Project overview
