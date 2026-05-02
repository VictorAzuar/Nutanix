# 6 - Compute and storage convergence

## Purpose of this chapter

This chapter explains what it means for compute and storage to converge in HCI. It builds directly on chapter 2, where AOS, CVM, AHV, Prism, DSF, RF, and data locality were introduced.

## Core definition

**Compute and storage convergence** means the same cluster nodes provide both workload execution and storage services. Instead of having compute hosts depend primarily on a separate storage array, Nutanix nodes contribute CPU, memory, local disks, and CVM-based storage services to a distributed platform.

Interview-ready version:

> In Nutanix HCI, compute and storage converge because each node can run workloads and also participate in distributed storage services through AOS and the CVM. This reduces dependency on separate external storage arrays, but it also means support must understand how workload placement, CVM health, storage replication, capacity, and networking interact.

## What convergence does and does not mean

Convergence does **not** mean compute and storage become the same thing. They remain separate resource domains with different failure modes and metrics.

What changes is the operational architecture:

| Traditional 3-tier | HCI convergence |
|---|---|
| Compute hosts run workloads | Nodes run workloads and contribute storage services |
| Storage array provides centralized persistence | AOS distributes storage across nodes |
| Scaling compute and storage often happens separately | Scaling often happens by adding nodes |
| Storage troubleshooting often focuses on array/SAN | Troubleshooting includes workload, CVM, node, disk, network, and capacity |

See chapter 1 for the traditional model and chapter 4 for software-defined storage.

## Why this matters operationally

In a converged platform, resource domains influence each other more visibly.

Examples:

- A noisy workload can consume compute and drive heavy storage I/O.
- A CVM under pressure can affect storage latency.
- A node failure changes both compute capacity and storage resiliency.
- A network issue can affect distributed storage behavior and VM performance.
- A nearly full cluster can make both performance and recovery worse.
- Maintenance or live migration can affect workload placement and data locality.

This is why Nutanix support should think in systems, not silos.

## Key concepts

### Node as a dual participant

A Nutanix node is not only a hypervisor host. It is also a participant in the storage fabric through its CVM and local disks.

### CVM as the convergence point

The CVM is where storage services become visible in the node architecture. If the CVM is unhealthy or resource-constrained, the node’s contribution to the distributed storage layer may be affected.

### Shared failure impact

A node failure may remove compute capacity, workload placement options, local storage devices, and one participant in cluster services. The cluster is designed to tolerate failures, but current RF compliance, capacity, and failure-domain state must be validated.

### Operational simplicity vs diagnostic coupling

Convergence simplifies deployment and day-to-day operations, but it can make diagnosis more interconnected. A symptom called “storage latency” might involve compute, network, CVM, workload, or capacity.

## Escalation example

Scenario: a customer reports that after a node entered maintenance mode, several VMs became slower.

Possible contributing factors:

- VMs moved to hosts with less available CPU or memory.
- Data locality changed after workload movement.
- Remote reads increased temporarily.
- CVMs are handling more activity during migration or recovery.
- Cluster is running background tasks.
- Network paths are congested.
- Capacity headroom is limited.

Manager-level response:

> I would separate the compute impact from the storage and cluster-state impact. We need to know where the VMs moved, whether resource contention appeared, whether data locality or remote reads changed, whether CVMs are healthy, and whether the cluster is doing background resiliency work.

## How to communicate convergence to executives

For an executive customer, avoid deep internals first:

> In a converged Nutanix environment, the same platform provides both compute and storage. That simplifies infrastructure operations, but during an incident we must check both workload resources and storage services together because they are part of the same distributed system.

For a technical customer:

> We are validating whether the symptom is caused by workload placement, CVM health, replication/rebuild activity, network behavior, or capacity pressure. These domains are interconnected in HCI, so we are investigating them in parallel rather than treating the issue as only compute or only storage.

## What to avoid

Avoid saying:

> Compute and storage convergence means there are fewer things that can go wrong.

Better:

> Convergence can reduce infrastructure silos and simplify operations, but the support model still requires disciplined troubleshooting across workload, hypervisor, CVM, storage, network, and capacity domains.

## Relationship to other chapters

- Chapter 2 defines the HCI component model.
- Chapter 3 explains how added nodes expand a converged platform.
- Chapter 4 explains software-defined storage.
- Chapter 8 explains data locality, one of the most important performance implications of convergence.
- Chapter 9 explains how convergence affects failure and recovery thinking.

## Triage questions

1. Is the symptom compute, storage, network, or application-facing?
2. Did it start after migration, maintenance, failure, expansion, or upgrade?
3. Which nodes are hosting the affected workloads?
4. Are CPU, memory, and hypervisor resources healthy?
5. Are CVMs healthy and properly resourced?
6. Is data locality or remote-read behavior relevant?
7. Is rebuild, re-replication, or background balancing active?
8. Is the cluster near capacity or below resilient capacity?
9. Are there network errors affecting CVM-to-CVM communication?
10. What evidence separates workload behavior from platform behavior?

## Minimum to memorize

> Compute and storage convergence means Nutanix nodes both run workloads and participate in distributed storage services. This reduces reliance on separate storage arrays, but support must investigate incidents across workload placement, hypervisor health, CVM health, storage replication, network health, and capacity. Convergence simplifies the architecture, but diagnosis remains system-level.
