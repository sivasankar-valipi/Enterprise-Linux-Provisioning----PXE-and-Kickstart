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

## PXE Boot Workflow

At this point, the PXE workflow begins automatically when the blank `rhel01` VM is powered on.

```text
┌──────────────────────────────┐
│       New Blank VM           │
│          rhel01              │
│     No OS / No ISO           │
└──────────────┬───────────────┘
               │
               ▼
       ┌─────────────────┐
       │ UEFI Network    │
       │     Boot        │
       └────────┬────────┘
                │
                │ DHCPDISCOVER
                ▼
       ┌─────────────────┐
       │      pxe01      │
       │  DHCP Server    │
       │ 192.168.43.134  │
       └────────┬────────┘
                │
                │ DHCPOFFER / DHCPACK
                │
                │ Client IP:
                │ 192.168.43.150
                │
                │ next-server:
                │ 192.168.43.134
                │
                │ boot file:
                │ grub/grubx64.efi
                ▼
       ┌─────────────────┐
       │      rhel01     │
       │  TFTP Request   │
       └────────┬────────┘
                │
                │ TFTP
                ▼
       ┌─────────────────┐
       │   grubx64.efi   │
       │     GRUB2       │
       └────────┬────────┘
                │
                ▼
             grub.cfg
                │
                ▼
       ┌─────────────────┐
       │     vmlinuz     │
       │    initrd.img   │
       └────────┬────────┘
                │
                ▼
       ┌─────────────────┐
       │     RHEL 9      │
       │     Anaconda    │
       │    Installer    │
       └────────┬────────┘
                │
        ┌───────┴────────┐
        │                │
    inst.repo          inst.ks
        │                │
        ▼                ▼
┌───────────────┐  ┌────────────────┐
│ HTTP RHEL     │  │ HTTP Kickstart │
│ Repository    │  │   rhel01.ks    │
│               │  │                │
│ BaseOS        │  │ Installation   │
│ AppStream     │  │ Instructions   │
└───────┬───────┘  └───────┬────────┘
        │                  │
        └────────┬─────────┘
                 ▼
        ┌─────────────────┐
        │ Automated RHEL  │
        │   Installation  │
        └────────┬────────┘
                 │
                 ▼
              Reboot
                 │
                 ▼
        ┌─────────────────┐
        │     rhel01      │
        │ rhel01.lab.local│
        │                 │
        │ RHEL 9 Installed│
        │ SSH Available   │
        └─────────────────┘
```

### Workflow Explanation

1. **UEFI Network Boot** — The blank VM starts from the network.
2. **DHCP** — `pxe01` assigns an IP address and provides the PXE boot server and boot file.
3. **TFTP** — The client downloads `grubx64.efi`.
4. **GRUB2** — GRUB loads `vmlinuz` and `initrd.img`.
5. **Anaconda** — The RHEL installer starts.
6. **`inst.repo`** — Anaconda retrieves the RHEL installation packages from the HTTP repository.
7. **`inst.ks`** — Anaconda retrieves the Kickstart file containing the automated installation instructions.
8. **Automated Installation** — Kickstart configures the disk, network, packages, users, services, and security settings.
9. **Reboot** — The server boots from its newly installed NVMe disk.
10. **RHEL Server Ready** — `rhel01.lab.local` is available through SSH and ready for Ansible onboarding.
