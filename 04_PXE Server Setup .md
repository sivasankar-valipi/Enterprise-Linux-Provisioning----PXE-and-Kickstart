## PXE Server Setup

This document describes the initial setup and validation of the PXE server before configuring PXE services such as:

**DHCP**

**TFTP**

**HTTP**

**UEFI PXE**

**Kickstart**

The PXE server is a dedicated RHEL 9 server running on VMware Workstation Pro.

  ## PXE Server Architecture
  
                         
                         Internet
                            │
                            │
                       VMware NAT
                    192.168.237.0/24
                            │
                            │
                    ┌───────▼───────┐
                    │     pxe01     │
                    │               │
                    │ ens160        │
                    │ 192.168.237.x │
                    │               │
                    │ ens224        │
                    │ 192.168.43.134│
                    └───────┬───────┘
                            │
                       Host-Only
                    192.168.43.0/24
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           rhel01         rhel02         rhel03

#### Verify:

Login using the root account

**1.** Hostname

hostname - hostnamectl

**2.** IP address and Gateway
 
IP address - NAT and Host-only

Gateway - ip route

**3.** Operating System and Kernel Version

OS - cat /etc/os-release

Kernal Version- uname -a

### Register RHEL with Red Hat

The PXE server needs access to Red Hat repositories so that required packages can be installed.

subscription-manager register

**Use credentials associated with your Red Hat Developer account here**

subscription-manager status

subscription-manager identity

subscription-manager repos --list-enabled

Important repositories for our lab are AppStream and BaseOS.

yum repolist

repo id                         repo name
rhel-9-for-x86_64-baseos-rpms   Red Hat Enterprise Linux 9 BaseOS
rhel-9-for-x86_64-appstream-rpms Red Hat Enterprise Linux 9 AppStream
