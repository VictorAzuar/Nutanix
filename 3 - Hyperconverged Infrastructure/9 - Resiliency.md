# Resiliency

## 1. Short definition

**Resiliency** is the ability of a platform, service, team, and operating process to continue delivering business-critical service despite failures, degradation, human error, maintenance, or unexpected incidents.

In a Nutanix context, resiliency means more than “the cluster has redundancy.” It includes:

* **Data resiliency**: data remains available after disk, node, block, or rack failures.
* **Service resiliency**: VMs and applications continue running or restart quickly after host failure.
* **Operational resiliency**: support, SRE, engineering, account teams, and customers coordinate effectively during incidents.
* **Disaster recovery resiliency**: workloads can recover across clusters or sites according to RPO/RTO targets.
* **Customer resiliency**: the customer has confidence, visibility, and a realistic recovery path during escalation.

For Nutanix AOS, resiliency is strongly tied to **Replication Factor**, **fault domains**, **availability domains**, **data placement**, **self-healing**, **AHV HA**, **NCC health checks**, **Prism visibility**, and **DR/Metro Availability**.

Nutanix documentation describes its architecture as designed around the inevitability of hardware failure: a cluster can tolerate one or two failures depending on the configured replication factor and container/cluster settings while still running guest VMs and responding through Prism. ([Nutanix Portal][1])

---

## 2. Clear explanation

In traditional infrastructure, resiliency often depends on dedicated redundant hardware: SAN controllers, RAID groups, HA pairs, external storage arrays, and manually designed failover paths. In **hyperconverged infrastructure**, resiliency is implemented primarily in software across a distributed cluster.

Nutanix combines compute, storage, virtualization, management, and data protection into a distributed platform. That means resiliency is not a single feature. It is a layered model.

At the **storage layer**, Nutanix protects data by storing multiple copies of data across different failure domains. This is usually discussed through **Replication Factor**, commonly RF2 or RF3. RF2 means two copies of data; RF3 means three copies. Nutanix also uses the term **data resiliency factor** or **replication factor** to describe how redundancy and availability are achieved. ([Nutanix Portal][2])

At the **cluster layer**, Nutanix models failure domains such as disk, node, block, and rack. The idea is that copies of data should not be placed in a way where one physical failure removes all replicas. For example, if the cluster is rack-aware, data placement should avoid putting all required copies in the same rack. Nutanix documentation describes data resiliency levels for combinations of replication factor, minimum nodes, blocks, and racks, including tolerance of disk, node, block, or rack failures depending on configuration. ([Nutanix Portal][3])

At the **virtualization layer**, AHV can provide VM high availability. If an AHV host fails, VM HA restarts affected VMs on another host in the cluster, considering available memory capacity and affinity/anti-affinity rules. ([Nutanix Portal][4])

At the **application layer**, resiliency may require guest clustering or application-native HA. Nutanix AHV also supports Pacemaker and STONITH patterns for certain guest VM HA designs, where application-level clustering can migrate or restart services if the application becomes unhealthy. ([Nutanix Portal][5])

At the **disaster recovery layer**, resiliency is about surviving larger failures: cluster outage, site failure, ransomware recovery, major operational mistake, or regional disaster. Nutanix supports DR patterns using protection domains, protection policies, snapshots, replication, NearSync, asynchronous replication, synchronous replication, Metro Availability, and Nutanix Disaster Recovery. Nutanix documentation distinguishes use cases such as protection domains for certain VMware ESXi deployments and Nutanix Disaster Recovery for larger deployments that require orchestration. ([Nutanix Portal][6])

The key interview point: **resiliency is not only architecture; it is operational behavior under failure**.

As a support manager, you are not expected to debug every low-level storage component like a principal engineer, but you must be able to:

* Understand what failed.
* Understand what level of redundancy remains.
* Prioritize customer impact.
* Ask the right technical questions.
* Coordinate SREs, support engineers, engineering, account teams, and the customer.
* Avoid actions that reduce resiliency further.
* Communicate clearly about risk, RPO, RTO, workaround, recovery, and next steps.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, resiliency matters because Nutanix customers usually run enterprise workloads: databases, VDI, ERP systems, Kubernetes platforms, file services, private cloud, and hybrid cloud workloads.

The Barcelona Worldwide Support role is described as part of the EMEA Support Team, with emphasis on collaboration, customer-first attitude, complex escalations, and connections with account teams, resolution managers, and customers. ([Nutanix Careers][7]) That aligns directly with resiliency escalations, because customers do not only need a technical answer; they need risk assessment, prioritization, and confident ownership.

In an enterprise support context, resiliency is critical because:

1. **Customers judge support during failure**

   A calm incident during normal operations is easy. The real test is when a disk, node, CVM, network path, rack, or site fails and the customer asks: “Are my workloads safe?”

2. **Resiliency incidents are business-impacting**

   Even if the platform is still technically “up,” degraded redundancy may mean increased risk. A customer may be running production with no additional failure tolerance.

3. **Wrong actions can make the incident worse**

   Restarting the wrong component, evacuating the wrong host, ignoring capacity constraints, or triggering upgrades during degraded state can increase risk.

4. **Support must bridge technical and executive communication**

   SREs may speak in terms of replicas, metadata, CVMs, NCC checks, Stargate, Curator, RF, or HA. Customers may ask: “Will my SAP system go down?” A support manager must translate.

5. **Resiliency affects SLA, MTTR, and escalation severity**

   A system that is online but degraded may still require Sev1 or Sev2 handling depending on exposure, workload criticality, and remaining redundancy.

6. **Nutanix sells operational simplicity**

   Customers expect Nutanix to reduce complexity compared with legacy infrastructure. Support must reinforce that promise during escalations.

---

## 4. Key concepts

### 4.1 Resilience vs availability vs reliability vs durability

These terms are often confused in interviews.

**Reliability** means the system performs correctly over time.

**Availability** means the system is accessible when needed.

**Durability** means data is not lost.

**Resiliency** means the system can absorb failure, degrade gracefully, recover, and continue service.

Example:

* A VM may remain **available** after a host failure if it restarts on another AHV host.
* Data may remain **durable** because replicas exist on other nodes.
* The platform is **resilient** if it detects the failure, maintains service, rebalances, alerts, and allows safe recovery.

### 4.2 Replication Factor: RF2 and RF3

**Replication Factor** defines how many copies of data are stored.

* **RF2**: two copies.
* **RF3**: three copies.

RF2 generally tolerates one relevant failure, depending on the configured fault domain and cluster layout. RF3 generally tolerates two relevant failures, again depending on layout and minimum requirements. Nutanix documentation maps data resiliency levels to replication factor and minimum disk/node/block/rack requirements. ([Nutanix Portal][3])

Interview framing:

> “I understand RF as a business-risk control, not just a storage setting. In escalation, I would want to know the configured RF, current failure domain, whether the cluster is degraded, and whether another failure could cause data unavailability.”

### 4.3 Fault domains and availability domains

A **fault domain** is a boundary where one failure can affect multiple components. Examples:

* Disk
* Node
* Block/chassis
* Rack
* Site

An **availability domain** in Nutanix helps reason about how the platform tolerates hardware failure. Nutanix states that hardware failure is inevitable and that the architecture is designed to tolerate one or two failures depending on replication factor and configuration while continuing to run VMs and respond through management. ([Nutanix Portal][1])

Practical support angle:

* If one disk fails, that may be low impact.
* If one node fails, check VM HA, data resiliency, capacity, and rebuild progress.
* If one rack fails, rack awareness and proper data placement become critical.
* If a site fails, you move into DR, RPO/RTO, and failover orchestration.

### 4.4 Data locality

In HCI, local reads are important for performance. Nutanix commonly emphasizes that data should be close to the VM when possible, while still being replicated across the cluster for resiliency.

Support interpretation:

* Locality helps performance.
* Replication helps resiliency.
* During failures or migrations, the platform may need to rebalance or rebuild data, affecting performance temporarily.

### 4.5 Self-healing and rebuild

A resilient distributed system should not just detect failure; it should repair redundancy.

After a disk or node failure, the system may need to rebuild missing replicas elsewhere. During this period, the system may be in a degraded state.

Triage questions:

* Is rebuild in progress?
* Is there enough capacity?
* Is cluster health green/yellow/red?
* Are there active NCC failures?
* Is performance degraded due to rebuild traffic?
* Is the customer exposed to another failure?

### 4.6 Prism, alerts, and NCC

**Prism** is the management and monitoring interface. **NCC**, Nutanix Cluster Check, is used for health checks and validation. Nutanix documentation recommends running NCC checks before upgrade procedures, and NCC can be run from Prism Element or command line depending on context. ([Nutanix Portal][8])

In support terms, NCC is valuable because it gives a repeatable health baseline. In escalation, it helps avoid relying on opinions.

A good manager-level statement:

> “Before approving risky operational actions such as upgrades, node maintenance, or failover, I would want objective health evidence: Prism alerts, NCC results, current resiliency status, capacity, and known active failures.”

### 4.7 AHV HA

**AHV VM High Availability** restarts VMs on another host if a host fails. Nutanix documentation states that VM HA considers memory capacity when calculating available resources and respects affinity and anti-affinity rules. ([Nutanix Portal][4])

Important nuance:

* AHV HA is not the same as application HA.
* Restarting a VM is not the same as zero downtime.
* Applications may require clustering, load balancing, database replication, or guest-level HA.

### 4.8 Disaster Recovery, RPO, and RTO

**RPO**: Recovery Point Objective. How much data loss is acceptable.

**RTO**: Recovery Time Objective. How long recovery may take.

Nutanix DR options include asynchronous, NearSync, synchronous replication, Metro Availability, protection domains, and Nutanix Disaster Recovery. Nutanix documentation notes that async, NearSync, and synchronous/Metro replication require sufficient node, disk, and Foundation resources to support RPO-based snapshot frequencies. ([Nutanix Portal][9])

Metro Availability is a synchronous data protection approach that can support zero RPO for certain use cases; Nutanix documentation describes Metro Availability as supporting RPO zero and enabling failover/continued access across geographically distributed clusters for supported services. ([Nutanix Portal][10])

Interview nuance:

> “Zero RPO does not automatically mean zero operational complexity. You still need correct design, supported topology, tested failover, network considerations, witness/quorum behavior where applicable, and a clear runbook.”

### 4.9 Protection Domains and Protection Policies

Older or specific DR designs may use **Protection Domains**. Newer Nutanix DR approaches use more entity-centric protection policies and recovery plans. Nutanix documentation notes that legacy DR configurations use protection domains and third-party integrations, while Nutanix Disaster Recovery offers more automated, entity-centric protection using categories to group VMs and automate protection as applications scale. ([Nutanix Portal][11])

Support angle:

* Know whether the customer uses PD-based DR or Nutanix Disaster Recovery.
* Check whether new VMs are actually protected.
* Validate snapshot schedules and replication status.
* Confirm RPO/RTO expectations versus actual configuration.

### 4.10 Degraded state

A degraded state means the system is operating but with reduced redundancy, performance, or failover margin.

Examples:

* One disk failed and rebuild is pending.
* One node is down and HA capacity is limited.
* RF2 cluster has lost one replica and is rebuilding.
* Replication lag means DR RPO is not being met.
* Network errors are causing CVM or storage path instability.
* A rack-aware design has insufficient rack diversity.

This is a key interview concept because enterprise customers often escalate before a full outage. They ask: “Are we safe?”

---

## 5. How it appears in a real escalation

### Scenario A — Disk failure during peak production

A customer opens a Sev2 case: Prism reports a disk failure in a production cluster. VMs are still running, but the customer is worried about data loss.

Your role:

* Confirm cluster health and data resiliency status.
* Ask for Prism alerts, NCC output, affected node/disk, cluster RF, capacity, and rebuild status.
* Ensure the customer avoids unnecessary restarts or upgrades while degraded.
* Coordinate replacement path with hardware/logistics if applicable.
* Communicate risk clearly: “Workloads are running, but redundancy may be reduced until rebuild completes or hardware is replaced.”
* Track progress until the cluster returns to healthy redundancy.

What you should not do:

* Say “no impact” just because VMs are still running.
* Treat it as purely hardware if data resiliency is degraded.
* Ignore capacity, rebuild, or secondary failure risk.

### Scenario B — Node failure and VM restarts

A node goes down. Several VMs restart on other AHV hosts. Some applications recover automatically; one database application does not.

Your role:

* Separate platform recovery from application recovery.
* Confirm AHV HA behavior.
* Check whether sufficient memory/capacity existed for restart.
* Verify if affinity/anti-affinity rules affected placement.
* Ask whether the application has its own clustering or dependency order.
* Coordinate with SREs for platform, customer app owners for application, and account team for business communication.

Good customer language:

> “The platform HA mechanism can restart VMs after a host failure, but application recovery depends on the guest OS and application architecture. We are validating both layers: cluster health and workload recovery.”

### Scenario C — Cluster running with reduced resiliency before planned upgrade

Customer wants to proceed with an AOS or hypervisor upgrade, but NCC reports health-check failures.

Your role:

* Do not approve risky maintenance blindly.
* Ask for NCC results, current alerts, capacity, and known failures.
* Explain that upgrades should be performed from a healthy baseline unless a support-approved exception exists.
* Escalate internally if the upgrade is needed to resolve the issue.
* Coordinate a risk-reviewed plan.

Nutanix documentation explicitly references running NCC checks before upgrade procedures. ([Nutanix Portal][8])

### Scenario D — DR replication lag

Customer expects 15-minute RPO but replication is lagging due to bandwidth saturation or snapshot backlog.

Your role:

* Validate configured RPO versus achieved RPO.
* Ask about bandwidth, changed data rate, snapshot schedule, replication status, target cluster health, and recent workload changes.
* Explain business exposure in terms of potential data loss.
* Bring in SRE/DR specialist if needed.
* Help prioritize short-term mitigation and long-term capacity/network planning.

### Scenario E — Site failure and Metro/DR failover

A customer loses a primary site or inter-site connectivity. They ask whether they should fail over.

Your role:

* Clarify if this is planned failover, unplanned failover, or network isolation.
* Confirm DR topology: Metro Availability, async DR, NearSync, protection policies, protection domains, Leap/Nutanix DR.
* Confirm RPO/RTO and last successful replication/snapshot.
* Identify split-brain risks and quorum/witness behavior where relevant.
* Ensure a single decision owner on the customer side.
* Coordinate technical action with clear customer comms.

---

## 6. Triage questions I should ask

### Impact and severity

1. What business service or workload is affected?
2. Are VMs down, degraded, or still running?
3. Is there data unavailability, data loss, or only reduced redundancy?
4. Is this production, pre-production, VDI, database, Kubernetes, file service, or backup infrastructure?
5. What is the customer’s declared severity and business impact?

### Cluster state

6. What is the cluster name, AOS version, hypervisor, and node count?
7. Is the cluster AHV, ESXi, or mixed context?
8. What is the current Prism health status?
9. Are there active critical alerts?
10. Has NCC been run? What checks failed?

### Resiliency state

11. What is the configured Replication Factor?
12. Is the cluster currently data-resilient?
13. What failure domain is affected: disk, node, block, rack, site?
14. Is rebuild/re-replication in progress?
15. Is there enough free capacity for rebuild?
16. Is the cluster rack-aware or block-aware?
17. Is there any second failure or intermittent component?

### VM/application state

18. Which VMs are impacted?
19. Did AHV HA restart the VMs?
20. Are affinity or anti-affinity rules relevant?
21. Is the application clustered at guest level?
22. Does the customer have application-level monitoring showing errors?

### Network and storage symptoms

23. Are there latency spikes, packet loss, CVM connectivity issues, or storage path errors?
24. Are specific hosts or CVMs showing abnormal latency?
25. Did anything change recently: firmware, network, switch, VLAN, MTU, routing, firewall, DNS, NTP?

### DR questions

26. Is the workload protected by Protection Domain, Protection Policy, Nutanix DR, Metro Availability, NearSync, or async replication?
27. What is the expected RPO/RTO?
28. What is the last successful snapshot or replication point?
29. Is failover being considered? Planned or unplanned?
30. Has the customer tested this failover before?

### Operational control

31. Who is the customer incident commander?
32. Who approves disruptive actions?
33. Is there a change freeze?
34. Are account team, TAM, engineering, or resolution manager involved?
35. What is the next communication deadline?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How do you define resiliency in enterprise support?
2. How would you manage a customer escalation where the cluster is still running but resiliency is degraded?
3. How do you communicate risk to a customer without creating panic?
4. How do you balance speed of recovery with avoiding further damage?
5. How would you coordinate SREs, engineering, account teams, and customer stakeholders?
6. What KPIs would you track for resiliency-related escalations?
7. How do you ensure your team learns from incidents?
8. How do you coach engineers handling high-pressure resiliency incidents?
9. How do you handle a customer demanding immediate failover when the technical team is not ready?
10. How do you decide when to escalate internally?

### Technical / SRE interview

1. What is Replication Factor in Nutanix?
2. What is the difference between RF2 and RF3?
3. What is a failure domain?
4. What happens when a disk or node fails in an HCI cluster?
5. What is the difference between VM HA and application HA?
6. How does AHV HA help after host failure?
7. What is the role of Prism during an incident?
8. What is NCC used for?
9. What are RPO and RTO?
10. What is the difference between asynchronous replication, NearSync, and synchronous replication?
11. What is Metro Availability?
12. What risks exist during degraded resiliency?
13. What information would you collect before approving an upgrade or maintenance operation?
14. How would you troubleshoot replication lag?
15. How would you explain degraded redundancy to a non-technical customer?

### Advanced / panel interview

1. A customer has lost one node in an RF2 cluster. What do you do?
2. A customer has a failed disk and wants to continue with a planned upgrade. How do you respond?
3. A customer claims Nutanix HA failed because an application did not recover after VM restart. How do you handle it?
4. A customer has DR configured but discovers new VMs were not protected. How do you manage the escalation?
5. A customer has a site outage and wants immediate failover. What questions do you ask first?
6. A rebuild is causing performance impact. How do you communicate and prioritize?
7. A customer is technically online but has no remaining failure tolerance. Is this urgent?
8. How would you drive a post-incident review for a resiliency event?
9. How do you separate platform responsibility from customer application responsibility?
10. How do you build trust during a long-running infrastructure incident?

---

## 8. Model answers in English

### Question 1: “How do you define resiliency in enterprise support?”

**Model answer:**

> I define resiliency as the ability of a system and an operations organization to absorb failures, continue delivering service, recover safely, and learn from the event. In enterprise support, resiliency is not just redundancy. It includes architecture, monitoring, incident response, customer communication, escalation paths, and post-incident improvement.
>
> In a Nutanix context, I would think about resiliency across several layers: data resiliency through replication factor and fault domains, VM availability through AHV HA, operational visibility through Prism and NCC, and disaster recovery through protection policies, replication, and Metro Availability where applicable.
>
> As a support manager, my job is not only to know the technology, but to ensure the right people are engaged, the customer understands the risk, and we do not take actions that reduce resiliency further.

### Question 2: “A customer has one failed node, but VMs are still running. How do you manage it?”

**Model answer:**

> First, I would not treat it as a non-issue just because the workloads are still running. I would clarify the business impact, affected workloads, cluster configuration, replication factor, capacity, current alerts, and whether the cluster remains data-resilient.
>
> I would ask the technical team to validate Prism health, NCC checks, rebuild or re-replication status, and whether AHV HA restarted any VMs. I would also check whether there is enough capacity to tolerate the current failure and whether another failure would create data unavailability.
>
> From a customer-management perspective, I would communicate clearly: the platform may be operating as designed, but redundancy may be reduced until recovery completes. Then I would coordinate the remediation path, provide regular updates, and make sure no risky maintenance or upgrades are performed while the cluster is degraded unless explicitly approved by Nutanix support.

### Question 3: “What is Replication Factor?”

**Model answer:**

> Replication Factor defines how many copies of data are maintained across the Nutanix cluster. RF2 means two copies; RF3 means three copies. The goal is to maintain data availability and durability if hardware components fail.
>
> The practical point is that RF must be understood together with the failure domain. It is not enough to say “RF2 equals safe.” I need to know whether the failure is a disk, node, block, or rack failure, whether the cluster meets the minimum requirements, whether data is still resilient, and whether rebuild is progressing.
>
> In an escalation, RF helps me assess remaining risk and explain it to the customer in business terms.

### Question 4: “What is the difference between VM HA and application HA?”

**Model answer:**

> VM HA protects the virtual machine from host failure by restarting it on another available host, assuming the cluster has sufficient resources and the HA policy allows it. Application HA is different: it means the application itself can continue or recover correctly, often through clustering, replication, load balancing, or application-level failover.
>
> A VM can restart successfully while the application still has issues because of database recovery, dependency order, service corruption, licensing, network dependencies, or application clustering problems.
>
> In a customer escalation, I would separate platform recovery from application recovery. I would validate that the Nutanix layer behaved correctly, but I would also help the customer identify what is needed at the guest OS and application layer.

### Question 5: “What would you do if NCC reports failures before a planned upgrade?”

**Model answer:**

> I would pause and assess the risk before proceeding. Upgrades should normally start from a healthy baseline. If NCC reports failures, I would ask the technical team to classify them: are they informational, known false positives, warnings, or blockers?
>
> I would review Prism alerts, cluster resiliency, capacity, versions, and whether the failed checks are related to the upgrade path. If the upgrade is needed to fix the issue, I would escalate internally and ask for a support-approved plan.
>
> As a manager, I would communicate to the customer that delaying the upgrade may be the safer option unless we have a validated remediation path. The objective is not only to complete the change, but to avoid turning a controlled maintenance window into an outage.

### Question 6: “How would you explain degraded resiliency to an executive customer?”

**Model answer:**

> I would avoid deep technical jargon at first. I would say: “Your services are currently running, but the platform has reduced protection because one of the redundant components has failed. The immediate priority is to restore full redundancy before another failure occurs.”
>
> Then I would explain what we are doing: validating cluster health, checking whether data rebuild is progressing, confirming available capacity, and coordinating hardware or software remediation.
>
> I would also provide a clear risk statement, next update time, and decision points. For example: “At this stage, we do not see data unavailability, but the environment is exposed to higher risk until resiliency is restored.”

### Question 7: “What is RPO and RTO?”

**Model answer:**

> RPO is the maximum acceptable data loss measured in time. For example, a 15-minute RPO means the business accepts losing up to 15 minutes of data in a disaster scenario. RTO is the maximum acceptable recovery time: how long the service can be unavailable before it must be restored.
>
> In support, I would always compare the customer’s expected RPO/RTO with the actual technical configuration. A customer may believe they have near-zero recovery, but the environment may only have asynchronous replication or untested failover procedures.
>
> During escalation, my role is to clarify the actual recovery options, the last valid recovery point, and the operational risks of failover.

### Question 8: “How would you troubleshoot replication lag?”

**Model answer:**

> I would first confirm the expected RPO and the actual lag. Then I would check whether the issue is source-side, network-side, or target-side. I would ask about changed data rate, snapshot schedule, available bandwidth, network latency, packet loss, target cluster health, storage capacity, and recent changes.
>
> I would also check whether the replication policy is correctly configured and whether there are failed or queued snapshots. From a management perspective, I would communicate the business exposure: if replication is behind, the customer’s effective RPO is worse than expected.
>
> The short-term goal is to stabilize replication and reduce lag. The long-term goal is to ensure the design can sustain the customer’s change rate and RPO requirements.

### Question 9: “How do you manage a high-pressure resiliency escalation?”

**Model answer:**

> I use a structured incident approach. First, I establish impact, severity, customer stakeholders, technical owner, and communication cadence. Then I separate immediate containment from root cause analysis.
>
> For resiliency incidents, I want the team to answer four questions quickly: what failed, what is still protected, what is exposed, and what action could make things worse?
>
> I would keep the customer informed with facts, not speculation. Internally, I would make sure SRE, support, engineering, account teams, and resolution management are aligned. After recovery, I would drive a post-incident review focused on prevention, runbook improvements, monitoring gaps, and team learning.

---

## 9. Connection with your experience

Your current experience maps very well to this topic if you position it correctly.

You already have strong experience in:

* 24/7 enterprise support
* Incident management
* SLA and MTTR
* Escalations
* Monitoring
* Cloud operations
* Coaching and team coordination
* Jira, Confluence, Salesforce
* Grafana, Kibana
* AWS, Azure, GCP, Kubernetes, Linux

The bridge to Nutanix is:

| Your experience                | Nutanix resiliency equivalent                                        |
| ------------------------------ | -------------------------------------------------------------------- |
| SLA / MTTR management          | RTO, escalation handling, service restoration                        |
| Incident management            | Cluster degradation, node failure, DR incident, customer escalation  |
| Monitoring with Grafana/Kibana | Prism alerts, NCC checks, cluster health, performance dashboards     |
| Cloud operations               | Distributed systems, failure domains, capacity, automation, recovery |
| Kubernetes operations          | Node failure, scheduling, self-healing, workload restart patterns    |
| SaaS operations                | Customer-facing availability, incident comms, RCA, prevention        |
| Team leadership                | Coordinating SREs, support engineers, account teams, engineering     |
| Confluence/Jira/Salesforce     | Runbooks, case hygiene, postmortems, escalation tracking             |

Your positioning should be:

> “I am not presenting myself as a Senior SRE individual contributor who has spent years inside AOS internals. I am positioning myself as a technical escalation manager who understands distributed infrastructure, asks the right questions, manages risk, coordinates experts, and communicates clearly with enterprise customers.”

That is a strong and credible angle.

---

## 10. Minimum I need to memorize

You should be able to explain these verbally without notes:

1. **Resiliency is layered**

   Data, VM, application, operational, and DR resiliency are different layers.

2. **RF2 vs RF3**

   RF2 means two copies; RF3 means three copies. More replicas generally means more failure tolerance, but actual protection depends on cluster layout and failure domain.

3. **Failure domains**

   Disk, node, block, rack, and site failures are different risk levels.

4. **Degraded state**

   A cluster can be online but exposed. Online does not always mean fully protected.

5. **AHV HA**

   Restarts VMs after host failure, but does not guarantee application-level continuity.

6. **NCC and Prism**

   Prism gives visibility; NCC provides health validation, especially before risky operations like upgrades.

7. **RPO/RTO**

   RPO = acceptable data loss. RTO = acceptable recovery time.

8. **DR options**

   Async replication, NearSync, synchronous replication, Metro Availability, protection domains, protection policies, Nutanix DR.

9. **Escalation mindset**

   During a resiliency incident, ask: what failed, what is affected, what protection remains, what action is safe, who needs to know, and when is the next update?

10. **Customer communication**

Translate technical state into business risk.

---

## 11. Advanced / optional level

You do not need to master these deeply for a manager interview, but knowing they exist will help you sound credible with SREs:

1. **AOS internal services**

   Stargate, Curator, Cassandra, Zookeeper, Medusa, Genesis. Useful to recognize, not necessary to explain deeply unless asked.

2. **Metadata resiliency**

   How metadata is distributed and protected across nodes/racks.

3. **Erasure Coding**

   Capacity optimization technique with different implications from simple replication.

4. **Rack awareness details**

   Minimum node/block/rack requirements for specific fault tolerance levels.

5. **Metro Availability edge cases**

   Witness behavior, network isolation, split-brain prevention, planned vs unplanned failover.

6. **Storage rebuild mechanics**

   How re-replication/rebuild affects capacity and performance.

7. **Advanced networking**

   CVM networking, VLANs, MTU, LACP, switch redundancy, storage latency impact.

8. **Hypervisor-specific behavior**

   AHV vs ESXi HA/DR behavior.

9. **Application-consistent snapshots**

   Difference between crash-consistent and application-consistent recovery.

10. **Kubernetes on Nutanix**

Resiliency of containerized workloads running on Nutanix infrastructure.

Use these as “recognition-level” topics. You do not need to overclaim expertise.

---

## 12. Final checklist

Before an interview, verify that you can answer these confidently:

* [ ] I can define resiliency in an enterprise support context.
* [ ] I can explain why resiliency is not the same as availability.
* [ ] I can explain RF2 and RF3.
* [ ] I can explain disk, node, rack, and site failure domains.
* [ ] I can describe what happens when a host fails.
* [ ] I can explain AHV HA at a manager-technical level.
* [ ] I can explain why an application may not recover even if the VM restarts.
* [ ] I can explain RPO and RTO.
* [ ] I can describe async, NearSync, synchronous replication, and Metro Availability at a high level.
* [ ] I can explain why NCC checks matter before upgrades or risky operations.
* [ ] I can manage a degraded-resiliency escalation.
* [ ] I can communicate technical risk to executives.
* [ ] I can connect this topic to my own incident-management and cloud-operations background.
* [ ] I can avoid overclaiming deep Nutanix internals while showing strong escalation judgment.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword  | Meaning                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| AHV                       | Acropolis Hypervisor; Nutanix’s native hypervisor.                                          |
| AHV HA                    | AHV High Availability; restarts VMs on another host after host failure.                     |
| AOS                       | Acropolis Operating System; Nutanix distributed infrastructure software.                    |
| Application HA            | High availability implemented inside the application or guest OS.                           |
| Async Replication         | Replication with a non-zero RPO; data is copied on a schedule.                              |
| Availability              | Whether a service is accessible when needed.                                                |
| Availability Domain       | Logical/physical failure boundary used to reason about failure tolerance.                   |
| Block                     | Nutanix hardware chassis or grouping, depending on platform model.                          |
| Business Impact           | Real customer/service consequence of a technical issue.                                     |
| Capacity Headroom         | Spare resources needed for failover, rebuild, or growth.                                    |
| Case Hygiene              | Quality and completeness of support case documentation.                                     |
| Cluster                   | Group of Nutanix nodes operating as one distributed system.                                 |
| Confluence                | Documentation/wiki platform often used for runbooks and knowledge bases.                    |
| Crash-consistent Snapshot | Snapshot preserving disk state, but not necessarily application transaction consistency.    |
| Customer Escalation       | Situation where customer impact, urgency, or dissatisfaction requires higher attention.     |
| CVM                       | Controller VM; Nutanix VM that provides storage and cluster services on a node.             |
| Data Locality             | Keeping data close to the VM using it, improving performance.                               |
| Data Protection           | Mechanisms such as snapshots, replication, and DR policies.                                 |
| Data Resiliency           | Ability to keep data available and protected after failures.                                |
| Degraded State            | System is running but with reduced redundancy, capacity, or protection.                     |
| Disaster Recovery         | Recovery strategy for major failures such as cluster or site outage.                        |
| Disk Failure              | Loss or malfunction of a physical storage device.                                           |
| DR                        | Disaster Recovery.                                                                          |
| Erasure Coding            | Storage efficiency technique that reduces capacity overhead versus full replication.        |
| Escalation Manager        | Person coordinating technical, customer, and internal response during escalations.          |
| Failure Domain            | Scope affected by a single failure, such as disk, node, rack, or site.                      |
| Failover                  | Moving service from failed or primary location/component to another.                        |
| Fault Tolerance           | Ability to continue operating after one or more failures.                                   |
| Grafana                   | Monitoring and visualization platform.                                                      |
| Guest Clustering          | HA clustering inside guest VMs, often at OS/application level.                              |
| HA                        | High Availability.                                                                          |
| HCI                       | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in software. |
| Incident Commander        | Person coordinating incident response and decisions.                                        |
| Jira                      | Issue and workflow tracking tool.                                                           |
| KPI                       | Key Performance Indicator.                                                                  |
| LCM                       | Life Cycle Manager; Nutanix tool for software/firmware lifecycle operations.                |
| MTTR                      | Mean Time To Resolve/Recover; time needed to restore service.                               |
| Metro Availability        | Synchronous DR/availability design for supported Nutanix scenarios.                         |
| NearSync                  | Near-synchronous replication pattern with low RPO.                                          |
| NCC                       | Nutanix Cluster Check; health-check tool for Nutanix environments.                          |
| Node                      | Physical server in a Nutanix cluster.                                                       |
| Prism                     | Nutanix management and monitoring interface.                                                |
| Prism Central             | Centralized management plane for multiple Nutanix clusters.                                 |
| Prism Element             | Management interface for an individual Nutanix cluster.                                     |
| Protection Domain         | Nutanix data protection construct used in certain DR designs.                               |
| Protection Policy         | Policy-based Nutanix DR/data protection configuration.                                      |
| RCA                       | Root Cause Analysis.                                                                        |
| Rebuild                   | Process of restoring redundancy after data/component loss.                                  |
| Re-replication            | Creating replacement replicas after a failure.                                              |
| Reliability               | Ability of a system to operate correctly over time.                                         |
| Replication Factor        | Number of data copies maintained across the cluster.                                        |
| Resiliency                | Ability to absorb failure, continue service, recover, and learn.                            |
| RF2                       | Replication Factor 2; two data copies.                                                      |
| RF3                       | Replication Factor 3; three data copies.                                                    |
| RPO                       | Recovery Point Objective; maximum acceptable data loss.                                     |
| RTO                       | Recovery Time Objective; maximum acceptable recovery time.                                  |
| Salesforce                | CRM/support-case platform commonly used for customer support workflows.                     |
| Self-healing              | System behavior that automatically repairs or restores redundancy.                          |
| Sev1 / Sev2               | Severity levels for critical or high-impact incidents.                                      |
| Site Failure              | Loss of an entire datacenter, region, or location.                                          |
| SLA                       | Service Level Agreement.                                                                    |
| Snapshot                  | Point-in-time copy used for recovery or replication.                                        |
| Split-brain               | Dangerous condition where two sides believe they are active owners.                         |
| SRE                       | Site Reliability Engineer / Engineering.                                                    |
| STONITH                   | “Shoot The Other Node In The Head”; fencing mechanism in HA clusters.                       |
| Synchronous Replication   | Replication where writes are committed across sites before completion.                      |
| Triage                    | Initial assessment of impact, urgency, scope, and next actions.                             |
| VM HA                     | Virtual machine high availability; VM restart/recovery after host failure.                  |
| Witness                   | Component used in some HA designs to help avoid split-brain.                                |

[1]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v6_10%3Aarc-failure-modes-c.html?utm_source=chatgpt.com "Prism 6.10 - Availability Domains - Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_0%3Aarc-metadata-req-rack-c.html&utm_source=chatgpt.com "Rack Awareness Metadata Requirements for Rack Fault Tolerance"
[3]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_0%3Aarc-data-resiliency-rack-c.html?utm_source=chatgpt.com "Prism 7.0 - Data Resiliency Levels for Block Fault Tolerance"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2156-NC2-on-Azure%3Atn-nutanix-ahv-virtual-machine-high-availability.html&utm_source=chatgpt.com "Nutanix AHV Virtual Machine High Availability"
[5]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v11_0%3Aahv-configuration-management-HA-pacemaker-clusters-c.html?utm_source=chatgpt.com "AHV 11.0 - Configuration and Management of High-Availability Pacemaker ..."
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=Nutanix_Hybrid_Cloud_Reference_Architecture%3Anutanix-disaster-recovery.html&utm_source=chatgpt.com "Nutanix Disaster Recovery"
[7]: https://careers.nutanix.com/en/jobs/30949/manager-worldwide-support/?utm_source=chatgpt.com "Manager, Worldwide Support, Barcelona, Spain | Nutanix Careers"
[8]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_5%3Ancc-ncc-checks-run-c.html?utm_source=chatgpt.com "Prism 7.5 - Nutanix Cluster Check (NCC)"
[9]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Element-Data-Protection-Guide%3Awc-dr-nearsync-resource-requirements-r.html&utm_source=chatgpt.com "PD-Based DR 7.5 - Resource Requirements Supporting Snapshot ... - Nutanix"
[10]: https://portal.nutanix.com/docs/Nutanix-Files-Manager-v5_3%3Afil-files-metro-ahv-c.html?utm_source=chatgpt.com "Synchronous Data Protection with Metro Availability on AHV"
[11]: https://portal.nutanix.com/page/documents/details?targetId=Leap-Xi-Leap-Admin-Guide%3ALeap-Xi-Leap-Admin-Guide&utm_source=chatgpt.com "Disaster Recovery (formerly Leap) pc.2022.6 - Nutanix"
