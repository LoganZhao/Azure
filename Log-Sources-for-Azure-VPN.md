# Jarvis and kusto queries

[[_TOC_]]

## Disclaimer

**_The following is an alternative if, for example, our primary tool ASC is having issues. ASC should always be your go to tool for troubleshooting VPN cases._**

## Jarvis 

All tables are going to be located in either :
- BRKGWT
- BRKGWM

![image.png](/.attachments/image-75b6eb69-c68c-4c07-a796-a2c19b683e40.png)

## Kusto

Installing Kusto:

http://Aka.ms/kusto

Adding the connection to gain access to the tables:

![image.png](/.attachments/image-ec22a6ce-bc66-4c1e-9b63-9b478462da76.png)

Replace cluster with aznw

![image.png](/.attachments/image-7f6b2158-53aa-4e47-a3e9-c915c47caa3a.png)

The majority of the tables will be located under aznwmds:

![image.png](/.attachments/image-ec539af9-32bb-4572-b378-252179d257ef.png)

**NOTE**: Apart from using the VPN Dashboards, PG has stated that they are primarily using the following 3 tables. What is nice about these tables is that they have a column for "Message level". This column is useful for seeing if the message is informational / warning / or if there is a problem Error

## Tunnel insights events table
(only available for Route Based gateways)<br/><br/>
Useful for seeing why a s2s tunnel disconncted for example

Sample Jarvis query (in BRKGWT Namespace):

<https://jarvis-west.dc.ad.msft.net/566C1992>

Sample Kusto Query:

``` cs
cluster("hybridnetworking").database("aznwmds").GatewayTenantInsightEventTable
| where Tenant == "xxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-08-07 11:00) and PreciseTimeStamp <= datetime(2019-08-07 13:00)
| where TIMESTAMP >=ago(4d)
| project TIMESTAMP, RoleInstance, EventCategory, MessageLevel, Message
```

## Gateway insights events table
(only available for Route Based gateways)<br/><br/>
Useful for seeing what is going on a with the instances themselves

Sample Jarvis query (in BRKGWT Namespace):

<https://jarvis-west.dc.ad.msft.net/F440CD61>

Sample Kusto Query:

``` cs
cluster("hybridnetworking").database("aznwmds").TunnelInsightsEventTable
| where * == "xxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-08-07 11:00) and PreciseTimeStamp <= datetime(2019-08-07 13:00)
| where TIMESTAMP >=ago(8h)
| project TIMESTAMP, RoleInstance, GatewayBuildVersion, MessageLevel, Message
```

## VPN Gateway Packets Insights Events table
(only available for Route Based gateways)<br/><br/>
Useful for seeing if packets were dropped due to TS mismatches for example

Sample Jarvis Query (in BRKGWT Namespace):

<https://jarvis-west.dc.ad.msft.net/422EAEA7>

Sample Kusto Query:

``` cs
cluster("hybridnetworking").database("aznwmds").VpnGatewayPacketsInsightEventTable
| where Tenant contains "xxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-08-07 11:00) and PreciseTimeStamp <= datetime(2019-08-07 13:00)
| where TIMESTAMP >=ago(1d)
| project TIMESTAMP, RoleInstance, MessageLevel, Message
```
## Ike logs table
(only available for Route Based gateways)<br/><br/>
Useful for seeing:
- inits sent or received
- rekeys
- retransmits
- reasons for why tunnel has gone down
- also useful for point to site clients using ikev2
- failover between instances due to planned upgrades / or quorum loss via message RPC call

Sample Jarvis query (in BRKGWT Namespace):

https://jarvis-west.dc.ad.msft.net/4740506E

Sample Kusto Query

``` cs
//Ikelogs
cluster("hybridnetworking").database("aznwmds").IkeLogsTable 
| where DeploymentId  contains "xxxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-11-08 05:00) and TIMESTAMP  <= datetime(2019-11-08 23:00)
| where TIMESTAMP >=ago(1h)
//| where EventMessage contains "7089a55b-3c16-49d2-9cce-0ea838e613a5"
//| where RoleInstance == "GatewayTenantWorker_IN_0"
| project GatewayBuildVersion, TIMESTAMP, RoleInstance, EventMessage
```

## Ike Packet Logs table
(only available for Route Based gateways)<br/><br/>
Useful for seeing 
- IKE packets as well. 
- Can tell if Ikev1 or Ikev2 packets are sent. 
- DPD Packets

Jarvis Query (in BRKGWT Namespace): https://jarvis-west.dc.ad.msft.net/31D0810A  
Add this string to the Client Query to make the table easier to read:  
```
select PreciseTimeStamp, EventMessage
```  
Kusto Query: This table does not exist in Kusto  

Example of DPD packets:  
- In the first packet (1), we see that the Azure GW receives an IKE packet from the remote site because it says `RECV Network Packet`. The remote site was the INITIATOR (we see it in the packet `0x8(INITIATOR, REQUEST)` so the remote end is sending the DPD packets to Azure first.  
- In the next packet (2), the Azure GW responds to the DPD packet, we know it’s the response because it says `SEND Network Packet` and also the the `messid` is the same as the first packet so we know it is a response.  
- Packets 1 and 2 are a set of DPD packets. Packet 3 and 4 are a separate set of DPD packets.  
- Notice how there are 5 seconds between DPD packets by viewing the PreciseTimeStamp column. If there is no traffic passing over the tunnel, DPD packets will continue to be sent in 5 second intervals.  
- Also note that for packets 3 and 4, the `messid` is different than packets 1 and 2. This shows that these are two unique exchanges.  

![zcab-vpn-ikepacketlogstable](/.attachments/zcab-vpn-ikepacketlogstable.png)

## Packet Insight Event Table  
(only available for Route Based gateways)<br/><br/>
PacketInsightEventTable shows when the VNG drops packets before forwarding them into a tunnel. For example, the VNG receives a packet from a VM in the VNet. It checks the destination IP and the configured Traffic Selectos for the active tunnels. If the destination IP of the VM's packet is not in any of the tunnels' traffic selectors, the VNG would drop the packet and not forward it into the tunnel to on-prem. These drops are logged in this table. Check this table if you suspect that the VNG is not forwarding packets to on-prem.  

[Jarvis Logs Link](https://jarvis-west.dc.ad.msft.net/45ED7A8D)  

Kusto Query:  
```Kusto
cluster('hybridnetworking').database('aznwmds').VpnGatewayPacketsInsightEventTable
| where GatewayId == "VNG GW ID goes here"
//| where Tenant == "VNG DepID/TenantID goes here - remove the comment marks // to use this filter"
| where TIMESTAMP > ago(1h)
//| where TIMESTAMP >= datetime(2019-02-27 17:15:00) and TIMESTAMP < datetime(2019-02-27 17:50:00)
```

## Tunnel Events table
(only available for Route Based gateways)<br/><br/>
Useful for seeing:
- Tunnel change state
- Was it a planned failover?
- Tunnel change reason ie DPD or peer sent a delete message
- how the tunnel was negotiated ie IKE and IPSEC parameters as well as what traffic selectors were used

Jarvis Query (in BRKGWT Namespace):

https://jarvis-west.dc.ad.msft.net/DCA95A1B 

Kusto Query:

``` cs
// tunnel events table
cluster("hybridnetworking").database("aznwmds").TunnelEventsTable
| where * contains "xxxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-10-09 08:00) and TIMESTAMP <= datetime(2019-10-09 14:00)
| where TIMESTAMP >=ago(7d)
| project TIMESTAMP, RoleInstance, Message, DownTimeInMilliSeconds, IsPlannedFailover, TunnelName ,TunnelStateChangeReason, NegotiatedSAs
```

## Tenant Events Table
(only to be used for Policy Based gateways)<br/><br/>
Useful for seeing if a tunnel went from connected to connecting or vice versa

Jarvis Query (in BRKGWT Namespace):

https://jarvis-west.dc.ad.msft.net/221BB347

Kusto Query:

``` cs
//gateway tenent events table 
cluster("hybridnetworking").database("aznwmds").GatewayTenantEventsTable
| where * contains "xxxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-08-01 02:30) and PreciseTimeStamp <= datetime(2019-08-01 02:45)
| where TIMESTAMP >=ago(1d)
| project TIMESTAMP, RoleInstance, Message, GatewayBuildVersion
```

## Gateway tenant logs table

Useful for:
- Seeing why BGP is not coming up for vpn
- Quorum loss causing a gateway instance to go from primary to secondary

Jarvis Query (in BRKGWT Namespace)

https://jarvis-west.dc.ad.msft.net/DB3AF15B

``` cs 
Kusto Query:
//Gateway Tenant Logs Table 
cluster("hybridnetworking").database('aznwmds').GatewayTenantLogsTable 
| where DeploymentId  contains "xxxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-10-24 08:00) and TIMESTAMP  <= datetime(2019-10-24 09:00)
| where TIMESTAMP >=ago(5h)
| where Message contains "BGP" //and (Message contains "quorum" or Message contains "connectivity state changed")
| where Message !contains "purging"
//| where RoleInstance == "GatewayTenantWorker_IN_0"
| project PreciseTimeStamp , RoleInstance , Message 
```

## Point to Site logs table
(only available for Route Based gateways)<br/><br/>
Useful for:
- Tracking point to site connections ie request received / duration / packets sent / disconnects
- Used for Ikev2 and Open VPN clients
- Ikev2 requests can be correlated with ike logs table

Jarvis Query (in BRKGWT Namespace):

https://jarvis-west.dc.ad.msft.net/1BE0CF12

Kusto Query:

``` cs
//point to site
cluster("hybridnetworking").database("aznwmds").P2SLogsTable
| where * contains "xxxxxxxxxxxxxxxxxxxxxxx"
//| where PreciseTimeStamp >= datetime(2019-10-15 12:00) and TIMESTAMP  <= datetime(2019-10-15 13:00)
| where EventMessage contains "7089a55b-3c16-49d2-9cce-0ea838e613a5"
| where TIMESTAMP >=ago(1d)
| project TIMESTAMP, RoleInstance, EventMessage 
```

## Async Worker Logs Table

Useful for seeing:
- Why a VPN gateway did not create
- Why a VPN gateway did not update
- Why a VPN gateway did not delete 

Jarvis query (in BRKGWM Namespace):

https://jarvis-west.dc.ad.msft.net/949B789F 

Sample Kusto Query:

``` cs
//asyncworkerlogstable
cluster("hybridnetworking").database("aznwmds").AsyncWorkerLogsTable
| where CustomerSubscriptionId  == "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
| where PreciseTimeStamp >=datetime(2019-11-08 11:00) and PreciseTimeStamp <=datetime(2019-11-08 14:00)
| where * contains "4c16c4eb-de1d-492c-b439-e07cfd206cc2"
| where Message contains "exception"
| project TIMESTAMP, OperationName, OperationId, CorrelationRequestId,ClientOperationId, Message
```

