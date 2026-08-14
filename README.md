# Enterprise-Linux-Provisioning --- PXE + Kickstart

#### Project Overview:

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
