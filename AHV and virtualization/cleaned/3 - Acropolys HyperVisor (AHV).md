# 3 - Acropolis Hypervisor (AHV)

## 1. Purpose of this chapter

This chapter gives the central AHV operating model for a support leadership role: what AHV does, what it does not do, and how it connects to AOS, CVMs, Prism, HA, live migration, virtual networking, and customer escalations.

## 2. Core definition

AHV, or Acropolis Hypervisor, is Nutanix’s native enterprise hypervisor. It runs VMs on Nutanix nodes and is managed through Prism while relying on AOS and CVMs for distributed storage and platform services.

## 3. Main explanation

AHV provides the VM runtime: vCPU scheduling, memory allocation, virtual disks, vNICs, VM power operations, lifecycle workflows, live migration, HA, and virtual networking integration. It is based on KVM concepts, but customers do not normally operate it like a generic Linux KVM host. Nutanix wraps the virtualization layer in Prism workflows and integrates it with AOS.

AHV runs the virtual machines. AOS and the Controller VMs provide the distributed storage and platform services. Prism exposes management, monitoring, alerts, tasks, and operational workflows. This separation matters. AHV is not AOS. AOS is the distributed infrastructure and data-services layer. AHV is the virtualization layer. Prism is the management and visibility layer.

In support, AHV cases often present as VM failure, migration failure, HA restart behavior, performance degradation, networking misconfiguration, guest driver problems, or Prism task errors. The manager-level discipline is to lead a layered investigation: guest, VM config, AHV host, CVM, storage, network, Prism task, recent changes, and external dependencies.

## 4. Key concepts

- AHV runs VMs; AOS provides distributed platform services.
- CVMs are critical because storage and platform behavior affect VM symptoms.
- Prism is the operational interface for AHV workflows, metrics, alerts, and tasks.
- AHV includes live migration, HA, VM lifecycle, virtual networking, and scheduling functions.
- Support investigations should isolate whether the issue is AHV-specific or caused by guest OS, storage, network, host, or customer application behavior.

## 5. Support relevance

AHV is strategically important because it removes the need for a separate third-party hypervisor stack in many Nutanix environments. For support, that simplifies some ownership boundaries but increases the need for Nutanix-specific escalation discipline. Customers expect one team to understand virtualization, HCI storage, network dependencies, lifecycle, and incident communication together.

## 6. Escalation scenario

A customer opens a Sev1 after several VMs become slow after a cluster upgrade. The support manager separates the case into workstreams: Prism task history, AHV host health, VM CPU Ready or scheduling delay, memory pressure, CVM health, storage latency, network drops, guest OS counters, and recent post-upgrade alerts. The team discovers one host has network errors affecting CVM traffic, causing storage latency symptoms that looked like a hypervisor problem.

## 7. Triage questions

- Are affected VMs concentrated on one AHV host or spread across the cluster?
- Do guest OS metrics, Prism metrics, and application metrics agree?
- Were there recent upgrades, migrations, maintenance mode operations, or policy changes?
- Are CVMs healthy and reachable?
- Is storage latency elevated for affected VMs or cluster-wide?
- Are virtual networks, VLANs, and physical uplinks healthy?
- Does the case require AHV, storage, network, or engineering escalation?

## 8. What to avoid

- Weak: “AHV is the whole Nutanix platform.” Better: “AHV is the hypervisor; AOS, CVMs, and Prism are separate but tightly integrated components.”
- Weak: “If Prism shows the VM running, AHV is fine.” Better: “Running state is not enough; performance, I/O path, guest health, and network reachability also matter.”
- Weak: “This is definitely an AHV bug.” Better: “We need evidence that isolates AHV from guest, storage, network, host, and workflow causes.”

## 9. Minimum to memorize

- AHV is Nutanix’s native hypervisor.
- AHV runs VMs; AOS/CVMs deliver distributed infrastructure services.
- Prism is the management and observability layer.
- AHV escalations require layered triage across VM, host, storage, network, and control plane.
- A support manager must communicate impact and evidence without overclaiming root cause.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| ADS | Acropolis Dynamic Scheduler for placement and contention management. |
| AHV | Acropolis Hypervisor. |
| AOS | Nutanix distributed infrastructure software. |
| CVM | Controller VM providing Nutanix services on each node. |
| HA | High Availability restart or recovery behavior for VMs. |
| KVM | Linux virtualization foundation used by AHV. |
| Prism | Nutanix management and monitoring interface. |
| vDisk | Virtual disk attached to a VM. |
| VM | Virtual machine. |
| vNIC | Virtual network interface. |
