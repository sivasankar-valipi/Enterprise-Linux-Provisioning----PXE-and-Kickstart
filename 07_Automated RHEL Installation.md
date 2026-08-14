# Create a New PXE Client VM in VMware Workstation Pro

The purpose of this project is to create a completely blank VM with no RHEL ISO attached. 
The VM will obtain RHEL entirely from the PXE infrastructure.

**1.** Start the New Virtual Machine

Open VMware Workstation Pro and select:

File
  ↓
New Virtual Machine

Choose:

Typical (recommended)

Click Next.

**2.** Do not provide an installer ISO

On the installer source screen, select:

I will install the operating system later

Do not select your RHEL ISO because we want:

**3.** Select the guest operating system

Select: Guest operating system:

Linux

Version:

Red Hat Enterprise Linux 9 64-bit

Click Next.

**4.** Configure VM name

Virtual machine name:rhel01

Choose the VM storage location and click Next.

**5.** Create the virtual disk

Disk size: 20 GB

**6.** Customize the hardware

Before finishing, select:

Customize Hardware

Configure approximately:

Memory:             2–4 GB
Processors:         2 vCPU
Hard Disk:          20 GB
Network Adapter:    PXE Host-Only
CD/DVD:             No ISO

PXE server has:

pxe01

ens224

192.168.43.134
       │
       ▼
192.168.43.0/24 PXE Network

**8.** Verify CD/DVD

Make sure there is no RHEL ISO connected.

**9.** Start the VM

Now power on: rhel01

The VM should attempt:

EFI Network...

At this point, the PXE workflow begins automatically:

New Blank VM
     │
     ▼
EFI Network Boot
     │
     │ DHCPDISCOVER
     ▼
pxe01 DHCP
     │
     │ IP address
     │ next-server
     │ grubx64.efi
     ▼
rhel01
     │
     │ TFTP
     ▼
grubx64.efi
     │
     ▼
GRUB Menu
     │
     ▼
vmlinuz + initrd.img
     │
     ▼
RHEL 9 Anaconda
     │
     │ inst.repo
     ├──────────────► HTTP RHEL Repository
     │
     │ inst.ks
     └──────────────► Kickstart
                           │
                           ▼
                    Automatic Install
                           │
                           ▼
                         Reboot
                           │
                           ▼
                    rhel01.lab.local
