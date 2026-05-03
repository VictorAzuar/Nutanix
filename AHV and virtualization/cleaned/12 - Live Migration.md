# 12 - Live Migration

## 1. Purpose of this chapter

This chapter explains live migration as a planned continuity tool for maintenance, balancing, and operational flexibility. It focuses on constraints and failure triage rather than treating migration as magic.

## 2. Core definition

Live migration is the movement of a running VM from one AHV host to another, and in supported cases across clusters, without planned VM shutdown and with minimal service interruption.

## 3. Main explanation

Live migration moves a VM’s execution from a source host to a destination host. Conceptually, memory and CPU/device state are transferred while storage and network continuity are preserved. In Nutanix, migration depends on AHV, Prism workflows, host resources, AOS/storage health, networking, and placement policies.

Typical intra-cluster use cases include host maintenance, upgrades, workload balancing, isolating a host, and reducing contention. Cross-cluster live migration adds stricter requirements: compatible networks, sufficient capacity, CPU compatibility, Prism Central coordination, reachability, and supported versions.

A failed migration is not automatically a hypervisor failure. Common blockers include insufficient destination capacity, CPU feature mismatch, affinity rules, passthrough devices, VM state, high dirty memory rate, network mapping, storage issues, host health, or Prism task errors.

## 4. Key concepts

- Live migration is planned movement; HA is unplanned restart after failure.
- Destination host capacity and compatibility are prerequisites.
- Network continuity is critical for avoiding customer-visible impact.
- Some VM configurations cannot migrate cleanly.
- Cross-cluster migration has stricter compatibility and reachability requirements.

## 5. Support relevance

Live migration sits inside change management. If it fails during maintenance, the business risk is not just one task error; the maintenance window, upgrade plan, SLA, and customer trust may be at risk. A support manager must decide whether to retry, pause, roll back, or escalate.

## 6. Escalation scenario

During firmware maintenance, several VMs evacuate but one critical VM refuses to migrate. The maintenance window is closing. The escalation lead checks destination capacity, affinity, passthrough devices, CPU compatibility, VM activity rate, Prism error text, host health, and business criticality. The decision is to pause maintenance for that host, avoid unsafe forced action, and schedule a controlled application downtime after identifying a passthrough device constraint.

## 7. Triage questions

- Is the migration intra-cluster or cross-cluster?
- What exact Prism task error or event was recorded?
- Does the destination host have enough CPU and memory?
- Are CPU compatibility, network mapping, and storage requirements satisfied?
- Does the VM use passthrough, affinity, pinned placement, or unusual devices?
- Is the VM dirtying memory too quickly for migration to converge?
- What is the operational risk if migration is retried or maintenance continues?

## 8. What to avoid

- Weak: “Live migration should always be invisible.” Better: “It is designed for minimal disruption, but workload behavior and constraints can still create risk.”
- Weak: “Retry until it works.” Better: “Validate blockers before retrying to avoid extending impact.”
- Weak: “Migration failure means AHV is broken.” Better: “Check capacity, compatibility, policy, VM state, network, storage, and host health.”

## 9. Minimum to memorize

- Live migration is planned movement of a running VM.
- It depends on host capacity, compatibility, network, storage, and policy constraints.
- It is different from HA restart.
- Failed migration during maintenance requires risk control.
- Cross-cluster migration has stricter prerequisites.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| CPU compatibility | Ability of destination host to support required CPU features. |
| Destination host | Host receiving the migrated VM. |
| Dirty memory | Memory pages changed during migration. |
| Intra-cluster migration | Migration within one cluster. |
| Live migration | Moving a running VM without planned shutdown. |
| OD-CCLM | On-Demand Cross-Cluster Live Migration. |
| Passthrough | Direct device assignment that can restrict migration. |
| Source host | Host currently running the VM. |
