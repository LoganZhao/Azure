# Azure VPN

[[_TOC_]]
## Short Link  
https://aka.ms/AnpVPNWiki

## Azure VPN Work Items

::: query-table a03cba6d-f949-4cba-95d1-dc9e3cad7be1
:::

## Feature Overview
----
Azure Gateway is Microsoft's first-party solution for Vnet VPN solutions. It
is comprised of at least two VM instances that negotiate and manage a
VPN connection with a 'VPN peer' in another Vnet or a customer's
on-premises site.

Read this [article](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)

## How it works / Architecture
----

An Azure gateway consists of a set of two Windows Server 2012R2 VMs with
custom VPN code running. They are in a PaaS *Cloud Service* instance
where in *active/passive scenario*, one VM is acting as the ‘active’
instance managing the VPN connection while the other VM is standing by
in a ‘passive’ mode waiting to take over when needed. This
Active/Passive gateway mode is the default configuration for Azure
gateways. If the ‘active’ or ‘primary’ instance goes down due to
required reboot or failure the ‘passive’ or ‘secondary’ VM will take
over running the VPN tunnel. These two VMs coordinate who is ‘primary’
via a blob lease on the gateway storage account. The ‘primary’ VM has a
lock on the gateway storage account. At regular/short intervals the
‘secondary’ tries to obtain a lease or lock on the blob. If the
‘primary’ VM is up and functioning, the secondary’s request is denied.

### Architecture Diagrams
---------------------

#### Active / Passive Deployment

![Active / Passive Deployment diagram](/.attachments/684px-AzGtwActivePassiveArchitecture.png)

#### Azure Gateway v2 Internals

![Azure Gateway v2 Internals diagram](/.attachments/ANPbrkv2archdiagram.png)

### Architecture Flow walk-throughs
-------------------------------

#### Control Paths
Azure API (ARM) --> Network Resource Provider (NRP) --> Regional Network
Manager (RNM) --> Gateway Manager (Async Worker) -->
Gateway Tenants

#### Data Paths

-   ER GWTs have active/active datapaths and control paths by default.
    You can delve deeper into this by looking at our “MSEE and GWT
    datapath” documentation.
-   VPN GWTs have active/passive datapaths by default. A so called
    active/active VPN GWT deployment might be active/active on its
    datapath, on its control path or on both. It depends on how customer
    architects the solution.
-   Traffic from Azure VMs to on-premises bypass the ER GWTs, but does
    not bypass the VPN GWTs.

A note about encryption:

-   ER GWTs do not encrypt nor decrypt traffic. At all.
-   VPN GWts do encrypt and decrypt traffic. They never forward traffic
    through a VPN tunnel without encrypting it, nor they forward traffic
    to the backend VMs without decrypting it.

Notes about Coexistence:

-   On a normal VPN or ER scenario it is the GWT who sends its learned
    routes to the VNet, so the VMs can learned them. In a coexistence
    scenario a solution was found to deal with collisions and overlaps
    in an elegant way that would not confuse customers.
-   A coexistence scenario has two pair of GWT instances: One for ER and
    the other for VPN. These two pairs of GWT instances establish BGP
    sessions between them, so the ER GWT pair will learn routes from the
    VPN pair. This is not to push these routes up to the MSEEs, but just
    because under this scenario only the ER GWT pair sends its learned
    routes to the VNet.
-   A common issue in coexistence scenarios where we have an ER circuit
    and a S2S VPN is for the S2S VPN to not work, even though the tunnel
    is up and everything looks as expected under normal troubleshooting.
    In these scenarios there might be a problem in the BGP session
    between the VPN and the ER gateways, in which case the ER gateway
    will not be learning any routes from the S2S VPN, so those routes
    will not be plumbed on the VM's VFP, thus breaking the S2S
    connectivity. This has been proven costly to troubleshoot due to the
    fact that an ER GWT pair is hardly seen as a potential culprit in a
    S2S tunnel between two VPN GWT pairs. Knowing how these coexistence
    scenarios are architectured should enable the engineer to think out
    of the box with their troubleshooting and find the issues sooner.
-   The usual troubleshooting implies reviewing GatewayTenantLogs tables
    for both GWT pairs (ER and VPN) and dumping the BGP tables for both
    from ACIS / Jarvis actions. The purpose of that is to confirm the
    health of the BGP connection between ER and VPN GWT pairs.

## Training and Other Resources
----

  - [Edu-LAB for VPN - Session One](https://web.microsoftstream.com/video/a98280cf-300e-4965-b6cf-e9efd88292ac)
  - [Gateway Tenant v3
    Introduction](https://microsoft.sharepoint.com/:v:/t/WoA/ERfDhwHqnjVLiTlvALCr1foBCocfWL05b1L4HyIafFZ73Q?e=915fdbf8b1ef41208b65184913b230bd)
