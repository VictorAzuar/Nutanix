# 17 - VM Performance Troubleshooting

## 1. Purpose of this chapter

This chapter consolidates the module into a practical escalation workflow for slow, unstable, or inconsistent VMs on AHV. It is the operating playbook chapter.

## 2. Core definition

VM performance troubleshooting on AHV is the structured process of identifying why a VM is slow or unstable by isolating guest OS, application, VM configuration, AHV host, CVM/storage, network, cluster health, and external dependencies.

## 3. Main explanation

Performance troubleshooting starts with symptom precision. “Slow” is not a diagnosis. Slow login, slow query, high latency, packet loss, CPU spikes, I/O wait, failed backups, and intermittent application hangs require different evidence.

The workflow is: define impact and timeline; scope blast radius; compare affected and unaffected systems; check recent changes; correlate guest metrics with Prism metrics; identify the dominant resource plane; validate host/CVM/cluster/network health; test hypotheses; communicate progress and next actions.

The manager’s value is in keeping the investigation from fragmenting. CPU, memory, storage, network, guest, database, backup, and application owners may all have partial evidence. The escalation lead turns that evidence into a timeline, decision log, mitigation path, and RCA-quality narrative.

## 4. Key concepts

- Start with symptom, impact, scope, and timeline.
- Compare guest metrics with Prism/AHV metrics.
- Separate CPU, memory, disk, network, guest OS, application, and cluster domains.
- Recent changes often explain sudden regressions.
- Mitigation and root cause can be separate workstreams.

## 5. Support relevance

Performance escalations test support leadership because customers often want immediate answers before evidence exists. The manager must communicate uncertainty honestly, drive parallel triage, and avoid both overpromising and analysis paralysis.

## 6. Escalation scenario

A customer reports that several VMs are intermittently slow after a weekend maintenance window. The support manager establishes a bridge, defines affected applications, collects timelines, separates VMs by host and subnet, checks Prism metrics, reviews migration and upgrade tasks, validates CVM health, and asks application teams for transaction timestamps. The investigation identifies packet drops on one top-of-rack switch path that affected migrated VMs on specific hosts.

## 7. Triage questions

- What exactly is slow and how is it measured?
- When did the symptom start and what changed before it?
- Is the issue one VM, one host, one subnet, one storage container, one cluster, or one application?
- Which resource plane shows abnormal behavior?
- Do guest metrics and Prism metrics agree or contradict each other?
- Are backups, snapshots, migrations, upgrades, or maintenance tasks overlapping?
- What mitigation can reduce impact while root cause analysis continues?

## 8. What to avoid

- Weak: “Everything is slow.” Better: “Name the exact symptom, metric, timestamp, and affected scope.”
- Weak: “No alerts means no platform issue.” Better: “Absence of alerts is useful but not conclusive; correlate metrics and customer timestamps.”
- Weak: “We need RCA before mitigation.” Better: “Stabilize customer impact while preserving evidence for RCA.”

## 9. Minimum to memorize

- Performance triage begins with precise symptom definition.
- Scope determines likely fault domain.
- Guest, Prism, host, CVM, storage, and network evidence must be correlated.
- Recent changes are high-value evidence.
- Support leadership balances mitigation, communication, and RCA quality.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Blast radius | Extent of affected systems or users. |
| CVM | Controller VM. |
| I/O wait | Guest-side waiting on storage or I/O completion. |
| Mitigation | Action to reduce impact before final root cause is proven. |
| MTTR | Mean Time To Resolution/Recovery. |
| RCA | Root Cause Analysis. |
| Storage latency | Delay in completing storage operations. |
| Symptom | Observable problem reported or measured. |
| Timeline | Ordered sequence of events and changes. |
