# VMware Workstation Setup

VMware Workstation is a desktop virtualization software that allows multiple operating systems to run simultaneously on a single physical computer.
It provides an isolated environment for testing, development, and learning without affecting the host operating system.

In this project, VMware Workstation Pro 26H1 is used to build a PXE and Kickstart for Linux Server Provisioning.

---

# Lab Requirements

| Component | Requirement |
|-----------|-------------|
| Host OS | Windows 10/11 (64-bit) |
| Hypervisor | VMware Workstation Pro |
| Guest OS | Red Hat Enterprise Linux |
| CPU | Intel VT-x / AMD-V Supported |
| RAM | Minimum 12 GB (16 GB Recommended) |
| Disk Space | 80 GB Free |

---

# ***Downloading VMware Workstation Pro***

Before downloading the software, you need to register for a free Broadcom account.

### Step 1: Create a Broadcom Account

- Visit **https://support.broadcom.com**
- Click **Register**.
- Complete the registration form with the required details.
- Verify your email address and activate your account.

### Step 2: Sign In

- Log in to the Broadcom Support Portal(https://support.broadcom.com) using your registered email address and password.

### Step 3: Navigate to Downloads

From the home page, navigate to:

```
My Downloads
    └── Free Software Downloads
```

### Step 4: Search for VMware Workstation Pro

- Search for **VMware Workstation Pro**.
- Select the required version.
- Verify your account details if prompted.

### Step 5: Download

- Accept the license agreement.
- Click **Download**.
- Save the installer to your local machine.

---

# ***Downloading Red Hat Enterprise Linux 9.x (RHEL) DVD ISO***

Red Hat Enterprise Linux (RHEL) is the operating system used throughout this lab. A free **Red Hat Developer Subscription for Individuals** allows you to download and use RHEL for learning and development purposes.

### Step 1: Create a Red Hat Account

- Visit the Red Hat Developer website (https://developers.redhat.com/blog/2021/02/10/how-to-activate-your-no-cost-red-hat-enterprise-linux-subscription?utm_source=chatgpt.com).
- Create a free Red Hat account.

### Step 2: Activate the Developer Subscription

- Sign up for the **Red Hat Developer Subscription for Individuals**.
- This subscription provides free access to Red Hat Enterprise Linux for development and lab environments.

### Step 3: Download the ISO

- Sign in to the Red Hat Developer Portal.
- Navigate to the **RHEL Downloads** page.
- Select the **Red Hat Enterprise Linux 9.x** release.
- Choose the **x86_64 DVD ISO** image (https://developers.redhat.com/products/rhel/download?utm_source=chatgpt.com).

### Step 4: Save the ISO

- Download the ISO file to your local machine.
- This ISO will be used to create the virtual machines in VMware Workstation.

---

**In this project, we are using Red Hat Enterprise Linux 9.0**

## Next Step

After downloading VMware Workstation Pro and the RHEL DVD ISO, proceed to **VMware Workstation Installation** to create the virtual machines for PXE and Kickstart.
