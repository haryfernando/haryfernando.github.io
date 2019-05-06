---
layout: post
lang: en
title: Red Hat Cluster
---
First of all, `the disclaimer`, hehe :

This is solely my humble notes, and I blog this just to make documentation that might be useful for later.

<!-- more -->

### Redhat Cluster Suite or High Availability Add-on

Red Hat Cluster Suite, from the big picture, is the same as [VCS](http://hary.my.id/blog/vcs) (Veritas Cluster Service), a `High Availability` software, but of course they are different with each other, mainly in the configuration (I found it difficult to configure the Redhat Cluster).

We are planning on using load balancing cluster. 

Load balancing cluster send network request to multiple nodes to balance the request load. If a node in a cluster got a problem, the load balancing software detects the failure and redirect the request to other nodes.
The outside of the cluster would not know that theres a problem in cluster.

### Installing Red Hat Cluster Suite.

We use Centos 6 for testing, but basically its the same as RHEL, even the packages and pretty much everything else.

The minimum packages for Red Hat Cluster Suite (RHCS) are: 

1. cman, and 
2. rgmanager

#### Install Centos OS

Download Centos installer ISO from [www.centos.org](http://www.centos.org).

Now, we assume that you already install the Centos 6.x (the latest is 6.4).

After the OS is installed, we install the `High Availability` packages (if not yet installed during OS installation).

    yum groupinstall "High Availiability"
    yum groupinstall "High Availiability Management"

#### Preparation

Make sure each node can communicate with each other.

We can define other node in /etc/hosts :

    # vi /etc/hosts
    192.168.1.170   node170
    192.168.1.174   node174

Disable firewall :

    # system-config-firewall

Disable selinux :

    # vi /etc/selinux/config

change the line contain `SELINUX=` becomes :

    SELINUX=disabled

We might need to `reboot` the OS now to apply selinux setting.

Set static IP

We dont use NetworkManager which is default in Centos OS.

    # vi /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE="eth0"              #depend on your interface
    HWADDR=XX:XX:XX:XX:XX:XX   #depend on your hardware mac addr
    NM_CONTROLLED="no"
    ONBOOT="yes"
    TYPE=Ethernet
    BOOTPROTO=static
    IPADDR=192.168.1.170       #depend on your ip addresss
    PREFIX=24                  #depend on your mask
    GATEWAY=192.168.1.1        #depend on your gateway
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=no
    DNS1=192.168.1.1           #depend on your dns server
    DNS2=8.8.8.8               #depend on your second dns server

Apply static ip :

    # service network restart

Disable Network Manager :

    # chkconfig NetworkManager off
    # service NetworkManager stop

Disable some services at boot :

    # chkconfig iptables off
    # chkconfig ip6tables off

Enable cluster service at boot :

    # chkconfig ricci on
    # chkconfig cman on
    # chkconfig rgmanager on
    # chkconfig modclusterd on

Stop some services that are not needed now :

    # service iptables stop
    # service ip6tables stop

Start service for clustering now :

    # service ricci start

Set password for user ricci :

    # passwd ricci

Enable luci, a web based cluster configuration :

    # chkconfig luci on
    # service luci start

Now we can access web based cluster configuration at [https://your-server-ip-addr:8084](#), with user ricci and password is defined above.

Create skeleton configuration.
For example we will name our cluster TEST1 with two nodes:

    # ccs_tool create -2 TEST1

Then we can manually edit `/etc/cluster/cluster.conf` and alter some changes.
After finished editing, do not forget to increment the field `config_version` by 1 and then validate the configuration file using:

    # ccs_config_validate

We need to copy this configuration to other node, so they will have identical config :

    # ccs -f /etc

To verify that all nodes have the identical configuration execute :

    # ccs -h host --checkconf

Finally start the cluster :

    # service cman start
    # service rgmanager start
    # service modclusterd start

To see the current status of cluster:

    # clustat

To see nodes in cluster:

    # cman_tool nodes

### Fence

Fence is very important in clustering. Fence is used to ensure that only a single node a time is using the data.

To see list of available fence :

    # ccs -h host --lsfenceopts

To see detail about a fence, eg fence_apc

    # ccs -h host --lsfenceopts fence_apc

To stop cluster services on all nodes (and chkconfig services off)

    # ccs -h host --stopall

To start cluster services on all nodes (and chkconfig services on)

    # ccs -h host --startall


Notes : This article is still not finish yet, maybe later we will add more. 

