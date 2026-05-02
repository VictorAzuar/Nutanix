# 9 - Resiliency

## Purpose of this chapter

This chapter explains resiliency in Nutanix HCI: how the platform is designed to continue operating through failures, and how a support manager should reason about risk, recovery, and customer communication. It builds on the HCI component model from chapter 2, scale-out from chapter 3, software-defined storage from chapter 4, and data locality from chapter 8.

## Core definition

**Resiliency** is the ability of a system to keep data and services available despite component failures, degradation, maintenance events, or unexpected disruptions.

In Nutanix HCI, resiliency depends on distributed software behavior: replication factor, failure-domain placement, CVM and node health, capacity headroom, rebuild/re-replication, networking, and operational discipline.

Interview-ready version:

> Nutanix resiliency is based on distributed data protection and cluster-level fault tolerance. Data is protected through replication and placement across failure domains, while the cluster can rebuild or re-replicate after failures. In support, the key is to validate actual cluster state: RF compliance, failure-domain tolerance, capacity headroom, current alerts, and whether recovery activity is affecting performance.

## Resiliency is not only redundancy

Redundancy means having extra components or data copies. Resiliency is broader. It includes:

- Detecting failure.
- Continuing service where possible.
- Preserving data availability.
- Rebuilding protection.
- Avoiding cascading impact.
- Communicating risk accurately.
- Preventing recurrence after the incident.

A support manager should not simply say “the system is redundant.” The better question is:

> What failure has occurred, what protection level remains, and what risk exists if another failure happens before recovery completes?

## Core concepts

### Replication factor

RF2 means two data copies. RF3 means three data copies. This was introduced in chapter 2. In a live escalation, the practical question is whether the cluster is currently compliant with its configured protection level.

### Failure domains

A failure domain is the boundary within which a failure can occur: disk, node, block, rack, site, or network segment. Resilient design places redundant data so that a single failure domain does not remove all required copies.

### Rebuild and re-replication

After a disk or node failure, the cluster may need to rebuild or re-replicate data to restore the desired protection level. During this period, performance and risk can change.

### Resilient capacity

The cluster needs enough safe capacity to recover from failures. Raw free space is not enough. A cluster that is too full may be unable to restore protection after a failure, increasing operational risk.

### Network-dependent recovery

Because storage is distributed, recovery workflows depend on reliable inter-node networking. Packet loss, congestion, MTU mismatch, or switch instability can slow or disrupt recovery.

## Escalation scenario

Scenario: a node fails in a production cluster. The customer asks: “Are we safe? Can we keep running production?”

A weak answer:

> Nutanix is resilient, so you should be fine.

A strong answer:

> The platform is designed to tolerate failures, but we need to validate the current state before making a risk statement. We are checking whether the cluster remains RF-compliant, whether rebuild or re-replication is running, whether there is sufficient resilient capacity, whether any additional alerts exist, and whether workloads are seeing performance impact. We will separate data availability, performance impact, and residual risk in our updates.

## How to assess risk

Use this structure:

| Area | Question |
|---|---|
| Failure | What failed: disk, node, CVM, NIC, block, rack, site, or software service? |
| Scope | One workload, one node, one cluster, multiple clusters, or site-wide? |
| Protection | Is the cluster RF-compliant and failure-domain compliant? |
| Recovery | Is rebuild or re-replication active? Is it progressing? |
| Capacity | Is there enough resilient capacity to recover? |
| Performance | Is recovery activity causing latency or throughput impact? |
| Network | Is inter-node communication healthy? |
| Change | Did a recent upgrade, expansion, maintenance, or migration contribute? |
| Communication | What has been promised to the customer, and what can be stated safely? |

## Manager-level priorities

During a resiliency escalation, the manager should drive:

1. **Impact clarity**: outage, degradation, risk-only, or planned change blocked.
2. **Risk clarity**: data availability, performance impact, and remaining fault tolerance.
3. **Technical ownership**: AOS/storage, network, hypervisor, hardware, customer operations.
4. **Safe mitigation**: avoid risky manual actions without documented guidance or engineering approval.
5. **Customer communication**: precise updates with facts, unknowns, next action, and risk level.
6. **Post-incident prevention**: RCA, capacity planning, health checks, runbook improvement, and customer guidance.

## Relationship to previous chapters

- Chapter 2 explains RF, CVM, AOS, DSF, and Prism.
- Chapter 3 explains why more nodes change scale and failure-domain behavior.
- Chapter 4 explains software-defined storage and why recovery is software-driven.
- Chapter 5 explains why networking affects resiliency workflows.
- Chapter 7 explains scaling risk when adding nodes.
- Chapter 8 explains how data locality and failure recovery can influence performance.

## What to avoid

Avoid saying:

> The cluster is resilient, so there is no risk.

Better:

> The cluster is designed for resiliency, but we need to validate current RF compliance, failure-domain state, capacity headroom, active recovery operations, and remaining tolerance before we characterize risk.

Avoid saying:

> Performance degradation during rebuild means the platform failed.

Better:

> Recovery activity can consume resources. We need to determine whether the impact is expected under current conditions, caused by capacity/network/workload pressure, or evidence of a fault requiring escalation.

## Triage questions

1. What failed or degraded?
2. Is there data unavailability, VM downtime, performance degradation, or risk-only impact?
3. Is the cluster RF-compliant?
4. What failure domain is affected: disk, node, block, rack, site, or network?
5. Is rebuild or re-replication running?
6. Is recovery progressing normally?
7. Is there enough resilient capacity?
8. Are CVMs and cluster services healthy?
9. Are there network errors affecting recovery or cluster communication?
10. Are workloads still running? Are they degraded?
11. What additional failure would create unacceptable risk?
12. What is the customer update cadence and executive communication path?

## Minimum to memorize

> Resiliency is not just having extra copies. In Nutanix, it means the distributed cluster can tolerate failures, preserve data availability, and rebuild protection when conditions allow. During support escalations, the manager must validate actual state: RF compliance, failure domains, CVM/node health, resilient capacity, rebuild progress, network health, and customer impact. Never make blanket safety statements without evidence.
