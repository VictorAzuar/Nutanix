# AOS and Storage — Acronyms and Keywords Dictionary

Consolidated glossary for the `AOS and Storage` section.

> Scope: Consolidated from the accessible Nutanix repository section. Terms are deduplicated inside this file, alphabetized inside each semantic family, and intentionally allowed to repeat across the three section files.

## Core AOS Storage Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| AHV | Acropolis Hypervisor; Nutanix native hypervisor. |
| AOS | Acropolis Operating System; Nutanix distributed software layer providing storage, platform services, resiliency, management, and operations. |
| AOS Storage | Nutanix software-defined distributed storage fabric used by workloads running on the cluster. |
| Block | Nutanix chassis/enclosure or hardware grouping that may function as a failure domain. |
| Cluster | Group of Nutanix nodes operating as one distributed system. |
| Container | Logical storage boundary inside a Nutanix storage pool. |
| Controller VM | VM running Nutanix storage/control services on each node. |
| CVM | Controller Virtual Machine; Nutanix VM on each node providing storage and cluster services. |
| Distributed Storage Fabric | Nutanix storage architecture pooling local disks across nodes into unified resilient storage. |
| DSF | Distributed Storage Fabric; Nutanix scale-out storage architecture. |
| Extent | Logical unit/chunk of data used internally by AOS storage. |
| Extent Group | Physical/logical grouping of extents used for placement and replication. |
| Extent Store | Persistent bulk storage layer in the Nutanix storage architecture. |
| Genesis | Nutanix service framework that starts and monitors services. |
| HCI | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in nodes. |
| Host | Physical hypervisor server in the Nutanix cluster. |
| Hypervisor | Software layer that runs VMs, such as AHV, ESXi, or Hyper-V. |
| Metadata | Data about data: mappings, placement, ownership, replicas, references, and state. |
| Metadata disk | Disk used by the CVM for metadata; relevant to supportability in some features. |
| Metadata service | Service involved in storing, accessing, coordinating, or scanning metadata. |
| Node | Physical Nutanix server contributing compute, storage, and services. |
| NutanixManagementShare | Built-in Nutanix container used by platform features; should not be casually deleted. |
| Prism | Nutanix management interface for monitoring, administration, alerts, and operations. |
| Prism Central | Centralized Nutanix management plane for multiple clusters and broader operations. |
| Prism Element | Cluster-level Nutanix management interface. |
| Stargate | Nutanix/AOS service responsible for storage I/O handling and data path operations. |
| Storage Container | Logical storage object/policy boundary where VMs, files, or vDisks are placed. |
| Storage controller | Component responsible for managing storage access and data services; in Nutanix this role is performed by CVMs. |
| Storage Policy | Policy-based mechanism for applying storage attributes at supported logical levels. |
| Storage Pool | Aggregated physical storage devices across Nutanix cluster nodes. |
| Unified Cache | Nutanix caching layer used in the storage architecture. |
| vBlock | Logical chunk of vDisk address space. |
| vDisk | Virtual disk object used by a VM and managed by AOS storage. |
| Volume Group | Nutanix block storage construct often used for iSCSI/direct block use cases. |

## Resiliency, Replication, and Failure Domains

| Term / Acronym / Keyword | Meaning |
|---|---|
| Availability | Ability of the platform or workload to remain accessible during failures. |
| Availability Domain | Failure boundary such as disk, node, block, rack, or site. |
| Block Awareness | Replica placement strategy to tolerate block-level failures. |
| Capacity Headroom | Free usable capacity needed for growth, rebuild, snapshots, and resiliency. |
| Cassandra | Distributed metadata store technology used by Nutanix services. |
| Checksum | Data-integrity mechanism used to detect corruption. |
| Cluster Resiliency | Ability of the cluster to tolerate failures while preserving service and data protection. |
| Data protection | Mechanisms such as snapshots, replication, redundancy, and backups to protect workloads. |
| Data Resiliency | Ability to maintain data availability and redundancy after failures. |
| Data Resiliency Status | Prism/cluster indicator showing whether the cluster can tolerate failures safely. |
| Degraded Resiliency | State where expected protection level is reduced. |
| Degraded State | Condition where the cluster runs with reduced redundancy, resilience, or performance. |
| Disk Failure | Physical or logical failure/degradation of a storage device. |
| Fault Domain | Component or boundary that can fail, such as disk, node, block, rack, or site. |
| Fault Tolerance | Ability to survive a configured number of failures without losing service or data availability. |
| FT | Fault Tolerance. |
| Medusa | Metadata access layer/interface in front of Cassandra. |
| Node Failure | Loss or unavailability of a Nutanix node, host, CVM, or critical node component. |
| Paxos | Consensus algorithm used for consistency in distributed systems. |
| Quorum | Minimum number of members required for safe distributed-system decisions. |
| Re-protection / Reprotection | Restoring required redundancy after a failure or under-replication. |
| Rebalancing | Redistributing data or workload across healthy resources. |
| Rebuild | Process of recreating missing redundant data after disk or node failure. |
| Rebuild Capacity | Capacity needed to restore redundancy after failed disk, node, block, or rack. |
| Rebuild Capacity Reservation | Reserved capacity feature for recovery/self-healing after failures. |
| Recovery Milestone | Measurable step in incident/rebuild recovery progress. |
| Redundancy factor | Level of data replica protection in a Nutanix cluster. |
| Replica | Protected copy of data stored on another disk/node. |
| Replication | Copying data to another node, cluster, or site for resilience or disaster recovery. |
| Replication Factor / RF | Number of data copies maintained for protection/availability. |
| RF | Replication Factor; number of copies maintained for availability. |
| RF2 | Replication Factor 2; two copies of data. |
| RF3 | Replication Factor 3; three copies of data for higher resiliency. |
| Risk Window | Period where the cluster is online but exposed to higher risk due to degraded resiliency. |
| Under-replicated | Data currently has fewer replicas than required. |
| Zeus | Nutanix library/interface for accessing Zookeeper-backed cluster configuration. |
| Zookeeper | Distributed coordination/configuration service used in Nutanix architecture. |

## Capacity, Data Efficiency, and Storage Economics

| Term / Acronym / Keyword | Meaning |
|---|---|
| Advertised Capacity | Logical capacity presented for a storage container. |
| Capacity deduplication | Deduplication aimed at reducing physical storage capacity usage. |
| Capacity Efficiency | Techniques such as compression, dedupe, and erasure coding to reduce physical storage usage. |
| Capacity optimization | Reducing physical storage usage through efficiency techniques. |
| Capacity Optimization Engine / COE | AOS mechanism for improving storage efficiency with data reduction techniques. |
| Capacity Reservation | Space reserved for specific operational or resiliency purposes. |
| Cold Tier | Lower-performance storage tier, often HDD-based in hybrid systems. |
| Compression | Data reduction technique that stores data in a smaller physical footprint. |
| Compression delay | Time between data write and post-process compression. |
| Data Efficiency | Techniques such as compression, deduplication, cloning, thin provisioning, and erasure coding. |
| Data reduction | General category including compression, deduplication, and erasure coding. |
| Deduplication | Storage efficiency technique that stores one copy of duplicate data blocks. |
| Dense node | Nutanix node with high storage capacity; relevant to some dedupe support considerations. |
| EC-X | Nutanix erasure coding feature for storage capacity efficiency. |
| Encrypted data | Data transformed for security; often compresses or deduplicates poorly. |
| Encryption | Protection of data at rest or in transit; affects efficiency behavior. |
| Erasure Coding | Capacity-efficient resiliency method using parity instead of full extra copies. |
| Fingerprint-on-write | Mechanism related to identifying duplicate data during write operations. |
| Fingerprinting | Creating identifiers for data blocks to detect duplicates. |
| Foreground I/O | Active application read/write traffic. |
| Hot Tier | High-performance storage tier, usually SSD/NVMe-based. |
| ILM | Intelligent/Information Lifecycle Management; data movement across tiers based on usage/access patterns. |
| Inline | Processing performed during the write path. |
| Inline compression | Compression performed during data ingestion/write path. |
| Logical Capacity | Capacity as seen by workloads before physical overhead or efficiency effects. |
| Logical usage | Capacity as seen before physical efficiency savings. |
| Physical Capacity | Actual disk capacity consumed at the infrastructure layer. |
| Physical usage | Actual storage consumed after efficiency techniques. |
| Post-process compression | Compression performed after data is written, usually by background jobs. |
| Raw Capacity | Total physical disk capacity before overheads. |
| Reserved Capacity | Capacity explicitly reserved for a storage container. |
| Resilient Capacity | Capacity level accounting for ability to tolerate/recover from failures. |
| Storage efficiency | Reduction of physical capacity usage through clones, compression, deduplication, erasure coding, or thin provisioning. |
| Thin Provisioning | Allocating logical capacity without immediately consuming all physical capacity. |
| Usable Capacity | Capacity available after overheads such as metadata, resiliency, and rebuild reserve. |
| Workload suitability | Assessment of whether a workload is appropriate for a feature such as dedupe/compression. |

## Data Path, Locality, and Performance

| Term / Acronym / Keyword | Meaning |
|---|---|
| Background job | Non-foreground task such as post-process compression, scan, cleanup, data optimization, or rebuild. |
| Cache | High-speed memory or flash used to accelerate read/write operations. |
| Curator | Nutanix background service/framework for scans, cleanup, data placement, rebalancing, optimization, and re-protection workflows. |
| Data Drive | Disk used to store user/workload data in the cluster. |
| Data Locality | Nutanix behavior that tries to serve VM I/O from the local node/CVM when possible. |
| Data placement | Process of deciding where blocks, extents, replicas, and related metadata live. |
| Disk Slot | Physical location of a disk in a node/chassis. |
| Foreground I/O | Active application read/write traffic. |
| Garbage collection | Process that reclaims unused storage and supports storage efficiency. |
| Hardware Alert | Prism or system alert indicating physical component issue. |
| HDD | Hard Disk Drive; spinning disk, often used for capacity/cold tier. |
| I/O | Input/output; read and write operations from workloads to storage. |
| IOPS | Input/Output Operations Per Second; storage performance metric. |
| Latency | Time taken to complete an I/O operation or application request. |
| Local I/O | I/O served through the CVM on the same node as the VM. |
| NVMe | High-performance SSD interface/protocol. |
| OpLog / Oplog | Nutanix write buffer/log/staging component used in the storage I/O path, especially for random writes. |
| Performance Degradation | System remains available but performs worse than expected. |
| Physical Disk | Actual storage hardware such as HDD, SSD, or NVMe. |
| QoS | Quality of Service; controls or prioritizes resource usage. |
| Queue depth | Number of outstanding I/O operations a host, adapter, device, or VM can queue. |
| RDMA | Remote Direct Memory Access; advanced high-performance network/storage data path concept. |
| Remote I/O | I/O served through another CVM over the network when local path is unavailable or unsuitable. |
| SPDK | Storage Performance Development Kit; user-space storage framework. |
| SSD | Solid-State Drive; flash-based storage device. |
| Storage latency | Delay in completing storage reads/writes. |
| Throughput | Amount of data transferred per second, usually MB/s or GB/s. |
| Tiering | Placement or movement of data across media types such as NVMe, SSD, and HDD. |
| Write Latency | Time required to complete a write operation. |

## Snapshots, Clones, Backup, and Recovery

| Term / Acronym / Keyword | Meaning |
|---|---|
| Application consistency | Snapshot/backup state where application writes are coordinated to improve recoverability. |
| Backup | Copy of data or workload state kept for recovery purposes. |
| Clone | Copy of a VM or disk created from an existing source. |
| Clone sharing | Capacity-efficient sharing of unchanged blocks between clone and source. |
| Consistency | Degree to which recovered data/application state is usable and coherent. |
| CSI | Container Storage Interface; standard for Kubernetes storage plugins. |
| Disaster Recovery / DR | Recovery strategy after major failure, site loss, or platform outage. |
| EBS | Elastic Block Store; AWS block storage service. |
| Full copy | Complete duplication of all data blocks. |
| Golden image | Standard VM image used to create clones, templates, or many similar VMs. |
| Guest customization | Post-clone changes to OS identity, hostname, network, or application settings. |
| Guest OS | Operating system running inside a VM. |
| Machine ID | Unique OS or application identity that may need regeneration after cloning. |
| Nutanix Volumes | Nutanix block storage service. |
| Protected VM | VM covered by Nutanix data protection or replication configuration. |
| Protection Domain | Nutanix data protection construct for snapshots and replication. |
| PVC | Persistent Volume Claim; Kubernetes request for persistent storage. |
| Quiescing | Pausing/flushing application or OS activity to create a consistent state. |
| Redirect-on-write | Technique where new writes go to new blocks rather than overwriting existing ones. |
| Restore | Recovery of data or workload from backup, snapshot, or clone. |
| RPO | Recovery Point Objective; acceptable amount of data loss measured in time. |
| RTO | Recovery Time Objective; acceptable time to restore service. |
| SID | Security Identifier; Windows identity value that may duplicate after cloning. |
| Snapshot | Point-in-time state/copy of data or VM disk state used for recovery, cloning, or backup workflows. |
| Snapshot retention | Policy defining how long snapshots are kept. |
| SSR | Self-Service Restore; file-level recovery from snapshots/recovery points. |
| Template | Reusable VM base configuration for deployment. |
| Test/dev | Non-production environment used for testing and development. |
| VDI | Virtual Desktop Infrastructure; often a good dedupe/clone candidate. |
| VM | Virtual Machine. |
| VSS | Volume Shadow Copy Service; Microsoft framework for consistent snapshots. |
| Write divergence | Growth of unique data after clones start writing changes. |

## Protocols, Hypervisors, and Integrations

| Term / Acronym / Keyword | Meaning |
|---|---|
| aCLI | AHV command-line interface used for Nutanix virtualization operations. |
| Datastore | Hypervisor-visible storage location, often mapped to a Nutanix container in VMware. |
| ESXi | VMware hypervisor commonly supported in Nutanix environments. |
| External CVM IP | CVM management-facing IP address. |
| Foundation | Nutanix deployment/provisioning tool that configures nodes and CVMs during installation. |
| GCP | Google Cloud Platform. |
| Grafana | Monitoring/visualization platform relevant for metrics and trend analysis. |
| Hyper-V | Microsoft hypervisor supported in some Nutanix environments. |
| iSCSI | Internet Small Computer Systems Interface; block storage protocol. |
| K8s / Kubernetes | Container orchestration platform. |
| Kibana | Log search and visualization tool. |
| LACP | Link Aggregation Control Protocol; network bonding protocol relevant in node/network issues. |
| LCM | Life Cycle Manager; Nutanix lifecycle/firmware/software update tool. |
| nCLI | Nutanix command-line interface for cluster and AOS management. |
| NFS | Network File System; protocol often used for VMware datastore access. |
| PCI passthrough | Direct assignment of PCI devices such as storage controllers to a VM. |
| SAN | Storage Area Network; traditional external block storage architecture. |
| SMBv3 | Server Message Block version 3; file storage protocol. |
| UVM | User VM; customer workload virtual machine. |
| VMware | Enterprise virtualization platform often integrated with Nutanix. |
| vSphere | VMware virtualization suite including ESXi and vCenter. |

## Operations, Support, and Escalation

| Term / Acronym / Keyword | Meaning |
|---|---|
| Alert | Prism or system warning indicating health, capacity, resiliency, or performance issue. |
| Boot Drive | Disk/device used for boot-related components, depending on platform. |
| Business Impact | Effect of an issue on customer operations, users, revenue, or SLA. |
| Engineering escalation | Escalation to product engineering for suspected defect or deep product issue. |
| Enterprise support | Support function for business-critical customer environments. |
| Escalation | Formal process for raising a customer issue to higher technical or management levels. |
| ETA | Estimated Time of Arrival/completion; used for rebuild completion estimates and updates. |
| Field Engineer | Engineer responsible for onsite hardware replacement or repair. |
| Firmware | Low-level software controlling hardware components such as disks, NICs, BIOS, or controllers. |
| Insights DB | Nutanix database used by Prism to surface operational and rebuild information. |
| Jira | Ticket/workflow or engineering-tracking tool used for cases, bugs, and action tracking. |
| KPI | Key Performance Indicator; metric used to track operational performance. |
| Maintenance Mode | Controlled state used to evacuate or prepare a host/node for planned work. |
| Monitoring | Collection and analysis of metrics, logs, events, and alerts. |
| MTTR | Mean Time To Repair/Restore/Resolution; operational recovery metric. |
| NCC | Nutanix Cluster Check; health-check framework/tooling for Nutanix clusters. |
| Observability | Ability to understand system state through metrics, logs, traces, and events. |
| P1 / P2 | Priority levels for severe support incidents. |
| P1 / Sev1 | Highest-priority incident, usually major production impact. |
| RCA | Root Cause Analysis; post-incident explanation of what happened, why, and prevention. |
| RFO | Reason for Outage; explanation of outage cause. |
| Runbook | Documented operational procedure for troubleshooting or recovery. |
| SLA | Service Level Agreement; contractual or operational target. |
| SRE | Site Reliability Engineer/Engineering; reliability-focused operations/engineering discipline. |
| Support Bundle | Diagnostic package collected for troubleshooting. |
| TAM | Technical Account Manager; customer-facing technical relationship/escalation role. |
| Triage | Initial structured investigation to classify impact, priority, symptoms, and next actions. |
| Worldwide Support | Global support organization handling enterprise customer issues across regions. |
