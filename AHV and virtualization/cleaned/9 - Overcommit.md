# 9 - Overcommit

## 1. Purpose of this chapter

This chapter explains overcommit as a deliberate capacity trade-off rather than a magic efficiency feature. It focuses on how overcommit affects risk, performance, HA, and escalation communication.

## 2. Core definition

Overcommit means assigning more virtual resources to VMs than the physical host or cluster has available, based on the expectation that not all workloads will consume their full allocation simultaneously.

## 3. Main explanation

CPU overcommit is common in virtualization because many VMs are idle or bursty. The risk appears when many vCPUs demand physical CPU time at once, causing scheduling delay and higher latency.

Memory overcommit is more sensitive. Reclamation techniques such as ballooning can help, but host swapping can create severe performance degradation. Production environments should treat memory overcommit cautiously, especially for databases, latency-sensitive applications, and clusters that rely on HA reservations.

Overcommit must be evaluated at several levels: individual VM sizing, host contention, cluster headroom, HA capacity, maintenance mode ability, workload cycles, and customer SLAs. The support manager should frame overcommit as a risk-management topic, not merely a utilization metric.

## 4. Key concepts

- CPU overcommit is about scheduling fairness and latency.
- Memory overcommit can degrade sharply if reclamation or swapping occurs.
- Overcommit ratios are starting points, not universal guarantees.
- HA and maintenance operations need real spare capacity.
- Oversized VMs can create contention even when average utilization looks low.

## 5. Support relevance

Overcommit cases often become political because customers want higher consolidation and lower cost, while support sees performance and availability risk. A support manager must communicate with evidence: measured demand, contention, HA headroom, affected workloads, and safe remediation options.

## 6. Escalation scenario

A customer consolidated many production VMs onto fewer hosts and now reports intermittent slowness during business hours. Prism shows CPU scheduling delay and memory pressure on two hosts, while the customer points to low average cluster utilization. The escalation lead explains peak concurrency, host-level hotspots, and HA headroom. The action plan includes right-sizing oversized VMs, balancing workloads, reviewing memory overcommit, and delaying further consolidation until risk is reduced.

## 7. Triage questions

- What are the configured vCPU:pCPU and memory ratios by host and cluster?
- Are symptoms aligned with peak workload periods?
- Is contention host-local or cluster-wide?
- Are critical VMs oversized or latency-sensitive?
- Is memory ballooning or host swap present?
- Can the cluster tolerate a host failure with current reservations?
- Did recent consolidation, migration, or policy changes alter overcommit risk?

## 8. What to avoid

- Weak: “The cluster average is fine.” Better: “Check host-level contention, peak demand, and HA headroom.”
- Weak: “Overcommit is always bad.” Better: “CPU overcommit can be safe when measured; memory overcommit requires stricter production caution.”
- Weak: “Add nodes and performance will improve.” Better: “Add capacity only after confirming the bottleneck and balancing plan.”

## 9. Minimum to memorize

- Overcommit trades utilization efficiency for performance and availability risk.
- CPU and memory overcommit behave differently.
- Peak demand matters more than averages during incidents.
- HA and maintenance require spare capacity.
- Right-sizing reduces risk more safely than blind expansion.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Ballooning | Guest-assisted memory reclamation. |
| Host swap | Hypervisor swapping memory to disk under pressure. |
| Overcommit | Allocating more virtual resources than physical resources. |
| pCPU | Physical CPU core or thread resource. |
| Right-sizing | Adjusting VM resources to actual workload needs. |
| vCPU:pCPU | Ratio of virtual CPUs to physical CPU resources. |
| Workload peak | Period of highest concurrent demand. |
