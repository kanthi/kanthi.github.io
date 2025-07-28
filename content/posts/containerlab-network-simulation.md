+++
title = "Containerlab: Nokia's Revolutionary Network Simulation Platform for Modern Network Engineers"
date = "2025-07-28T12:00:00+05:30"
author = ""
draft = true
authorTwitter = "" #do not include @
cover = ""
tags = ["containerlab", "nokia", "networking", "simulation", "docker", "srl", "ubuntu", "network-automation"]
keywords = ["containerlab", "nokia", "network simulation", "docker networking", "SR Linux", "network labs", "network automation", "ubuntu server"]
description = "Master Containerlab, Nokia's powerful network simulation platform that enables realistic network topologies using containers. Complete guide with practical lab examples."
showFullContent = false
readingTime = false
hideComments = false
+++

Network simulation and testing have traditionally required expensive hardware or complex virtualization setups. Enter **Containerlab** - Nokia's open-source network simulation platform that's revolutionizing how network engineers learn, test, and validate network configurations using lightweight containers.

## What is Containerlab?

Containerlab is a CLI tool that orchestrates and manages container-based networking labs. Developed by Nokia, it allows network engineers to build complex network topologies using containerized Network Operating Systems (NOS) like Nokia SR Linux, Arista cEOS, Cisco XRd, and many others.

Unlike traditional network simulators that require heavy virtual machines, Containerlab leverages Docker containers to create realistic network environments that are:
- **Lightweight**: Containers use fewer resources than VMs
- **Fast**: Labs spin up in seconds, not minutes
- **Realistic**: Uses actual network operating systems
- **Scalable**: Can simulate large topologies on modest hardware

## Why Containerlab is Game-Changing for Network Engineers

### 1. **Real Network Operating Systems**
Instead of simplified simulators, Containerlab runs actual network operating systems in containers, providing authentic CLI experiences and feature sets.

### 2. **Rapid Prototyping**
Network engineers can quickly test configurations, validate designs, and troubleshoot issues without physical hardware.

### 3. **CI/CD Integration**
Perfect for network automation testing, allowing configuration validation in continuous integration pipelines.

### 4. **Cost-Effective Learning**
Students and professionals can access enterprise-grade network operating systems without expensive hardware.

### 5. **Reproducible Environments**
Lab topologies are defined in YAML files, making them version-controllable and shareable.

## Installation on Ubuntu Server

### Prerequisites
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose-plugin -y

# Logout and login to apply group changes
```

### Installing Containerlab
```bash
# Method 1: Using the install script (recommended)
bash -c "$(curl -sL https://get.containerlab.dev)"

# Method 2: Download binary directly
sudo curl -sL https://github.com/srl-labs/containerlab/releases/latest/download/containerlab_linux_amd64.tar.gz | \
sudo tar -xzf - -C /usr/local/bin

# Method 3: Using package manager
echo "deb [trusted=yes] https://apt.fury.io/netdevops/ /" | \
sudo tee -a /etc/apt/sources.list.d/netdevops.list
sudo apt update && sudo apt install containerlab

# Verify installation
containerlab version
```

### Post-Installation Setup
```bash
# Enable IP forwarding
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Install additional tools
sudo apt install -y bridge-utils net-tools tcpdump wireshark-common

# Pull common network OS images
docker pull ghcr.io/nokia/srlinux:latest
docker pull ceos:latest  # If you have access to Arista cEOS
```

## Basic Containerlab Concepts

### Lab Definition File
Containerlab uses YAML files to define network topologies:

```yaml
# basic-lab.yml
name: basic-lab
topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
    srl2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
  links:
    - endpoints: ["srl1:e1-1", "srl2:e1-1"]
```

### Basic Commands
```bash
# Deploy a lab
containerlab deploy -t lab.yml

# List running labs
containerlab inspect

# Connect to a node
docker exec -it clab-basic-lab-srl1 sr_cli

# Destroy a lab
containerlab destroy -t lab.yml
```

## Comprehensive Lab Examples

### Lab 1: Basic Two-Node SR Linux Lab

Create a simple two-node topology to understand basic concepts:

```
┌─────────┐    e1-1 ┌─────────┐
│  leaf1  ├─────────┤  leaf2  │
│         │  10.1.1 │         │
└─────────┘    /30  └─────────┘
```

Create a simple two-node topology to understand basic concepts:

```yaml
# labs/basic-srl-lab.yml
name: basic-srl-lab
topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/leaf1.cfg
    leaf2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/leaf2.cfg
  links:
    - endpoints: ["leaf1:e1-1", "leaf2:e1-1"]
```

Configuration files:
```bash
# configs/leaf1.cfg
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.1.1.1/30
set / network-instance default interface ethernet-1/1.0
```

```bash
# configs/leaf2.cfg
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.1.1.2/30
set / network-instance default interface ethernet-1/1.0
```

Deploy and test:
```bash
# Deploy the lab
containerlab deploy -t labs/basic-srl-lab.yml

# Connect to leaf1
docker exec -it clab-basic-srl-lab-leaf1 sr_cli

# Test connectivity
A:leaf1# ping 10.1.1.2
```

### Lab 2: Three-Tier Data Center Fabric

Create a realistic data center topology with spines and leaves:

```
                    ┌─────────┐    ┌─────────┐
                    │ spine1  │    │ spine2  │
                    │ AS65000 │    │ AS65000 │
                    └────┬────┘    └────┬────┘
                         │              │
           ┌─────────────┼──────────────┼─────────────┐
           │             │              │             │
           │             │              │             │
    ┌──────▼──┐   ┌──────▼──┐    ┌──────▼──┐   ┌──────▼──┐
    │  leaf1  │   │  leaf2  │    │  leaf3  │   │  leaf4  │
    │ AS65001 │   │ AS65002 │    │ AS65003 │   │ AS65004 │
    └────┬────┘   └────┬────┘    └────┬────┘   └────┬────┘
         │             │              │             │
    ┌────▼────┐   ┌────▼────┐    ┌────▼────┐   ┌────▼────┐
    │ server1 │   │ server2 │    │ server3 │   │ server4 │
    │192.168.1│   │192.168.2│    │192.168.3│   │192.168.4│
    └─────────┘   └─────────┘    └─────────┘   └─────────┘
```

```yaml
# labs/dc-fabric.yml
name: dc-fabric
topology:
  nodes:
    # Spine switches
    spine1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd3
      startup-config: configs/spine1.cfg
    spine2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd3
      startup-config: configs/spine2.cfg

    # Leaf switches
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/leaf1.cfg
    leaf2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/leaf2.cfg
    leaf3:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/leaf3.cfg

    # Servers
    server1:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 192.168.1.10/24 dev eth1
        - ip route add default via 192.168.1.1
    server2:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 192.168.2.10/24 dev eth1
        - ip route add default via 192.168.2.1
    server3:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 192.168.3.10/24 dev eth1
        - ip route add default via 192.168.3.1

  links:
    # Spine to Leaf connections
    - endpoints: ["spine1:e1-1", "leaf1:e1-49"]
    - endpoints: ["spine1:e1-2", "leaf2:e1-49"]
    - endpoints: ["spine1:e1-3", "leaf3:e1-49"]
    - endpoints: ["spine2:e1-1", "leaf1:e1-50"]
    - endpoints: ["spine2:e1-2", "leaf2:e1-50"]
    - endpoints: ["spine2:e1-3", "leaf3:e1-50"]

    # Server connections
    - endpoints: ["leaf1:e1-1", "server1:eth1"]
    - endpoints: ["leaf2:e1-1", "server2:eth1"]
    - endpoints: ["leaf3:e1-1", "server3:eth1"]
```

BGP configuration for spine1:
```bash
# configs/spine1.cfg
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.1.0/31
set / interface ethernet-1/2 admin-state enable
set / interface ethernet-1/2 subinterface 0 ipv4 address 10.0.1.2/31
set / interface ethernet-1/3 admin-state enable
set / interface ethernet-1/3 subinterface 0 ipv4 address 10.0.1.4/31

set / network-instance default interface ethernet-1/1.0
set / network-instance default interface ethernet-1/2.0
set / network-instance default interface ethernet-1/3.0

set / routing-policy policy accept-all default-action policy-result accept

set / network-instance default protocols bgp autonomous-system 65000
set / network-instance default protocols bgp router-id 1.1.1.1
set / network-instance default protocols bgp group leafs export-policy accept-all
set / network-instance default protocols bgp group leafs import-policy accept-all
set / network-instance default protocols bgp neighbor 10.0.1.1 peer-group leafs
set / network-instance default protocols bgp neighbor 10.0.1.1 peer-as 65001
set / network-instance default protocols bgp neighbor 10.0.1.3 peer-group leafs
set / network-instance default protocols bgp neighbor 10.0.1.3 peer-as 65002
set / network-instance default protocols bgp neighbor 10.0.1.5 peer-group leafs
set / network-instance default protocols bgp neighbor 10.0.1.5 peer-as 65003
```

### Lab 3: Multi-Vendor Network Lab

Demonstrate interoperability between different network operating systems:

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ client1 │    │   SRL   │    │  cEOS   │    │   FRR   │
│ Alpine  ├────┤ Nokia   ├────┤ Arista  ├────┤ Linux   │
└─────────┘    │         │    │         │    │         │
               └─────────┘    └─────────┘    └────┬────┘
                                                  │
                                            ┌─────▼─────┐    ┌─────────┐
                                            │   BIRD    │    │ client2 │
                                            │  Router   ├────┤ Alpine  │
                                            └───────────┘    └─────────┘
```

```yaml
# labs/multi-vendor.yml
name: multi-vendor
topology:
  nodes:
    # Nokia SR Linux
    srl:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2

    # Arista cEOS (if available)
    eos:
      kind: ceos
      image: ceos:latest

    # FRRouting
    frr:
      kind: linux
      image: frrouting/frr:latest
      exec:
        - sysctl -w net.ipv4.ip_forward=1
        - sysctl -w net.ipv6.conf.all.forwarding=1

    # Linux router with BIRD
    bird:
      kind: linux
      image: pierky/bird:latest

    # Client nodes
    client1:
      kind: linux
      image: alpine:latest
    client2:
      kind: linux
      image: alpine:latest

  links:
    - endpoints: ["srl:e1-1", "eos:eth1"]
    - endpoints: ["eos:eth2", "frr:eth1"]
    - endpoints: ["frr:eth2", "bird:eth1"]
    - endpoints: ["srl:e1-2", "client1:eth1"]
    - endpoints: ["bird:eth2", "client2:eth1"]
```

### Lab 4: EVPN-VXLAN Data Center

Advanced lab demonstrating EVPN-VXLAN overlay:

```
                    ┌─────────┐    ┌─────────┐
                    │   RR1   │────│   RR2   │  Route Reflectors
                    │ BGP RR  │    │ BGP RR  │  (Underlay + EVPN)
                    └────┬────┘    └────┬────┘
                         │              │
           ┌─────────────┼──────────────┼─────────────┐
           │             │              │             │
    ┌──────▼──┐   ┌──────▼──┐    ┌──────▼──┐
    │  VTEP1  │   │  VTEP2  │    │  VTEP3  │  VXLAN Tunnel
    │ Leaf    │   │ Leaf    │    │ Leaf    │  Endpoints
    └────┬────┘   └────┬────┘    └────┬────┘
         │             │              │
    ┌────▼────┐   ┌────▼────┐    ┌────▼────┐
    │ host-a1 │   │ host-a2 │    │ host-b1 │
    │VLAN 100 │   │VLAN 100 │    │VLAN 200 │
    │10.1.1.10│   │10.1.1.20│    │10.2.2.10│
    └─────────┘   └─────────┘    └─────────┘
```

```yaml
# labs/evpn-vxlan.yml
name: evpn-vxlan
topology:
  nodes:
    # Route Reflectors
    rr1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd3
      startup-config: configs/rr1.cfg
    rr2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd3
      startup-config: configs/rr2.cfg

    # VTEP Leaves
    vtep1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/vtep1.cfg
    vtep2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/vtep2.cfg
    vtep3:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/vtep3.cfg

    # Hosts in different VLANs
    host-a1:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.1.1.10/24 dev eth1
        - ip route add default via 10.1.1.1
    host-a2:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.1.1.20/24 dev eth1
        - ip route add default via 10.1.1.1
    host-b1:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.2.2.10/24 dev eth1
        - ip route add default via 10.2.2.1

  links:
    # Underlay connections
    - endpoints: ["rr1:e1-1", "vtep1:e1-49"]
    - endpoints: ["rr1:e1-2", "vtep2:e1-49"]
    - endpoints: ["rr1:e1-3", "vtep3:e1-49"]
    - endpoints: ["rr2:e1-1", "vtep1:e1-50"]
    - endpoints: ["rr2:e1-2", "vtep2:e1-50"]
    - endpoints: ["rr2:e1-3", "vtep3:e1-50"]
    - endpoints: ["rr1:e1-10", "rr2:e1-10"]

    # Host connections
    - endpoints: ["vtep1:e1-1", "host-a1:eth1"]
    - endpoints: ["vtep2:e1-1", "host-a2:eth1"]
    - endpoints: ["vtep3:e1-1", "host-b1:eth1"]
```

### Lab 5: MPLS Service Provider Core

Simulate a service provider MPLS network with PE and P routers:

```
                    ┌─────────┐
                    │   P1    │  Provider Core
                    │ Router  │  (MPLS LSR)
                    └────┬────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
    ┌──────▼──┐   ┌──────▼──┐   ┌──────▼──┐
    │  PE1    │   │   P2    │   │  PE2    │
    │Provider │   │Provider │   │Provider │
    │ Edge    │   │ Core    │   │ Edge    │
    └────┬────┘   └────┬────┘   └────┬────┘
         │             │             │
    ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
    │  CE1    │   │  CE2    │   │  CE3    │
    │Customer │   │Customer │   │Customer │
    │ Edge    │   │ Edge    │   │ Edge    │
    └─────────┘   └─────────┘   └─────────┘
```

```yaml
# labs/mpls-sp-core.yml
name: mpls-sp-core
topology:
  nodes:
    # Provider Core Router
    p1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd3
      startup-config: configs/p1.cfg

    # Provider Edge Routers
    pe1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/pe1.cfg
    pe2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/pe2.cfg

    # Provider Router
    p2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/p2.cfg

    # Customer Edge Routers
    ce1:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/ce1.conf
    ce2:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/ce2.conf
    ce3:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/ce3.conf

  links:
    # Provider network core
    - endpoints: ["p1:e1-1", "pe1:e1-49"]
    - endpoints: ["p1:e1-2", "p2:e1-49"]
    - endpoints: ["p1:e1-3", "pe2:e1-49"]
    - endpoints: ["p2:e1-1", "pe1:e1-50"]
    - endpoints: ["p2:e1-2", "pe2:e1-50"]

    # Customer connections
    - endpoints: ["pe1:e1-1", "ce1:eth1"]
    - endpoints: ["p2:e1-10", "ce2:eth1"]
    - endpoints: ["pe2:e1-1", "ce3:eth1"]
```

### Lab 6: Kubernetes Network Simulation

Simulate Kubernetes networking with CNI integration:

```
                    ┌─────────────────────────┐
                    │     Control Plane       │
                    │   ┌─────────────────┐   │
                    │   │  kube-master    │   │
                    │   │   (etcd, api)   │   │
                    │   └─────────────────┘   │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
            ┌───────▼───┐ ┌──────▼───┐ ┌──────▼───┐
            │  worker1  │ │ worker2  │ │ worker3  │
            │           │ │          │ │          │
            │ ┌───────┐ │ │┌───────┐ │ │┌───────┐ │
            │ │ pod-a │ │ ││ pod-b │ │ ││ pod-c │ │
            │ └───────┘ │ │└───────┘ │ │└───────┘ │
            └───────────┘ └──────────┘ └──────────┘
```

```yaml
# labs/k8s-network.yml
name: k8s-network
topology:
  nodes:
    # Kubernetes Master
    master:
      kind: linux
      image: kindest/node:v1.28.0
      exec:
        - kubeadm init --pod-network-cidr=10.244.0.0/16
      ports:
        - "6443:6443"
        - "8080:8080"

    # Worker Nodes
    worker1:
      kind: linux
      image: kindest/node:v1.28.0
      exec:
        - sysctl -w net.ipv4.ip_forward=1
    worker2:
      kind: linux
      image: kindest/node:v1.28.0
      exec:
        - sysctl -w net.ipv4.ip_forward=1
    worker3:
      kind: linux
      image: kindest/node:v1.28.0
      exec:
        - sysctl -w net.ipv4.ip_forward=1

    # Network switch (simulating ToR)
    tor-switch:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2

    # Load balancer
    lb:
      kind: linux
      image: haproxy:latest
      binds:
        - haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg

  links:
    - endpoints: ["tor-switch:e1-1", "master:eth1"]
    - endpoints: ["tor-switch:e1-2", "worker1:eth1"]
    - endpoints: ["tor-switch:e1-3", "worker2:eth1"]
    - endpoints: ["tor-switch:e1-4", "worker3:eth1"]
    - endpoints: ["tor-switch:e1-10", "lb:eth1"]
```

### Lab 7: SD-WAN Simulation

Simulate Software-Defined WAN with multiple sites:

```
    Site A (HQ)              Site B (Branch)           Site C (Branch)
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│  ┌───────────┐  │      │  ┌───────────┐  │      │  ┌───────────┐  │
│  │  vEdge-A  │  │      │  │  vEdge-B  │  │      │  │  vEdge-C  │  │
│  │ (SD-WAN)  │  │      │  │ (SD-WAN)  │  │      │  │ (SD-WAN)  │  │
│  └─────┬─────┘  │      │  └─────┬─────┘  │      │  └─────┬─────┘  │
│        │        │      │        │        │      │        │        │
│  ┌─────▼─────┐  │      │  ┌─────▼─────┐  │      │  ┌─────▼─────┐  │
│  │   LAN-A   │  │      │  │   LAN-B   │  │      │  │   LAN-C   │  │
│  │10.1.0.0/24│  │      │  │10.2.0.0/24│  │      │  │10.3.0.0/24│  │
│  └───────────┘  │      │  └───────────┘  │      │  └───────────┘  │
└─────────────────┘      └─────────────────┘      └─────────────────┘
         │                        │                        │
         └────────────────────────┼────────────────────────┘
                                  │
                        ┌─────────▼─────────┐
                        │    vController    │
                        │   (Orchestrator)  │
                        │   ┌───────────┐   │
                        │   │  vManage  │   │
                        │   │  vSmart   │   │
                        │   │  vBond    │   │
                        │   └───────────┘   │
                        └───────────────────┘
```

```yaml
# labs/sdwan-simulation.yml
name: sdwan-simulation
topology:
  nodes:
    # SD-WAN Controller Components
    vmanage:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y python3 python3-pip
        - pip3 install flask requests
      ports:
        - "8443:8443"

    vsmart:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y python3 python3-pip bird2

    vbond:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y python3 python3-pip

    # Site A (Headquarters)
    vedge-a:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/vedge-a.conf
    lan-a:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.1.0.100/24 dev eth1
        - ip route add default via 10.1.0.1

    # Site B (Branch Office)
    vedge-b:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/vedge-b.conf
    lan-b:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.2.0.100/24 dev eth1
        - ip route add default via 10.2.0.1

    # Site C (Branch Office)
    vedge-c:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/vedge-c.conf
    lan-c:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.3.0.100/24 dev eth1
        - ip route add default via 10.3.0.1

    # Internet/Transport Network
    isp-router:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2

  links:
    # Controller connections
    - endpoints: ["vmanage:eth1", "isp-router:e1-10"]
    - endpoints: ["vsmart:eth1", "isp-router:e1-11"]
    - endpoints: ["vbond:eth1", "isp-router:e1-12"]

    # Site connections to transport
    - endpoints: ["vedge-a:eth0", "isp-router:e1-1"]
    - endpoints: ["vedge-b:eth0", "isp-router:e1-2"]
    - endpoints: ["vedge-c:eth0", "isp-router:e1-3"]

    # LAN connections
    - endpoints: ["vedge-a:eth1", "lan-a:eth1"]
    - endpoints: ["vedge-b:eth1", "lan-b:eth1"]
    - endpoints: ["vedge-c:eth1", "lan-c:eth1"]
```

### Lab 8: Network Security Lab

Simulate network security scenarios with firewalls and IDS:

```
    DMZ Zone              Internal Zone           Management Zone
┌─────────────┐      ┌─────────────────┐      ┌─────────────────┐
│             │      │                 │      │                 │
│ ┌─────────┐ │      │  ┌───────────┐  │      │  ┌───────────┐  │
│ │Web Server│ │      │  │File Server│  │      │  │   SIEM    │  │
│ │   DMZ   │ │      │  │    LAN    │  │      │  │ Security  │  │
│ └─────────┘ │      │  └───────────┘  │      │  │   Mgmt    │  │
│             │      │                 │      │  └───────────┘  │
└──────┬──────┘      └────────┬────────┘      └────────┬────────┘
       │                      │                        │
       └──────────────────────┼────────────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │                   │
                    │    Firewall       │
                    │  ┌─────────────┐  │
                    │  │   pfSense   │  │
                    │  │     IDS     │  │
                    │  └─────────────┘  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │    Internet       │
                    │   (Simulated)     │
                    └───────────────────┘
```

```yaml
# labs/network-security.yml
name: network-security
topology:
  nodes:
    # Firewall/Security Gateway
    firewall:
      kind: linux
      image: pfsense/pfsense:latest
      ports:
        - "443:443"
        - "80:80"
      exec:
        - sysctl -w net.ipv4.ip_forward=1
        - iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

    # IDS/IPS System
    ids:
      kind: linux
      image: jasonish/suricata:latest
      binds:
        - suricata.yaml:/etc/suricata/suricata.yaml
      exec:
        - suricata -c /etc/suricata/suricata.yaml -i eth1

    # DMZ Web Server
    web-server:
      kind: linux
      image: nginx:alpine
      exec:
        - ip addr add 192.168.10.10/24 dev eth1
        - ip route add default via 192.168.10.1
      ports:
        - "8080:80"

    # Internal File Server
    file-server:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 192.168.20.10/24 dev eth1
        - ip route add default via 192.168.20.1
        - apk add --no-cache samba

    # SIEM System
    siem:
      kind: linux
      image: elastic/elasticsearch:7.15.0
      ports:
        - "9200:9200"
        - "5601:5601"
      exec:
        - ip addr add 192.168.30.10/24 dev eth1

    # Attacker Machine (for testing)
    attacker:
      kind: linux
      image: kalilinux/kali-rolling:latest
      exec:
        - apt update && apt install -y nmap metasploit-framework

    # Network Switch
    switch:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2

  links:
    # External connections
    - endpoints: ["firewall:eth0", "attacker:eth1"]

    # Internal network through switch
    - endpoints: ["firewall:eth1", "switch:e1-1"]
    - endpoints: ["ids:eth1", "switch:e1-2"]
    - endpoints: ["web-server:eth1", "switch:e1-10"]
    - endpoints: ["file-server:eth1", "switch:e1-11"]
    - endpoints: ["siem:eth1", "switch:e1-12"]

    # IDS monitoring port (span/mirror)
    - endpoints: ["ids:eth2", "switch:e1-20"]
```

### Lab 9: IPv6 Transition Lab

Demonstrate IPv6 transition mechanisms:

```
    IPv4 Only Network        Dual Stack Network       IPv6 Only Network
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│                     │   │                     │   │                     │
│  ┌───────────────┐  │   │  ┌───────────────┐  │   │  ┌───────────────┐  │
│  │   Client-v4   │  │   │  │  Dual-Stack   │  │   │  │   Client-v6   │  │
│  │ 192.168.1.10  │  │   │  │   Router      │  │   │  │2001:db8::10/64│  │
│  └───────────────┘  │   │  │IPv4 + IPv6    │  │   │  └───────────────┘  │
│                     │   │  └───────────────┘  │   │                     │
└──────────┬──────────┘   └──────────┬──────────┘   └──────────┬──────────┘
           │                         │                         │
           └─────────────────────────┼─────────────────────────┘
                                     │
                           ┌─────────▼─────────┐
                           │                   │
                           │  6to4 Tunnel      │
                           │  NAT64/DNS64      │
                           │  Dual Stack Lite  │
                           └───────────────────┘
```

```yaml
# labs/ipv6-transition.yml
name: ipv6-transition
topology:
  nodes:
    # IPv4 only client
    client-v4:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 192.168.1.10/24 dev eth1
        - ip route add default via 192.168.1.1
        - echo "nameserver 8.8.8.8" > /etc/resolv.conf

    # IPv6 only client
    client-v6:
      kind: linux
      image: alpine:latest
      exec:
        - ip -6 addr add 2001:db8:2::10/64 dev eth1
        - ip -6 route add default via 2001:db8:2::1
        - echo "nameserver 2001:4860:4860::8888" > /etc/resolv.conf

    # Dual stack router with transition mechanisms
    ds-router:
      kind: linux
      image: frrouting/frr:latest
      startup-config: configs/ds-router.conf
      exec:
        - sysctl -w net.ipv4.ip_forward=1
        - sysctl -w net.ipv6.conf.all.forwarding=1
        - modprobe sit

    # NAT64/DNS64 Gateway
    nat64-gw:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y tayga bind9
        - sysctl -w net.ipv4.ip_forward=1
        - sysctl -w net.ipv6.conf.all.forwarding=1

    # 6to4 Tunnel Endpoint
    tunnel-ep:
      kind: linux
      image: alpine:latest
      exec:
        - modprobe sit
        - ip tunnel add sit1 mode sit remote any local 192.168.100.1
        - ip link set sit1 up
        - ip addr add 2002:c0a8:6401::1/16 dev sit1

    # Internet simulator
    internet:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 203.0.113.1/24 dev eth1
        - ip -6 addr add 2001:db8:ffff::1/64 dev eth1

  links:
    - endpoints: ["client-v4:eth1", "ds-router:eth1"]
    - endpoints: ["client-v6:eth1", "ds-router:eth2"]
    - endpoints: ["ds-router:eth3", "nat64-gw:eth1"]
    - endpoints: ["ds-router:eth4", "tunnel-ep:eth1"]
    - endpoints: ["nat64-gw:eth2", "internet:eth1"]
    - endpoints: ["tunnel-ep:eth2", "internet:eth2"]
```

### Lab 10: Network Automation and Monitoring

Complete network automation lab with monitoring stack:

```
                    Monitoring & Automation Stack
    ┌─────────────────────────────────────────────────────────────┐
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
    │  │Prometheus│  │ Grafana │  │ Ansible │  │ NetBox  │        │
    │  │Metrics  │  │Dashboard│  │Automation│ │  IPAM   │        │
    │  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
    └─────────────────────┬───────────────────────────────────────┘
                          │ Management Network
    ┌─────────────────────┼───────────────────────────────────────┐
    │                     │                                       │
    │     Production Network Infrastructure                       │
    │                                                             │
    │  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
    │  │ Router1 │────│ Switch1 │────│ Switch2 │────│ Router2 │  │
    │  │         │    │         │    │         │    │         │  │
    │  └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
    │       │              │              │              │       │
    │  ┌────▼───┐     ┌────▼───┐     ┌────▼───┐     ┌────▼───┐  │
    │  │Server1 │     │Server2 │     │Server3 │     │Server4 │  │
    │  └────────┘     └────────┘     └────────┘     └────────┘  │
    └─────────────────────────────────────────────────────────────┘
```

```yaml
# labs/network-automation.yml
name: network-automation
topology:
  nodes:
    # Monitoring Stack
    prometheus:
      kind: linux
      image: prom/prometheus:latest
      ports:
        - "9090:9090"
      binds:
        - prometheus.yml:/etc/prometheus/prometheus.yml

    grafana:
      kind: linux
      image: grafana/grafana:latest
      ports:
        - "3000:3000"
      env:
        GF_SECURITY_ADMIN_PASSWORD: admin

    # Automation Tools
    ansible:
      kind: linux
      image: ansible/ansible:latest
      binds:
        - ansible-playbooks:/playbooks
        - ansible.cfg:/etc/ansible/ansible.cfg

    netbox:
      kind: linux
      image: netboxcommunity/netbox:latest
      ports:
        - "8000:8080"
      env:
        SUPERUSER_NAME: admin
        SUPERUSER_PASSWORD: admin

    # Network Infrastructure
    router1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/router1.cfg
      labels:
        monitoring: "true"
        ansible-group: routers

    router2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      startup-config: configs/router2.cfg
      labels:
        monitoring: "true"
        ansible-group: routers

    switch1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd1
      startup-config: configs/switch1.cfg
      labels:
        monitoring: "true"
        ansible-group: switches

    switch2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd1
      startup-config: configs/switch2.cfg
      labels:
        monitoring: "true"
        ansible-group: switches

    # Servers
    server1:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y node-exporter
        - systemctl enable node-exporter
    server2:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y node-exporter
        - systemctl enable node-exporter
    server3:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y node-exporter
        - systemctl enable node-exporter
    server4:
      kind: linux
      image: ubuntu:20.04
      exec:
        - apt update && apt install -y node-exporter
        - systemctl enable node-exporter

  links:
    # Management network
    - endpoints: ["prometheus:eth1", "switch1:e1-20"]
    - endpoints: ["grafana:eth1", "switch1:e1-21"]
    - endpoints: ["ansible:eth1", "switch1:e1-22"]
    - endpoints: ["netbox:eth1", "switch1:e1-23"]

    # Production network backbone
    - endpoints: ["router1:e1-1", "switch1:e1-1"]
    - endpoints: ["switch1:e1-2", "switch2:e1-1"]
    - endpoints: ["switch2:e1-2", "router2:e1-1"]

    # Server connections
    - endpoints: ["router1:e1-10", "server1:eth1"]
    - endpoints: ["switch1:e1-10", "server2:eth1"]
    - endpoints: ["switch2:e1-10", "server3:eth1"]
    - endpoints: ["router2:e1-10", "server4:eth1"]
```

## Advanced Features and Use Cases

The labs we've covered demonstrate various advanced networking scenarios:

- **Lab 5 (MPLS)**: Service provider core networks with MPLS LSPs
- **Lab 6 (Kubernetes)**: Container networking and CNI integration
- **Lab 7 (SD-WAN)**: Software-defined WAN with centralized control
- **Lab 8 (Security)**: Network security with firewalls and IDS/IPS
- **Lab 9 (IPv6)**: IPv6 transition mechanisms and dual-stack
- **Lab 10 (Automation)**: Complete monitoring and automation stack

### Network Automation Testing
```yaml
# labs/automation-test.yml
name: automation-test
topology:
  nodes:
    dut1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      labels:
        ansible-group: switches
        role: access
    dut2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      labels:
        ansible-group: switches
        role: distribution
  links:
    - endpoints: ["dut1:e1-1", "dut2:e1-1"]
```

Ansible integration:
```bash
# Generate Ansible inventory
containerlab inspect -t labs/automation-test.yml --format json | \
jq -r '.containers[] | "\(.name) ansible_host=\(.ipv4_address)"' > inventory

# Run Ansible playbook
ansible-playbook -i inventory configure-switches.yml
```

## Running the Advanced Labs

### Prerequisites for Advanced Labs
```bash
# Install additional tools for complex labs
sudo apt install -y python3-pip python3-venv

# Install network automation tools
pip3 install ansible netmiko napalm

# Pull additional container images
docker pull frrouting/frr:latest
docker pull prom/prometheus:latest
docker pull grafana/grafana:latest
docker pull elastic/elasticsearch:7.15.0
docker pull kalilinux/kali-rolling:latest
docker pull kindest/node:v1.28.0
```

### Lab Deployment Examples
```bash
# Deploy MPLS Service Provider lab
containerlab deploy -t labs/mpls-sp-core.yml

# Check MPLS forwarding table
docker exec -it clab-mpls-sp-core-pe1 sr_cli "show network-instance default route-table"

# Deploy Security lab with monitoring
containerlab deploy -t labs/network-security.yml

# Access SIEM dashboard
curl http://localhost:9200/_cluster/health

# Deploy IPv6 transition lab
containerlab deploy -t labs/ipv6-transition.yml

# Test IPv6 connectivity
docker exec -it clab-ipv6-transition-client-v6 ping6 2001:db8:2::1
```

### Traffic Generation and Testing
```yaml
# labs/traffic-test.yml
name: traffic-test
topology:
  nodes:
    switch:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2

    # Traffic generator
    tgen:
      kind: linux
      image: networkstatic/iperf3:latest
      exec:
        - ip addr add 192.168.1.100/24 dev eth1

    # Traffic receiver
    receiver:
      kind: linux
      image: networkstatic/iperf3:latest
      exec:
        - ip addr add 192.168.2.100/24 dev eth1

  links:
    - endpoints: ["switch:e1-1", "tgen:eth1"]
    - endpoints: ["switch:e1-2", "receiver:eth1"]
```

### CI/CD Pipeline Integration
```yaml
# .github/workflows/network-test.yml
name: Network Configuration Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Install Containerlab
      run: bash -c "$(curl -sL https://get.containerlab.dev)"

    - name: Deploy test topology
      run: containerlab deploy -t tests/test-topology.yml

    - name: Run configuration tests
      run: |
        # Test connectivity
        docker exec clab-test-switch1 sr_cli "ping 10.1.1.2"

        # Validate BGP sessions
        docker exec clab-test-switch1 sr_cli "show network-instance default protocols bgp neighbor"

    - name: Cleanup
      run: containerlab destroy -t tests/test-topology.yml
```

## Monitoring and Troubleshooting

### Built-in Monitoring
```bash
# Monitor lab resources
containerlab inspect --format table

# View container logs
docker logs clab-lab-name-node-name

# Monitor network traffic
sudo tcpdump -i clab-lab-name-node1-e1-1

# Access container shell
docker exec -it clab-lab-name-node1 bash
```

### Integration with Monitoring Tools
```yaml
# labs/monitored-lab.yml
name: monitored-lab
topology:
  nodes:
    switch1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
      type: ixrd2
      ports:
        - "57400:57400"  # gNMI port

    # Prometheus for metrics collection
    prometheus:
      kind: linux
      image: prom/prometheus:latest
      ports:
        - "9090:9090"
      binds:
        - prometheus.yml:/etc/prometheus/prometheus.yml

    # Grafana for visualization
    grafana:
      kind: linux
      image: grafana/grafana:latest
      ports:
        - "3000:3000"
      env:
        GF_SECURITY_ADMIN_PASSWORD: admin

  links:
    - endpoints: ["switch1:e1-1", "prometheus:eth1"]
```

## Performance Optimization

### Resource Management
```bash
# Limit container resources
docker update --memory=512m --cpus=0.5 clab-lab-name-node1

# Monitor resource usage
docker stats $(docker ps --filter "name=clab-" --format "{{.Names}}")
```

### Lab Optimization Tips
1. **Use appropriate node types**: Don't use ixrd3 when ixrd2 suffices
2. **Limit startup configs**: Only include necessary configuration
3. **Use bind mounts**: For large files, use bind mounts instead of copying
4. **Clean up regularly**: Remove unused labs and images
5. **Use Docker BuildKit**: Enable for faster image builds

## Best Practices and Tips

### Lab Organization
```bash
# Recommended directory structure
labs/
├── basic/
│   ├── two-node.yml
│   └── configs/
├── datacenter/
│   ├── spine-leaf.yml
│   └── configs/
├── service-provider/
│   ├── mpls-vpn.yml
│   └── configs/
└── automation/
    ├── test-lab.yml
    └── playbooks/
```

### Configuration Management
```bash
# Use Git for version control
git init
git add labs/ configs/
git commit -m "Initial lab setup"

# Use templates for common configurations
# configs/templates/bgp-base.j2
set / network-instance default protocols bgp autonomous-system {{ asn }}
set / network-instance default protocols bgp router-id {{ router_id }}
```

### Security Considerations
```bash
# Run containers with limited privileges
# In lab YAML:
security-opt:
  - no-new-privileges:true
  - seccomp:unconfined

# Use non-root users where possible
user: 1000:1000

# Limit network access
network-mode: none  # For isolated testing
```

## Troubleshooting Common Issues

### Container Startup Issues
```bash
# Check container logs
docker logs clab-lab-name-node-name

# Verify image availability
docker images | grep srlinux

# Check resource constraints
docker system df
docker system prune
```

### Network Connectivity Issues
```bash
# Verify bridge creation
brctl show

# Check interface status
ip link show

# Verify container networking
docker exec -it clab-lab-name-node1 ip addr show
```

### Performance Issues
```bash
# Monitor system resources
htop
iotop
nethogs

# Check Docker daemon logs
journalctl -u docker.service

# Optimize Docker settings
# /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

## Future of Network Simulation with Containerlab

Containerlab is rapidly evolving with new features and capabilities:

1. **Expanded NOS Support**: More network operating systems being containerized
2. **Cloud Integration**: Native support for cloud-based labs
3. **Enhanced Automation**: Better integration with CI/CD pipelines
4. **Improved Monitoring**: Built-in telemetry and observability features
5. **GUI Interface**: Web-based lab management and visualization

## Conclusion

Containerlab represents a paradigm shift in network simulation and testing. By leveraging containers instead of virtual machines, it provides a lightweight, fast, and realistic environment for network engineers to learn, test, and validate network configurations.

Whether you're a student learning networking concepts, a network engineer testing new configurations, or a DevOps professional implementing network automation, Containerlab offers the tools and flexibility you need. Its ability to run actual network operating systems in containers, combined with its simple YAML-based configuration, makes it an invaluable tool for modern network operations.

The examples and labs provided in this guide should give you a solid foundation to start building your own network simulations. As you become more comfortable with Containerlab, you'll discover its true power lies in its ability to create complex, realistic network environments that closely mirror production networks.

Start with the basic labs, experiment with different topologies, and gradually work your way up to more complex scenarios. The investment in learning Containerlab will pay dividends in your network engineering career, providing you with a powerful platform for continuous learning and testing.