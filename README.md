# Enterprise-Linux-Provisioning --- PXE + Kickstart

Project Overview

This project demonstrates an enterprise-style Linux server provisioning and configuration-management workflow.

The automated operating-system provisioning using:

**PXE — Preboot Execution Environment**

**DHCP — IP address and PXE boot information**

**TFTP — Network boot files**

**HTTP — RHEL installation repository and Kickstart files**

**UEFI/GRUB2 — Network bootloader**

**Kickstart — Automated RHEL installation**

After the operating system is provisioned, the server will be handed over to Ansible for configuration management, security hardening, patching, application deployment, and ongoing lifecycle management.

### The overall lifecycle is:

   Bare Server
       │
       ▼
   UEFI PXE Boot
       │
       ▼
      DHCP
       │
       ▼
      TFTP
       │
       ▼
   GRUB2 Bootloader
       │
       ▼
 Kernel + initrd
       │
       ▼
 HTTP Repository
       │
       ▼
   Kickstart
       │
       ▼
 RHEL Installed
 
 ## Lab Architecture

Our current lab uses two networks on the PXE server.

                         Internet
                            │
                            │
                     VMware NAT Network
                     192.168.237.0/24
                            │
                            │
                     ┌──────▼──────┐
                     │    pxe01    │
                     │             │
                     │ ens160      │
                     │ 192.168.237.132    │
                     │             │
                     │ ens224      │
                     │ 192.168.43.134     │
                     └──────┬──────┘
                            │
                     PXE Network
                    192.168.43.0/24
                            │
                ┌───────────┼───────────┐
                │           │           │
              rhel01      rhel02      rhel03
              
#### Network interfaces

**pxe01:**

ens160:

IP: 192.168.237.132/24

Purpose: Internet/NAT connectivity

ens224:

IP: 192.168.43.134/24

Purpose: PXE provisioning network

The important design decision is that DHCP runs only on the PXE network.

We disabled VMware's DHCP service on the PXE VMnet because having two DHCP servers on the same network causes unpredictable DHCP responses.

3. What is PXE?

PXE stands for Preboot Execution Environment.

PXE allows a computer to boot an operating system installer from the network instead of requiring:

DVD
USB
Local ISO
Existing operating system

This is especially useful when an organization needs to provision hundreds or thousands of servers.

For example:

100 blank servers
       │
       ▼
      PXE
       │
       ▼
100 RHEL installations

No administrator needs to insert an ISO into every server.

4. Why PXE is required

A completely new server may have:

CPU
RAM
Disk
Network adapter
UEFI firmware

but:

No operating system
No SSH
No Ansible
No Python

Therefore, Ansible cannot manage it yet.

PXE solves this initial problem.

The server firmware itself supports network boot.

The lifecycle is:

Blank Server
     │
     ▼
UEFI Network Boot
     │
     ▼
PXE Server
     │
     ▼
RHEL Installer
     │
     ▼
RHEL OS
     │
     ▼
SSH
     │
     ▼
Ansible

This is why PXE comes before Ansible in our architecture.

5. DHCP Role in PXE

DHCP normally provides:

IP address
Subnet mask
Gateway
DNS

For PXE, DHCP also provides information about the network boot server and boot file.

Our configuration concept is:

subnet 192.168.43.0/24

range:
192.168.43.150 - 192.168.43.200

next-server:
192.168.43.134

boot file:
grub/grubx64.efi
What does next-server mean?
next-server 192.168.43.134;

It means:

The PXE client should contact 192.168.43.134 to obtain its network boot file.

Since 192.168.43.134 is pxe01, the PXE server provides the boot files.

It does not mean that next-server provides the RHEL installation itself.

6. DHCP PXE Flow

When rhel01 starts:

rhel01
   │
   │ DHCPDISCOVER
   ▼
pxe01
   │
   │ DHCPOFFER
   │ IP = 192.168.43.150
   │
   │ next-server = 192.168.43.134
   │ bootfile = grub/grubx64.efi
   ▼
rhel01

Then the client knows:

My IP:
192.168.43.150

PXE server:
192.168.43.134

Boot file:
grub/grubx64.efi
7. TFTP Role

TFTP = Trivial File Transfer Protocol.

PXE commonly uses TFTP to provide the initial bootloader and boot files.

Our TFTP server runs on pxe01.

Important:

/usr/bin/tftp

is the TFTP client.

The server package is:

tftp-server

We verified:

tftp-server-5.2-35.el9.x86_64

TFTP normally uses:

UDP 69
8. Why TFTP is used

The PXE client initially doesn't have an operating system or normal application environment.

It needs a very simple mechanism to retrieve the first bootloader.

The flow is:

PXE Client
    │
    │ TFTP
    ▼
pxe01
    │
    ├── grubx64.efi
    ├── vmlinuz
    └── initrd.img
9. UEFI PXE vs Legacy BIOS PXE

Initially we looked for:

pxelinux.0

because traditional BIOS PXE commonly uses PXELINUX.

However, our environment is:

RHEL 9
x86_64
UEFI

and our ISO contains:

/mnt/rhel9/EFI/BOOT/grubx64.efi

Therefore, we moved to the UEFI + GRUB2 approach.

Our PXE boot chain is:

UEFI Firmware
      │
      ▼
DHCP
      │
      ▼
TFTP
      │
      ▼
grubx64.efi
      │
      ▼
GRUB configuration
      │
      ▼
vmlinuz + initrd.img

This is much more appropriate for our modern RHEL 9 lab.

10. GRUB2 Role

GRUB2 is the bootloader.

Our TFTP structure is approximately:

/var/lib/tftpboot/
│
├── grub/
│   ├── grubx64.efi
│   └── grub.cfg
│
├── vmlinuz
└── initrd.img

The GRUB configuration tells the RHEL installer:

Which kernel to load
Which initrd to load
Where the installation repository is
Where the Kickstart file is
11. RHEL Kernel and initrd

From our RHEL 9 ISO we found:

/mnt/rhel9/images/pxeboot/vmlinuz
/mnt/rhel9/images/pxeboot/initrd.img
vmlinuz

This is the compressed Linux kernel.

initrd.img

This contains the initial userspace environment required to start the RHEL installer.

The PXE process loads:

vmlinuz
   +
initrd.img

Then the RHEL installer starts.

12. HTTP Repository

We use Apache HTTP Server to provide the RHEL installation tree.

The ISO contents are made available under:

/var/www/html/rhel9/

Conceptually:

/var/www/html/rhel9/
├── BaseOS/
├── AppStream/
├── EFI/
├── images/
├── isolinux/
└── ...

The client accesses it through:

http://192.168.43.134/rhel9/
13. Why HTTP is used instead of TFTP for the entire ISO

TFTP is simple but inefficient for transferring large amounts of data.

Therefore PXE normally uses:

TFTP
 ↓
Initial boot files

and then:

HTTP
 ↓
Large installation repository

This is much more efficient.

Our architecture is:

              PXE Client
                  │
             ┌────┴────┐
             │         │
            TFTP      HTTP
             │         │
             ▼         ▼
         Boot files   RHEL
                      repository
14. inst.repo

The RHEL kernel command line contains:

inst.repo=http://192.168.43.134/rhel9/

This tells Anaconda:

Use this HTTP location as the RHEL installation source.

So:

inst.repo
    │
    ▼
RHEL installation packages
15. What is Anaconda?

Anaconda is the RHEL installer.

After the kernel and initrd start, Anaconda takes over.

Without Kickstart:

Anaconda
   │
   ├── Language
   ├── Keyboard
   ├── Disk
   ├── Network
   ├── Timezone
   ├── Software
   ├── Users
   └── Security

The screenshot we saw from rhel01 displaying:

WELCOME TO RED HAT ENTERPRISE LINUX 9.0

proved that the PXE boot chain successfully reached Anaconda.

That means we successfully tested:

DHCP       ✓
UEFI PXE   ✓
TFTP       ✓
GRUB       ✓
Kernel     ✓
initrd     ✓
HTTP       ✓
Anaconda   ✓
16. Kickstart

Kickstart is Red Hat's automated installation mechanism.

Instead of an administrator answering Anaconda's questions manually, Kickstart provides the answers in a file.

For example:

Language
Keyboard
Timezone
Network
Disk partitioning
Packages
Users
Root password
SELinux
Firewall
Services
Post-install commands

The Kickstart file will be hosted on:

/var/www/html/kickstart/

For example:

rhel01.ks

and accessed through:

http://192.168.43.134/kickstart/rhel01.ks
17. inst.ks

The kernel command line will eventually contain:

inst.ks=http://192.168.43.134/kickstart/rhel01.ks

This tells Anaconda:

Download this Kickstart file and use it to automate the installation.

Therefore:

inst.repo
     │
     └── Installation source

inst.ks
     │
     └── Installation instructions

These are two completely different things.

18. Complete PXE + Kickstart Flow

Once Kickstart is added, our final provisioning flow is:

                    BLANK SERVER
                         │
                         ▼
                    UEFI PXE Boot
                         │
                         ▼
                       DHCP
                         │
              IP + PXE information
                         │
                         ▼
                       TFTP
                         │
                    grubx64.efi
                         │
                         ▼
                       GRUB
                         │
                    grub.cfg
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          vmlinuz                 initrd.img
             │                       │
             └───────────┬───────────┘
                         ▼
                     Anaconda
                         │
             ┌───────────┴───────────┐
             │                       │
         inst.repo                inst.ks
             │                       │
             ▼                       ▼
      RHEL repository           Kickstart
             │                       │
             └───────────┬───────────┘
                         ▼
                  RHEL Installation
                         │
                         ▼
                    Reboot
                         │
                         ▼
                    RHEL Server
19. Why this architecture scales

Suppose an organization has:

100 servers

Every server can perform:

PXE
 ↓
DHCP
 ↓
TFTP
 ↓
GRUB
 ↓
HTTP
 ↓
Kickstart
 ↓
RHEL

The administrator doesn't manually install each server.

For example:

rhel01 ─┐
rhel02 ─┤
rhel03 ─┤
rhel04 ─┤
...     ├──► PXE Infrastructure
rhel99 ─┤
rhel100─┘

This is the provisioning layer.

20. Where Ansible enters

This is the most important architectural separation in our project.

PXE/Kickstart:

Builds the server.

Ansible:

Manages the server.

So:

                 PROVISIONING
                      │
               PXE + Kickstart
                      │
                      ▼
                  RHEL Server
                      │
                      ▼
                    SSH
                      │
                      ▼
               Ansible Control
                      │
                      ▼
            CONFIGURATION MANAGEMENT

Ansible does not need to know about the server while it is completely blank.

Once RHEL is installed and SSH is available, we add it to Ansible inventory.

21. PXE server can also be an Ansible managed node

Our architecture will eventually be:

                  ansible01
                Control Node
                     │
           ┌─────────┼─────────┐
           │         │         │
          SSH       SSH       SSH
           │         │         │
           ▼         ▼         ▼
         pxe01     rhel01    rhel02

So:

pxe01

is:

PXE server
+
Ansible managed node

while:

ansible01

is:

Ansible Control Node

The roles are different.

22. Enterprise Lifecycle

Our final project will implement this complete lifecycle:

              SERVER PROVISIONING
                       │
                       ▼
                 PXE/Kickstart
                       │
                       ▼
                   RHEL OS
                       │
                       ▼
                ANSIBLE ONBOARDING
                       │
                       ▼
             CONFIGURATION MANAGEMENT
                       │
        ┌──────────────┼───────────────┐
        ▼              ▼               ▼
     Security       Packages        Services
        │              │               │
        └──────────────┼───────────────┘
                       ▼
                    Patching
                       │
                       ▼
                Application Deployment
                       │
                       ▼
                   Monitoring
                       │
                       ▼
                 Compliance
                       │
                       ▼
                Ongoing Management
23. Current Project Progress

We're actually further along than it might feel.

Component	Status
VMware PXE network	✅
pxe01	✅
Dual-NIC design	✅
DHCP	✅
VMware DHCP conflict identified/fixed	✅
TFTP server	✅
RHEL 9 installation tree	✅
HTTP repository	✅
UEFI PXE	✅
GRUB2	✅
vmlinuz	✅
initrd.img	✅
RHEL Anaconda	✅
Kickstart	🔄 Next
Automated RHEL installation	🔄 Next
Ansible Control Node	⏳
Inventory	⏳
Roles	⏳
Configuration Management	⏳
Patching	⏳
Security Hardening	⏳
Git/GitLab	⏳
CI/CD	⏳
The most important achievement so far

We have already proven:

BLANK VM
   ↓
UEFI PXE
   ↓
DHCP
   ↓
TFTP
   ↓
GRUB
   ↓
RHEL Kernel
   ↓
RHEL Initrd
   ↓
HTTP Repository
   ↓
ANACONDA

The next milestone is to add:

Kickstart

so that instead of seeing the language-selection screen and manually installing RHEL, rhel01 will install itself automatically.

Then we'll deliberately treat that freshly installed rhel01 as an Ansible managed node. That will give you the complete practical understanding of PXE → Kickstart → Ansible, which is exactly the foundation we want before moving into enterprise Ansible roles and automation.
