# 2 - Type 1 Hypervisors

## 1. Purpose of this chapter

This chapter clarifies what “bare-metal hypervisor” means and why Type 1 hypervisors dominate enterprise virtualization. It also positions AHV correctly against ESXi and Hyper-V without reducing the comparison to feature slogans.

## 2. Core definition

A Type 1 hypervisor runs directly on physical server hardware and manages VMs without requiring a general-purpose host operating system underneath. AHV, VMware ESXi, Microsoft Hyper-V, and KVM-based enterprise platforms are examples of this class.

## 3. Main explanation

Type 1 hypervisors exist to provide strong isolation, predictable performance, resource scheduling, and operational control at server level. They abstract CPU, memory, disk, and network hardware so several workloads can safely share a host.

The important enterprise distinction is not only technical architecture. Type 1 hypervisors are surrounded by management, HA, migration, monitoring, backup, and lifecycle tooling. AHV is different from a standalone hypervisor because Nutanix embeds it into the HCI operating model: AOS supplies distributed storage and cluster services, CVMs participate in the data path, and Prism provides management.

A support manager should compare hypervisors through operational questions: How are VMs created and monitored? How does HA work? How are migrations performed? How does storage attach to the VM? How are drivers handled? What breaks during upgrades? What evidence is available during incidents?

## 4. Key concepts

- Bare-metal placement gives the hypervisor direct control over server resources.
- Enterprise value comes from the ecosystem: HA, migration, monitoring, lifecycle, backup, and supportability.
- AHV should be compared through operating model, not only feature parity.
- Type 2 hypervisors are useful for desktop or lab use, but they are not the normal model for critical enterprise clusters.
- Compatibility, drivers, and management workflows often matter as much as the hypervisor label.

## 5. Support relevance

Customers often arrive with assumptions formed on VMware or Hyper-V. A support manager must translate familiar concepts while warning about differences. For example, a customer may assume that every VMware operational habit maps one-to-one to AHV. Some concepts do map, such as VM lifecycle, HA, and live migration; others require Nutanix-specific understanding, such as Prism workflows, CVM health, AOS storage path, and AHV guest drivers.

## 6. Escalation scenario

A customer migrating from ESXi to AHV reports that a critical Windows VM boots but has poor network throughput. The escalation should not become a generic hypervisor debate. The lead frames the issue around migration artifacts: VirtIO driver status, vNIC type, VLAN mapping, IP configuration, host placement, guest OS counters, Prism network metrics, and physical uplink health. This keeps the case evidence-driven and avoids “AHV versus VMware” arguments.

## 7. Triage questions

- Which source hypervisor did the workload come from?
- Is the issue conceptual, migration-related, driver-related, or runtime performance-related?
- What VM hardware, disk controller, NIC, and guest tools changed during migration?
- Does the issue reproduce on newly created AHV-native VMs?
- Which management plane shows the failure: Prism, guest OS, backup tool, or customer monitoring?
- Are HA, migration, and backup expectations based on another platform?
- Is the configuration supported for the AHV/AOS version in use?

## 8. What to avoid

- Weak: “AHV is just like ESXi.” Better: “AHV provides similar enterprise virtualization functions, but its operational model is integrated with Nutanix AOS, CVMs, and Prism.”
- Weak: “Type 1 means there are no OS dependencies.” Better: “Type 1 means bare-metal virtualization, but firmware, drivers, kernel modules, management services, and hardware compatibility still matter.”
- Weak: “Feature parity is the main comparison.” Better: “Supportability, workflows, monitoring, lifecycle, and migration behavior are equally important.”

## 9. Minimum to memorize

- Type 1 hypervisors run directly on server hardware.
- AHV is a Type 1 enterprise hypervisor integrated into Nutanix HCI.
- Customers may compare AHV to ESXi or Hyper-V, but support must focus on evidence and workflow differences.
- Migration issues often involve drivers, virtual hardware, networking, or tooling assumptions.
- Do not overpromise one-to-one behavior across hypervisors.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| AHV | Nutanix native Type 1 hypervisor. |
| Bare metal | Software running directly on physical hardware. |
| ESXi | VMware enterprise Type 1 hypervisor. |
| Hyper-V | Microsoft enterprise Type 1 hypervisor. |
| KVM | Linux kernel-based virtualization technology. |
| Type 1 hypervisor | Bare-metal hypervisor used for enterprise virtualization. |
| Type 2 hypervisor | Hosted hypervisor running on top of a general-purpose OS. |
| VirtIO | Paravirtualized driver model commonly used with AHV/KVM. |
| VM hardware | Virtualized devices presented to a guest OS. |
