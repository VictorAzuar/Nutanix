# [AOS / Storage] — Rebuild / Resync

## 1. Short definition

**Rebuild / resync** in Nutanix AOS is the process by which the cluster restores data resiliency after a failure, maintenance event, or topology change by recreating missing or degraded data copies across healthy disks, nodes, blocks, or racks.

In practical interview language:

> Rebuild is how Nutanix self-heals after losing a disk, node, or failure domain. Resync is the broader process of bringing data placement and redundancy back to the expected state after an interruption, failure, maintenance activity, or configuration change.

Nutanix documentation describes AOS as managing storage resources to preserve data and system integrity during node, disk, application, or hypervisor failures, and Prism exposes cluster resiliency and rebuild progress information through the UI. ([Nutanix Portal][1])

---

## 2. Clear explanation

In a Nutanix cluster, storage is not managed like a traditional RAID group inside a single storage array. AOS uses a **distributed storage fabric** across the nodes in the cluster. Each node contributes storage, and the Controller VMs work together to present storage to workloads.

When data is written, Nutanix keeps multiple copies according to the configured **replication factor** or resiliency policy. In a common **RF2** design, the cluster keeps two copies of data, so it can tolerate one relevant failure. In **RF3**, it keeps more redundancy and can tolerate more failures, assuming the cluster meets the required design constraints.

When something fails, for example a disk or node, AOS detects that some data has lost one of its expected redundant copies. The cluster then starts rebuilding that missing redundancy somewhere else in the cluster, using the remaining healthy copy or copies as the source.

The important idea is this:

> Nutanix rebuild is distributed. The cluster does not rebuild a whole RAID set from one parity group. It rebuilds missing replicated data across the remaining healthy resources.

That has several operational implications:

1. **Rebuild consumes cluster resources**
   CPU, storage bandwidth, network bandwidth, and disk I/O may be used during rebuild.

2. **Capacity matters**
   The cluster needs enough free capacity to recreate the missing copies. Nutanix has a **Rebuild Capacity Reservation** feature that reserves capacity for recovery based on fault tolerance, failure domain, and total storage capacity. ([Nutanix Portal][2])

3. **Failure domain matters**
   A rebuild after one failed disk is different from a failed node, block, or rack. The larger the failed domain, the more data may need to be recreated.

4. **Workload impact depends on context**
   A healthy, well-sized cluster may rebuild with limited impact. A cluster already under high write load, low free capacity, hardware degradation, or network congestion may experience latency or service impact.

5. **AOS tries to distinguish planned maintenance from real failure**
   Nutanix has cluster resiliency preference options. In “Smart” mode, the system differentiates between failure and maintenance activity to avoid unnecessary rebuilds during planned operations. ([Nutanix Portal][3])

6. **Progress is visible in Prism**
   Prism can show rebuild progress and ETA for disks or nodes. Nutanix documentation states that Curator periodically sends rebuild information to Insights DB, and Prism queries this information to show rebuild progress in resiliency widgets. ([Nutanix Portal][4])

For an interview, avoid saying only “Nutanix copies data.” A stronger answer is:

> Nutanix AOS distributes VM data across the cluster with redundancy. If a disk or node fails, the system identifies which extents have lost redundancy and rebuilds the missing copies onto healthy storage, while balancing resiliency, performance, capacity, and workload impact.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, rebuild and resync matter because they sit at the intersection of **data availability, customer trust, incident communication, and operational risk**.

You do not need to sound like the deepest storage kernel engineer. You need to sound like someone who can manage a high-severity escalation involving customer data risk, performance degradation, hardware replacement, and executive pressure.

This topic matters because:

1. **Customers panic when they see “resiliency degraded”**
   Even if workloads are still running, the customer may interpret it as data loss risk. A support manager must explain risk accurately.

2. **Rebuild can affect performance**
   During rebuild, customers may see increased latency, lower throughput, or noisy-neighbor effects.

3. **Rebuild depends on capacity**
   If the cluster lacks free capacity, the system may not be able to fully restore resiliency. Nutanix’s rebuild capacity reservation exists specifically to help ensure recovery capacity is available. ([Nutanix Portal][2])

4. **Escalations require prioritization**
   The manager must decide whether this is a hardware dispatch issue, performance issue, capacity emergency, software issue, network issue, or customer communication issue.

5. **It tests SRE/support maturity**
   Interviewers may want to see if you understand the difference between:

   * service down,
   * degraded resiliency,
   * performance degradation,
   * data unavailable,
   * data loss risk,
   * and customer-perceived outage.

Your positioning should be:

> I may not be the person debugging every internal AOS component, but I understand the operational risk model: what failed, what redundancy remains, whether rebuild is progressing, whether capacity is sufficient, whether workloads are impacted, and how to communicate the situation clearly to the customer and engineering.

---

## 4. Key concepts

### AOS

**AOS** is the Nutanix software layer that provides the distributed storage, virtualization integration, data protection, and operational intelligence of the platform.

For rebuild/resync, AOS is responsible for maintaining storage resiliency after failures.

---

### Distributed Storage Fabric

Nutanix does not depend on an external SAN as the primary model. Each node contributes local storage, and AOS aggregates those resources into a distributed storage pool.

Nutanix documentation describes AOS storage as providing storage resources locally to VMs on the same host while all controllers in the cluster cooperate to serve I/O. ([Nutanix Portal][5])

Interview phrasing:

> Rebuild is faster and more flexible than traditional array-centric thinking because the work is distributed across the cluster rather than tied to a single controller or RAID group.

---

### CVM — Controller VM

Each Nutanix node runs a **Controller VM**, which provides storage services for that node and participates in the distributed cluster.

If a CVM, host, disk, or node has an issue, storage resiliency and I/O routing may be affected depending on the scenario.

---

### Stargate

**Stargate** is commonly described in Nutanix architecture as the data I/O service inside the CVM. It handles read/write I/O and interacts with the distributed storage layer.

For your level, know this:

> Stargate is involved in serving storage I/O. If Stargate or the CVM on a node is unavailable, I/O may be served through other CVMs and the cluster may need to restore resiliency.

You do not need to memorize internal process behavior unless the interview becomes very deep.

---

### Curator

**Curator** is associated with background scans, cleanup, and data placement activities in Nutanix architecture. For rebuild visibility, official Prism documentation states that **Curator periodically sends rebuild information to Insights DB**, which Prism then uses to display rebuild progress. ([Nutanix Portal][4])

Interview phrasing:

> I understand that rebuild is not just a UI state; there are background services responsible for tracking and coordinating data resiliency, and Prism surfaces that progress to operations teams.

---

### Replication Factor: RF2 / RF3

**RF2** generally means two copies of data. It is designed to tolerate one relevant failure, depending on failure domain and cluster health.

**RF3** provides higher resiliency, typically for more critical environments, but requires more capacity and appropriate cluster sizing.

Do not overstate. Say:

> The exact failure tolerance depends on the configured replication factor, failure domain, available capacity, and current cluster health.

---

### Failure domain

A **failure domain** is the unit of failure the cluster is designed to tolerate, such as:

* disk,
* node,
* block,
* rack.

Nutanix rebuild capacity calculations consider parameters such as **fault tolerance, failure domain, and total storage capacity**. ([Nutanix Portal][2])

---

### Rebuild Capacity Reservation

This is a Nutanix feature that reserves storage capacity so the cluster has space to rebuild after a failure.

Official Nutanix documentation says AOS can reserve capacity for rebuilding failed disks, nodes, blocks, or racks, and that the required capacity is calculated based on parameters such as fault tolerance, failure domain, and total cluster storage capacity. ([Nutanix Portal][6])

Important operational point:

> A cluster can have enough capacity to run today’s workloads but not enough safe headroom to rebuild after a failure.

That is a strong support-manager insight.

---

### Data Resiliency / Cluster Resiliency widgets

Prism provides resiliency status and rebuild progress. For disk and node rebuilds, Nutanix documentation describes monitoring progress and ETA from Prism resiliency widgets. ([Nutanix Portal][4])

In an escalation, these widgets are part of your first evidence-gathering path.

---

### Planned maintenance vs actual failure

A planned node reboot should not always trigger the same response as an unexpected node failure. Nutanix documentation notes that the “Smart” resiliency preference can distinguish planned outage or maintenance activity from failure to avoid unnecessary rebuilds. ([Nutanix Portal][3])

This is very interview-relevant because managers care about unnecessary customer impact.

---

## 5. How it appears in a real escalation

### Scenario 1 — Disk failure, rebuild in progress

A customer reports:

> “Prism shows degraded resiliency after a disk failure. Are we at risk of data loss?”

Your support response should establish:

* Which disk failed?
* Is the disk fully failed, degraded, or intermittently failing?
* Is rebuild running?
* What is the rebuild percentage and ETA?
* Is the cluster still fault tolerant?
* Is there sufficient free capacity?
* Are workloads impacted?
* Is there abnormal latency?
* Is hardware replacement required?
* Are there other simultaneous alerts?

Customer-facing explanation:

> The cluster has detected a failed disk and is rebuilding the missing redundant data onto healthy storage. The immediate priority is to confirm that rebuild is progressing, that there is enough capacity, and that no additional failure would put the customer at unacceptable risk. We would monitor Prism resiliency status, validate alerts, and coordinate hardware replacement if needed.

---

### Scenario 2 — Node failure, VMs restarted, rebuild starts

A node goes down unexpectedly. HA restarts VMs elsewhere. The customer sees both VM impact and storage resiliency warnings.

This is more severe because there are two tracks:

1. **Compute availability** — did VMs restart successfully?
2. **Storage resiliency** — is data still protected and rebuilding?

Your escalation structure:

* Restore service first.
* Confirm current data resiliency.
* Confirm rebuild progress.
* Determine whether the node is recoverable.
* Avoid making the situation worse through uncontrolled reboots.
* Coordinate with hardware, SRE, engineering, and customer stakeholders.

Strong interview phrase:

> I would separate service restoration from resiliency restoration. The workloads may be back online before the cluster has fully restored redundancy, so communication must distinguish availability from full resiliency.

---

### Scenario 3 — Rebuild stuck or very slow

Customer reports:

> “Rebuild has been running for hours and seems stuck.”

Possible causes:

* heavy production I/O,
* insufficient capacity,
* another disk or node issue,
* network congestion,
* CVM/service issue,
* metadata/service health problem,
* software bug,
* throttling or prioritization,
* repeated component flapping.

Manager-level response:

> I would not immediately assume a product defect. I would first validate whether rebuild is genuinely stalled or just slow, compare progress over time, check concurrent alerts, verify capacity headroom, look at cluster health, review workload pressure, and escalate to SRE/engineering if the rebuild process is not making forward progress.

---

### Scenario 4 — Low capacity blocks safe rebuild

Customer has high storage utilization and then loses a disk or node. Prism warns that the cluster cannot guarantee rebuild capacity.

This is a classic enterprise support escalation because the customer may have created the risk through capacity pressure, but support still needs to help calmly.

Your message:

> The cluster needs free physical capacity not only to run workloads but also to restore redundancy after failure. If usage is too high, the priority is to reduce risk: stop non-essential writes, delete or move unnecessary data if safe, add capacity, replace failed hardware, and monitor resiliency until the cluster returns to a protected state.

Nutanix documentation explicitly warns that enabling rebuild capacity reservation near the resilient capacity threshold can create outage risk, and recommends ensuring usage is not above the documented threshold before enabling it. ([Nutanix Portal][2])

---

### Scenario 5 — Maintenance triggers unexpected resync

After planned maintenance, the customer sees resync/rebuild activity and performance degradation.

Your escalation approach:

* Was maintenance properly planned?
* Was the node placed into the correct maintenance mode?
* Was the outage duration long enough to trigger rebuild?
* Was the cluster resiliency preference set appropriately?
* Did another component fail during maintenance?
* Are workloads latency-sensitive?

Interview phrasing:

> I would treat planned maintenance as a risk-controlled operation. If rebuild starts unexpectedly, I would check whether the system interpreted the event as a failure, whether the maintenance workflow was followed correctly, and whether customer impact requires throttling, scheduling, or escalation.

---

## 6. Triage questions I should ask

Use these in interviews and mock escalations.

### Impact

1. Are production workloads down, degraded, or only showing resiliency alerts?
2. Is the issue affecting all VMs or a subset?
3. Is there customer-visible latency, I/O timeout, application error, or only Prism alarms?
4. What is the business criticality of the affected workloads?

### Failure scope

5. What failed: disk, node, CVM, host, block, rack, network path, or service?
6. Is the failed component hard down or flapping?
7. Are there multiple simultaneous failures?
8. Is this a planned maintenance event or an unexpected failure?

### Resiliency

9. What is the current resiliency state in Prism?
10. Is rebuild/resync running?
11. What is the rebuild percentage and ETA?
12. Is the cluster still able to tolerate another failure?
13. What replication factor / fault tolerance is configured?

### Capacity

14. How much usable and physical capacity is available?
15. Is rebuild capacity reserved?
16. Is the cluster near its resilient capacity threshold?
17. Are snapshots, clones, logs, or backups consuming unexpected space?

### Performance

18. Did latency increase after rebuild started?
19. Are there hot VMs, heavy write workloads, or backup jobs running?
20. Is the network healthy between nodes?
21. Are any disks showing high latency or errors?

### Operations

22. Has hardware replacement been initiated?
23. Were any recent upgrades, patches, or maintenance actions performed?
24. Are there open NCC/health check failures?
25. Is engineering escalation needed because rebuild is stalled, unsafe, or causing severe impact?

### Communication

26. Who is the customer incident commander?
27. What update cadence does the customer expect?
28. What is the next measurable milestone: rebuild percentage, hardware replacement, service recovery, or RCA?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you handle a customer escalation where a Nutanix cluster is rebuilding after a node failure?
2. How do you communicate degraded resiliency without creating panic?
3. How do you balance customer pressure for an ETA with technical uncertainty?
4. What KPIs would you track during a rebuild-related escalation?
5. How would you coordinate support, SRE, engineering, field, and account teams?
6. How do you distinguish service restoration from full resiliency restoration?
7. What would you do if a customer wants to reboot more nodes while rebuild is still in progress?
8. How would you manage a Sev1 where performance is degraded during rebuild?
9. How would you coach a junior support engineer handling a rebuild case?
10. How would you write the executive summary after the incident?

### Technical / SRE interview

1. What is the difference between rebuild and resync?
2. What happens when a disk fails in a Nutanix cluster?
3. What happens when a node fails?
4. Why does capacity matter during rebuild?
5. What is RF2 vs RF3?
6. What is a failure domain?
7. How can rebuild affect performance?
8. How would you verify whether rebuild is progressing?
9. What would you check if rebuild appears stuck?
10. What role do Prism, Curator, Stargate, and CVMs play conceptually?

### Customer-management / panel interview

1. A customer says, “Are we going to lose data?” What do you say?
2. A customer demands an exact rebuild ETA. How do you respond?
3. The customer has low free capacity and a failed disk. What is your plan?
4. The customer wants to postpone hardware replacement because workloads are running. How do you respond?
5. Engineering says the cluster is safe, but the customer’s CIO is worried. What do you communicate?
6. How do you avoid overpromising during an active rebuild?
7. How would you handle a rebuild that causes latency in a mission-critical database?
8. How would you structure the bridge call?

---

## 8. Model answers in English

### Q1. What is rebuild in Nutanix AOS?

**Model answer:**

> In Nutanix AOS, rebuild is the self-healing process that restores data resiliency after a disk, node, or larger failure domain becomes unavailable. Since Nutanix uses distributed storage rather than a traditional RAID model, the cluster identifies the data that has lost redundancy and recreates the missing copies across healthy resources. From a support perspective, I would monitor whether rebuild is progressing, whether the cluster still has enough capacity, whether workloads are impacted, and whether the customer remains protected against additional failures.

---

### Q2. What is the difference between rebuild and resync?

**Model answer:**

> I use rebuild to refer specifically to recreating missing redundant data after a failure, such as a failed disk or node. Resync is a broader operational term: it can describe the cluster bringing data placement, redundancy, or metadata state back into consistency after an interruption, maintenance event, or topology change. In an escalation, I would focus less on terminology and more on the operational questions: what triggered it, is resiliency degraded, is progress being made, is capacity sufficient, and is there customer impact?

---

### Q3. Why does rebuild capacity matter?

**Model answer:**

> Rebuild capacity matters because a cluster needs free physical capacity to recreate missing redundant copies after a failure. A customer may have enough capacity to run workloads day to day, but not enough headroom to safely rebuild after losing a disk, node, or other failure domain. That changes the risk profile of the incident. I would check free capacity, resilient capacity, rebuild capacity reservation, and whether the cluster can return to the expected fault-tolerant state.

---

### Q4. How would you handle a rebuild escalation as a support manager?

**Model answer:**

> I would structure the escalation around impact, resiliency, progress, and communication. First, I would confirm whether workloads are down, degraded, or only showing resiliency alerts. Second, I would validate the failure scope: disk, node, CVM, host, or network. Third, I would check whether rebuild is progressing and whether there is enough capacity. Fourth, I would assign clear owners for hardware replacement, technical investigation, customer updates, and engineering escalation if needed. I would communicate in business terms: current impact, current risk, actions in progress, next milestone, and what we are doing to reduce risk.

---

### Q5. A customer asks, “Are we going to lose data?” What do you say?

**Model answer:**

> I would avoid both panic and false reassurance. I would say: “At this moment, we need to validate the current resiliency state and confirm whether the cluster still has the required redundant copies. The system is designed to rebuild lost redundancy after component failures, and we are checking whether that process is progressing normally. We will also verify whether the cluster can tolerate another failure during the rebuild window. I will give you a risk-based update, not just a generic ‘everything is fine’ answer.”

That answer is strong because it is honest, controlled, and customer-safe.

---

### Q6. What would you check if rebuild is slow?

**Model answer:**

> I would check whether the rebuild is actually stalled or simply progressing slowly. Then I would look at cluster capacity, concurrent alerts, disk health, node health, network health, CVM/service health, and workload pressure. Heavy write workloads, backup jobs, degraded hardware, low capacity, or network issues can all affect rebuild performance. If progress is not moving or the customer impact is severe, I would escalate to senior SRE or engineering with evidence: timestamps, progress trend, alerts, capacity data, performance metrics, and recent changes.

---

### Q7. How can rebuild affect application performance?

**Model answer:**

> Rebuild consumes resources: storage I/O, network bandwidth, CPU, and background service capacity. In a healthy and well-sized cluster the impact may be limited, but in a busy environment rebuild can increase latency, especially for write-heavy or latency-sensitive workloads. As a support manager, I would correlate the start of rebuild with application metrics, storage latency, VM-level symptoms, and cluster health before concluding causality.

---

### Q8. How would your SaaS/cloud background help here?

**Model answer:**

> My background in SaaS and cloud operations is directly relevant because the escalation pattern is similar: protect service availability, understand degraded redundancy, monitor recovery progress, communicate risk, and coordinate teams under pressure. In cloud operations, we often distinguish between service restored and system fully healthy. I would apply the same discipline here: workloads may be running, but until rebuild completes and resiliency returns to normal, the incident is not fully closed.

---

### Q9. What should a support manager know versus what should be escalated to engineering?

**Model answer:**

> A support manager should understand the failure model, customer impact, resiliency state, rebuild progress, capacity risk, and communication plan. I do not need to personally debug every internal AOS service at code level, but I need to know when the situation is outside normal operating behavior. For example, rebuild not progressing, inconsistent resiliency state, repeated component flapping, severe performance impact, or possible software defect should trigger escalation to senior SRE or engineering with a clean evidence package.

---

### Q10. What would you do if the customer wants to perform more maintenance while rebuild is still running?

**Model answer:**

> I would strongly advise against adding risk until we understand the current fault tolerance. During rebuild, the cluster may be in a reduced resiliency state. Taking another node or disk offline could increase the risk of data unavailability or customer impact. I would ask the technical team to confirm current resiliency, expected rebuild completion, and whether the cluster can tolerate the planned action. The safe default is to postpone non-essential maintenance until resiliency is restored.

---

## 9. Connection with my experience

Your current Harmonic background maps well to this topic. The key is to translate your experience into Nutanix language.

### 24/7 enterprise support

You already understand:

* severity classification,
* bridge calls,
* escalation ownership,
* customer communication,
* handovers,
* follow-the-sun support,
* incident timelines.

For Nutanix, the technical object changes, but the operating model is familiar.

Your phrasing:

> In my current role, I manage 24/7 enterprise incidents where we separate customer impact, platform health, recovery progress, and RCA. I would apply the same structure to Nutanix rebuild escalations: identify impact, stabilize, monitor recovery, communicate risk, and coordinate technical owners.

---

### SLA, MTTR, and KPIs

A rebuild case may not always be a full outage, so classic MTTR is not enough.

Relevant metrics:

* time to acknowledge,
* time to identify failed component,
* time to confirm resiliency state,
* rebuild start time,
* rebuild completion time,
* time to hardware dispatch,
* customer update cadence,
* workload latency during rebuild,
* time to restore full fault tolerance,
* number of escalations reopened due to incomplete closure.

Strong interview line:

> In storage resiliency incidents, I would measure both service recovery and resiliency recovery. The customer may be operational before the platform is fully protected again.

---

### Cloud operations

In AWS/Azure/GCP, you understand concepts like redundancy, failure zones, replication, and degraded infrastructure. Use that analogy carefully:

> I think of rebuild similarly to restoring redundancy after losing an availability-zone component or storage replica in cloud operations. The service may still be available, but the risk posture is degraded until replication is healthy again.

---

### Monitoring: Grafana, Kibana, alerts

Your monitoring experience helps with:

* trend-based analysis,
* detecting whether rebuild is progressing,
* correlating latency with background operations,
* distinguishing symptoms from root cause,
* building incident timelines.

Interview phrase:

> I would not rely only on one alert. I would correlate Prism status, performance metrics, logs, customer symptoms, and recent changes.

---

### Jira, Confluence, Salesforce

This is directly relevant for support management:

* Salesforce: customer case ownership and communication.
* Jira: engineering escalation and defect tracking.
* Confluence: runbooks, postmortems, known-error articles, training.
* Incident process: handover, SLA tracking, RCA.

Your Confluence “Mutantix Training” is good preparation because it mirrors how enterprise support teams institutionalize knowledge.

---

## 10. Minimum I need to memorize

Memorize these points cold.

1. **Rebuild restores lost redundancy after a disk, node, or failure domain issue.**

2. **Nutanix storage is distributed, not traditional RAID-centric.**

3. **RF2 generally means two copies and tolerance of one relevant failure; RF3 provides higher resiliency but needs more resources and correct design.**

4. **Capacity is critical. A cluster needs enough free/rebuild capacity to restore redundancy.**

5. **Failure domain matters: disk failure is different from node, block, or rack failure.**

6. **Rebuild can impact performance because it consumes I/O, network, and compute resources.**

7. **Prism shows resiliency status, rebuild progress, and ETA for disk/node rebuild scenarios.**

8. **Curator and Insights DB are involved in surfacing rebuild progress to Prism.**

9. **A support manager must distinguish between service availability and full resiliency restoration.**

10. **During rebuild, avoid unnecessary additional maintenance or risky actions.**

11. **For escalations, ask: what failed, what impact, what resiliency remains, is rebuild progressing, is capacity sufficient, and who owns next actions?**

12. **Your role is technical escalation management: coordinate, communicate, reduce risk, and escalate with evidence.**

---

## 11. Advanced / optional level

You can leave these as advanced unless interviewers push deeper.

### More advanced topics

* Exact internal AOS component behavior during rebuild.
* Detailed Stargate I/O path.
* Cassandra metadata internals.
* OpLog and Extent Store internals.
* Erasure Coding interactions with rebuild.
* RF2/RF3 conversion details.
* Specific CLI commands for checking resiliency.
* NCC health check interpretation.
* Detailed AHV vs ESXi behavior during node failure.
* Network-level rebuild bottleneck analysis.
* Tuning rebuild throttling or background task prioritization.
* Rack-aware / block-aware placement specifics.
* Internal log analysis.

### How to handle advanced questions

Use this answer pattern:

> I understand the operational model and the main components. For exact internal behavior or version-specific commands, I would validate against the Nutanix documentation and involve senior SRE or engineering. In an escalation, my responsibility would be to make sure we collect the right evidence, avoid risky actions, communicate clearly, and drive the case to resolution.

This is not weak. For a manager role, it is mature.

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What rebuild means in Nutanix AOS.
* [ ] Why Nutanix rebuild is different from traditional RAID rebuild.
* [ ] What RF2 and RF3 mean operationally.
* [ ] Why capacity headroom matters.
* [ ] What rebuild capacity reservation is.
* [ ] How failure domain changes risk.
* [ ] How rebuild can affect performance.
* [ ] How to monitor rebuild progress in Prism.
* [ ] What questions to ask during a rebuild escalation.
* [ ] How to communicate degraded resiliency to a customer.
* [ ] How to distinguish service recovery from resiliency recovery.
* [ ] When to escalate to SRE or engineering.
* [ ] How your SaaS/cloud incident-management experience maps to this topic.

Your best “manager-level” summary:

> In a Nutanix rebuild escalation, I would focus on customer impact, current resiliency, rebuild progress, capacity headroom, performance impact, and risk communication. My goal would be to restore or preserve service, reduce the probability of a second failure causing greater impact, coordinate the right technical owners, and keep the customer informed with precise, evidence-based updates.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword     | Meaning                                                                                           |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| AOS                          | Nutanix operating software providing distributed storage and platform services.                   |
| AHV                          | Acropolis Hypervisor; Nutanix’s native virtualization hypervisor.                                 |
| Alert                        | Notification indicating a health, capacity, performance, or resiliency condition.                 |
| Block                        | Physical Nutanix chassis or hardware grouping; can be a failure domain.                           |
| Capacity Headroom            | Free capacity available for growth, operations, and recovery.                                     |
| Cluster                      | Group of Nutanix nodes operating as one distributed system.                                       |
| Cluster Resiliency           | Ability of the cluster to tolerate failures while preserving service and data protection.         |
| Controller VM / CVM          | Nutanix VM on each node that provides storage and cluster services.                               |
| Curator                      | Nutanix background service associated with scans, cleanup, data placement, and rebuild reporting. |
| Data Locality                | Nutanix principle of serving VM I/O locally where possible for performance.                       |
| Data Resiliency              | Ability to maintain data availability and redundancy after failures.                              |
| Degraded Resiliency          | State where the cluster has lost some expected redundancy but may still be serving workloads.     |
| Disk Failure                 | Loss or degradation of a physical storage device.                                                 |
| Distributed Storage Fabric   | Nutanix storage architecture that pools and distributes storage across nodes.                     |
| ETA                          | Estimated Time of Arrival / completion; used for rebuild completion estimates.                    |
| Extent                       | Logical unit or portion of data managed by the distributed storage layer.                         |
| Failure Domain               | Unit of infrastructure failure, such as disk, node, block, or rack.                               |
| Fault Tolerance              | Ability to withstand one or more failures without losing service or data availability.            |
| HA                           | High Availability; capability to restart or keep workloads available after failure.               |
| HCI                          | Hyperconverged Infrastructure; combines compute, storage, and virtualization in one platform.     |
| I/O                          | Input/Output; read and write operations from applications or VMs.                                 |
| Insights DB                  | Nutanix database used by Prism to surface operational and rebuild information.                    |
| Jira                         | Issue and engineering-tracking tool commonly used for escalations or defects.                     |
| KPI                          | Key Performance Indicator; metric used to track operational performance.                          |
| Latency                      | Time taken to complete an operation, especially storage read/write I/O.                           |
| Maintenance Mode             | Controlled state used to perform planned work on a host or node.                                  |
| MTTR                         | Mean Time To Repair/Recover; time required to restore service or health.                          |
| NCC                          | Nutanix Cluster Check; health-check framework used to validate cluster conditions.                |
| Node                         | Physical server in a Nutanix cluster.                                                             |
| OpLog                        | Nutanix write buffer/journal area used in the storage I/O path.                                   |
| Prism                        | Nutanix management interface for monitoring and administration.                                   |
| Prism Element                | Cluster-level Nutanix management interface.                                                       |
| Prism Central                | Centralized management interface for multiple Nutanix clusters.                                   |
| RCA                          | Root Cause Analysis; post-incident explanation of cause and corrective actions.                   |
| Rebuild                      | Process of recreating missing redundant data after failure.                                       |
| Rebuild Capacity             | Capacity needed to restore redundancy after a failed disk, node, block, or rack.                  |
| Rebuild Capacity Reservation | Feature that reserves capacity for recovery from failure.                                         |
| Recovery Milestone           | Measurable step in incident recovery, such as rebuild started or completed.                       |
| Replication Factor           | Number of redundant data copies maintained by the cluster.                                        |
| Resilient Capacity           | Capacity threshold related to maintaining failure recovery capability.                            |
| Resync                       | Process of bringing data redundancy, placement, or consistency back into expected state.          |
| RF2                          | Replication Factor 2; commonly two copies, typically tolerating one relevant failure.             |
| RF3                          | Replication Factor 3; higher redundancy, typically tolerating more failures with proper design.   |
| SLA                          | Service Level Agreement; contractual or operational service commitment.                           |
| SRE                          | Site Reliability Engineering; discipline focused on reliability, automation, and operations.      |
| Stargate                     | Nutanix CVM service commonly associated with serving storage I/O.                                 |
| Storage Pool                 | Aggregated physical storage resources in a Nutanix cluster.                                       |
| Triage                       | Initial structured investigation to classify impact, scope, risk, and next actions.               |
| VM                           | Virtual Machine; workload running on the virtualized infrastructure.                              |
| Workload Impact              | Customer-visible effect on applications, VMs, latency, or availability.                           |

[1]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v7_0%3AvSphere-Admin6-AOS-v7_0&utm_source=chatgpt.com "AOS 7.0 - vSphere Administration Guide for AOS - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7%3Awc-storage-rebuild-capacity-reserve-wc-c.html&utm_source=chatgpt.com "Prism 6.7 - Rebuild Capacity Reservation - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7%3Awc-set-cluster-resiliency-wc-t.html&utm_source=chatgpt.com "Prism 6.7 - Setting Cluster Resiliency Preference (nCLI) - Nutanix"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_8%3Awc-home-rebuild-progress-monitor-node-wc-t.html&utm_source=chatgpt.com "Prism 6.8 - Monitoring Node Rebuild Progress - Nutanix"
[5]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Aaos-storage.html&utm_source=chatgpt.com "AOS Storage - Nutanix"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism%3Awc-maintenance-resiliency-aos-c.html&utm_source=chatgpt.com "Prism 7.5 - Cluster Rebuild Preference - Nutanix"
