## The step-by-step procedure for installing Red Hat Enterprise Linux 9.0 on virtual machines that will serve as Pxe01 server. 

**Step 1**

Power on the virtual machine.

The system boots from the ISO.

You'll see the Red Hat boot menu.

**Step 2**

Install Red Hat Enterprise Linux 9.0

or

Test this media and install Red Hat Enterprise Linux

Which option should we choose?

Choose

Install Red Hat Enterprise Linux

Why?

The "Test this media" option verifies that the ISO image is not corrupted.

Since we downloaded the ISO directly from the Red Hat website, we can safely skip the media test to reduce installation time.

**Step 3**

Language Selection

Choose --> English

**Step 4**

Installation Destination

You'll see, VMware Virtual NVMe Disk 20 GB

Select the disk (Double click on it).

Choose --> Automatic Storage Configuration

Why Automatic?

The installer automatically creates the required partitions:

/boot,
/,
Swap (if needed)

For a lab setup, automatic partitioning is simple and sufficient.

**Step 5**

Open --> Software Selection

Choose --> Minimal Install

Why Minimal Install?

A Red Hat cluster node does not require a graphical desktop environment. Installing only the minimal package set provides several advantages:

Reduced disk usage
Lower memory consumption
Smaller attack surface
Faster boot times
Fewer packages to update and maintain

This is the recommended installation type for production cluster nodes.

**Step 6**

Click --> Network & Hostname

Enable the network interface by toggling it ON.

Set the hostname.

For Node1:

pxe01

Click --> Apply

**Step 7**

User Settings

Configure the root password.

Create a standard user if required.

**Step 8**

Click --> Begin Installation

Wait for the installation to complete.

Once finished,

Click --> Reboot System
