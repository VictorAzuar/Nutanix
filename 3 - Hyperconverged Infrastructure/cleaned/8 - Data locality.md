# 8 - Data locality

## Purpose of this chapter

This chapter explains data locality as a performance and troubleshooting concept in Nutanix HCI. It depends on the component model from chapter 2 and the convergence model from chapter 6.

## Core definition

**Data locality** means serving VM I/O from the node where the VM is running whenever possible, rather than forcing every read and write through a centralized external array.

In Nutanix, the local CVM normally plays a central role in serving I/O for workloads on the same node. If data is remote because a VM moved or the cluster is rebalancing, AOS can still serve the data through the distributed storage fabric, but latency and network usage may differ.

Interview-ready version:

> Data locality is the Nutanix design principle of keeping or serving VM data close to where the VM runs. It improves performance by reducing unnecessary network traversal, but after migration, failover, maintenance, or rebalancing, some I/O may temporarily involve remote reads until locality is restored or cached.

## Why data locality matters

Traditional 3-tier infrastructure usually sends VM storage I/O to an external array. HCI changes that path. Because storage services run inside the cluster, locality can reduce latency and east-west traffic.

However, locality is not a static guarantee. VM placement changes, node maintenance, HA events, and cluster balancing can all change where workloads run relative to their data.

This matters because customers may report performance changes after:

- Live migration.
- HA restart.
- Node maintenance.
- Cluster expansion.
- Workload rebalancing.
- Backup or snapshot activity.
- Node or disk failure.

## Local vs remote I/O

Simple model:

```text
Local read:
VM -> local hypervisor -> local CVM -> local data

Remote read:
VM -> local hypervisor -> local CVM -> remote CVM/node -> data
```

Remote I/O is not automatically a problem. A distributed storage system must be able to read and write across nodes. The support question is whether remote activity, network health, CVM load, or background work is contributing to the observed symptom.

## Operational implications

### After VM movement

If a VM moves to another node, some data may initially be remote. The platform can adapt, cache, or re-localize data, but the customer may notice temporary changes depending on workload profile and cluster state.

### During maintenance

Maintenance can change VM placement and cluster activity. If several workloads move at once, compute pressure, CVM load, and data locality effects can overlap.

### During failures

After a node or disk failure, the cluster may be serving data from replicas while also rebuilding protection. Performance symptoms should be interpreted alongside resiliency and capacity state.

### During expansion

Adding nodes does not instantly make all data local to all new workload placements. Workload placement, data movement, and background balancing should be considered.

## Escalation example

Scenario: after planned maintenance, a customer says a database VM is slower even though all nodes are healthy.

Possible explanations:

- The VM moved to a different node.
- Reads are temporarily remote.
- The destination node has more CPU or memory pressure.
- CVM load increased during maintenance.
- Backup or snapshot activity overlaps with the maintenance window.
- Network latency or packet loss affects remote I/O.
- Cluster background tasks are running.

Manager-level response:

> I would correlate the performance change with VM movement and maintenance timing. Then I would check whether the affected VM changed hosts, whether remote reads increased, whether the local CVM and destination node are healthy, and whether network or background cluster activity is contributing.

## How to explain it to a customer

Use a balanced explanation:

> Nutanix is designed to optimize data access close to the VM, but after planned movement or failure recovery, some data access patterns can temporarily change. We are checking whether the observed latency is related to VM placement, data locality, CVM health, network behavior, or other activity during the same window.

Avoid overpromising:

> Locality is an optimization, not a reason to ignore the rest of the stack.

## Relationship to other chapters

- Chapter 2 introduces AOS, CVM, and DSF.
- Chapter 3 explains scale-out clusters.
- Chapter 4 explains software-defined storage.
- Chapter 6 explains compute and storage convergence.
- Chapter 9 explains how failure and rebuild behavior interact with data placement.
- Chapter 10 frames locality as one of HCI’s benefits and trade-offs.

## What to avoid

Avoid saying:

> Data is always local, so network does not matter.

Better:

> Nutanix optimizes for locality, but distributed storage still depends on healthy CVMs, healthy nodes, sufficient capacity, and reliable network communication.

## Triage questions

1. Did the affected VM move recently?
2. Did the issue start after maintenance, HA, migration, expansion, or failure?
3. Is the latency read-heavy, write-heavy, or mixed?
4. Is the problem one VM, a group of VMs, one node, or the whole cluster?
5. Are local and remote reads relevant to the symptom?
6. Is the local CVM healthy?
7. Is the destination node under CPU, memory, or I/O pressure?
8. Are there network errors affecting inter-node traffic?
9. Is rebuild, re-replication, balancing, backup, or snapshot activity running?
10. Is the customer impact temporary degradation or sustained risk?

## Minimum to memorize

> Data locality means Nutanix tries to serve VM I/O close to where the VM runs. It can improve performance by reducing unnecessary network traversal. After VM movement, maintenance, failure, or expansion, some data may be served remotely while the platform adapts. In support, data locality must be evaluated together with CVM health, node load, network quality, capacity, and background activity.
