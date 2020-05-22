{id: ch-analytics}
# Analytics

Almost everything about Network Insight has something to do with analytics in some form. From correlating flows to infrastructure objects to creating network topologies from the available network devices, it's all analyzing the data that's coming in. I like to call these types of analytics, 'passive' analytics. Mostly because you need to go to a page, dashboard, or do a search query to look at the data, and you would be looking at something specific, like troubleshooting issues between 2 VMs or planning out security for a specific application.

Then there are the 'active' analytics, which actively starts monitoring a set of VMs, containers, physical servers, or any other type of network flows; and triggers an alert if something happens. There are 3 kinds of these active analytics inside Network Insight and here's a brief overview:

**Outliers**
: Detect whether a group of VMs, Containers, or physical IP addresses that are supposed to behave the same way, are not behaving the same way. Think of a group of web servers that are behind a load balancer, serving up the same web application. If everything is operating normally, each web server should have the same load. When an outlier is detected, for instance, when 1 of the web servers goes berserk and starts sending out tons of network traffic, you get alerted.

**Thresholds (Static)**
: These are pretty straightforward; you can get an alert when a certain threshold is breached. Because the thresholds are network focused, triggers can be network traffic rates, network packet counts, and network packet drops (packet loss). For example, set an alert for when an application starts showing more than 2% of packet loss or for when an entire tenant or department (all their VMs combined) starts receiving or sending out more than 100Mbps of network traffic.

**Thresholds (Dynamic)**
: Building on the static thresholds, but cooler. These dynamic thresholds take the same approach, but with a slight difference in configuration. You don't set an actual threshold; Network Insight determines the thresholds based on past behavior. Just like static thresholds, these are available on network traffic rate, network packet counts, and drops.

**Top Talkers**
: This one is a bit of an oddball in my self-proclaimed 'active' analytics, as it does not send out alerts. It does show a bunch of actively analyzed data, such as top talking objects, new internet services accessed (in the last 24 hours), new VMs talking to the internet, new services exposed to the internet, and more.

Let's get into the particulars of each of these.

## Outlier Detection

The Cambridge dictionary defines outliers as the following:

> *"a fact, figure, piece of data, etc. that is very different from all the others in a set and does not seem to fit the same pattern."* [^8]

[^8]: <https://dictionary.cambridge.org/dictionary/english/outlier>

That is precisely how it defined inside Network Insight. Using self-defined groups of similar VMs, Kubernetes Pods or physical servers, Network Insight starts to monitor them and raise alerts whenever 1 of breaks the pattern that all of them have. As we grow into a more distributed and microservice types of applications and infrastructures, more and more workloads only have 1 job, and it is possible to predict its behavior and say that it is always supposed to behave the same way as its brethren.

Traditional environments are exclude, though, as those have servers with the same functionality as well; think of DNS servers, Active Directory controllers, SQL Clusters, and more. Depending on how many roles these more traditional servers have (can't have too many different roles. Otherwise, behavior would be too different), it also adds value to monitor these.

You start by configuring an outlier configuration that has all the aspects of the group of workloads that is monitored. You also need to select the metrics that the monitoring happens on. In the example below, I'll create an outlier group for 5 DNS servers. First, start by giving it a name (1). In this case, **DNS Servers**.

{caption: "Outlier configuration options"}
![](images/image72.png)

Grouping (2 & 3) is done based on application tiers or security groups. The security groups can be NSX-v, NSX-T, VMware Cloud on AWS, or native AWS security groups, anything that has the security group concept. While security groups can be used to indicate tiers in an application, they are typically used for wildly different purposes as well. Security groups can be used to segment tenants or departments from each other, or segmentation between different applications, and so on. That's why I prefer the use of application tiers for outlier groups. Your application constructs should already exist inside Network Insight for the application dashboard, security planning, and more.

I> Make sure a group with a minimum of 3 VMs or physical IP addresses is selected, as that is the minimum for an outlier group. It's not possible to have good results with outlier detection on just 2 members.

Outlier detection uses the data from network flows, where traffic rate, number of packets, or the number of flow sessions can be used to base the outliers on (4). Take note that the metrics on **Total Flow Traffic** and **Network Traffic Rate** are based on the same data. The only difference is that total flow traffic is the total amount of bytes, and the network traffic rate is the average (basically, the bandwidth graph) traffic rate. The other options are self-explanatory.

You can decide which direction (5) should be monitored: incoming, outgoing, or both. The reason for picking a single direction could be simple:

- You don't care about incoming traffic and only want to pick up outgoing traffic (to detect outgoing denial of service attacks).
- The outgoing traffic is too random to predict (maybe the case with streaming or download servers), and only the incoming traffic is predictable.

If you're going in a specific direction, you usually know why. Most of the time, selecting both directions is the way to go.

Apart from the traffic direction, the type of traffic (6) is also an option. Only monitoring East-West traffic, only internet traffic, or all traffic can be selected. This is useful to limit the monitoring on only internet traffic, and with that limit the load on the Network Insight platform, if you fully trust and don't care about the East-West traffic. Another reason to choose the traffic type is if the East-West traffic is predictable and flows between the different application tiers and if the internet traffic is unpredictable.

At number 7, we have the option to limit outlier detection on a specific network port. By default, every network port that is detected coming in and going out from the members is used for detection. But if you're only interested in web traffic or another application that uses a specific port, you're free to select these. Fun fact: the configuration page detects which network ports are in use for the group you've selected, and it shows them.

I> Another reason for selecting the ports, is that an outlier group currently supports a maximum of 20 ports to monitor simultaneously. A warning triggers when there are over 20 ports detected when 'All Ports'

Lastly, the sensitivity (8) can be selected: low, medium, or high. The high sensitivity triggers more quickly, and the low sensitivity triggers less quickly.

Changing the sensitivity setting affects the outlier detection by changing something called the z-score for the median absolute deviation (MAD) algorithm [^9] that is used to detect outliers. The median absolute deviation is a mathematic process to get the median (average) value of a range of numbers. This process is applied to the metric numbers you've selected to monitor, such as network traffic.

[^9]: Median Absolute Deviation: <https://en.wikipedia.org/wiki/Median_absolute_deviation>

As a straightforward example, consider this number range: 1, 3, 4, 6, 10. The median value is 4, not only because it's in the middle, but also because it's the closest to the average of the sum of this number range (4.8). Then the z-score comes in to determine what percentage above and below the median is considered to be an outlier.

Low sensitivity means that the median has a **60%** buffer. So, take the previous number range and get the median, which is 4. Then apply a 60% buffer below and above that 4. Meaning the lower boundary is **1.6**, and the high boundary is **6.4**. In that number range, numbers **1** and **10** would be outliers as they fall out of the buffer.

Medium sensitivity has a buffer of **40%**. On the same number range and the same median, 4, the lower boundary is 2.4, and the higher boundary is 5.6. Meaning 1, 6, and 10 are classified as outliers.

High sensitivity has a buffer of **30%**. The same exercise on the number range; the lower boundary is 2.8, and the higher boundary is 5.2. Numbers 1, 6, and 10 are classified as outliers, the same as the medium sensitivity, in this case.

And this concludes our brief maths class. ;-)

When everything is filled out, you get a preview of the outlier graphs. The graph uses historical data to show a graphical representation of the selected metric (network rate, packets or flows), which allows you to preview the result, see any current outliers that are already detected and save it to the system when you're happy with the configuration.

Once saved, the outlier configuration is available under the Outlier pages, and it starts monitoring and generating events when outliers are detected. Jumping to a specific outlier group is accessible by doing this search query: `Analytics Outlier Configuration 'DNS Servers'`

{caption: "Outlier detection result"}
![](images/image73.png)

Above is the result of the DNS Server outlier detection. It's clear that two of the DNS servers (**cmbu-sc2dc-01** and **cmbu-wdcdc-01**) are behaving entirely different than the others. That is why they are marked as an outlier.

Once the outlier configuration is in place, Network Insight starts to generate events (alerts) whenever an outlier is detected. The cool part about these events that the evidence graph is included:

{caption: "Outlier detection event"}
![](images/image74.png)

Getting alerts from the outliers is a bit trickier than expected. In order to get either emails or SNMP traps when an outlier is detected, you have to set up a user-defined alert from the **outlier events** search query. Do so using the alarm bell icon at the top right, and you can choose between getting emails, and SNMP traps when there's a new outlier detected. Out-of-the-box alerts for outliers are currently not there, so this is a work-around.

## Static Thresholds

Thresholds are relatively straightforward: set a boundary as a threshold limit, and as soon as something goes over it -- trigger an alert.

Network Insight thresholds are the same way. Based on networking metrics such as traffic rate, number of packets and packet drops, a boundary can be set, and alerts are sent out when a workload (or group of workloads) crosses that boundary. Static thresholds are really that simple. Dynamic thresholds have a bit more bite to them, which is why I'll go into those in the next chapter. But for now, let's focus on what you can do with static thresholds.

Let's say you've got a workload or group of workloads that should not over a network traffic rate of 1Gbit per second. This can be a tenant, department, or just a specific web application that's running on a distributed cluster, or even a streaming application that's running on 1 VM. If it hits that 1Gbit, you want to be notified.

Another example is when a workload should always generate traffic, like a storage or database replication, or an Active Directory cluster, even DNS servers should typically always generate traffic. In this case, when the traffic flatlines or gets under a set boundary, you want to be notified.

And as the last example, incoming internet traffic can be monitored in a threshold. If your data center never has more incoming traffic than 1Gbit per second and it suddenly receives 5Gbit per second from the internet, there's a chance the network is under a denial of service attack. When this happens, you would like to be notified about it.

For these use cases, thresholds can be configured. Thresholds can be found under the Analytics menu, or you can simply type **threshold configuration** into the search bar.

The main page of the thresholds lists all configured thresholds and the related events (threshold breaches) along with options to modify and remove them. Let's go through the creation of a threshold.

{caption: "Creating a threshold"}
![](images/image75.png)

A lot is going on on that page; let's take it step-by-step. Your attention in the screenshot will probably go straight to the preview (5). But to get there, first fill out a name for the threshold (1) -- something to recognize it by when it triggers an event, and an alert is sent out.

Next, thresholds can be based on a few objects. The scope can be set to either **Virtual Machines**. This opens up a search definition, and you can dynamically scope any VM with certain characteristics, such as being in a specific vCenter folder, having a name prefix, residing in an AWS VPC, or with an X number of vCPUs, and more. You can also combine conditions, for example, all VMs in a specified vCenter folder and a specific network.

Besides the VM search, the scope can also be set to Flows. This allows you to search for flows, just as with the VM option. Meaning you can search for flows with a specific destination port (for example, all traffic going to DNS servers), destination country, incoming traffic from the internet, outgoing traffic to the internet, you name it. Combining conditions is also possible with the flow search, for example, incoming internet traffic coming from a specific country.

Lastly, the scope can also be set to an **Application**. These are the applications that explained in the chapter [Application Constructs]{#application-constructs}.

I> Typically, the Application scope setting is used to monitor workloads running specific applications. The Flow scope setting is mostly used in cases where specific network traffic has to be monitored (like internet traffic or inter-data center traffic).

While selecting the scope, there's a handy reference (2) to see how many VMs and Physical IP addresses are found in this scope. Physical IPs can show up in applications and or flow searches; VMs show up in any of the scope options (if applicable, of course).

When the scope is set, move on to the condition (3). This condition is written in a sentence format with a bunch of options embedded; that's to understand easier what you're configuring.

Here's the default:

{format: console, line-numbers: false}
```
For metric network traffic rate aggregated over virtual machine when any value exceeds threshold
Upper bound: ZZZ Kbps
```

This text translates to that by default, it looks at the network traffic rate on an individual VM basis and generates alerts when it reaches more than ZZZ Kbit per second. The default is the setting that is most used, but there's a world of options here.

### Monitored Metric

First, the metric option can be set to 6 different metrics. I'll explain each of these in the sections below.

#### network traffic rate
This is the network traffic rate in Kbit per second. This is the last real-time value in the bandwidth graphs of a workload.

#### max network traffic rate
Network traffic rate in Kbit per second, aggregated over 24 hours, and this maximum value represents the peak of that traffic.

#### total network traffic
Network traffic rate in GB aggregated over 24 hours, and this value represents going over that amount of traffic in a day.

#### network packet count
The number of packets. This can be useful to detect denial of service attacks that are based on a lot of smaller packets. The packet-count would go up, but not necessarily the bandwidth itself (these smaller packets can be 2-5% of regular traffic).

#### network packet drop
The number of packets that are dropped. There are mechanisms in place (TCP has this built-in) to recover from dropped packets, and it's pretty standard for packet drops to occur. Especially over the internet. Don't put the value at 1 -- you'll go crazy with alerts.

#### network packet drop percent
The number of packets that are dropped in percentage form. This is a better option when looking at packet drops. When network traffic goes up, the packet drops do as well; they usually scale up with the amount of traffic. Typically, packet drops over 5% impacts traffic significantly, and between 1 and 2.5% is 'acceptable'. [^packetloss-reference]

[^packetloss-reference]: https://en.wikipedia.org/wiki/Packet_loss

### Aggregation

The next option, **aggregated over**, determines at what level the threshold is constructed. If there's a group of workloads selected, you can decide to put the threshold over this entire group (option **entire scope**) or to put the threshold on each member. For example, if you're creating a threshold for a tenant/department, which is not allowed to go over 1Gbit per second, select the **entire scope** option. If you want to get alerts when any VM inside a bigger group goes over 500Mbit per second (no matter what the rest of the VMs in the group do); select **virtual machine** as the aggregation to individualize the monitoring.

I should note that the options in the aggregation option change per scope type. If **Application** or **Virtual Machines** is selected, you can aggregate on the entire scope or per virtual machine. When the option **Flows** is selected, the aggregation option changes to **source VM/IP**, **destination VM/IP**, **service endpoint**, and **entire scope**. The first two and last options are self-explanatory. A service endpoint is a service, like a webserver running on a workload. For example, VM webserver-01 port 80 and VM webserver-01 port 443 are two different service endpoints.

So, with the Flow scope option, there is more choice to individualize the monitoring, which makes sense, because flows have more context into the traffic and can separate service ports that run on the same workload.

{id: ch-threshold-breach-value}
### Threshold Breach Value

Let's move on to the options that determine when the threshold is breached. First, the **when** option determines whether it needs to watch the latest real-time value (option **any value**), or **the** **average** during a specified time window.

If you select **any value**, it simply monitors the traffic and creates an alert when it crosses the threshold. If there are many spikes in the network, this could lead to false positives. That's why there's also **the average** option. The average option takes the last **30** or **60** minutes into account when monitoring, grab the average of that time, and use that average as the threshold. This smoothens out the spikes and reduces alerts.

I> When selecting **any value** (and not the average of 30/60 minutes), Network Insight takes the average of the maximum value of the last 5 minutes to trigger on. To be specific, **max network traffic** rate and **total network traffic** take the max value of the last 5 minutes. The rest of the metrics take the average value of the last 5 minutes.

Then, select whether you want to create the alert when the metric **exceeds** the threshold upper bound, or if it **drops below** the lower bound, or **is outside the range** of a lower and upper bound, or **deviates from past behavior**. The upper and lower bounds are something you'll be able to configure at the bottom of the condition, and the fields change depending on which option is selected. When **deviates from past behavior** is selected, the threshold becomes dynamic, and Network Insight figures out the correct threshold values. More on that later.

### Alerting

Let's not forget to set an alert based on the threshold you're creating. Whenever a threshold is breached, an event is created. This event is visible in the open problems list, and you can search for all threshold events using this search query:

`events where name = 'AnalyticsThresholdEvent'`

If the breach showing up in the web interface and you don't want an email or SNMP Trap when it gets breached, just select a severity (importance), and you're good to go and save the threshold. It starts monitoring and lets you know in the interface when it breaches. This is fine for not-that-important workloads and breaches that you don't need to know right away. Always determine this on a per case basis, as you definitely don't want to overcrowd the email or SNMP Trap notifications. If there are too many, people stop paying attention.

When this is one of those important thresholds, and someone needs to be notified on the moment it gets breached, configure multiple notifications options. Configure 1 or more email addresses to notify and tick the checkbox **Send SNMP Trap** to have Network Insight pass the event to another system and takes care of the actual notifications.

Now you're really done; go ahead and save the threshold to the system to activate it.

## Dynamic Thresholds

As announced, here's a bit more information on what happens when you select **deviates from past behavior** as the threshold breach trigger. This is where the system learns a baseline of network behavior of a workload or group of workloads and uses the baseline of that past behavior to determine whether it is doing something that is out of character. For example, if the workload does a steady stream of 10Mbit per second regularly and then a spike of 100Mbit per second happens, it is qualified as a threshold breach.

The other way around also triggers a threshold breach; if the workload does 10Mbit per second regularly, and it suddenly drops to 1Mbit per second, a threshold breach is raised.

Without setting a static upper or lower boundary value, Network Insight simply notifies you when workloads divert from their routine.

I know I said the mathematics class was over, but I do want to give you some background on how this works. Bear with me.

It does this by creating a baseline of network traffic behavior, for a minimum of 7 days and a maximum of 21 days. Before that 7 days of history is available, Network Insight keeps analyzing the network traffic. However, it will not yet generate any alerts based on suspected threshold breaches. Data that is older than 21 days is aged out of the dataset and not be used for the analytics. This is typically enough time to get a clear view of the intended behavior of a workload and alert when it goes out of bounds.

The baseline data is a so-called moving mean[^11] and standard deviation[^12] dataset. The mean (average) is calculated over the network traffic of the last 21 days. This mean value is then used as the benchmark to calculate the standard deviation. In somewhat plain English, the standard deviation is a measurement that quantifies the amount of variation within a dataset. There are examples in the Wikipedia article I linked in the footnote, but it gives a numbered value to compare a value with the average.

[^11]: Moving Mean: <https://en.wikipedia.org/wiki/Mean>
[^12]: Standard Deviation: <https://en.wikipedia.org/wiki/Standard_deviation>

Dynamic thresholds have the option to be configured with a **high**, **medium**, and **low** sensitivity. The **high** sensitivity uses a standard deviation value of **2**, meaning if the current network traffic value is **2** times as much or less than the average of the last 21 days, the threshold is triggered. The **medium** sensitivity uses the standard deviation value of **2.5**, meaning **2.5** times as much or less than the average of the last 21 days. The **low** sensitivity uses a standard deviation value of **3**; meaning, well, you get it.

To make it a little more complicated, you can also select whether to use the average of the last 30 or 60 minutes or use the maximum or average of the last 5 minutes. This is the same concept as we saw in the INFO a few pages above, in the chapter [Threshold Breach Value](#ch-threshold-breach-value).
