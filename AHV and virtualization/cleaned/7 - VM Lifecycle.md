# 7 - VM Lifecycle

## 1. Purpose of this chapter

This chapter turns VM operations into a support triage model. It covers the lifecycle from creation through deletion and explains how lifecycle actions become escalation triggers.

## 2. Core definition

VM lifecycle is the set of operations used to create, configure, start, stop, resize, migrate, snapshot, protect, recover, monitor, and delete a virtual machine.

## 3. Main explanation

In AHV, VM lifecycle actions are typically performed through Prism Element, Prism Central, APIs, or automation. A VM is not just a name and power state. It has CPU, memory, disks, NICs, guest OS, drivers, snapshots, protection settings, placement policies, and historical tasks.

Many support cases are lifecycle cases: VM will not power on, resize did not apply, snapshot caused confusion, migration failed, clone behaves differently, delete was accidental, recovery point is missing, or Prism task is stuck. Each case requires a timeline. What was the requested action? Who or what initiated it? What prerequisites were assumed? What state did Prism report? What changed inside the guest?

A support manager should ensure lifecycle triage separates desired state from actual state. For example, a VM may be “powered on” but the guest OS may be hung; a snapshot may exist but not provide the recovery granularity the customer expects; a migration task may fail because of host capacity, network mapping, affinity, passthrough, or CPU compatibility.

## 4. Key concepts

- VM lifecycle includes provisioning, configuration, operation, protection, movement, recovery, and retirement.
- Prism tasks and event history are central evidence.
- Lifecycle failures often come from prerequisites, policy constraints, capacity, compatibility, or guest behavior.
- Snapshots are not a complete backup strategy by default.
- Power state, guest health, application health, and network reachability are different states.

## 5. Support relevance

Lifecycle issues are common because they sit at the boundary between customer action and platform behavior. The support manager must clarify risk, protect data, stop unsafe repeated attempts, and coordinate the right SME before the customer turns an operational mistake into data loss or prolonged outage.

## 6. Escalation scenario

A customer tries to expand CPU and memory for a production VM during a change window. The VM becomes unstable and the application team demands rollback. The escalation lead captures the pre-change configuration, whether hot-add was supported, guest OS behavior, Prism task status, snapshots or backups, application owner approval, and rollback path. The team discovers the VM was over-provisioned across NUMA boundaries and the guest application required a controlled restart.

## 7. Triage questions

- What lifecycle action was attempted and by whom?
- Was it performed through Prism, API, automation, or guest OS tools?
- What was the VM state before the action?
- What did Prism tasks and events report?
- Were snapshots, backups, or protection policies available before the change?
- Did the issue affect power state, guest OS, application health, network, or storage?
- Is rollback safer than continued troubleshooting in place?

## 8. What to avoid

- Weak: “The VM is up, so lifecycle is fine.” Better: “We must verify guest, application, network, storage, and recent task state.”
- Weak: “Just retry the task.” Better: “First understand why the task failed and whether retrying increases risk.”
- Weak: “A snapshot means we are protected.” Better: “Confirm snapshot type, consistency, restore scope, retention, and whether backup/DR requirements are met.”

## 9. Minimum to memorize

- VM lifecycle spans create, configure, power, resize, migrate, protect, recover, and delete.
- Prism task history is critical evidence.
- Desired state and actual state must be separated.
- Repeated lifecycle retries can increase risk.
- Support leadership must balance restoration, data protection, and change control.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Clone | Copy of a VM or disk created from an existing source. |
| Guest OS | Operating system inside the VM. |
| Lifecycle | Set of operational states and actions for a VM. |
| Prism task | Recorded Nutanix workflow action. |
| Recovery point | Point in time available for restore. |
| Snapshot | Point-in-time capture of VM disk state or metadata. |
| vDisk | Virtual disk. |
| VM | Virtual machine. |
| vNIC | Virtual network adapter. |
