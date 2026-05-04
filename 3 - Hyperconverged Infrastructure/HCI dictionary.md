# Hyperconverged Infrastructure — Acronyms and Keywords Dictionary

Consolidated glossary for the `3 - Hyperconverged Infrastructure` section.

> Scope: Consolidated from the accessible Nutanix repository section. Terms are deduplicated inside this file, alphabetized inside each semantic family, and intentionally allowed to repeat across the three section files.

## Core HCI Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| 3-tier architecture | Traditional infrastructure model separating compute, network, and storage into independent layers. |
| AHV | Acropolis Hypervisor; Nutanix native virtualization hypervisor integrated with AOS and Prism. |
| AOS | Acropolis Operating System; Nutanix software layer providing distributed storage, data services, resiliency, upgrades, and cluster intelligence. |
| Availability domain | Failure boundary such as disk, node, block, rack, or site used to reason about resiliency. |
| Block | Physical Nutanix chassis/enclosure that may contain one or more nodes and can be treated as a failure domain. |
| Block awareness | Replica placement strategy designed to reduce impact from a block-level failure. |
| Cluster | Group of Nutanix nodes operating together as one distributed infrastructure platform. |
| Compute | CPU and memory resources used to run workloads. |
| Compute/storage convergence | Architectural principle of combining compute and storage resources into one distributed platform. |
| Converged infrastructure | Pre-integrated compute, network, and storage stack that may still keep compute and storage logically separate. |
| CVM | Controller Virtual Machine; critical Nutanix VM on each node providing storage and cluster services. |
| Data locality | Nutanix behavior that serves VM I/O near the VM location when possible to reduce latency and network traffic. |
| Distributed system | System composed of multiple cooperating components; central mental model for Nutanix clusters. |
| DSF | Distributed Storage Fabric; Nutanix distributed storage layer aggregating local disks into resilient shared storage. |
| Failure domain | Boundary where a component failure can affect service or data availability, such as disk, node, block, rack, or site. |
| HCI | Hyperconverged Infrastructure; architecture combining compute, storage, virtualization, networking integration, and management in a software-defined cluster. |
| Hybrid node | Node with mixed flash and disk storage, typically SSD/NVMe plus HDD. |
| Node | Physical server in a Nutanix cluster contributing compute and usually storage. |
| Prism | Nutanix management and monitoring interface for health, alerts, capacity, performance, tasks, and operations. |
| Prism Central | Centralized Nutanix management plane for multiple clusters and advanced operational capabilities. |
| Prism Element | Cluster-level Nutanix management interface for a single cluster. |
| Scale-out | Growth model where capacity and/or performance increases by adding nodes. |
| Shared-nothing | Architecture where each node owns local resources while distributed software coordinates the platform. |

## Storage, Data Services, and Resiliency

| Term / Acronym / Keyword | Meaning |
|---|---|
| Array | Centralized storage system providing block or file storage to servers. |
| Backup | Copy of data or workload state kept for recovery purposes. |
| Block storage | Storage model where data is exposed as fixed-size blocks, commonly through SAN technologies. |
| Capacity | Total usable or available resource amount in a system. |
| Capacity headroom | Free resource margin reserved for growth, rebuild, snapshots, recovery, and safe operations. |
| Centralized storage | Storage architecture where persistent data resides in dedicated external arrays rather than inside each compute host. |
| Compression | Storage efficiency technique that reduces the physical size of stored data. |
| Container / Storage Container | Logical Nutanix storage boundary presented to workloads or hypervisors and used for policy application. |
| DAS | Direct-Attached Storage; storage physically attached to a server. |
| Data placement | Process of deciding where data and replicas are stored across the cluster. |
| Data protection | Mechanisms such as snapshots, replication, backups, and redundancy used to prevent data loss. |
| Data resiliency | Ability to preserve availability and data integrity after failures. |
| Deduplication | Storage efficiency technique that avoids storing duplicate data blocks. |
| Degraded cluster | Cluster state where one or more components have failed or resilience is reduced. |
| Disk rebuild | Process of reconstructing data after a disk failure using redundancy mechanisms. |
| DR | Disaster Recovery; process and technology for recovering services after major failure, outage, or site loss. |
| EC-X | Erasure Coding Extension; Nutanix capacity optimization feature that reduces overhead compared with full replication. |
| Erasure coding | Data protection and capacity-efficiency technique using fragments and parity rather than full extra copies. |
| Failover | Moving or restarting workloads/services on a backup component, host, cluster, or site after failure. |
| Fault tolerance | Ability to continue operating after component failure. |
| HDD | Hard Disk Drive; magnetic disk storage often used for capacity tiers in hybrid systems. |
| Hotspot | Localized resource concentration causing performance degradation. |
| I/O | Input/Output; read and write operations between workloads and storage or network systems. |
| IOPS | Input/Output Operations Per Second; metric for storage read/write operation rate. |
| NAS | Network-Attached Storage; file-level storage accessed over a network. |
| NFS | Network File System; file protocol often used for VMware datastores. |
| NVMe | High-performance SSD interface/protocol used for low-latency storage. |
| Object storage | Storage model where data is stored as objects with metadata. |
| Physical disk | Actual storage hardware such as HDD, SSD, or NVMe. |
| QoS | Quality of Service; resource control or prioritization mechanism. |
| RAID | Traditional disk redundancy model; useful contrast with Nutanix distributed resiliency. |
| Raw capacity | Total physical disk capacity before resiliency, metadata, and operational overheads. |
| Rebalance / Rebalancing | Redistribution of data or load across nodes or disks. |
| Rebuild | Reconstruction of data redundancy after disk or node failure. |
| Replication | Maintaining additional copies of data for resilience or disaster recovery. |
| Replication Factor / RF | Number of data copies maintained for protection. |
| RF2 | Replication Factor 2; two copies of data. |
| RF3 | Replication Factor 3; three copies of data for higher resiliency. |
| SAN | Storage Area Network; dedicated storage architecture providing block-level storage. |
| Snapshot | Point-in-time copy of data or VM state. |
| SSD | Solid-State Drive; flash-based storage device. |
| Storage array | Dedicated external storage system used in traditional architectures. |
| Storage controller | Component that manages storage I/O and data services. |
| Storage fabric | Distributed storage layer coordinating storage resources. |
| Storage pool | Logical or physical aggregation of storage resources across a cluster. |
| Thin provisioning | Allocating logical capacity without reserving all physical capacity upfront. |
| Tiering | Placement or movement of data across media types such as NVMe, SSD, and HDD. |
| Usable capacity | Capacity available after metadata, resiliency, rebuild reserve, and other overheads. |
| VDI | Virtual Desktop Infrastructure. |
| vDisk | Virtual disk object consumed by a VM. |
| Volume | Logical block storage unit presented to a host or application. |

## Virtualization and Compute

| Term / Acronym / Keyword | Meaning |
|---|---|
| ADS | Acropolis Dynamic Scheduler; Nutanix service for workload placement and resource usage optimization. |
| APD | All Paths Down; VMware condition where a host temporarily loses all paths to storage. |
| Application tier | Application architecture layer where business logic runs; distinct from infrastructure 3-tier architecture. |
| CPU | Central Processing Unit; physical compute resource. |
| CPU contention | Multiple workloads competing for limited physical CPU resources. |
| CPU Ready | VMware metric showing time a VM is ready to run but waiting for physical CPU resources. |
| ESXi | VMware enterprise bare-metal hypervisor; Nutanix can run with AHV or ESXi in supported deployments. |
| HA | High Availability; design principle keeping services running or restarting workloads after failures. |
| Host | Physical server running a hypervisor and hosting VMs. |
| Host isolation | Condition where a host loses management or cluster connectivity and may trigger HA behavior. |
| Hyper-V | Microsoft hypervisor for running virtual machines. |
| Hypervisor | Software layer that runs and manages virtual machines. |
| KVM | Kernel-based Virtual Machine; Linux virtualization technology underlying AHV concepts. |
| Live migration | Moving a running VM from one host to another with minimal or no downtime. |
| Memory ballooning | Hypervisor technique used to reclaim memory from VMs during memory pressure. |
| Memory pressure | Condition where host or VM RAM is insufficient for demand. |
| NCI | Nutanix Cloud Infrastructure; Nutanix platform including AOS, AHV, DR, and related capabilities. |
| Noisy neighbor | Workload consuming excessive shared resources and affecting others. |
| NUMA | Non-Uniform Memory Access; server memory architecture relevant to large VM sizing and performance. |
| PDL | Permanent Device Loss; VMware condition where a storage device is permanently unavailable. |
| VM | Virtual Machine; software-defined server running on a hypervisor. |
| VMware | Enterprise virtualization and cloud infrastructure platform/vendor. |
| Workload | Application, VM, service, or process running on infrastructure. |

## Networking and Connectivity

| Term / Acronym / Keyword | Meaning |
|---|---|
| Backup traffic | Network traffic generated by backup systems when copying VM, file, database, or application data. |
| CRC errors | Cyclic Redundancy Check errors; link-level errors suggesting corruption or transmission issues. |
| Data plane | Infrastructure path carrying actual workload, storage, or application traffic. |
| DNS | Domain Name System; service resolving names to IP addresses. |
| East-west traffic | Traffic between systems inside a datacenter, cluster, or application environment. |
| Fabric | Network of switches and links, often used for SAN or datacenter switching. |
| FC | Fibre Channel; high-performance storage networking protocol commonly used in SAN environments. |
| Firewall | Security device or rule set controlling network traffic. |
| HBA | Host Bus Adapter; adapter connecting servers to storage networks, commonly Fibre Channel. |
| IP | Internet Protocol; core addressing and routing protocol for network communication. |
| IPMI | Intelligent Platform Management Interface; out-of-band server management interface. |
| iSCSI | Internet Small Computer Systems Interface; protocol carrying SCSI commands over IP networks. |
| Jumbo frames | Ethernet frames larger than standard MTU, often used in storage or high-throughput networks. |
| LACP | Link Aggregation Control Protocol; bundles multiple physical network links into one logical link. |
| LAN | Local Area Network; network connecting systems within a local site or datacenter. |
| Link flap | Network link repeatedly going up and down. |
| Management network | Network used for infrastructure management traffic. |
| Management plane | Control, configuration, monitoring, and administration layer. |
| Migration traffic | Traffic generated during VM live migration or storage migration. |
| MTU | Maximum Transmission Unit; maximum packet size on a network path. |
| Multipathing | Use of multiple storage paths for redundancy and load balancing. |
| NIC | Network Interface Card; adapter connecting a server to a network. |
| North-south traffic | Traffic entering or leaving a datacenter, cluster, or application environment. |
| NTP | Network Time Protocol; clock synchronization protocol. |
| Packet loss | Network condition where packets fail to reach the destination. |
| Production network | Network carrying application or user-facing service traffic. |
| Routing | Forwarding traffic between IP networks. |
| SAN zoning | Fibre Channel access control mechanism defining which initiators can see which targets. |
| Storage network | Specialized network path carrying storage traffic; part of the network tier in 3-tier architecture. |
| VLAN | Virtual Local Area Network; Layer 2 segmentation method. |

## Operations, Support, and Incident Management

| Term / Acronym / Keyword | Meaning |
|---|---|
| Alert | Notification indicating a health, capacity, performance, or resiliency condition. |
| Availability | Ability of a system or service to remain operational and accessible. |
| Blast radius | Scope of impact of an incident across users, services, VMs, hosts, clusters, or sites. |
| BMC | Baseboard Management Controller; out-of-band hardware management interface for health, power, and remote access. |
| Bottleneck | Constrained component limiting overall performance. |
| Business impact | Practical effect of an issue on customer operations, users, revenue, or SLA. |
| CE | Community Edition; free Nutanix edition used for labs and learning. |
| Change window | Approved period for planned upgrades, maintenance, or infrastructure change. |
| CLI | Command-Line Interface; text-based administration and troubleshooting interface. |
| Cloud operations | Operating cloud or cloud-like production infrastructure. |
| Confluence | Documentation and knowledge-management platform. |
| CSAT | Customer Satisfaction; support performance/customer sentiment metric. |
| Degradation | Partial service impairment where a system remains available but performs below expectation. |
| Enterprise support | Support for business-critical customer environments with strong SLAs and escalation expectations. |
| Escalation | Raising an issue to higher support, SRE, engineering, management, TAM, or account team. |
| Field Engineer | Engineer responsible for on-site hardware replacement or repair. |
| Firmware | Low-level software embedded in hardware devices such as NICs, BIOS, disks, or controllers. |
| GCP | Google Cloud Platform; public cloud provider relevant to hybrid cloud discussions. |
| Grafana | Monitoring and dashboarding platform for metrics and alerts. |
| Gray failure | Partial or intermittent failure without a clean up/down signal. |
| IC | Individual Contributor; hands-on technical role without people-management responsibility. |
| Incident management | Process for detecting, triaging, resolving, communicating, and reviewing incidents. |
| Insights | Nutanix operational analytics/telemetry context for alerts and health indicators. |
| ITSM | IT Service Management; processes and tools for incidents, changes, requests, and problems. |
| Jira | Issue and workflow tracking system used for incidents, escalations, bugs, and tasks. |
| K8s / Kubernetes | Container orchestration platform relevant to distributed systems and workload operations. |
| Kibana | Log analysis and visualization tool. |
| KPI | Key Performance Indicator; metric used to track system, service, or team performance. |
| Latency | Time taken for an operation to complete. |
| LCM | Life Cycle Manager; Nutanix software/firmware lifecycle and update management. |
| Linux | Operating system widely used in infrastructure and cloud environments. |
| Monitoring | Collection and analysis of metrics, logs, events, and alerts. |
| MTTR | Mean Time To Resolution/Restore/Repair; time required to recover service or resolve an incident. |
| NCC | Nutanix Cluster Check; health-check utility for Nutanix clusters. |
| Observability | Ability to understand system state through metrics, logs, traces, alerts, and events. |
| P1 | Priority 1 incident; usually critical production impact. |
| RCA | Root Cause Analysis; formal explanation of what happened, why, impact, and prevention. |
| RFO | Reason for Outage; explanation of outage cause. |
| Runbook | Documented operational procedure for troubleshooting or recovery. |
| SaaS | Software as a Service. |
| Sev1 | Severity 1; critical incident with major production or business impact. |
| SLA | Service Level Agreement; contractual or operational service commitment. |
| SRE | Site Reliability Engineering/Engineer; reliability, automation, performance, and operations role/discipline. |
| Support bundle | Diagnostic package collected for troubleshooting. |
| TAM | Technical Account Manager; customer-facing technical relationship owner. |
| Throughput | Amount of data transferred over time, often MB/s or GB/s. |
| Triage | Structured initial assessment of impact, scope, urgency, symptoms, and next action. |
| Worldwide Support | Global support organization operating across regions and time zones. |
| X-Play | Nutanix Prism Central automation capability based on triggers and actions. |
