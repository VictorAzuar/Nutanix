# 13 - Host Maintenance Mode

## 1. Purpose of this chapter

This chapter explains maintenance mode as a controlled host-drain workflow. It emphasizes pre-checks, workload evacuation, blockers, and safe decision-making during maintenance windows.

## 2. Core definition

Host maintenance mode is the AHV operational state used to prepare a host for maintenance by preventing new VM placement and evacuating or stopping workloads according to their configuration and mobility constraints.

## 3. Main explanation

Maintenance mode is not just a button. It is a risk-managed workflow. The host is marked unschedulable, eligible VMs are migrated away, special workloads may block evacuation, and the team validates that cluster health and resiliency remain acceptable before proceeding with hardware work, firmware updates, troubleshooting, reboot, or replacement.

Common blockers include insufficient capacity on other hosts, VM-host affinity, passthrough devices, pinned VMs, RF1 or non-resilient configurations, unhealthy CVMs, active storage rebuilds, network problems, Prism task failures, or maintenance conflicts with customer workloads.

A support manager should treat maintenance mode failures as change-risk incidents. The key decisions are whether to pause, retry, manually migrate safe workloads, shut down approved VMs, reschedule, or escalate to SRE/engineering.

## 4. Key concepts

- Maintenance mode drains a host before intervention.
- Unschedulable means no new workload placement should occur there.
- Evacuation depends on capacity, compatibility, VM mobility, and policies.
- Maintenance mode does not necessarily power off the host.
- Failed entry into maintenance mode is a risk signal, not merely an inconvenience.

## 5. Support relevance

Maintenance mode cases often occur under time pressure: field engineer onsite, change window running, firmware update scheduled, or hardware replacement waiting. The support manager must protect customer uptime while keeping the operation moving.

## 6. Escalation scenario

A customer tries to place a host into maintenance mode before replacing a NIC. The workflow stalls with two VMs still on the host. The escalation lead identifies one VM with host affinity and another with GPU passthrough. The customer confirms a maintenance exception for the GPU VM but not the affinity VM. The team pauses the hardware replacement, updates the plan, and avoids an unapproved shutdown of a production workload.

## 7. Triage questions

- Why is the host entering maintenance mode: upgrade, hardware, troubleshooting, or replacement?
- Is cluster health green enough to proceed?
- Is there enough capacity on remaining hosts?
- Which VMs failed to evacuate and why?
- Are affinity, passthrough, pinned, RF1, or special workload constraints present?
- Are CVMs and storage resiliency healthy?
- What is the approved rollback or pause decision if evacuation fails?

## 8. What to avoid

- Weak: “Just force maintenance mode.” Better: “Validate customer impact, workload constraints, resiliency, and rollback before forcing any action.”
- Weak: “Maintenance mode equals powered off.” Better: “Maintenance mode prepares the host; shutdown or reboot is a separate step.”
- Weak: “If one VM blocks, the platform failed.” Better: “A blocker may be a valid safety constraint that prevents data loss or downtime.”

## 9. Minimum to memorize

- Maintenance mode is a controlled host drain.
- It depends on health, capacity, policies, and VM mobility.
- Some workloads cannot live migrate automatically.
- Failed maintenance mode requires risk-based decision-making.
- Do not force actions without customer approval and evidence.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Evacuation | Moving workloads off a host. |
| Host affinity | Policy tying a VM to specific hosts. |
| Maintenance mode | State used to prepare a host for intervention. |
| Passthrough | Direct hardware assignment to a VM. |
| Pinned VM | VM constrained to a host or device. |
| RF1 | Single-copy storage configuration with reduced resiliency. |
| Unschedulable | Host state preventing new workload placement. |
