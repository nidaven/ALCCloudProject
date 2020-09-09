# Working With Multiple VPCs (Virtual Private Clouds)

- [Working With Multiple VPCs (Virtual Private Clouds)](#working-with-multiple-vpcs-virtual-private-clouds)
  - [1.1. Overview](#11-overview)
  - [1.2. Objectives](#12-objectives)
  - [1.3. Task 1. Create custom mode VPC networks with firewall rules](#13-task-1-create-custom-mode-vpc-networks-with-firewall-rules)
    - [1.3.1 Creating the managementnet network](#131-creating-the-managementnet-network)
    - [1.3.2 Creating the privatenet network](#132-creating-the-privatenet-network)
    - [1.3.3 Creating the firewall rules for managementnet](#133-creating-the-firewall-rules-for-managementnet)
  - [1.4 Task 2. Create VM instances](#14-task-2-create-vm-instances)
  - [1.5 Task 3. Explore the connectivity between VM instances](#15-task-3-explore-the-connectivity-between-vm-instances)
    - [1.5.1 Ping the external IP addresses](#151-ping-the-external-ip-addresses)
    - [1.5.2 Ping the Internal IP Addresses](#152-ping-the-internal-ip-addresses)
  - [1.6 Task 4. Create a VM instance with multiple network interfaces](#16-task-4-create-a-vm-instance-with-multiple-network-interfaces)

## 1.1. Overview

In the network diagram below, we create two custom-mode networks with each having their own VMs and firewall rules.
The networks are: **managementnet** and **privatenet** respectively. Interconnectivity, will be tested between VMs on one of these networks using another VM that has a multiple-interfaces on each of these networks. Connectivity will be testing using both internal IPs and external IPs.

![Multiple VPC architecture](./images/multivpcoverview.png)

## 1.2. Objectives

In this lab, you learn how to perform the following tasks:

- Create custom mode VPC networks with firewall rules
- Create VM instances using Compute Engine
- Explore the connectivity for VM instances across VPC networks
- Create a VM instance with multiple network interfaces

## 1.3. Task 1. Create custom mode VPC networks with firewall rules

Here we create the two custom networks, **managementnet** and **privatenet**, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic.

### 1.3.1 Creating the managementnet network

1. Create the network named **managementnet** first

    ```bash
    gcloud compute networks create managementnet --subnet-mode=custom
    ```

1. Create the subnetwork next

    ```bash
    gcloud compute networks subnets create managementsubnet-us --network=managementnet --region=us-central1 --range=10.130.0.0/20
    ```

### 1.3.2 Creating the privatenet network

1. Create the network named **privatenet** network first

    ```bash
    gcloud compute networks create privatenet --subnet-mode=custom
    ```

1. Create a us subnetwork next

    ```bash
    gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/20
    ```

1. Create an eu subnetwork

    ```bash
    gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
    ```

    List all networks

    ```bash
    gcloud compute networks list
    ```

    ```bash
    gcloud compute networks subnets list --sort-by=NETWORK
    ```

### 1.3.3 Creating the firewall rules for managementnet

```bash
gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

```bash
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

```bash
gcloud compute firewall-rules list --sort-by=NETWORK
```

## 1.4 Task 2. Create VM instances

```bash
export vmzone=us-central1-c
```

1. Create the **managementnet-us-vm** instance

    ```bash
    gcloud compute instances create managementnet-us-vm --machine-type=n1-standard-1 --zone=$vmzone --subnet=managementsubnet-us
    ```

1. Create **privatenet-us-vm** instance

    ```bash
    gcloud compute instances create privatenet-us-vm --zone=$vmzone --machine-type=n1-standard-1 --subnet=privatesubnet-us
    ```

## 1.5 Task 3. Explore the connectivity between VM instances

Set the environment variable for *PROJECT_ID*

```bash
export PROJECT_ID=$(gcloud info --format='value(config.project)')
```

### 1.5.1 Ping the external IP addresses

> Before SSH into the remote machine, note down the following IP addresses as we will test connectivity to these VMs from the remote machine.

1. Get external IP address for **mynet-eu-vm**

    ```bash
    gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='value(networkInterfaces.accessConfigs[0].natIP)'
    ```

1. Get external IP address for **managementnet-us-vm**

    ```bash
    gcloud compute instances describe managementnet-us-vm --zone=vmzone --format='value(networkInterfaces.accessConfigs[0].natIP)'
    ```

1. Get external IP address for **privatenet-us-vm**

    ```bash
    gcloud compute instances describe privatenet-us-vm --zone=vmzone --format='value(networkInterfaces.accessConfigs[0].natIP)'
    ```

1. Finally SSH into **mynet-us-vm**

    ```bash
    gcloud compute ssh --project $PROJECT_ID --zone $vmzone mynet-us-vm
    ```

    > Note: You will need to press enter key till a successful shell login. This will only be required for the first remote access.

1. Ping the various VMs

    ```bash
    ping -c 3 <Enter mynet-eu-vm's external IP here>
    ```

    ```bash
    ping -c 3 <Enter managementnet-us-vm's external IP here>
    ```

    ```bash
    ping -c 3 <Enter privatenet-us-vm's external IP here>
    ```

    > There should be connectivity to all of the 3 external VM IPs

1. Exit the remote computer

    ```bash
    exit
    ```

### 1.5.2 Ping the Internal IP Addresses

1. Get internal IP address for **mynet-eu-vm**

    ```bash
    gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='value(networkInterfaces.networkIP)'
    ```

1. Get internal IP address for **managementnet-us-vm**

    ```bash
    gcloud compute instances describe managementnet-us-vm --zone=$vmzone --format='value(networkInterfaces.networkIP)'
    ```

1. Get internal IP address for **privatenet-us-vm**

    ```bash
    gcloud compute instances describe privatenet-us-vm --zone=$vmzone --format='value(networkInterfaces.networkIP)'
    ```

1. SSH back into the VM

    ```bash
    gcloud compute ssh --project $PROJECT_ID --zone $vmzone mynet-us-vm
    ```

1. Test the connectivity to the internal IP addresses

    ```bash
    ping -c 3 <Enter mynet-eu-vm's internal IP here>
    ```

    ```bash
    ping -c 3 <Enter managementnet-us-vm's internal IP here>
    ```

    ```bash
    ping -c 3 <Enter privatenet-us-vm's internal IP here>
    ```

    There should be only be connectivity to the **mynet-eu-vm** virtual machine as they are on the same network and can communicate via internal IP addresses

1. Exit the VM

    ```bash
    exit
    ```

## 1.6 Task 4. Create a VM instance with multiple network interfaces

1. Create the instance with the different interfaces

    ```bash
    gcloud compute instances create vm-appliance --zone=us-central1-c --machinetype=n1-standard-4 \
    --network-interface subnet=privatesubnet-us \
    --network-interface subnet=managementsubnet-us \
    --network-interface subnet=mynetwork
    ```

1. Check the machine to show the newly created network interfaces by SSH

    ```bash
    gcloud compute ssh --project=$PROJECT_ID --zone=$vmzone vm-appliance
    ```

    > You will be prompted to allow the connection by **ENTER**, two other prompts will come up, press **ENTER** for the next set of prompts.

1. List the network interfaces within the VM instance

    ```bash
    sudo ifconfig
    ```

    Below is the expected output

    ```bash
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
            inet 172.16.0.3  netmask 255.255.255.255  broadcast 172.16.0.3
            inet6 fe80::4001:acff:fe10:3  prefixlen 64  scopeid 0x20<link>
            ether 42:01:ac:10:00:03  txqueuelen 1000  (Ethernet)
            RX packets 626  bytes 171556 (167.5 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 568  bytes 62294 (60.8 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
            inet 10.130.0.3  netmask 255.255.255.255  broadcast 10.130.0.3
            inet6 fe80::4001:aff:fe82:3  prefixlen 64  scopeid 0x20<link>
            ether 42:01:0a:82:00:03  txqueuelen 1000  (Ethernet)
            RX packets 7  bytes 1222 (1.1 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 17  bytes 1842 (1.7 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    eth2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
            inet 10.128.0.3  netmask 255.255.255.255  broadcast 10.128.0.3
            inet6 fe80::4001:aff:fe80:3  prefixlen 64  scopeid 0x20<link>
            ether 42:01:0a:80:00:03  txqueuelen 1000  (Ethernet)
            RX packets 17  bytes 2014 (1.9 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 17  bytes 1862 (1.8 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    ```

1. Ping the following vm-instances to test connectivity

    Use the respective IP addresses you took down earlier.

    ```bash
    ping -c 3 <Enter mynet-eu-vm's internal IP here>
    ```

    This should work!

    ```bash
    ping -c 3 <Enter managementnet-us-vm's internal IP here>
    ```

    This should work as well

    ```bash
    ping -c 3 <Enter privatenet-us-vm's internal IP here>
    ```

    This should work!

    ```bash
    ping -c 3 privatenet-us-vm
    ```

    This should work as well due to DNS name resolving to the primary network interface (nic0) of the vm instance.

    ```bash
    ping -c 3 <Enter mynet-us-vm's internal IP here>
    ```

1. List the IP routes associated with the VM's network interface

    ```bash
    ip route
    ```

In this lab, we successfully created tow custom-mode networks **Privatenet** and **managementnet**. We created firewall rules in each of these networks as well as VM instances. We explored connectivity between the instances in these different networks by pinging both their internal and external IP addresses. We saw that we couldnt achieve connectivity to an instance in another network through its internal IP address, but we connect through its external IP address as it is routed through the internet. Lastly we created a VM instance with multiple interface, this allows us to connect or ping the instances in different networks through their internal IP addresses.
