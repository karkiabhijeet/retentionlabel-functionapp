# Copilot Instructions for Asset ID Update Azure Function

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is an Azure Function project designed to automatically update Asset IDs in files that have the "FAR - Label" retention label in SharePoint and OneDrive.

## Key Features
- **Timer Trigger**: Runs daily at 6 AM UTC
- **Microsoft Graph Integration**: Accesses SharePoint sites and OneDrive files
- **Retention Label Detection**: Identifies files with "FAR - Label" retention label
- **Asset ID Management**: Adds Asset IDs to files that don't have them
- **Multi-Platform Support**: Works with SharePoint and OneDrive

## Architecture
- Uses Azure Functions v4 with .NET 6
- Leverages Microsoft Graph SDK for Office 365 integration
- Implements Client Credentials authentication flow
- Supports multiple file types: .txt, .docx, .doc, .xlsx, .xls, .pptx, .ppt, .pdf

## Development Guidelines
- Follow async/await patterns for all Graph API calls
- Implement proper error handling and logging
- Use environment variables for sensitive configuration
- Test with limited scope before production deployment

## Required Permissions
The Azure AD application needs the following Microsoft Graph permissions:
- Sites.ReadWrite.All
- Files.ReadWrite.All
- User.Read.All
- InformationProtectionPolicy.Read.All

## Configuration
Update local.settings.json with:
- TENANT_ID: Your Azure AD tenant ID
- CLIENT_ID: Your Azure AD application client ID
- CLIENT_SECRET: Your Azure AD application client secret
