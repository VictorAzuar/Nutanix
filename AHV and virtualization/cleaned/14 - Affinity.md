# 14 - Affinity and Anti-Affinity

## 1. Purpose of this chapter

This chapter explains placement rules as operational risk tools. It focuses on how affinity and anti-affinity affect HA, migration, maintenance, licensing, and customer expectations.

## 2. Core definition

Affinity and anti-affinity rules control where VMs are allowed or preferred to run in a cluster. Affinity usually keeps a VM on specific hosts; anti-affinity attempts to keep related VMs apart.

## 3. Main explanation

Placement policies express business or technical intent. VM-host affinity may be needed for licensing, hardware locality, passthrough devices, or operational constraints. However, it reduces scheduler flexibility and can block HA restart, live migration, or maintenance mode if allowed hosts are unavailable.

VM-VM anti-affinity is used to reduce correlated failure risk. For example, domain controllers, load-balanced application nodes, or database replicas should not all run on the same physical host. In many implementations, anti-affinity is best-effort rather than an absolute guarantee, especially under capacity pressure or during operational workflows.

The support manager must distinguish “policy intent” from “availability guarantee.” Placement rules improve architecture only when there is enough capacity and when the customer understands the trade-offs.

## 4. Key concepts

- VM-host affinity can be a hard constraint.
- VM-VM anti-affinity is often best-effort placement intent.
- Placement rules can help resiliency but also reduce flexibility.
- Affinity can block HA, live migration, and maintenance mode.
- Categories or groups may be used to manage policy at scale.

## 5. Support relevance

Affinity cases are support-relevant because customers may create rules for licensing or resiliency and later be surprised when operations are blocked. The support manager should ask whether the rule is still required, what risk it was designed to reduce, and what new risk it introduces.

## 6. Escalation scenario

A host fails and a licensing VM with host affinity does not restart. The customer expects HA to recover it. The escalation lead confirms the VM was restricted to the failed host and one other host that lacked available memory. The team restores service by freeing capacity on the allowed host, then recommends redesigning the licensing constraint and HA reservation model.

## 7. Triage questions

- Which VMs are governed by affinity or anti-affinity rules?
- Are the rules mandatory or best-effort?
- Why were the rules created: licensing, resiliency, performance, locality, or operations?
- Do the rules block HA, migration, or maintenance?
- Is there enough capacity to satisfy the placement policy during failure?
- Did recent host failure or maintenance expose a policy constraint?
- Should the customer revise the rule or add capacity?

## 8. What to avoid

- Weak: “Anti-affinity guarantees separation.” Better: “Anti-affinity expresses separation intent; enforcement depends on platform behavior, capacity, and policy type.”
- Weak: “Affinity improves control with no downside.” Better: “Affinity can reduce scheduler flexibility and recovery options.”
- Weak: “HA failed.” Better: “Check whether placement policy prevented a valid HA target.”

## 9. Minimum to memorize

- Affinity controls where a VM can run.
- Anti-affinity attempts to keep related VMs apart.
- Hard constraints can block recovery workflows.
- Placement rules require enough capacity to be useful.
- Always connect placement policy to business intent and operational risk.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Affinity | Placement rule keeping a VM on selected hosts. |
| Anti-affinity | Placement rule intended to separate VMs. |
| Blast radius | Scope of impact from one failure. |
| Category | Logical grouping used for policy assignment. |
| Hard rule | Mandatory placement constraint. |
| Placement policy | Rule influencing where VMs run. |
| Soft rule | Best-effort placement preference. |
| VM-host affinity | Rule tying a VM to host(s). |
| VM-VM anti-affinity | Rule attempting to separate related VMs. |
