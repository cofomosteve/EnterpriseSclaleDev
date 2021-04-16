| Enterprise-Scale Design Principles | ARM Template - EN | ARM Template - FR | Scale without refactoring |
|:-------------|:--------------|:--------------|:--------------|
|![Best Practice Check](https://azurequickstartsservice.blob.core.windows.net/badges/subscription-deployments/create-rg-lock-role-assignment/BestPracticeResult.svg)| [![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcofomosteve%2FEnterpriseSclaleDev%2Fmain%2Fdocs%2Freference%2Fadventureworks%2FarmTemplates%2Fes-hubspoke.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fcofomosteve%2FEnterpriseSclaleDev%2Fmain%2Fdocs%2Freference%2Fadventureworks%2FarmTemplates%2Fportal-es-hubspoke.json)| [![Deployer Vers Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcofomosteve%2FEnterpriseSclaleDev%2Fmain%2Fdocs%2Freference%2Fadventureworks%2FarmTemplates%2Fes-hubspoke.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fcofomosteve%2FEnterpriseSclaleDev%2Fmain%2Fdocs%2Freference%2Fadventureworks%2FarmTemplates%2Fportal-es-hubspoke.json) | Yes |

# Deploy Enterprise-Scale with hub and spoke architecture

The Enterprise-Scale architecture is modular by design and allow organizations to start with foundational landing zones that support their application portfolios and add hybrid connectivity with ExpressRoute or VPN when required. Alternatively, organizations can start with an Enterprise-Scale architecture based on the traditional hub and spoke network topology if customers require hybrid connectivity to on-premises locations from the begining.

This reference implementation also allows the deployment of platform services across Availability Zones (such as VPN or ExpressRoute gateways) to increase availability uptime of such services. 

## Customer profile

This reference implementation is ideal for customers that have started their Enterprise-Scale journey with an Enterprise-Scale foundation implementation and then there is a need to add connectivity on-premises datacenters and branch offices by using a traditional hub and spoke network architecture. This reference implementation is also well suited for customers who want to start with Landing Zones for their net new
deployment/development in Azure by implementing a network architecture based on the traditional hub and spoke network topology.

## How to evolve from Enterprise-Scale foundation

If customer started with a Enterprise-Scale foundation deployment, and if the business requirements changes over time, such as migration of on-premise applications to Azure that requires hybrid connectivity, you will simply create the **Connectivity** Subscription, place it into the **Platform > Connectivity** Management Group and assign Azure Policy for the hub and spoke network topology.

## Pre-requisites

To deploy this ARM template, your user/service principal must have Owner permission at the Tenant root.
See the following [instructions](https://docs.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin) on how to grant access.

### Optional prerequisites

The deployment experience in Azure portal allows you to bring in existing (preferably empty) subscriptions dedicated for platform management, connectivity and identity. It also allows you to bring existing subscriptions that can be used as the initial landing zones for your applications.

To learn how to create new subscriptions programatically, please visit this [link](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/programmatically-create-subscription?tabs=rest).

To learn how to create new subscriptions using Azure portal, please visit this [link](https://azure.microsoft.com/en-us/blog/create-enterprise-subscription-experience-in-azure-portal-public-preview/).

## What will be deployed?

By default, all recommendations are enabled and you must explicitly disable them if you don't want it to be deployed and configured.

- A scalable Management Group hierarchy aligned to core platform capabilities, allowing you to operationalize at scale using centrally managed Azure RBAC and Azure Policy where platform and workloads have clear separation.
- Azure Policies that will enable autonomy for the platform and the landing zones.
- An Azure subscription dedicated for **management**, which enables core platform capabilities at scale using Azure Policy such as:
  - A Log Analytics workspace and an Automation account
  - Azure Security Center monitoring
  - Azure Security Center (Standard or Free tier)
  - Azure Sentinel
  - Diagnostics settings for Activity Logs, VMs, and PaaS resources sent to Log Analytics
- An Azure subscription dedicated for **connectivity**, which deploys core Azure networking resources such as:
  - A hub virtual network
  - Azure Firewall (optional - deployment across Availability Zones)
  - ExpressRoute Gateway (optional - deployment across Availability Zones)
  - VPN Gateway (optional - deployment across Availability Zones)
  - Azure Private DNS Zones for Private Link
- (Optionally) An Azure subscription dedicated for **identity** in case your organization requires to have Active Directory Domain Controllers in a dedicated subscription.
- Landing Zone Management Group for **corp** connected applications that require connectivity to on-premises, to other landing zones or to the internet via shared services provided in the hub virtual network.
  - This is where you will create your subscriptions that will host your corp-connected workloads.
- Landing Zone Management Group for **online** applications that will be internet-facing, where a virtual network is optional and hybrid connectivity is not required.
  - This is where you will create your Subscriptions that will host your online workloads.
- Landing zone subscriptions for Azure native, internet-facing **online** applications and resources.
- Landing zone subscriptions for **corp** connected applications and resources, including a virtual network that will be connected to the hub via VNet peering.
- Azure Policies for online and corp-connected landing zones, which include:
  - Enforce VM monitoring (Windows & Linux)
  - Enforce VMSS monitoring (Windows & Linux)
  - Enforce Azure Arc VM monitoring (Windows & Linux)
  - Enforce VM backup (Windows & Linux)
  - Enforce secure access (HTTPS) to storage accounts
  - Enforce auditing for Azure SQL
  - Enforce encryption for Azure SQL
  - Prevent IP forwarding
  - Prevent inbound RDP from internet
  - Ensure subnets are associated with NSG

![Enterprise-Scale with connectivity](./media/es-hubspoke.png)

## Next steps

### From an application perspective:

Once you have deployed the reference implementation, you can create new subscriptions, or move an existing subscriptions to the **Landing Zones** > **Online** or **Corp**  management group, and finally assign RBAC to the groups/users who should use the landing zones (subscriptions) so they can start deploying their workloads.

Refer to the [Create Landing Zone(s)](../../EnterpriseScale-Deploy-landing-zones.md) article for guidance to create Landing Zones.


## File -> New -> Region

Companies wants to leverage new Azure regions and deploy the workload closer to the user; and, they will be adding new Azure regions as business demand arises. As a part of Enterprise-Scale design principle of policy-driven governance, they will be assigning policies in their environment with a number of regions they would like to use and policies will ensure their Azure Environment is setup correctly:

### Management

All reference customers have decided to use a single Log Analytics workspace. When the first region is enabled, they will deploy Log Analytics workspace in their management subscription. No action will be required when enabling subsequent Azure regions as Azure Policy will ensure all platform logging is routed to the workspace. Azure Policy is extensively used for various management operations. Refer to [How does Azure Policies in Enterprise-scale Landing Zone help?](./azpol.md) to learn more about various management and deployment operations enabled in Enterprise Scale landing zone via Azure Policy .

### Networking

Here customers are taking different architecture designs. The following examples are for the Contoso reference implementation:

A policy will continuously check if a Virtual WAN VHub already exist in "Connectivity" subscription for all enabled regions and create one if it does not. Configure Virtual WAN VHub to secure internet traffic from secured connections (spoke VNets inside Landing Zone) to the internet via Azure Firewall.

For all Azure Virtual WAN VHubs, Policies will ensure that Azure Firewall is deployed and linked to the existing global Azure Firewall Policy as well as the creation of a regional Firewall policy, if needed.


An Azure Policy will also deploy default NSGs and UDRs in Landing Zones and, while NSG will be linked to all subnets, UDR will only be linked to VNet injected PaaS services subnets. The Azure Policy will ensure that the right NSG and UDR rules are configured to allow control plane traffic for VNet injected services to continue to work but only for those Azure PaaS services that have been approved as per the [Service Enablement Framework](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/security-governance-and-compliance#whitelist-the-service-framework) described in this document. This is required as, when landing zone VNets get connected to Virtual WAN VHub, they will get the default route (0.0.0.0/0) configured to point to their regional Azure Firewall, hence UDR and NSG rules are required to protect and manage control plane traffic for VNet injected PaaS services (such as SQL MI).

For cross-premises connectivity, Policy will ensure that ExpressRoute and/or VPN gateways are deployed (as required by the regional VHub), and it will connect the VHub to on-premises using ExpressRoute (by taking the ExpressRoute Resource ID and authorization key as parameters). In case of VPN, Contoso can decide if they use their existing SD-WAN solution to automate the connectivity from branch offices into Azure via S2S VPN, or alternatively, Contoso can manually configure the CPE devices on the branch offices and then let Azure Policy to configure the VPN sites in Azure Virtual WAN. As Contoso is rolling out a SD-WAN solution to manage the connectivity of all their branches around the globe, their preference is to use the SD-WAN solution, which is a solution certified with Azure Virtual WAN, to connect all their branches to Azure.

## File -> New -> Landing Zone (Subscription)

Reference customer wants to minimize the time it takes to create Landing Zones and do not want central IT to become a bottleneck. Subscriptions will be the unit of management for the landing zones and each business owner will have access to an Azure Billing Profile that will allow them to create new subscriptions (a.k.a. Landing Zones) with an ability to delegate this task to their own IT teams.
Once new a subscription is provisioned, the subscription will be automatically placed in the desired management group and subject to any configured policy.

Networking:

1) Create Virtual Network inside Landing Zone and establish Virtual Network peering with VWAN VHub in the same Azure region
2) Create Default NSG inside Landing Zone with default rules e.g. no RDP/SSH from Internet
3) Ensure new subnets are created inside Landing Zone and have NSGs
4) Default NSG Rules cannot be modified e.g. RDP/SSH from Internet
5) Enable NSG Flow logs and connect it to Log Analytics Workspace in management Subscription.
6) Protect Virtual Network traffic across VHubs with NSGs.

IAM

1) Create Azure AD Group for Subscriptions access
2) Create Azure AD PIM Entitlement for the scope

# File -> New -> Sandbox

Sandbox Subscriptions are for experiment and validation only. Sandbox Subscriptions will not be allowed connectivity to Production and Policy will prevent the connectivity to on-premises Resources.

## File -> Delete -> Sandbox/Landing Zone

Susbcription will be moved to a decommissioned Management Group. Decommissioned Management Group policies will deny creation of new services and a Subscription cancellation request will be sent.
