# Appendix B - Linux, Kernel, and KVM Layering

## 1. Purpose of this appendix

This appendix clarifies the basic relationship between Linux, the Linux kernel, KVM, and AHV. It is intended to remove confusion before studying KVM in more detail, without duplicating the full KVM or hypervisor chapters.

## 2. Core definition

Linux is often used to describe a full operating system, but technically Linux is the kernel: the core layer that manages CPU, memory, storage, networking, processes, and hardware access.

KVM, or Kernel-based Virtual Machine, is a virtualization technology inside the Linux kernel. It allows the Linux kernel to provide hypervisor capabilities. AHV uses KVM-based virtualization concepts, but Nutanix adds the enterprise platform, storage, management, lifecycle, and support model around it.

## 3. Main explanation

The kernel is the deepest part of an operating system. Applications and system services do not normally talk directly to hardware. They rely on the kernel to manage hardware resources safely and efficiently.

A Linux-based operating system can be understood like this:

```text
Applications and services
Linux distribution tools and system components
Linux kernel
Physical hardware
````

Ubuntu, Red Hat Enterprise Linux, Debian, SUSE, and similar systems are Linux distributions. They are full operating systems built around the Linux kernel. The Linux kernel is the common core; the distribution adds tools, services, installers, package management, support model, and configuration.

KVM sits inside the Linux kernel and adds virtualization capability. In generic Linux/KVM environments, administrators may also see QEMU, libvirt, virsh, bridges, and Linux networking. In AHV, those lower-level concepts exist behind a Nutanix-supported operating model. VMs should normally be managed through Prism and supported Nutanix workflows rather than by treating AHV hosts like generic Linux KVM servers.

The correct Nutanix mental model is:

```text
Application inside the VM
Guest operating system
Virtual hardware: vCPU, vRAM, vNIC, vDisk
AHV hypervisor layer
KVM/QEMU-based virtualization foundation
AOS, CVMs, distributed storage, and platform services
Physical Nutanix node
```

This is still a simplified model. AOS and CVMs are not just passive layers below AHV. They are tightly integrated Nutanix platform services. AHV runs the VMs; AOS and the CVMs provide distributed storage and platform services; Prism provides management, monitoring, alerts, tasks, and operational workflows.

## 4. Key concepts

* Linux technically means the kernel, although people often use “Linux” to mean a full Linux-based operating system.
* Ubuntu, Red Hat, Debian, and SUSE are Linux distributions.
* The kernel is the core layer that manages hardware resources.
* KVM is virtualization functionality inside the Linux kernel.
* AHV is KVM-based, but it is not the same thing as generic KVM.
* AHV should be operated through Nutanix-supported workflows.
* AOS, CVMs, Prism, storage, networking, and cluster health must be considered together with AHV.

## 5. Support relevance

This distinction matters because support evidence may mention Linux, KVM, QEMU, libvirt, AHV, CVMs, Prism tasks, storage paths, or host-level logs.

A support manager does not need to debug Linux kernel internals. The important skill is understanding the layers well enough to route the case correctly, avoid unsupported actions, and communicate clearly with senior SREs or engineering.

The main mistake to avoid is saying that AHV is “just KVM.” A better statement is that AHV uses KVM-based virtualization concepts, while Nutanix provides the enterprise platform, management plane, distributed storage integration, lifecycle tooling, and support model.

## 6. Escalation scenario

A customer reads an online Linux/KVM article and asks whether they can apply the same host-level workaround on AHV. The support manager explains that AHV is KVM-based, but production AHV hosts must be managed through supported Nutanix procedures. The team gathers Prism tasks, VM configuration, AHV host evidence, CVM health, storage and network indicators, and escalates through the appropriate Nutanix support path if low-level evidence is involved.

## 7. Triage questions

* Is the issue visible in the guest OS, Prism, AHV host evidence, CVM health, or multiple layers?
* Is the customer using supported Nutanix workflows or direct host-level Linux/KVM changes?
* Does the evidence mention KVM, QEMU, libvirt, or host-level virtualization components?
* Could the symptom actually come from storage, networking, guest OS, or application behavior?
* Is engineering needed because the evidence points below the normal Prism/AHV workflow layer?
* Were any unsupported host-level commands or configuration changes applied?
* What is the safest supported next step?

## 8. What to avoid

* Weak: “Linux and Ubuntu are the same thing.” Better: “Ubuntu is a Linux distribution built around the Linux kernel.”
* Weak: “KVM is a separate hypervisor installed on top of Linux like an app.” Better: “KVM is virtualization functionality inside the Linux kernel.”
* Weak: “AHV is just KVM.” Better: “AHV uses KVM-based virtualization, but Nutanix adds the platform, storage, management, lifecycle, and support model.”
* Weak: “Use generic Linux/KVM commands from the internet.” Better: “Use supported Nutanix procedures and escalate when low-level intervention is required.”

## 9. Minimum to memorize

* Linux technically means the kernel.
* A Linux distribution is a full operating system built around the Linux kernel.
* Ubuntu and Red Hat are Linux distributions.
* KVM is virtualization capability inside the Linux kernel.
* AHV is Nutanix’s enterprise hypervisor built on KVM-based concepts.
* AHV is not managed like a generic Linux KVM server.
* AHV runs VMs; AOS and CVMs provide distributed storage and platform services; Prism provides management and monitoring.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning                                                                                            |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| AHV               | Nutanix native hypervisor.                                                                         |
| AOS               | Nutanix software layer providing distributed storage and platform services.                        |
| CVM               | Controller VM that runs Nutanix services on each node.                                             |
| Distribution      | Full operating system built around the Linux kernel, such as Ubuntu or Red Hat.                    |
| Kernel            | Core operating system layer that manages hardware resources.                                       |
| KVM               | Kernel-based Virtual Machine; virtualization functionality inside the Linux kernel.                |
| Linux             | Technically the kernel; commonly used to mean a Linux-based operating system.                      |
| Prism             | Nutanix management and monitoring interface.                                                       |
| QEMU              | User-space component commonly associated with VM process and device emulation in KVM environments. |
| Ubuntu            | Example of a Linux distribution.                                                                   |

