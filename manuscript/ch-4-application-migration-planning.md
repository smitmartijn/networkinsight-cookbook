{id: ch-application-migration-planning}
# Application Migration Planning

When you decide to migrate your applications to Public Clouds, like AWS, Azure, or VMware Cloud on AWS (VMC on AWS), a few critical considerations should be kept in mind. First and foremost, insight into your applications and application behavior is critical.

Questions to ask yourself are:

1. "Which components make up an application?"
2. "To what resources does it need access to function?"
3. "How much data is being shipped from and to that application?"
4. "What is the network throughput profile (packets per second, routed or switched traffic, throughput) of the application?"
5. "What components of the application are worth migrating?"

There are four steps in the application migration process:

1. Assess the current application landscape and get insight into the application blueprints.
2. Use this insight to determine suitable applications for migration and create migration waves, aka *Move Groups*, which are groups of applications that are migrated together.
3. Bandwidth requirements and related cloud costs is not something you want to skip. For internet-facing applications, these bandwidth requirements depend on the end-users, and where in the world they are coming located.
4. Migrate the selected applications.
5. Validate application behavior, post-migration.

vRealize Network Insight Cloud can help with steps 1, 2, 3, and 5. [VMware HCX](https://cloud.vmware.com/vmware-hcx) -- workload migration tool extraordinaire, can help with step 4. This chapter focuses heavily on steps 2 and 3, which are critical in the planning phase. For the scoop on step 1, refer back to the chapter about [Application Discovery](#ch-application-discovery).

{id: ch-application-discovery-assessment}
## Application Discovery & Assessment

It's quite common to have an incomplete CMDB or to have multiple sources where applications and the associated workloads are documented. vRealize Network Insight can use a combination of sources (tags, naming conventions, CMDB input) to discover the applications from the infrastructure metadata. For more details on application discovery, refer back to the chapter about [Application Discovery](#ch-application-discovery).

After getting the applications into Network Insight, there is a clear view of application dependencies and network requirements. Navigate to the Security Planner (refer back to [Analyzing Network Flows](#ch-analyzing-network-flows) for a refresher) and group the donut by Applications.

{caption: "Application Migration Planning -- Dependency mapping", id: "fig-application-migration-planning", width: "70%"}
![](images/ch-4/dependency-mapping.png)

By hovering over a slice or using the search bar at the top right of the widget, you can find the application you want to map dependencies for. By clicking the slice, you get the details of that application.

What you're doing here ([Figure: Dependency mapping](#fig-application-migration-planning)), is called **Application Dependency Mapping**. Because it's evident that the application CRM-Records has only incoming connections from 5 other applications, it can be concluded that CRM-Records does not have any dependencies on other applications. However, it is used by 5 other applications, making CRM-Records a dependency for those other applications. This becomes important when you're planning a migration, as applications that are heavily dependent on each other should be grouped and migrated simultaneously, to prevent slowing down the connections between these applications.

{caption: "Application Migration Planning -- Application Details", id: "fig-application-details"}
![](images/ch-4/application-details.png)

I've focused on the CRM-Records application here, which consists of a MySQL database server. You can see the other applications that are talking to this application in [Figure: Dependency mapping](#fig-application-migration-planning). When you zoom in on the application itself, the network requirements for this application present themselves ([Figure: Application Details](#fig-application-details)). By adding up the different Service Endpoints, you can determine that this application is using up **1.1MB** of total traffic, with a peak throughput of **343bps**. Service Endpoints is a way of classifying specific services because multiple services can live on the same workload. Obvio1usly, this example application isn't representative of an actual production application, expect higher numbers in real life.

When looking at this window, you can see the types of network ports being used as service endpoints within this application. These network ports could impact the decision whether or not to migrate this application to the cloud. Or determine to which cloud locations you are allowed to migrate it. There might be some service or network port that, for security reasons, your security policies would prevent from hosting outside the private data center. Databases with Personally Identifiable Information (PII) data might fall under regional laws, and these laws would prohibit the data from being hosted on another country or continent.

While doing this exercise per application, it might turn out that the application is hugely bandwidth-hungry towards the internal users. The **External Services Accessed** tab is where the dependent service endpoints for this application are found. It provides the same view as in [Figure: Application Details](#fig-application-details), only the other way around, from this application to other services in the infrastructure.

This same window can be used to implement security policies when migrating the application, as the [Recommended Firewall Rules](#ch-recommended-firewall-rules) tab shows the exact firewall rules that need to be implemented for this application.

## Internet Traffic Patterns

One of the perks of migration to a public cloud is that you can choose the optimal location in the world to make sure your users benefit. But, without networking insights, this is often overlooked. For internet-facing applications, Network Insight can help with determining where in the world the network traffic is coming from, so you can choose the most optimal cloud location. It does this by attaching geolocation properties (City, Country, Region, and Continent) to each network flow that it processes, using the widely used GeoIP database from [MaxMind](https://www.maxmind.com/en/geoip-demo). Considering that it is possible to filter, group, and sort by every property, you can get a clear insight into where the end-users of your applications are located.

### Per Application

One way to go about it is by looking up bandwidth requirements per application. In this case, you would determine that a set of applications should be migrated because of business reasons (availability, flexibility, etc.), and you then go about to verify the possibility and impact of migration these applications.

In this case, the application called **CRM-Records** is one of the applications that is selected for migration, and I'll show you how to get the required information. First, let's start with the geographical distribution of network traffic for this application. Execute the following search query:

`sum(bytes) of Flows where Application = 'CRM-Records' group by Country`

{caption: "Application Migration Planning -- Traffic per Country"}
![](images/ch-4/traffic-per-country.png)

The above search query gives the amount of traffic for the **CRM-Records** application, grouped by country. As you can see, most of the traffic is coming from Northern Europe. To optimize the user experience, the application should be migrated to London (which is currently the most prominent, central, and connected location in Northern Europe).

### Bandwidth Egress Costs

By not specifying a source or destination country, both egress and ingress traffic numbers are combined. As you might know, ingress network traffic to public clouds costs nothing (or very little); it's the egress network traffic where they get you. To prevent financial surprises, be sure to identify the egress network traffic numbers by using this search query:

`sum(bytes) of Flows where Source Application = 'CRM-Records' group by Destination Country`

{caption: "Application Migration Planning -- Egress traffic per Country", id: "fig-egress-traffic-per-country", "width: 70%"}
![](images/ch-4/egress-traffic-per-country.png)

This output paints a clear picture of how much egress network traffic this application is sending out, which would be leaving the public cloud, and which is billed by the provider. It's also possible to make it easier, by removing the group by statement like so:

`sum(bytes) of Flows where Source Application = 'CRM-Records' and Flow Type = 'Destination is Internet'`

{caption: "Application Migration Planning -- Egress traffic total", id: "fig-egress-traffic-total", width: "80%"}
![](images/ch-4/egress-traffic-total.png)

B> Note: the results of search queries in [Figure "Egress traffic per Country"](#fig-egress-traffic-per-country) and [Figure "Egress traffic total"](#fig-egress-traffic-total) have been taken on different dates, which is why they differ in numbers.

Keen observers noticed that I replaced `group by Destination Country` with `Flow Type = 'Destination is Internet'` in the last two examples. The reason for this is simple; when using the Country property in a search query, the results only show internet flows, as those are the only ones that have a geolocation attached. When not looking for a location property, the results include all network flows, including the flows that stay within the datacenter. By using the Flow Type filter, the results are limited to the flows going to the internet.

### Looking at all Applications

A different way of going about it is by looking at bandwidth requirements for all applications and determining the migration waves based on that information. This typically happens when the entire application landscape is migrated and not specific applications.

This process starts by getting a list of the applications that are the most bandwidth-hungry and looking at their dependencies. Let's start by getting a list of the top applications by executing this search query:

`sum(bytes) of Flows where Source Application = 'CRM-Records' and Flow Type = 'Destination is Internet'`

{caption: "Application Migration Planning -- All application traffic"}
![](images/ch-4/all-application-traffic.png)

Focus on the top 10 applications and map out their dependencies, as we've talked about in the chapter [Application Discovery & Assessment](#ch-application-discovery-assessment). When going through that exercise, migration waves automatically starts to form, as the dependencies (applications) of the top 10 applications should be put in the same migration wave.

## Creating Migration Waves

After gaining visibility into the existing applications and their behavior, it's time to define the migration waves and put together the applications that should be migrated together. From the application list, that is curated inside Network Insight; assign priorities. This is typically done based on business reasons, as there are always some applications more critical than others (e.g., production vs. test). Priority can be given to applications that badly need to be able to expand using cloud resources. Priority could be given to applications that live on infrastructure that's beyond their end-of-life status, and there's a risk of downtime. I could go on; there are many reasons why specific applications can get priority; it usually comes down to the reasons why your organization decided to migrate to the cloud in the first place! ;-)

Now, while I'm focusing on a per-application process (as most organizations tend to focus on applications), this same exercise can be done on a per-network (VLAN) basis; just change the grouping. If everything is being migrated, it's also possible that the migration waves on a per-network basis. If the entire application landscape is to be migrated, it might even be easier to migrate on a per-network basis, as there is no need for complicated networking configurations during the migration (i.e., stretching the networks between the locations to keep the network of the workloads available).

T> It is possible to get a list of applications that are on a network by executing the following search query:
T>
T> `Application where IP Endpoint.Network Interface.L2 Network = 'VLAN-10'`

After assigning priorities, the creation of the applications migration wave groups can begin. While doing this, the following questions should be considered:

1. What are the network requirements to move this migration wave?
   - How much traffic will the migration wave create between the cloud and on-prem network?
   - Have you sized the connection to the cloud to support this traffic?
   - What ports and protocols do the applications need to communicate on, and will the network allow this communication?
2. Will there be any limits triggered by the destination cloud?
   - Is there a limit for the number of VMs in a network?
   - Is there a limit to the amount of throughput or packets per second?
     -  There might be a difference between the maximum throughput or packets per second for internal traffic and internet traffic.
3. Is a temporary layer-2 bridge necessary?
   - If you are migrating on a per-application basis, it might be required to create a layer-2 bridge to allow the IP subnet to exist in both the on-prem infra as the cloud infra. VMware HCX can create these layer-2 extensions, but there is a limit to how much throughput and the number of layer-2 extensions it can handle.
   - Should proximity routing be enabled while the layer-2 extension is online?

While network requirements (number 1) are the most important to check, as it directly impacts application performance during or after migration, the rest of these questions can prevent the migration from finishing. If not checked correctly and in advance, you might hit these limits while in the middle of the migration and need to halt the migration, or even reverse the migration. Unfortunately, this has happened to plenty of organizations, costing them user-experience by causing downtime, extra time to either revert or changing the migration plan.

To start answering these questions, create another application within Network Insight that's called "Migration Wave X" and place the applications that are in each migration wave inside. Essentially, you are creating nested applications. To know which applications should be in the same migration wave, map out application dependencies as I talked about in [Application Discovery & Assessment](#ch-application-discovery-assessment).

After creating these applications, it is possible to scope the security planner on each migration wave and see its dependencies.

As an example, the below screenshot lists the security planner for Migration Wave 1, which is built with the previously used application CRM-Records. This migration wave 1 includes the CRM-Records app, and all of its dependents (Test-CRM, Webshop, SAP, VDI, and Tanzu tees).

{caption: "Application Migration Planning -- Migration Wave Dependency Mapping", width: "70%"}
![](images/ch-4/migration-wave-dependency-mapping.png)

Just like with the previous application dependency mapping exercise, zooming in (clicking) on each of these connections is possible. Make sure the requirements to move this migration wave are clear, by investigating all connections that flow to the remaining infrastructure (shared physical/virtual service slices) are not overly excessive. If it turns out the requirements between this migration wave and some physical service is something like 1Gbit per second, you might need to reexamine the migration wave.

By the way, this migration wave is absolutely perfect, as it has no dependencies on other migration waves. Typically, you would see some connections going to other migration waves. That helps you determine the order of when to migrate the migration waves, to limit the time that the dependency network traffic has to flow from the cloud to the yet-to-be migrated waves.

Before moving on, make sure you have a full picture of the network requirements within this migration wave; click the Migration Wave slice and check internal traffic requirements, and check the connections between the internet, and lastly, all remaining infrastructure connections. Use the same data to determine what security policies should be in place between the cloud and your on-premises data center, to prevent connections being blocked while migrating.

## Limitation Check

The cloud is not unlimited, no matter what the cloud providers say. There are limits to how big workloads can be, limits to how many workloads can share a single network, limits to the number of security policies, limits to the number of networks inside a virtual cloud construct, I can go on. Before attempting to migrate anything, create a design for your new cloud infrastructure, and use the cloud provider limits to make sure everything fits.

At this phase, the data collected in the previous chapters comes into play. You now have a clear picture of the network requirements, which applications to move and the number of VMs serving to those applications, differences between network throughput and packets per second, for internal traffic, internet traffic, and traffic flowing to your on-premises data center.

### Compute & Storage

Network Insight also has compute and storage data, which can be used to understand the requirements to size the target cloud. For example, to get the number of CPU cores and consumed memory by all workloads in a migration wave, use this search query:

`sum(CPU Cores), sum(Memory Consumed) of VMs where application = 'Migration Wave 1'`

{caption: "Application Migration Planning -- CPU & Memory requirements", width: "70%"}
![](images/ch-4/cpu-memory-requirements.png)

While the same can be achieved for disk usage and make the picture complete, Network Insight is focused on the networking aspects of your applications, not the compute & storage. It's entirely possible to get an overview of the current usage, but Network Insight should not be used for capacity management and prediction for compute and storage resources. [vRealize Operations](http://vmware.com/go/vrops) (vROps) is much better in getting that picture.

Have vROps monitor your environment for a few weeks, and it constructs a dynamic picture of the compute and storage resources needed to host the applications; past, current, and future requirements. It also has a built-in [Migration Planning feature](https://docs.vmware.com/en/vRealize-Operations-Manager/8.0/com.vmware.vcom.core.doc/GUID-A0D6E8A5-58F9-43CC-BB29-AB0AFDBCE1A2.html), which provides you with the resources needed for the migration (including future growth). It even provides the costs of hosting these resources in several different clouds (native AWS, VMware Cloud on AWS, IBM Cloud, Azure, Google Cloud).

### Network

Let's get back to the comfort zone of Network Insight. To properly plan the destination cloud infrastructure, we need to know the network throughput and packet per second usage of the workloads inside the migration wave. Because Network Insight correlates all possible information together, we can find these metrics and link them back to the migration wave group directly.

Uncovering the required information can be done by using the search engine. Let's start by looking up the network traffic that is purely internet traffic. Execute the following search query:

`series(sum(byte rate),300) of flow where source application = 'Migration Wave 1' and flow type = 'Destination is Internet'`

{caption: "Application Migration Planning -- Internet Traffic of Migrate Wave 1"}
![](images/ch-4/migration-wave-1-internet-traffic.png)

To deeply understand the mechanics of this search, wait until you've reached the [Using the Search Engine](#ch-search) chapter. For now, know that this result shows you the sum of the byte rate (network traffic), of network flows where Migration Wave 1 is the source, talking to the internet. While this example shows a lack of internet traffic (having a whopping 2bps at some point), it gives a clear picture of the internet traffic behavior, and therefore requirements.

The above search shows the internet traffic over a period of time (by default, the last 24 hours). You should also get the maximum peak throughput of all historical flow data that Network Insight has stored. This is possible by adding the max() operator:

`max(series(sum(byte rate),300)) of flow where Source Application = 'Migration Wave 1' and Flow Type = 'Destination is Internet'`

{caption: "Application Migration Planning -- Peak internet Traffic of Migrate Wave 1", width: "30%"}
![](images/ch-4/migration-wave-1-peak-internet-traffic.png)

This search shows you the peak network throughput. If Network Insight has been running for 1 month, the maximum peak of traffic per second in that month, was **901.1 kbps**. This number can be used for sizing the internet gateway at the destination cloud.

#### Traffic Types

Get these graphs and maximum values for a few different traffic types; internet, east-west, and within the migration wave. Splitting up the different types of traffic is easy; the **flow type** can be changed to view a bunch of different traffic types. From the Internet to East-West, to Routed, to Switched, and a lot more. Have a look at the auto-completion and scroll through the options:

{caption: "Application Migration Planning -- Flow Types", width: "60%"}
![](images/ch-4/flow-types.png)

There are over 50 different flow types, which all categorize different types of network behavior. Depending on what limitations the target cloud has, different flow types should be used to see the requirements.

#### Packets per second

Let's go through one more example with another metric. In most cases, there is a limit of packets per second at the destination cloud. To make sure there are no surprises, get the packets per second as well. All you have to do is to change the focus property in the search query, like this:

`series(sum(flow.totalPackets.delta.summation.number),300) of flow where Source Application = 'Migration Wave 1' and Flow Type = 'Destination is Internet'`

{caption: "Application Migration Planning -- Internet Packets p/s of Migrate Wave 1"}
![](images/ch-4/migration-wave-1-internet-packets.png)

Make sure the packet per second rate is also sized properly, and use the max operator to get the maximum number of packets per second:

`max(series(sum(flow.totalPackets.delta.summation.number),300)) of flow where source application = 'Migration Wave 1' and flow type = 'Destination is Internet'`

{caption: "Application Migration Planning -- Peak internet packets p/s of Migrate Wave 1", width: "30%"}
![](images/ch-4/migration-wave-1-peak-internet-packets.png)

Rinse and repeat this for other flow types, such as routed and switched traffic by changing the flow types. Using these searches gets you all network data you need to determine the minimum requirements.

After determining the minimum requirements for the network at the destination cloud, make sure you add a buffer on top. Remember, all data is current data and does not take into account application growth. Typically, adding a buffer of 40 to 50 percent for healthy growing applications, is a good practice.

## Migrating the Applications

Now that the planning is done, the migrations itself can begin. I highly recommend [VMware HCX](https://cloud.vmware.com/vmware-hcx) for this task, and not only because it's one of VMware's products. It allows for cold and live migrations from and to different vSphere platforms, using already built-in tooling, such as vSphere replication. HCX can extend layer-2 networks to stretch an IP subnet between on-premises and VMC on AWS.

Learn more about [VMware HCX here](https://cloud.vmware.com/i-need-to/migrate-to-the-cloud).

## Validating Application Behavior

After migrating an application, you can validate its behavior by using the security planner. Check the application connectivity from before the application was migrated and after. You can do this by using the built-in time machine on every page:

{caption: "Application Migration Planning -- Validate Application with the Time Machine"}
![](images/ch-4/validation-time-machine.png)

There are more ways to validate application behavior. For example, the Application Dashboard shows a topology of the application, all metrics that relate to it, and all events that might cause problems.

Go through at least the Security Planner and the Application Dashboard. These provide an excellent pre-migration and post-migration picture, and you can make sure the application is still communicating with the right services. If the network fabric of the target cloud is based on VMware NSX (like VMware Cloud on AWS, IBM Cloud, or a private cloud), Network Insight also logs network flows that are denied by the security policies. Meaning, the Security Planner and Application Dashboard also shows network traffic that is being denied by the firewall. This is especially good to check when doing the post-migration validation.