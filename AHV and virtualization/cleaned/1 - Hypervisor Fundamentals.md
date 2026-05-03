# 1 - Hypervisor Fundamentals

## 1. Purpose of this chapter

This chapter establishes the virtualization baseline needed to understand AHV escalations. It focuses on the mental model a support manager needs: how physical resources become virtual resources, where symptoms can appear, and how to avoid blaming the hypervisor before the fault domain is isolated.

## 2. Core definition

A hypervisor is the software layer that allows multiple virtual machines to run on one physical server by abstracting CPU, memory, storage, and network resources. In Nutanix, AHV is the native hypervisor, but it operates as part of a larger HCI stack rather than as a standalone virtualization product.

## 3. Main explanation

A physical host contributes CPU cores, RAM, NICs, and disks. The hypervisor presents those resources to VMs as vCPUs, vRAM, vNICs, and vDisks. A VM then runs a guest operating system and applications on top of that virtual hardware.

In Nutanix, the virtualization path must be read together with the HCI path: application, guest OS, virtual hardware, AHV host, AOS services, CVM, distributed storage, physical network, and cluster health. AHV runs the virtual machines. AOS and the Controller VMs provide the distributed storage and platform services. Prism exposes management, monitoring, alerts, tasks, and operational workflows.

For enterprise support, the practical skill is fault-domain isolation. A customer may report that a VM is slow, frozen, unreachable, or unavailable. The root cause may be in the application, guest OS, VM configuration, hypervisor scheduling, storage latency, network loss, cluster services, or hardware. The support leader must make the investigation structured enough that specialists can move quickly without duplicating work or making unsupported assumptions.

## 4. Key concepts

- Physical resources become virtual resources: CPU to vCPU, RAM to vRAM, NIC to vNIC, disk to vDisk.
- Type 1 hypervisors run directly on server hardware and are the enterprise norm.
- VM symptoms are not automatically hypervisor defects.
- AHV must be understood with AOS, CVMs, Prism, networking, and storage path behavior.
- The escalation frame should separate symptom, impact, scope, recent change, and evidence.

## 5. Support relevance

Hypervisor fundamentals matter because many severe cases begin with an imprecise statement: “the VM is down” or “Nutanix is slow.” A support manager must translate that into testable domains: power state, guest health, CPU scheduling, memory pressure, storage latency, packet loss, Prism tasks, host health, and cluster alerts. The goal is not to personally debug every kernel detail, but to ensure the right evidence reaches the right SME quickly.

## 6. Escalation scenario

A customer reports that a revenue-critical VM became unreachable after a maintenance window. One team blames AHV, another blames the guest OS, and the network team says nothing changed. The escalation lead asks for the exact outage timeline, VM power state, Prism tasks, guest console status, host placement, recent migration events, network reachability tests, and cluster alerts. The team discovers the VM is powered on and healthy at the console, but its vNIC is attached to the wrong VLAN after a post-maintenance configuration change.

## 7. Triage questions

- What is the exact symptom: powered off, boot failure, unreachable, slow, or application error?
- Is the impact one VM, multiple VMs on one host, one subnet, one cluster, or one application?
- What changed immediately before the symptom appeared?
- Does the VM console work even if the network path fails?
- Do Prism metrics agree with guest OS metrics?
- Are there host, CVM, storage, or network alerts at the same timestamp?
- Which team owns the next evidence-producing action?

## 8. What to avoid

- Weak: “The hypervisor is probably the issue.” Better: “We need to isolate guest, host, storage, network, and control-plane evidence before naming the fault domain.”
- Weak: “The VM is up, so the platform is fine.” Better: “Power state is only one signal; guest health, I/O, network, and application checks still matter.”
- Weak: “Virtualization hides the hardware.” Better: “Virtualization abstracts hardware, but hardware pressure and failure still surface through the virtual layer.”

## 9. Minimum to memorize

- A hypervisor abstracts physical resources into virtual resources.
- AHV is Nutanix’s native hypervisor and part of the Nutanix HCI operating model.
- VM symptoms must be split across guest, VM, host, storage, network, cluster, and application domains.
- A support manager drives structure, evidence, ownership, and customer communication.
- Do not identify root cause from symptoms alone.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| AHV | Acropolis Hypervisor, Nutanix’s native hypervisor. |
| AOS | Nutanix software layer providing distributed storage and platform services. |
| CVM | Controller VM that runs Nutanix services on each node. |
| Guest OS | Operating system running inside a VM. |
| Hypervisor | Virtualization layer that runs and manages VMs. |
| Prism | Nutanix management and monitoring interface. |
| vCPU | Virtual CPU assigned to a VM. |
| vDisk | Virtual disk presented to a VM. |
| VM | Virtual machine. |
| vNIC | Virtual network interface assigned to a VM. |
