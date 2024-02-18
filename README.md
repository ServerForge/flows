# ServerForge Flow Monitoring


Simple sFlow/Netflow monitoring setup using Docker,pmacct,ClickHouse, and Kafka.
This is an early work in progress.

## Requirements

 1. BGP router with full tables.
 2. VM/Server with 4 Cores, 8G of ram, 128G of disk.
 3. sFlow/Netflow capable router.
 4. An Existing Grafana server.
 
## Instructions

### Configure sFlow/Netflow Hostnames/Directions
Add entries to the top of the pretag.map file in the following format, One for each router. The IP field should be the AgentID for sFlow, or the IP/IP6 for Netflow.  jeq should be "dir_tag" if the specified router is sampling your wan port. If your router is sampling an internal port it should be set to "dir_tag_rev".

    set_label=Router1 ip=192.168.1.1 jeq=<dir_tag|dir_tag_rev>

### Configure Network List
Configure your networks in the networks.lst "ASN,PREFIX" one per line.

    123,192.168.1.0/24
    123,fc00::/7

### Configure Peering Agent Map.
Edit the peering_agent.map file and add the Router ID of your bgp peer that will provide BGP tables to the local route collector. If this is not set to the RouterID it will not map the prefixes to the ASNs from this routers BGP session.

    bgp_ip=192.168.1.1     ip=0.0.0.0/0

### Edit Data Retention Length (Optional)
Edit the docker-compose.yml and adjust the policy value for the flow-consumer service. This value is used for the retention length for the ClickHouse database and will only be applied on first start. Default 365 DAYS.

### Setup iBGP to Route Collector 
Setup and iBGP session from your BGP router with full tables to the IP of your server. You do not need to configure anything for this on the collector side, it will simply establish a session using the ASN of any router trying to establish a session with it.
Note: It will take a minute or so for the BGP session to come up once the stack is started.

### Configure Your Firewall
Configure your firewall to restrict access to the 2055/6343 Netflow/sFlow ports to only allow your routers and only allow your grafana instance to the ClickHouse Database port 9000. 

### Configure your sFlow/Netflow devices
Configure your network devices to send sflow data to the IP of your server using a sample rate of 1000.

### Start The Stack
    docker-compose up -d

### Configure Grafana
If you do not already have the ClickHouse plugin installed in your grafana you need to install it.
It can be installed under the plugins section.

Configure a clickhouse datasource using the IP of your server, port 9000, native, username of default, and password empty.
(Future releases will support authenticated DB)

On your dashboard page select the "New" dropdown and click "Import". Use grafana ID 20502 and select your new ClickHouse DB as the data source. 

You should now be able to view the dashboard, and should see metrics being displayed. You may see some statistics for AS0 whenever it has recently started due to the iBGP session being down.

## Known Issues:
If you use add-path-tx-all on your BGP router and send several million routes to the collector it will not be able to search them quick enough to populate AS data for flows. 
