# 8 - VM Resources: CPU, Memory, Disk, and NIC

## 1. Purpose of this chapter

This chapter gives a practical model for analyzing VM resources on AHV. It helps a support manager convert vague performance complaints into CPU, memory, disk, network, guest, and cluster workstreams.

## 2. Core definition

VM resources are the virtual CPU, memory, disk, and network devices assigned to a VM and backed by physical host and cluster resources.

## 3. Main explanation

Every VM consumes four core resource planes: CPU, memory, disk/storage I/O, and network I/O. Problems often appear as “the application is slow,” but the bottleneck may be in only one plane.

CPU issues include high guest CPU, scheduling delay, too few vCPUs, too many vCPUs, noisy neighbors, or host contention. Memory issues include under-sizing, leaks, swapping, ballooning, NUMA effects, and HA reservation pressure. Disk issues include high latency, IOPS limits, snapshot/backup pressure, CVM/storage health, or container-level contention. Network issues include vNIC configuration, VLAN mapping, MTU, bond/uplink faults, packet loss, DNS, firewall, or upstream switch issues.

The support manager’s role is to force structured comparison: guest metrics versus Prism metrics, one VM versus many VMs, one host versus cluster-wide, before and after change, and affected versus unaffected workloads.

## 4. Key concepts

- Configured resources are not the same as consumed resources.
- Over-allocation can harm performance, especially with CPU and NUMA-sensitive workloads.
- Storage symptoms may originate from CVM, cluster, network, backup, snapshot, or workload behavior.
- Network symptoms require guest, vNIC, VLAN/subnet, host uplink, and upstream validation.
- Resource triage depends on scope and timeline.

## 5. Support relevance

Enterprise customers expect support to move faster than “check CPU and memory.” A strong manager directs parallel evidence gathering: guest counters, Prism charts, host health, storage latency, network drops, recent tasks, and workload baselines. This reduces MTTR and prevents blind resizing as the default answer.

## 6. Escalation scenario

A customer reports that a payroll application is slow only at month-end. The team sees high disk latency during backup overlap, moderate CPU use, no memory pressure, and network stability. The escalation lead coordinates backup schedule review, storage latency analysis, Prism metrics, and application batch timing instead of approving a CPU increase that would not address the bottleneck.

## 7. Triage questions

- Which resource plane shows pressure: CPU, memory, disk, or network?
- Do guest OS metrics match Prism/AHV metrics?
- Is the issue constant, intermittent, or workload-cycle dependent?
- Is the VM oversized, undersized, or recently resized?
- Are other VMs on the same host affected?
- Are snapshots, backups, migrations, or maintenance tasks running?
- Does moving or comparing the VM change the symptom?

## 8. What to avoid

- Weak: “Add more CPU.” Better: “Confirm CPU saturation or scheduling delay before resizing.”
- Weak: “Memory is available, so memory is not involved.” Better: “Check guest swapping, ballooning, NUMA alignment, and host pressure.”
- Weak: “Disk latency means bad disks.” Better: “Check workload, snapshots, CVM health, storage fabric, network path, and cluster events.”

## 9. Minimum to memorize

- VM performance maps to CPU, memory, disk, and network planes.
- Configured capacity differs from active consumption.
- Right-sizing is safer than reflexive over-sizing.
- Guest and Prism metrics must be correlated.
- Scope and recent change drive triage priority.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Ballooning | Memory reclamation mechanism involving guest cooperation. |
| CPU Ready | Scheduling delay concept where vCPU waits for physical CPU time. |
| IOPS | Input/output operations per second. |
| MTU | Maximum Transmission Unit. |
| NUMA | Non-Uniform Memory Access architecture. |
| vCPU | Virtual CPU. |
| vDisk | Virtual disk. |
| vNIC | Virtual network adapter. |
| vRAM | Virtual memory assigned to a VM. |
