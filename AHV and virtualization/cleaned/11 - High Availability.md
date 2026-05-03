# 11 - High Availability

## 1. Purpose of this chapter

This chapter explains AHV High Availability as a recovery capability with capacity and policy constraints. It emphasizes expectation management: HA reduces downtime risk, but it is not a guarantee that every workload restarts in every condition.

## 2. Core definition

High Availability in AHV is the capability to restart affected VMs on surviving hosts after a host failure, subject to available capacity, HA configuration, cluster health, and placement constraints.

## 3. Main explanation

HA answers a simple question: if a host fails, where can its VMs restart? In AHV, compute recovery depends on CPU and memory availability on remaining hosts. Storage availability depends on Nutanix AOS resiliency, replication, and cluster health.

Best-effort HA attempts to restart VMs when resources are available. Guaranteed HA or reservation-based models preserve capacity for failure scenarios, reducing the risk that a host failure leaves VMs powered off. Affinity, anti-affinity, passthrough devices, pinned VMs, insufficient resources, or degraded cluster health can all affect outcomes.

HA must not be confused with live migration. Live migration is planned movement of a running VM. HA restart is recovery after unplanned host failure and normally involves restart behavior, not seamless continuation of CPU execution.

## 4. Key concepts

- HA protects against host failure by restarting VMs elsewhere.
- Best-effort HA depends on available resources at the time of failure.
- Guaranteed HA requires capacity reservation and planning.
- HA is not the same as application-level clustering or disaster recovery.
- Affinity, passthrough, and capacity constraints can limit HA behavior.

## 5. Support relevance

HA escalations are high-stakes because customers often believe “HA enabled” means no downtime and guaranteed recovery. A support manager must clarify actual configuration, remaining capacity, affected workloads, restart order, business impact, and whether the customer also needs application-level HA or DR.

## 6. Escalation scenario

A host fails during business hours. Most VMs restart, but several remain powered off. The customer claims HA failed. The escalation lead checks HA mode, available memory on surviving hosts, affinity rules, passthrough devices, VM priority, recent overcommit, and cluster health before explaining that best-effort HA lacked sufficient reserved capacity for all workloads. The immediate action is restoration prioritization; the follow-up is HA capacity design review.

## 7. Triage questions

- Was HA configured as best-effort or guaranteed/reserved?
- Which host failed and which VMs were affected?
- Did VMs restart, remain powered off, or boot with guest/application errors?
- Was there enough CPU and memory on surviving hosts?
- Were affinity, anti-affinity, passthrough, or pinned configurations involved?
- Was storage resiliency healthy before and after the failure?
- What is the customer’s RTO expectation and does HA alone meet it?

## 8. What to avoid

- Weak: “HA means no downtime.” Better: “HA reduces downtime by restarting workloads, but restart success depends on capacity, policies, and health.”
- Weak: “All VMs should restart automatically.” Better: “We need to validate HA mode, reservations, constraints, and available resources.”
- Weak: “HA is DR.” Better: “HA handles local host failure; DR handles broader site or cluster recovery scenarios.”

## 9. Minimum to memorize

- AHV HA restarts VMs after host failure.
- Best-effort HA is not guaranteed recovery.
- Capacity reservation matters.
- Live migration and HA restart are different.
- Customer RTO may require application HA or DR beyond AHV HA.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Affinity | Rule constraining where a VM can run. |
| Best-effort HA | HA behavior without guaranteed reserved capacity. |
| Guaranteed HA | HA mode with reserved recovery capacity. |
| HA | High Availability. |
| Live migration | Planned movement of a running VM. |
| RTO | Recovery Time Objective. |
| VM restart | Powering on a VM after failure on another host. |
