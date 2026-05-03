# 10 - NUMA Basics

## 1. Purpose of this chapter

This chapter explains NUMA only as far as needed for VM sizing and performance escalation leadership. It highlights when NUMA matters and how poor alignment can look like a generic performance problem.

## 2. Core definition

NUMA, or Non-Uniform Memory Access, is a server architecture where CPU sockets or groups have faster access to their local memory than to memory attached to another socket or NUMA node.

## 3. Main explanation

Virtualization abstracts hardware, but it does not eliminate memory locality. A small VM may fit comfortably inside one physical NUMA node. A large VM may span multiple NUMA nodes. If vCPU and memory placement is poorly aligned, the guest may experience higher latency and inconsistent performance.

In AHV contexts, NUMA becomes relevant for large databases, analytics workloads, memory-heavy applications, latency-sensitive workloads, and VMs resized beyond a host’s local NUMA boundaries. vNUMA can expose NUMA topology to the guest so NUMA-aware operating systems and applications can schedule more intelligently.

A support manager does not need to tune scheduler internals. The required skill is knowing when to suspect NUMA: performance changes after migration, poor performance after resize, large VM slower than smaller VMs, or workload performing well on one host but not another.

## 4. Key concepts

- Local memory access is faster than remote memory access.
- Large or wide VMs are more NUMA-sensitive.
- vNUMA helps the guest understand virtual NUMA topology.
- Oversizing can push a VM across NUMA boundaries.
- NUMA symptoms often look like application or CPU performance issues.

## 5. Support relevance

NUMA is important in enterprise support because many high-value workloads are large and latency-sensitive. Customers may not use the term NUMA; they may say “the database is slower after migration” or “the VM got worse after adding CPUs.” The support manager should make sure performance SMEs validate sizing, placement, host topology, and workload requirements.

## 6. Escalation scenario

A customer increases a SQL Server VM from 16 to 48 vCPUs and reports worse transaction latency. The team initially considers storage, but Prism shows no storage spike. The support manager asks for host CPU topology, VM sockets/cores configuration, memory size, previous sizing, guest SQL configuration, and whether the VM now spans NUMA nodes. The remediation is to right-size the VM and align vCPU/memory topology instead of adding more resources.

## 7. Triage questions

- How large is the VM relative to the host NUMA node size?
- Did performance change after resize or migration?
- Is the workload NUMA-aware, such as a database or analytics engine?
- Does the issue occur only on certain hosts?
- What are the VM socket/core and memory settings?
- Are guest OS and application metrics showing latency despite available CPU?
- Should performance specialists review vNUMA and sizing?

## 8. What to avoid

- Weak: “More vCPU always helps large workloads.” Better: “Large VMs must be sized with NUMA topology and application behavior in mind.”
- Weak: “NUMA is too low-level for support.” Better: “Support leaders need to know when NUMA may be the escalation domain.”
- Weak: “The host has enough total memory.” Better: “Total memory is not the same as local memory access pattern.”

## 9. Minimum to memorize

- NUMA affects memory latency based on CPU locality.
- Large VMs can span NUMA nodes and become sensitive to placement.
- vNUMA exposes topology to the guest.
- Performance regressions after resize or migration should trigger NUMA review.
- Right-sizing may outperform adding resources.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Local memory | Memory directly attached to a CPU socket or NUMA node. |
| NUMA | Non-Uniform Memory Access. |
| Remote memory | Memory attached to another NUMA node and accessed with higher latency. |
| Right-sizing | Sizing a VM according to workload need and hardware topology. |
| vNUMA | Virtual NUMA topology exposed to a VM. |
| Wide VM | VM that needs resources across multiple NUMA nodes. |
