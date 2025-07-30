# Deployment Guide for Asset ID Update Azure Function

## Overview
This guide provides step-by-step instructions for deploying the Asset ID Update Azure Function to Azure.

## Prerequisites

### 1. Azure Resources
- Azure subscription
- Resource group
- Storage account for the function app
- Function App (Consumption or Premium plan)

### 2. Azure AD Application
- Registered Azure AD application
- Required Microsoft Graph permissions
- Client secret

### 3. Development Tools
- Azure CLI
- Azure Functions Core Tools
- .NET 6 SDK

## Azure AD Application Setup

### 1. Create Azure AD Application

```bash
# Create the application
az ad app create --display-name "AssetID-Update-Function" --sign-in-audience "AzureADMyOrg"

# Note the Application (client) ID from the output
```

### 2. Configure API Permissions

In the Azure Portal:
1. Navigate to Azure Active Directory > App registrations
2. Select your application
3. Go to API permissions
4. Add the following Microsoft Graph permissions:
   - `Sites.ReadWrite.All` (Application)
   - `Files.ReadWrite.All` (Application)
   - `User.Read.All` (Application)
   - `InformationProtectionPolicy.Read.All` (Application)

### 3. Grant Admin Consent

1. In API permissions page, click "Grant admin consent"
2. Confirm the consent

### 4. Create Client Secret

1. Go to Certificates & secrets
2. Click "New client secret"
3. Add description and set expiration
4. Copy the secret value (you won't see it again)

## Azure Resources Setup

### 1. Create Resource Group

```bash
az group create --name rg-assetid-update --location "East US"
```

### 2. Create Storage Account

```bash
az storage account create \
  --name stassetidupdate \
  --resource-group rg-assetid-update \
  --location "East US" \
  --sku Standard_LRS
```

### 3. Create Function App

```bash
az functionapp create \
  --resource-group rg-assetid-update \
  --consumption-plan-location "East US" \
  --runtime dotnet \
  --functions-version 4 \
  --name func-assetid-update \
  --storage-account stassetidupdate
```

## Application Settings Configuration

### 1. Configure Authentication Settings

```bash
# Set Tenant ID
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "TENANT_ID=your-tenant-id-here"

# Set Client ID
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "CLIENT_ID=your-client-id-here"

# Set Client Secret
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "CLIENT_SECRET=your-client-secret-here"
```

### 2. Optional: Configure Additional Settings

```bash
# Set timezone (if needed)
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "WEBSITE_TIME_ZONE=UTC"

# Enable Application Insights (recommended)
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "APPINSIGHTS_INSTRUMENTATIONKEY=your-appinsights-key"
```

## Deploy the Function

### 1. Build the Project

```bash
# From the project directory
dotnet build --configuration Release
```

### 2. Deploy to Azure

```bash
# Deploy using Azure Functions Core Tools
func azure functionapp publish func-assetid-update
```

### 3. Verify Deployment

```bash
# Check function status
az functionapp function show \
  --resource-group rg-assetid-update \
  --name func-assetid-update \
  --function-name AssetIdUpdateFunction
```

## Post-Deployment Configuration

### 1. Verify Function Execution

1. Go to Azure Portal > Function App
2. Navigate to your function app
3. Go to Functions > AssetIdUpdateFunction
4. Check the Monitor tab for execution logs

### 2. Test the Function

```bash
# Trigger the function manually for testing
az functionapp function invoke \
  --resource-group rg-assetid-update \
  --name func-assetid-update \
  --function-name AssetIdUpdateFunction
```

### 3. Monitor Execution

1. Check Application Insights for detailed logs
2. Monitor Function App logs in Azure Portal
3. Set up alerts for failures

## Security Best Practices

### 1. Use Key Vault for Secrets

```bash
# Create Key Vault
az keyvault create \
  --name kv-assetid-update \
  --resource-group rg-assetid-update \
  --location "East US"

# Store client secret in Key Vault
az keyvault secret set \
  --vault-name kv-assetid-update \
  --name "ClientSecret" \
  --value "your-client-secret-here"

# Configure Function App to use Key Vault reference
az functionapp config appsettings set \
  --name func-assetid-update \
  --resource-group rg-assetid-update \
  --settings "CLIENT_SECRET=@Microsoft.KeyVault(VaultName=kv-assetid-update;SecretName=ClientSecret)"
```

### 2. Enable Managed Identity

```bash
# Enable system-assigned managed identity
az functionapp identity assign \
  --name func-assetid-update \
  --resource-group rg-assetid-update

# Grant Key Vault access to the managed identity
az keyvault set-policy \
  --name kv-assetid-update \
  --object-id $(az functionapp identity show --name func-assetid-update --resource-group rg-assetid-update --query principalId -o tsv) \
  --secret-permissions get
```

## Monitoring and Maintenance

### 1. Set Up Alerts

```bash
# Create action group for notifications
az monitor action-group create \
  --name ag-assetid-alerts \
  --resource-group rg-assetid-update \
  --short-name assetid

# Create alert for function failures
az monitor metrics alert create \
  --name "Asset ID Function Failures" \
  --resource-group rg-assetid-update \
  --scopes "/subscriptions/{subscription-id}/resourceGroups/rg-assetid-update/providers/Microsoft.Web/sites/func-assetid-update" \
  --condition "count 'FunctionExecutionCount' > 0" \
  --action ag-assetid-alerts
```

### 2. Regular Maintenance Tasks

- Monitor function execution logs
- Review and rotate client secrets
- Update Azure AD application permissions as needed
- Monitor Microsoft Graph API usage and throttling
- Update function code for new requirements

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   - Verify Azure AD application permissions
   - Check client secret expiration
   - Ensure admin consent is granted

2. **Permission Errors**
   - Verify Microsoft Graph permissions
   - Check if permissions require admin consent
   - Ensure the application has access to SharePoint/OneDrive

3. **Function Execution Errors**
   - Check Application Insights logs
   - Verify environment variables
   - Monitor function timeout settings

4. **Rate Limiting**
   - Implement retry logic
   - Monitor Graph API usage
   - Consider throttling function execution

### Debug Mode

For troubleshooting, you can:

1. Enable detailed logging in the function
2. Use local debugging with Azure Storage Emulator
3. Test with a smaller set of users/sites first
4. Monitor Graph API responses and errors

## Scaling Considerations

### Performance Optimization

1. **Batch Processing**: Process files in batches to avoid timeouts
2. **Parallel Processing**: Use parallel execution for multiple sites/users
3. **Caching**: Cache frequently accessed data
4. **Filtering**: Implement smart filtering to process only relevant files

### Cost Optimization

1. **Consumption Plan**: Use for variable workloads
2. **Premium Plan**: Use for consistent workloads requiring faster cold starts
3. **Scheduling**: Optimize schedule to run during off-peak hours
4. **Monitoring**: Track execution time and costs

This deployment guide provides a comprehensive approach to setting up and maintaining the Asset ID Update Azure Function in a production environment.
