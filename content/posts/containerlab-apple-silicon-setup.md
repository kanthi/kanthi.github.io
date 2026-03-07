+++
title = "Setting Up Containerlab on Apple Silicon Macs in 2026"
date = "2026-01-20T22:30:00+05:30"
author = "Kanthi"
draft = true
authorTwitter = ""
cover = ""
tags = ["containerlab", "networking", "docker", "apple-silicon", "network-automation", "macos"]
keywords = ["containerlab", "apple silicon", "m1", "m2", "m3", "m4", "mac", "docker", "network simulation", "network labs"]
description = "A practical guide to setting up Containerlab on Apple Silicon Macs for network simulation and automation. Transform your Mac into a powerful network lab environment."
showFullContent = false
readingTime = true
hideComments = false
+++

As network engineers and automation enthusiasts, we often need realistic lab environments to test configurations, learn new technologies, and validate designs. Traditional solutions like GNS3 or EVE-NG typically require x86 machines or complex virtualization setups. But what if you're on an Apple Silicon Mac?

Enter **Containerlab** — a container-based networking lab orchestration tool that's changing how we build network labs. This guide shows you how to get it running on your M1/M2/M3/M4 Mac.

## What is Containerlab?

Containerlab is a CLI tool that orchestrates container-based networking labs. It allows you to build complex network topologies using containerized Network Operating Systems (NOS) like Nokia SR Linux, Arista cEOS, Cisco images, and more.

### Why Containerlab Over Traditional Simulators?

- **Lightweight**: Containers use far fewer resources than VMs
- **Fast**: Labs spin up in seconds, not minutes
- **Reproducible**: Topologies defined in YAML files, perfect for version control
- **Realistic**: Uses actual network operating systems with real CLI experiences

## The Apple Silicon Challenge

Here's the reality: Containerlab itself runs natively on Linux. On Macs (including Apple Silicon), we need a Linux environment. The good news is that Docker Desktop provides this transparently through its VM backend.

The trickier part is that many network OS images (like Cisco IOS-XE, Juniper vMX) are built for x86_64 architecture. On Apple Silicon, these require emulation, which comes with performance trade-offs. However, several images work well:

- **Nokia SR Linux**: Native ARM64 support
- **FRRouting**: Works great on ARM
- **Linux containers**: Perfect for hosts and clients
- **Some vendor images**: Available through Docker Hub with ARM support

## Prerequisites

Before we start, ensure you have:

1. **macOS** on Apple Silicon (M1/M2/M3/M4)
2. **Docker Desktop** installed and running
3. **Homebrew** for easy installation

## Step 1: Install Docker Desktop

If you haven't already, install Docker Desktop:

```bash
brew install --cask docker
```

Launch Docker Desktop and ensure it's running. You can verify with:

```bash
docker version
```

## Step 2: Install Containerlab

The easiest way to install Containerlab on macOS is via the install script:

```bash
bash -c "$(curl -sL https://get-clab.srlinux.dev)"
```

Alternatively, use Homebrew:

```bash
brew install containerlab
```

Verify the installation:

```bash
containerlab version
```

To upgrade an existing installation:

```bash
containerlab version upgrade
```

## Step 3: Your First Lab

Let's create a simple two-node topology. Create a file called `lab.yaml`:

```yaml
name: my-first-lab

topology:
  nodes:
    router1:
      kind: linux
      image: frrouting/frr:latest
    router2:
      kind: linux
      image: frrouting/frr:latest

  links:
    - endpoints: ["router1:eth1", "router2:eth1"]
```

Deploy the lab:

```bash
containerlab deploy --topo lab.yaml
```

You'll see output showing the nodes being created. Once complete, you can connect to the nodes:

```bash
docker exec -it clab-my-first-lab-router1 vtysh
```

## A More Realistic Example

Here's a Cisco-style lab using available images:

```yaml
name: cisco_lab

mgmt:
  network: custom_mgmt
  ipv4-subnet: 172.100.100.0/24

topology:
  nodes:
    R1:
      kind: cisco_vios
      image: asifsyd/cisco_vios:15.9.3M6
      mgmt-ipv4: 172.100.100.11
    SW1:
      kind: cisco_iol
      image: asifsyd/cisco_iol:l2-17.12.01
      mgmt-ipv4: 172.100.100.12

  links:
    - endpoints: ["R1:eth1", "SW1:eth1"]
```

> **Note**: Some Cisco images on Docker Hub are community-built. Always verify licensing and availability.

## Essential Containerlab Commands

```bash
# Deploy a topology
containerlab deploy --topo lab.yaml

# View running labs
containerlab inspect

# Visualize topology in browser
containerlab graph --topo lab.yaml

# Connect to a node
docker exec -it <container-name> <shell>

# Destroy a lab
containerlab destroy --topo lab.yaml --cleanup
```

## Working with Different Node Types

### Nokia SR Linux (ARM64 Native!)

Nokia SR Linux has excellent ARM64 support, making it perfect for Apple Silicon:

```yaml
name: srlinux-lab

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

Connect to SR Linux:

```bash
docker exec -it clab-srlinux-lab-srl1 sr_cli
```

### FRRouting for Open-Source Routing

FRR is a fantastic choice for building routing labs:

```yaml
name: frr-lab

topology:
  nodes:
    r1:
      kind: linux
      image: frrouting/frr:latest
      exec:
        - sysctl -w net.ipv4.ip_forward=1
    r2:
      kind: linux
      image: frrouting/frr:latest
      exec:
        - sysctl -w net.ipv4.ip_forward=1
    client:
      kind: linux
      image: alpine:latest

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "client:eth1"]
```

### Arista cEOS

Arista cEOS images require a free account to download. Once you have the tarball:

```bash
docker import cEOS64-lab-4.27.0F.tar ceos:4.27.0F
```

Then use it in your topology:

```yaml
nodes:
  eos1:
    kind: ceos
    image: ceos:4.27.0F
```

## VS Code Integration

The **Containerlab VS Code Extension** makes managing labs much easier:

- Syntax highlighting for topology files
- Quick commands to deploy/destroy labs
- Node connection shortcuts

Search for "Containerlab" in the VS Code extensions marketplace.

## Tips for Apple Silicon Users

### 1. Use ARM-Native Images When Possible

Performance is significantly better with ARM-native images. Nokia SR Linux and FRRouting are your best friends here.

### 2. Resource Management

Docker Desktop allocates limited resources by default. For larger labs, increase the memory and CPU allocation in Docker Desktop settings.

### 3. Check Image Architecture

Before pulling an image, verify its architecture:

```bash
docker manifest inspect <image> | grep architecture
```

### 4. Consider a Linux VM for x86 Images

For labs requiring x86-only images, consider running a Linux VM via UTM or Parallels with x86 emulation. This provides better compatibility than Docker's Rosetta emulation for some workloads.

## Performance Expectations

On my M3 MacBook Pro:

- **Nokia SR Linux**: Excellent, feels native
- **FRRouting**: Great performance
- **Alpine/Linux containers**: Instant
- **x86 emulated images**: Usable but noticeably slower

For learning BGP, OSPF, and network automation fundamentals, the FRRouting + SR Linux combination is incredibly capable.

## What's Next?

Once you're comfortable with basics:

1. **Explore EVPN-VXLAN labs** with SR Linux
2. **Integrate with Ansible** for automation testing
3. **Build CI/CD pipelines** to validate configurations
4. **Try netlab** — a higher-level abstraction over Containerlab

## Conclusion

Containerlab transforms your Apple Silicon Mac into a powerful network lab environment. While not every network OS image works perfectly on ARM, the ecosystem is improving rapidly. For learning, testing, and automation development, it's an excellent choice.

The days of needing a dedicated server rack or expensive VMs to practice networking are over. With Containerlab and Docker, your laptop becomes your lab.

---

*Happy labbing! 🧪*
