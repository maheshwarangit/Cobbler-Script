# Cobbler-Script
1 Cobbler Host, 2 Homed
Cobbler Host will also works as DefaultGW (Nat and DNS-Forward included)

1 or more Host to deploy - automated Deployment  during first Boot

Software:
Centos 7
Cobbler v2.6.11
epel-release-lates-7 Repository
named.service (Bind)
dhcp.service (isc-dhcp)
NFS-Utils (CentOS ISO shared, using NFS)

Installation and Configuration in a few Steps (shortned, always root):

Install Software and Dependencies
Disable Services and seLinux
Costumization of named and dhcp template
Costumization of cobbler/settings
Enable Routing and NAT
Import Distro

Full Steps as followed:

yum -y update
rpm -Uvh https://dl.fedoraproject.org/pub/epel...
yum -y  update
yum -y  install bind bind-utils 
yum -y  install httpd.x86_64
yum -y  install dhcp
yum -y  install cobbler cobbler-web
systemctl stop named.service
systemctl stop http.service
systemctl stop dhcp.service
systemctl stop tftp.service
systemctl stop cobblerd.service
systemctl stop firewalld.service
systemctl disable named.service
systemctl disable http.service
systemctl disable dhcp.service
systemctl disable tftp.service
systemctl disable cobblerd.service
systemctl disable firewalld.service
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
init 6
sed -i 's/^manage_dhcp: 0/manage_dhcp: 1/g' /etc/cobbler/settings
sed -i 's/^manage_dns: 0/manage_dns: 1/g' /etc/cobbler/settings
sed -i 's/^restart_dhcp: 0/restart_dhcp: 1/g' /etc/cobbler/settings
sed -i 's/^restart_dns: 0/restart_dhcp: 1/g' /etc/cobbler/settings
sed -i 's/^bind_master: 127.0.0.1/bind_master: 192.168.100.1/g' /etc/cobbler/settings
sed -i 's/^server: 127.0.0.1/server: 192.168.100.1/g' /etc/cobbler/settings
sed -i 's/^next_server: 127.0.0.1/next_server: 192.168.100.1/g' /etc/cobbler/settings
systemctl enable cobblerd.service
systemctl start cobblerd.service
systemctl enable httpd.service
systemctl start httpd.service
cobbler check
cobbler get-loaders
yum -y install pykickstart
systemctl enable dhcpd.service
systemctl enable named.service
systemctl restart cobblerd.service
echo "net.ipv4.ip_forward = 1" /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
systemctl disable firewalld.service
yum -y install iptables-service
modprobe ip_conntrack_ftp
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
systemctl enable iptables
service iptables save
systemctl start  iptables.service
yum -y install  nfs-utils.x86_64
mkdir -p /mnt/cobbler
mount server01:/mnt/Volume_0/NFS/cobbler /mnt/cobbler
cobbler signature update
mkdir -p /mnt/iso
mount -o loop /mnt/cobbler/CentOS-7-x86_64-Minimal.iso /mnt/iso
cobbler import --path=/mnt/iso --name=Centos-7 --arch=x86_64
umount /mnt/iso
systemctl enable tftp.service
systemctl start tftp.service
cobbler sync
