{id: ch-automating}
# Automating Network Insight

We've now come to a topic that's very close to my heart: automation. Because deep, deep, deep down, I'm really lazy. But mostly because I don't like having to do repetitive tasks.

Automation helps you be more efficient and save time on menial tasks. It gets things done faster and does each time exactly the same. Automation helps in bringing the control closer to the users of your products and services and freeing up the IT staff to do better things than just provisioning virtual machines and installing applications manually. Like architecting the infrastructure to become more resilient, explore new technologies to push your infrastructure further. You know, doing more fun stuff.

In the data center world, we're usually talking about deploying application stacks using virtual machines with all kinds of configurations, including storage, networking & compute options via an automation and orchestration layer.

Now I hear you wondering, "What does that have to do with a troubleshooting & monitoring tool?". Well, a lot!

## Pushing Data In

There are a few reasons to push data into Network Insight. They all have to do with providing more context or pushing in configuration data. Let's start with the context reason first.

As you've learned in the chapter [Application Security Planning](#ch-application-security-planning), application context can be discovered from metadata straight from the infrastructures' inventory, or it can be created manually (or via the API). While application discovery works like a charm, it is also good to eliminate the (manual) discovery step by pushing the application construct into Network Insight directly from the infrastructure automation system that is provisioning the application onto the infrastructure. That way, you've got instant context for any newly deployed application.

The other reason to push data into Network Insight is to keep configuration synchronized. Data sources are an excellent example of this configuration. If you have a lot of physical switches, routers, or firewalls deployed in the infrastructure and new ones are added regularly; you could automate the creation of those devices in Network Insight when they are added to the network. That way, they are instantly monitored and available for troubleshooting exercises without having to add them manually.

## Pulling Data Out

At the other end of the spectrum, pulling data out of Network Insight for other systems is another reason to start automating. It is a treasure trove of information, and other systems you might have put in place could benefit from it.

One widely used example is to take the network traffic flows for particular virtual machines (the "highly confidential" or "at-risk" ones) and forward the details of those flows to a Security Information and Event Management (SIEM) system. The Siem system then correlates those flows to other events that applications or other systems generate. Because Network Insight is context-rich, you can be very specific with which flows you export. In one of the real-world examples, I'll get to later (Exporting Network Flows for Security Analytics), it is as simple as setting a simple tag on a virtual machine for it to be included in the broader security scrutiny.

Another example is to take virtual machine information and export it into a Configuration Management Database (CMDB) and make sure the virtual machine details are always in sync with the actual environment.

If that's not interesting enough, how about generating daily reports of the configuration changes in your environment? Every changed firewall rule, every created or deleted virtual machine, virtual machine hardware upgrades, any changes that created a problem inside the infrastructure, you name it.

I could go on with examples, but seeing the range of things you can do should spark some ideas. Take any bit of information Network Insight has and combine it with your systems; there are endless possibilities.

Automation makes it all possible.

## API

Now that we've covered why you would want to automate Network Insight, let's take a look at how. Network Insight has a private and public REST API hosted on the Platform appliance.

Taking one step back, a REST API stands for [Representational State Transfer](https://en.wikipedia.org/wiki/Representational_state_transfer) [Application Programming Interface](https://en.wikipedia.org/wiki/Application_programming_interface). REST has become an industry-standard in the last few years, simply because it's dead simple. You can do incredibly advanced automation with REST APIs, but in the basics, it's just HTTP calls, and you can use any HTTP client (like a browser or cURL) to call on these APIs. When testing with APIs, I recommend you use the [Postman](https://www.getpostman.com/) client, which is perfect for API development.

Inside REST, there's a reference architecture that lets you do HTTP calls in different ways that have different effects. If that sounds fuzzy, here's a technical explanation:

HTTP has several different methods, like GET to retrieve information, POST to deliver information (the content should be put in the body of the request), PUT to modify information, and DELETE to remove information. These methods translate to the action you want to take in the API. If you make a GET request, you'll get information back from the API. If you make a DELETE request, the API knows that you want to delete an object.

When receiving the response from the API, there are a few aspects to take into account. The response HTTP status code (200, 400, etc.) indicates whether the API call was a success (200), or if the request was formatted incorrectly (400), or if you don't have an authorized session (401), or if the API broke something (500). There are other codes, but these are the most relevant. If you requested content (GET), the returning body contains the requested information.

Officially (as not all APIs adhere to this), REST APIs should use JavaScript Object Notation (JSON) as the format in which the data is transferred between client and server. JSON is also an industry standard, which has the benefit that almost every programming or scripting language knows how to read JSON and present it back to you in a usable form. It also helps that it's human-readable.

This book isn't about teaching you about these standards and technologies, and for the rest of this chapter, I'm going to assume you have a basic grasp of REST calls and JSON. I'm also not going through all API calls and functionalities, but I'll give you enough information to get started. Complement this chapter by reading the vRealize Network Insight API Guide.

I> Currently, Network Insight rate limits the number of API calls per second to 20. When doing API calls, rate limit your script not to overload the API. You will be throttled, and the API starts giving out error code **429**.

### API Explorer

Documentation is essential, and Network Insight has an API Explorer built into the interface. The API Explorer holds the reference documentation for each API call that's available. If you're looking for parameters to include in the API call, this is the place. It also has a convenient feature where you can try out the API call directly from the interface to help in the process of formatting the right structure for a successful API call.

It can be found under the gear icon on the top right; API Documentation. There are three tabs: API Reference, Documentation, and Related Code Samples. The reference tab lists all API calls and their format and parameters (you'll use this one the most), and the documentation tab contains links to the API Guide and OpenAPI specification. When you're connected to the internet with your browser, the Related Code Samples tab also appears. This tab is especially cool because it retrieves example scripts from the [VMware Sample Exchange](https://code.vmware.com/samples?categories=Sample&keywords=&tags=vRealize%20Network%20Insight&groups=&filters=&sort=dateDesc&page=). If you need to get some inspiration on use cases, or just want to consume the scripts -- there are some excellent examples there.

{caption: "Built-in API Explorer"}
![](images/image96.png)

### Swagger / OpenAPI Specification

An [OpenAPI](https://swagger.io/specification/) (previously known as Swagger) specification is a description format for an API that describes the contents of an API. Each endpoint, input and output parameters, authentication methods, and other details around the API. This specification is what the developers use to describe all API endpoints, and it is used to generate the documentation you see in the API Explorer.

It can also be used to generate code for client libraries automatically, or otherwise known as software development kits (SDK). While Network Insight does not have those SDKs yet, I'm holding out hope for future availability.

The OpenAPI specification is an excellent reference source when you're building automation that consumes the custom objects that the API provides. For instance, when retrieving information about entities (VMs, IP Sets, Hosts, all objects from your infrastructure), there is an **entity\_type** field inside the returned object data. This **entity\_type** describes the schema of data that is included in the result. By looking up the entity type inside the OpenAPI specification, you know what schema you can expect to be able to use.

For example, here's a snippet of the "VirtualMachine" entity type:

{format: json}
```
"VirtualMachine": {
    "allOf": [
      {
        "$ref": "#/definitions/BaseVirtualMachine"
      },
      {
        "properties": {
          "cluster": {
            "$ref": "#/definitions/Reference"
          },
          "resource_pool": {
            "$ref": "#/definitions/Reference"
          },
          // ...snip...
          "cpu_count": {
            "type": "integer",
            "format": "int32"
          },
          "memory": {
            "type": "integer",
            "format": "int32"
          }
        }
      }
    ]
  }
```

These definitions can be nested, which the field **allOf** is referencing too. The **VirtualMachine** entity inherits everything from **BaseVirtualMachine**. Then we have the fields that are specific to a VM, like **cpu\_count** and **memory**, but also references to other entities.

If you look at the **cluster** field, it has **Reference** as value. The **Reference** field means that the result contains references to other entity IDs, which you can request separately. In this case, the entity ID that is returned in the cluster field is the vSphere DRS Cluster, and you can retrieve the details about that DRS cluster by referencing that ID.

{id: ch-authentication}
### Authentication

Before you can do any other API calls, you need to authorize your session and get an authentication token. This token is used throughout any subsequent API calls that you want to execute.

To get an authentication token, use the **/auth/token** API endpoint. This endpoint is the only place where you'll need to specify a username and password; all the other endpoints rely on the authentication token for proper authorization.

As we're sending data (the credentials) across, the /auth/token endpoint takes a POST request. Formatting the POST body is fairly straight forward using JSON:

{format: json}
```
{
  "username": "admin@vrni.com",
  "password": "myPassword",
  "domain": {
    "domain_type": "LDAP",
    "value": "example.com"
  }
}
```

This example uses a credentials that is tied to a LDAP directory with the domain name **example.com**. If you were to use credentials that are local to the Network Insight platform, you would use something like this:

{format: json}
```
{
  "username": "admin@local",
  "password": "myPassword",
  "domain": {
    "domain_type": "local",
    "value": "local"
  }
}
```

If the call is successful (right credentials, formatted correctly and the API or backend isn't on fire), you will receive a HTTP code 200 back and the response body will look something like this:

{format: json}
```
{
  "token": "1rT7tm4riiACSfxrO2BvkA==",
  "expiry": 1509332642427
}
```

An authorization token is valid for 5 hours after you've requested it. Usually, API calls are done on demand, but if you're scheduling calls continuously, keep in mind to refresh the token at least every 5 hours. Otherwise, it expires, and your subsequent API calls return a 401 (unauthorized). The **expiry** field is the exact time that the token expires. It is a precise time, as it is formatted as an epoch timestamp in milliseconds (epoch usually stands for the seconds from January 1^st^, 1970 at 00:00:00 GMT). You can use this timestamp to see if your token is due for a refresher.

Then there's the **token** field. This base64 encoded string is to be used in any following API calls in the HTTP headers, like so:

{format: console, line-numbers: false}
```
Authorization: NetworkInsight 1rT7tm4riiACSfxrO2BvkA==
Content-Type: application/json
```

Save the authentication token to a variable that you can reuse inside the script or workflow that you are building.

**vRealize Network Insight Cloud Authentication**

As you may remember from the chapter [Hosted Architecture (SaaS)](#ch-hosted-architecture)), there is a difference in authentication between the on-premises Network Insight and the one that's delivered in the VMware Cloud. There is a single-sign-on platform in place for all VMware Cloud Services (CSP) products, meaning there is no user management within Network Insight, and you'll need to go through CSP to get an authentication token to use in the Network Insight API calls.

CSP works with so-called refresh tokens as a way of authentication. When using the API, you need to exchange a refresh token for an authentication token and use that when talking to the Network Insight API (or any other Cloud Service).

Refresh tokens are linked to a CSP account. To create one, log into <https://cloud.vmware.com> and go to your Console. Then open up your personalized menu (top right) and click **My Account**. Select the **API Tokens** tab and click **Generate a new API token** to get a refresh token. It'll have this format:

{line-numbers: false}
```
ax22ea9i-139b-344x-lif7-ex6856ce57fa
```

These refresh tokens are valid for 6 months, which you need to keep in mind when building automation based on these tokens.

After getting a refresh token, there's a single CSP API call you need to call to exchange a refresh token for an authentication token:

<https://console.cloud.vmware.com/csp/gateway/am/api/swagger-ui.html#/Authentication/getAccessTokenByApiRefreshTokenUsingPOST>

Executing this API call successfully, will give you a result like this:

{format: json}
```
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9…verylongstring",
  "expires_in": 0,
  "id_token": "string",
  "refresh_token": "string",
  "scope": "string",
  "token_type": "string"
}
```

The **access\_token** is the important field here, which you need to save and consider as the authentication token for the Network Insight API calls. There is, however, a slight difference in how you send this token across to the API. Instead of sending the **Authorization** header in your API call, you'll need to insert a header called **csp-auth-token** with the value of the authentication token that you've retrieved from the CSP API:

{line-numbers: false}
```
csp-auth-token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9…verylongstring
```

As a final note, it is also worth noting that the URL for the Network Insight as-a-Service API is the same for all environments: <https://api.mgmt.cloud.vmware.com/ni/api-endpoint>.

Once you're authenticated, return to the API Explorer and find the API call that you need to retrieve or push the information you want.

### A Few Examples

When using the API, you most likely have a fixed goal and know what you need to do to get there. The API Explorer is your best friend for getting examples. Below are a few examples which are to illustrate how to work with the API in a broad sense. There are some concepts you need to take into account.

#### Retrieving a list of VMs

Using Network Insight to get a list of VMs might be useful if you have a large environment with multiple vCenters. Network Insight aggregates all data in one place, so you'd only have to do a single API call to get every VM.

First, we need to find the right API call to get this list. Turn to the API Explorer, and you find a call to the endpoint **/entities/vms**. This has the description "List VMs", so it's probably the one we need. If you then look at the available parameters, this is what shows:

{caption: "Parameters for API endpoint /entities/vms"}
![](images/image98.png)

For all the Entity endpoints, you'll see that the parameters are pretty much the same. There is a **size** parameter for the number of results returned on a page, a **cursor** parameter to indicate from which page you want to start getting results, and a **start\_time** and **end\_time**, which can be used to go back in time. Remember that there is a timeline to show historical data? Using **start\_time** and **end\_time** can get the list of VMs that existed a week ago, including the ones that have been deleted since.

The **cursor** is important to get full results. By default, the results for returned entities are paged in pages of the indicated **size** (default 10 results), and with every returned page, there'll also be a next **cursor** value in there, which you can use to request the next page. Have a look at this example:

{caption: "Using Postman to execute API endpoint /entities/vms"}
![](images/image99.png)

In this example, I'm using Postman to get a list of VMs, limited by 2 results. The first thing you'll notice is that there are actually any VM attributes listed in the results, just **entity\_id**s. This works the same with all entities; it returns a list of references to entities. You can take the **entity\_id** and get detailed information by using the specific entity type API endpoint. In the case of the first result, this is **/entities/vms/17603:1:1010454414**, but we'll get to that.

Inside the result, the "results" array contains the resulting entities (thanks, captain obvious!). I'd like to get your attention on the other results. As said, there's a **cursor** value that indicates the next page, a **total\_count** with the number of total results available (regardless of the page limit) and the timeline values. The timeline is by default set to the current time if you do not give a timeline yourself.

We've got 2 results now, and there are 109 in total, which makes this next API call:

{line-numbers: false}
```
https://ni-platform.lab.local/api/ni/entities/vms?size=2&cursor=Mg==
```

The following result is the next 2 VMs in the list and another cursor value. Continue like this until you've got all 109 VMs returned. Do this inside a loop and dynamically look for the **total\_count** and the current count of results to determine to do another API call or be satisfied with the results. You could also look out for the **cursor** value. If there are no more pages, the cursor value is empty, and you can stop looping through the API calls.

#### Creating an Application

Now you know how to retrieve information, let's take a look at creating something via the API. Of all API calls that push data towards Network Insight, creating applications is the most used one as it can be used on an ongoing basis. Applications are the most dynamic factor in most environments, usually in a way that new applications are being spun up all the time.

If you want to get the right context for troubleshooting and monitoring, you need to populate those new applications inside Network Insight. Couple this process closely to your application deployment automation and use the Network Insight API to provision applications while the application is being deployed. Automagically.

There are two steps required to create an application container:

1. Create the application itself
2. Create a tier within the newly created application with a filter that points to workloads (using tags, VM names, folders, any logical object in the virtualization layer).

In the API Explorer, there's an entire section devoted to application management. You'll quickly find the endpoints **/groups/applications** (POST) and **/groups/applications/{id}/tiers** (POST), which are needed to create an application.

Creating an application via /groups/application (POST) doesn't require much; just an application name. The result contains the **entity\_id** that it has given the newly created application construct. Store that for the next call. Here's an example using the name **My-New-Application**. The top text area is the body that is being sent to the API, and the bottom text area is the result that the API returns:

{caption: "Using Postman to create an application via the API"}
![](images/image100.png)

You now have an empty application without any tiers. Let's add some!

Here's where the previously saved **entity\_id** comes in handy, as the API endpoint looks like this: **/groups/applications/17603:561:840848559/tiers**

The body of this endpoint is a bit more elaborate, though, as it needs not only a name but also a filter to determine which workloads are added to this tier. The filter is basically a search query, so you can filter on any logical object (tags, VM names, folders, and more) to get the right VMs in the tier. In this example, I'll use a simple search based on a VM's name. Here's the formatted API call:

{caption: "Using Postman to create an application tier via the API"}
![](images/image101.png)

As you can see, there is a **name** field (which is the name the new tier is given) in the body, and an array called **group\_membership\_criteria**. This is where you define the search query that looks for workloads to put into the tier.

I've used an **entity\_type** called **BaseVirtualMachine** and a filter that looks for the names **VM01** and **VM02**. The filter can be formatted using the search query logic (throwback to chapter [Using the Search Engine](#ch-search)), and you can find the **entity\_type** options in the OpenAPI specification under the *definitions* structure (examples are: Cluster, SecurityTag, EC2SecurityGroup, and on).

The result of creating the application tier is the tier definition echoed back plus the newly assigned **entity\_id** along with a reference to the parent applications' ID.

You now have a new application with a single-tier with 2 VMs in it. If you need multiple tiers, rinse and repeat the second call.

## Using PowerShell (PowervRNI)

Moving on from using raw APIs, there are tools available to get you faster up and running. When you're integrating with an existing automation or orchestration platform, you might need to use the API directly, but if you just want to execute a simple script to speed up a task, abstraction tools are the way to go. With these, you can simply execute a function, and it handles the API endpoint calls for you. Just focus on the bare minimum input and get on with the results.

In this chapter, I'm going to walk you through using PowerShell to talk to the Network Insight API.

### Why PowerShell?

This is a question I get a lot. There are a lot of scripting frameworks available, and there are usually two camps: Linux based and Windows-based tools. This is mostly because these scripting frameworks are built into the operating system itself.

People from the Linux world usually default to scripting languages like Python, Ruby, or even Perl. Because these are (usually) installed by default on a Linux distribution.

People from the Windows world usually default to PowerShell, because it's available by default on the newer versions of Windows.

But there's more. One of the reasons PowerShell is a popular scripting language to use is because it uses a natural language design of its features. All functions that you can use are named in a very natural way. If you want to get some information, the function starts with **Get-**. If you want to set an option or parameter, the function starts with **Set-**. If you want to invoke a command of some sort, the function starts with **Invoke-**. There are more of these prefixes, but these ones are used the most.

PowerShell is also easily extendable with modules. Modules are basically just text files with custom functions that you can download and load or create yourself and publish. There is a central repository located on [powershellgallery.com](https://www.powershellgallery.com/), from which you can download modules manually and install them or simply install them directly from a PowerShell window with **Install-Module moduleName**. There are currently over 4.200 modules on the PowerShellGallery, so plenty to choose from.

It also helps that PowerShell was made a multi-platform tool for Windows, Linux, and macOS with version 6.0 at the beginning of 2018. I've been running it on macOS ever since and haven't looked back at my Windows virtual machine. ;-)

If you would like to learn more about PowerShell, I highly recommend the [PowerShell: Getting Started](https://app.pluralsight.com/library/courses/powershell-getting-started/table-of-contents) course by Michael Bender on Pluralsight.

### PowervRNI

Most of the VMware related PowerShell modules, official and community-developed, start with the prefix **Power** to relate it to PowerShell. For example, we have [PowerNSX](https://github.com/vmware/powernsx), [PowerCLI](https://code.vmware.com/tool/vmware-powercli), [PowervRA](https://github.com/jakkulabs/PowervRA), and a couple others.

When Network Insight 3.6 with the first version of the public API came out in November 2017, the PowerShell module to make use of this API was already in the making, as it was highly requested amongst the companies I was working with. The module was named [PowervRNI](https://github.com/PowervRNI/powervrni), to conform to the unofficial naming convention.

It currently has 67 functions and around 95% of API coverage.

My goal with PowervRNI was pretty straight forward: make it easy to get started with the Network Insight public API and make it easy to integrate with other systems. I'm proud to say that it has been used to solve a breadth of use cases involving data sharing issues between multiple systems. From security use-cases where extra data around network traffic flows and infrastructure info is sent to a SIEM system to better collation, to importing a large amount of data sources (usually physical network devices) into Network Insight during the initial set up, to integrations with CMDB systems to synchronize the CMDB inventory with the real-life situation. I'll take you through a few example use cases in the upcoming chapters, but up-to-date examples are found [on my blog](http://lostdomain.org/tag/powervrni/).

#### Getting Started with PowervRNI

There are two ways to get PowervRNI on your system: a manual and automated way.

##### Installing Automagically

Using the built-in cmdlet **Install-Module** downloads the module source files for you and place them somewhere where PowerShell knows to load them. Open up a PowerShell window and execute these commands:

{format: console, line-numbers: false}
```
PS C:\> Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
PS C:\> Install-Module PowervRNI
```

The first command is to make the PowerShellGallery a trusted source. You'll only have to do this once, and it prevents showing a notice that asks you for permission to download from the PowerShellGallery because it's an untrusted repository. You can still install the module without adding it to the trusted repositories, but it's just cleaner this way.

After the installation has completed, load PowervRNI like this:

{format: console, line-numbers: false}
```
PS C:\> Import-Module PowervRNI
```

Notice that there's a slight difference in loading the PowervRNI module. When installing a module via **Install-Module**, you can load the modules from anywhere, and you don't have to give the relative path to the module source files. It's also an added benefit that you can update the module with **Update-Module** as well. That's why I highly recommend doing it via the automagical way if you have the option.

##### Installing Manually

Manually installing PowervRNI can be a good (or the only) option on systems where you don't have permission to use the cmdlet **Install-Module**. Although the newer PowerShell versions have to ability to install modules just for yourself (parameter "*-Scope CurrentUser*"), it can still be locked down.

A manually install simply means downloading the module source files and placing them locally, where you can load them.

Get the module source files from here: <https://powervrni.github.io/>

And then open up a PowerShell window, change directory to where you have put the module source files and load PowervRNI like this:

{format: console, line-numbers: false}
```
PS C:\> Import-Module PowervRNI.psd1
```

##### Getting Familiar

Now that PowervRNI is on your system and loaded, take a little time to explore the available cmdlets to see what's available and which ones you would like to try. Get a complete list of available cmdlets by executing **Get-Command**:

{format: console, line-numbers: false}
```
PS C:\> Get-Command -Module PowervRNI

CommandType   Name                         Version    Source
-----------   ----                         -------    ------
Function      Connect-NIServer             1.4.74     PowervRNI
Function      Connect-vRNIServer           1.4.74     PowervRNI
Function      Disable-vRNIDataSource       1.4.74     PowervRNI
Function      Disconnect-vRNIServer        1.4.74     PowervRNI
Function      Enable-vRNIDataSource        1.4.74     PowervRNI
Function      Get-vRNIAPIVersion           1.4.74     PowervRNI
Function      Get-vRNIApplication          1.4.74     PowervRNI
...etc...
```

Every cmdlet in PowervRNI is well-documented and has examples of its usage, and you can get that documentation directly in your PowerShell console:

{format: console, line-numbers: false}
```
PS C:\> Get-Help Get-vRNIVM -Examples

NAME
    Get-vRNIVM

SYNTAX
    Get-vRNIVM [-Limit <Int32>] [[-Name] <String>] [-Connection <PSObject>] [<CommonParameters>]

SYNOPSIS
    Get virtual machines from vRealize Network Insight.

    -------------------------- EXAMPLE 1 --------------------------
    PS C:\> Get-vRNIVM

    List all VMs in your vRNI environment (note: this may take a while if you have a lot of VMs)
```

##### Connecting to Network Insight

In the previous chapters, you've learned about the way Network Insight handles authentication (via authorization tokens). PowervRNI uses the authorization tokens for all API calls, which means you need to connect and authorize first.

There are two cmdlets to connect a Network Insight instance. There's **Connect-vRNIServer** to connect to vRealize Network Insight (the on-prem variant), and there's **Connect-NIServer** to connect to the vRealize Network Insight Cloud variant.

**Connect-NIServer** is extremely simple to use and only required the Refresh Token that you have created in chapter [Authentication](#ch-authentication).

{format: console, line-numbers: false}
```
PS C:\> Connect-NIServer -RefreshToken 4b891k9e-9swd-43e1-bd93-06xxxa600d7
Server				CSPToken
------				------
api.mgmt.cloud.vmware.com/ni	eyJhbXbi9iJSUzI1NiIsInR5cCI6k..verylongstring
```

This function exchanged the Refresh Token for an authentication token, and that is used for subsequent API calls, and you are now free to use the rest of the cmdlets inside PowervRNI.

**Connect-vRNIServer** takes a few more parameters, as the on-premises instance would have more details like the IP or hostname of the Platform VM and the credentials to connect.

There are two ways to handle the credentials. You can input the username and password when you connect (you pass them as parameters, or be prompted for them), or you can create a [PSCredential](https://blog.kloud.com.au/2016/04/21/using-saved-credentials-securely-in-powershell-scripts/) file once and refer to that file when connecting. Creating a PSCredential file would be the preferred way to connect if you're running a scheduled task. Remember, each time a plain text password is stored somewhere, a puppy dies.

{format: console, line-numbers: false}
```
PS C:\> Connect-vRNIServer -Server ni-platform.lab.local -User 'admin@local'

PowerShell credential request
vRealize Network Insight Platform Authentication
Password for user admin@local: ********


Server                           AuthToken                AuthTokenExpiry
------                           ---------                ---------------
ni-platform.lab.local SbX94W8aGCk7yzX3EpQU9A== 02/02/2019 17:02:55
```

If the connection is a success, you see the authentication token being returned, which is stored and used for subsequent API calls.

{pagebreak}

## Automation Use Cases

Now that we've covered the **how** of automation with Network Insight, let's focus on the **why**.

In the upcoming chapters, I'll give you four examples of use cases that might give some inspiration. These examples are not all there is, of course, the amount of value you can get out of automation are as vast as the data that's being collected by Network Insight. The following examples are real-life use cases that I've helped organizations to implement.

### Integrating with Infrastructure Automation & Orchestration systems

Beginning with the first of two application-focused use cases, let's have a look at integrating infrastructure automation and orchestration systems. In the context of this integration, the infrastructure automation system is used to deploy applications and all its dependencies. From the virtual machines with their compute, storage, and network properties, to installing required software packages and configuring them. Full application deployment.

Why is that important? Well, remember that having the application context in Network Insight enriches your experience immensely. How great would it be to have the application context be created at the same time as the actual application is being deployed? That's where integrating into your automation system comes into play.

#### Example

vRealize Automation (vRA) is an infrastructure lifecycle management and automation platform. vRA can be compared to an octopus; it extends its reach to other systems to execute actions (like deploying a virtual machine) and can grow extra (custom) arms to systems that it does not talk to out-of-the-box.

As an example, I'll be using a 3-tiered application blueprint that contains a web server, application server, and database server tier. Of the web and application tiers, there can be multiple VMs deployed, but there's just one database server. They are linked to vSphere templates, which are cloned when a deployment is requested.

{caption: "Example vRealize Automation 3-tiered Application Blueprint"}
![](images/image102.png)

This blueprint design relates directly to an application construct within Network Insight, using the blueprint name as the application name and the different types of machine deployments (Web, App, DB) as the names for the tiers. When this blueprint is deployed by someone, vRA deploys the virtual machines, networks, storage, and software packages on the newly created virtual machines. After that work is done, we can insert a custom vRealize Orchestrator (vRO, the octopus' engine) workflow that takes this newly created application and creates the application context inside Network Insight.

Inside vRA, there's an Event Broker that you can use to kick off workflows during specific stages of a deployment process. You can create a subscription on the event that signals when a deployment is done, and all virtual machines have been deployed. This subscription also indicates which vRO workflow should be started.

I> The vRO workflow and a step by step guidance of how to install it can be [found on my blog](http://lostdomain.org/2018/11/08/integrating-vrealize-automation-with-network-insight/). The actual code is not the point of this chapter; I'm giving you an example of how it can work with vRA, but this can be applied with any and all infrastructure automation & orchestration systems.

Here's a graphical overview of how the process works so that you can translate it to your system.

{caption: "Push Applications from Automation workflow"}
![](images/image103.png)

{pagebreak}

### Importing Applications from Configuration Management Databases

Application Discovery is essential to get the right application context in Network Insight. While you can natively integrate the ServiceNow CMDB, there are ways to integrate other CMDBs using the API. If you are not using ServiceNow for your CMDB needs, don't fret. I think it's pretty safe to say that all relevant CMDB systems have a way to export data from its systems, and most also have an API.

If the CMDB has an API, excellent! You can set up a periodical synchronization between the CMDB and Network Insight. This workflow should retrieve a list of application names, optional tiers, and all the virtual machines and physical IP addresses attached to that application. You then use that list and ask the Network Insight API if this application already exists and compare the virtual machines and physical IP addresses to see if anything has changed (and then update it). If the application doesn't exist, create it with the info from the CMDB.

Here's a visual representation of this workflow:

{caption: "Import Applications from CMDB workflow"}
![](images/image104.png)

#### Example

Importing data from a CMDB can be easy when it has an API. iTop is an open-source CMDB system that can track infrastructure inventory and changes to it. I use it to track my inventory (yes, I need a system for that. Don't judge me. ;-)).

In the examples directory of PowervRNI, there is an example script that pulls out application items from iTop, traverses the relationship tree, and discovers VMs that are attached to that application. Then it adds that application tree as application constructs into Network Insight.

As the script is just a bit too big to paste here, I'll leave you with a link to it:

<https://github.com/PowervRNI/powervrni/blob/master/examples/cmdb-import-from-itop.ps1>

{pagebreak}

### Exporting Network Flows for Security Analytics

Getting out of the application realm, let's focus on a security use case. As Network Insight gathers all network flows going through the network, it has visibility on what's actually happening with your applications. Is it behaving accordingly, sending and receiving traffic that it is supposed to? For instance, a web server should only send out connections that it needs to support its website (usually just HTTP\[S\]). If it starts sending out SSH traffic, it's probably compromised and is trying to compromise other systems.

While you can create user-defined events based on anomalies like that and have an alert sent out when that starts happening, it's usually more prevalent to send the network flow information to a SIEM system. That system would also be able to take the application generated logs and correlate them together for an end to end view.

Sending all network flows all the time might not be a preferable situation, as the amount of data can overwhelm the SIEM, and it might not be interesting to record every network flow. Instead, what I've seen is more a reactive setup based on other events. The SIEM or other monitoring system catches something that seems off and sets a tag on a virtual machine, which in turn triggers the network flow import to SIEM.

Of course, you can customize this to your requirements.

#### Example

Using PowervRNI, exporting flows, and sending them to another system is fairly simple as there's a cmdlet needed to retrieve the network flows: **Get-vRNIFlow**. Here's an example:

{format: console, line-numbers: false}
```
PS C:\> Get-vRNIFlow -Limit 1

entity_id                  : 17603:515:1298175574
name                       : 10.8.20.66(ns3.lostdomain.local) -> 31.3.105.98(NL) [port:53]
entity_type                : Flow
source_vm                  : @{entity_id=17603:1:859119666; entity_type=VirtualMachine}
source_vnic                : @{entity_id=17603:18:292970721; entity_type=Vnic}
source_datacenter          : @{entity_id=17603:105:88573490; entity_type=VCDatacenter}
source_ip                  : @{ip_address=10.8.20.66; netmask=255.255.255.255; …}
destination_ip             : @{ip_address=31.3.105.98; netmask=255.255.255.255; ...}
source_l2_network          : @{entity_id=17603:12:727099007; entity_type=VlanL2Network}
port                       : @{start=53; end=53; display=53; iana_name=dns; iana_port_display=53 [dns]}
source_folders             : {@{entity_id=17603:81:2131637277; entity_type=Folder}}
destination_folders        : {}
source_resource_pool       : @{entity_id=17603:79:876290838; entity_type=ResourcePool}
source_cluster             : @{entity_id=17603:66:694428497; entity_type=Cluster}
protocol                   : UDP
source_ip_sets             : {}
destination_ip_sets        : {}
source_security_groups     : {}
destination_security_groups: {}
traffic_type               : INTERNET_TRAFFIC
source_security_tags       : {}
destination_security_tags  : {}
source_host                : @{entity_id=17603:4:208253188; entity_type=Host}
source_vm_tags             : {Tags:BackDatUp}
destination_vm_tags        : {}
within_host                : False
firewall_action            : ALLOW
flow_tag                   : {INTERNET_TRAFFIC, SRC_IP_VM, DST_IP_INTERNET, DIFF_HOST...}
```

Flow records have a bunch of correlated information attached, as you can see in the example above. I'd like to highlight a few fields on which you could filter, which would be beneficial when looking for specific flows:

{caption: "Interesting Flow properties"}
| Field                | Description |
| :---                 | :---        |
| **traffic\_type**    | Which way is the traffic going? **INTERNET\_TRAFFIC** or **EAST\_WEST\_TRAFFIC** are possibilities. |
| **source\_\***       | Source of traffic; not only IP address, but also context like **vm**, **vnic**, **datacenter**, etc. |
| **destination\_\***  | Same as above, only for the destination (including context) |
| **firewall\_action** | When integrated with VMware NSX Data Center, this can show flows which are blocked by the Distributed Firewall (DFW) |

{pagebreak}

### Tenant Bandwidth Chargeback / Showback

Using the network flow data in Network Insight, you can also make money. I used to work for a provider that had a range of services, like leasing out physical data center room and network connectivity to or simple virtual machines. All services had something in common; we charged for bandwidth. This is reasonably common in provider land, and everyone has their way of metering, we had a custom application (that I created) that listened on a mirror (span) port and looked at the actual traffic of our tenants. It saved the byte size of the packets linked to a source IP and saved that number to a database. At the end of every month, we generated reports from this database to invoice the customers for the amount of bandwidth they used that month.

While we had our issues, this was a pretty clean approach. Other service providers didn't have the option of creating their own program to do this and relied on multiple data sources to tell them the total bandwidth usage. I've heard many stories about losing revenue because they couldn't get the right data to invoice their customers.

This bandwidth data is available within Network Insight, and there are ways to get a clean list of bandwidth usage per IP (source or destination) in several formats. Using the API is one of those formats. In the below example, I'll go through a straightforward example of how to use the API and retrieve bandwidth usage per source IP address.

#### Example

There is an API endpoint called aggregation (_/api/ni/search/aggregation_), which can be used to fetch sums or averages of a specified metric. In this case, we'll be using the aggregation endpoint to retrieve the sum of flows coming and going to a specific IP address. To understand what's going on in that script, here's the API call:

{format: json}
```
POST https://ni-platform.lab.local/api/ni/search/aggregation
{
    "entity_type": "Flow",
    "aggregations": [
    {
        "aggregation_type": "SUM",
        "field": "flow.totalBytes.delta.summation.bytes"
    }
    ],
    "filter": "source_ip.ip_address = '10.8.20.66'",
    "start_time": 1560589466,
    "end_time": 1560675866
}
```

Inside this call, the **aggregations** field is where we specify the operation the aggregation needs to perform. In this case, it's a **SUM** operation on the field
**flow.totalBytes.delta.summantion.bytes**. This field translates to "SUM(Bytes) of Flow" in the regular UI search bar, as the API search uses the internal naming convention of objects.

Also take note of the filter, where the search condition is given. This can be a single condition, as in the example above, but also multiple conditions. For example, if you don't care about splitting out download and upload and want a single number for all bandwidth combined, use:

{format: json, line-numbers: false}
```
"filter": "source_ip.ip_address = '10.8.20.66' or destination_ip.ip_address = '10.8.20.66' "
```

You can modify the filter to use VMs or any other of the objects inside Network Insight. Get creative!

Lastly, the **start\_time**, and **end\_time** fields indicate the time window that the result should be based on.

As a result of this API call, you will get a recap of the aggregation request and the value of the query:

{format: json}
```
{
  "aggregations": {
    "field": "flow.totalbytes.delta.summation.bytes",
    "aggregation_type": "SUM"
    "value": 4978402
  },
  "total_count": 1,
  "start_time": 1560589466,
  "end_time": 1560675866
}
```

The field **aggregations.value** is the number of bytes that match the filter.

You can find the PowervRNI example script here: <https://github.com/PowervRNI/powervrni/blob/master/examples/get-bandwidth-usage-per-ip.ps1>.

## Automation Summary

I hope the examples in this chapter have been helpful to get your imagination going on what is possible when you start automating Network Insight. These examples have just scraped the surface of what's possible, and there's a whole integration ecosystem possible that takes advantage of the data.

If you have any other use-cases that you are building, please do reach out to me via any of my communication channels; I'd like to learn from you. :-)
