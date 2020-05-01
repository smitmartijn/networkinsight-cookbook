{id: ch-architecture}
# Architecture
*...Or how things are put together.*

Network Insight consists of two components;

1) the Platform,
2) the Collector (previously known as the Proxy).

In simple terms; the Platform is where you connect your browser to use the product and retrieve information and view the pretty graphs. It also The Collector is what connects to the data sources and relays information to the Platform for processing.

Here's how the architecture looks and how communication flows between the different components:

{caption: "Platform Architecture Diagram"}
![](images/image54.png)

There are a lot of things happening in both layers, which I'll go further into in the following chapters.

## Platform

When you log into Network Insight via your browser and look at all the marvelous data that has been collected, you're looking at the interface of the Platform. It is the doorway into Network Insight and houses the data that is collected from your environment. Data collection happens at the Collector layer, but the Platform keeps the data persistently, where the Collector only has temporary data.

There are a few options to vertically scale, meaning there's a t-shirt size to the resources that a single appliance will consume. At the time of this writing, you've got the option between **medium**, **large** or **extra-large**. You can also scale out the Platform layer horizontally to support storing more data, but more on that in chapter [Scalability and Availability (Clustering)](#ch-clustering). In any case, always consult the most recent [documentation](https://docs.vmware.com/en/VMware-vRealize-Network-Insight/index.html) on which t-shirt size you need to pick and to determine whether you need to build a cluster to provide support for the size of the environment it's going to monitor.

The Platform layer can have multiple Collectors from which it receives data from. Communication between the Platform and Collector only happens one-way, from the Collector to the Platform. This means there does not have to be direct connectivity from the Platform towards the Collectors, which can be put behind a NAT boundary. As long as the Collector can reach the Platform, you're good to go.

Inside the Platform, there are a few different layers which serve different purposes. It is a brick and loosely-coupled architecture, which means multiple Platforms can be placed together and the service layers will stretch between the different bricks.

Here is a representation of the service layers that live inside a Platform:

{caption: "Platform VM Internal Architecture"}
![](images/image55.png)

### Presentation Service Layer

The User-Interface (UI), REST API and Search Engine services are responsible for the presentation of data to the user, whether it via a browser or an API output. The function of the UI is pretty straightforward; present an interface to any browser that connects. In turn, the UI uses the API to retrieve data and configure any setting you're changing.

The REST API is split up into two sections: a private API and a public API. More on the public API can be found in the chapter on [Automating Network Insight](#ch-automating) -- but the primary goal of the public API is to let you, or your automation and/or orchestration systems talk to Network Insight and retrieve data from it.

All UI interactions with data is done via the private API. As the name suggests, this part should be not used by any automation work that you would like to build. The format and output are likely to change throughout different versions.

{caption: "Private API in action"}
![](images/image56.png)

### Search Engine

While the search engine is worth a whole separate chapter, in this context is it worth explaining that the search engine drives all interactions. Anything that you do in the interface is a search command. This is also reflected in the search bar:

{caption: "Searching your data center"}
![](images/image57.png)

Talking to a backend powered by Elastic Search, it takes search queries in technical natural language. It searches through configuration data, events, performance stats and can do so in a time machine to get results for a specific time frame. Unlike the open source database backends, the search engine is a service entirely built by the Network Insight team.

More details on the search engine (and how to use it) in chapter [Using the Search Engine](#ch-search) Engine.

### Data Service Layer

Everything about your environment is stored inside the data service layer. Configuration data, events and performance stats are all stored as efficient as possible. This means that there are multiple database systems present.

Searching firstly goes against an Elastic Search database which holds indexed configuration data that points to a PostgreSQL and HBASE database where metrics and configuration is held in a timestamped format. Using Elastic Search and having it index the other databases, speeds up any search operation and improves the user experience via the web interface.

Configuration data is stored in PostgreSQL and each configuration is timestamped with a time & date when this configuration was detected. Because of these timestamps, this is also where changes in configuration are held. Virtual Machines, Networks, Firewall rules are examples of the data available in this PostgreSQL database.

The results from the Elastic Search database also contain index pointers that link to metrics located in another database; HBASE. For example, a switchport object would have a link to the HBASE database where the bandwidth usage metrics are stored. That combination makes that the switchport configuration (port type, which VLANs, etc.) is presented along with its bandwidth usage graph.

## Collector

The Collector is the artist formerly known as the Proxy. If you come across any mentions of a proxy appliance in the documentation; the Proxy was renamed to Collector. Now with that out of the way...

All data comes into Network Insight via the Collector. Whether it's NetFlow records being sent to it, or it connecting to data sources like vSphere, physical switches, firewalls, routers, load balancers or compute systems, it is the entry point for all environmental data.

Collectors can be scaled out by using multiple appliances for your environment and they can be scaled up in resources by choosing a different deployment model (**medium**, **large** or **extra-large** at the time of this writing). Just as with the Platform, always consult the most recent [documentation](https://docs.vmware.com/en/VMware-vRealize-Network-Insight/index.html) to get the latest requirements for your environment.

You can strategically place the Collectors as well. It does a couple of things to the incoming data to compress it (more on that later) and allow it to be sent to the Platform more efficiently. Traffic between the Data Sources and the Collectors is heavier than between the Collectors and Platform appliances, which makes it more efficient to place Collectors in remote locations.

{caption: "Platform & Collector relationships"}
![](images/image58.png)

Networking management is usually as locked down as possible to protect the management interfaces of your network devices. It makes sense to provide access to Network Insight for people that do not have any business connecting directly to networking devices, so you would want to segment those. In this instance, you would put a Collector inside the networking management segment and only permit it to connect to the Platform. The Collector would connect directly to the Data Sources inside the secure segment and you don't have to allow incoming communication from outside the secure segment.

I> You can have multiple Collectors reporting up to a single Platform. Data Sources cannot be shared and have a one-on-one mapping to a Collector.

### Collector Services

Inside a Collector, there are several services running to make sure Data Sources are being polled for configuration & metrics and flow records can be ingested.

The polling agents and Flow Processor are proprietary code, but there are several open-source services running:

- **PostgreSQL** is used to store all collected data, before sending it on to the Platform.

- **NGINX** to act as a transport to the Platform and receive incoming webhooks (just vRealize Log Insight for now).

- **Nfcapd** and **Sfcapd** are used to receive NetFlow and sFlow (respectively) records and store them in native capd data files.
  - These are slightly modified to support IPFIX from the VDS and ignore information that Network Insight does not need, to improve performance.

{caption: "Collector VM internal architecture", width: "80%"}
![](images/image59.png)

{id: ch-flow-processor}
### Flow Processor

The Flow Processor is a custom service that runs on the Collector and it, surprisingly, processors incoming network flows from the NetFlow and sFlow sources and translates them into a data format Network Insight can use.

On the Collector, there are two services running for the incoming NetFlow (including IPFIX) and/or sFlow: nfcapd and sfcapd. These are open-source services that collect flow data in a standardized way and generate data files that are stored on the local filesystem of the Collector. These data files can be read manually with the nfdump utility, if needed.

By now, you should have configured a data source that's sending flow data to a Collector, whether it be vCenter, NSX or a physical network device; here's what happens with those flows:

{caption: "Collector NetFlow processing"}
![](images/image60.png)

The service nfcapd receives the NetFlow records from the reporting devices and saves them to memory. **Every minute**, it flushes the memory contents into nfcapd data files. This is the same for sFlow records received by sfcapd, except that the output is, surprise, sfcapd data files.

**Every 5 minutes**, the Flow Processor runs. It reads in the data files generated by both nfcapd and sfcapd, using bookmarks to remember which flows it already has seen. This is where the flow records are reduced in size by ignoring unused fields and going from a 5-tuple (destination IP, source IP, destination port, source port, protocol) record to a 4-tuple record and neglecting the source port field.

Attached to each flow, the processor stores a reporter value and any NSX firewall rule information that's available. Flows coming from the NSX firewall are enhanced and have the rule-id and action (allow or deny) embedded in the IPFIX record.

The reporter IP address is stored so that you can relate to where the flows come from, which would typically be the IP of the vCenter (and ESXi hosts) or physical network device.

I> By neglecting for source port, Network Insight typically assumes that you'll be using a stateful firewall when it generates the recommended firewall rules. In this day and age that should be a safe assumption but be aware of this.

The metrics of the flow records (bytes sent, number of packets and session count) are also split from the flow itself and put into a new record, specifically used for showing a bandwidth graph with each flow.

During this exercise, the flows are also deduplicated as only the metrics of multiple flows of the same 4-tuple record are kept. The bytes sent, number of packets and session counts are simply added up to form a single flow record with the correct metrics. This

**Every 10 minutes**, the collector service will read out the waiting queue of processed flow records and sends these to the Platform. After that, the Collector is done and marks the sent flows to be processed and eventually archived (it keeps records for 2 days, after which they are rotated away).

These timers mean that it could be possible that you'll see flows turn up in the interface at a maximum delay of 16 minutes. This could be longer if the Collector is overloaded, but you'll see a warning being raised, if that happens.

Once a flow gets to the Platform, it gets enriched with data from the rest of the environment that flow relates to. For a virtual machine, the vNIC is looked up first by using the source and destination IP. It uses both source and destination IPs to pinpoint the right vNIC, in case of overlapping IP ranges inside the environment. When it has the vNIC and virtual machine that the flow belongs to, it retrieves a bit more meta data (like cluster, resource pool, etc) and stores it with the flow record. This is all to speed up the search process.

This enrichment process also checks whether there have been any changes to this meta data (i.e. the virtual machine has vMotioned to another host) and updates it, if needed.

{id: ch-bandwidth-requirements-for-flows}
#### Bandwidth Requirements for Flows

One of the most frequently asked questions is "how much bandwidth is this NetFlow thing going to use?" -- Which is a good question! It can help you with planning out the Network Insight topology and determining placing of the Collectors. It is, however, a tough question.

Both NetFlow and sFlow are flow based, which maps to a 5-tuple matrix. Source, destination IPs and port numbers and the protocol will be a unique flow. Your environment might have bandwidth heavy services (like a file share, a massive database) that get used by just a few servers. For example, if there's a single server talking to a single file share, reading and writing multiple terabytes to that file server; that is a single flow. If you have 100 servers that are all talking to 100 application servers; you would have 10.000 flows.

It's not a simple calculation and it'll vary per application landscape. However, we can do some rough calculations. Let's start with the bandwidth a specific amount of flows could generate.

A NetFlow version 5 packet is 48 bytes and each outgoing packet holds an average of 20 flow records, with 24 bytes of overhead per packet. That is 984 bytes per second for 20 flow records.

I> For 20.000 flow records per second, we can use the following calculation for the bandwidth required: (48 \* 20) + 24 = 984 bytes for a packet with 20 records. Multiply that by 1.000 for 20.000 flow records per second, and we get 984.000 bytes per second and 7.872.000 bits, or 7.8Mbit per second.

However, when the question is; "how many flows will a specific amount of application bandwidth produce?" -- it becomes a bit more complex. On average a flow is around 200Kbit per second[^6], based on monitoring networks of the likes of Google and Facebook (data on enterprise networks is understandably lacking). This means monitoring 1Gbit of traffic would produce around 5.242 flows per second, meaning around 2Mbit per second of flow records would be sent from that original 1Gbit traffic.

{aside, class: information}
To turn 1Gbit of traffic into a number that represents the bandwidth generated by flows, we can use the following calculation:

1Gbit = 1024 \* 1024 = 1.048.576 Kbit / 200Kbit for 1 flow = 5.242 flows / 20 flow records per packet = 262 packets with flow records = 257.906 bytes = 2.063.251 bits = 2.06Mbit per second.

In other words; generated flow traffic is about 0.2% of the actual traffic.
{/aside}

Now, this data is based on user and application traffic. When you dive into the data center, there are a lot of variants. For instance, a single host talking to a NAS and sending multiple terabytes to that NAS, is likely to only produce a single flow, which will only be updated with the amount of bandwidth that flow is consuming. High size transfers in a single session will generate less flows than low size transfers by 1000s of users.

Keep in mind that these calculations are based on averages and could wildly differ from what you'll see in your own network. The best answer is always; "let's enable network flow collection and see how much traffic it's actually generating".

So yeah, that's why the default answer for this question is; it depends.

{id: ch-aws-azure-flow-processing}
#### AWS & Azure Flow Processing

While NetFlow and sFlow are protocols that push data to the Collector, the flow records in AWS & Azure is a different story. When enabled, there is a flow log (per AWS VPC and per Azure Network Security Group) that keeps all records of network flow inside and outside the public cloud environment. These flow logs are stored in either AWS CloudWatch, an AWS S3 storage bucket, or Azure storage account, and can be viewed via the AWS Console, the Azure Portal, or via their API.

Network Insight uses the applicable public cloud API to collect these logs periodically from AWS and Azure. It uses the same polling interval as the regular flow processor uses: every 10 minutes. The Collector responsible for collecting the rest of the inventory information (EC2 instances, Azure VMs, networks, security policies, etc.) is also the one that collects the flow logs. It is architected this way, so that the Collector can make direct links between the flow records and the EC2 instances or Azure VMs to which those flow records belong.

By default, the flow records that come from AWS & Azure are not that user friendly. For instance, they contain instance IDs and not the instance names. When the collector downloads the flow logs, it'll link the right instances (or outside sources like VMs, physical servers that run on-premises) and add more context details (tags, inter-VPC traffic, everything that is relatable, it will relate) to the flows, before sending it to the platform.

When Network Insight retrieves the flow logs, it will read the logs every **10 minutes**, parse them and correlate them to the related objects, like the EC2 instance or Azure VM, network interface, AWS VPC, Azure VNet, etc., etc. The collector appliance remembers the already retrieves logs, by setting a bookmark on the last log line it reads. That bookmark is then used the next time it ingests the flow logs and it will start parsing the logs from the bookmark.

Keep in mind that logging flows in both AWS and Azure, will generate data per flow. It'll cause storage usage and some resource usage; it will cost money to have it enabled. It depends on the number of flows going through the cloud, but it's typically not that much. Both AWS and Azure charge per GB of logs generated (around 50 cents per GB per month), and these flow logs are stored in text format.

For reference, in a demo lab I manage, there are **3.300** flows stored and it is taking up **4.2MB** of storage. It's not going to be the biggest cost on your cloud bill, but it isn't free. ;-)

### Connecting to Data Sources

Data sources are the virtual and physical devices that are added to Network Insight, assigned to a Collector, which then starts pulling data from these devices. It does this by connecting to the device based on the best suited protocol for the job. Which means that the Collector will connect over SSH to grab configuration data, it uses SNMP to grab the metric data (for the bandwidth graphs), or it uses an API if the device has one.

This means the Collector has to have management access to the devices that are added to it. Be sure to open up any and all firewalls for connections from the Collector to these devices over ports 22 (TCP, SSH), 161 (UDP, SNMP), and 80 and 443 (TCP, HTTP(s)) so that it can actually connect. Connectivity to these types of devices (i.e. network switches) are mostly closed, and rightly so. I've seen to many implementations where it people were side tracked because there was no connectivity allowed between the Collector and network devices. With the exception of incoming network flows via NetFlow or sFlow, all connections are outbound from the Collector.

Polling happens on an interval between 5 and 15 minutes, depending on the type of data source. Most polling happens every 10 minutes, with the exception of Brocade VDX (every 15 minutes) and VMware NSX (every 3 minutes for security events, 5 minutes for regular events and 10 minutes for other configuration items). Every other data source will be polled every 10 minutes.

SNMP polling happens every 5 minutes on every data source. Of course, these intervals can change when a new Network Insight version is released; always double check in the documentation for the exact intervals. This is just to give you an idea.

I> Polling happens independently per data source and on an interval timer. There is no persistent connection being left open to the data sources; no permanent SSH connection. It will reconnect each time.

There are a few built-in protection systems to protect the data sources from being overwhelmed by the polling and stressing them out. This protects them if they are already stressed for some reason, so the Collector does not push them over the edge and impact the operations of the device.

One of these protections is that the polling does not actually happen every 10 minutes. When polling concludes, a timer is started that counts down to 10 minutes and then another poll is started. Meaning that if the poll itself takes longer because the device is stressed, there is always a 10-minute breathing room and polls don't start to overlap.

Another protection layer goes into the specific data source. If something is known for not handling commands in rapid succession or if it does not handle bulk data transfer well, there is a throttling mechanism in place to make sure the Collector does not cross the threshold and does not stress the device out. For example, the Manager for NSX for vSphere is known to not to handle rapid fire of API calls very well (sorry, not sorry); so, the Collector pauses a bit between each API call. The same is true for Cisco Nexus switches, so, there's also a pause between commands that are executed on each Nexus switch.

Generally, all that Network Insight needs is a read-only user on the data sources. It only wants to read in the configuration and operational data by executing show commands, SNMP reading for metrics, or retrieve this data via an API. However, there is one notable exception where running a show command is proxied via another command that does need more than read-only.

This exception is the CheckPoint firewall integration. To get the gateway device interfaces and IP routes, Network Insight needs to run the **run-script** API call; which requires read-write permissions.

Use the reference document called **vRealize Network Insight Data Source Integration Reference Guide** [^7] to get a list of all commands, SNMP OIDs and API calls that are executed by Network Insight. You can use that list to create a customized user that can only execute these commands and speed up the implementation by quickly getting the approval of your security department. There will be no discussion about what impact Network Insight could have on your devices, as it is crystal clear what it's doing.

[^7]: This guide is currently not available publicly; ask your VMware representative for it.

### Connecting to the Platform

Previously, I mentioned that the communication between the Platform and Collectors are one-way. The Collectors initiate the connection and pushes the data across, unidirectionally. Because of this architecture, you can place the Collectors behind a firewall and/or NAT configuration, and it'll work. This resolves some security concerns around the Collector that needs connectivity to the physical network devices and management components like vCenter, the NSX Manager, etc. Place the Collector in the network management zone and only allow it to communicate to outside the network management zone over port 443 towards the Platform appliance(s), and your security architect should be happy!

SSL encryption is being used to send data from the Collector to the Platform. This is done using certificates that are exchanged during the setup of the Collector, when it first connects to the Platform. The Platform saves the presented Collector certificate and then proceeds to verify every future connection and make sure it's verified and secure. This makes man-in-the-middle attacks on the connection much harder (nothing is impossible to hack, except for a stone brick).

I> The certificate that is used for a specific Collector, is also used to encrypt the data source credentials. This is why you have to re-enter the credentials when moving a data source between Collectors.

#### Data Compression

There's a compression procedure when it comes to ingesting data source data and the data that's being sent to the Platform. The incoming network flows are trimmed down, as Network Insight doesn't need all the fields provided by standard NetFlow, IPFIX or sFlow records. Network Insight only cares about 4-tuple network flow records (source IP, destination IP, protocol and destination port), so it trims the standard 5-tuple flow records by removing the source port.

Note: If VMware NSX is integrated and sending network flow data, the saved flow records also include the firewall rule ID that the traffic has passed through or been blocked by. This is so Network Insight can correlate the network flow directly to a firewall rule.

Commands that are executed on physical network devices and called upon APIs also might contain unnecessary output which it will ignore and not pass on to the Platform.

The Collector also keeps a local table of state for the objects (VMs, networks, flows, etc.) that it tracks and only if the state of that object changes, it sends an update to the Platform. If you have a very static environment and nothing is changed (i.e. no new VMs, no vMotions), the Collector will still continuously poll the data sources for information, but it is possible that the Collector is not pushing any changes towards the Platform. This saves a bunch of bandwidth between the Collector and Platform and also saves the Platform from having to bother with unnecessary updates.

However, to make sure that the Collector keeps the Platform informed, a keep alive with the latest known status of the data source is sent every 4 hours.

#### Offline Caching

If the Collector can't reach the Platform for any reason (like connectivity, or the Platform is simply down), it will hold an offline cache of the data that it's collecting. It will continue polling data sources and processing data from the incoming network flows and will store the incoming data in a cache directory. When the Platform reports back to duty, the Collector will empty out its cache and the history on the Platform is amended; like it was never gone in the first place.

To determine how long this offline cache will last (so, the amount of time a Platform can be missing in action), you first need to know that there are 2 different caches; 1 for network flows and 1 for polled data (everything else).

I'll go through both offline caches below and try to give you an idea of how long the Platform can stay offline. In both cases, it's good to know that the data stored in these caches has a maximum amount of storage and once that maximum has been reached, the Collector will start pruning the older data and will keep the newer data coming in.

##### Network Flow Cache

The network flow cache has a maximum size of **15GB**. If you recall, a packet with 20 NetFlow records is around **984** bytes. To fill up the caching, it would take around **335.544.320** flow records. Here's how I got there:

{aside, class: information}
To fill up the network flow cache of 15GB:

15GB = 15 \* 1024 \* 1024 \* 1024 = 16.106.127.360 bytes / 960 bytes for 20 flows = 16.777.216 \* 20 flows = 335.544.320 total flows.
{/aside}

It depends on how much flows the Collector is receiving, but the offline cache for flows can last a pretty long time! Let's say it's receiving **5.000** flows per second (roughly 1Gbit p/s of real traffic), that **15G**B will last for about **18,5** hours.

Each Collector has this network flow cache directory, so do these calculations based on the usage of the individual collector; not your entire environment.

You can find this cache directory here: **/var/flows/vds/nfcapd** -- This directory stores raw nfcapd files, which could be read using the nfdump utility.

##### Polled Data Cache

Now for 'everything else' -- which is comprised out of metrics, inventory, events, and configuration data that's being pulled from the data sources the collector has. This cache has a maximum size of **10GB** and is stored in **/var/BLOB\_STORE** on the Collector. The data that is stored here, is in Self-Describing-Message format (SDM), which is proprietary to Network Insight and you can't read it to see what's in them. They're essentially used as update messages from the Collector to the Platform to inform the Platform of all metrics and changes that occur.

It's a little harder to predict how long this cache can stay offline, though.

This all depends on the churn of the environment it's monitoring; how many events (i.e. vMotions) there are, how many VM actions (new VMs, removals, virtual hardware updates, etc.) happen, how many configuration updates your physical network devices have, how many network topology updates (including routing updates for new IP subnets). That's just events and configuration updates; it also depends on how many objects (VMs, switch ports) Network Insight is pulling metrics from. You get the point; it depends.

However, to give you an idea of the storage needed for a period of time, consider the following environment:

3 vCenters with 3 NSX for vSphere Managers, 9 physical network switches, 400 VMs, 500 switch ports for metrics. There's a relatively low VM churn of 10 VMs deployed each day and 10 removed VMs and about 20 VM updates happen. Every VM is injected into the network topology by announcing an IP prefix of /32 for it, so that routing updates happening, each time a VM gets deployed or removed.

Described above is a small business environment with a relative medium amount of churn. The experiment we did was to turn off the Platform for 12 hours.

In those **12 hours**, there was a buildup of **400MB** of SDMs. Meaning this environment could handle a Platform outage of **307 hours** (or 12 days). That should be enough time to get the Platform back up, right?! ;-)

You could use this experiment and get a (very) rough estimate on how long your own Platform could be offline, before data pruning starts to happen. Let's focus on the VMs and let's say you've got the maximum of 10.000 VMs that a Collector is monitoring. In that case, we take the 400MB of those 400 VMs in 12 hours and multiple that by 25 (because 10.000 / 400 = 25), and we get **9.8GB** in **12 hours** for offline caching on **10.000 VMs**.

Again, this is a very rough estimate. The point of this entire chapter is that you should not worry about data loss when you're performing Platform upgrades or when the Platform is down for other reasons. You'd have plenty of time finishing up the upgrade or even restore a full backup, if the Platform goes belly up completely.

{id: ch-hosted-architecture}
## Hosted Architecture (SaaS)

If you're using vRealize Network Insight Cloud, the appliances used are exactly the same as the ones you would use on-premises. There are 2 notable differences:

- Authentication is handled by single-sign-on coupled with the VMware Cloud Services Portal,
- There are a few user-interface changes, for example the styling colors have been changed to match the other VMware Cloud Services and a couple of settings are not available (LDAP, User Management & Mail Server)

Authentication is handled by the VMware Cloud Services Portal, which explains the missing authentication settings. VMware has its own mail servers in place for the service, so you don't have to configure a mail server.

Think of it like this; with vRealize Network Insight Cloud, VMware hosts and maintains the Platform, which means you only have to deploy the Collectors in your own environment. Data flow is also the same, which means the Collector talks unidirectional to the Platform. In this case the Platform is hosted on the internet, which means your Collectors need to have internet access for this to work.

I> The Platform appliance of Network Insight is multi-tenant capable out of the box. It currently takes a lot of effort (and it's not user-friendly and not supported) to get multiple tenants activated. I've tried and broke a few Platforms. VMware is using this multi-tenancy capability in the Cloud variant. I'm holding out hope that multi-tenancy will be activated in the on-premises variant as well.

{caption: "Architecture for vRealize Network Insight Cloud"}
![](images/image61.png)

I> When using vRealize Network Insight Cloud, you only have to deploy the Collector in your environment. It requires connectivity to the Platform, which means internet connectivity is required for the Collector.


{id: ch-clustering}
## Scalability and Availability (Clustering)
TODO!
One Platform and Collector pair can collect a large amount of VM data and network flows; currently 10.000 VMs and 10.000.000 network flows total. However, if you need to go over these maximums, it is possible to create a cluster of Platform appliances to support the bigger environments.

Note that I’ll be calling the appliances a so-called ‘brick’ – this is a term for a VM that’s a part of the Network Insight setup. Just in a house, the Network Insight setup can be built from multiple bricks of multiple sizes. These bricks can be Platforms or Collectors.

Important to know is that a medium brick deployment can also be scaled vertically to become a large brick deployment. All you have to do is to shut the VM down, adjust the virtual hardware to match the large brick deployment specifications and tadaaa, it’s a large brick!

<insert current maximum table here>

If a single large brick Platform isn’t enough to sustain the amount of VMs or network flows you need to monitor, a horizontal scaling exercise can be done. Add multiple Platform bricks into a cluster and the number of VMs and network flows scale up.

Currently, you can create a cluster of a maximum of 10 Platform bricks. Meaning you can monitor up to 100.000 VMs (10.000 per Platform * 10 Platforms) and XXX network flows (xxx per Platform * 10 Platforms) with the maximum size cluster.

The Collectors cannot be clustered at this time, so the maximum amount of VMs and network flows coming from a single data source has a limit of the large Collector: 10.000 VMs and 10.000.000 network flows.

Creating a cluster is pretty straight forward; first, you deploy the first Platform (which will be referred to as Platform1), set up its networking, license and then go to the Install & Something page in order to create a cluster.
