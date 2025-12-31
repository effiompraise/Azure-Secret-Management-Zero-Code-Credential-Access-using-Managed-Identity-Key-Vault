# Azure Secret Management: Zero-Code Credential Access using Managed Identity & Key Vault

![Azure Key Vault Banner](/images/keyvault_banner.png)

## Introduction

In this project, I secured a cloud-native web application by implementing a **Zero-Code** authentication architecture. The primary goal was to eliminate the security risk of hardcoded credentials (connection strings) in the application source code.

By integrating **Azure Key Vault** with **Azure App Service** and using **System-Assigned Managed Identity**, I created a secure flow where the application retrieves its database credentials only at runtime. This "passwordless" handshake ensures that no sensitive keys effectively exist within the codebase or configuration files, significantly reducing the attack surface.

## Objectives

- **Eliminate Hardcoded Secrets:** Remove Cosmos DB connection strings from `appsettings.json` and code.
- **Implement Managed Identity:** Enable System-Assigned Identity for the App Service to establish trust without service principals.
- **Centralize Secret Management:** Store and manage all application secrets in a secure Azure Key Vault.
- **Enforce Least Privilege:** Configure Key Vault Access Policies to grant the application only `Get` and `List` permissions.
- **Validate End-to-End Security:** Deploy a .NET web app that successfully connects to Cosmos DB using the retrieved secrets.

## Tech Stack

![Azure Key Vault](https://img.shields.io/badge/Azure_Key_Vault-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Managed Identity](https://img.shields.io/badge/Managed_Identity-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Cosmos DB](https://img.shields.io/badge/Azure_Cosmos_DB-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![App Service](https://img.shields.io/badge/Azure_App_Service-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)

- **Azure Key Vault** - Secure secret storage.
- **System-Assigned Managed Identity** - Identity management for Azure resources.
- **Azure Cosmos DB** - Backend NoSQL database.
- **Azure App Service** - PaaS web hosting.
- **RBAC / Access Policies** - Granular permission control.

## Architecture Overview

### Secret Retrieval Flow

![Key Vault Architecture](/images/keyvault_architecture.png)

1.  **Identity Creation:** When the App Service starts, Azure creates a System-Assigned Identity for it in Entra ID (Active Directory).
2.  **Authentication:** The application attempts to access Key Vault. Azure validates the application's identity.
3.  **Authorization:** Key Vault checks its Access Policies. Since we explicitly granted this specific identity `Get` permissions, the request is allowed.
4.  **Retrieval:** The connection string is returned to the application in memory.
5.  **Connection:** The app uses the connection string to authenticate to Cosmos DB.

## Implementation Steps

### Phase 1: Infrastructure Setup

I provisioned the core resources required for the lab:
* **Resource Group:** `rg_centralus_...`
* **Cosmos DB Account:** `mydevsecopscosmosdb` (NoSQL API)
* **Key Vault:** `myvaultkey1` (Standard Tier)

*Note: Initial setup was performed via the Azure Portal for visibility, but this pattern is fully compatible with Terraform/Bicep.*

![Cosmos DB Setup](/images/Cosmos%20DB.png)
*Figure 1: Configuring the Cosmos DB account with Provisioned Throughput capacity mode.*

### Phase 2: Secret Centralization

Instead of pasting the connection string into the app, I stored it securely:
1.  Navigated to the Cosmos DB "Keys" blade and copied the `Primary Connection String`.
2.  Created a new **Secret** in Key Vault named `mysecret`.
3.  Pasted the value, ensuring it is now encrypted at rest.

![Secret Creation](/images/Secret%20creation.png)
*Figure 2: Successfully created the 'mysecret' credential in Azure Key Vault.*

### Phase 3: Application Development & Configuration

I utilized a .NET Core MVC web application designed to fetch secrets at runtime.

1.  **Project Setup:** Created a new MVC project and installed the necessary NuGet packages (`Microsoft.Azure.KeyVault`, `Microsoft.Azure.Services.AppAuthentication`).
2.  **Code Integration:** Updated `HomeController.cs` to use `KeyVaultClient`.
3.  **Secret Identifier:** Replaced the placeholder in the code with the actual **Secret Identifier URL** from Key Vault.

![VS Code Setup](/images/VS%20code%20setup.png)
*Figure 3: Configuring the application code to point to the Key Vault Secret Identifier.*

### Phase 4: Identity & Access Configuration

This is the critical security step. I enabled the identity for the web app and authorized it:

1.  **App Service -> Identity:** Switched Status to `On`. This registered the app in Azure AD.
2.  **Key Vault -> Access Policies:** Added a new policy.
    * **Principal:** Selected the App Service.
    * **Permissions:** `Secret Management` -> `Get`, `List`.
    * **Result:** The app can now "read" the vault, but cannot modify it.

*This step ensures that even if someone has the code, they cannot access the database without being the authenticated App Service.*

### Phase 5: Verification

To verify the success of the implementation:
1.  Deployed the application to Azure App Service using the Azure CLI/VS Code extension.
2.  Browsed to the Web App URL.
3.  The application successfully authenticated to Key Vault, retrieved the connection string, and connected to Cosmos DB to create/read documents.

![Deployment Success](/images/Deployment.png)
*Figure 4: Successful deployment of the Key Vault resource.*

## Strategic Analysis

### Why Managed Identity?
| Feature | Service Principal | Managed Identity (This Project) |
| :--- | :--- | :--- |
| **Credential Management** | You manage client ID/Secret | **Azure manages it automatically** |
| **Rotation** | Manual/Scripted rotation required | **Automatic rotation by Azure** |
| **Security Risk** | Secret can be leaked/stolen | **Identity is tied to the resource; cannot be stolen** |

### Security Impact
By adopting this architecture, we have effectively mitigated **Credential Theft**. Even if an attacker gains access to the application source code repository (Git), they will find no credentials to exploit. They would need to compromise the running Azure resource itself to gain access.

## Key Results

* **Zero-Code Access:** Implemented a credential-less handshake between the Web App and Key Vault.
* **Risk Reduction:** Removed high-value database keys from the application layer.
* **Auditability:** Key Vault logs now provide a clear audit trail of exactly when the application accesses the database credentials.

---

## Repository Contents

* `/images/` - Architecture diagrams and proof-of-concept screenshots.

## About This Project

**Role:**
Cloud Security Engineer

**Skills Demonstrated:**
Application Security (AppSec), Identity & Access Management (IAM), Azure Key Vault, Managed Identities.