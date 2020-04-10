**Abstract**

If you are in any way affiliated with network and virtual infrastructure troubleshooting and monitoring, you're going to like this book. If you're driven crazy by having to overlap multiple tools to get the information needed to troubleshoot an issue in your environment, you're going to love this book.

As the title suggests, this book will give you a deep dive look at VMware's holistic networking and virtual infrastructure monitoring & troubleshooting product; vRealize Network Insight.

Network Insight is famous for a couple reasons:

1.  It has the capability to kickstart your way to a secure segmented network by creating visibility into what actually happens on your network.

2.  It sucks up all the operational and configuration data of your virtual (vSphere, Kubernetes, Bare metal), public cloud (AWS & Azure), SD-WAN, and physical network environment and allows you to overlay the two to create a holistic view of your entire environment and use that to significantly reduce time spent troubleshooting and gathering data from your environment.

3.  Analytics that go over all the data that's pulled into Network Insight, provide a great deal of useful information; from "who is using my applications?", to "are there any abnormalities in the network behavior?", to all the way to "how much is this cloud egress bandwidth costing me?".

You will be guided through the components and architecture of Network Insight and discover hidden gems and secrets throughout the platform. This book will take you from beginner to a vRealize Network Insight Samurai and we'll have some fun along the way!



Introduction -- About the author
================================

As I'm sitting in the airport, waiting for my flight after a team building exercise with the Networking & Security Business Unit within VMware, the thought going through my head is: why? Why on earth are you starting another big project that's going to take up a ton of time? The reason is simple: Hi, I'm Martijn Smit and I'm an information sharing addict (echo's: "hi Martijn!").

I'm proud to be Dutch and proud to be in the industry I'm in. My career started at a hosting company, in which I spent 8 years moving from the webhosting help desk to colocation & dedicated server support, to managing the internet-network and helping to build a new datacenter from the ground up. After leaving this provider, I joined a value-added reseller company, with which I spent 5 years designing and deploying data center infrastructures, including storage, compute, networking and virtualization. I was also in the innovation group that brought the last two together to support virtual networking; Software-Defined Networking (SDN).

I currently work at VMware in Technical Marketing for vRealize Network Insight, as I fully believe in the way Network Insight is approaching Networking & Security troubleshooting and monitoring and want to spread the word. My previous role was to guide customers to the world of virtual networking; NSBU Solutions Engineer. Having said that; don't worry, this book is not going to be a sales pitch but hardcore technical information (just as I like it. ;-)).

Apart from my passion for technology, I believe in a healthy balance and taking care of yourself, so often take 2 to 3-hour bike rides, exercise every day and eat proper food. One of my favorite hobbies is spending time on doing elaborate meals using my Kamado Egg barbecue. I also tear through books and have a goal to read at least 30 books yearly. As mentioned before, I'm an information sharing addict and mostly do so by giving talks and doing technical blogs.

If you would like to see more of my rants, you can follow me on twitter on [\@smitmartijn](https://twitter.com/smitmartijn) or on my blog at <https://lostdomain.org>

Back to this book; so, I have been dealing with the entire data center stack for years. One of the things that always annoyed me was that there was no good solution to troubleshoot and monitor that entire stack; it was always separate solutions for each layer. In 2014, I came across a company called ArkinNet, which had an amazing product which could collect data from multiple layers (storage, compute and networking) and present this data in a holistic way. After doing some due diligence, I immediately started talks to include Arkin into our product portfolio and I've been in love with the product ever since.

Arkin was later acquired in June of 2016 by VMware and it is now known as vRealize Network Insight.

Let's dive in!

P.S: I've written this book on personal title, not as a VMware employee. Any opinions in here are my own and not per se the opinion of VMware.

Syntax(?) to-do
===============

Underlines = links

search query = which can be placed in the Network Insight search engine

Foreword by Shiv Agarwal
========================

*Founder of ArkinNet, currently Vice-President of Network Insight with VMware*

VMware vRealize Network Insight (or vRNI, or Network Insight) has seen a massive adoption in VMware customer base helping our customers get end-to-end visibility and operational simplicity as they embrace a software defined approach to networking and security. Network Insight completes VMware's Virtual Cloud Network vision and story by providing seamlessly visibility and converged network operations across the data center (virtual and physical) and hybrid cloud as well as branch offices and remote sites (via SD-WAN integration).

Jogging down the memory lane, Network Insight came into VMware through the Arkin (ArkinNet) acquisition. As it happens so often in Silicon Valley, my co-founder and I were at VMware before we went out and started Arkin (in 2013). We had joined VMware (in 2008) as part of the Blue Lane acquisition. At Blue Lane, we had built a virtual firewall which became the first-generation virtual firewall (VMware NSX DFW) inside VMware. During our first tenure at VMware (2008-2013), NSX was in its infancy. We saw enterprise customers struggling to operate their virtual networking stack. They were trying to use their existing legacy processes and toolset. Their people's mindset was geared and tuned to managing physical networks. Virtualization was new to the network operators. That's when my cofounder and I got the idea of starting Arkin. You start a company with a big vision, ours was to transform how networks are operated. Idea was to bring consumer grade simplicity to managing networks. We wanted to challenge the status quo. Our first set of use cases was to help customers implement micro-segmentation and operationalize NSX. NSX was becoming the dominant network virtualization stack and we betted on it. We got the first product out in 18 months with some of the marquee NSX customers using it in production and were acquired by VMware in 36 months. At VMware, it was like a match made in heaven. Thanks to the NSX sales team, the two products together (NSX and Network Insight) started flying off the shelf! It's been fun! I tell my team often that the acquisition by VMware was a mere pit stop in our journey, which, at the time of writing this foreword, is still continuing.

We continue to build. Network Insights expanded charter and scope now includes end-to-end network operations - monitoring, troubleshooting and optimization. By combining the different types of network data (flows, packets, metrics, config, streaming, etc.), we have provided a unique platform for our customers to converge their traditionally silo-ed visibility and realize a multitude of use cases around next generation networking and security. We have also created a unique advantage for ourselves by adding a strong application context to network and security dataset. Applications are the lifeline of an enterprise and Network Insight's powerful application discovery and planning feature enables our customers to see their network and security data through the lens of their applications. We are thus elevating IT and empowering them to have a more business-oriented conversation with their line of businesses.

Over the next few years, we see the operational silos breaking at a rapid pace and a lot of automation happening, ultimately leading to self-driving networks. That's the future. Silos create inefficiency and finger pointing. Our vision is to bring a high degree of efficiency in network operations through convergence, consumer grade experience and analytical insights. We continue to deliver upon our vision by investing in new areas. Recently, we acquired a company, Veriflow, which has pioneered the area of network verification in software. This technique is used in many mission critical industries where failure can be catastrophic such as airlines and space. Networking is mission critical for our customers. With this acquisition, we will be arming our customers with network modeling and prediction and significantly push the frontier of network operations in the enterprises.

I am very happy and excited to be writing this foreword for Martijns book on VMware vRealize Network Insight. Martijn has been the technical face and flag bearer of Network Insight in the EMEA region for a long time. I hope the insights captured in his book will trigger in the mind of its readers a genuine thought about transforming their network and security operations.

Pre-face
========

This book is for people in jobs or interests related to networking and security in private, hybrid and/or public clouds. Managing these networks and security policies becomes a much easier job with Network Insight and this book will try to explain best how to go about managing those networks and how Network Insight itself is positioned to do so.

Sometimes it's not all in the name. This is also true for Network Insight, as it gives you not just insight into your network, but your compute, storage and network layers.

With Network Insight, you can take the guesswork out of deploying micro-segmentation with comprehensive network flow analytics to map out real-time traffic and model security groups and firewall rules to successfully implement micro-segmentation security policies. It also helps to improve performance and availability of the infrastructure by combining and correlating virtual and physical compute, storage and networking components to provide a clear and full picture of the infrastructure.

It does not discriminate between virtual machines or physical servers, provides detailed information about the smallest workloads (containers), has integrations with the VMware Virtual Cloud Network vision and everything that runs beneath the Virtual Cloud Network.

Network Insight collects data from data sources like VMware vSphere, VMware NSX, Physical network devices (switches, routers, load balancers, and firewalls), Physical converged systems, IPAM systems and log collectors. All this information is put in a structured database, correlated and available via the intuitive user interface and API. The way this converged information is disclosed with the user interface is what makes Network Insight unique and such a pleasure to work with.

It\'s all about the fundamentals of the platform, as it's designed from the ground up to be as open as possible. This means you can retrieve any and all the data that is gathered and do all kinds of neat things with it like filtering, grouping, sorting and perform other modifiers on it (more on that in the chapter []{.underline}

[\
Using the Search Engine]{.underline}).

Apart from configuration and operational data, you can also send real-time network flow (NetFlow or sFlow) data to Network Insight to map out which workloads in your environment talk to each other. Because all data is correlated, the network flow data is linked to the source or destination workload (virtual machine or physical host) and you can see the name of the workload related to the flow, instead of just seeing that **10.0.0.10** talks to **10.0.1.11** over port **80**.


{class: information}
B> **Configuration data** is meant as the configuration of the data source (i.e. show running-configuration on a physical Cisco device and the inventory of a VMware vCenter, etc.). **Operational data** is meant as dynamic, changing data on data sources (i.e. the route and mac tables on a network device, IP addresses of virtual machines, etc.).


{class: discussion}
B> This is a discussion blurb.

{class: error}
B> This is an error blurb.

{class: information}
B> This is an information blurb.

{class: question}
B> This is a question blurb.

{class: tip}
B> This is a tip blurb.

{class: warning}
B> This is a warning blurb.

{class: exercise}
B> This is an exercise blurb.

{class: tip}
W> This is a tip blurb, not a warning blurb.

A> This is a short aside.


{aside}
# A Note About Asides

This is a longer aside.

It can have multiple paragraphs.

Asides can also have headings, like this one does.

Multi-paragraph asides are more pleasant using this syntax.
{/aside}


This network flow data is typically generated by the vSphere Distributed Switch or a physical network device.

Due to the technical and sometimes very specific nature of this book, it's advised to have a Network Insight instance ready to go while you are reading; so, you can try things out with data from your own infrastructure!

The content of this book is based on Network Insight 5.0 (with some small nuggets on 5.1, because I took too long to write it). Considering the product team is an innovation engine and moves really quickly (delivers major features every 3 months), you need to doublecheck the details when you're using a newer version. This book also does not intend to replace the [official documentation](https://docs.vmware.com/en/VMware-vRealize-Network-Insight/index.html), but rather complement it. The specific technical details in this book will age, and rightly so.









