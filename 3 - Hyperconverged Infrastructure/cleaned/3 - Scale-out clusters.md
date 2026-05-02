# 3 - Scale-out clusters

## Purpose of this chapter

This chapter explains how Nutanix grows as a distributed platform. It builds on chapter 2, so the core definitions of AOS, CVM, AHV, Prism, DSF, RF, and data locality are not repeated here.

## Core definition

A **scale-out cluster** increases capacity and performance by adding more nodes to a distributed system instead of replacing existing hardware with a larger centralized system.

In Nutanix HCI, adding a node can contribute:

- CPU and memory for workloads.
- Local storage capacity.
- Storage performance.
- Another CVM participating in distributed storage services.
- Additional cluster resources for resiliency and operations.

Interview-ready version:

> A Nutanix scale-out cluster is a distributed HCI platform where adding nodes expands compute, storage capacity, storage performance, and the number of components participating in cluster services. It is not just adding servers; it is expanding the distributed platform.

## Why scale-out matters

In traditional 3-tier architecture, compute and storage scale separately. Adding compute hosts may increase CPU and memory, but the centralized storage array can still become the bottleneck. Storage expansion may require shelves, controllers, array upgrades, new licenses, or a platform refresh.

In Nutanix HCI, the design goal is different. The cluster grows horizontally: more nodes participate in the same distributed platform. That makes scaling operationally simpler, but it also means the environment must be treated as a distributed system.

See chapter 1 for the full 3-tier contrast and chapter 2 for the Nutanix component model.

## What happens when a node is added

At a high level, cluster expansion requires:

1. Discovery of the new node.
2. Network consistency with the existing cluster.
3. AOS, hypervisor, firmware, and component compatibility.
4. CVM participation in cluster services.
5. Capacity and resiliency validation.
6. Monitoring for background tasks, balance, alerts, and performance impact.

A support manager does not need to know every low-level command, but must understand the operational risk: a node expansion is a capacity change, a network change, a version-alignment event, and a resiliency event at the same time.

## Key operational concepts

### Cluster

A group of nodes operating as one Nutanix platform. A cluster is not a loose collection of hosts; it is a coordinated distributed system.

### Node

A physical server contributing compute, memory, local disks, networking, and usually a CVM.

### Resource pool

AOS aggregates local resources into a distributed storage platform. The customer experiences shared infrastructure, but the implementation is distributed across nodes.

### Workload balance

Adding nodes does not automatically mean every workload instantly benefits. Workload placement, hot spots, data locality, background movement, and customer expectations must be managed.

### Fault domains

Scale-out design must consider where redundant data is placed. Disk, node, block, rack, and site failure domains matter when customers ask whether the cluster can survive a particular failure.

### Resilient capacity

A cluster can look like it has free capacity but still lack enough safe headroom to recover from a failure. This matters especially during expansion, rebuild, and degraded-state escalations.

## Common escalation scenario

Scenario: a customer expands a 4-node cluster to 6 nodes. The expansion completes, but later they report higher VM latency and Prism alerts.

Possible causes:

- New nodes have network configuration differences: VLAN, MTU, LACP, uplink, routing, or switch-port configuration.
- AOS, AHV/ESXi, firmware, or driver versions are misaligned.
- Data rebalancing or background tasks are consuming resources.
- Existing capacity was already critically high.
- Workloads are still imbalanced.
- A hot VM or workload masks the expected improvement.
- CVM-to-CVM communication is unhealthy.
- Data resiliency is degraded or rebuild is active.

Manager-level response:

> I would avoid assuming that adding nodes automatically fixes performance. I would validate business impact, cluster health, resiliency, capacity headroom, network consistency, version alignment, and background activity before driving remediation.

## How to lead the escalation

Use parallel workstreams:

| Workstream | What to validate |
|---|---|
| Impact | Outage, degradation, alerts only, capacity risk, executive escalation |
| Change | What was added, when, how, and whether the workflow fully completed |
| Health | Prism alerts, NCC results, nodes, disks, CVMs, services |
| Resiliency | RF compliance, failure-domain tolerance, rebuild/re-replication |
| Capacity | Current utilization, resilient capacity, growth projection |
| Performance | Latency, IOPS, throughput, hot spots, background jobs |
| Network | VLANs, MTU, LACP, packet loss, CRC errors, CVM communication |
| Compatibility | AOS, hypervisor, firmware, drivers, LCM state |

## What to avoid

Do not say:

> Adding nodes always improves performance immediately.

Better:

> Adding nodes increases available resources, but realized performance depends on workload placement, data locality, network health, version consistency, background activity, and whether the original bottleneck was actually resource-related.

## Triage questions

1. What changed and when?
2. How many nodes were added?
3. Did the expansion complete cleanly?
4. Is the issue cluster-wide or limited to certain VMs, nodes, containers, or networks?
5. Are all nodes and CVMs healthy?
6. Is the cluster RF-compliant?
7. Is rebuild, re-replication, scan, or balancing activity running?
8. Is capacity still high after expansion?
9. Are network settings consistent across old and new nodes?
10. Are AOS, hypervisor, firmware, NCC, and LCM versions aligned?
11. What is the safe mitigation path?
12. What customer communication cadence is required?

## Minimum to memorize

> Scale-out means growing horizontally by adding nodes. In Nutanix, each additional HCI node can add compute, storage capacity, storage performance, and another participant in AOS cluster services. The support risk is that expansion can expose network, version, capacity, resiliency, or workload-balance problems. A good support manager validates health and risk before promising performance improvement.
