# 1 - Traditional 3-tier architecture

## Purpose of this chapter

This chapter establishes the baseline model that Nutanix HCI is designed to simplify. Later chapters should not re-explain the full 3-tier model; they should simply refer back to this section when contrasting HCI with legacy infrastructure.

## Core definition

Traditional **3-tier infrastructure architecture** separates enterprise infrastructure into three main layers:

1. **Compute**: physical servers, CPU, memory, hypervisors, and workload placement.
2. **Network**: production, management, migration, backup, replication, and storage connectivity.
3. **Storage**: centralized SAN or NAS platforms providing persistent data services.

This is not the same as application 3-tier architecture, which usually means web, application, and database tiers. Here we are talking about infrastructure.

A strong interview answer:

> Traditional 3-tier infrastructure separates compute, network, and storage. Compute runs the workloads, network connects the environment and carries several traffic types, and storage provides persistent shared data services. The model is mature and reliable, but troubleshooting can be complex because incidents often cross layers, teams, and vendors.

## How the model works

In a classic VMware or enterprise virtualization environment, VMs run on compute hosts but their virtual disks often live on external shared storage. A simplified I/O path looks like this:

```text
VM -> hypervisor -> host NIC/HBA -> network or SAN fabric -> storage array -> disks/flash
```

This architecture gives enterprises strong capabilities: shared datastores, live migration, snapshots, replication, backup integration, and mature operational practices. The trade-off is that the environment becomes operationally segmented. Compute, network, and storage may be owned by different teams, supported by different vendors, and scaled on different timelines.

## Key layers

### Compute

Compute is where workloads execute. It includes servers, CPUs, memory, firmware, hypervisors, VM scheduling, high availability, and live migration. Typical symptoms include CPU contention, memory pressure, host failure, firmware mismatch, and noisy-neighbor workloads.

### Network

Network is broader than user-facing connectivity. It may carry management traffic, VM traffic, vMotion/live migration traffic, backup traffic, replication traffic, and storage traffic such as Fibre Channel, iSCSI, or NFS.

Important correction:

> A storage network is not a fourth main tier. It is a specialized and critical part of the network tier.

Network problems can appear as storage latency, application timeouts, cluster instability, or VM connectivity issues.

### Storage

Storage provides persistent data services through arrays, LUNs, volumes, datastores, snapshots, replication, deduplication, compression, caching, and disaster recovery features. Typical issues include high datastore latency, controller failover, pool saturation, queue depth limits, snapshot impact, replication lag, and thin provisioning exhaustion.

## Why this matters for Nutanix support

Most Nutanix customers either migrated from 3-tier, still run 3-tier alongside Nutanix, or compare Nutanix against a known SAN/NAS operating model. A Worldwide Support Manager does not need to replace a senior storage engineer, but must understand the customer’s old architecture well enough to lead the escalation intelligently.

The central management question is:

> Is the symptom coming from compute, network, storage, virtualization, application behavior, or a recent change?

This framework is reused throughout the rest of the HCI module.

## Common escalation pattern

Scenario: VMs are slow after a maintenance window.

Possible causes:

| Area | Examples |
|---|---|
| Compute | CPU contention, memory pressure, host firmware issue, VM placement imbalance |
| Network | VLAN change, packet loss, MTU mismatch, storage path issue, DNS/NTP issue |
| Storage | Array latency, controller failover, overloaded pool, snapshot or replication impact |

Manager-level response:

> I would first define business impact, affected scope, and timeline. Then I would split the investigation across compute, network, and storage, using evidence rather than assumptions. The goal is to isolate the failing domain while avoiding vendor finger-pointing and keeping customer communication structured.

## What not to repeat in later chapters

Later chapters should not re-define all three layers in detail. Use this reference instead:

> See chapter 1 for the baseline 3-tier model: compute, network, and centralized storage.

## Triage questions

Use these as the base checklist for 3-tier or HCI comparison cases:

1. What application or service is affected?
2. Is this outage, degradation, or risk-only?
3. What changed recently?
4. Which hosts, VMs, networks, datastores, or sites are in scope?
5. Are compute resources saturated?
6. Are there packet drops, link errors, VLAN/MTU changes, DNS/NTP issues, or storage path instability?
7. Is storage latency high at datastore, array, controller, or disk level?
8. Are backup, snapshot, replication, rebuild, or maintenance tasks running?
9. Who owns each workstream?
10. What is the mitigation path and next customer update?

## Minimum to memorize

> Traditional 3-tier infrastructure separates compute, network, and storage. It is mature and powerful, but operationally complex because performance and availability issues often cross multiple layers and vendors. Nutanix HCI changes the model by distributing storage services across the cluster and integrating operations through software, while networking and disciplined operations remain critical.
