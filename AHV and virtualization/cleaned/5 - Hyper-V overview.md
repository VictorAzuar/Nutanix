# 5 - Hyper-V Overview

## 1. Purpose of this chapter

This chapter summarizes Hyper-V as another enterprise hypervisor context for Nutanix support, especially for migration, Windows workload expectations, clustering terminology, and customer communication.

## 2. Core definition

Hyper-V is Microsoft’s Type 1 hypervisor for running Windows and Linux VMs. It is commonly integrated with Windows Server, Failover Clustering, virtual switches, PowerShell, System Center, Windows Admin Center, and Microsoft disaster recovery tooling.

## 3. Main explanation

Hyper-V customers often think in Microsoft ecosystem terms: Windows Server hosts, Failover Clustering, Cluster Shared Volumes, Hyper-V Replica, virtual switches, PowerShell, and Windows guest integration. When such customers move to AHV, the VM concept remains familiar but the platform responsibilities change.

In AHV, VM runtime is integrated with Nutanix AOS, CVMs, Prism, and distributed storage. Networking shifts from Hyper-V virtual switch constructs to AHV virtual networking, VLANs, OVS-related behavior, and possibly Nutanix Flow Virtual Networking. Guest integration shifts from Hyper-V Integration Services to Nutanix Guest Tools and VirtIO drivers where applicable.

For support leadership, the key is translation. When a customer says a migrated Windows VM “worked on Hyper-V,” ask which layer changed: driver, boot mode, disk controller, NIC, IP plan, clustering configuration, backup agent, domain policy, time sync, or storage behavior.

## 4. Key concepts

- Hyper-V is deeply tied to Windows Server and Microsoft management tools.
- Hyper-V migrations to AHV often involve Windows drivers, boot configuration, virtual NICs, and backup agents.
- Failover Clustering and AHV HA are not the same thing.
- Hyper-V Replica and Nutanix DR/protection workflows require separate planning.
- Microsoft terminology should be translated carefully into Nutanix equivalents.

## 5. Support relevance

Hyper-V knowledge helps avoid miscommunication with Windows-centric enterprise customers. It also helps the support manager identify when the issue belongs to guest OS configuration, Microsoft clustering, Active Directory, backup software, or Nutanix platform behavior.

## 6. Escalation scenario

A customer migrates a Windows file server from Hyper-V to AHV. The VM boots, but users report intermittent disconnects. The team checks AHV network metrics, VLAN mapping, VirtIO NIC driver, Windows event logs, SMB errors, DNS, MTU, and whether the VM was part of a Hyper-V Failover Cluster role. The issue is traced to an outdated NIC driver and a retained static route from the old environment.

## 7. Triage questions

- Was the VM migrated from standalone Hyper-V or a Failover Cluster?
- Did the boot mode, disk controller, NIC model, or driver stack change?
- Are Windows event logs showing storage, network, time sync, or driver errors?
- Is the workload dependent on Microsoft clustering, SMB, iSCSI, or domain policy?
- Does the problem follow the VM, host, subnet, or user location?
- Are backup, antivirus, or monitoring agents compatible after migration?
- Is the customer expecting Hyper-V Replica behavior from a different Nutanix protection workflow?

## 8. What to avoid

- Weak: “Hyper-V and AHV are basically the same.” Better: “They both run VMs, but their management, storage, networking, and guest integration models differ.”
- Weak: “This is a Microsoft issue.” Better: “We need evidence to separate Windows guest behavior from AHV platform behavior.”
- Weak: “If it booted, migration succeeded.” Better: “Boot success does not validate drivers, networking, performance, protection, or application behavior.”

## 9. Minimum to memorize

- Hyper-V is Microsoft’s enterprise hypervisor.
- Hyper-V customers may bring Windows-centric assumptions into AHV cases.
- Windows VM migration issues often involve drivers, networking, boot, clustering, or backup tooling.
- Failover Clustering is application/OS-level and must not be confused with AHV HA.
- Translate concepts without promising identical behavior.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| AHV | Nutanix native hypervisor. |
| CSV | Cluster Shared Volume in Microsoft clustering. |
| Failover Clustering | Microsoft HA clustering technology. |
| Hyper-V | Microsoft Type 1 hypervisor. |
| Hyper-V Replica | Microsoft VM replication feature. |
| NGT | Nutanix Guest Tools. |
| SMB | Server Message Block protocol. |
| VirtIO | Paravirtualized AHV/KVM drivers. |
| Windows Admin Center | Microsoft management interface for Windows infrastructure. |
