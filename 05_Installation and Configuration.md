# DHCP, TFTP and PXE Installation and Configuration

## Install and Configure DHCP Server

yum install dhcp-server -y

verify: rpm -qa | grep dhcp

Backup DHCP Configuration file:

cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak

Edit Config file:

vi /etc/dhcp/dhcpd.conf

authoritative;

default-lease-time 600;
max-lease-time 7200;

subnet 192.168.43.0 netmask 255.255.255.0 {

    range 192.168.43.150 192.168.43.200;

    option subnet-mask 255.255.255.0;

    next-server 192.168.43.134;
    filename "grub/grubx64.efi";
}

What does next-server mean?

next-server 192.168.43.134;

The PXE client should contact 192.168.43.134 to obtain its network boot file.

Since 192.168.43.134 is pxe01, the PXE server provides the boot files.

It does not mean that next server provides the RHEL installation itself.

Our PXE network is:

192.168.43.0/24

PXE server:

192.168.43.134

DHCP client range:

192.168.43.150 - 192.168.43.200

                    PXE Network
                 192.168.43.0/24
                         │
                         │
                  ┌──────▼──────┐
                  │    pxe01    │
                   192.168.43.134  
                  │             │
                  │ DHCP Server │
                  └──────┬──────┘
                         │
                DHCP leases
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           rhel01      rhel02     rhel03
     192.168.43.150 192.168.43.151 192.168.43.152

Start and Enable the Service:

systemctl start dhcpd

systemctl enable dhcpd --now

Configuration test:

dhcpd -t /etc/dhcp/dhcpd.conf

## Install and Configure TFTP

yum install tftp-server -y

Location: /var/lib/tftpboot - Server

          /usr/bin/tftp - Client
          
Start and Enable the Service:

systemctl start tftp

systemctl enable tftp --now
          
#### Prepare TFTP Directory

mkdir -p /var/lib/tftpboot/grub

cp /mnt/rhel9/EFI/BOOT/grub*64.efi /var/lib/tftpboot/grub

cp /mnt/rhel9/images/pxeboot/vmlinuz /var/lib/tftpboot/

cp /mnt/rhel9/images/pxeboot/initrd.img /var/lib/tftpboot/

#### Create GRUB Config

vi /var/lib/tftpboot/grub/grub.cfg

set timeout=10
set default=0

menuentry 'Install RHEL 9' {
    linuxefi /vmlinuz inst.repo=http://192.168.43.134/rhel9/
    initrdefi /initrd.img
}

## Install and Configure HTTP Service

yum install httpd -y

systemctl start httpd

systemctl enable httpd -y

Copy Boot files to HTTP repository

mkdir -p /var/www/html/rhel9

cp -av /mnt/rhel9/. /var/www/html/rhel9

du -sh /var/www/html/rhel9

ls -lh /var/www/html/rhel9

**Note: /mnt/rhel9 is mount point for OS RHEL9.0**

systemctl restart httpd

Test and Verify:

curl -I http://192.168.43.134/rhel9/

#### Configure Firewall

firewall-cmd --permanent --add-service=dhcp

firewall-cmd --permanent --add-service=tftp

firewall-cmd --permanent --add-service=http

firewall-cmd --reload

firewall-cmd --list-services
