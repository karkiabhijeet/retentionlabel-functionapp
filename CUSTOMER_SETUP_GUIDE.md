# Customer Setup Guide - Asset ID Update Azure Function

## 🎯 Overview
This guide helps you deploy and configure the Asset ID Update Azure Function in your Microsoft 365 environment.

## 📋 Prerequisites Checklist

- [ ] Azure subscription with sufficient permissions
- [ ] Microsoft 365 tenant with admin access
- [ ] Azure CLI installed
- [ ] Azure Functions Core Tools installed
- [ ] .NET 6 SDK installed

## Step 1: Azure AD Application Setup

### 1.1 Create Azure AD Application
1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to **Azure Active Directory** > **App registrations**
3. Click **"New registration"**
4. Fill in:
   - **Name**: `AssetID-Update-Function`
   - **Supported account types**: `Accounts in this organizational directory only`
   - **Redirect URI**: Leave blank
5. Click **"Register"**
6. **IMPORTANT**: Copy the **Application (client) ID** and **Directory (tenant) ID**

### 1.2 Configure API Permissions
1. In your app registration, go to **API permissions**
2. Click **"Add a permission"** > **Microsoft Graph** > **Application permissions**
3. Add these permissions:
   - `Sites.ReadWrite.All`
   - `Files.ReadWrite.All`
   - `User.Read.All`
   - `InformationProtectionPolicy.Read.All`
4. Click **"Grant admin consent for [Your Organization]"**
5. Confirm the consent

### 1.3 Create Client Secret
1. Go to **Certificates & secrets**
2. Click **"New client secret"**
3. Add description: `AssetID Function Secret`
4. Set expiration: `24 months` (recommended)
5. Click **"Add"**
6. **CRITICAL**: Copy the secret **Value** immediately (you won't see it again!)

## Step 2: Azure Resources Setup

### 2.1 Create Resource Group
```bash
az group create --name rg-assetid-prod --location "East US"
```

### 2.2 Create Storage Account
```bash
az storage account create \
  --name stassetidprod[RANDOM] \
  --resource-group rg-assetid-prod \
  --location "East US" \
  --sku Standard_LRS
```
*Replace [RANDOM] with 3-4 random characters to ensure uniqueness*

### 2.3 Create Function App
```bash
az functionapp create \
  --resource-group rg-assetid-prod \
  --consumption-plan-location "East US" \
  --runtime dotnet \
  --runtime-version 8 \
  --functions-version 4 \
  --name func-assetid-prod-[RANDOM] \
  --storage-account stassetidprod[RANDOM]
```

## Step 3: Configure Application Settings (PRODUCTION)

**🚨 NEVER put secrets in code files! Use Azure application settings:**

```bash
# Set Tenant ID
az functionapp config appsettings set \
  --name func-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --settings "TENANT_ID=your-actual-tenant-id"

# Set Client ID  
az functionapp config appsettings set \
  --name func-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --settings "CLIENT_ID=your-actual-client-id"

# Set Client Secret
az functionapp config appsettings set \
  --name func-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --settings "CLIENT_SECRET=your-actual-client-secret"
```

## Step 4: Deploy the Function

### 4.1 Build the Project
```bash
# Navigate to the project directory
cd AssetID-Update

# Restore packages and build
dotnet restore
dotnet build --configuration Release
```

### 4.2 Deploy to Azure
```bash
# Deploy the function
func azure functionapp publish func-assetid-prod-[RANDOM]
```

## Step 5: Verify Deployment

### 5.1 Check Function Status
1. Go to Azure Portal > Function Apps
2. Select your function app
3. Go to **Functions** > **AssetIdUpdateFunction**
4. Check that it shows as "Enabled"

### 5.2 Test Function Execution
```bash
# Manually trigger the function for testing
az functionapp function invoke \
  --resource-group rg-assetid-prod \
  --name func-assetid-prod-[RANDOM] \
  --function-name AssetIdUpdateFunction
```

### 5.3 Monitor Logs
1. In Azure Portal, go to your Function App
2. Navigate to **Monitor** > **Logs**
3. Check for successful execution and any errors

## Step 6: Security Best Practices (Recommended)

### 6.1 Use Azure Key Vault for Secrets
```bash
# Create Key Vault
az keyvault create \
  --name kv-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --location "East US"

# Store client secret in Key Vault
az keyvault secret set \
  --vault-name kv-assetid-prod-[RANDOM] \
  --name "ClientSecret" \
  --value "your-actual-client-secret"

# Update Function App to use Key Vault reference
az functionapp config appsettings set \
  --name func-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --settings "CLIENT_SECRET=@Microsoft.KeyVault(VaultName=kv-assetid-prod-[RANDOM];SecretName=ClientSecret)"
```

## Step 7: Schedule Configuration

The function is currently set to run daily at 6 AM UTC. To change this:

1. Modify the timer trigger in `AssetIdUpdateFunction.cs`:
```csharp
[TimerTrigger("0 0 6 * * *")] // 6 AM UTC daily
```

2. Redeploy the function after changes

### Common CRON Expressions:
- `0 0 6 * * *` - Daily at 6 AM UTC
- `0 0 2 * * *` - Daily at 2 AM UTC  
- `0 0 6 * * 1-5` - Weekdays only at 6 AM UTC
- `0 0 */6 * * *` - Every 6 hours

## 📊 Monitoring and Maintenance

### Enable Application Insights (Recommended)
```bash
az functionapp config appsettings set \
  --name func-assetid-prod-[RANDOM] \
  --resource-group rg-assetid-prod \
  --settings "APPINSIGHTS_INSTRUMENTATIONKEY=your-appinsights-key"
```

### Set Up Alerts
1. Create alerts for function failures
2. Monitor execution duration
3. Set up notifications for errors

## 🔧 Configuration Reference

### Required Azure AD Information:
- **Tenant ID**: Found in Azure AD > Overview
- **Client ID**: Found in App Registration > Overview  
- **Client Secret**: Created in App Registration > Certificates & secrets

### Azure Resource Names (customize these):
- **Resource Group**: `rg-assetid-prod`
- **Storage Account**: `stassetidprod[RANDOM]`
- **Function App**: `func-assetid-prod-[RANDOM]`
- **Key Vault**: `kv-assetid-prod-[RANDOM]`

## 🚨 Important Security Notes

1. **Never commit secrets to source control**
2. **Use Key Vault for production secrets**
3. **Rotate client secrets every 12-24 months**
4. **Monitor function execution and API usage**
5. **Use least privilege principle for permissions**

## 📞 Support

If you encounter issues:
1. Check Azure Function logs in the portal
2. Verify Azure AD permissions are granted
3. Ensure client secret hasn't expired
4. Check Microsoft Graph API limits and throttling

## 🔄 Updates and Maintenance

### To Update the Function:
1. Make code changes
2. Build: `dotnet build --configuration Release`
3. Deploy: `func azure functionapp publish func-assetid-prod-[RANDOM]`

### Regular Maintenance:
- Monitor function execution logs
- Check client secret expiration dates
- Review and update permissions as needed
- Monitor Microsoft Graph API usage
