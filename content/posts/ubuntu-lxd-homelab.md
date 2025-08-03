---
title: "Build the Ultimate Home Lab with Ubuntu Server and LXD"
date: 2025-08-02T16:00:00-05:00
tags: ["ubuntu", "lxd", "homelab", "sysadmin", "networking", "security", "linux", "containers"]
draft: true
---

For anyone serious about a career in System Administration, Network Engineering, or Cybersecurity, a hands-on lab is not just a nice-to-have—it's essential. Reading books and watching tutorials can only take you so far. True understanding comes from building, breaking, and fixing things. But setting up a lab with physical hardware is expensive, and using traditional VMs can be slow and resource-heavy.

Enter the perfect solution: **Ubuntu Server LTS** as a rock-solid host and **LXD** for creating lightweight, full-system containers.

LXD is not like Docker. While Docker containers are designed to run a single application process, LXD containers are full-fledged system containers. They behave like lightweight virtual machines, each with its own full OS, init system, and network stack. This makes them the perfect tool for simulating entire networks of servers on a single machine, with minimal overhead.

This guide will walk you through setting up a powerful home lab on Ubuntu Server with LXD, complete with three detailed lab scenarios: a web server stack, a network DMZ, and an Intrusion Detection System (IDS).

### Part 1: The Foundation - Installing Ubuntu and LXD

Your lab starts with a single host machine. This can be an old desktop, a NUC, or a server. The more CPU cores and RAM, the more complex your labs can be.

1.  **Install Ubuntu Server:** Download the latest LTS version of [Ubuntu Server](https://ubuntu.com/download/server) and install it on your host machine. The standard installation is perfect.

2.  **Install LXD:** Once Ubuntu is running, install LXD from the Snap store, which is the recommended method.

    ```/dev/null/bash#L1
    sudo snap install lxd --channel=latest/stable
    ```

3.  **Initialize LXD:** Before you can create containers, you need to initialize LXD. This configures storage and networking.

    ```/dev/null/bash#L1
    sudo lxd init
    ```

    You'll be asked a series of questions. For a simple and powerful setup, I recommend the following answers:
    *   `Would you like to use LXD clustering?` -> **no** (unless you have multiple hosts)
    *   `Do you want to configure a new storage pool?` -> **yes**
    *   `Name of the new storage pool` -> **default**
    *   `Name of the storage backend to use` -> **zfs** (It's robust and offers advanced features)
    *   `Create a new ZFS pool?` -> **yes**
    *   `Would you like to use an existing block device?` -> **no** (Let it create a loop device, which is a file-backed disk)
    *   `Size in GB of the new loop device` -> **30** (or more, depending on your disk space)
    *   `Would you like to connect to a MAAS server?` -> **no**
    *   `Would you like to create a new local network bridge?` -> **yes**
    *   `What should the new bridge be called?` -> **lxdbr0** (the default is fine)
    *   `What IPv4 address should be used?` -> **auto**
    *   `What IPv6 address should be used?` -> **auto**
    *   `Would you like LXD to be available over the network?` -> **no**
    *   `Would you like stale cached images to be updated automatically?` -> **yes**
    *   `Would you like a YAML "lxd init" preseed to be printed?` -> **no**

Your foundation is now ready. The `lxdbr0` bridge will act as a virtual switch for your containers, providing DHCP and DNS services.

### Part 2: The Labs

Let's dive in and build three distinct labs. Each demonstrates a different concept and can be built, destroyed, and rebuilt in minutes.

#### Lab A: System Administration - A Load-Balanced Web Stack

**Goal:** Simulate a production web environment with a load balancer distributing traffic to two identical web server backends.

**ASCII Diagram:**
```/dev/null/text#L1-10
+--------------------------+
|  Host Machine            |
|    +-------------------+   |
|    | lxdbr0 (Virtual   |   |
|    |   Switch)         |   |
|    +-------------------+   |
|      |       |       |     |
|      |       |       |     |
+------|-------|-------|-----+
       |       |       |
  +----v---+ +---v----+ +---v----+
  |  lb01  | | web01  | | web02  |
  |(HAProxy)||(Nginx) | |(Nginx) |
  +--------+ +--------+ +--------+
```

**Setup Steps:**

1.  **Launch the Containers:** We'll launch three Ubuntu 22.04 containers.

    ```/dev/null/bash#L1-3
    lxc launch ubuntu:22.04 lb01
    lxc launch ubuntu:22.04 web01
    lxc launch ubuntu:22.04 web02
    ```

2.  **Configure Web Servers:** Get a shell inside `web01` and `web02` to install Nginx and create a unique index page to know which server is responding.

    ```/dev/null/bash#L1-4
    # Do this for both web01 and web02
    lxc exec <container_name> -- bash
    apt update && apt install -y nginx
    echo "<h1>Response from <container_name></h1>" > /var/www/html/index.html
    exit
    ```

3.  **Configure the Load Balancer:** Now set up HAProxy on `lb01`.

    ```/dev/null/bash#L1-15
    lxc exec lb01 -- bash
    apt update && apt install -y haproxy jq
    # Get the IP addresses of the web servers
    WEB01_IP=$(lxc list web01 --format=json | jq -r '.[0].state.network.eth0.addresses[0].address')
    WEB02_IP=$(lxc list web02 --format=json | jq -r '.[0].state.network.eth0.addresses[0].address')

    # Configure HAProxy
    cat <<EOF >> /etc/haproxy/haproxy.cfg

    frontend http_front
       bind *:80
       default_backend http_back

    backend http_back
       balance roundrobin
       server web01 $WEB01_IP:80 check
       server web02 $WEB02_IP:80 check
    EOF

    systemctl restart haproxy
    exit
    ```

4.  **Expose the Lab:** To access the lab from your host machine, forward port 8080 on the host to port 80 on the load balancer container.

    ```/dev/null/bash#L1
    lxc config device add lb01 proxy80 proxy listen=tcp:0.0.0.0:8080 connect=tcp:127.0.0.1:80
    ```

**Experiment:** Open your browser and go to `http://127.0.0.1:8080`. Refresh the page several times. You should see the response alternate between `web01` and `web02`. Now, stop `web01` (`lxc stop web01`) and notice how HAProxy automatically directs all traffic to `web02`.

#### Lab B: Networking - A Secure DMZ

**Goal:** Create an isolated network (a DMZ or "Demilitarized Zone") for a public-facing web server, protected by a dedicated router/firewall container.

**ASCII Diagram:**
```/dev/null/text#L1-15
(Internet / Your LAN)
        |
+-------+----------------+
| Host Machine           |
|  +-----v-----------+   |
|  | lxdbr0          |   |
|  +-----------------+   |
|          |             |
|  +-------v-------+   +--------------------+
|  | router        |   | dmz-net            |
|  | eth0 -> lxdbr0|   | (192.168.50.0/24)  |
|  | eth1 -> dmz-net|   +--------------------+
|  +---------------+             |
|                                |
+--------------------------------+
                                 |
                          +------v------+
                          | web-in-dmz  |
                          +-------------+
```
**Setup Steps:**

1.  **Create a New Network:** First, create a new, isolated virtual switch for our DMZ.

    ```/dev/null/bash#L1
    lxc network create dmz-net ipv4.address=192.168.50.1/24 ipv4.nat=true
    ```

2.  **Launch Containers:** Launch a `router` container connected to *both* networks and a `web-in-dmz` container connected *only* to the new `dmz-net`.

    ```/dev/null/bash#L1-2
    lxc launch ubuntu:22.04 router -n lxdbr0 -n dmz-net
    lxc launch ubuntu:22.04 web-in-dmz -n dmz-net
    ```

3.  **Configure the Router:** We need to enable IP forwarding and set up a simple firewall with UFW.

    ```/dev/null/bash#L1-10
    lxc exec router -- bash
    # Enable IP forwarding
    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
    sysctl -p

    # Install and configure UFW
    apt update && apt install -y ufw
    ufw default deny incoming
    ufw default allow outgoing
    ufw allow ssh     # Allow SSH for management
    ufw allow 80/tcp  # Allow HTTP traffic
    ufw enable
    exit
    ```

4.  **Expose the DMZ:** Forward traffic from the host to the router, which will then forward it to the web server.

    ```/dev/null/bash#L1
    lxc config device add router proxy80 proxy listen=tcp:0.0.0.0:8081 connect=tcp:192.168.50.10:80
    ```
    *(Assuming `web-in-dmz` got the IP `192.168.50.10`. Verify with `lxc list`)*

**Experiment:** Install Nginx on `web-in-dmz`. You should be able to access it from your host via `http://127.0.0.1:8081`. Now, try to ping `web-in-dmz` from another container on the `lxdbr0` network (like `lb01`). It should fail because the `router`'s firewall is blocking it.

#### Lab C: Security - Network Intrusion Detection

**Goal:** Use the DMZ setup to install Suricata, a powerful Network Intrusion and Detection System (IDS/IPS), on our router to monitor all traffic heading to our web server.

**ASCII Diagram:** (Same as the DMZ lab, but with Suricata on the router)
```/dev/null/text#L1-15
(Internet / Your LAN)
        |
+-------+----------------+
| Host Machine           |
|  +-----v-----------+   |
|  | lxdbr0          |   |
|  +-----------------+   |
|          |             |
|  +-------v-------+   +--------------------+
|  | router        |   | dmz-net            |
|  | (Suricata)    |   | (192.168.50.0/24)  |
|  +---------------+   +--------------------+
|                                |
+--------------------------------+
                                 |
                          +------v------+
                          | web-in-dmz  |
                          +-------------+
```
**Setup Steps:**

1.  **Install Suricata:** Get a shell on the `router` container from our previous lab.

    ```/dev/null/bash#L1-3
    lxc exec router -- bash
    apt update && apt install -y suricata
    exit
    ```

2.  **Configure Suricata:** Tell Suricata which network interface to monitor. In our case, it's `eth1`, which connects to the DMZ.

    ```/dev/null/bash#L1-2
    # Find the interface name in /etc/netplan/*.yaml inside the container
    sudo sed -i 's/AF_PACKET_IFACE: eth0/AF_PACKET_IFACE: eth1/' /etc/suricata/suricata.yaml
    ```
3.  **Update Rules and Run:**

    ```/dev/null/bash#L1-3
    lxc exec router -- bash
    suricata-update
    systemctl restart suricata
    exit
    ```

**Experiment:** We need an "attacker" container. Use the `lb01` container from the first lab.
From `lb01`, run a suspicious-looking `curl` command against the web server's public entry point.

```/dev/null/bash#L1-2
# From inside the lb01 container
curl -A "BlackSun" http://<IP_of_router_on_lxdbr0>
```

Now, check the Suricata logs on the `router` container. You should see an alert!

```/dev/null/bash#L1-2
# From inside the router container
tail /var/log/suricata/fast.log
```

You'll find a log entry flagging the "BlackSun" User-Agent, which is a common signature in IDS rulesets.

#### Lab D: System Administration - Centralized Time Server (NTP)

**Goal:** Create a central NTP server for your lab network. All other containers will synchronize their time with this server, ensuring consistent timestamps for logs and services. We will use Chrony, a modern and robust NTP implementation.

**ASCII Diagram:**
```/dev/null/text#L1-10
+-----------------------------+
|  lxdbr0 (Virtual Switch)    |
|                             |
+--^-----------^-----------^--+
   |           |           |
+--v-------+ +--v-------+ +--v-------+
| ntp-svr  | | client01 | | client02 |
|(Chrony Srv)|(Chrony Cli)|(Chrony Cli)|
+----------+ +----------+ +----------+
```

**Setup Steps:**

1.  **Launch Containers:**

    ```/dev/null/bash#L1-3
    lxc launch ubuntu:22.04 ntp-svr
    lxc launch ubuntu:22.04 client01
    lxc launch ubuntu:22.04 client02
    ```

2.  **Configure the NTP Server:**

    ```/dev/null/bash#L1-12
    lxc exec ntp-svr -- bash
    apt update && apt install -y chrony

    # Get the network range of lxdbr0
    SUBNET=$(lxc network get lxdbr0 ipv4.address)/$(lxc network get lxdbr0 ipv4.netmask)

    # Configure chrony to allow clients from the lab network
    cat <<EOF >> /etc/chrony/chrony.conf

    # Allow NTP client access from our LAN.
    allow $SUBNET
    EOF

    systemctl restart chrony
    exit
    ```

3.  **Configure NTP Clients:** Now, point `client01` and `client02` to use `ntp-svr`.

    ```/dev/null/bash#L1-9
    # Get the IP of the ntp-svr
    NTP_SVR_IP=$(lxc list ntp-svr --format=json | jq -r '.[0].state.network.eth0.addresses[0].address')

    # Run these steps on both client01 and client02
    lxc exec <client_name> -- bash
    apt update && apt install -y chrony
    # Comment out the default pool and add our local server
    sed -i 's/^pool/#pool/' /etc/chrony/chrony.conf
    echo "server $NTP_SVR_IP iburst" >> /etc/chrony/chrony.conf
    systemctl restart chrony
    exit
    ```

**Experiment:** On one of the clients, check the NTP sources.

```/dev/null/bash#L1-2
lxc exec client01 -- chronyc sources
```
After a minute, you should see an entry for your `ntp-svr`'s IP address, marked with a `^*`, indicating it's the primary sync source. This confirms your lab has a centralized and consistent time source.

#### Lab E: System Administration - Network File Sharing (NFS & Samba)

**Goal:** Set up a central file server container that shares directories using both NFS (for Linux clients) and Samba (for Windows/macOS/Linux clients).

**ASCII Diagram:**
```/dev/null/text#L1-12
+-----------------------------------+
|      lxdbr0 (Virtual Switch)      |
+--^-----------------^-----------^--+
   |                 |           |
+--v-----------+ +---v------+ +--v-------+
| fileserver   | | nfs-cli  | | smb-cli  |
|(NFS + Samba) | |(Linux)   | |(Any OS)  |
+--------------+ +----------+ +----------+
```
**Setup Steps:**

1.  **Launch Containers:** We need a server and two clients.

    ```/dev/null/bash#L1-3
    lxc launch ubuntu:22.04 fileserver
    lxc launch ubuntu:22.04 nfs-client
    lxc launch ubuntu:22.04 smb-client
    ```

2.  **Configure the File Server:**

    ```/dev/null/bash#L1-26
    lxc exec fileserver -- bash
    apt update && apt install -y nfs-kernel-server samba

    # Create directories to share
    mkdir -p /srv/nfs/shared
    mkdir -p /srv/samba/public

    # Set permissions
    chown nobody:nogroup /srv/nfs/shared
    chmod 777 /srv/samba/public

    # -- Configure NFS Exports --
    echo "/srv/nfs/shared    *(rw,sync,no_subtree_check)" >> /etc/exports
    exportfs -a
    systemctl restart nfs-kernel-server

    # -- Configure Samba Share --
    cat <<EOF >> /etc/samba/smb.conf
    [public]
       comment = Public Storage
       path = /srv/samba/public
       read only = no
       browsable = yes
       guest ok = yes
    EOF

    systemctl restart smbd
    # Create a test file in each share
    echo "Hello from NFS!" > /srv/nfs/shared/hello.txt
    echo "Hello from Samba!" > /srv/samba/public/hello.txt
    exit
    ```

3.  **Configure the NFS Client:**

    ```/dev/null/bash#L1-9
    # Get fileserver IP
    FILESERVER_IP=$(lxc list fileserver --format=json | jq -r '.[0].state.network.eth0.addresses[0].address')

    lxc exec nfs-client -- bash
    apt update && apt install -y nfs-common
    mkdir /mnt/nfs
    mount $FILESERVER_IP:/srv/nfs/shared /mnt/nfs
    # Check if it worked
    cat /mnt/nfs/hello.txt
    exit
    ```

4.  **Configure the Samba Client:**

    ```/dev/null/bash#L1-10
    # Get fileserver IP
    FILESERVER_IP=$(lxc list fileserver --format=json | jq -r '.[0].state.network.eth0.addresses[0].address')

    lxc exec smb-client -- bash
    apt update && apt install -y smbclient
    # List shares anonymously
    smbclient -L //$FILESERVER_IP -N
    # Connect and download the file
    smbclient //$FILESERVER_IP/public -N -c 'get hello.txt'
    cat hello.txt
    exit
    ```

**Experiment:** Create a file on the `nfs-client`'s mount point (`echo "NFS rocks" > /mnt/nfs/test.txt`). Now, log into the `fileserver` and verify the file exists in `/srv/nfs/shared`. This setup is the backbone for centralized storage in any small to medium network.

### Conclusion

You have now built three foundational labs that can be extended in countless ways. The beauty of LXD is its speed and low resource usage. You can tear down these labs and rebuild them in minutes without any risk to your host system.

This setup provides an unparalleled environment for learning:
*   **System Administration:** Practice service configuration, scripting, and automation.
*   **Networking:** Master concepts like routing, firewalling, NAT, and network segmentation.
*   **Security:** Learn how to deploy monitoring tools, analyze traffic, and test defenses.

So go ahead—add a database container, build a VPN gateway, or try to hack your own web server. Your ultimate home lab awaits.
