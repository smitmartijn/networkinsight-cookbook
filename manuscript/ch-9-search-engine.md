{id: ch-search}
# Using the Search Engine
**This chapter was authored by Rohit Reja, Senior Staff Engineer on Network Insight. He was part of the original ArkinNet team.**
**Contact Rohit here: <https://www.linkedin.com/in/rohit-reja-1b11335/>**

Edited by: Martijn Smit

vRealize Network Insight provides a datacenter semantics aware search engine to find objects of interest based on their configuration quickly. This search engine sits on top of a highly scalable Elastic Search cluster, and it can provide rich insights into sophisticated virtual and physical data center compute and network fabric.

The search engine is powered by an Elastic Search cluster, which is the first service that gets addressed when a search is requested. Elastic Search can point to other databases from where Network Insight pulls its data. For example, index pointers for metric refer to the HBase database, while configuration items (like VM properties) get referred to PostgreSQL. Indexing happens continuously; each time an object is updated in any database, Elastic Search updates its index.

This indexing process can lag a few seconds, causing old data to show up still or new data not to show up yet. That's why you can monitor the indexer lag from the **Install and Support** page in the **Settings**.

The syntax is a technical natural language, and the search bar helps by doing autocomplete on keywords and values (i.e., VM names). You can simply start typing what you need into the search bar, and chances are, you find the search query you need. Think of it as Google for your data center.

Configuration objects, metrics, pinboards, static pages, and flows can all be searched. Configuration objects are single values that typically don't change every second (e.g., *CPU Count*, *Host*, *VM*, *Distributed Virtual Portgroup*, and more). Metrics have multiple values on a timeline and tend to change a lot (e.g., *CPU Usage*, *Network Rate*, *Write Latency*, and more). The ability to search for pinboards and static pages (such as the Security Planner page) makes it so that you can navigate entirely by using the search bar, just like on a command-line interface.

The technical natural language that is used is almost precisely like database query syntax. If you have a database administrator (DBA) background (or DBA colleague sitting next to you), the syntax is even more intuitive.

Let's look at some examples:

Find all virtual machines with 4 vCPUs.

`VM where CPU count = 4`

Display a sum of traffic flow for all servers (virtual and physical) in VLAN 2003

`sum(bytes) of flows where source l2 network = VLAN-2003`

Find all virtual machines to whom VMs in VXLAN 5001 are talking to.

`list(dst vm) of flows where src l2 network = vxlan-5001`

Find all applications that are talking to the application 3TierApp02 and display how much traffic is being sent.

`sum(bytes) of flow where Destination Application = '3TierApp02' group by Source Application`

{caption: "Search query & results example"}
![](images/image76.png)

The cool part about the search engine is, is that all results and searches are reusable. You can create pinboards from different searches, personalizing these pinboards completely. You can also create search-based events and get alerts whenever the result of a search changes (get notified whenever firewall rule changes happen) or when there are no results (get notified when something is deleted).

Search is also exposed via the public APIs for automation, making it possible to programmatically execute searches and get back the results for further processing. Considering all data inside Network Insight is available via the search, this means that you can export and process any information for your purposes. Organizations have built automation processes around Network Insight to import data into Security Event & Information Management (SIEM) systems, Security Operation Centers (SOC) get notified on any firewall rule changes and verify them against the change management trackers, service providers pull network bandwidth calculations to use for billing purposes, the list goes on. More information on how to automate with Network Insight and examples in the chapter [Automating Network Insight](#ch-automating).

## Building Searches

vRealize Network Insight provides a structured query with components illustrated in the below diagram. I'll explain the different components in this chapter.

{caption: "Search query structure"}
![](images/image77.jpeg)

Consider the following query:

`VMs where IP Address = '10.10.10.0/24' order by vCPUs`

Let's break it down and align it with the query structure.

- *"VMs"* is the entity type,

- *"where ip address = 10.10.10.0/24"* is the filter clause,

- *"order by vCPUs"* is the order by clause (sorting).

I> Perhaps you noticed that there is a subnet in the filter clause. As a general tip: Network Insight is subnet aware. So, you can search for anything with an IP address in a specific subnet range. In the above example, the results come from all VMs that have an IP address inside 10.10.10.0/24.

## Entity Types

An entity type represents the type of object that we want to search. The entity is the base for the results that come back and is the starting point for the filter. In the above example, VMs is the entity type on which the results are based on. Then it goes on and filters the results based on a property (*IP Address*) that is attached to that base entity.

The entity type can be in singular and plural form; VM or VMs. The entity type is the only mandatory property in a search query. The projection, filter, group by and order by clauses are optional.

Some common entity types are:

- VM (Virtual Machine)
- Host (ESXi Host and other Hosts)
- Datastore
- Vxlan / Vlan
- Pnics (Physical NICs)
- Vnics (VM NICs)
- VRF (Logical Routers)
- Route
- Application
- Flow

A full list of entity types can be found inside Network Insight. Simply search for 'help' or click the link "Learn more about supported properties" that is placed on the bottom of each search (see Figure 74).

{caption: "Search help: find all supported properties", width: "50%"}
![](images/image78.png)

To give you a few more examples of how to spot what the entity type is, the table below lists a few.

{caption: "Search examples, mapping out entity types", column-widths: "70%,30%"}
| Search query                                                | Entity Type |
| :---                                                        | :--- |
|  `Router Interface where IP Address like '192.168.10.*'`   | Router Interface |
|  `Application where Problem = 'Threshold Event'`          | Application |
|  `Datastore where Free Space Percent < 10%`                | Datastore |
|  `Flows where Source Application = CRM`                     | Flows |
|  `Network Rate of Switch Ports order by Network Rate`       | Switch Ports |

**Note**: The last example becomes more apparent when we go through the [Property](#ch-property) explanation.

{id: ch-meta-entity-types}
## Meta Entity Types

The meta entity type can refer to multiple concrete types. For example, VM refers to VMware VM, Azure VM, and AWS EC2 Instance. The same goes for Firewall Rule, which includes NSX Firewall Rules, AWS Firewall Rules, Palo Alto firewall rules, basically any firewall rules that Network Insight discovers. When searching for a meta entity type, the interface shows all included entity types. This allows you to filter on the exact firewall rules types you're searching for.

{caption: "Search for a meta entity type and get all included entity types", width: "40%"}
![](images/image79.png)

Meta types exist to make the experience holistic, transparent to you. The infrastructure engineer deals with a vast amount of different configurations and entity types. It's almost necessary to allow you to search for firewall rule and let the underlaying tool figure out on which actual platform (whether it be NSX, a physical firewall, or the cloud (and then which cloud)) you need to be. The Network Insight team designed the meta entity types to make your life easier.

## Entity Property

Properties on entities are the details you're typically interested in. These properties can be grouped into the following categories:

### Configuration Property

Configuration properties of an entity are something that typically does not change that often. It would need interaction on the infrastructure to change.

Example configuration properties are; CPU Count, Memory, OS, in VMs. Or, Destination Port, Protocol, Source IP, in Flows. These properties tend not to change a lot.

### Reference Property

These are rather interesting. Objects inside Network Insight get correlated to other objects that relate to it, and these related objects get linked to the object in question. To make that a little less vague, a VM is directly related to the host it lives on, the datastore where it stores its data, the network, and port group that it is connected to, and on.

These reference properties appear as links when looking at an object. On the VM dashboard, or when looking at VM search results, you can notice that you can click on a bunch of properties and get taken to their dashboard. Those are reference properties.

You can also use these reference properties in searches, but more on that in [Reference Traversal Queries](#ch-reference-traversal-queries).

### Metric Property

Metrics are properties that do change a lot and have a time-series attached to it. Examples for VMs are CPU Usage, Packet Drop Rate, Write Latency.

To get an idea of what metric properties are available for an entity type, head to its dashboard, and look at the Metrics heading. The screenshot below gives an example of the metric properties on a switch port.

{caption: "Search Metric Properties: example on switch port"}
![](images/image80.png)

### Meta Property

Remember the [Meta Entity Types](#ch-meta-entity-types)? Meta properties are the same thing, only attached to another entity. This is a group of properties that's consolidated under a single property.

For example, the Flow entity has a meta property called VM. This refers to both Source VM and Destination VM. Here's an example:

`Flow where VM = 'my-VM'`

Above search is equivalent to:

`Flow where Source VM = 'my-vm' or Destination VM = 'my-vm'`

Similarly, this search:

`Flow where IP Address = '192.168.10.0/24'`

is equivalent to:

`Flow where Source IP Address = '192.168.10.0/24' or Destination IP Address = '192.168.10.0/24'`

These meta properties make it easier to execute searches and optimize the amount of typing you have to do.

## Filter

A filter clause is used to filter search results. The condition in a filter clause consists of an entity property, comparison operator, and value. This is also where all previously discussed properties can be used to narrow down the results.

Here's an example:

`VMs where CPU Usage > 80%`

In this search query, the VM is the entity type, and the filter is based on the metric property for CPU Usage. By using the comparison operator (**\>**) and value, the returned VM list is filtered on VMs that have a CPU Usage higher than 80%.

There's a lot possible with the filter conditions and comparison operators. Here is a graphical overview of the syntax:

{caption: "Search filters; condition and comparison operators"}
![](images/image81.jpeg)

It is possible to use multiple filter conditions in a single search, and you can mix and match, however you would like. To get a little more feeling with how these operators work and for examples on how they work, see the table below.

{caption: "Search filter condition examples", column-widths: "10%,30%,50%"}
| Operator   | Description                        | Example |
| :---       | :---                               | :--- |
| `AND`        | Multiple conditions need to be met | `VM where Account = 'AWS_xx' and AWS VPC = 'My-VPC'` |
| `OR`         | Single condition should be met     | `Flow where Port = 80 OR Port = 443` |
| `LIKE`       | Wildcard search                    | `VM where name LIKE 'web'` |
| `=`          | Exact search                       | `VM where name = 'web01'` |
| `!=`         | Anything but                       | `VM where name != 'web01'` |
| `<`          | Lower than                         | `Datastore where Free Space Percent < 10%` |
| `<=`         | Lower than or equal to             | `VMs where CPU Count <= 2` |
| `>`          | Higher than                        | `VMs where CPU Usage > 80%` |
| `>=`         | Higher than or equal to            | `VMs where Memory >= 8GB` |
| `IN()`       | Values in this list                | `Flows where Port IN(80, 443)` |
| `NOT IN()`   | Values not in this list            | `Datastore where Filesystem Type NOT IN('NFS', 'VSAN')` |
| `IS SET`     | Value has to be set                | `Flow where Firewall Rule IS SET` |
| `IS NOT SET` | Value has not to be set            | `VM where Security Group IS NOT SET` |

## Projections

A projection clause in a search query decides what fields must be displayed from the filtered entities. By default, the search results list commonly used properties of an object. For example, if you search for VMs, the results list shows the VM name, network it's attached to, disk information, and IP address. You can then either expand the result or go to the VM dashboard to get the rest of the property information (such as CPU, memory, power state, and on).

Using the projection clause, you can bring up a specific property directly in the results. Projections are an optional clause. If the projection clause is not specified, then the default set of properties are shown for entities in the results. Projection clause may contain one or more of the following items:

1. Property
2. Count
3. List
4. Aggregation
5. Series

{id: ch-property}
### Property

If you search for an entity, a default set of properties are shown in the search results. If you want to see other properties (including metric properties), you can use property projection. For example, consider the following search query:

`OS, CPU Cores of VMs where Name = myvm`

The above search query shows the operating system and the number of CPU Cores of the VM named 'myvm', as shown below:

{caption: "Searching with property projection"}
![](images/image82.png)

As you can see, the requested properties are now shown first, in a highlighted fashion.

A projection clause can also include metric properties, combined with configuration properties. This automatically shows a line graph with the requested metric on the results page. If no time window is provided, the metrics are shown for the last 24 hours.

Here's an example:

`CPU Cores, CPU Usage Rate of VMs where Name = myvm`

{caption: "Searching with property projection, including metrics"}
![](images/image83.png)

The above example can be done on any object, whether it be a VM, flow, datastore, switch port, and on. You can also add multiple metrics to the search query, making multiple line graphs appear in the results. A good use case for that would be to search for network rate and packet loss on switch ports.

Using the Network Insight search engine is by far the quickest way to get specific information from your environment, dependent on that you know what you are looking for. This can be a specific object (troubleshooting a switch or VM), or specific metrics.

### Count

Mostly used when building pinboards, the Count projection can return the sum of the number of objects in a search query result. It can be used to count the number of VMs inside a cluster and network flows coming from a specific region, ports or source IP range, number of datastores that have a write latency higher than 5 milliseconds, the list goes on.

Here are a few examples:

`count of VMs where Security Group = 'sg-1' and CPU Usage Rate > 60%`

`count of Flows where Source IP = 10.152.30.208/24`

`count of Switch Port where Total Packet Drops > 100`

`count of Datastore where Write Latency > 5ms`

{caption: "Searching with a count operator"}
![](images/image84.png)

Considering the results of these search queries is a number, these search queries are perfect to put on an automatically refreshing pinboard that is displayed on a monitor that's glued to the wall of your office.

### List

This is another interesting one. The list operator should be used whenever the filter condition cannot be applied or if it is related to the object you want back. To make that a little more concrete, here's an example:

`list(Host) of VMs where CPU Usage Rate > 95%`

{caption: "Searching with a list operator"}
![](images/image85.png)

In this case, a list of Hypervisors is returned that hosts any VM that has a higher CPU usage of 95%.

The object inside the list operator needs to be directly associated with the object in the filter and not indirectly associated. Meaning you can get a list of Hosts from VMs, Datastores from Hosts (and VMs), Switch Ports from Hosts, and more. However, you cannot get a list of Switch Ports from VMs, as the direct association of Switch Ports is with Hosts.

Alternatively, you can use reference queries, which can go more in-depth (and do indirect associations), but also have more overhead. More information on reference queries in the chapter [Reference Traversal Queries](#ch-reference-traversal-queries).

### Aggregate Functions

Just as the Count operator, an aggregate function allows you to calculate a single value from multiple sources. This retrieves *numerical* or *metric* properties and executes a calculation on them. The outcome of that calculation is included in the search results.

I'll go into the supported aggregate functions below.

#### Max

Get the highest value out of the search results. Consider the search returns vCPU numbers for your VMs, the max operator is able to tell you what the highest number of vCPUs on a VM is.

Example:

`max(vCPUs) of VMs`

{caption: "Searching with a max operator", width: "40%"}
![](images/image86.png)

The above result indicates that out of 789 VMs, the VM with the highest number of vCPUs has 16 vCPUs.

#### Sum

Sometimes humans keep it simple and straightforward. These aggregate functions are an example of this. The sum operation, well, returns the sum of the requested values. You can use this to get the number of VMs attached to a VLAN, the amount of memory in use by VMs on a specific host, the amount of traffic going towards the internet (or certain geographical regions), and more. The possibilities seem to be only limited by your imagination.

Example:

`sum(Bytes) of Flow where Source Continent = 'Europe'`

{caption: "Searching with a sum operator", width: "50%"}
![](images/image87.png)

#### Min

The min operator is effectively the same as the max operation; it just gets the minimum value of the search results. Ever wondered which switch has the lowest number of switch ports in use? Ever wondered which blade chassis has the least blades in use? The min operator can help.

#### Avg

Lastly, the avg operator can be used to get an average value from the search results. Just like the previously discussed operators, Network Insight goes through the results of the search query and returns the average of those results. What is the average count of vCPUs or Memory for VMs on a cluster, what is the average transfer bytes of the flows in your VDI cluster, compared to the production server cluster, and on.

Here's an example:

`avg(vCPUs) of VMs group by Host order by avg(vCPUs)`

{caption: "Searching with an avg operator"}
![](images/image88.png)

The above example returns the average count of vCPUs of VMs on each host.

Before I forget, all aggregation operators can also be used in the **group by** clause, as you can see in the above example. When these operators are combined with the **group by** clause, there's an opportunity to **order by** something—more on ordering search results later, but know that the aggregation operators are included.

### Series

The series function is one of the coolest projection functions, by far. Network Insight can show line graphs for any metric that it collects. It can show you custom graphs of network bandwidth usage, CPU usage, memory usage; you name it. The one thing that the out-of-the-box metrics have in common is that those line graphs only show one metric type for a single object in the same graph.

Meaning, there are different line graphs for the CPU, memory, and network usage on the VM dashboard. There is also a different line graph for the network usage per each VM. That is where the series projection comes in.

It can combine metrics from multiple objects into a single line graph. For example, it can combine all network flows coming from VMs in the same cluster, or all network flows going towards the internet, or all CPU usage of all VMs in a specific cluster, or all VDI network traffic, I could go on.

Here's an example, combining all internet traffic into a single line graph:

`series(sum(Byte Rate)) of Flows where Flow Type = 'Destination is Internet'`

{caption: "Search; using the series() projection to combine metrics"}
![](images/image89.png)

The metrics inside the series projection can be any of the metrics available in Network Insight. You can also request multiple series in the same search, to compare different metrics on the same timeline. An example of this is to look for the average used Memory and Network Rate metrics for all servers with 'Web' in their name:

`series(avg(Memory Usage)), series(avg(Network Rate)) of VMs where Name like Web`

{caption: "Search; using multiple series() projections to combine metrics"}
![](images/ch-9/multiple-series.png)

## Ordering

All search results can be sorted by using the **order by** clause. This clause takes one parameter, and results can be ordered descending or ascending (the default is descending).

Ordering becomes handy when you are looking for objects that are using the most resources. A few examples below.

Looking for the workloads that send the most network traffic? `Order by Bytes`

Looking for the VMs with the most vCPUs? `Order by CPU Count`

Looking for the VMs that use the least memory? `Order by Memory Usage asc`

The web interface can also be used to define the **order by** clause by using the Sort selection.

T> When using the web interface to select the sort and filters, the search query that's displayed on the page automatically changes to reflect the selected options. You can then copy and paste the search query into the search bar.

## Grouping

When you're using the search engine to identify trends, you typically want to group objects, to get the right data. For example, you can group network flows by Country, list the number of VMs inside Security Groups, list the amount of bandwidth used by network ports, list the number of VMs on different operating systems (find the 2 VMs that are still running Windows 2003), the list goes on.

The **group by** operator can be used to group entities together when they are grouped by a property. When using this operator, the group name and the number of entities found is returned as a result.

Here's an example that lists the number of VMs in each security group:

`VMs group by Security Group`

{caption: "Search; using the group by operator"}
![](images/image91.png)

It gets more interesting when you start combining grouping with the previously explained Aggregate Functions. By combining these two, you can retrieve the sum, maximum, minimum, or average of a property while grouping it on another property. Let's look at a few examples below.

Retrieving the amount of bandwidth that comes out of each L2 network:

`sum(Bytes) of Flows group by Source L2 Network`

{caption: "Search; using the group by operator and aggregate functions for L2 traffic"}
![](images/image92.png)

Retrieving the number of incoming firewall rules inside your AWS accounts, grouping by VPC. This displays the number of rules and the number of security groups inside each AWS VPC:

`sum(Incoming Rule Count) of AWS Security Group group by AWS VPC`

{caption: "Search; using the group by operator and aggregate functions for AWS rules"}
![](images/image93.png)

## Limiting

When you order a search, sometimes you're only interested in the top 10 flows. Or the top 5 VMs that are using the most storage, CPU, or memory. To limit the search results by a specific number of results, simply add the **limit** operator. For example:

`VMs order by CPU Cores limit 5`

`Flows where Flow Type = 'Destination is Internet' order by Bytes limit 10`

{id: ch-reference-traversal-queries}
## Reference Traversal Queries

Let's say you want to filter the search results based on a property that is not directly attached to the entity you're searching for. An example of this would be to filter on the CPU usage of the Host while searching for VMs. In this case, reference traversal queries can be used to refer to the property that you want to filter.

To refer to the properties of a referenced object, use a DOT(.) operator with the primary property. Behind the 'parent' property, the child properties can be used normally. Think of this traversal as going a folder deeper into the entities.

Let's look at a few examples to give you a better understanding of how this looks.

Find all VMs on hosts with a high CPU utilization:

`VMs where Host.CPU Usage Rate > 95%`

Find all VMs on a specific compute blade:

`VMs where Host.Blade = '[ucs-1.vrni.cmbu.local]-[sys/chassis-1/blade-7]'`

Find all VMs that are located on a cluster that does not have the NSX Distributed Firewall enabled:

`VMs where Cluster.Firewall Feature Enabled = false`

Take note of the DOT (.) in between the entity (Host or Cluster) that is directly attached to the entity that is being searched on (VM) and the property that is being filtered on. Because Network Insight only correlates and stores properties that are directly related to a specific entity, the VM entity does not have the CPU Usage Rate of the Host that it is on. The Host can change with a vMotion, so it doesn't make sense to relate to the Host CPU usage directly. However, using these reference traversal queries, you can still filter based on the properties of the Host (or Cluster, or anything else that comes up in the VM filters).

I> Reference Traversal Queries are heavy on the database. Just like joined queries in relational databases, these queries cause Network Insight to traverse through multiple database tables and compare records. Try to avoid using them often, and don't be surprised if they take a few seconds.

## Nested Queries

As an alternative to reference traversal queries, nested queries are also possible. The results of a nested query can be used to filter in the original query. Nested queries are executed first, meaning you'll have the results available to filter on the main query. While this is similar to reference traversal queries, it's written differently.

To make this tangible, let's have a look at the examples of the previous chapter, only in nested query form:

Find all VMs on hosts with a high CPU utilization:

`VMs where Host in (Host where CPU Usage Rate > 95%)`

Find all VMs on a specific compute blade:

`VMs where Host in (Host where Blade = '[ucs-1.vrni.cmbu.local]-[sys/chassis-1/blade-7]')`

Find all VMs that are located on a cluster that does not have the NSX Distributed Firewall enabled:

`VMs where Cluster in (Cluster where Firewall Feature Enabled = false)`

As you can see, it's a bit more text than using reference traversal queries, but to me, this is more readable.

## Time Control

You can use the time control to change the time point of the results. By default, searches involving configuration returns the last known configuration ('now'), and metrics are returned for the last 24 hours.

For example, *bytes of flow* shows traffic for flows over a period of 24 hours, and *firewall rule* returns the firewall rules that currently exist. Time control can be used to return configuration properties at a specific point in time or to modify the time span of returned metrics.

You can change the time span using the time indicator that is on the right of the search query, inside the web interface. It can also be done from inside the search query itself.

{caption: "Search; time control in the web interface", id: "fig-time-control", width: "50%"}
![](images/image94.png)

As you can see in the [figure above](#fig-time-control), it's easy to jump from the current time to some most used points, such as **Yesterday**, **Last 3 Days**, and on. You can also use the **At** option to jump to a particular time, used for getting to know the exact configuration at that time. Then there's the **Between** option, where you can specify a time range, mostly used for metrics to show graphs of a specific time period.

You can also specify the time control inside a search query by using the following syntax:

`*search query* in last X (minutes|hours|days|weeks|months)`

Here are two examples of a time-controlled search:

`Flow in last 48 hours`

`changes in last 3 hours`

The above example uses searches that return results (metrics or change events) in a time range. You can do something similar to search for configuration of a specific time in point, like this:

`Firewall Rules at November 5 14:00`

The above example returns all firewall rules that existed on November 5^th^ at 14:00 (sorry (not sorry), it does not take pm/am times, have to provide the 24-hours format. ;-) ).

When you start combining configuration and metric results while using time control, it picks the highest value of time. For some examples, see below.

`VMs where CPU Usage > 80%`

The result of the above query, are all the VMs that have had a CPU usage higher than 80% in the last 24 hours.

`VMs where CPU Count = 4`

The result of the above query shows all the VMs that currently have a CPU count of 4.

`VMs where CPU Count = 4 and CPU Usage > 80%`

The result of the above query is all the VMs that have had CPU usage higher than 80% and where the CPU count 4 in the last 24 hours.
