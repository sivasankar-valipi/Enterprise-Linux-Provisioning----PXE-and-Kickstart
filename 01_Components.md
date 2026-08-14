
## PXE - Preboot Execution Environment

PXE allows a computer to boot an operating system installer from the network instead of requiring: DVD USB Local ISO Existing operating system.

This is especially useful when an organization needs to provision hundreds or thousands of servers.

**Why PXE is required**

A completely new server may have CPU, RAM, Disk, Network adapter, UEFI firmware, but no operating system, no SSH, no Ansible, no Python. Therefore, Ansible cannot manage it yet.

PXE solves this initial problem.

The server firmware itself supports network boot.

The lifecycle is:

```text
Blank VM / Server
       |
       v
UEFI Network Boot
       |
       v
PXE Server (pxe01)
       |
       v
RHEL Installer (Anaconda)
       |
       v
Kickstart File (ks.cfg)
       |
       v
RHEL OS
       |
       v
SSH Access
       |
       v
Ansible Control Node
       |
       v
Ansible Automation
```

This is why PXE comes before Ansible in our architecture.

## DHCP

DHCP normally provides IP address Subnet mask Gateway DNS.

For PXE, DHCP also provides information about the network boot server and boot file.

## TFTP - Trivial File Transfer Protocol

PXE commonly uses TFTP to provide the initial bootloader and boot files.

Our TFTP server runs on pxe01.

/usr/bin/tftp - TFTP Client

/var/lib/tftpboot - TFTP Server

TFTP Port: 69

Why TFTP is used

The PXE client initially doesn't have an operating system or normal application environment. It needs a very simple mechanism to retrieve the first bootloader.

The flow is:

```text
PXE Client
    |
    | Blank VM
    |
    | DHCP
    v
PXE Server (pxe01)
    |
    | TFTP
    v
Boot Files
    |
    +-- grubx64.efi
    |
    +-- vmlinuz
    |
    +-- initrd.img
    |
    v
RHEL Installer (Anaconda)
```

## HTTP Repository

We use Apache HTTP Server to provide the RHEL installation tree.

The ISO contents are made available under:/var/www/html/rhel9/

The client accesses it through: http://192.168.43.134/rhel9/

Why HTTP is used instead of TFTP for the entire ISO

TFTP is simple but inefficient for transferring large amounts of data.

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

# Anaconda

Anaconda is the installation program used by Red Hat Enterprise Linux.

When a RHEL system is installed, Anaconda is responsible for performing the actual installation of the operating system.

# Kickstart

Kickstart is an automated installation configuration file used by RHEL and other Red Hat-based systems.

**Anaconda performs the RHEL installation, while Kickstart tells Anaconda how the installation should be performed**
