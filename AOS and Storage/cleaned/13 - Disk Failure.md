# 13 - Disk Failure

## 1. Purpose of this chapter
This chapter explains disk failure as a distributed storage resiliency event, not merely a hardware replacement task. Support managers need this because disk cases involve data protection, rebuild progress, customer anxiety, hardware logistics, and safe-change control.

## 2. Core definition
A **disk failure** is the loss or degradation of a physical storage device in a Nutanix node. AOS should continue serving protected data from surviving replicas when the cluster is healthy, then rebuild missing redundancy onto healthy resources.

## 3. Main explanation
In Nutanix, a failed disk affects more than the physical server that contains it. Because storage is distributed, some data replicas may have lived on that device. AOS detects the failure, marks affected data as needing re-protection, and begins restoring redundancy elsewhere.

A single disk failure should not normally mean data loss in a healthy, correctly protected cluster. The support risk is the window after the failure: the cluster may be degraded, rebuild may consume resources, capacity may be tight, and a second failure could increase exposure.

The manager’s role is to ensure the failed component is correctly identified, the cluster is not pushed into additional risk, the customer understands current protection state, and hardware replacement follows the validated workflow.

## 4. Key concepts
- **Failed disk**: drive has failed or is reporting critical errors.
- **Predictive failure**: disk may still be online but is at high risk.
- **Rebuild / re-protection**: data copies are restored elsewhere.
- **Hot-swap workflow**: replacement process must follow platform guidance.
- **Capacity headroom**: needed to rebuild missing copies.
- **Secondary risk**: additional disk/node problems during degraded state.

## 5. Support relevance
Disk failures are common but still escalation-prone. Customers may ask if data is safe, whether they can continue production, whether latency is expected, and when hardware will be replaced. Support must avoid both panic and over-reassurance.

## 6. Escalation scenario
A disk fails in an RF2 cluster during month-end processing. Prism reports degraded resiliency. Rebuild starts, but the customer wants to remove and reseat the disk immediately to “speed things up.”

The manager should stop unsafe improvisation. Confirm the failed disk identity, cluster protection state, rebuild progress, capacity headroom, and hardware replacement guidance. Communicate that removing or changing additional components during re-protection can increase risk unless validated by Support.

## 7. Triage questions
1. Which disk failed: node, slot, serial number, tier, and model?
2. Is the disk failed, missing, degraded, or predictive-failure?
3. What RF and failure-domain design protect the affected data?
4. Is the cluster fully protected, degraded, or rebuilding?
5. Is rebuild progressing and is capacity sufficient?
6. Are there other disk, node, CVM, or network alerts?
7. Are workloads seeing latency or failed I/O?
8. Has the customer removed, reseated, or replaced anything already?
9. What is the validated replacement and communication plan?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “It is only a disk, not a serious issue.” | “A disk failure can be routine, but protection state and rebuild risk must be validated.” |
| “Swap the disk immediately.” | “Follow the supported replacement sequence and confirm re-protection state first.” |
| “There is no risk because RF2 is enabled.” | “RF2 helps tolerate one relevant failure, but risk increases while degraded.” |
| “Latency is unrelated to the disk failure.” | “Rebuild, remote access, or disk-tier pressure can affect latency; validate with metrics.” |

## 9. Minimum to memorize
A disk failure is a distributed resiliency event. AOS serves data from replicas and rebuilds missing protection, but support must validate current protection, capacity, rebuild progress, secondary alerts, and safe replacement workflow.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Capacity headroom | Free capacity required for safe rebuild and operations. |
| Degraded resiliency | State where configured protection is temporarily reduced. |
| Disk failure | Loss or critical fault of a physical storage device. |
| Hot-swap | Replacing hardware while the system remains online, if supported. |
| Predictive failure | Warning state suggesting a disk may fail soon. |
| Rebuild | Recreating missing data copies after failure. |
| Re-protection | Restoring expected redundancy after failure. |
| RF2 | Two-copy protection model. |
| RF3 | Three-copy protection model. |
| Serial number | Hardware identifier used to avoid replacing the wrong disk. |
| Slot | Physical disk position in the node chassis. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
