---
# Mandatory fields.
title: Security for Azure Digital Twins solutions
titleSuffix: Azure Digital Twins
description: Learn about Azure Digital Twins security best practices.
author: baanders
ms.author: baanders # Microsoft employees only
ms.date: 03/01/2022
ms.topic: conceptual
ms.service: digital-twins

# Optional fields. Don't forget to remove # if you need a field.
# ms.custom: can-be-multiple-comma-separated
# ms.reviewer: MSFT-alias-of-reviewer
# manager: MSFT-alias-of-manager-or-PM-counterpart
---

# Secure Azure Digital Twins

This article explains Azure Digital Twins security best practices. It covers roles and permissions, managed identity, private network access with Azure Private Link, service tags, encryption of data at rest, and Cross-Origin Resource Sharing (CORS).

For security, Azure Digital Twins enables precise access control over specific data, resources, and actions in your deployment. It does so through a granular role and permission management strategy called [Azure role-based access control (Azure RBAC)](../role-based-access-control/overview.md).

Azure Digital Twins also supports encryption of data at rest.

## Roles and permissions with Azure RBAC

Azure RBAC is provided to Azure Digital Twins via integration with [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD).

You can use Azure RBAC to grant permissions to a *security principal*, which may be a user, a group, or an application service principal. The security principal is authenticated by Azure AD, and receives an OAuth 2.0 token in return. This token can be used to authorize an access request to an Azure Digital Twins instance.

### Authentication and authorization

With Azure AD, access is a two-step process. When a security principal (a user, group, or application) attempts to access Azure Digital Twins, the request must be *authenticated* and *authorized*. 

1. First, the security principal's identity is authenticated, and an OAuth 2.0 token is returned.
2. Next, the token is passed as part of a request to the Azure Digital Twins service, to authorize access to the specified resource.

The authentication step requires any application request to contain an OAuth 2.0 access token at runtime. If an application is running within an Azure entity such as an [Azure Functions](../azure-functions/functions-overview.md) app, it can use a *managed identity* to access the resources. Read more about managed identities in the next section.

The authorization step requires that an Azure role be assigned to the security principal. The roles that are assigned to a security principal determine the permissions that the principal will have. Azure Digital Twins provides Azure roles that encompass sets of permissions for Azure Digital Twins resources. These roles are described later in this article.

To learn more about roles and role assignments supported in Azure, see [Understand the different roles](../role-based-access-control/rbac-and-directory-admin-roles.md) in the Azure RBAC documentation.

#### Authentication with managed identities

[Managed identities for Azure resources](../active-directory/managed-identities-azure-resources/overview.md) is a cross-Azure feature that enables you to create a secure identity associated with the deployment where your application code runs. You can then associate that identity with access-control roles, to grant custom permissions for accessing specific Azure resources that your application needs.

With managed identities, the Azure platform manages this runtime identity. You don't need to store and protect access keys in your application code or configuration, either for the identity itself, or for the resources you need to access. An Azure Digital Twins client app running inside an Azure App Service application doesn't need to handle SAS rules and keys, or any other access tokens. The client app only needs the endpoint address of the Azure Digital Twins namespace. When the app connects, Azure Digital Twins binds the managed entity's context to the client. Once it's associated with a managed identity, your Azure Digital Twins client can do all authorized operations. Authorization will then be granted by associating a managed entity with an Azure Digital Twins Azure role (described below).

#### Authorization: Azure roles for Azure Digital Twins

Azure provides two Azure built-in roles for authorizing access to the Azure Digital Twins [data plane APIs](concepts-apis-sdks.md#overview-data-plane-apis). You can refer to the roles either by name or by ID:

| Built-in role | Description | ID | 
| --- | --- | --- |
| Azure Digital Twins Data Owner | Gives full access over Azure Digital Twins resources | bcd981a7-7f74-457b-83e1-cceb9e632ffe |
| Azure Digital Twins Data Reader | Gives read-only access to Azure Digital Twins resources | d57506d4-4c8d-48b1-8587-93c323f6a5a3 |

You can assign roles in two ways:
* Via the access control (IAM) pane for Azure Digital Twins in the Azure portal (see [Assign Azure roles using the Azure portal](../role-based-access-control/role-assignments-portal.md))
* Via CLI commands to add or remove a role

For more detailed steps on how to do so, try it out in the Azure Digital Twins [Connect an end-to-end solution](tutorial-end-to-end.md).

For more information about how built-in roles are defined, see [Understand role definitions](../role-based-access-control/role-definitions.md) in the Azure RBAC documentation. For information about creating Azure custom roles, see [Azure custom roles](../role-based-access-control/custom-roles.md).

##### Automating roles

When referring to roles in automated scenarios, it's recommended to refer to them by their IDs rather than their names. The names may change between releases, but the IDs won't, making them a more stable reference in automation.

> [!TIP]
> If you're assigning roles with a cmdlet, such as `New-AzRoleAssignment` ([reference](/powershell/module/az.resources/new-azroleassignment)), you can use the `-RoleDefinitionId` parameter instead of `-RoleDefinitionName` to pass an ID instead of a name for the role.

### Permission scopes

Before you assign an Azure role to a security principal, determine the scope of access that the security principal should have. Best practices dictate that it's best to grant only the narrowest possible scope.

The following list describes the levels at which you can scope access to Azure Digital Twins resources.
* Models: The actions for this resource dictate control over [models](concepts-models.md) uploaded in Azure Digital Twins.
* Query Digital Twins Graph: The actions for this resource determine ability to run [query operations](concepts-query-language.md) on digital twins within the Azure Digital Twins graph.
* Digital Twin: The actions for this resource provide control over CRUD operations on [digital twins](concepts-twins-graph.md) in the twin graph.
* Digital Twin relationship: The actions for this resource define control over CRUD operations on [relationships](concepts-twins-graph.md) between digital twins in the twin graph.
* Event route: The actions for this resource determine permissions to [route events](concepts-route-events.md) from Azure Digital Twins to an endpoint service like [Event Hubs](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md), or [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

### Troubleshoot permissions

If a user attempts to perform an action not allowed by their role, they may receive an error from the service request reading `403 (Forbidden)`. For more information and troubleshooting steps, see [Troubleshoot Azure Digital Twins failed service request: Error 403 (Forbidden)](troubleshoot-error-403-digital-twins.md).

## Managed identity for accessing other resources

Setting up an [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) *managed identity* for an Azure Digital Twins instance can allow the instance to easily access other Azure AD-protected resources, such as [Azure Key Vault](../key-vault/general/overview.md). The identity is managed by the Azure platform, and doesn't require you to provision or rotate any secrets. For more about managed identities in Azure AD, see [Managed identities for Azure resources](../active-directory/managed-identities-azure-resources/overview.md). 

Azure supports two types of managed identities: system-assigned and user-assigned. Currently, Azure Digital Twins supports only *system-assigned identities*. 

You can use a system-assigned managed identity for your Azure Digital Instance to authenticate to a [custom-defined endpoint](concepts-route-events.md#create-an-endpoint). Azure Digital Twins supports system-assigned identity-based authentication to endpoints for [Event Hubs](../event-hubs/event-hubs-about.md) and [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) destinations, and to an [Azure Storage Container](../storage/blobs/storage-blobs-introduction.md) endpoint for [dead-letter events](concepts-route-events.md#dead-letter-events). [Event Grid](../event-grid/overview.md) endpoints are currently not supported for managed identities.

For instructions on how to enable a system-managed identity for Azure Digital Twins and use it to route events, see [Route events with a managed identity](how-to-route-with-managed-identity.md).

## Private network access with Azure Private Link

[Azure Private Link](../private-link/private-link-overview.md) is a service that enables you to access Azure resources (like [Azure Event Hubs](../event-hubs/event-hubs-about.md), [Azure Storage](../storage/common/storage-introduction.md), and [Azure Cosmos DB](../cosmos-db/introduction.md)) and Azure-hosted customer and partner services over a private endpoint in your [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md). 

Similarly, you can use private endpoints for your Azure Digital Twins instance to allow clients located in your virtual network to securely access the instance over Private Link. Configuring a private endpoint for your Azure Digital Twins instance enables you to secure your Azure Digital Twins instance and eliminate public exposure. Additionally, it helps avoid data exfiltration from your [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md).

The private endpoint uses an IP address from your Azure VNet address space. Network traffic between a client on your private network and the Azure Digital Twins instance traverses over the VNet and a Private Link on the Microsoft backbone network, eliminating exposure to the public internet. Here's a visual representation of this system:

:::image type="content" source="media/concepts-security/private-link.png" alt-text="Diagram showing a network that is a protected VNET with no public cloud access, connecting through Private Link to an Azure Digital Twins instance.":::

Configuring a private endpoint for your Azure Digital Twins instance enables you to secure your Azure Digital Twins instance and eliminate public exposure, as well as avoid data exfiltration from your VNet.

For instructions on how to set up Private Link for Azure Digital Twins, see [Enable private access with Private Link](./how-to-enable-private-link.md).

### Design considerations 

When working with Private Link for Azure Digital Twins, here are some factors you may want to consider:
* Pricing: For pricing details, see [Azure Private Link pricing](https://azure.microsoft.com/pricing/details/private-link). 
* Regional availability: For Azure Digital Twins, this feature is available in all the Azure regions where Azure Digital Twins is available. 
* Maximum number of private endpoints per Azure Digital Twins instance: 10

For information on the limits of Private Link, see [Azure Private Link documentation: Limitations](../private-link/private-link-service-overview.md#limitations).

## Service tags

A *service tag* represents a group of IP address prefixes from a given Azure service. Microsoft manages the address prefixes encompassed by the service tag and automatically updates the service tag as addresses change, minimizing the complexity of frequent updates to network security rules. For more information about service tags, see [Virtual network tags](../virtual-network/service-tags-overview.md). 

You can use service tags to define network access controls on [network security groups](../virtual-network/network-security-groups-overview.md#security-rules) or [Azure Firewall](../firewall/service-tags.md), by using service tags in place of specific IP addresses when you create security rules. By specifying the service tag name (in this case, AzureDigitalTwins) in the appropriate **source** or **destination** field of a rule, you can allow or deny the traffic for the corresponding service. 

Below are the details of the AzureDigitalTwins service tag.

| Tag | Purpose | Can use inbound or outbound? | Can be regional? | Can use with Azure Firewall? |
| --- | --- | --- | --- | --- |
| AzureDigitalTwins | Azure Digital Twins<br>Note: This tag or the IP addresses covered by this tag can be used to restrict access to endpoints configured for [event routes](concepts-route-events.md). | Inbound | No | Yes |

### Using service tags for accessing event route endpoints 

Here are the steps to access [event route](concepts-route-events.md) endpoints using service tags with Azure Digital Twins.

1. First, download this JSON file reference showing Azure IP ranges and service tags: [Azure IP Ranges and Service Tags](https://www.microsoft.com/download/details.aspx?id=56519). 

2. Look for "AzureDigitalTwins" IP ranges in the JSON file.  

3. Refer to the documentation of the external resource connected to the endpoint (for example, the [Event Grid](../event-grid/overview.md), [Event Hubs](../event-hubs/event-hubs-about.md), [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md), or [Azure Storage](../storage/blobs/storage-blobs-overview.md) for [dead-letter events](concepts-route-events.md#dead-letter-events)) to see how to set IP filters for that resource.

4. Set IP filters on the external resource(s) using the IP ranges from Step 2.  

5. Update the IP ranges periodically as required. The ranges may change over time, so it's a good idea to check them regularly and refresh them when needed. The frequency of these updates can vary, but it's a good idea to check them once a week.

## Encryption of data at rest

Azure Digital Twins provides encryption of data at rest and in-transit as it's written in our data centers, and decrypts it for you as you access it. This encryption occurs using a Microsoft-managed encryption key.

## Cross-Origin Resource Sharing (CORS)

Azure Digital Twins doesn't currently support Cross-Origin Resource Sharing (CORS). As a result, if you're calling a REST API from a browser app, an [API Management (APIM)](../api-management/api-management-key-concepts.md) interface, or a [Power Apps](/powerapps/powerapps-overview) connector, you may see a policy error.

To resolve this error, you can do one of the following actions:
* Strip the CORS header `Access-Control-Allow-Origin` from the message. This header indicates whether the response can be shared. 
* Alternatively, create a CORS proxy and make the Azure Digital Twins REST API request through it. 

## Next steps

* See these concepts in action in [Set up an instance and authentication](how-to-set-up-instance-portal.md).

* See how to interact with these concepts from client application code in [Write app authentication code](how-to-authenticate-client.md).

* Read more about [Azure RBAC](../role-based-access-control/overview.md).