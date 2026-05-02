# 2 - Hyperconverged Infrastructure

## Purpose of this chapter

This chapter explains HCI as the architectural answer to the limitations of traditional 3-tier infrastructure. It assumes you already know the baseline model from chapter 1, so it does not repeat the full compute/network/storage explanation.

## Core definition

**Hyperconverged Infrastructure (HCI)** is a software-defined architecture that combines compute, storage, virtualization, networking integration, and management into a distributed cluster of standard servers.

For Nutanix, the practical idea is:

> Each node contributes CPU, memory, local disks, network connectivity, and Nutanix software services. AOS pools and protects those resources, AHV or another supported hypervisor runs workloads, and Prism provides operational management.

Interview-ready version:

> Nutanix HCI replaces the traditional dependency on separate compute hosts, storage arrays, and storage fabrics with a distributed software-defined platform. Storage services run across the cluster through AOS and CVMs, while workloads run on AHV or another supported hypervisor and are managed through Prism.

## What changes compared with 3-tier

In traditional 3-tier architecture, persistent VM data usually sits on external shared storage. In HCI, storage is distributed across the same cluster that runs the workloads.

This does **not** mean networking disappears. Networking remains critical for VM traffic, management, replication, CVM-to-CVM communication, cluster health, and external integrations.

The important change is the failure and operations model:

| Traditional 3-tier | Nutanix HCI |
|---|---|
| Compute and storage are separate domains | Compute and storage participate in one distributed platform |
| Storage array is a central dependency | Local disks are pooled and protected by software |
| Scaling often happens separately | Adding nodes can add compute, capacity, and performance |
| Troubleshooting crosses vendors | Troubleshooting crosses distributed software components |

See chapter 1 for the full 3-tier baseline.

## Core Nutanix components

### AOS

**AOS** is the Nutanix software platform. It provides distributed storage, resiliency, data services, cluster intelligence, upgrades, and operational capabilities. In interview terms, AOS is the platform layer that makes local disks behave like resilient shared infrastructure.

### CVM

The **Controller VM (CVM)** runs on each node and delivers Nutanix storage and platform services. It is one of the most important support concepts because CVM health can affect storage I/O, Prism services, cluster communication, and resiliency workflows.

Simple mental model:

```text
VM -> hypervisor -> local CVM -> AOS distributed storage services -> local/remote disks across cluster
```

### AHV

**AHV** is Nutanix’s native hypervisor. It runs virtual machines. Do not confuse it with AOS:

> AOS is the distributed infrastructure and data-services layer. AHV is the virtualization layer.

Nutanix can also support other hypervisors in specific deployments, but AOS remains the Nutanix platform foundation.

### Prism Element and Prism Central

**Prism Element** manages a local Nutanix cluster.

**Prism Central** provides centralized management across multiple clusters and additional operational capabilities.

For a support manager, Prism is not just a GUI. It is where customers observe health, alerts, capacity, performance, and operational state.

### DSF

**Distributed Storage Fabric (DSF)** is the Nutanix distributed storage layer. It aggregates local drives across nodes and presents resilient storage to workloads.

### Replication factor

**RF2** keeps two copies of data. **RF3** keeps three copies. The key operational question is not just the configured RF value, but whether the cluster is currently healthy enough and has enough capacity to maintain that protection.

### Data locality

Data locality means Nutanix tries to serve VM I/O close to where the VM is running. This is covered in detail in chapter 8. Here you only need the concept: HCI performance benefits when I/O is local or efficiently served through the distributed fabric.

### Resilient capacity

A cluster may have raw free space but insufficient capacity to recover safely after a failure. This becomes critical in escalations involving full clusters, node failures, rebuilds, or expansion.

## Why this matters for Worldwide Support

Nutanix support escalations are distributed-system escalations. A customer may report “storage latency,” but the cause could involve CVM load, network drops, data locality, rebuild activity, capacity pressure, hypervisor behavior, backup load, firmware, or a real product defect.

The support manager’s job is to structure the case:

1. Separate business impact from technical symptoms.
2. Define the affected scope.
3. Identify recent changes.
4. Validate cluster health and resiliency.
5. Assign the right technical workstreams.
6. Communicate facts, risk, next actions, and customer impact clearly.

## Escalation example

Scenario: after a node failure, production VMs are slow and the customer asks whether data is safe.

Strong response:

> The platform is designed to maintain availability through replication, but we need to validate the current cluster state. We will check whether the cluster is RF-compliant, whether rebuild or re-replication is in progress, whether capacity is sufficient, and whether workloads are seeing performance impact. We will communicate data-risk assessment separately from performance-impact assessment.

That answer is technically credible without pretending to be the deepest AOS engineer.

## What not to repeat in later chapters

Later chapters should not re-explain AOS, CVM, AHV, Prism, RF2/RF3, and data locality from scratch. Use this chapter as the reference point:

> See chapter 2 for the core Nutanix HCI component model: AOS, CVM, AHV, Prism, DSF, RF, and data locality.

## Minimum to memorize

> HCI combines compute, storage, virtualization, networking dependencies, and management into a distributed software-defined cluster. In Nutanix, AOS provides distributed storage and data services, CVMs run those services on every node, AHV or another supported hypervisor runs workloads, and Prism provides operational control. The support challenge is understanding the cluster as a distributed system, not as a single appliance.
