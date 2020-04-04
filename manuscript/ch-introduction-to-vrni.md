# Introduction to Network Insight
With all that data the actual use cases for Network Insight are limitless, but there are four main use cases that will help you get started to get an idea on what to use it for.

## Use Case -- Application Security Planning (Micro-segmentation)

Security in the data center is evolving and micro-segmentation is a technique that’s being applied more and more. You can write an entire book about the technique itself, but I’ll give you a quick summary in chapter Application Security Planning  when we go into the depths of the micro-segmentation planner.

The hard part about micro-segmentation is where to get started. As a security person, you need a lot more information about the workloads then you normally get from the application team or vendor, to properly perform micro-segmentation.

Using the network flow data and workload information that is collected by Network Insight, it provides you with a jumping board to accelerate any micro-segmentation implementation. You’ll get clear views into the communication between workloads and applications and you have the ability to export recommended security policies that can be applied to VMware NSX.

{caption: "Global numbers of Network Traffic movement"}
![Global numbers of Network Traffic movement](images/image1.png)

{caption: "Recommended Firewall Rules Grouped by Application"}
![Recommended Firewall Rules Grouped by Application](images/image2.png)

## Use-Case -- Getting actual visibility into your environment

Besides using NetFlow data to provide insight into the traffic going through your network, Network Insight also gathers data from your private cloud running on vSphere, public cloud on AWS and physical equipment that helps run those environments. It uses all that data to paint a complete picture of your workloads, from a virtual machine to the physical wire between 2 routers to the public cloud instance where your web server is running.

When it comes to troubleshooting, that is where the gold is. Network Insight provides a holistic view of your entire environment, which means you can quickly and easy find root-causes to any issues you’re having.

{caption: "Topology chart: gluing physical and virtual together"}
![Topology chart: gluing physical and virtual together](./images/image3.png)

Using the network topology maps, you can quickly determine issues on a specific network path or using the search engine to look for network devices that are misbehaving or showing anomalies, your troubleshooting process can be much more efficient and quicker.

## Use-Case -- Doing the health check boogy

After getting all of this data from your virtual and physical environment, Network Insight will check the configuration of virtual and physical devices against the VMware Knowledge Base (KB) and best practices. It will provide you with a list of problems in the configuration and ways on how to fix them.

These problems will be categorized into severities: Critical, Moderate, Warning & Info and example might be that it has discovered an MTU mismatch (important when using an overlay) on the physical networking equipment or if high availability on an NSX Edge is not enabled and you’re at risk for downtime when it needs a fail over.

{caption: "Health Check and Health Alerts"}
![Health Check and Health Alerts](images/image4.png)

These health checks are a good way to keep your environment in check and configured as per the latest best practices.

## Use-Case -- Migrating to the Cloud (or anywhere)

These days, it’s not surprising to have a cloud-first strategy when it comes to developing and deploying new applications. Public Clouds like Amazon Web Services (AWS), Microsoft Azure or Google Cloud Platform have extensive services that save you a lot of trouble having to build your own. Solidified services like Infrastructure- (workloads), Database-, Storage- and CDN-as-a-Service are making way for services that can be used to kickstart application development. Things like Artificial Intelligence & Machine Learning, Containers, IoT management, facial recognition, forecasting, serverless, are all being offered as-a-Service so developers can focus on their actual business requirements and churning out code for that purpose. Not be side-tracked by creating their own generic (but mandatory) services which they only need to support their applications and isn’t core business.

What does this have to do with Network Insight, you ask? Well, unless your organization is starting from scratch and has no history whatsoever, you’re going to have existing infrastructure and applications. In certain cases, strategies come into life to use public clouds with IaaS instead of on-premises and applications need to be migrated towards the public cloud.

This use-case is also valid for migrations in general. Whether it be to the public cloud, to a different on-premises data center or migration away a piece of your company. Any scenario where you have an existing application infrastructure and need to migrate those existing workloads.

Here is where Network Insight can help you out map the application landscape and discover what talks to each other. Applications are usually interconnected, even if they shouldn’t be, at all. As it’s in the best interest for application performance to keep a grasp on those interconnections, you need to have a proper map of your application landscape.

As Network Insight understands application constructs and sees everything that happens on the network, it will give you the map of your application landscape on a silver platter. Besides network data, this map will be supplemented by compute and storage performance data to get a handle on the needed resources for the migration. You can then turn around and use this map and plan out your migration.

With a good understanding of the application map, it is much easier to retain application performance when doing migrations.

A deep dive on this topic can be found in the chapter Application Migration Planning.


## Use-Case -- Visibility for Containers

According to results of a survey that the Cloud Native Containers Foundation did in 2018, the main concerns around moving containerized applications into production (and keeping them there & healthy) is around **Complexity**, **Monitoring**, **Networking** and **Security**.

{caption: "CNCF 2018 Survey results"}
![CNCF 2018 Survey results](images/image5.tiff)

VMware has seen these challenges and is taking them on in several ways. One of those ways is through NSX Data Center for Containers, where network virtualization is delivering simplicity, flexibility and security up to the container level. Operations engineers can use the same security controls over containers as they can over VMs and make sure the applications are put into production safely -- whether the application is hosted on-premises or in the public cloud. That's **Networking** and **Security** covered.

Network Insight comes in to provide clarity of the container environment, reducing the **Complexity** and allows you to **Monitor** this environment for any **Networking** and **Security** related events. It does this by making connections between the container world, virtualized world and physical world. In doing so, it creates full end-to-end visibility in order to make sure the production environment is up to snuff and no components are acting up.

Network Insight can also be used to plan out the security for these container workloads. Due to the tight integration with NSX Data Center for Containers, Network Insight gains the same network flow visibility as with VMs. This means the same security planner can be used to map out network connectivity between the different levels of the containerized application and the best part is that Network Insight can generate recommended firewall rules based on those real-time network flows. Oh, and it also allows these recommended firewall rules to be exported in a format that Kubernetes understands (YAML). Applying these rules is as simple as performance the export and using kubectl to apply them directly to a running application.

More on how to do this in the chapter [Application Security Planning]{.underline}

## vRealize Network Insight versus vRealize Network Insight Cloud (SaaS)

When reading about Network Insight, you might come across two versions of the name: vRealize Network Insight (or vRNI) and vRealize Network Insight Cloud (or vRNI Cloud). The reason for this is that there is an on-premises and a software-as-a-service (SaaS) version.

In 2018, VMware came out with a SaaS version which is hosted on their Cloud Services (<https://cloud.vmware.com>) infrastructure. The reasons for this are to simplify deployments and unburden organizations with upgrades or availability of the platform. VMware takes care of the upgrades and availability and you can simply consume the product. Capability-wise, the 2 are equal; they have the same features and same interface. Everything in this book pertains to both versions.

The architectural components in the on-premises version and SaaS version are similar, they're only hosted in different locations. The chapter [Architecture]{.underline} goes deeply into the architecture of both versions, but here's a sneak peek: Network Insight consists out of collector nodes and platform nodes. The collectors talk to your infrastructure endpoints to collect data and the platform is the central data repository and it is your entry point (the user-interface).

In the on-premises version, both components run on your own infrastructure (which can even be air-gapped) and you keep everything local. With the SaaS version, the collectors run locally (in order for them to connect to your infrastructure) and they send their data towards the cloud hosted platform nodes.

This usually brings up 2 security questions; how is the data transferred and what exactly is stored in the cloud? Stay tuned for details on the transfer in the [Architecture]{.underline} chapter, but let's have a look at what data is stored inside the platform.

Collectors look at data center infrastructure equipment (switches, routers & firewalls) configuration like firewall rules and switchport configurations, virtual workload environments for virtual machines, network traffic flow data (optional but valuable) where source, destination and protocol are recorded. Of the virtual machines the metadata is recorded (network settings, hostname, OS type, etc.) and cross-linking is done between the different data types. For instance, a network flow can have a VM tagged to it if the VM was either the source or destination of the flow.

It is contained to infrastructure metadata and network flows though, no user or application data from inside the VM is collected. This makes it a lot easier to determine whether you could host this metadata in the Cloud, as most governmental regulations about data locality pertain to user and/or application data.

If you can make use of hosted version, it does save you a bunch of management effort by outsourcing that to VMware.

## Virtual Cloud Network

Network Insight fits neatly into the Virtual Cloud Network (VCN) vision of VMware and to set the stage, let's talk about the Virtual Cloud Network for a moment.

Networking itself is not new to VMware. They have been doing virtual networking for more than a decade in the core of virtualization and it's been said VMware is the leading vendor with most used switch ports on the virtual switches inside vSphere. In 2009 VMware released vShield Networking & Security to branch out to all kinds of networking services, such as firewalling, routing, load balancing, VPN, NAT, etc. That evolved into NSX for vSphere and NSX for Multi-Hypervisor Center, which were released in 2013.

Since then, NSX has turned into a platform for networking & security that extends throughout the entire enterprise network. With NSX Data Center providing services for applications hosted in the data center, NSX Cloud that integrates natively into public clouds to allow the same security policies everywhere (instead of point-solutions per cloud), NSX SD-WAN by VeloCloud to bridge the gap between branch locations and the data centers and clouds, AppDefense for zero trust application security on the operating system level, to NSX Hybrid Connect (HCX) that makes it possible to migrate workloads between the different places where these workloads can live.

{caption: "VMware NSX Portfolio"}
![VMware NSX Portfolio](images/image6.tiff)

This translates to uniform networking & security policies and services that can be delivered anyplace that can run software (so, everywhere). As application workloads keep spreading out from the data center towards more locations to be either close to their data and operating spaces (IoT is really becoming a thing), or locations that have specialized services (AI/ML as a Server, Function as a Service, things like that), we can keep the same network & security policies in place, we can deliver the required networking services on all locations, and we can do that without having to learn the platform specific networking components.

The Virtual Cloud Network is already being delivered by this family of products and will only get better whilst they get more and more integrated.

Network Insight is at the center of these developments, integrating tightly with NSX and the underlying transport networks, making sure you can monitor and troubleshoot the Virtual Cloud Network and the underlying physical networks that are being used. It is the main operational dashboard from where the health of the network is found and where each individual component can be found and be seen in the big picture of the entire network. You can zoom in on components that are misbehaving and troubleshoot them in context of the configuration and status of neighboring components.

Network Insight is used for planning purposes to determine what dependencies there are in order to migrate applications between locations and determine what impact such an operation has on network performance. The same insights can be used to properly plan security policies for the applications living inside the network, making sure you can secure your applications -- where ever they might go.

{caption: "VMware Virtual Cloud Network"}
![VMware Virtual Cloud Network](images/image7.tiff)

Network Insight is the key to making the Virtual Cloud Network vision a reality. By broadening the insights to everything in and around the Virtual Cloud Network, it allows you to execute on this vision, while keeping it easy to operate and maintain.