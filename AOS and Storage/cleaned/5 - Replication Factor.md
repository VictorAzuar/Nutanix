# 5 - Replication Factor

## 1. Purpose of this chapter
This chapter explains Replication Factor as the core Nutanix resiliency concept. A support manager must understand RF because customer risk during disk, node, block, or rack failures depends on current protection state, not on generic claims that the platform is resilient.

## 2. Core definition
**Replication Factor (RF)** defines how many copies of data AOS maintains across the cluster for availability and failure tolerance. RF2 generally means two copies; RF3 generally means three copies, subject to design and failure-domain requirements.

## 3. Main explanation
Nutanix protects data by storing redundant copies across different failure domains. This is different from thinking only in terms of traditional RAID groups. RF affects usable capacity, rebuild behavior, and customer risk.

RF2 is common and typically protects against one relevant failure when the cluster is healthy and correctly designed. RF3 provides stronger tolerance but consumes more capacity and requires appropriate cluster size and layout. A cluster can be online but degraded if it has lost one copy and has not yet restored the expected redundancy.

For support leadership, RF is a communication tool: it helps explain what the system was designed to tolerate, what has happened, what risk remains, and why some actions should wait until re-protection completes.

## 4. Key concepts
- **RF2**: two copies of data.
- **RF3**: three copies of data, with stricter capacity and topology needs.
- **Failure domain**: independent unit of failure such as disk, node, block, rack, or site.
- **Degraded state**: available but below desired protection.
- **Re-protection**: restoring data copies after a failure.
- **Capacity trade-off**: more copies improve tolerance but reduce usable capacity.

## 5. Support relevance
RF appears in almost every serious storage escalation: disk failure, node failure, maintenance planning, cluster expansion, capacity pressure, and customer questions about data loss risk. A manager should avoid binary reassurance and instead communicate current protection: healthy, degraded, rebuilding, or exposed to additional failure.

## 6. Escalation scenario
A customer with RF2 loses one node during a production window. VMs restart elsewhere and data remains accessible, but Prism reports degraded resiliency.

The manager should say that availability and full protection are different. The immediate goals are to confirm VM impact, validate surviving replicas, monitor re-protection, avoid additional disruptive maintenance, check capacity headroom, and coordinate hardware or node recovery.

## 7. Triage questions
1. What RF is configured for the affected workloads or container?
2. Is the cluster currently fully protected, degraded, or rebuilding?
3. What failure domain was lost: disk, node, block, rack, or site?
4. Is there enough capacity to restore the configured RF?
5. Are any additional disks, nodes, CVMs, or networks unhealthy?
6. Is re-protection progressing or stalled?
7. Are there planned operations that should be paused?
8. What risk wording has been given to the customer?
9. Does the topology actually support the claimed tolerance?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “RF2 means no risk.” | “RF2 generally tolerates one relevant failure when healthy, but risk increases while degraded.” |
| “RF3 is always better.” | “RF3 improves tolerance but consumes more capacity and requires suitable design.” |
| “The VM is online, so protection is fine.” | “Availability and protection state must be checked separately.” |
| “Rebuild is automatic, so no action is needed.” | “Rebuild is automatic, but we must monitor progress, capacity, and secondary risk.” |

## 9. Minimum to memorize
RF defines the number of data copies. RF2 means two copies; RF3 means three copies. After a failure, the cluster may stay online but degraded until re-protection restores the expected copy count. Always communicate current protection state, not just platform capability.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Availability Domain | Logical grouping used to place replicas across independent failure areas. |
| Block Awareness | Placement strategy that considers hardware blocks as failure domains. |
| Capacity overhead | Usable-capacity cost of storing redundant data. |
| Degraded State | Operational state with reduced redundancy or protection. |
| Failure Domain | Unit that can fail independently, such as node or rack. |
| Fault Tolerance | Number or class of failures a design can survive. |
| RF | Replication Factor; number of data copies maintained. |
| RF2 | Two-copy data protection model. |
| RF3 | Three-copy data protection model. |
| Re-protection | Process of restoring required redundancy after a failure. |
| Resiliency | Ability to continue operating through component failures. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
