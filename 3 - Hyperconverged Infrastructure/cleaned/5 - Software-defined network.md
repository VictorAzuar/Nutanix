# 5 - Software-defined network

## Purpose of this chapter

This chapter explains software-defined networking in the context of Nutanix HCI support. It assumes the reader already understands the 3-tier baseline from chapter 1 and the HCI component model from chapter 2.

## Core definition

**Software-defined networking (SDN)** separates network control, policy, and automation from purely hardware-centric configuration. Instead of treating networking only as switch-by-switch configuration, SDN uses software to define, enforce, and operate connectivity and security policies.

In a Nutanix context, networking includes both the physical underlay and the software-defined services above it. Nutanix can simplify network operations, but it does not remove the need for correct physical network design.

Interview-ready version:

> Software-defined networking means network connectivity and security are increasingly managed through software policy rather than only through manual hardware configuration. In Nutanix HCI, the physical network still matters, especially for CVM-to-CVM traffic and VM connectivity, while software-defined services can help with segmentation, automation, and operational control.

## Why networking remains critical in HCI

A common mistake is to think HCI reduces the importance of networking. It does not. HCI reduces dependency on external storage arrays and dedicated storage fabrics, but the cluster still depends on healthy network communication.

Important traffic types:

- VM production traffic.
- Hypervisor management traffic.
- Prism and cluster management traffic.
- CVM-to-CVM communication.
- Replication and disaster recovery traffic.
- Backup traffic.
- Live migration or workload movement traffic.
- External integrations such as DNS, NTP, AD/LDAP, cloud, monitoring, and automation.

This links back to chapter 1: network is one of the three main infrastructure layers, and in HCI it remains a major failure domain.

## Underlay and overlay

### Physical underlay

The underlay is the physical network: switches, NICs, uplinks, VLANs, routing, MTU, LACP, cabling, transceivers, and firewall paths. If the underlay is unstable, higher-level software services will inherit that instability.

Typical underlay issues:

- Packet loss.
- CRC errors.
- Link flapping.
- MTU mismatch.
- VLAN mismatch.
- LACP or bond misconfiguration.
- Asymmetric routing.
- Congested uplinks.
- DNS or NTP failure.

### Software-defined layer

The software-defined layer provides logical networking, policy, segmentation, visibility, and automation. In Nutanix conversations, this may include Flow-related concepts, microsegmentation, virtual networking, and centralized policy management.

For a Worldwide Support Manager, the key is not to oversell SDN as magic. The right framing is:

> SDN improves operational control and policy management, but it depends on a healthy physical network and clear design boundaries.

## Network as an escalation domain

Network problems often masquerade as something else.

| Symptom | Possible network-related cause |
|---|---|
| Storage latency | CVM-to-CVM packet loss, congestion, MTU issue |
| VM timeouts | Firewall, routing, DNS, packet drops |
| Cluster alerts | Management or backplane connectivity issue |
| Failed expansion | VLAN/uplink mismatch on new nodes |
| Replication lag | WAN bandwidth, packet loss, firewall, latency |
| Upgrade failure | Management path, repository access, DNS/NTP issue |

Manager-level response:

> I would not treat networking as only a connectivity question. In HCI, network health affects storage performance, cluster services, replication, upgrades, and customer workload availability.

## Escalation example

Scenario: after adding nodes to a cluster, the customer reports intermittent VM latency and Prism alerts.

Possible network causes:

- New nodes are on a different VLAN or trunk configuration.
- MTU differs between old and new paths.
- LACP or bond configuration is inconsistent.
- Switch ports have errors or packet drops.
- CVM-to-CVM communication is degraded.
- DNS or NTP inconsistencies cause management symptoms.
- East-west cluster traffic is congested.

How to lead:

1. Confirm customer impact and affected scope.
2. Correlate symptoms with the network change or expansion timeline.
3. Validate physical link health and switch errors.
4. Compare network configuration between healthy and affected nodes.
5. Separate VM production traffic, management traffic, and CVM/cluster traffic.
6. Engage network specialists early if packet loss or path instability is suspected.
7. Keep customer updates evidence-based.

## What to avoid

Avoid saying:

> Nutanix HCI removes the need for storage networking, so network is less important.

Better:

> Nutanix reduces reliance on external SAN/NAS fabrics, but the cluster still depends heavily on reliable east-west and management networking. Network issues can directly affect storage performance, resiliency workflows, replication, and customer workloads.

## Relationship to other chapters

- Chapter 1 explains network as one of the three infrastructure tiers.
- Chapter 2 introduces CVM-to-CVM communication as part of HCI.
- Chapter 3 shows why node expansion depends on network consistency.
- Chapter 4 explains why SDS performance depends partly on network health.
- Chapter 8 expands on data locality and remote reads.
- Chapter 9 covers resiliency workflows that rely on healthy networking.

## Triage questions

1. Is the issue connectivity, latency, packet loss, timeout, replication lag, or alerting?
2. Is it isolated to specific VMs, hosts, nodes, VLANs, racks, or sites?
3. Did the issue start after a switch, VLAN, firewall, routing, MTU, LACP, expansion, or upgrade change?
4. Are physical ports showing CRC errors, drops, flaps, or congestion?
5. Are old and new nodes configured consistently?
6. Is CVM-to-CVM communication healthy?
7. Are DNS and NTP stable?
8. Is storage/CVM traffic sharing congested links with VM or backup traffic?
9. Are firewall rules affecting management, replication, or external dependencies?
10. Which team owns the network evidence and next diagnostic step?

## Minimum to memorize

> HCI does not eliminate networking. Nutanix reduces dependency on separate storage arrays and storage fabrics, but cluster health still depends on reliable physical and logical networking. In support, network issues can appear as VM slowness, storage latency, replication lag, cluster alerts, or failed expansion. A good manager separates underlay health, logical policy, recent changes, and customer impact.
