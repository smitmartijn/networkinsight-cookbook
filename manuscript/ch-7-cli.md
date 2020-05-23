{id: ch-cli}
# Using the Command Line and Troubleshooting

When Network Insight is set up and acting normally, you do not need the command-line interface. Apart from the initial **setup** command, the command line is not used much. It's mostly used to troubleshoot the Platform or Collector appliances themselves and change settings, such as the networking configuration. There are a few commands and other tricks to call out, which I'll do in this chapter.

## Logging In

During the deployment of Network Insight, specifically during the setup command, you are prompted to configure passwords for 2 users: **support** and **consoleuser**. You are able to log into the appliances (both the Platforms and Collectors) using both of these usernames, but they serve different purposes.

The username **consoleuser** is the one that you would typically use. It has a lovely shell wrapper called the vRealize Network Insight Command Line Interface (vRNICLI: yes, it needs a cooler name), and managing a Network Insight appliance should go through this user and shell wrapper. It has a lot of sanity checks (almost; there are danger zones) that makes sure that you can't damage the appliance.

Then there's the support username. This one brings you straight to the underlying Ubuntu operating system and even allows you to switch to root by using `sudo su -`. Don't use this user (or root access) unless directly so by VMware support or if you know what you're doing. The potential damage is catastrophic if the wrong file is modified or the wrong command executed.

I'll be focusing on the vRNI CLI functionality, which has a fair amount of commands:

{caption: "Command line list of commands"}
![](images/ch-7/command-list.png)

While the same commands exist on both of the appliances, some specific commands only apply to the Collector. Run those on the Platform, and it merely outputs that these only work on the Collector. For now, let's focus on the commands that work on both the Platform and the Collector.

## Commands for Troubleshooting

When something's wrong, it's DNS. Or NTP. Always.

But just to be sure all bases are covered, here are a few steps to troubleshoot a Network Insight appliance.

If the Network Insight appliance is misbehaving, the web interface is not responding, and connections are timing out, the Platform is not seeing a Collector and is saying it's offline, any issues that have to do with connectivity or reachability, here's where you start.

{caption: "Restarting services via CLI", width: "50%"}
![](images/ch-7/restarting-services.png)

Start with ```show-service-status```, which gives you an overview of all critical services and their respective status. If a service is not running, you can get more information on the reason why it's not running by running ```show-service-status --debug```. This command reaches out to the service and does a few extra checks on the service and its log files.

Attempt to correct a not running service, by using ```services start <service-name>``` and see if the service stays online. If it does not, move on to checking the networking services of the appliance:

{format: console, line-numbers: false}
```
(cli) show-connectivity-status
Platform VM
Deployment UUID: U66cXXdXX-7694-47dc-a2ee-63XXdXXaXXab
Deployment ID: DP85XXX
Instance ID: IP85XXX
Created On: Wed Feb 21 12:28:41 UTC 2018 (1519216121)
IP Address: 10.196.164.25
Netmask: 255.255.255.0
Gateway: 10.196.164.253
DNS nameservers: 10.173.22.4 10.173.22.5
NTP servers: time.vmware.com
NTP status: InSync
Gateway ping status: Success
Upgrade connectivity status (svc.ni.vmware.com:443): Passed
Support connectivity status (support2.ni.vmware.com:443): Passed
Performance Telemetry connectivity status (svc.ni.vmware.com:443): Passed
Registration connectivity status (reg.ni.vmware.com:443): Passed
Registration status: Registered
Registration Certificate Validity: Apr 25 00:00:12 2021 GMT
(cli)
```

Running the `show-connectivity-status` command takes a few seconds, but it saves you time in the end because it combines a few other commands into a single command. It shows you the networking configuration, so you can check whether it's appropriately configured, NTP status (and if it's in sync or not), and it tests connectivity to the online upgrade and support services in VMware's' cloud. The entire networking stack is tested using this command.

If any of the network settings are incorrect (and the above output gives an error), they can be corrected using `change-network-settings`. Be aware that the appliance reboots after changing these settings, but you are warned by the CLI as well.

If NTP is not in sync, you can dive deeper into the reason by using `ntp diagnose`. This goes into a deep dive into all aspects of the NTP service, for instance, what the drift is, what time the synchronization last happened, it also does a port scan on the NTP server to see if it can reach it, and more.

### Logs

When the network settings and NTP checks out, it's time to dive into the logs. The command `log-trace` is available to view a list of different logs, search inside those logs, and monitor/follow them in real-time. You can also open up historical logs that have been rotated away. To make it a little bit easier, Network Insight allows you to look at the individual log files, but also the component (service) itself, and it figures out the latest available log file for that component. My advice is to look at the component level and not the actual log files unless you're looking for an event at a specific date and time.

There are 2 relevant components you can look at while doing your troubleshooting: **saasservice** and **restapilayer**. These 2 services are pretty much the most critical services in the Platform appliance. All data is received via the **saasservice** (the Collector sends updates via it), and the **restapilayer** is the internal API service that provides the web interface with data. If you're having issues with data collection or presentation, this is where I would start.

{format: console, line-numbers: false, caption: "Listing available log components and following the saasservice"}
```
(cli) log-trace list components
metric-store-updater
service-health
saasservice
foundationdb
tsdb-server
elasticsearch
vipservice
support-com-logs
path-analyzer
config-store-cleanup
restapilayer
hadoop-hdfs
nginx
RetentionTool
healthservice
hbase
batch-analytics
tracking
launcher
upstart
policy-manager
stunnel4
kafka
wf_emitter_dump
flow-cleanup
lshell
expressJs-server
(cli) log-trace follow saasservice
37442894
naci.eng.vmware.com), KeyValue(key:dpState, value:ACTIVE), KeyValue(key:lastModifiedTimestamp, value:1569400342257), KeyValue(key:skip.certificate.validation, value:false), KeyValue(key:modelKey, value:10000:918:1271298996), KeyValue(key:NSX_HOST, value:nsxmgr-a.nsxonaci.eng.vmware.com), KeyValue(key:NSX_USER, value:admin), KeyValue(key:NSX_PWD, value:), KeyValue(key:ENCRYPTED_CONFIG, value:false), KeyValue(key:healthStatus, value:HEALTHY), KeyValue(key:healthError, value:SUCCEEDED), KeyValue(key:healthErrorCode, value:0), KeyValue(key:lastActivityTimestamp, value:1588330025983)])
INFO [2020-05-01 10:58:45,371] vnera.SaasListener.ServiceThriftListener:[ServiceThriftListener_ServiceImpl:fetchDataProviderListCollectorsFromKeyValueStore:3973] - [dw-75308 - POST /resttosaasservlet] - Time to fetch DP info for collector-IP8683G = 161
INFO [2020-05-01 10:58:45,374] jetty.server.RequestLog:[Slf4jRequestLog:write:60] - [dw-75308] - 127.0.0.1 - - [01/May/2020:10:58:45 _0000] "POST /resttosaasservlet HTTP/1.1" 200 107443
INFO [2020-05-01 10:58:46,312] vnera.SaasListener.ServiceThriftListener:[ServiceThriftListener_ServiceImpl:validateMessage:1112] - [dw-75276 - POST /collectortosaasservlet] - Validating message for customerId:10000 collectorId:IPUSNZ7 hmac:1588330725811:HMAC_:10000:7DJOEwCNZZj99r9fJss2ug==
INFO [2020-05-01 10:58:46,316] grid.uploadHandler.UploadHandler:[UploadHandler:publishMessagesToKafka:543] - [dw-75276 - POST /collectortosaasservlet] - Sending messages to kafka: count=4
INFO [2020-05-01 10:58:46,319] jetty.server.RequestLog:[Slf4jRequestLog:write:60] - [dw-75276] - 100.200.99.6 - - [01/May/2020:10:58:46 _0000] "POST /collectortosaasservlet HTTP/1.0" 200 33
INFO [2020-05-01 10:58:48,160] vnera.SaasListener.ServiceThriftListener:[ServiceThriftListener_ServiceImpl:validateMessage:1112] - [dw-75322 - POST /collectortosaasservlet] - Validating message for customerId:10000 collectorId:IP8683G hmac:1588330728160:HMAC_:10000:57FXS81EpPLFHh6GCrJ1rw==
INFO [2020-05-01 10:58:48,164] grid.uploadHandler.UploadHandler:[UploadHandler:publishMessagesToKafka:543] - [dw-75322 - POST /collectortosaasservlet] - Sending messages to kafka: count=2
INFO [2020-05-01 10:58:48,166] jetty.server.RequestLog:[Slf4jRequestLog:write:60] - [dw-75322] - 10.196.164.26 - - [01/May/2020:10:58:48 _0000] "POST /collectortosaasservlet HTTP/1.0" 200 33
INFO [2020-05-01 10:58:49,371] vnera.SaasListener.ServiceThriftListener:[ServiceThriftListener_ServiceImpl:validateMessage:1112] - [dw-75410 - POST /collectortosaasservlet] - Validating message for customerId:10000 collectorId:IP8683G hmac:1588330729371:HMAC_:10000:ktH3sAfAJVJdSpqWH/QwPw==
INFO [2020-05-01 10:58:49,374] grid.uploadHandler.UploadHandler:[UploadHandler:publishMessagesToKafka:543] - [dw-75410 - POST /collectortosaasservlet] - Sending messages to kafka: count=1
^C(cli)
```

Using `log-trace list components`, a list of available components is presented, and you can use these components to **grep** from or **follow**. In the above screenshot, you can see the list of components on the Platform appliance and the beginning of a **follow** on the **saasservice** components. For those who know their way around Linux/Unix; **follow** is essentially a `tail -f` alias.

You can see a bunch of errors in the above screenshot, related to a rogue Collector that is sending this Platform updates, while it's not registered with the Platform (meaning the Platform is ignoring its messages). There are a lot of possible messages, and I don't have to space to list them all; always look for something in the **ERROR**, or **Exception** category, and interpret the message to the best of your ability. The messages themselves are pretty self-explanatory, most of the time.

If you're looking errors that might have occurred at a specific date and time, you can either open up the right log file using `log-trace display` and look for errors manually; or you can search the logs using `log-trace grep`:

{format: console, line-numbers: false, caption: "Searching in logs"}
```
(cli) log-trace grep ERROR saasservice
INFO [2020-05-01 09:42:16,921] vnera.SaasListener.ServiceThriftListener:[ServiceThriftListener_ServiceImpl:fillDPStatus:4235] - [saas-exec-0] - customerId: 10000, dpId: AZURE_5627a892-5565-4e20-8780-18208fb9f368, collectionStatus: UNHEALTHY, collectionTime: 1588325513610, finalStatus: UNHEALTHY, finalTime: 1588325513610, errorCode: AZURE_NETWORK_WATCHER_STORAGE_ERROR, errorMsg:
ERROR [2020-05-01 09:42:16,955] DPConnectionConfigUtils:[DPConnectionConfigUtils:getHost:78] - [dw-75059 - POST /resttosaasservlet] - No host found for dataSource:NSX, dsKey: NSX_URL, dsConfig: DataSourceConfiguration(dataSource:NSXCONTROLLER, keyValueList:[KeyValue(key:ENABLE_NSX_CONTROLLER, value:false), KeyValue(key:NSX_CONTROLLER_PWD, value:), KeyValue(key:_collectorId, value:IP8683G), KeyValue(key:nickName, value:NSXV-Controller), KeyValue(key:dpId, value:NSXCONTROLLER_sc2nsx.cmbu.local), KeyValue(key:dpState, value:ACTIVE), KeyValue(key:lastModifiedTimestamp, value:1532016723386), KeyValue(key:skip.certificate.validation, value:false), KeyValue(key:modelKey, value:10000:918:1880869159), KeyValue(key:NSX_HOST, value:sc2nsx.cmbu.local), KeyValue(key:NSX_USER, value:admin), KeyValue(key:NSX_PWD, value:), KeyValue(key:ENCRYPTED_CONFIG, value:false), KeyValue(key:healthStatus, value:HEALTHY), KeyValue(key:healthError, value:SUCCEEDED), KeyValue(key:healthErrorCode, value:0), KeyValue(key:lastActivityTimestamp, value:1588324086947)])
ERROR [2020-05-01 09:42:16,955] DPConnectionConfigUtils:[DPConnectionConfigUtils:getHost:78] - [dw-75059 - POST /resttosaasservlet] - No host found for dataSource:NSX, dsKey: NSX_URL, dsConfig: DataSourceConfiguration(dataSource:NSXCONTROLLER, keyValueList:[KeyValue(key:ENABLE_NSX_CONTROLLER, value:false), KeyValue(key:NSX_CONTROLLER_PWD, value:), KeyValue(key:_collectorId, value:IP8683G), KeyValue(key:nickName, value:NSXV-Controller), KeyValue(key:dpId, value:NSXCONTROLLER_wdcnsx-master.cmbu.local), KeyValue(key:dpState, value:ACTIVE), KeyValue(key:lastModifiedTimestamp, value:1532017949582), KeyValue(key:skip.certificate.validation, value:false), KeyValue(key:modelKey, value:10000:918:833398457), KeyValue(key:NSX_HOST, value:wdcnsx-master.cmbu.local), KeyValue(key:NSX_USER, value:admin), KeyValue(key:NSX_PWD, value:), KeyValue(key:ENCRYPTED_CONFIG, value:false), KeyValue(key:healthStatus, value:HEALTHY), KeyValue(key:healthError, value:SUCCEEDED), KeyValue(key:healthErrorCode, value:0), KeyValue(key:lastActivityTimestamp, value:1588325858037)])
```

If you're still lost and have not found an indicator for what's causing your issue, it is now time to open a VMware support case and provide them with the support bundles and open up the support tunnel for them to have a look.

## Configuring Syslog

All Network Insight appliances have a vRealize Log Insight agent installed, which forwards logs to a Syslog server. However, it's not enabled or configured by default. I highly recommend enabling Syslog on all appliances, so there's a central place to look at the logs. Both of the Network Insight appliance types do keep a history of logs, but they get rotated away. Busy systems only keep a few days' worth of logs. When all logs are forwarded to a central log repository, you can decide how long to keep them.

{format: console, line-numbers: false, caption: "Configuring the vRealize Log Insight agent"}
```
(cli) log-insight show
Checking the status of Loginsight agent
Log Insight agent is not configured yet.
usage: log-insight set [-h] --ip-fqdn IP_FQDN [--port PORT] [--tags TAGS]

optional arguments:
    -h, --help        show this help message and exit
    --port PORT       LogInsight server with ssl port, default: 9543
    --tags TAGS       Common key=value tags, Ex: "key1=value1,key2=value2"

 required arguments:
    --ip-fqdn IP_FQDN IP or FQDN of LogInsight server

(cli) log-insight set --ip-fqdn vrli.lab.local
 Validating server reachability for vrli.lab.local:9543
 Server is reachable

 Starting configuration with default tags
 Configured successfully
(cli)
```

The above command is pretty straightforward; supply the IP address or fully qualified domain name of the Log Insight instance, provide an optional port number if the CFAPI protocol is not running on its default port (9543), and provide optional tags.

These tags are interesting if you would like to pass on extra information in the log messages. Log Insight recognizes and stores them, and you can search and filter on; think of them as identifiers for the specific appliance you are currently configuring.

I> When you configure Syslog via the web interface, you only have the option to send out Syslog messages over UDP, which is unencrypted. However, if you use the log-insight command to configure Syslog to Log Insight, the log messages are encrypted by default.

## Configuring a Proxy

When using the Network Insight as a Service deployment model, there's a chance that the Collector, that is deployed inside your data center, has to connect out to the Network Insight service via a web proxy. Proxies are commonly put in place for security reasons, where outgoing HTTP and HTTPS connections need to be screened and secured by this proxy service. As the Platform has outgoing connections towards the VMware cloud to check for updates and initiate a support tunnel (which goes over HTTPS) on-demand, the Platform could also require a web proxy. In any case, the web proxy configuration is a per appliance configuration (it needs to be done on each appliance separately) and only available via the CLI command `set-web-proxy`.

{format: console, line-numbers: false, caption: "Enabling web proxy"}
```
(cli) set-web-proxy enable --ip-fqdn 10.8.0.200 --port 8080
Stopping services

active
active
active
active
Enable http proxy connections...

web proxy connection enabled
Connected to web proxy 10.8.0.200:8080
Starting services

(cli)
```

I> In the on-premises deployment, communication between the Collector and Platform does not go through the web proxy. Only connections to the upgrade and support tunnel services are proxied. In the Network Insight as a Service deployment, communication between the Collector and Platform does go through the web proxy (as the Platform in the SaaS deployment is external).

## Platform Specific Commands

Most of the CLI commands are exactly the same between the Platform and Collector appliances, with only 3 exceptions. I'll discuss these exceptions in the following chapters.

The Platform only has a single command that is specific to it, and it's only applicable in a cluster as well. If you have a single Platform node and have not scaled it to support a large environment, this is not for you.

A Network Insight cluster can be moved between networks and change IP addresses if needed. Let's say it was deployed in a testing environment first, and after a while, you wanted to promote it and move it to the production environment. The consequence can be that it needs to move to a network (typically a different VLAN) with a different IP range. As all appliances in a cluster talk to each other based on their IP address, they all would need to be informed of the network changes, using the command **update-IP-change**. Here's how the workflow looks like in a 3-node cluster:

1. Move Platform1 into the new network and update its IP address via the CLI using:
   - `change-network-settings`
2. Change the IP address of Platform1 on both Platform2 and Platform3 by using:
   - `update-IP-change <old-Platform1-IP> <new-Platform1-IP>`
3. Move Platform2 into the new network and update its IP address via the CLI using:
   - `change-network-settings`
4. Change the IP address of Platform2 on both Platform1 and Platform3 by using:
   - `update-IP-change <old-Platform2-IP> <new-Platform2-IP>`
5. Move Platform3 into the new network and update its IP address via the CLI using:
   - `change-network-settings`
6. Change the IP address of Platform3 on both Platform1 and Platform2 by using:
   - `update-IP-change <old-Platform3-IP> <new-Platform3-IP>`

After executing these steps on all Platform nodes in the cluster, it continues regular operation. If you have a more significant cluster, such as with 5 or 10 nodes, the same steps apply; just multiply by the number of nodes are there.

{format: console, line-numbers: false, caption: "Changing the IP address of a clustered node"}
```
(cli) update-IP-change
usage: update-IP-change [-h] old_ip new_ip
update-IP-change: error: too few arguments
(cli) update-IP-change 10.79.41.179 10.79.41.180

It is recommended to use VM console for this operation. SSH session may disconnect unexpectedly.
Appliance will be rebooted in the end, continue? (y/n) y
Configured hosts/firewall successfully
Rebooting appliance now...
Connection to 10.79.41.179 closed by remote host.
Connection to 10.79.41.179 closed.
```

In the above example, you can see an example where a Platform is moved from IP address **10.79.41.179** to **10.79.41.180**. This command is executed on a different Platform, and after the changed Platform had its IP address changed via the **change-network-settings** command.

I> Warning: the update-IP-change command does not check whether the new IP address is actually valid and the right Platform appliance. Make sure you enter the correct new IP address and that it is up and running before you execute it!

## Collector Specific Commands

There are only 2 commands that are specifically relevant to the Collector appliance: **set-proxy-shared-secret** and **vrni-proxy**. You may have to move a collector between different Platform appliances, or if the Platform appliance went belly up for some reason (got deleted and needed to be redeployed). You can move an existing collector that had a previous relationship to a Platform, to another Platform using these commands:

{caption: "Moving a Collector between Platforms"}
![](images/ch-7/moving-collector-between-platforms.png)

First, generate a shared secret by adding a new Collector VM using the web interface, under **Settings** and **Install and Support** page. Copy and paste the newly generated shared secret to the set-proxy-shared-secret command to that it is trusted by the new Platform. Then update the IP address or fully qualified domain name of the Platform that this Collector this be reporting to using `vrni-proxy set-platform --ip-or-fqdn <ip/fqdn>`. After a few minutes, you see the Collector show up in the web interface, and it is ready to use.

## Command Reference

Just for your convenience, here's a recap of the commands that come in handy when troubleshooting Network insight or commands that are good to know.

{caption: "Useful CLI commands", column-widths: "10%,30%,50%", width: 100%}
| Appliance | Command                                        | Description |
| :----     | :---                                           | :--- |
| Both      | log-insight                                    | Show, set, enable, disable or diagnose the vRealize Log Insight agent |
| Both      | log-trace                                      | Log files; follow them as they get filled up, grep inside them or display a specific log file |
| Both      | modify-password                                | Change the password of the support and/or consoleuser users. |
| Both      | nslookup, ping, ssh-server, telnet, traceroute | Handy troubleshooting commands for networking issues. |
| Both      | ntp                                            | Show, set, manually sync, or diagnose NTP configuration. |
| Both      | services                                       | Start, stop, or restart services. |
| Collector | set-proxy-shared-secret                        | Set a new shared secret on a Collector. Used when moving a Collector. |
| Both      | show-connectivity-status                       | Shows the network configuration and tests the connectivity to VMware's' upgrade and support tunnel services. |
| Both      | show-service-status                            | List the status of all services. |
| Both      | support-bundle                                 | Create, delete, or copy a support bundle that contains all logs and configuration for VMware support to troubleshoot with. |
| Both      | support-tunnel                                 | Enables or disables the support tunnel. This initiates a tunnel to VMware support and lets them log in to your system to have a look. |
| Both      | telemetry                                      | Enables or disables telemetry. This is performance data that is sent to VMware for analysis. VMware support might ask you to enable this when there's an issue. |
| Platform  | update-IP-change                               | Update the IP address of another Platform appliance. Used when you have a Platform cluster. |
| Collector | vrni-proxy                                     | Create a new Platform pairing. Used when moving a Collector. |

Want to see the options or sub-commands for the above commands? Just try the command without any parameters, and it lists the help text associated with it.
