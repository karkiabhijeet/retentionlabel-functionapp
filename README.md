# Asset ID Update Azure Function

An Azure Function that automatically detects files with "FAR - Label" retention labels in SharePoint and OneDrive, and updates them with Asset IDs if they don't already have one.

## 🚀 Quick Deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkarkiabhijeet%2Fretentionlabel-functionapp%2Fmain%2Fazuredeploy.json)

## Overview

This solution provides automated Asset ID management for compliance purposes by:
- Running daily to scan SharePoint sites and OneDrive locations
- Identifying files with "FAR - Label" retention labels
- Adding unique Asset IDs to files that don't have them
- Supporting multiple file formats including Office documents and text files

## Features

- **Automated Daily Execution**: Timer-triggered function runs at 6 AM UTC daily
- **Microsoft Graph Integration**: Uses Microsoft Graph API to access SharePoint and OneDrive
- **Retention Label Detection**: Identifies files with specific retention labels
- **Smart Asset ID Generation**: Creates unique Asset IDs with timestamp and GUID components
- **Multi-Format Support**: Handles .txt, .docx, .doc, .xlsx, .xls, .pptx, .ppt, .pdf files
- **Comprehensive Logging**: Detailed logging for monitoring and debugging

## Prerequisites

- Azure subscription
- Azure AD application with appropriate permissions
- Azure Functions Core Tools (for local development)
- .NET 8 SDK

## Required Azure AD Permissions

Your Azure AD application needs the following Microsoft Graph API permissions:

- `Sites.ReadWrite.All` - Access SharePoint sites and modify files
- `Files.ReadWrite.All` - Access and modify files in OneDrive
- `User.Read.All` - Read user profiles to access OneDrive
- `InformationProtectionPolicy.Read.All` - Read retention labels

## 📖 Documentation

- [Customer Setup Guide](CUSTOMER_SETUP_GUIDE.md) - Complete deployment instructions
- [Deployment Guide](DEPLOYMENT.md) - Step-by-step Azure deployment
- [Project Summary](PROJECT_SUMMARY.md) - Technical overview and architecture

## Setup Instructions

### 1. Clone and Setup Project

```bash
# Clone the repository
git clone https://github.com/karkiabhijeet/retentionlabel-functionapp.git
cd retentionlabel-functionapp

# Install dependencies
dotnet restore
```

### 2. Configure Azure AD Application

1. Register a new application in Azure AD
2. Grant the required API permissions listed above
3. Create a client secret
4. Note down the Tenant ID, Client ID, and Client Secret

### 3. Configure Local Settings

Copy `local.settings.json.template` to `local.settings.json` and update with your Azure AD application details:

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "TENANT_ID": "your-tenant-id-here",
        "CLIENT_ID": "your-client-id-here",
        "CLIENT_SECRET": "your-client-secret-here"
    }
}
```

### 4. Local Development

```bash
# Start Azurite (Azure Storage Emulator)
# Run the function locally
func start
```

### 5. Deploy to Azure

See the [Customer Setup Guide](CUSTOMER_SETUP_GUIDE.md) for complete deployment instructions.

## How It Works

1. **Timer Trigger**: The function executes daily at 6 AM UTC
2. **Site Discovery**: Retrieves all SharePoint sites in the tenant
3. **OneDrive Access**: Accesses all users' OneDrive locations
4. **File Scanning**: Recursively scans all files in drives and document libraries
5. **Label Detection**: Checks each file for the "FAR - Label" retention label
6. **Asset ID Check**: Examines file content for existing Asset ID
7. **Asset ID Addition**: Adds a unique Asset ID if none exists
8. **File Update**: Uploads the modified file back to its location

## Asset ID Format

Asset IDs are generated in the format: `AST-YYYYMMDD-XXXXXXXX`

Where:
- `AST` - Static prefix
- `YYYYMMDD` - Current date
- `XXXXXXXX` - 8-character uppercase GUID segment

Example: `AST-20250125-A1B2C3D4`

## Monitoring and Logs

The function provides comprehensive logging at various levels:
- **Information**: Normal operation progress
- **Warning**: Non-critical issues (e.g., inaccessible OneDrive)
- **Error**: Critical errors requiring attention

Monitor logs through:
- Azure Portal Function App logs
- Application Insights (if configured)
- Local console output during development

## Security Considerations

- Store sensitive configuration in Azure Key Vault for production
- Use managed identity instead of client secrets when possible
- Regularly rotate client secrets
- Monitor API usage and permissions
- Implement proper error handling to avoid credential exposure

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
