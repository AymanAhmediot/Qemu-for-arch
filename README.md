# Qemu-for-arch

## QEMU/KVM Dependancies to install:

``sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat ebtables iptables libguestfs --needed``

## In file add `/etc/libvirt/libvirtd.conf`

``
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
``
## Then add your user and create group:

``
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
``

### After adding your user reboot the system.

## Enable libvirtd 

``
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
``
## Enable virt network interface to use the default interface and auto start it 

``
sudo virsh net-start default
sudo virsh net-autostart default
``

# Now you are good to go 

## Troubleshooting 

### Check if IP Forwarding Enabled on Host by 

``
cat /proc/sys/net/ipv4/ip_forward
``

If it 1 that means it enable if else enable it be that command 

``
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
``

###  No Internet in KVM/QEMU VM (libvirt NAT) 

Check what is default network interface name

``
ip route | grep default
``

Make a new network for vm and add it the virt network interface  

``
sudo iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -o enp3s0 -j MASQUERADE
``

Now do not forget allow forwarding in iptables

``
sudo iptables -A FORWARD -i virbr0 -j ACCEPT
sudo iptables -A FORWARD -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
``
Reset virt network interface 

``
sudo virsh net-destroy default
sudo virsh net-start default
``
