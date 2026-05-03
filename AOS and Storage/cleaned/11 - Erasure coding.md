# 11 - Erasure Coding

## 1. Purpose of this chapter
This chapter explains erasure coding as a capacity-efficiency mechanism with resiliency and performance trade-offs. Support managers need this because customers may expect instant savings without understanding cold data, parity, rebuild behavior, or failure-domain design.

## 2. Core definition
**Erasure coding (EC / EC-X)** is a storage-efficiency technique that reduces capacity overhead by using data fragments plus parity instead of keeping full replicated copies for eligible data.

## 3. Main explanation
Replication protects data by keeping full copies. Erasure coding protects eligible data more space-efficiently by calculating parity across a stripe. If a component fails, missing data can be reconstructed from remaining fragments and parity.

In Nutanix support conversations, EC should be framed as a capacity feature, not a performance accelerator. It is typically more useful for colder data and may show delayed or workload-dependent savings. During failures or rebuilds, reconstructing data can introduce additional work, so the current cluster state matters.

A mature answer is not “EC saves capacity.” It is: EC can improve usable capacity when workloads, cluster size, failure domains, and platform state are suitable, but it must be managed carefully during degraded states or performance escalations.

## 4. Key concepts
- **Parity**: calculated information used to reconstruct missing data.
- **Stripe**: group of data/parity fragments used by EC.
- **Cold data**: data that is not actively changing and is more suitable for EC.
- **Savings delay**: EC savings may appear only after eligibility and background processing.
- **Failure-domain awareness**: parity/data placement must respect failure boundaries.
- **Read/rebuild overhead**: reconstruction can consume resources during failure.

## 5. Support relevance
EC escalations often involve expected savings not appearing, capacity pressure despite EC being enabled, latency during rebuild, or confusion between replication and parity. The manager should align the customer on workload eligibility, current health, and whether EC is relevant to the immediate problem.

## 6. Escalation scenario
A customer enabled EC-X on a cluster with capacity pressure. A week later, savings are below expectations. Then a node failure occurs and the customer reports latency during re-protection.

The response should separate two issues: savings eligibility and failure-state performance. Validate the write-cold window or eligibility criteria, EC scope, workload churn, background processing, cluster capacity, RF state, and whether rebuild/reconstruction is contributing to latency. Avoid blaming EC without evidence, but avoid claiming it is free.

## 7. Triage questions
1. Where is EC enabled and for which data/workloads?
2. Is the data cold enough or stable enough to be eligible?
3. What savings were expected and who set that expectation?
4. Are compression or deduplication also enabled?
5. Is the cluster healthy or degraded?
6. Is rebuild, re-protection, or rebalancing active?
7. Are latency symptoms read-heavy, write-heavy, or rebuild-related?
8. Does the cluster topology support the relevant failure domain?
9. Is the case about capacity planning, performance, or failure recovery?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Erasure coding is just better replication.” | “EC trades full-copy overhead for parity-based efficiency and has different operational behavior.” |
| “EC gives instant savings.” | “Savings depend on eligibility, workload churn, and background processing.” |
| “EC improves performance.” | “EC is primarily a capacity-efficiency feature; performance impact must be measured.” |
| “EC removes the need for RF.” | “EC complements the protection design but does not remove the need to understand resiliency state.” |

## 9. Minimum to memorize
Erasure coding uses parity to reduce capacity overhead for eligible data. It is mainly a capacity-efficiency feature, not a guaranteed performance improvement. Savings and risk depend on workload coldness, cluster health, topology, and rebuild state.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Cold data | Data with low write/change activity and better EC suitability. |
| EC | Erasure Coding; parity-based capacity-efficiency method. |
| EC-X | Nutanix erasure coding implementation/feature name. |
| Failure domain | Independent failure boundary such as node, block, or rack. |
| Parity | Calculated information used to reconstruct missing data. |
| Rebuild | Recovery process after component failure. |
| RF | Replication Factor; full-copy protection model. |
| Stripe | Grouping of data and parity fragments. |
| Usable capacity | Capacity available to workloads after protection overhead. |
| Write-cold | Data that has not changed for a defined period. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
