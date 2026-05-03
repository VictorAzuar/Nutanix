# 10 - Deduplication

## 1. Purpose of this chapter
This chapter explains deduplication as a capacity-optimization feature that must be tied to workload patterns and operational risk. Support managers need this because customers often expect fixed savings percentages without understanding duplicate-block suitability, metadata overhead, or timing.

## 2. Core definition
**Deduplication** reduces physical storage usage by identifying duplicate data blocks and storing a shared copy while maintaining logical references for workloads that need that data.

## 3. Main explanation
Deduplication is most useful when many workloads share identical data: VDI desktops, VM templates, golden images, OS binaries, application stacks, and similar cloned environments. It is usually less useful for unique, encrypted, compressed, or rapidly changing datasets.

The support nuance is that deduplication is not free. It may require fingerprinting, metadata tracking, memory, CPU, and background processing. Savings should be evaluated against workload type, cluster sizing, AOS version, and the customer’s performance and resiliency requirements.

Deduplication should not be treated as an emergency capacity switch. It may help the long-term footprint, but urgent capacity incidents usually still require safe cleanup, snapshot management, workload control, or expansion.

## 4. Key concepts
- **Duplicate block**: identical stored data that can be referenced once.
- **Fingerprinting**: method used to identify identical data patterns.
- **Metadata overhead**: tracking references consumes system resources.
- **VDI/template suitability**: repeated OS images often dedupe well.
- **Savings variability**: results depend heavily on workload content.
- **Capacity vs performance trade-off**: saving space can introduce processing cost.

## 5. Support relevance
Deduplication escalations often start with statements like “we expected 40% savings,” “capacity still grows,” or “performance changed after enabling dedupe.” A support manager should align expectations, confirm configuration, understand workload suitability, and avoid overpromising savings.

## 6. Escalation scenario
A customer running mixed database, log, and VDI workloads enables deduplication and sees high savings for desktops but almost none for databases. The customer assumes dedupe is malfunctioning.

The correct response is to explain workload variance. VDI may contain many identical OS and application blocks; databases and logs may have more unique or changing data. The team should validate dedupe scope, data type, measurement window, and whether capacity pressure comes from snapshots, clones, or growth rather than dedupe failure.

## 7. Triage questions
1. Which workloads are under the deduplication policy?
2. Are those workloads expected to contain duplicate blocks?
3. Is data encrypted, compressed, random, or high-churn?
4. What savings are visible by logical versus physical usage?
5. Did dedupe get enabled before or during a crisis?
6. Are snapshots, clones, backups, or replication contributing to usage?
7. Has performance changed since enabling dedupe?
8. Are cluster CPU, memory, and metadata health normal?
9. What exact customer expectation was set about savings?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Deduplication saves space on all workloads.” | “Deduplication depends on duplicate-block patterns and workload type.” |
| “Low savings means the feature is broken.” | “Low savings may be normal for unique, encrypted, or high-churn data.” |
| “Enable dedupe to solve this incident.” | “Treat dedupe as optimization; urgent capacity risk needs validated containment.” |
| “Dedupe has no performance cost.” | “Deduplication can introduce metadata and processing overhead.” |

## 9. Minimum to memorize
Deduplication stores duplicate data once and references it logically. It works best with repeated data patterns and may be weak for unique, encrypted, compressed, or changing workloads. Never promise fixed savings.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Clone | Copy that may share base data and increase dedupe opportunity. |
| Deduplication | Avoiding repeated physical storage of identical data. |
| Duplicate block | Data block identical to another stored block. |
| Fingerprint | Signature used to identify duplicate data. |
| Golden image | Standard source image often duplicated across many VMs. |
| High-churn workload | Workload that changes data frequently. |
| Metadata overhead | Resource cost of tracking dedupe references. |
| Physical usage | Backend capacity actually consumed. |
| Template | Reusable VM image that can create duplicate data patterns. |
| VDI | Virtual Desktop Infrastructure; often dedupe-friendly. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
