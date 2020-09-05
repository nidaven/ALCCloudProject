# 

## Overview

In this lab, you configure VPC network peering between two networks. Then you verify private communication between two VMs in those networks, as illustrated in this diagram:

![vpcpeering](./images/VPCpeering.png)

VPC network peering allows you to build SaaS (Software as a service) ecosystems in Google Cloud, which makes services available privately across different VPC networks within and across organizations. This allows workloads to communicate in private RFC 1918 space.

VPC network peering gives you several advantages over using external IP addresses or VPNs to connect networks, including:

Network Latency: Public IP networking results in higher latency than private networking.

Network Security: Service owners do not need to have their services exposed to the public internet and deal with its associated risks.

Network Cost: Google Cloud charges egress bandwidth pricing for networks using external IPs to communicate, even if the traffic is within the same zone. If, however, the networks are peered, they can use internal IPs to communicate and save on those egress costs. Regular network pricing still applies to all traffic.

In this lab, we perform the following tasks:

- Explore connectivity between non-peered VPC networks
- Configure VPC network peering
- Verify private communication between peered VPC networks
- Delete VPC network peering

## Task 1 - Explore connectivity between non-peered VPC networks

* [ ] Code to explore connectivity

## Task 2 - Configure VPC network peering

```bash
gcloud compute networks peerings create peering-1-2 \
--network=mynetwork \
--peer-project=privatenet \
--auto-create-routes
```

```bash
gcloud compute networks peerings create peering-2-1 \
--network=privatenet \
--peer-project=mynetwork \
--auto-create-routes
```

## Task 3 - Verify private communication between peered VPC networks

* [ ]  ssh and verify private / internal IP connection between networks


## Task 4 - Delete VPC network peering

* [ ] Confirm that this peering delete code works

```bash
gcloud compute networks peerings delete peering 1-2
```