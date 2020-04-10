# Network Insight into Public Clouds

The concept of a cloud is nothing new. Organizations have been hosting their applications on other people's computers for ages. We were running vCloud Director 1.5 at a data center service provider that was renting out VMs to customers (Infrastructure as a Service), back in 2011. Before using vSphere VMs and a portal like vCloud Director, we were using FreeBSD jails and Virtuozzo (based on CentOS) to deliver VPSs (Virtual Private Server). These services were mostly contained to Infrastructure as a Service (IaaS) and added manual services (monitoring, backups, pro-active maintenance, etc.). Fun times!

While it's not new, the speed at which the cloud provider industry has evolved in the last few years, is pretty ridiculous. Going from providing simple infrastructure as a service to database, storage, machine learning, blockchain, IoT, and all other current buzzwords, all as a service. Certain business cases of running entire companies in the public clouds are a no-brainer (some cases are not, so always do your homework) and it's never been easier to start a new project and getting it off the ground. Get your new product to your customers in days, if not hours. All you need is a credit card.

That's where the problem lies, as well. Because it's so easy to get up and running, developers and/or business groups start to run their products in the cloud, without informing the IT infrastructure department where the responsibility to keep the company's digital footprint up and running. Even if IT gets involved and there's a proper master account set up with linked/sub accounts for the business groups and gets a proper view of what's happening, there's another problem rearing its head.

Public clouds are starting to specialize in specific services. While every single one will keep the basis of IaaS and complementary things as databases as a service, specific public clouds are better with certain services. For example, things like machine learning, artificial intelligence, ARM CPUs, GPUs; certain clouds are just better than other clouds. Amazon even has a satellite ground station as a service, these days. In any case, the probability that your organization will be adopting multiple public cloud services, while keeping an on-premises data center presence, is high. Meaning you will be tasked with managing and securing multiple clouds eventually, if that's not the case already.

Do to this successfully, while not growing grey hair along the way, you need a tool that can span multiple clouds and present all different clouds in a universal way. Network Insight will inventory all networking and security aspects of the on-premises data center, public clouds and branch locations and present its feature set over all these different locations; the same way.

That last bit sounds very salesy, but basically it means this: network traffic flows are correlated between all these locations and the security planner is available to make sure all workloads are secured across all locations, network traffic analytics can detect anomalies no matter where the workloads are being hosted, application discovery spans private, hybrid, and public clouds, there is a global inventory of all network and security objects (virtual networks, security policies, gateways, VPNs, those sort of things) which can be used to troubleshoot issues and monitor the network, all in the same interface, providing the same experience.

## Amazon Web Services (AWS)

With 47.8%[^4] of the public cloud market in its pocket, Amazon Web Services is a clear leader. It pays to be the first, but it also helps that AWS has over 165 (and counting) different services. Ranging from infrastructure as-a-service, to database, analytics, application services, deployment management, mobile, developer tools and internet-of-things enablement services.

You're most likely most familiar with their Elastic Computing Cloud (EC2), the Simple Storage Service (S3), and the Relational Database Service (RDS). These services get used to most by organizations that are looking to the public cloud for workload placement, i.e. get some cheaper infrastructure. I'm not going to go into whether moving existing workloads (instead of refactoring them to take advantage of native public cloud services), is a good or bad idea, but how Network Insight helps you in managing the pain if your organization decides to go.

On whether it's a good idea or not, just please make sure the homework and cost projections are done before you go.

### AWS Networking Tools

There are a few native network monitoring and troubleshooting tools inside AWS:

-   **Mirror sessions**; essentially a port mirror that can be placed on an Elastic Network Interface (ENI). You do some advanced things here, like creating filters to only mirror specific traffic based on source, destination, protocol, and ports. The traffic is delivered to another ENI, meaning you have to run an EC2 instance where you then catch the traffic using something like Wireshark or tcpdump. Another cool fact about these mirror sessions, is that each session is encapsulated into a separate VXLAN network, meaning you can separate out the different mirror sessions by looking at the VXLAN ID on the target ENI.

-   **Flow Logs**: the ability to log all IP traffic that is incoming and outgoing to network interfaces in a specific VPC. Think hit count logs, but only for all IP traffic. You can configure it only for the entire VPC and determine whether you want to only log accepted traffic, or also log rejected traffic by the security groups that are in place. These logs will be sent either to CloudWatch or an S3 bucket.

-   **Status Checks**: checks whether services are running, reachable, and performing. This is more on the application level, where the network is used to do these checks.

-   **CloudWatch Logs**: you can send all kinds of logs and metrics to CloudWatch. It allows you to access, and correlate all this data, between different AWS services. It is comparable to something like Splunk or vRealize Log Insight.

These tools are point solutions; solving 1 issue per tool. The mirror sessions are great for on demand troubleshooting but needs an EC2 instance to direct the traffic to (which you need to set up yourself). The Flow Logs can also serve their purpose but that has its scalability limits. It's literally structured logs with the following format:

No friendly names of the account name or EC2 instance, just the IDs. You would need to find the EC2 instance, look at the network interface and grab its ID, before this would make sense.

Using Network Insight for the same job, is a lot easier & quicker. As mentioned before, it will correlate everything together -- meaning it will look up the network interface IDs, attach them to the EC2 instance and allow you to look up all flows from a specific (or a group) EC2 instance.

It won't only collect the flow logs, but also all inventory (EC2, VPC, and everything attached to those) to do inventory tracking over time. The timeline functionality will be available for all AWS compute and network inventory. It will generate events for certain best practices (like security rule maximums against security groups), and you can create user defined events to get alerts when something changes in AWS.

And it will do this in the same interface where you have your on-premises workloads, making it easy to get the global overview. Because unless you're a unicorn (congrats!), you'll also have on-premises networks to manage. At least, you'll have something more than just AWS.

### Adding AWS to Network Insight

There are two ways to add AWS to Network Insight. Using a standard account which houses network & compute instances directly or using a master account that holds a number of different standard accounts, which in turn hold the network & compute instances.

When you have a larger organization that uses AWS, the chance is that separate accounts are created to hold different departments, projects, or persons. This is to separate out management of the resources or having the ability to figure out which department or project is costing the most money. In short; a master account is, as the name suggests, a master of all linked accounts and has visibility into all of them. A linked account has only a small portion of the organization's resources.

If you do have a master account you can add that to Network Insight, and it will discover all linked accounts under it (existing and new) and start collecting data from the network & compute resources that are under those linked accounts.

{caption: "AWS Master account link diagram"}
![](images/image40.tiff)

If you do not have a master account, simply add the standard account. If you have more than one account, simply add them all; there is no limit to the amount of added accounts.

**Requirements**

There are not many. Firstly, an account with read access on the account, EC2 instances and the log repository (to get the flow logs). For a full account policy template, check out the [documentation](https://docs.vmware.com/en/VMware-vRealize-Network-Insight/5.0/com.vmware.vrni.using.doc/GUID-122546CC-FB0F-4D8F-B4E8-225B372CDC31.html).

Secondly, the collector appliance which will poll AWS, will need to be able to access the AWS API over the internet (so, open up the firewall). If you're using vRealize Network Insight Cloud, the connection will go from the VMware Cloud to the AWS API, no local connections needed.

I> When adding an AWS account for Network Insight, make sure it has programmatic access. You'll need the generated access key and secret access key, to add it to Network Insight.

**Enable VPC Flow Logs**

In order to collect network flow logs from AWS, it has to be set up first. Flow Logs is an option per VPC, meaning you need to enable in inside the AWS console on a per-VPC basis. In order to do so, create a Log Group inside CloudWatch where the logs will go, and configure these settings on the VPC:

{caption: "AWS: Setting up VPC Flow Logs"}
![](images/image41.png)

Set the **Filter** option to **All**. It will determine to log both allowed network flows and the blocked network flows by any security group rules that are in place. Currently, Network Insight only grabs the allowed flows, but that might change in the future.

The IAM role should be a role that is allowed to write to the CloudWatch Log Group.

**Geo-location Blocking**

When you're dealing with a global cloud presence, and different parts of your organization are located in different public cloud regions, it is possible that the persons managing the EMEA resources are completely different from the persons that are managing the AMER resources -- and have nothing to do with each other. Picture this; a global organization with a single master account for billing purposes, but linked accounts for the localized teams.

In that case, you might want to have different instances of Network Insight, in order to have the localized teams only look at their own resources. This is possible with the geo-location blocking support inside Network Insight.

During the process of adding the AWS master access as a data source, you can specify from which AWS regions Network Insight is allowed to collect information from. It will leave the unselected regions alone.

{caption: "Adding AWS Account"}
![](images/image42.png)

The above [Figure 39]{.underline} depicts the screen that allows you to add an AWS account. Select a collector appliance that will connect to the AWS API (over internet), supply the access key and secret access key, and hit the Validate button. The collector will now go out to the AWS API and validate the credentials. If it succeeds, the rest of the options will be made available.

I> vRealize Network Insight Cloud has a shared collector for public cloud discovery. You don't have to set up a collector to add a public cloud, meaning you don't have to select the collector in the above process.

If you're adding a master account and would like to collect all linked accounts, tick the first checkbox.

Once you've set up the Flow Logs on your VPCs, also tick the checkbox to enable flow data collection. It will make sure that the collector will start looking at the CloudWatch logs to collect network flows from AWS.

Lastly, you can determine if you want to limit the collection process to only selected AWS regions. Tick the last checkbox, and select the regions you want to collect from, if that is the case.

Give it a nickname and save the new data source. Next, inventory collection will start.

### Inventory Collection

Once AWS has been added as a data source, the collector appliance will start collecting information from AWS via its APIs. This happens every **10 minutes**.

All computing, networking, and relevant administrative objects are pulled in. Things like the AWS account, EC2 instances, firewall rules, security groups, IP subnets, VPCs, and VPC peering connections are given their own dashboards. It'll be possible to use the Network Insight search to uncover the inventory of the AWS environments. This will list all AWS objects across all added AWS accounts; having everything in the same place. Just type in AWS, in order to see the options:

{caption: "AWS Search options"}
![](images/image43.png)

During the inventory, the incoming data is correlated to each other. EC2 instances will have a direct link to its network interface, the applicable firewall rules, availability zone, region, network flows, all the way up to the AWS account. This means you can use all these linked entities to filter, for example; search for all EC2 instances that belong to a specific account or VPC.

### Network Flows

When flow logging is configured on a VPC to log towards CloudWatch, the network flows can be seen from the CloudWatch console. The log streams are grouped by elastic network interface and the action taken by the AWS Security Rule (accept or deny). This is the exact view that Network Insight retrieves using the API.

{caption: "AWS CloudWatch listing network flow logs"}
![](images/image44.png)

What Network Insight does with the flow logs and how they are processed, is similar for both AWS and Azure -- and it is described in the chapter [AWS & Azure Flow Processing]{.underline}, which is located in the chapter about the [Architecture]{.underline}.

When network flows are retrieved from CloudWatch, AWS entities will show up in the [Application Security Planning]{.underline}. Flow visibility will be there throughout Network Insight and you'll be able to generate recommended firewall rules that can be applied onto AWS security groups.

### Network Path Visibility

While retrieving the networking configuration from AWS, Network Insight also constructs a view of the network topologies that are available within AWS, and also any VPN connectivity that connects back to an on-premises (or any of the other platforms that is supported by Network Insight) infrastructure.

While Amazon does not allow you insights into the actual physical equipment that is used to host your AWS resources, there are several logical networking components that we *can* see.

EC2 instances are hosted on a subnet, subnets are connected using a routing table, internet connectivity will go over an internet gateway, VPN connections over a virtual private gateway, and different VPCs can be connected via a VPC peering connection. Network Insight can construct a logical network topology, based on all those components.

{caption: "AWS Network topology between two VMs in different VPCs"}
![](images/image45.png)

We can also monitor these logical network components separately, and for example, monitor the connection from the EC2 instance to the subnet or routing table. Using routing algorithms, Network Insight constructs these topologies on demand, whenever you request a path, by searching for path from AWS EC2 xxx to AWS EC2 yyy.

You can request paths between EC2 instances, or between an EC2 instance and a vSphere VM that's either hosted on-premises or in VMware Cloud on AWS.

{caption: "AWS Network topology between on-premises and an AWS VPC"}
![](images/image46.png)

The above Figure 43 shows a network topology coming from an on-premises vSphere VM behind VMware NSX for vSphere, with a NSX Edge connecting with a layer-3 VPN to a Virtual Private Gateway on AWS.

I> Drawing network topologies into and from AWS is currently only supported with layer-3 VPN tunnels, except for connections going to the internet (as that uses the AWS internet gateway).

Because AWS is pretty restrictive in how you configure the network components that might show up in the network topology, there's not a whole lot to keep in mind. Just make sure connectivity is actually online, and Network Insight will be able to discover the topology. If a path does not show, it is most likely because there's an actual problem on the connectivity and Network Insight will indicate where that problem might be.

### Security Group Tracking

When a network is virtualized and connected directly to the internet, as public clouds typically are, I'd say it's more important to keep control of the security policies, then it would be when your workloads are all on-premises behind an internet firewall which you control. It's way easier to misconfigure security policies and allow all kinds of unwanted incoming connections.

AWS makes it super easy to configure security groups when deploying new EC2 instances; simply create a security group with specific firewall rules for that EC2 instance or select an existing security group. It's when you need to start managing the infrastructure, when they start to bite you.

Security groups in AWS are separately managed. They have a dedicated section in the AWS console. This makes sense. What does not make sense, is that there's no overview of all EC2 instances that are connected to a specific security group. It also doesn't help that the security groups that are attached directly to EC2 instances, can be superseded by something called Network ACLs, which are attached to the VPC.

Network ACLs are global firewall rules which are applied to everything inside the VPC. These ACLs can overrule security group firewall rules. If a Network ACL denies incoming HTTP traffic and the security group firewall rules allow it, the traffic will be denied as the Network ACL takes precedence. It makes sense if you think of the network ACLs to be the border firewall. However, these overlapping rules are not displayed together anywhere, which makes it hard to troubleshoot connectivity issues.

The first thing Network Insight helps with in AWS security management, is putting all the firewall rules that are applicable (security groups) on the same page, when you're looking at a specific EC2 instance. Furthermore, the dashboard for a security group shows all EC2 instances that are attached to the security group. You can execute searches to look for all security groups that allow a certain port from the internet. In short, Network Insight makes it a lot easier to view the security configuration.

The second thing is that for each object in Network Insight, there's a timeline created. Meaning you can scroll back in time and look at the configuration at a specific point in time. Every time a configuration change is detected, a new version is created and there will be a 'blip' on the time line, indicating that there's been a change at that time. If the rules of a security group are changed (added, edited, or removed), an audit log will be created that will list what exactly has been changed and when.

## Microsoft Azure

Microsoft has been hammering on the road of their Azure public cloud offering for a while now. While they're not the first, they're making it pretty easy for Microsoft customers to consume Azure and place Windows workloads in their cloud. With about 15.5%[^5] of the public cloud market share, they are the number two cloud.

While I'll focus mostly on the Azure VM service and the services that support computing resources, Azure has an enormous portfolio of services. Ranging from infrastructure services, to databases, blockchain, identity, application development, and (not limited to) internet-of-things services. They claim they have over 600 services, but they also count sub-services like virtual networks (which I think is a part of the infrastructure/compute service), something which AWS does not count as a separate service.

I'm not going to repeat all the added value that Network Insight brings to Azure, it's exactly the same as with AWS. To recap; Network Insight will bring all networking & security objects into the same interface as on-premises networking & security objects (including the time machine with point in time versions of configurations), it will create correlations between all Azure objects which cannot be easily seen in the Azure Portal (for example; all VMs connected to an Application Security Group), all in the same interface.

### Azure Networking Tools

I'm pleasantly surprised by Microsoft in this case. Azure is a lot more network & security admin friendly then AWS is. It has a bunch of tools that are similar to what AWS but Azure certainly offers more elaborate and extensive tools. Here's a list of the most useful ones:

-   **Security Center**: advises on the 'secureness' of security rules. For example, it detects and alerts on 'allow any any' rules.

-   **Network Watcher**: this is a suite of tools, such as:

    -   **Packet capture**: start a capture of all network traffic and dump the logs in a storage account. No destination VM is needed, like with AWS, and there are options to filter the traffic based on IP source, destination, protocol, and ports.

    -   **IP flow verify**: simulates whether a given network flow (you can input the IP source, destination, protocol, and port) will be allowed through the security policies. It'll display the exact security rule that is allowing or blocking the flow.

    -   **VPN troubleshoot**: does several health checks on a VPN connection and displays a possible cause for the VPN tunnel or the connectivity going over it being down.

    -   **Traffic Analysis**: this comes close to what Network Insight does. Reports will be generated on the NSG flow logs (see below), which contains statistics like; top talkers, geographic traffic distribution, VPN utilization, and some cool network topology diagrams.

-   **Network Security Group Flow logs**: this will log all traffic going over the NSG to a storage account. It's stored in a similar fashion as AWS, as in security rule hit logs.

-   **Virtual Network Tap (vTAP)**: tech preview of distributed port mirroring for virtual networks. The traffic will be sent to a third-party appliance (Gigamon, Flowmon, Ixia, etc.) for analysis.

Although the network & security troubleshooting tooling is relatively complete, these are still point solutions for Azure. If your organization hosts everything on Azure, you can definitely get away with it. But chances are that not everything is there and there are hybrid applications. That's when you need a solution like Network Insight to not have to jump between interfaces and look at your entire network as a whole. Anyway, let's see how that looks!

### Adding Azure to Network Insight

Before adding Azure as a data source, make sure the Network Security Group (NSG) flow logs are configured and sending network flow logs to a storage account. More on that later in [Network Flows]{.underline}.

Network Insight uses an application identity registration in order to get access to the Azure APIs. Application registrations can span a single or multiple Azure accounts, allowing it to see collect data from all Azure accounts under an organization. This application registration will have separate permissions, which can be managed for the sole purpose to let Network Insight collect data.

Meaning, you can determine to which resources it has access, by giving or revoking permissions. Where the application registration will not have access, Network Insight will not see. While I'd recommend giving it access to the entire inventory, you can hide certain parts. Maybe because of confidentially reasons, maybe because of Network Insight licensing reasons (and you just don't want to pay for the development VMs).

More information on the application registration process can be found in the [Azure documentation](https://docs.microsoft.com/en-us/graph/auth-register-app-v2). In any case, make sure the application registration has the following permissions:

Alternatively, you can also use the built-in roles: **Storage Account Key Operator Service** **Role**, **Network Contributor**, and **Reader**.

Once your application registration is done, you'll have an **Application ID**, **Directory (tenant) ID**, and an **Application Secret Key** to show for it. In order to complete the registration inside Network Insight, you also need the Subscription ID. This ID can be retrieved by looking at the Subscriptions page in the Azure portal.

All IDs will look something like this: *77fb6532-da95-4a50-b00f-190ae836b2d8*

{caption: "Adding an Azure data source"}
![](images/image47.png)

Just like with AWS; the collector appliance which is selected, needs internet connectivity to go to the Azure API and collect data. Make sure the firewall(s) are opened up to allow it to connect. When using vRealize Network Insight Cloud, the collector will be the cloud-based collector -- in which case you do not have to deploy one yourself.

I> When using vRealize Network Insight Cloud, it is only supported to use the shared collector with public clouds. In theory you can deploy a collector and configure AWS or Azure on that collector, but keep in mind that that is not supported.

### Inventory Collection

Once Azure has been added to Network Insight, it will start collecting data every **10 minutes**.

All computing, networking, and relevant administrative objects are pulled in. Things like the Azure Data Sources (accounts), Azure VMs, Network Security Groups, Application Security Groups, Azure Subscriptions, Network Interfaces, Virtual Networks, and more, are given their own dashboards. It'll be possible to use the Network Insight search to uncover the inventory of the Azure environments. This will list all Azure objects across all added Azure subscriptions; having everything in the same place. Just type in Azure, in order to see the options:

{caption: "Azure Search options"}
![](images/image48.png)

During the inventory, the incoming data is correlated to each other. The Azure VM dashboard will display all its network interfaces, VNets, Subnets, all applicable security rules (both from Application Security Groups and Network Security Groups), and all network flows.

Just like with everything else in Network Insight; after it is correlated, you can use it to filter searches on. For example; searching for all Azure VMs of a specific type, all VMs that are a part of a certain Application Security Group, or all VMs that are receiving incoming traffic from a specific country, all of this is possible.

### Network Flows

To get Azure network flows to show up in Network Insight, these flows need to be logged first. Flow logging is a part of the Network Watcher suite and is enabled on a per Network Security Group (NSG) basis. An NSG can be assigned on a per VM Network Interface or per Virtual Network Subnet basis, so you have great flexibility with where flow logging is enabled. Honestly, the easiest way to do it; is to enable it on the NSG that is used for the entire virtual network.

On each NSG, there's also a target **Storage Account**. The logs will be stored as **BLOBs** in that storage account. The logs will be readable through the Azure Portal, so you can verify whether logging is actually happening.

Before enabling NSG flow logging, make sure you have storage accounts that are located in the same regions as your NSGs are located, to avoid inter-region traffic costs.

{caption: "Azure flow logs structure"}
![](images/image49.png)

The structure of the NSG flow logs is *really* elaborate. If you browse to the blobs of the target storage account, you will find a folder structure that holds a folder per subscription, resource group, virtual network, NSG name, and a folder per date & time; separating out the year, month, day hour, and even minute in a separate folder. The lowest folder is for the MAC address of the Azure VM that is receiving or transmitting the network flows. Getting to the actual flow logs requires a dive into 15 levels (!!) of this folder structure. You can't say it's not organized, though!

The JSON file which holds the actual network flows, is also really structured, and looks a bit like the example below. This is one record from these JSON files:

The keen observer would notice a few things: the flows that are logged are 5-tuple (source IP, destination IP, source port, destination port & protocol), and the security rule that allows or denies this traffic is also listed. This means that Network Insight will be able to show **blocked flows**. Also, the 5-tuples are trimmed down and deduplicated to the 4-tuple flows that Network Insight shows; we don't really care about the source port.

### Network Path Visibility

As of vRealize Network Insight 5.0, there is no network path visibility into Azure network topologies. All objects are collected, and you can manually see what is connected to each other, but there's no visualization of a VM to VM path.

Please talk to your VMware representative to get extra priority on this feature, if you are using Azure and on-premises infrastructure. The network path visibility is the best way to troubleshoot end to end topologies.

### Security Group Tracking

Inside Azure, there are two kinds of security groups; Network Security Groups (NSG), and Application Security Groups (ASG). The ASG is higher level and essentially is a collection of VMs. The NSG is the group that holds the security rule configuration, and can be applied to VM Network Interfaces, or Virtual Network Subnets. Inside NSGs, you'll have inbound and outbound security rules that dictate network traffic. These security rules are where the ASG comes back, as you can ASGs as source or destination. So, you can create an ASG, put the VMs in it which make up an application, then security that application using inbound and outbound security rules inside NSGs.

In some interviews with organizations that are using Azure constructs, it seems as though the way to go is: use application security groups to allow everything the application itself needs (i.e. API access to an internet service, incoming web traffic to the web servers, etc.), and use network security groups to set global security rules (i.e. everything is allowed to use DNS, to be contacted by an internal IP range, etc.), and internal application micro-segmentation, if needed.

I> The effective firewall rules for an Azure VM can be a combination of Network- and applied Security Rules to Application Security Groups. It's wise to create a security rule design and designate rule priority ranges to NSGs and ASGs.

Although Azure does a good job at presenting the applicable firewall rules on the VM dashboards, it hard to find out what changed at a certain time. The time line functionality in Network Insight gives all Azure the same version history as any other object. This allows you to get notifications when security groups change, and to look at a time line of changes when you're troubleshooting a connectivity issue.

## VMware Cloud on AWS (VMC)

VMware and AWS have partnered to provide solutions that help organizations adopt hybrid cloud to increase flexibility and reduce cost, while leveraging their existing IT investments and expertise. Due to the nature of VMware Cloud on AWS (VMC), managing workloads in the hybrid cloud is not that different from managing them on-premises, existing tools can be used to manage, monitor, and troubleshoot the workloads, and there's not an entire re-education needed of the persons that will manage it.

{caption: "VMware Cloud on AWS -- spanning networking & security across clouds"}
![](images/image50.png)

When you have on-premises infrastructure and want to offload the workloads to a cloud provider, in order not to have to manage a data center anymore; VMware Cloud on AWS (VMC) is one of the easiest options. As VMC is essentially a VMware Software-Defined Data Center (SDDC) with all infrastructure components that you would have on-premises (vSphere, vSAN, and NSX), it is not that different to manage. VMC will also run the same Virtual Machine format as on-premises vSphere runs, so it's easy to migrate existing VMs from on-premises to VMC.

The fact that it's almost the same as an on-premises SDDC, makes it behave the exact same way inside Network Insight as well. All inventory will be discovered, from VMs, to NSX switches, routers, firewall rules, the vSAN datastore; everything. This inventory will blend in with regular VMs, NSX objects, they will just have a differentiated property called SDDC Type. This property can have ONPREM and VMC as its value.

SDDC Type can be used in searches and will be displayed on dashboards. For example, to get a list of all VMs that are running on VMC, execute this search query: VMs where SDDC Type = VMC

I> Multiple VMware Cloud on AWS SDDCs can be added to Network Insight, converging all inventory, network flows, and other metrics into a single interface.

Everything else within Network Insight is exactly the same as an on-premises SDDC. There's one exception, which is how you add VMC as a data source. Let's take a look in the next chapter.

### Adding VMware Cloud on AWS to Network Insight

Just like with a regular vCenter and NSX Manager, there's an order in which to add VMC as a data source. First, add the vCenter and then the NSX Manager.

Adding the vCenter is simple; all you need is the IP address or hostname, and credentials for read-only access and read the inventory. The IP address or hostname can be the external facing connection to the vCenter, or the internal IP address -- provided you have connectivity to the internal network of the SDDC via VPN or Direct Connect.

{caption: "VMware Cloud on AWS -- Adding the vCenter"}
![](images/image51.png)

After adding the vCenter, it's the turn of the NSX Manager, which is a bit more involved. Although VMC runs NSX as an on-premises SDDC would run it; customers only get selective permissions. For instance, you cannot log into the NSX Manager directly and have to go through the Cloud Services Portal (CSP) website to manage the networking & security configuration in VMC. Customers cannot manage the backend configuration, like the overlay network settings, uplink port bindings, edge nodes, etc. It makes sense too, because VMware manages that for you; just focus on what the VMs & applications inside the SDDC need -- simply network connectivity, security policies, and maybe a load balancer.

Adding Network Insight into the mix does not change the permission level. When Network Insight is connected to an NSX environment on VMC, it gets the same privileges as the customer would. That is why the authentication from Network Insight to the NSX Manager in VMC, works a little different. Instead of a username and password as credential, VMC uses a so-called CSP Refresh Token.

With this refresh token, it is possible to converse with the CSP APIs. The refresh token is tied to a specific user account inside CSP and can be created with limited permissions, and limited lifespan. You can create multiple refresh tokens, and I suggest you create one for each separate purpose -- so you know exactly which token is used by what (and can revoke it, if needed). More information on how to create the CSP Refresh Token, check out the [VMware documentation](https://docs.vmware.com/en/Management-Packs-for-vRealize-Operations-Manager/1.0/vmc/GUID-3B8C8821-FB07-412F-A2E4-C5CA34D8A473.html).

{caption: "VMware Cloud on AWS -- Adding the NSX Manager"}
![](images/image52.png)

In short, Network Insight talks to the CSP API in order to get network & security objects from VMC. However, that API is located on the NSX Manager that is hosted inside VMC -- meaning the IP address or hostname can be the external IP address of the NSX Manager, or the internal IP address, as long as you have VPN or Direct Connect connectivity to the internal VMC SDDC network.

### Network Flows

Because NSX-T powers the virtual networking of VMware Cloud on AWS, Network Insight uses the same method to ingest network flows from the VMC networks; the NSX-T Distributed Firewall generates NetFlow (IPFIX) and sends it to the Network Insight collector appliance. The flows that arrive at the collector, get processed (correlated against objects like VMs, virtual network, geolocation, etc.), and then get send to the platform appliance **every 10 minutes**.

Connectivity to the vCenter and NSX Manager is something to take into account when designing your environment. As opposed to AWS and Azure, the network flows from VMC come from actual NetFlow (IPFIX), and not from log files existing out of purely text. This has to be a consideration in where the collector appliance is placed, as NetFlow takes up more bandwidth over flow log polling with AWS & Azure. Calculations as to how much traffic you can expect, are in the [Bandwidth Requirements for Flows]{.underline} chapter.

It's not only more bandwidth usage, NetFlow is transmitted in UDP and clear text; which is not something you want to send over the internet. The best course of action is to place the collector appliance for VMC on the internal network. This can be both the internal network inside the VMC SDDC, or it can be the internal network in an on-premises SDDC.

When placing the collector appliance inside VMC, make sure there is connectivity between the collector and the management network where the vCenter and NSX Manager reside.

When placing the collector appliance on an on-premises infrastructure, make sure there is either VPN or Direct Connect connectivity to the management network of VMC, and communication is allowed through the firewall.

In any case, you want to make sure the NetFlow does not go over the internet and is kept as close as possible to the NSX Manager.

### Network Path Visibility

While retrieving the networking configuration from NSX inside VMC, Network Insight also constructs a view of the network topologies that are available within AWS, and also any VPN connectivity that connects back to an on-premises (or any of the other platforms that is supported by Network Insight) infrastructure.

I> Direct Connect is currently not supported as of version 5.0. Network Insight will get all the NSX networking & security configuration but will not be able to draw a network topology from VMC to on-premises via a Direct Connect.

VMware Cloud on AWS is hosted on AWS (duh), meaning we do not have full access to the actual hardware network appliances that are being used in their infrastructure. Network Insight will not be able to pull data from appliances like the top-of-rack switch, or core routers that connect your SDDC to the internet. However, everything inside the SDDC is fair game.

NSX provides the virtual networking & security stack for VMC, so we have insights into the networks the VMs are attached to, and the first two routed hops (NSX T0 and T1 gateways, or otherwise known Compute Gateway (CGW) and Provider Gateway (PGW)). On the T0 router, internet is directly attached, and VPNs can be terminated.

Paths can be drawn from a VM inside VMC to the internet, showing the routing and firewall rules in between. More interestingly, paths can also be drawn between a VM on VMC and an EC2 instance in AWS, between a VM on VMC and an on-premises infrastructure, and between different VMs that are both hosted on VMC.

The VMC to AWS or on-premises path depend on layer-3 VPN connectivity to another device that Network Insight supports and is reading out as a data source. For example, the NSX-v Edge, Palo Alto Firewalls, CheckPoint Firewalls, or terminated directly on an AWS Virtual Private Gateway.

Here's an example path between a VM hosted on VMC and an on-premises SDDC, which are connected via a VPN tunnel that terminates on an NSX-v Edge appliance:

{caption: "VMware Cloud on AWS -- Hybrid Path"}
![](images/image53.png)

Just like with AWS, there is not much room for customization on the network topology and connectivity for VMware Cloud on AWS. The defaults suffice for most customers. This is a good thing, as you don't have to worry about setting up the network topology. Because of this approach, there's not much requirements that Network Insight has, when it comes to showing VM to VM paths inside VMC or between different locations. Just have the layer-3 VPN in place and make sure all networking devices are added as a data source, and Network Insight will discover the VMC network topologies.