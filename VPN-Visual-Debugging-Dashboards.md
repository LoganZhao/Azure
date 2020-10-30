# VPN Visual Debugging Dashboards

[[_TOC_]]

## Overview

In early/mid-2019, the VPN Product Group, as a part of Project Fulcrum, built all-new Jarvis Dashboards to assist CSS in supporting Azure VPN Gateway SRs. 

Pain points prior to these dashboards include:

- Too many log sources, SEs aren't necessarily sure which to investigate
- SEs are unsure which data is relevant in logs
- Jarvis dgrep limits search window to a *max* of 7 days

This document outlines the new Jarvis Dashboards, how to use them, and some example scenarios. This information is part of a Feature Friday presentation, which can be found at the link below:

[WATCH: Azure VPN Feature Friday](https://microsoft.sharepoint.com/:v:/r/teams/WAG/AzureNetworking/aznetcce/Shared%20Documents/Feature%20Friday/VPN%20Gateway/VPN%20GW%20debugging%20dashboards%20and%20Insights/190524%20EMEA-Americas%20Feature%20Friday%20-%20VPN%20GW%20Debugging%20Dashboards%20(EMEA_Americas).mp4?csf=1&e=eEJ72I)

## Visual Debugging through Dashboards

This is the link to the Jarvis VPN Debugging dashboards: <https://jarvis-west.dc.ad.msft.net/dashboard/BrkProd/BrkGWT/VpnGateway>. It can also be retrieved from [Azure Support Center](https://azuresupportcenter.msftcloudes.com/) under the new `Visual Debugging` tab on the Gateway resource:

![vpndash01.jpg](/.attachments/vpndash01.jpg)

### Available Dashboards Overview

The VPN Visual Debugging dashboards are broken into four child Jarvis dashboards:

- Control Path
  - This details GWM control-path operations and statuses for the gateway. This includes things like GWT upgrades, customer-initiated operations, etc.
- Tenant General
  - Provides details about the tenant itself. This includes things like number of connections, instanceHeartbeat, etc
- Tenant Tunnel Stats
  - Provides statistics per-tunnel on the gateway. Bytes/packets information, bandwidth usage, etc.
- Tenant System Stats
  - What's the compute resource of the GWT instance - CPU/Memory, etc.

### Dashboard Functionality

Every dashboard has filter capabilities (if linked from ASC, these will be automatically filled). Each of these can be found via Azure Support Center if needed:

- GatewayId
- DeploymentId / TenantId
- CustomerSubscriptionId
- RoleInstance
- Primary/Secondary Instance
- TunnelIp (RemoteIdentifier)

Each dashboard also has relevant contextual links to associated Jarvis dgrep Logs to investigate an issue further. If you click on the time series on a particular dashboard, it'll bring you to the relevant logs automatically, no searching required.

## Dashboards In-Depth

Here are the four currently-available dashboards, with details.

NOTE: In each dashboard, ensure you have `Toggle Drilldown` enabled, so the click-through links work:

![vpndash03.jpg](/.attachments/vpndash03.jpg)

### Control Path Dashboard

Filters Available:

- Account: `BrkProd` (Should always be `BrkProd` unless otherwise advised by the VPN PG or in extremely rare customer scenarios)
- CustomerSubscriptionId
- GatewayId
- ArmId

**NOTE**: if these dashboards show "This Query Returned no Results", it means the specific dashboard did not log any relevant events during the timeframe.

#### Failed GatewayManager (PUT/DELETE) Sdk Operations

This outlines any PUT/DELETE operation failures that have occurred, along with the reason.

- Example - Failed LocalNetworkGateway deletion due to `DeleteLngWithExistingConnectionsNotAllowed`

    ![vpndash02.jpg](/.attachments/vpndash02.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayManagerLogs` table in Jarvis Logs

#### All Work Item Operations

This outlines all GWM work items on the gateway - particularly any Gateway Upgrades that may have taken place.

- Example - Gateway upgrade occurred:

    ![vpndash04.jpg](/.attachments/vpndash04.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > AsyncWorkerLogs` table in Jarvis Logs

### Tenant General Dashboard

Filters Available:

- Account: `BrkProd` (Should always be `BrkProd` unless otherwise advised by the VPN PG or in extremely rare customer scenarios)
- GatewayId
- Deploymentid
- IsPrimary
- RoleInstance

#### Gateway RoleInstance Heartbeat

This graph shows the heartbeat of the currently-active gateway (`GatewayTenantWorkerIN_0/1`), along with the GWT version of those GWTs.

- Example - Gateway failover & upgrade occurred (you can see the heartbeat swap to the other GW, and you can see that the old GWT version dropped to zero and the new GWT version went to `1`)

    ![vpndash05.jpg](/.attachments/vpndash05.jpg)

#### GatewayConnectionCount

Shows the number of active connections on the gateway, per instance. **NOTE**: This is the number of *connection objects* per GWT - for Active-Active connections, you will have **one** connection object spread across **two** GatewayTenants. This is how you may see 0.5 connections active per GWT.

- Example - two connections to start, then changed to four connections, including some dropped connections:

    ![vpndash06.jpg](/.attachments/vpndash06.jpg)

- Example - Active-Active connection object spread across both GWTs during a failover/upgrade:

    ![vpndash07.jpg](/.attachments/vpndash07.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayHealthFiveMinutes` table in Jarvis Logs

#### BlobLeaseStatus

Shows the currently-leased blobs for the VPN Gateway. Each gateway keeps three blob leases to keep itself alive in case we have storage outages in a region. This is useful to see if there are any blob lease issues. You'll also see new blob leases when a GW upgrades.

- Example - an Active-Active gateway, which means there are actually *two* sets of blob leases - `GatewayBlobLease` (GatewayTenantIN_0) and `GatewayBlobLease1` (GatewayTenantWorkerIN_1). This example is during a GW upgrade, so you see each set of three blob leases switch:

    ![vpndash09.jpg](/.attachments/vpndash09.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayTenantBlobLeaseOperations` table in Jarvis Logs

#### Global Gateway Flow Counters

Provides insight into the active flows on the gateway. Useful for telling if a customer is over-utilizing the gateway. Remember, Azure has a limitation on the number of active flows per VM, and can be found in the Azure Docs: [Virtual Machine Network Throughput](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-machine-network-throughput#flow-limits-and-recommendations).

- Example - A gateway nearing 500K flows per tenant:

    ![vpndash08.jpg](/.attachments/vpndash08.jpg)

**NOTE**: There is a graph for per-tunnel flows in the `TenantTunnelStats > Tunnel Flow Counters` graph.

#### Failed Gateway Operaton Count

Shows failed opeations on the gateway along with the reason why the operation failed.

- Example - Some operations failing due to TimeoutException and CommunicationException (GWM cannot contact the GW, likely due to NSG or UDR):

    ![vpndash10.jpg](/.attachments/vpndash10.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayConfigurationlogs` and `BrkGWM > GatewayTenantInsightEvents` table in Jarvis Logs

#### Wmi Operations & Failed Wmi Operations

[Wmi](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi) operations are a Windows OS construct that is the mechanism that GatewayManager uses to make changes to the custom IKE driver in GWTv3. When you make a change at the Azure-level (add a connection, change an LNG IP, etc), the change is written to the IKE driver inside the OS. These graphs will show you those operations, along with any failures that may have occurred. **NOTE** If you have *anything* in `Failed Wmi Operations` around the time of your customer's issue, please consult ANPTA via Microsoft Teams immediately, as it means that the GW IKE driver was unable to be properly configured.

- Example - Wmi operations that result from a PUT operation on the gateway:

    ![vpndash11.jpg](/.attachments/vpndash11.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayConfigurationLogs` table in Jarvis Logs

#### SLB Probe Status

This shows SLB responses to the various necessary ports used by the VPN Gateway. If you are having issues with your gateway that *aren't* related to an upgrade or customer operation, and you notice that some SLB probes seem to be down or flapping, it could indicate an issue and would merit further investigation.

- Example - Showing SLB probes "flapping" due to a GWT upgrade (this would be expected as the GWT reboots)

    ![vpndash12.jpg](/.attachments/vpndash12.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > SlbEventsTable` table in Jarvis Logs

#### SLB Probes Received

Shows how many raw probes the GWTs received from SLB. This can be relevant if SLB stops probing the GWTs for some reason, which would cause them to be brought offline and no traffic would pass. These should generally all be stable, aside from upgrades, failovers, customer operations, etc.

- Example - Showing normal SLB probes received. Notice that they are extremely stable:

    ![vpndash13.jpg](/.attachments/vpndash13.jpg)

- Example - Showing not-normal SLB probes received. Notice that probes drop to zero and never recover after a GWT upgrade:

    ![vpndash14.jpg](/.attachments/vpndash14.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > SlbEventsTable` table in Jarvis Logs

#### BGP Peer Status & BGP Peer Events

Shows the status of all connected BGP peers per GWT (relevant if Active-Active), along with what they are (IPSecOnPremEndpoint, IPSecActiveActiveGateway, ExpressRouteGateway), and any relevant events. Useful to tell if a customer's BGP peer was connected at a given time, or if routes were added/removed at a given time..

- Example - Showing a BGP peer drop during a GWT failover & upgrade:

    ![vpndash15.jpg](/.attachments/vpndash15.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayTenantLogs` table in Jarvis Logs, Filtered to show `<BGP>` events.

- Example - Showing BGP routes being added and removed during a GWT failover & upgrade:

    ![vpndash16.jpg](/.attachments/vpndash16.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > GatewayTenantBgpEvents` table in Jarvis Logs to find out exactly what routes were added/removed.

### TenantTunnelStats Dashboard

Filters Available:

- Account: `BrkProd` (Should always be `BrkProd` unless otherwise advised by the VPN PG or in extremely rare customer scenarios)
- GatewayId
- DeploymentId

#### Tunnel Connection State

Shows the current state of each tunnel (per localNetworkGateway VIP) on each GWT instance to tell if it is UP or DOWN at a given time. Useful to find out if an IKE tunnel dropped for some time.

- Example: Showing some recent IKE drops on a gateway:

    ![vpndash17.jpg](/.attachments/vpndash17.jpg)

- Click-through Drilldown links will bring you to relevant `BrkGWM > TunnelInsightEvent` table in Jarvis Logs to find out exactly what happened during that timeframe to drop the tunnel.

#### Tunnel Bandwidth in Bits per Second

Shows the ingress, egress and total bandwidth through each tunnel during the timeframe. Metric is in bits per second. Useful to tell if one tunnel is extra talkative. Also useful to identify drops in throughput that could indicate some sort of problem with the tunnel.

- Example: Showing some throughput (topping out at ~1750Kbps or ~1.75Mbps) that spikes, then drops, then spikes again:

    ![vpndash18.jpg](/.attachments/vpndash18.jpg)

#### Tunnel Packet Count

Shows the ingress, egress and total packet counts through each tunnel during the timeframe. Metric is in packets per time period. Useful to tell if one tunnel is extra talkative. Also useful to identify drops in packets that could indicate some sort of problem with the tunnel.

- Example: Showing a spike to 300k packets in a somewhat short time period. Could have an affect on other tunnels if one tunnel is flooding the gateway with packets:

    ![vpndash19.jpg](/.attachments/vpndash19.jpg)

#### Tunnel Packet Drop Count

Shows the number of packets that were dropped by the gateway along with the reason for the drop. 

- Example: Showing dropped packets due to Traffic Selectors mismatch after a re-key:

    ![vpndash20.jpg](/.attachments/vpndash20.jpg)

#### Peak Bandwidth over 5 mins

Shows the **peak** bandwidth observed every 5 minutes on the gateway. While the average throughput could be lower, you may see massive spikes over an extremely short period of time that don't necessarily show in the `Tunnel Bandwidth in Bits per Second` graph. This is useful to identify those spikes.

- Example: Showing a spike to 80Mbps, also confirming that the bandwidth dropped to nearly zero during a timeframe:

    ![vpndash21.jpg](/.attachments/vpndash21.jpg)

#### Peak Packets/sec over 5 mins

Shows the **peak** packet/sec observed every 5 minutes on the gateway. While the average packet/sec could be lower, you may see massive spikes over an extremely short period of time that don't necessarily show in the `Tunnel Packet Count` graph. This is useful to identify those spikes.

- Example: Showing a spike to 15K packets/sec, also confirming that the packets/sec dropped to nearly zero during a timeframe:

    ![vpndash22.jpg](/.attachments/vpndash22.jpg)

### TenantSystemStats Dashboard

- Account: `BrkProd` (Should always be `BrkProd` unless otherwise advised by the VPN PG or in extremely rare customer scenarios)
- GatewayId
- DeploymentId

This dashboard details the usual resource usage of the GatewayTenantWorker VMs that the gateways are built upon. They're fairly self-explanatory, but users should be wary of any discrepancies that may cause issues with the VPN tunnels - CPU usage spiked to 100%, etc. This may be indicative of an issue with the GW or unexpected behavior on the VPN tunnels.

## Frequently Asked Questions

- Do these dashboards work for GWTv2? Are stats populated for gateways prior to moving to GWTv3?
  - Dashboards do work for GWTv2 and data would be populated for GWTv3 gateways from when they were GWTv2 - aside from some graphs which are specific to the GWTv3-specific IKE driver. For example `Wmi Operations` and some graphs in `TenantTunnelStats` will be unavailable.
  - If the GWT version is 19.* or greater, you will see all graphs, irrespective of the GWTv2/GWTv3 distinction.
  - This also is similar for RouteBased vs PolicyBased - PolicyBased will have everything that doesn't involve the IKEv2 custom driver.
- Sometimes I see metrics that should be either `0` or `1` as some number betwen - for example, on `Tunnel Connection Stat`, I see `0.8` when i should see `0` or `1`.
  - This is due to Jarvis Dashboard limitations of rendering of the graphs. If you're too zoomed out, you may notice that Jarvis provides the "average". Best practice is to zoom in as much as possible to identify your issue.
  - Even if fully zoomed in, Jarvis provides per-minute averages, and the statistics are sometimes per-second, so Jarvis averages them.
- Do you *have* to filter by a single gateway, or can we filter by other means?
  - You can mix and match filters as you please - if you want to filter by a localNetworkGateway VIP, you can theoretically check out statistics of all Azure gateways that are connected to that customer VIP.

## Example Scenarios

Below you will find some real-world example customer scenarios presented by the VPN Product Group, and how the VPN Visual Debugging dashboards helped solve the issue much more quickly than via Jarvis/Kusto Logs alone.

### Datapath Downtime

**Scenario:** Customer has an active-active gateway with BGP, but finds that he experienced downtime of ~10mins when a gateway tenant goes down, when he should see no drops at all.

Overall debugging steps:

1. Check `TenantTunnelStats` Dashboard to verify VPN tunnel state
2. Check `Control Path` Dashboard to verify if customer did any operations to cause downtime
3. Check Instance State in `Tenant General` Dashboard
4. Check BGP State in `Tenant General` Dashboard

Please find this debugging scenario in the [Feature Friday session](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/aznetcce/Shared%20Documents/Feature%20Friday/VPN%20Gateway/190524%20EMEA-Americas%20Feature%20Friday%20-%20VPN%20GW%20Debugging%20Dashboards%20(EMEA_Americas).mp4?csf=1&e=Q2531S&cid=705c5b31-85f4-4ed0-b760-2faaa18997e1) at the `26:35` timestamp.

### Performance Hit

**Scenario:** Customer gateway throughput decreases intermittently.

Overall debugging steps:

1. Check `TenantTunnelStats` Dashboard to verify VPN tunnel state
2. Check `Control Path` Dashboard to verify if customer did any operations to cause downtime
3. Check `Tenant General` Dashboard to see if there was a set configuration
4. Check `TenantSystemStats` Dashboard to see if there are any anomalies in the GWTs.

Please find this debugging scenario in the [Feature Friday session](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/aznetcce/Shared%20Documents/Feature%20Friday/VPN%20Gateway/190524%20EMEA-Americas%20Feature%20Friday%20-%20VPN%20GW%20Debugging%20Dashboards%20(EMEA_Americas).mp4?csf=1&e=Q2531S&cid=705c5b31-85f4-4ed0-b760-2faaa18997e1) at the `33:35` timestamp.

### Performance Hit after PUT Operation

**Scenario:** Customer gateway throughput decreases dramatically after PUT operation.

1. Check `TenantTunnelStats` Dashboard to verify VPN tunnel state
2. Check `Control Path` Dashboard to verify if customer did any operations to cause downtime
3. Check `Tenant General` Dashboard to see if there was a set configuration

Please find this debugging scenario in the [Feature Friday session](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/aznetcce/Shared%20Documents/Feature%20Friday/VPN%20Gateway/190524%20EMEA-Americas%20Feature%20Friday%20-%20VPN%20GW%20Debugging%20Dashboards%20(EMEA_Americas).mp4?csf=1&e=Q2531S&cid=705c5b31-85f4-4ed0-b760-2faaa18997e1) at the `39:45` timestamp.

### Datapath Stops Working After Moving to A/A

**Scenario:** Customer moved from active-passive to active-active, and datapath stopped working.

1. Check `Control Path` Dashboard to verify if customer did any operations to cause downtime
2. Check `Tenant General` Dashboard to check the exact time of PUT operation
3. Check `TenantTunnelStats` Dashboard to verify VPN tunnel state and bandwidth

Please find this debugging scenario in the [Feature Friday session](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/aznetcce/Shared%20Documents/Feature%20Friday/VPN%20Gateway/190524%20EMEA-Americas%20Feature%20Friday%20-%20VPN%20GW%20Debugging%20Dashboards%20(EMEA_Americas).mp4?csf=1&e=Q2531S&cid=705c5b31-85f4-4ed0-b760-2faaa18997e1) at the `48:45` timestamp.
