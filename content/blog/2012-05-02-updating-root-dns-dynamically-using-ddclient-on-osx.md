---
title: Updating Root DNS Dynamically Using ddclient on OS X Server
date: 2012-05-02
author: Carl Hoyer
description: HowTo use ddclient to update root dns records for your own domain using ddclient on OS X.
keywords: sysadmin, dns, dyndns, ddns, ddclient, easydns, howto, mamp, diymacserver
tags: sysadmin
asset_path: /2012/05/02/updating-root-dns-dynamically-using-ddclient-on-osx/
---

I host a number of small websites on a [custom built OS X MAMP stack](http://diymacserver.com) that runs off an internet connection that has a dynamically assigned public IP address. My DNS service provider of choice is [EasyDNS](https://web.easydns.com/) and while the remainder of this post is specific to EasyDNS, you should be able to apply the general gist of this post to other DNS services.

Normally the fact that your public IP is dynamic wouldn't be too much of an issue, simply use one of the plethora of [Dynamic DNS (DDNS)](http://en.wikipedia.org/wiki/Dynamic_DNS) [services that now exist](http://dnslookup.me/dynamic-dns/), create a CNAME in [DNS](http://en.wikipedia.org/wiki/Domain_Name_System) for one of your domain hosts and *Bob's your uncle*.

However, if you want to support a [no-www](http://no-www.org/) configuration for your domain (e.g. [http://pixolium.ca](http://pixolium.ca)), you need a way to dynamically update the root DNS entry. The majority of DNS servers and DNS hosting services DO NOT allow you to CNAME your root domain. While not specifically forbidden in the [DNS RFC](http://www.faqs.org/rfcs/rfc1034.html), it is not recommended.


### [No-WWW](http://no-www.org/), No Problem. ddclient to the Rescue.

The missing glue to update your domain's root record is provided by [ddclient](http://sourceforge.net/apps/trac/ddclient), *a Perl client used to update dynamic DNS entries for accounts on 'Dynamic DNS Network Services' free DNS service which currently supports a lot of different routers and a few different services*.

On OS X, the least painful way to install *ddclient* is via [Homebrew](http://mxcl.github.com/homebrew/) using the command:

	brew install ddclient


### EasyDNS setup

[EasyDNS](https://web.easydns.com/) supports dynamic records which are updatable via a web based API. ddclient natively supports updating EasyDNS dynamic records via the API.

To enable EasyDNS dynamic records, within your EasyDNS control panel:

#### Administer Domain DNS

Choose the domain you wish to administer and select *DNS*

<img src='/2012/05/02/updating-root-dns-dynamically-using-ddclient-on-osx/easydns-administer-dns.png'>

#### Edit Dynamic Records

Locate the section entitled *dynamic records*. Click the wrench icon to edit.

<img src='/2012/05/02/updating-root-dns-dynamically-using-ddclient-on-osx/easydns-dynamic-records.png'>

#### Enable Dynamic Authentication

In the dynamic authentication section **enable** *DYN Authentication Token*. Copy the token string provided to you will need this in your ddclient config.

<img src='/2012/05/02/updating-root-dns-dynamically-using-ddclient-on-osx/easydns-dyn-auth-token.png'>

### The ddclient Configuration File

The EasyDNS specific ddclient config file looks like this:

```sh
###################################################################
## ddclient configuration file for EasyDNS
##
## Updates easyDNS for listed domains enabling use of no-www.
## Scheduled via launchd daemon.
##
##
## http://sourceforge.net/projects/ddclient/
##
## /usr/local/etc/ddclient/ddclient.conf
###################################################################

pid=/usr/local/var/run/ddclient/pid		# record PID in file.
ssl=yes
protocol=easydns
use=web

## easyDNS -- mysuperawesomedomain.com
server=api.cp.easydns.com, login='<your easydns username>', password='<your easydns DYN Authentication Token>', mysuperawesomedomain.com
```


### Automatically Start ddclient and Keep it Running

To make sure ddclient does what it supposed to do, [create a launchd job](http://developer.apple.com/library/mac/#documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html) in /Library/LaunchDaemons with the filename `org.ddclient.plist` with the following contents:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>org.ddclient</string>
  <key>OnDemand</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/Cellar/ddclient/3.8.0/sbin/ddclient</string>
    <string>-file</string>
    <string>/usr/local/etc/ddclient/ddclient.conf</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>StartCalendarInterval</key>
  <dict>
    <key>Minute</key>
    <integer>0</integer>
  </dict>
  <key>WatchPaths</key>
  <array>
    <string>/usr/local/etc/ddclient</string>
  </array>
  <key>WorkingDirectory</key>
  <string>/usr/local/etc/ddclient</string>
</dict>
</plist>
```

To load and activate the .plist file use the following command:

	sudo launchctl load /Library/LaunchDaemons/org.ddclient.plist

To verify that it loaded correctly use the following command:

	sudo launchctl list

If you see `org.ddclient` in the list you are good to go.

### Further Reference
[RFC 1912: Common DNS Operational and Configuration Errors](http://tools.ietf.org/html/rfc1912)
