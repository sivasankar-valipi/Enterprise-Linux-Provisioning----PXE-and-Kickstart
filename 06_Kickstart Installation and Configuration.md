# Kickstart Installation and Configuration

### Create kickstart

mkdir -p /var/www/html/kickstart

vi /var/www/html/kickstart/rhel01.ks

#version=RHEL9

# =========================

# System

# =========================

lang en_US.UTF-8

keyboard us

timezone Asia/Kolkata --utc

# =========================

# Network

# =========================

network --bootproto=dhcp --device=link --activate --hostname=rhel01.lab.local

# =========================

# Installation Source

# =========================

url --url="http://192.168.43.134/rhel9/"

# =========================

# Authentication

# =========================

rootpw --plaintext RedHat@123

user --name=redhat --groups=wheel --password=Ansible@123 --plaintext

# =========================

# Disk

# =========================

ignoredisk --only-use=nvme0n1

clearpart --all --initlabel --drives=nvme0n1

autopart --type=lvm

# =========================

# Bootloader

# =========================

bootloader --location=mbr --boot-drive=nvme0n1

# =========================

# Security

# =========================

selinux --enforcing

firewall --enabled --service=ssh

# =========================

# Services

# =========================

services --enabled="sshd,chronyd"

# =========================

# Packages

# =========================

%packages

@^minimal-environment

openssh-server

sudo

vim-enhanced

curl

wget

rsync

chrony

bash-completion

%end

# =========================

# Post Installation

# =========================

%post --log=/root/ks-post.log

echo "Provisioned by PXE + Kickstart" > /etc/motd

# Allow ansible user sudo access

echo "redhat ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/redhat

chmod 440 /etc/sudoers.d/redhat

%end

# =========================

# Reboot

# =========================

reboot

#### Check and Validate kickstart syntax

yum install pykickstart -y

ksvalidator /var/www/html/kickstart/rhel01.ks

#### Test HTTP service can serve the kickstart

curl -I http://192.168.43.134/kickstart/rhel01.ks

#### Now connect kickstart to PXE

Edit /var/lib/tftpboot/grub/grub.cfg file and add this kickstart URL in grub file

vi /etc/lib/tftpboot/grub/grub.cfg

At linuxefi line add this at end: inst.ks=http://192.168.43.134/kickstart/rhel01.ks

**The New VM should go from: EFI Network -> GRUB -> RHEL9 Installation -> Installation Complete -> Reboot -> rhel01.lab.local**

