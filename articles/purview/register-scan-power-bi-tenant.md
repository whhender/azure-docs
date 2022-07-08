---
title: Connect to and manage a Power BI tenant same tenant
description: This guide describes how to connect to a Power BI tenant in the same tenant as Microsoft Purview, and use Microsoft Purview's features to scan and manage your Power BI tenant source.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-map
ms.topic: how-to
ms.date: 07/07/2022
ms.custom: template-how-to, ignite-fall-2021
---

# Connect to and manage a Power BI tenant in Microsoft Purview (Same Tenant)

This article outlines how to register a Power BI tenant in a **same-tenant scenario**, and how to authenticate and interact with the tenant in Microsoft Purview. For more information about Microsoft Purview, read the [introductory article](overview.md).

## Supported capabilities

|**Metadata Extraction**|  **Full Scan**  |**Incremental Scan**|**Scoped Scan**|**Classification**|**Access Policy**|**Lineage**|**Data Sharing**|
|---|---|---|---|---|---|---|---|
| [Yes](#deployment-checklist)| [Yes](#deployment-checklist)| Yes | No | No | No| [Yes](how-to-lineage-powerbi.md)| No |

### Supported scenarios for Power BI scans

|**Scenarios**  |**Microsoft Purview public access allowed/denied** |**Power BI public access allowed /denied** | **Runtime option** | **Authentication option**  | **Deployment checklist** | 
|---------|---------|---------|---------|---------|---------|
|Public access with Azure IR     |Allowed     |Allowed        |Azure Runtime      | Microsoft Purview Managed Identity   | [Review deployment checklist](#deployment-checklist) |
|Public access with Self-hosted IR     |Allowed     |Allowed        |Self-hosted runtime        |Delegated Authentication  | [Review deployment checklist](#deployment-checklist) |
|Private access     |Allowed     |Denied         |Self-hosted runtime        |Delegated Authentication  | [Review deployment checklist](#deployment-checklist) |
|Private access     |Denied      |Allowed        |Self-hosted runtime        |Delegated Authentication  | [Review deployment checklist](#deployment-checklist) |
|Private access     |Denied      |Denied         |Self-hosted runtime        |Delegated Authentication  | [Review deployment checklist](#deployment-checklist) |

### Known limitations

-  If Microsoft Purview or Power BI tenant is protected behind a private endpoint, Self-hosted runtime is the only option to scan.
-  Delegated authentication is the only supported authentication option if self-hosted integration runtime is used during the scan.
-  You can create only one scan for a Power BI data source that is registered in your Microsoft Purview account.
-  If Power BI dataset schema isn't shown after scan, it's due to one of the current limitations with [Power BI Metadata scanner](/power-bi/admin/service-admin-metadata-scanning).
-  Empty workspaces are skipped.
-  Payload is currently limited to 1MB and 300 columns when scanning.

## Prerequisites

Before you start, make sure you have the following prerequisites:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- An active [Microsoft Purview account](create-catalog-portal.md).

## Authentication options 

- Managed Identity 
- Delegated Authentication


## Deployment checklist
Use any of the following deployment checklists during the setup or for troubleshooting purposes, based on your scenario:   

# [Public access with Azure IR](#tab/Scenario1)
### Scan same-tenant Power BI using Azure IR and Managed Identity in public network

1. Make sure Power BI and Microsoft Purview accounts are in the same tenant.
1. Make sure Power BI tenant ID is entered correctly during the registration.
1. Make sure your [PowerBI Metadata model is up to date by enabling metadata scanning.](/power-bi/admin/service-admin-metadata-scanning-setup#enable-tenant-settings-for-metadata-scanning)
1. From Azure portal, validate if Microsoft Purview account Network is set to public access.
1. From Power BI tenant Admin Portal, make sure Power BI tenant is configured to allow public network.
1. In Azure Active Directory tenant, create a security group.
1. From Azure Active Directory tenant, make sure [Microsoft Purview account MSI is member of the new security group](#authenticate-to-power-bi-tenant-managed-identity-only).
1. On the Power BI Tenant Admin portal, validate if [Allow service principals to use read-only Power BI admin APIs](#associate-the-security-group-with-power-bi-tenant) is enabled for the new security group.

# [Public access with Self-hosted IR](#tab/Scenario2)
### Scan same-tenant Power BI using self-hosted IR and Delegated Authentication in public network

1. Make sure Power BI and Microsoft Purview accounts are in the same tenant.
1. Make sure Power BI tenant ID is entered correctly during the registration.
1. Make sure your [PowerBI Metadata model is up to date by enabling metadata scanning.](/power-bi/admin/service-admin-metadata-scanning-setup#enable-tenant-settings-for-metadata-scanning)
1. From Azure portal, validate if Microsoft Purview account Network is set to public access.
1. From Power BI tenant Admin Portal, make sure Power BI tenant is configured to allow public network.
1. Check your Azure Key Vault to make sure:
   1. There are no typos in the password.
   2. Microsoft Purview Managed Identity has get/list access to secrets.
1. Review your credential to validate: 
   1. Client ID matches _Application (Client) ID_ of the app registration.
   2. Username includes the user principal name such as `johndoe@contoso.com`.
1. Validate Power BI admin user settings to make sure:
   1. User is assigned to Power BI Administrator role.
   2. At least one [Power BI license](/power-bi/admin/service-admin-licensing-organization#subscription-license-types) is assigned to the user.
   3. If user is recently created, sign in with the user at least once to make sure password is reset successfully and user can successfully initiate the session.
   4. There's no MFA or Conditional Access Policies are enforced on the user.
1. Validate App registration settings to make sure:
   1. App registration exists in your Azure Active Directory tenant.
   2. Under **API permissions**, the following **delegated permissions** and **grant admin consent for the tenant** is set up with read for the following APIs:
      1. Power BI Service Tenant.Read.All
      2. Microsoft Graph openid
      3. Microsoft Graph User.Read
    3. Under **Authentication**, **Allow public client flows** is enabled.
2. Validate Self-hosted runtime settings:
   1. Latest version of [Self-hosted runtime](https://www.microsoft.com/download/details.aspx?id=39717) is installed on the VM.
   2. Network connectivity from Self-hosted runtime to Power BI tenant is enabled.
   3. Network connectivity from Self-hosted runtime to Microsoft services is enabled.
   4. [JDK 8 or later](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) is installed.

# [Private access](#tab/Scenario3)
### Scan same-tenant Power BI using self-hosted IR and Delegated Authentication in a private network

1. Make sure Power BI and Microsoft Purview accounts are in the same tenant.
1. Make sure Power BI tenant ID is entered correctly during the registration.
1. Make sure your [PowerBI Metadata model is up to date by enabling metadata scanning.](/power-bi/admin/service-admin-metadata-scanning-setup#enable-tenant-settings-for-metadata-scanning)
1. Check your Azure Key Vault to make sure:
   1. There are no typos in the password.
   2. Microsoft Purview Managed Identity has get/list access to secrets.
1. Review your credential to validate: 
   1. Client ID matches _Application (Client) ID_ of the app registration.
   2. Username includes the user principal name such as `johndoe@contoso.com`.
1. Validate Power BI admin user to make sure:
   1. User is assigned to Power BI Administrator role.
   2. At least one [Power BI license](/power-bi/admin/service-admin-licensing-organization#subscription-license-types) is assigned to the user.
   3. If user is recently created, sign in with the user at least once to make sure password is reset successfully and user can successfully initiate the session.
   4. There's no MFA or Conditional Access Policies are enforced on the user.
1. Validate Self-hosted runtime settings:
   1. Latest version of [Self-hosted runtime](https://www.microsoft.com/download/details.aspx?id=39717) is installed on the VM.
   2. [JDK 8 or later](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) is installed.
1. Validate App registration settings to make sure:
   1. App registration exists in your Azure Active Directory tenant.
   2. Under **API permissions**, the following **delegated permissions** and **grant admin consent for the tenant** is set up with read for the following APIs:
      1. Power BI Service Tenant.Read.All
      2. Microsoft Graph openid
      3. Microsoft Graph User.Read
   3. Under **Authentication**, **Allow public client flows** is enabled.
2. Review network configuration and validate if:
   1. A [private endpoint for Power BI tenant](/power-bi/enterprise/service-security-private-links) is deployed. (Optional)
   2. All required [private endpoints for Microsoft Purview](/azure/purview/catalog-private-link-end-to-end) are deployed.
   3. Network connectivity from Self-hosted runtime to Power BI tenant is enabled.
   3. Network connectivity from Self-hosted runtime to Microsoft services is enabled through private network.

---

## Register Power BI tenant

This section describes how to register a Power BI tenant in Microsoft Purview for same-tenant scenario.

1. Select the **Data Map** on the left navigation.

1. Then select **Register**.

    Select **Power BI** as your data source.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/select-power-bi-data-source.png" alt-text="Image showing the list of data sources available to choose.":::

1. Give your Power BI instance a friendly name.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-friendly-name.png" alt-text="Image showing Power BI data source-friendly name.":::

    The name must be between 3-63 characters long and must contain only letters, numbers, underscores, and hyphens.  Spaces aren't allowed.

    By default, the system will find the Power BI tenant that exists in the same Azure Active Directory tenant.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-datasource-registered.png" alt-text="Image showing the registered Power BI data source.":::

## Scan same-tenant Power BI

> [!TIP]
> To troubleshoot any issues with scanning:
> 1. Confirm you have completed the [**deployment checklist for your scenario**](#deployment-checklist).
> 1. Review our [**scan troubleshooting documentation**](register-scan-power-bi-tenant-troubleshoot.md).

### Scan same-tenant Power BI using Azure IR and Managed Identity
This is a suitable scenario, if both Microsoft Purview and Power BI tenant are configured to allow public access in the network settings. 

#### Authenticate to Power BI tenant-managed identity only

> [!Note]
> Follow steps in this section, only if you are planning to use **Managed Identity** as authentication option.

In Azure Active Directory Tenant, where Power BI tenant is located:

1. In the [Azure portal](https://portal.azure.com), search for **Azure Active Directory**.
   
2. Create a new security group in your Azure Active Directory, by following [Create a basic group and add members using Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

    > [!Tip]
    > You can skip this step if you already have a security group you want to use.

3. Select **Security** as the **Group Type**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/security-group.png" alt-text="Screenshot of security group type.":::

4. Add your Microsoft Purview managed identity to this security group. Select **Members**, then select **+ Add members**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-group-member.png" alt-text="Screenshot of how to add the catalog's managed instance to group.":::

5. Search for your Microsoft Purview managed identity and select it.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-catalog-to-group-by-search.png" alt-text="Screenshot showing how to add catalog by searching for its name.":::

    You should see a success notification showing you that it was added.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/success-add-catalog-msi.png" alt-text="Screenshot showing successful addition of  catalog managed identity.":::

#### Associate the security group with Power BI tenant

1. Log into the [Power BI admin portal](https://app.powerbi.com/admin-portal/tenantSettings).
   
2. Select the **Tenant settings** page.

    > [!Important]
    > You need to be a Power BI Admin to see the tenant settings page.

3. Select **Admin API settings** > **Allow service principals to use read-only Power BI admin APIs (Preview)**.
   
4. Select **Specific security groups**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/allow-service-principals-power-bi-admin.png" alt-text="Image showing how to allow service principals to get read-only Power BI admin API permissions.":::

5. Select **Admin API settings** > **Enhance admin APIs responses with detailed metadata** > Enable the toggle to allow Microsoft Purview Data Map automatically discover the detailed metadata of Power BI datasets as part of its scans.

    > [!IMPORTANT]
    > After you update the Admin API settings on your power bi tenant, wait around 15 minutes before registering a scan and test connection.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-sub-artifacts.png" alt-text="Image showing the Power BI admin portal config to enable subartifact scan.":::

    > [!Caution]
    > When you allow the security group you created (that has your Microsoft Purview managed identity as a member) to use read-only Power BI admin APIs, you also allow it to access the metadata (e.g. dashboard and report names, owners, descriptions, etc.) for all of your Power BI artifacts in this tenant. Once the metadata has been pulled into the Microsoft Purview, Microsoft Purview's permissions, not Power BI permissions, determine who can see that metadata.
  
    > [!Note]
    > You can remove the security group from your developer settings, but the metadata previously extracted won't be removed from the Microsoft Purview account. You can delete it separately, if you wish.

### Create scan

To create and run a new scan, do the following:

1. In the Microsoft Purview Studio, navigate to the **Data map** in the left menu.

1. Navigate to **Sources**.

1. Select the registered Power BI source.

1. Select **+ New scan**.

2. Give your scan a name. Then select the option to include or exclude the personal workspaces.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-setup.png" alt-text="Image showing Power BI scan setup.":::

    > [!Note]
    > Switching the configuration of a scan to include or exclude a personal workspace will trigger a full scan of PowerBI source.

3. Select **Test Connection** before continuing to next steps. If **Test Connection** failed, select **View Report** to see the detailed status and troubleshoot the problem.
    1. Access - Failed status means the user authentication failed. Scans using managed identity will always pass because no user authentication required.
    2. Assets (+ lineage) - Failed status means the Microsoft Purview - Power BI authorization has failed. Make sure the Microsoft Purview managed identity is added to the security group associated in Power BI admin portal.
    3. Detailed metadata (Enhanced) - Failed status means the Power BI admin portal is disabled for the following setting - **Enhance admin APIs responses with detailed metadata**

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-test-connection-status-report.png" alt-text="Screenshot of test connection status report page.":::

4. Set up a scan trigger. Your options are **Recurring**, and **Once**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/scan-trigger.png" alt-text="Screenshot of the Microsoft Purview scan scheduler.":::

5. On **Review new scan**, select **Save and run** to launch your scan.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/save-run-power-bi-scan-managed-identity.png" alt-text="Screenshot of Save and run Power BI source using Managed Identity.":::

### Scan same tenant using Self-hosted IR and Delegated authentication

This scenario can be used when Microsoft Purview and Power BI tenant or both, are configured to use private endpoint and deny public access. Additionally, this option is also applicable if Microsoft Purview and Power BI tenant are configured to allow public access.

For more information related to Power BI network, see [How to configure private endpoints for accessing Power BI](/power-bi/enterprise/service-security-private-links).

For more information about Microsoft Purview network settings, see [Use private endpoints for your Microsoft Purview account](catalog-private-link.md).

To create and run a new scan, do the following:

1. Create a user account in Azure Active Directory tenant and assign the user to Azure Active Directory role, **Power BI Administrator**. Take note of username and sign in to change the password.

1. Assign proper Power BI license to the user.  

1. Navigate to your Azure key vault.

1. Select **Settings** > **Secrets** and select **+ Generate/Import**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-key-vault.png" alt-text="Screenshot how to navigate to Azure Key Vault.":::

1. Enter a name for the secret and for **Value**, type the newly created password for the Azure AD user. Select **Create** to complete.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-key-vault-secret.png" alt-text="Screenshot how to generate an Azure Key Vault secret.":::

1. If your key vault isn't connected to Microsoft Purview yet, you'll need to [create a new key vault connection](manage-credentials.md#create-azure-key-vaults-connections-in-your-microsoft-purview-account) 
   
1. Create an App Registration in your Azure Active Directory tenant. Provide a web URL in the **Redirect URI**. Take note of Client ID(App ID).

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-create-service-principle.png" alt-text="Screenshot how to create a Service principle.":::
  
1. From Azure Active Directory dashboard, select newly created application and then select **App registration**. From **API Permissions**, assign the application the following delegated permissions and grant admin consent for the tenant:

   - Power BI Service Tenant.Read.All
   - Microsoft Graph openid
   - Microsoft Graph User.Read

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-delegated-permissions.png" alt-text="Screenshot of delegated permissions for Power BI Service and Microsoft Graph.":::
    
1. Under **Advanced settings**, enable **Allow Public client flows**.

2. In the Microsoft Purview Studio, navigate to the **Data map** in the left menu.

1. Navigate to **Sources**.

1. Select the registered Power BI source.

1. Select **+ New scan**.

1. Give your scan a name. Then select the option to include or exclude the personal workspaces.
   
    >[!Note]
    > Switching the configuration of a scan to include or exclude a personal workspace will trigger a full scan of PowerBI source.

1. Select your self-hosted integration runtime from the drop-down list. 

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-shir.png" alt-text="Image showing Power BI scan setup using SHIR for same tenant.":::

1. For the **Credential**, select **Delegated authentication** and select **+ New** to create a new credential.

1. Create a new credential and provide required parameters:
    
    - **Name**: Provide a unique name for credential
    - **Client ID**: Use Service Principal Client ID (App ID) you created earlier   
    - **User name**: Provide the username of Power BI Administrator you created earlier
    - **Password**: Select the appropriate Key vault connection and the **Secret name** where the Power BI account password was saved earlier.
    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-delegated-authentication.png" alt-text="Screenshot of the new credential menu, showing Power B I credential with all required values supplied.":::

1. Select **Test Connection** before continuing to next steps. If **Test Connection** failed, select **View Report** to see the detailed status and troubleshoot the problem
    1. Access - Failed status means the user authentication failed. Scans using managed identity will always pass because no user authentication required.
    2. Assets (+ lineage) - Failed status means the Microsoft Purview - Power BI authorization has failed. Make sure the Microsoft Purview managed identity is added to the security group associated in Power BI admin portal.
    3. Detailed metadata (Enhanced) - Failed status means the Power BI admin portal is disabled for the following setting - **Enhance admin APIs responses with detailed metadata**

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-test-connection-status-report.png" alt-text="Screenshot of test connection status report page.":::

1. Set up a scan trigger. Your options are **Recurring**, and **Once**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/scan-trigger.png" alt-text="Screenshot of the Microsoft Purview scan scheduler.":::

1. On **Review new scan**, select **Save and run** to launch your scan.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/save-run-power-bi-scan.png" alt-text="Screenshot of Save and run Power BI source.":::

## Next steps

Now that you've registered your source, follow the below guides to learn more about Microsoft Purview and your data.

- [Data Estate Insights in Microsoft Purview](concept-insights.md)
- [Lineage in Microsoft Purview](catalog-lineage-user-guide.md)
- [Search Data Catalog](how-to-search-catalog.md)
