---
layout: post
title: "How To Open A Port In CentOS / RHEL 7"
description: "This tutorial will walk you through opening a port in the default firewall in CentOS 7, firewalld."
comments: true
keywords: "centos"
---
##Open Specific Port

Opening a port in firewalld is fairly straightforward, in the below example we allow traffic in from any source IP address to TCP port 100. First we modify the persistent configuration, then we reload firewall-cmd to load this change into the running configuration.

>[root@centos7 ~]# firewall-cmd --permanent --add-port=100/tcp
>success
>[root@centos7 ~]# firewall-cmd --reload
>success
>If the --permanent flag is not specified, this will only change the running configuration but will not be saved.

We can check the ports that are opened in the current default zone with ‘--list-ports’.

>[root@centos7 ~]# firewall-cmd --list-ports
>100/tcp
As expected we see that TCP port 100 is open.

Should we wish to remove a port, we can use ‘--remove-port=’ instead.

We can also open a range of ports in the same way.

>[root@centos7 ~]# firewall-cmd --permanent --add-port=200-300/tcp
>success
##Open Predefined Service

Rather than manually specifying a port number to allow through the firewall, we can make use of a bunch of predefined services which may be easier. For example instead of opening TCP port 80, we can use the ‘http’ service.

>[root@centos7 ~]# firewall-cmd --permanent --add-service=http
>success
>[root@centos7 ~]# firewall-cmd --reload
>success
Now if we list the services that are accepted through the firewall, we will see http listed along with ssh and dhcpv6-client, which are allowed through by default.

>[root@centos7 ~]# firewall-cmd --list-services
>dhcpv6-client http ssh
This is a predefined service and can be found as an XML file in the /usr/lib/firewalld/services/ directory. Here’s what the http service we just used looks like.

```
[root@centos7 ~]# cat /usr/lib/firewalld/services/http.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>WWW (HTTP)</short>
  <description>HTTP is the protocol used to serve Web pages. If you plan to make your Web server publicly available, enable this option. This option is not required for viewing pages locally or developing Web pages.</description>
  <port protocol="tcp" port="80"/>
</service>
```
We can create custom services by copying one of these into the /etc/firewalld/services/ directory and then customizing it. The services in the /usr/lib/firewalld/services/ directory should NOT be modified, changes should be copied into /etc/firewalld/services/ followed by a reload of firewall-cmd to pick up the changes.

##Services Or Manual Ports?

Why would we want to use services if we can just specify the port? Modules can be specified in a service, for example samba.xml loads the module “nf_conntrack_netbios_ns” for us when it’s enabled, along with four different ports which is a lot easier than doing all of this ourselves as we don’t need to memorize all of the ports required for a service.

Still not a fan of firewalld? Don’t worry, you can always install ifconfig in CentOS 7 instead, however note that this is considered deprecated.