# Transfer subscription from one Active Directory tenant to another

## Documentation

General:

- [Transfer an Azure subscription to a different Azure AD directory](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription)
- [application (App registrations) resource type](https://learn.microsoft.com/en-us/graph/api/resources/application?view=graph-rest-1.0)
- [Python SDK for Microsoft Microsoft Graph API](https://github.com/microsoftgraph/msgraph-sdk-python-core)

Export:

- [Understand the impact of transferring a subscription](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#understand-the-impact-of-transferring-a-subscription)
- [Check Azure SQL databases with Azure AD authentication](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#list-azure-sql-databases-with-azure-ad-authentication)
- [List role assignments](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#save-all-role-assignments)
- [List custom roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#save-custom-roles)
- [List managed identities](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#list-role-assignments-for-managed-identities)
- [List Key Vault access policies](https://learn.microsoft.com/en-us/azure/role-based-access-control/transfer-subscription#list-key-vaults)

Import:

- [Associate or add an Azure subscription to your Azure Active Directory tenant](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory)

## Integration

### Export

General steps:

1. Make sure no unrecovable resources are present in the subscription
2. Iterate on every resources to export the configuration

Sequence diagram:

```mermaid
sequenceDiagram
    autonumber

    actor Admin
    actor Script
    participant Local database
    participant Azure (REST API)
    participant Old AAD (Microsoft Graph API)

    script ->> Azure (REST API): Confirm there is no Azure SQL databases with Azure AD authentication integration enabled
    script ->> Azure (REST API): Confirm there is no Azure database for MySQL with Azure AD authentication integration enabled
    script ->> Azure (REST API): Confirm there is no Azure Kubernetes Service
    script ->> Azure (REST API): Confirm there is no Azure Active Directory Domain Services
    script ->> Azure (REST API): Confirm there is no Microsoft Dev Box
    script ->> Azure (REST API): Confirm there is no Azure Deployment Environments

    alt Tests failed
        script --> script: Exit now
    end

    par Custom roles
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): Get the template
            script ->> Local database: Persist the template
        end

    and System-assigned managed identities
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): Get the template
            script ->> Local database: Persist the template
        end

    and User-assigned managed identities
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): Get the template
            script ->> Local database: Persist the template
        end

    and App registrations
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): Get the template
            script ->> Local database: Persist the template
        end

    and Azure Storage & Azure Data Lake Storage Gen2
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): ACLs
            script ->> Local database: Persist the template
        end

    and Azure Data Lake Storage Gen1
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): ACLs
            script ->> Local database: Persist the template
        end

    and Azure Files
        loop Iterate on all resources from Azure
            script ->> Old AAD (Microsoft Graph API): ACLs
            script ->> Local database: Persist the template
        end
    end
```

### Import

General steps:

1. Make sure the user have access to both directories
2. Execute the subscription transfer (wait for the transfer to complete)
3. Iterate on every resources to either: re-enable or import the configuration

Sequence diagram:

```mermaid
sequenceDiagram
    autonumber

    actor Admin
    actor Script
    participant Local database
    participant Azure (web portal)
    participant Azure (REST API)
    participant Old AAD (Microsoft Graph API)
    participant New AAD (Microsoft Graph API)
    participant Azure Key Vault
    participant Azure File Sync

    Admin ->> Old AAD (Microsoft Graph API): Make sure user has "Owner" role on the subscription
    Admin ->> New AAD (Microsoft Graph API): Make sure user exists in the directory
    Admin ->> Azure (web portal): Click on "change directory" and apply
    Admin ->> Admin: Wait from minutes to hours

    par Azure File Sync
        loop Iterate on all hardware
            Script ->> Azure File Sync: Update directory
        end

    and Azure Key Vaults
        loop Iterate on all resources Azure Key Vault resources from Azure
            Script ->> Azure Key Vault: Change tenant ID

            loop Iterate on access policies
                script ->> Azure Key Vault: Get the access policy
                script ->> Azure Key Vault: Create the fixed access policy
                script ->> Azure Key Vault: Delete the old access policy
            end
        end

    and Custom roles
        loop Iterate on all resources from database
            script ->> Local database: Read the template
            script ->> New AAD (Microsoft Graph API): Create the role

            loop Iterate on assignments
                script ->> New AAD (Microsoft Graph API): Create the role assignment
            end
        end

    and System-assigned managed identities
        loop Iterate on all resources from database
            script ->> Local database: Read the template
            script ->> New AAD (Microsoft Graph API): Enable the identity

            loop Iterate on assignments
                script ->> New AAD (Microsoft Graph API): Create the role assignment
            end
        end

    and User-assigned managed identities
        loop Iterate on all resources from database
            script ->> Local database: Read the template
            script ->> New AAD (Microsoft Graph API): Create the identity

            loop Iterate on assignments
                script ->> New AAD (Microsoft Graph API): Create the role assignment
            end
        end

    and App registrations
        loop Iterate on all resources from database
            script ->> Local database: Read the template
            script ->> New AAD (Microsoft Graph API): Create the app
        end

    and Azure Storage & Azure Data Lake Storage Gen2
        loop Iterate on all resources from database
            loop Iterate on ACLs
                script ->> Local database: Read the template
                script ->> Azure (REST API): Create the ACL
            end
        end

    and Azure Data Lake Storage Gen1
        loop Iterate on all resources from database
            loop Iterate on ACLs
                script ->> Local database: Read the template
                script ->> Azure (REST API): Create the ACL
            end
        end

    and Azure Files
        loop Iterate on all resources from database
            loop Iterate on ACLs
                script ->> Local database: Read the template
                script ->> Azure (REST API): Create the ACL
            end
        end
    end
```
