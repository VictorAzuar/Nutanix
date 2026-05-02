# Software Defined Networking

## 1. Short definition

**Software Defined Networking (SDN)** is a networking model where network behavior is controlled through software rather than being configured only device-by-device on physical switches, routers, and firewalls.

In a Nutanix context, SDN is mainly relevant through **Nutanix Flow Virtual Networking**, which provides software-defined networking for AHV environments using constructs such as **VPCs, subnets, VLAN networks, overlay networks, gateways, routing, NAT, VPN, and isolation policies**. Nutanix describes Flow Virtual Networking as an SDN solution that provides multi-tenant isolation, self-service provisioning, and IP address preservation using virtual network components separate from the physical network. ([Nutanix Portal][1])

---

## 2. Clear explanation

Traditional networking is built around physical devices: switches, routers, firewalls, VLANs, trunk ports, routing tables, ACLs, and physical cabling. Every change often requires coordination between virtualization, network, security, and data center teams.

**SDN abstracts part of that complexity into software.** Instead of thinking only in terms of physical switches and routers, you define logical networks in a management/control plane. Those logical definitions are then enforced across the infrastructure.

A simple way to explain it in an interview:

> “SDN separates the logical intent of the network from the physical implementation. Instead of manually configuring every network device, we define connectivity, segmentation, routing, and security policies in software, and the platform translates that intent into the required data-plane behavior.”

In Nutanix, this becomes especially relevant because the platform is not only compute and storage. Nutanix is an **HCI platform**, so networking is part of the operational model. VMs, AHV hosts, Prism Central, Flow Virtual Networking, VLANs, VPCs, overlays, physical uplinks, and external networks all need to work together.

There are two major layers to keep clear:

### Physical networking

This includes:

* Top-of-rack switches
* Physical NICs
* Bonds
* Uplinks
* LACP or active-backup configurations
* VLAN trunks
* MTU
* Physical routing
* Firewalls outside the Nutanix cluster

Nutanix AHV networking guidance includes virtual switches, uplinks, bonds, VLANs, and production network-change best practices. The AHV networking best-practices documentation has been actively updated, including updates in 2025 and 2026, which signals that this is still an operationally important area. ([Nutanix Portal][2])

### Software-defined / logical networking

This includes:

* Virtual switches
* Virtual NICs
* VLAN-backed virtual networks
* Overlay networks
* VPCs
* Subnets
* Virtual gateways
* NAT / No-NAT connectivity
* Network isolation
* VPN or network extension
* Policy-based security

Nutanix Flow Virtual Networking supports both **VLAN networks** and **overlay networks**. Nutanix documentation states that Network Controller supports creating VLAN subnets and also supports migration from VLAN Basic Subnets to VLAN Subnets, depending on version and limitations. ([Nutanix Portal][3])

The important interview point is this:

> SDN does not eliminate physical networking. It depends on it. If the underlay is broken, the overlay will also suffer.

That sentence is very useful in a Nutanix support interview.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** in Nutanix enterprise support or SRE, SDN matters because many high-severity customer escalations are not “just one VM is down.” They are often complex service-impacting events involving multiple layers:

* VM connectivity
* AHV host networking
* VLAN configuration
* Physical switch trunks
* MTU mismatch
* Prism / Flow configuration
* Routing
* Firewall rules
* Hybrid cloud connectivity
* Customer change windows
* Misaligned ownership between server, virtualization, network, and security teams

The manager does not need to be the deepest individual contributor in every protocol. But the manager must be able to **drive the escalation intelligently**.

You need to understand enough to ask:

* Is this a control-plane issue or a data-plane issue?
* Is the problem isolated to one VM, one subnet, one host, one cluster, one VLAN, or one site?
* Did it start after a network change, Prism change, Flow change, upgrade, migration, or firewall update?
* Are we dealing with underlay connectivity, overlay connectivity, routing, name resolution, security policy, or performance?
* Which teams need to be on the bridge: Nutanix support, customer network team, VMware/AHV team, security team, cloud provider, or hardware vendor?

For Nutanix specifically, SDN also matters because Flow Virtual Networking is designed to simplify and isolate networks across private and hybrid cloud environments. Nutanix positions Flow Virtual Networking as a way to simplify creating, isolating, and managing software-defined networks connecting applications across hybrid multicloud environments. ([Nutanix][4])

For a support manager, the key value is not just knowing the product names. It is being able to **coordinate diagnosis across layers without letting teams blame each other blindly**.

---

## 4. Key concepts

### Control plane vs data plane

The **control plane** decides how networking should behave. It stores or distributes configuration, topology, routes, policies, and logical network definitions.

The **data plane** forwards actual traffic.

In an SDN incident, always ask:

> “Is the configuration wrong, or is the forwarding path broken?”

Examples:

* A VPC exists in Prism, but traffic does not flow.
* A subnet is configured, but VMs cannot reach the gateway.
* Routing looks correct, but packets are dropped by a firewall.
* A policy was pushed, but enforcement is inconsistent.
* A physical uplink fails, affecting the data path while the logical configuration appears healthy.

### Underlay vs overlay

The **underlay** is the physical or base network: switches, routers, NICs, bonds, VLAN trunks, MTU, cabling, routing.

The **overlay** is the logical network built on top of the underlay. In Nutanix Flow Virtual Networking, overlay networks can provide isolated virtual networking constructs such as VPCs and subnets.

Interview phrasing:

> “In SDN troubleshooting, I separate underlay and overlay. The overlay gives agility and isolation, but it depends on a healthy underlay: physical links, VLAN reachability, routing, MTU, and consistent switch configuration.”

### VLAN networks

A **VLAN** is a Layer 2 segmentation mechanism. In Nutanix AHV, each virtual network maps to a VLAN and bridge, and Nutanix documentation states that VLANs and AHV virtual networks must also exist on the physical top-of-rack switches unless automation handles that provisioning. ([Nutanix Portal][5])

For troubleshooting, VLAN problems often look like:

* VM has no network connectivity
* VM can reach local subnet but not external networks
* Some AHV hosts work, others do not
* VMs lose connectivity after migration
* One VLAN works, another fails
* Customer forgot to trunk VLAN on ToR switch
* Native VLAN mismatch
* Incorrect VLAN ID in Prism

### Overlay networks

An **overlay network** creates logical connectivity independent of the direct physical VLAN design. This is useful for tenant isolation, IP preservation, and self-service provisioning.

Nutanix describes Flow Virtual Networking as providing multi-tenant isolation and IP address preservation using VPCs, subnets, and virtual components separate from the physical network. ([Nutanix Portal][1])

For interviews, emphasize:

> “Overlay networking gives flexibility, but it adds another troubleshooting layer. I would verify the logical configuration, then validate the underlying transport: host connectivity, MTU, routing, and external gateway reachability.”

### VPC

A **Virtual Private Cloud** is an isolated virtual network domain. Nutanix documentation describes a VPC as a virtualized network of resources isolated from the rest of the resource pool, supporting secure virtual networking with automation and scaling. ([Nutanix Portal][6])

In an enterprise support context, VPCs matter because they allow:

* Tenant isolation
* Environment isolation
* Application segmentation
* IP overlap handling
* Controlled ingress/egress
* Hybrid cloud-style networking inside private cloud

### Subnets

A **subnet** defines an IP network range. In SDN, subnets are not only “IP ranges”; they are also logical objects attached to a VPC, VLAN, gateway, routing domain, or policy framework.

Nutanix Flow Virtual Networking supports creating subnets and distinguishes between subnet types such as VLAN and overlay subnets. ([Nutanix Portal][7])

### NAT and No-NAT

**NAT** translates private/internal IP addresses to another address when traffic exits a network. **No-NAT** preserves the original IP addressing end-to-end.

Nutanix developer documentation explains that Flow Virtual Networking can create overlay networks and VPCs with network spaces separate from other VPCs, and discusses NAT and No-NAT VPC designs. ([nutanix.dev][8])

Operationally:

* NAT is simpler when the external network does not need to know internal VPC routes.
* No-NAT is better when customers need full routed visibility, firewall inspection, or IP preservation.
* No-NAT usually requires correct external routing.

### MTU

**MTU** is the maximum packet size allowed on a network path. Overlay networking can introduce encapsulation overhead. That means MTU mismatches can cause subtle failures, especially when small packets work but large packets fail.

Nutanix developer guidance for enabling Flow Virtual Networking notes that jumbo frame support may be needed on the physical network and virtual switch for Windows VMs, and suggests lowering VM MTU if the physical network MTU cannot be increased. ([nutanix.dev][9])

This is a strong troubleshooting point for interviews.

### Prism Central / Prism Element

**Prism Central** is the centralized management plane. **Prism Element** manages an individual cluster. Flow Virtual Networking is deployed and managed through Prism Central according to Nutanix documentation. ([Nutanix Portal][10])

For support management, this matters because control-plane visibility, version compatibility, and configuration state may live in Prism Central, while symptoms may appear on individual AHV hosts, VMs, or clusters.

### AHV networking

**AHV** is Nutanix’s native hypervisor. AHV networking involves virtual switches, bridges, bonds, uplinks, VLANs, and VM NICs.

Nutanix AHV networking documentation recommends using Prism web interface exclusively for network management from certain versions onward, rather than making manual changes directly on hosts. ([Nutanix Portal][11])

That matters in support because manual host-level changes can create configuration drift and increase risk during escalations.

---

## 5. How it appears in a real escalation

### Scenario 1: Customer reports that VMs in a new VPC cannot reach external services

Possible symptoms:

* VMs can ping each other inside the VPC.
* VMs cannot reach DNS, internet, corporate services, or another VLAN.
* No recent application deployment issue.
* Issue started after creating a new VPC or subnet.

Likely areas:

* VPC gateway
* NAT / No-NAT configuration
* External routable IP
* Missing external route
* Firewall blocking
* DNS misconfiguration
* Incorrect subnet association
* Physical underlay issue

Manager response:

> “I would first contain the impact: which VPC, which subnets, which workloads, and whether production is affected. Then I would split the troubleshooting into inside-VPC connectivity, VPC-to-external connectivity, routing/NAT, and physical underlay validation. I would keep the customer bridge focused on evidence rather than assumptions.”

### Scenario 2: Connectivity fails only after VM migration between AHV hosts

Possible symptoms:

* VM works on Host A but not Host B.
* Same VLAN works for other VMs.
* Problem follows the host, not the VM.
* Started after maintenance, switch change, or host expansion.

Likely areas:

* VLAN not trunked on one ToR port
* Bond/uplink misconfiguration
* LACP inconsistency
* Incorrect bridge/uplink mapping
* Physical switch port issue
* Host network drift

Manager response:

> “That pattern suggests the guest configuration may not be the primary issue. I would ask the SREs to compare host-level network configuration, uplinks, switch ports, VLAN trunking, and recent changes. From a management perspective, I would also involve the customer network team early because host-specific failures often cross the virtualization/network boundary.”

### Scenario 3: Application outage after firewall or segmentation policy change

Possible symptoms:

* VM is up.
* Network path was working before.
* Specific ports fail.
* Only one application tier is affected.
* Recent change in Flow Network Security, firewall, or external ACL.

Likely areas:

* Security policy
* Microsegmentation rule
* East-west traffic
* North-south traffic
* Firewall deny
* Incorrect application dependency mapping

Manager response:

> “I would treat it as a change-correlated incident. We need to verify the exact change window, affected flows, source/destination/port, and whether rollback is available. I would avoid broad rollback unless business impact requires it, but I would push for quick validation using packet tests, policy hits, and known-good comparisons.”

### Scenario 4: Intermittent connectivity or performance issue after enabling overlay networking

Possible symptoms:

* Ping works.
* Large file transfers fail.
* Some protocols hang.
* Windows VMs show strange behavior.
* No obvious packet loss on small tests.

Likely areas:

* MTU mismatch
* Jumbo frames not enabled end-to-end
* Encapsulation overhead
* Physical switch configuration
* Firewall dropping fragmented packets

Manager response:

> “I would specifically ask about MTU because overlay networking can introduce encapsulation overhead. I would ask the technical team to test with different packet sizes and the DF bit where possible, and to validate MTU end-to-end across VM, virtual switch, host uplink, physical switch, and external network.”

---

## 6. Triage questions I should ask

### Impact and scope

1. What is the business impact?
2. Which applications, tenants, VPCs, subnets, clusters, hosts, or sites are affected?
3. Is the issue total outage, partial outage, degraded performance, or intermittent connectivity?
4. Is it affecting north-south, east-west, or both traffic?
5. Are all VMs affected or only specific workloads?

### Timeline

6. When did the issue start?
7. What changed before the issue: upgrade, migration, switch change, firewall rule, Prism change, Flow change, host maintenance?
8. Was there a maintenance window?
9. Has the customer attempted rollback?

### Layer isolation

10. Does VM-to-VM connectivity inside the same subnet work?
11. Does VM-to-gateway connectivity work?
12. Does connectivity fail only outside the VPC?
13. Does DNS fail, or does raw IP connectivity also fail?
14. Does the problem follow the VM, the host, the subnet, the VLAN, or the cluster?

### Underlay checks

15. Are physical links up?
16. Are all required VLANs trunked on all relevant switch ports?
17. Are bonds/LACP configured consistently?
18. Is MTU consistent end-to-end?
19. Are there physical switch errors, drops, flaps, or spanning-tree events?

### Overlay / SDN checks

20. Is the VPC configured correctly?
21. Is the subnet associated with the correct VPC?
22. Is NAT or No-NAT expected?
23. Are routes advertised or configured correctly?
24. Are security policies blocking traffic?
25. Is the issue isolated to overlay networks or also present on VLAN-backed networks?

### Support management questions

26. Who owns the customer network, firewall, cloud, and virtualization layers?
27. Do we need a joint bridge with Nutanix, customer network team, security team, and application owner?
28. Is there a workaround while root cause analysis continues?
29. What evidence do we need before escalating internally?
30. What customer update cadence is required?

---

## 7. Likely interview questions

1. What is Software Defined Networking?
2. How would you explain SDN to an enterprise customer?
3. What is the difference between underlay and overlay networking?
4. How does SDN apply to Nutanix?
5. What is Nutanix Flow Virtual Networking?
6. What is the role of Prism Central in Flow Virtual Networking?
7. What is a VPC?
8. What is the difference between VLAN and overlay networking?
9. How would you troubleshoot a VM that cannot reach the network in AHV?
10. How would you handle an escalation where VMs lose connectivity after migration?
11. How would you troubleshoot a VPC where internal communication works but external connectivity fails?
12. What is the difference between NAT and No-NAT?
13. Why does MTU matter in SDN?
14. What would you ask the customer during a high-severity network escalation?
15. How do you avoid finger-pointing between virtualization, network, and security teams?
16. As a manager, how technical do you need to be in an SDN escalation?
17. How would you communicate a complex networking issue to executives?
18. How would you coach a support team to improve SDN troubleshooting?
19. What KPIs would you use to evaluate network escalation performance?
20. What is your role if the issue is caused by the customer’s physical network, not Nutanix?

---

## 8. Model answers in English

### Question: What is Software Defined Networking?

**Model answer:**

> Software Defined Networking is an approach where network configuration and behavior are controlled through software rather than only through manual configuration of physical devices. The key idea is abstraction: we define logical networks, policies, segmentation, and routing in a control plane, and the platform enforces that behavior in the data plane.
>
> In an enterprise support context, I see SDN as a way to increase agility and consistency, but also as an additional layer that must be troubleshot carefully. I would always separate the logical SDN configuration from the physical underlay, because an overlay network still depends on healthy physical connectivity, VLANs, uplinks, routing, and MTU.

### Question: How does SDN relate to Nutanix?

**Model answer:**

> In Nutanix, SDN is relevant through AHV networking and Nutanix Flow Virtual Networking. Flow Virtual Networking allows customers to create logical networking constructs such as VPCs, subnets, VLAN or overlay networks, and external connectivity models. This helps with multi-tenancy, isolation, automation, and hybrid cloud-style operations.
>
> From a support manager perspective, I do not need to replace a senior SRE, but I need to understand the architecture well enough to guide triage, ask the right questions, identify the right teams, and make sure we separate Nutanix platform issues from physical network, firewall, routing, or customer-side misconfiguration.

### Question: What is the difference between underlay and overlay?

**Model answer:**

> The underlay is the physical or base network: switches, routers, NICs, uplinks, bonds, VLAN trunks, MTU, and routing. The overlay is the logical network created on top of that underlay, such as a VPC or software-defined subnet.
>
> In troubleshooting, I would never assume the overlay is broken just because the application cannot connect. I would validate whether the issue is inside the logical network, at the gateway, in routing or NAT, in security policy, or in the physical underlay. A healthy SDN environment requires both a correct logical configuration and a reliable physical network.

### Question: How would you troubleshoot a VM connectivity issue in AHV?

**Model answer:**

> I would start with scope and recent changes. Is it one VM, one subnet, one VLAN, one host, one cluster, or one site? Then I would check whether the VM has the correct IP configuration, vNIC, subnet, gateway, DNS, and security policy.
>
> Next, I would isolate the failure domain: VM-to-VM in the same subnet, VM-to-gateway, VM-to-external IP, DNS resolution, and application port connectivity. If the issue follows the host after migration, I would suspect host uplink, VLAN trunking, bond configuration, or physical switch configuration. If the issue appears only in overlay networks, I would also check VPC, subnet, gateway, NAT or routing, and MTU.

### Question: A customer says their VPC workloads cannot reach external services. What do you do?

**Model answer:**

> I would first establish impact: affected VPC, subnets, workloads, business services, and severity. Then I would validate internal connectivity inside the VPC. If internal connectivity works but external access fails, I would focus on VPC gateway, NAT or No-NAT design, routes, firewall rules, DNS, and external network reachability.
>
> As the support manager, I would make sure the right stakeholders are on the bridge: Nutanix SREs, the customer’s network team, firewall team, and application owner. I would also push for a clear communication rhythm: current impact, working theory, actions in progress, next update time, and rollback or workaround options.

### Question: Why does MTU matter in SDN?

**Model answer:**

> MTU matters because overlay networking can add encapsulation overhead. A path may pass small packets but fail with larger packets, which creates confusing symptoms: ping may work, but file transfers, database connections, or certain application protocols may hang or fail.
>
> In an escalation, I would ask the technical team to test packet size and validate MTU consistently across the VM, virtual switch, host uplink, physical switch, and external path. I would also check whether jumbo frames are required or whether the VM MTU needs to be adjusted.

### Question: What is your role as a manager in a deep technical SDN escalation?

**Model answer:**

> My role is to structure the escalation, not to pretend I am the deepest expert in every protocol. I need enough technical fluency to challenge assumptions, ask the right diagnostic questions, separate underlay from overlay, and ensure the right specialists are involved.
>
> I would drive impact assessment, customer communication, prioritization, evidence collection, internal escalation, and post-incident learning. Technically, I would make sure the team validates the full path: VM, vNIC, subnet, VPC or VLAN, gateway, policy, routing, physical uplinks, external firewall, and recent changes.

### Question: How do you prevent finger-pointing in network escalations?

**Model answer:**

> I focus the bridge on evidence and failure-domain isolation. Instead of saying “it is the network” or “it is the platform,” I would ask the team to prove where traffic stops. Does it leave the VM? Does it reach the gateway? Is it seen on the physical switch? Is it blocked by policy? Does it fail only after migration?
>
> I also make ownership explicit. Nutanix may own the platform layer, the customer may own physical switches or firewalls, and a cloud provider may own another part of the path. My job is to coordinate those owners around data, not assumptions.

---

## 9. Connection with my experience

Your background maps well to this topic because SDN escalations are essentially **multi-layer incident-management problems**.

From your Harmonic / cloud operations experience, you already understand:

* 24/7 support operations
* Severity management
* SLA and MTTR pressure
* Customer communication during incidents
* Monitoring and observability
* Escalation ownership
* Cross-functional coordination
* Post-incident reviews
* SaaS/cloud infrastructure complexity
* Routing issues, firewall issues, DNS issues, Kubernetes networking symptoms, and cloud security groups

The positioning you want is:

> “I am not applying as a Senior SRE individual contributor. I am applying as a technical escalation manager who can understand SDN and Nutanix architecture deeply enough to lead the response, coordinate specialists, communicate clearly with customers, and drive incidents to resolution.”

You can connect SDN to your experience like this:

> “In cloud and SaaS operations, I have often seen incidents where the application symptom is misleading. A service may look down, but the root cause is DNS, routing, firewall policy, load balancer configuration, or network segmentation. I would apply the same structured approach in Nutanix: isolate the layer, validate recent changes, measure impact, coordinate the right experts, and keep the customer informed.”

That is a strong answer because it converts your gap — not being a Nutanix-native SRE — into a management strength: structured escalation leadership.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **SDN means network behavior controlled through software-defined logical constructs.**
2. **Control plane decides; data plane forwards.**
3. **Underlay is physical; overlay is logical.**
4. **Overlay depends on a healthy underlay.**
5. **Nutanix Flow Virtual Networking is Nutanix’s SDN capability for AHV environments.**
6. **Flow uses constructs such as VPCs, subnets, VLAN networks, overlay networks, NAT, routing, and isolation.**
7. **A VPC is an isolated virtual network domain.**
8. **VLAN-backed networks still require correct physical VLAN trunking.**
9. **MTU matters because overlays may add encapsulation overhead.**
10. **In an escalation, isolate scope: VM, subnet, host, cluster, VLAN, VPC, gateway, route, firewall, physical network.**
11. **As a manager, your role is to structure triage, coordinate experts, manage communication, and drive resolution.**
12. **Avoid blaming “the network”; prove where traffic stops.**

A good 30-second verbal answer:

> “Software Defined Networking abstracts networking into software-controlled logical constructs. In Nutanix, this is relevant through AHV networking and Flow Virtual Networking, where customers can use VPCs, subnets, VLAN or overlay networks, and gateway connectivity to build isolated and automated network environments. In support, I would troubleshoot SDN by separating control plane from data plane and underlay from overlay. I would validate the VM, subnet, gateway, routing, NAT, security policy, MTU, and physical VLAN/uplink path. As a manager, my job is to coordinate the right experts, keep the customer informed, and drive evidence-based isolation rather than finger-pointing.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the first manager interview, but you should recognize the terms:

* BGP routing with VPC external connectivity
* EVPN / VXLAN concepts
* Distributed routing
* East-west microsegmentation
* Service insertion
* Network function virtualization
* Advanced packet captures on AHV hosts
* Open vSwitch internals
* Flow Network Security policy enforcement
* Detailed AHV CLI commands
* Deep LACP hashing behavior
* Asymmetric routing
* ECMP
* Hybrid cloud network extension
* VPN tunnel debugging
* IPAM integration
* Kubernetes CNI internals on Nutanix

For a technical panel, you should be able to say:

> “I am comfortable driving the troubleshooting model and understanding the architecture. For very deep packet-level or protocol-level analysis, I would involve the relevant SRE or networking specialist, while ensuring we keep a clear hypothesis, evidence trail, and customer communication plan.”

That is the right manager-level posture.

---

## 12. Final checklist

Before the interview, make sure you can explain:

* [ ] What SDN is.
* [ ] Why SDN matters in HCI.
* [ ] Difference between underlay and overlay.
* [ ] Difference between control plane and data plane.
* [ ] What Nutanix Flow Virtual Networking does.
* [ ] What AHV networking means at a high level.
* [ ] What Prism Central is used for.
* [ ] What a VPC is.
* [ ] What a subnet is.
* [ ] Difference between VLAN and overlay subnet.
* [ ] Why physical VLAN trunking still matters.
* [ ] What NAT and No-NAT mean.
* [ ] Why MTU can break overlay networking.
* [ ] How to triage VM connectivity issues.
* [ ] How to handle a customer bridge during a network escalation.
* [ ] How to avoid finger-pointing between Nutanix, customer network, firewall, and cloud teams.
* [ ] How to connect your SaaS/cloud incident-management experience to Nutanix enterprise support.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------- |
| ACL                      | Access Control List; rule set controlling permitted or denied traffic.                               |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                   |
| AOS                      | Acropolis Operating System; core Nutanix software platform.                                          |
| API                      | Application Programming Interface; software interface for automation or integration.                 |
| BGP                      | Border Gateway Protocol; routing protocol often used for dynamic route exchange.                     |
| Bond                     | Logical grouping of physical NICs for redundancy or throughput.                                      |
| Bridge                   | Virtual Layer 2 construct connecting VM traffic to host networking.                                  |
| CNI                      | Container Network Interface; Kubernetes networking plugin model.                                     |
| Control plane            | Layer that defines network configuration, topology, routes, and policies.                            |
| Data plane               | Layer that forwards actual network packets.                                                          |
| DF bit                   | “Don’t Fragment” flag used when testing MTU/path MTU behavior.                                       |
| DNS                      | Domain Name System; resolves names to IP addresses.                                                  |
| East-west traffic        | Traffic between workloads inside the data center or cloud environment.                               |
| ECMP                     | Equal-Cost Multi-Path; routing across multiple equal-cost paths.                                     |
| EVPN                     | Ethernet VPN; advanced technology for scalable Layer 2/Layer 3 overlays.                             |
| Flow Network Security    | Nutanix security capability for policy-based network protection/microsegmentation.                   |
| Flow Virtual Networking  | Nutanix SDN solution for logical networking, VPCs, subnets, isolation, and connectivity.             |
| Gateway                  | Network device or logical function used to reach another network.                                    |
| HCI                      | Hyperconverged Infrastructure; integrated compute, storage, virtualization, and management platform. |
| IPAM                     | IP Address Management; system for managing IP allocation and tracking.                               |
| Jumbo frames             | Ethernet frames larger than standard 1500-byte MTU.                                                  |
| L2                       | Layer 2; data-link layer, switching, MAC addresses, VLANs.                                           |
| L3                       | Layer 3; network layer, IP routing.                                                                  |
| LACP                     | Link Aggregation Control Protocol; protocol for bundling physical links.                             |
| MTTR                     | Mean Time To Repair/Restore; average time to recover service.                                        |
| MTU                      | Maximum Transmission Unit; maximum packet/frame size on a network path.                              |
| NAT                      | Network Address Translation; translates one IP address to another.                                   |
| No-NAT                   | Connectivity model where original IP addresses are preserved.                                        |
| North-south traffic      | Traffic entering or leaving the data center, cluster, VPC, or application environment.               |
| Overlay                  | Logical network built on top of the physical network.                                                |
| Packet capture           | Diagnostic method for inspecting network packets.                                                    |
| Prism Central            | Nutanix centralized management plane.                                                                |
| Prism Element            | Nutanix cluster-level management interface.                                                          |
| RCA                      | Root Cause Analysis; structured analysis after an incident.                                          |
| Routing                  | Process of forwarding traffic between IP networks.                                                   |
| SDN                      | Software Defined Networking; software-controlled logical networking model.                           |
| SLA                      | Service Level Agreement; contractual or operational service target.                                  |
| SRE                      | Site Reliability Engineering; discipline focused on reliability, automation, and operations.         |
| Subnet                   | IP network segment, often attached to a VLAN, VPC, or routing domain.                                |
| ToR                      | Top-of-Rack switch; physical switch connecting servers in a rack.                                    |
| Trunk port               | Switch port carrying multiple VLANs.                                                                 |
| Underlay                 | Physical or base network supporting overlay connectivity.                                            |
| Uplink                   | Physical or logical connection from host networking to external switches.                            |
| vNIC                     | Virtual Network Interface Card attached to a VM.                                                     |
| VLAN                     | Virtual Local Area Network; Layer 2 segmentation method.                                             |
| VM                       | Virtual Machine.                                                                                     |
| VPC                      | Virtual Private Cloud; isolated logical network environment.                                         |
| VPN                      | Virtual Private Network; encrypted or logical network connection across networks.                    |
| VXLAN                    | Virtual Extensible LAN; common encapsulation technology for network overlays.                        |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide-v6_0_0%3Aear-flow-nw-overview-pc.html&utm_source=chatgpt.com "Flow Virtual Networking Network Controller 6.0 - Flow Virtual ..."
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2071-AHV-Networking%3ABP-2071-AHV-Networking&utm_source=chatgpt.com "Nutanix AHV Networking Best Practices"
[3]: https://portal.nutanix.com/docs/Nutanix-Flow-Virtual-Networking-Guide-v6_0_0%3Aear-flow-nw-subnet-types-pc-c.html?utm_source=chatgpt.com "Flow Virtual Networking Network Controller 6.0 - Network Types"
[4]: https://www.nutanix.com/products/flow/networking?utm_source=chatgpt.com "Connect Virtual Networks and Extend Your Cloud Environment - Nutanix Flow"
[5]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-virtual-local-area-networks.html&utm_source=chatgpt.com "Nutanix AHV Virtual Local Area Networks"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide%3Aear-flow-nw-vpc-pc-c.html&utm_source=chatgpt.com "Flow Virtual Networking Network Controller 7.0 - Virtual Private Cloud ..."
[7]: https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide-v5_0_0%3Aear-flow-nw-create-subnet-pc-t.html&utm_source=chatgpt.com "Flow Virtual Networking Network Controller 5.0 - Creating a Subnet"
[8]: https://www.nutanix.dev/configuring-nat-and-no-nat-vpcs-in-flow-virtual-networking/?utm_source=chatgpt.com "Configuring NAT and No NAT VPCs in Flow Virtual Networking"
[9]: https://www.nutanix.dev/enabling-nutanix-flow-virtual-networking-on-ahv/?utm_source=chatgpt.com "Enabling Nutanix Flow Virtual Networking on AHV – Nutanix.dev"
[10]: https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Flow-Virtual-Networking-Guide%3ANutanix-Flow-Virtual-Networking-Guide&utm_source=chatgpt.com "Flow Virtual Networking Network Controller 7.0 - Nutanix"
[11]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2071-AHV-Networking%3Aprism-uplink-configuration-and-virtual-switches.html&utm_source=chatgpt.com "Prism Uplink Configuration and Virtual Switches - Nutanix"
