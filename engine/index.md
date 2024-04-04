# CyberSecurityEngine


Here's something about Shodan.

<!--more-->

# 1.Shodan

## 1.1What's Shodan？

> Shodan is a [search engine](https://en.wikipedia.org/wiki/Search_engine) that lets users search for various types of servers ([webcams](https://en.wikipedia.org/wiki/Webcam), [routers](https://en.wikipedia.org/wiki/Router_(computing)), [servers](https://en.wikipedia.org/wiki/Server_(computing)), etc.) connected to the [internet](https://en.wikipedia.org/wiki/Internet) using a variety of filters.[[1\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-1) Some have also described it as a search engine of [service banners](https://en.wikipedia.org/wiki/Banner_grabbing), which are [metadata](https://en.wikipedia.org/wiki/Metadata) that the [server](https://en.wikipedia.org/wiki/Server_(computing)) sends back to the client.[[2\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-shodanabout-2) This can be information about the server software, what options the service supports, a welcome message or anything else that the client can find out before interacting with the server.
>
> Shodan collects data mostly on web servers ([HTTP](https://en.wikipedia.org/wiki/HTTP)/[HTTPS](https://en.wikipedia.org/wiki/HTTPS) – ports 80, 8080, 443, 8443), as well as [FTP](https://en.wikipedia.org/wiki/FTP) (port 21), [SSH](https://en.wikipedia.org/wiki/Secure_Shell) (port 22), [Telnet](https://en.wikipedia.org/wiki/Telnet) (port 23), [SNMP](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol) (port 161), [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) (ports 143, or (encrypted) 993), [SMTP](https://en.wikipedia.org/wiki/SMTP) (port 25), [SIP](https://en.wikipedia.org/wiki/Session_Initiation_Protocol) (port 5060),[[3\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-shodanfaq-3) and [Real Time Streaming Protocol](https://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol) (RTSP, port 554). The latter can be used to access webcams and their video streams.[[4\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-4)
>
> It was launched in 2009 by [computer programmer](https://en.wikipedia.org/wiki/Computer_programmer) John Matherly, who, in 2003,[[5\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-SMH_interview-5) conceived the idea of searching devices linked to the Internet.[[6\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-wash_post-6) The name Shodan is a reference to [SHODAN](https://en.wikipedia.org/wiki/SHODAN), a character from the [System Shock](https://en.wikipedia.org/wiki/System_Shock) video game series.[[5\]](https://en.wikipedia.org/wiki/Shodan_(website)#cite_note-SMH_interview-5)

Shodan is a search engine, like google, but instead of searching for websites, it searches for internet-connected devices — from routers and servers, to Internet of Things (IoT) devices, such as thermostats and baby monitors, to complex systems that govern a wide range of industries, including energy, power, and transportation.

With well-based knowledge about web, it's not difficult for to find that Shodan can find anything that connects directly to the internet — and if your internet-facing devices aren’t protected, Shodan can tell hackers everything they need to know to break into your network.

Nevertheless, Shodan isn't designed for Hackers. Hackers use similar port-crawling tools to invade internet-connected devices,  like other search engines we are going to mention.

With Simple user interface and public nature, Shodan is crucial resource used by cybersecurity experts to help protect   individuals, enterprises, and even public utilities from cyber attacks. But keep in mind that searching with Shodan is a little more complicated than a basic Google search. 

Users can perform a search using the Shodan search engine based on an IP address, device name, city, and/or a variety of other technical categories. Users can sign up for free accounts, but they are very limited — Shodan limits its free service to only 50 search results.

By the way,  Shodan in fact comes from the pet project of a young computer programmer，John Matherly, who figured out a way to map each device connected to the internet by constantly crawling the web for randomly generated IP addresses, and it's he that eventually developed a search engine to search through his growing database of internet-connected devices. The first Release of Shodan was publish in 2009.

## 1.2How does Shodan Work

Shodan works by requesting connections to every imaginable internet protocol (IP) address on the internet and indexing the information that it gets back from those connections requests.

Shodan crawls the web for devices using a global network of computers and severs that are running 24/7 (Full time)

IP address

An IP address is your device’s digital signature — it’s what allows Google to tailor searches to your location, and it’s what allows all internet-connected devices to communicate with each other (VPNs  hide your IP address so that you can’t be tracked by your ISP or other browser-tracking tools online).

After learning Computer Networks, it's no doubt that Internet-connected devices have specific “ports” that are designed to transmit certain kinds of data. Once you’ve established a device’s IP address, you can establish connections to each of its ports. There are ports for email, ports for browser activity, ports for printers and routers — 65,535 ports in all.

When a port is set to "open", it's available for access, which allowing your printer to establish a connection with your computer. For instance, your computer at the open port, and the printer sends "Banner" of information that contains the information your computer needs to interact with the printer.

Shodan works by "knocking" at every imaginable port possible IP address all day. Some of these ports return nothing, but many of them respond with Banners that contain important metadata about the devices Shodan is requesting a connection with.

The Banner can provide all sorts of identifying information, but here are some of more common fields you will see in the banner:

> - Device name: What your device calls itself online. For example, Samsung Galaxy S21.
> - IP address: A unique code assigned to each device, which allows the device to be identified by servers.
> - Port #: Which protocol your device uses to connect to the web.
> - Organization: Which business owns your “IP space”. For example, your internet service provider, or the business you work for.
> - Location: Your country, city, county, or a variety of other geographic identifiers.

This also determinate the basic usages of Shodan. Of course, some devices even include their default login and password, make and model, and software version, which can all be exploited by hackers.

## 1.3What can you Find

Generally, any device connected to the internet can potentially show up in a Shodan search.

Since the first release in 2009, a large community of hackers and researchers have been cataloging the devices like:

- Wireless moniters
- Internet routers
- Security cameras
- Maritime satellites
- Water treatment facilities
- Traffic light systems
- Pay phones
- Power infrasttructures

Before scared, remember that Shodan merely indexes publicly available information. Getting the information isn't equal to  making the disaster. In case of industrial computers and old SCADA systems, many of them are protected by passwords and other measures such as two-factor authentication, firewalls and etc.

> Whereas, Shodan does reveal just how much of our information is publicly available. If there is an internet-facing webcam whose default logins is not changed, hackers can access it without your knowledge, gaining an easy window into your home. In fact, webcams are one of the most commonly searched terms on [Shodan’s “Explore” page](https://www.shodan.io/explore).This is another reason why it’s so important to use an antivirus program like [Norton](https://www.safetydetectives.com/best-antivirus/norton/) which can flag network vulnerabilities and give you a warning if other apps or users are accessing your webcam or microphone.

## 1.4How to use Shodan Search Engine

As Shodan is designed for professional groups,  performing a search on Shodan isn't as simple as performing  a Bing search. Simply put, it's user interface is not that friendly.

Here we got an example in process of finding Cisco devices in NYC.

```shodan
Cisco city:“New York”
```

Some basic filters.

- `city: find devices in a particular city.`
- `country: find devices in a particular country.`
- `geo: search for specific GPS coordinates.`
- `hostname: find values that match the hostname.`
- `product: search the name of the software or product identified in the banner.`
- `os: search based on operating system.`
- `port: find particular ports that are open.`
- `before/after: find results within a timeframe.`
- `isp:`
- `version:`
- `before/after: before:"11-11-15" #2015-11-11`
- `net:"210.45.240.0/24"`

To get more you can use your ChatGPT or read Shodan Documentation for applying with Python or Rust.



However, in shodan there are different ranks of account.

## 1.5 Application

Shodan is commonly used for identification of potential security issues with their devices.

For more and more internet-connected devices, our chances of falling victim to malicious attack get higher. By identifying all of the devices connected to the internet, displaying what information those devices are sharing with the public, and making it clear how easy that information is to access, Shodan can help users to reinforce their security in a variety of ways.





# LINK

- [Shodan](https://www.shodan.io/)
