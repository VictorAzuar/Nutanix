# 4 - Software-defined storage

## Purpose of this chapter

This chapter explains software-defined storage in Nutanix HCI. It assumes the reader already understands the 3-tier baseline from chapter 1, the Nutanix component model from chapter 2, and scale-out behavior from chapter 3.

## Core definition

**Software-defined storage (SDS)** separates storage services from a dedicated external storage array and implements them in software across standard servers.

In Nutanix, SDS is delivered through AOS and the Distributed Storage Fabric. Local disks inside cluster nodes are pooled, protected, monitored, and presented to workloads as resilient storage.

Interview-ready version:

> Software-defined storage means that storage intelligence is implemented in software rather than being tied to a traditional external array. In Nutanix, AOS pools local disks across nodes, protects data through replication and failure-domain awareness, and presents resilient storage to workloads through the distributed storage fabric.

## Why SDS matters in HCI

In traditional 3-tier infrastructure, storage intelligence lives mainly in a SAN or NAS platform: controllers, RAID, snapshots, replication, caching, deduplication, and provisioning are managed by the array.

In Nutanix HCI, storage intelligence becomes part of the distributed platform. This is what allows HCI to reduce dependency on separate storage arrays and dedicated storage fabrics.

The architectural shift:

```text
Traditional model:
Servers depend on external shared storage.

Nutanix SDS model:
Cluster nodes contribute local disks, and AOS turns them into distributed resilient storage.
```

This does not make storage “simple” in the sense of being trivial. It makes storage operationally integrated with the HCI platform.

## Core ideas

### Storage is pooled

Each node contributes local disks. AOS aggregates them into a cluster-wide storage system. Customers and workloads consume logical storage, while the platform handles placement, protection, and access.

### Storage services are distributed

Storage services are not concentrated only in one array controller. CVMs across the cluster participate in storage I/O and cluster services. This is why CVM health and node health matter in storage escalations.

### Protection is software-driven

Instead of relying only on array-level RAID, Nutanix protects data using replication factor, placement rules, checksums, self-healing, and failure-domain awareness. RF2 and RF3 were introduced in chapter 2; resiliency behavior is covered further in chapter 9.

### Performance is tied to locality and network health

Because storage is distributed, performance can depend on local disk health, CVM load, node balance, inter-node network health, data locality, and background tasks. Chapter 8 covers data locality in detail.

### Capacity must be interpreted carefully

Raw capacity, usable capacity, and resilient capacity are different operational concepts. A cluster can be dangerously full even if some free capacity remains, especially if it cannot self-heal after a failure.

## Support implications

When a customer says “storage is slow,” do not assume the root cause is only disks.

Possible domains:

| Domain | Examples |
|---|---|
| Workload | High write rate, bursty database, backup job, snapshot activity |
| Compute | CPU contention, memory pressure, noisy neighbor |
| CVM/AOS | CVM resource pressure, service issue, rebuild, re-replication |
| Network | CVM-to-CVM packet loss, MTU issue, congestion, VLAN issue |
| Capacity | Low headroom, insufficient resilient capacity, hot tier |
| Hardware | Disk errors, NIC errors, firmware/driver issue |

Manager-level framing:

> I would treat storage latency as a symptom, not a root cause. In Nutanix, storage performance can be influenced by workload behavior, CVM health, network consistency, capacity, background resiliency activity, and hardware state.

## Escalation example

Scenario: a database VM shows high write latency after a backup window.

A structured response:

1. Confirm business impact and scope.
2. Determine whether the issue is one VM, one node, one container, or cluster-wide.
3. Check whether the timing overlaps with backup, snapshot, replication, migration, rebuild, or upgrade activity.
4. Validate CVM health, node health, disk health, and network errors.
5. Check capacity and resilient capacity.
6. Assign technical owners for workload, storage/AOS, network, and customer communication.

Customer-facing language:

> We are validating whether this is a storage-system issue or a workload and cluster-state interaction. We will check latency, CVM health, network health, background activity, and capacity before identifying the safest mitigation path.

## What to avoid

Avoid saying:

> Nutanix removes storage complexity.

Better:

> Nutanix changes where storage complexity lives. It moves many storage services into distributed software, which can simplify operations, but support still requires strong understanding of capacity, resiliency, locality, network health, and workload behavior.

## Relationship to other chapters

- Chapter 1 explains the traditional external storage model.
- Chapter 2 introduces AOS, CVM, DSF, RF, and Prism.
- Chapter 3 explains why SDS benefits from scale-out nodes.
- Chapter 8 explains data locality.
- Chapter 9 explains resiliency and failure behavior.
- Chapter 10 discusses trade-offs.

## Triage questions

1. What workload is affected?
2. Is the symptom latency, IOPS, throughput, timeout, or alerting?
3. Is the issue isolated or cluster-wide?
4. Did it start after backup, snapshot, migration, expansion, upgrade, or failure?
5. Are CVMs healthy and reachable?
6. Are there disk, node, NIC, or firmware alerts?
7. Is rebuild or re-replication running?
8. Is capacity or resilient capacity under pressure?
9. Are there packet drops, CRC errors, MTU issues, or CVM-to-CVM network problems?
10. What is the safe workaround: move workload, throttle backup, add capacity, pause risky changes, or engage engineering?

## Minimum to memorize

> Software-defined storage means storage services are delivered by software rather than a dedicated external array. In Nutanix, AOS and DSF pool local disks across nodes, protect data through replication and failure-domain awareness, and present resilient storage to workloads. In support, storage symptoms must be investigated across workload, CVM, network, capacity, hardware, and background activity.
