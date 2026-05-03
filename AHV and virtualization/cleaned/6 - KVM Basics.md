# 6 - KVM Basics

## 1. Purpose of this chapter

This chapter explains the KVM foundation behind AHV at the right depth for support leadership: enough to understand architecture, escalation language, and evidence paths, without pretending to be a low-level virtualization engineer.

## 2. Core definition

KVM, or Kernel-based Virtual Machine, is a Linux virtualization technology that turns the Linux kernel into a hypervisor. AHV uses KVM-based virtualization concepts, with Nutanix providing the operational, storage, management, and support layers around it.

## 3. Main explanation

KVM provides kernel-level virtualization acceleration. QEMU commonly provides user-space device emulation and VM process handling. libvirt is a management API and daemon often associated with controlling KVM/QEMU VMs.

In generic Linux environments, administrators may interact directly with KVM, QEMU, libvirt, virsh, bridges, and Linux networking. In Nutanix AHV, those layers are abstracted and managed through Nutanix workflows. The customer should normally manage VMs through Prism and supported Nutanix procedures rather than treating each AHV host as a hand-tuned Linux virtualization server.

A support manager should know KVM terms because they appear in logs, engineering escalations, compatibility discussions, and troubleshooting language. However, the support posture should remain Nutanix-specific: validate Prism tasks, AHV host health, VM configuration, CVM/storage health, networking, and supported procedures.

## 4. Key concepts

- KVM is the Linux kernel virtualization foundation.
- QEMU provides VM process and device emulation functions.
- libvirt is a management interface used in many KVM environments.
- AHV operationalizes KVM inside the Nutanix platform.
- Low-level commands can be risky if used outside supported guidance.

## 5. Support relevance

KVM basics help the support manager communicate with senior SREs and engineering. They also help prevent two errors: sounding ignorant when KVM/QEMU appears in evidence, and encouraging unsupported manual changes on production AHV hosts.

## 6. Escalation scenario

An enterprise customer finds an online KVM workaround and asks whether they can apply it directly on AHV hosts to fix a migration problem. The support manager explains that AHV is KVM-based but supported through Nutanix workflows. The team gathers the Prism task error, VM configuration, host compatibility information, relevant logs, and escalates through the supported AHV path rather than applying an unsupported host-level modification.

## 7. Triage questions

- Is the issue visible in Prism, guest OS, AHV host logs, or all three?
- Is the customer using supported Nutanix procedures or direct Linux/KVM changes?
- Does the problem involve VM process state, device emulation, migration, or host resource scheduling?
- Are QEMU/KVM/libvirt errors symptoms or root-cause evidence?
- Is engineering needed because the evidence points below the Prism/AHV workflow layer?
- Were any unsupported host-level changes made?
- Can the issue be reproduced safely without touching production hosts manually?

## 8. What to avoid

- Weak: “AHV is just KVM.” Better: “AHV uses KVM-based virtualization, but Nutanix adds the platform, management, storage, lifecycle, and support model.”
- Weak: “Use generic KVM commands from the internet.” Better: “Use supported Nutanix procedures and escalate if low-level intervention is required.”
- Weak: “I do not need to know KVM at all.” Better: “I need enough KVM vocabulary to understand logs and route escalations intelligently.”

## 9. Minimum to memorize

- KVM is the Linux virtualization foundation behind AHV concepts.
- QEMU and libvirt may appear in technical evidence.
- AHV should be operated through supported Nutanix workflows.
- Generic KVM fixes are not automatically safe or supported on AHV.
- Manager-level knowledge means knowing when to involve deep virtualization SMEs.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| AHV | Nutanix native hypervisor. |
| KVM | Kernel-based Virtual Machine. |
| libvirt | API/daemon commonly used to manage KVM/QEMU VMs. |
| Prism | Nutanix management plane. |
| QEMU | User-space emulator and VM process component used with KVM. |
| SRE | Site Reliability Engineer or senior support/operations specialist. |
| virsh | Command-line tool often used with libvirt. |
| VM process | Host-side process representing a running VM. |
