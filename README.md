# KVM-Virtualization-Configuration

# Description:
Configure Kernel-based Virtual Machine (KVM) virtualization on a CentOS server to create and manage virtualized environments. Define virtual machine settings, network configurations, and storage allocations to optimize performance and resource utilization.

Here's a step-by-step guide to achieve your KVM virtualization setup on a CentOS server:

# 1. Install KVM and Related Packages
Update your system:

sudo yum update -y
[root@dlp ~]# dnf -y install qemu-kvm libvirt virt-install

sudo dnf install libvirt virt-manager

<img width="428" alt="virt" src="https://github.com/user-attachments/assets/07907aa7-12fb-4ec1-af5a-a357cbf12317">


<img width="426" alt="dsaas" src="https://github.com/user-attachments/assets/6f8cfc3e-873f-4d7f-aebb-ea4d78930603">

# Granting regular (non-root) user access to KVM
The following allows a non-root user access to manage KVM guests

For systems with a libvirt group
This works on at least Ubuntu 19.10, and CentOS 8. Not been tested on other systems.

<img width="409" alt="y6" src="https://github.com/user-attachments/assets/8440ab10-4dee-49e6-bc6b-51f7140433c9">


sudo usermod -aG libvirt $USER


<img width="419" alt="3213" src="https://github.com/user-attachments/assets/3e7cbab4-91f3-45e6-8516-919c959fbb7f">


# confirm modules are loaded
[root@dlp ~]# lsmod | grep kvm
kvm_intel             348160  0
kvm                  1056768  1 kvm_intel
irqbypass              16384  1 kvm

<img width="468" alt="virt" src="https://github.com/user-attachments/assets/aeb0497b-a1ab-45ef-88a2-340ac4744025">



[root@dlp ~]# systemctl enable --now libvirtd

<img width="431" alt="ddddd" src="https://github.com/user-attachments/assets/bec4e04e-8663-4f7b-9750-7cf218f3f1ec">


# 2. Verify Hardware Virtualization Support
Check for virtualization extensions:

<img width="417" alt="dww" src="https://github.com/user-attachments/assets/5a5e225a-19cc-4d19-8b14-5d22d4a35b64">


lscpu | grep -E 'Virtualization|VT-x|AMD-V'
Look for lines indicating VT-x or AMD-V. If they are present, virtualization is supported.

Ensure virtualization is enabled in the BIOS. This typically involves rebooting the server and checking BIOS settings for virtualization options.

# 3. Create a Storage Pool
Define a storage pool using virsh:

sudo virsh pool-define-as --name default --type dir --target /var/lib/libvirt/images
Start and autostart the storage pool:

sudo virsh pool-start default
sudo virsh pool-autostart default

<img width="436" alt="vss" src="https://github.com/user-attachments/assets/6689a531-f5bb-4a34-8be8-4b33e82dc209">


<img width="320" alt="65i" src="https://github.com/user-attachments/assets/2fff472f-c4c6-49f2-bc5e-0b8c3c74b1d5">


# 4. Configure Network Settings
Create a bridge network:

Edit the network configuration file, typically located at /etc/sysconfig/network-scripts/ifcfg-br0. Example content:

DEVICE=br0
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=dhcp
Adjust settings based on your network configuration.

<img width="155" alt="324" src="https://github.com/user-attachments/assets/44c1eeb1-6235-4316-8585-afe63ea2aa47">

Modify the Ethernet interface to use the bridge:

# add bridge [br0]
[root@dlp ~]# nmcli connection add type bridge autoconnect yes con-name br0 ifname br0
Connection 'br0' (ef307e3b-454f-46bc-a9a8-e2fdc93a2199) successfully added.
# set IP address for [br0]
[root@dlp ~]# nmcli connection modify br0 ipv4.addresses 10.0.0.30/24 ipv4.method manual

<img width="450" alt="3211" src="https://github.com/user-attachments/assets/ac8b3578-f564-43f6-ab97-28cd7ec9129a">

Edit the Ethernet interface configuration file (e.g., /etc/sysconfig/network-scripts/ifcfg-eth0):

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
MASTER=br0
SLAVE=yes

<img width="89" alt="vd" src="https://github.com/user-attachments/assets/b95cc9ee-e715-4713-afc8-a430203664e7">


Restart network services:

sudo systemctl restart network
Configure firewall rules (if needed):

sudo firewall-cmd --add-service=libvirt --permanent
sudo firewall-cmd --reload

<img width="388" alt="ll" src="https://github.com/user-attachments/assets/f668ff32-7cae-4e40-8896-bb58304fb9b2">


# 5. Create Virtual Machine Instances

Use virt-manager or virt-install to create a VM:

sudo virt-install \
  --name myvm \
  --ram 2048 \
  --disk path=/var/lib/libvirt/images/myvm.qcow2,size=10 \
  --vcpus 2 \
  --os-type linux \
  --os-variant centos7.0 \
  --network bridge=br0 \
  --graphics none \
  --console pty,target_type=serial \
  --location 'http://mirror.centos.org/centos/7/os/x86_64/' \
  --extra-args 'console=ttyS0,115200n8 serial'
<img width="477" alt="downl" src="https://github.com/user-attachments/assets/c1002113-6c73-4905-b019-03c8747a5e3a">


<img width="447" alt="dw1" src="https://github.com/user-attachments/assets/c686b826-1fb2-4e24-8ed0-7be7451d98f8">



<img width="365" alt="lok" src="https://github.com/user-attachments/assets/f9b3370a-5940-46b3-8a95-75dfeabb9a4f">


  <img width="402" alt="42" src="https://github.com/user-attachments/assets/f76e2672-74cd-4a50-85ab-e283e5997c69">


<img width="292" alt="sca" src="https://github.com/user-attachments/assets/42ceeca8-a9c4-45db-89c3-35aae7be8e70">

# 6. Customize Virtual Machine Settings
Edit VM settings with virt-manager or virsh:

sudo virsh edit myvm
Adjust settings for CPU, memory, and other parameters as needed.
<img width="353" alt="das" src="https://github.com/user-attachments/assets/e6570ee1-5bc3-4a7f-b62a-9dabf095cb99">


# 7. Attach Storage Volumes
Create and attach additional storage volumes:

sudo virsh vol-create-as default myvolume.qcow2 20G --format qcow2

<img width="360" alt="ds" src="https://github.com/user-attachments/assets/b945560e-e9e8-4bb9-a50a-3b4203f48d67">

sudo virsh attach-disk myvm /var/lib/libvirt/images/myvolume.qcow2 vdb --targetbus virtio

<img width="365" alt="csaa" src="https://github.com/user-attachments/assets/c832bff1-8666-47df-a155-6e6320ca35fe">


# 8. Enable Additional Features
Live migration:

sudo virsh migrate --live myvm qemu+ssh://destination-server/system
Create a snapshot:

<img width="362" alt="651" src="https://github.com/user-attachments/assets/ad618841-f1be-4bfa-a2a2-f8008049d181">

sudo virsh snapshot-create-as myvm snapshot1 "Snapshot description"

<img width="364" alt="ewq" src="https://github.com/user-attachments/assets/c016ebd0-b9d5-4818-9637-82099d54f9fb">


CPU pinning and memory ballooning can be configured in the VM XML configuration:

sudo virsh edit myvm

<img width="298" alt="qweq" src="https://github.com/user-attachments/assets/8c8c291e-d773-42b2-bf4b-5b3a60997edf">


# 9. Test KVM Virtualization Setup
Deploy and test VMs: Deploy VMs, run workloads, and monitor their performance using tools like top, htop, or virt-top.


<img width="362" alt="41" src="https://github.com/user-attachments/assets/c4edc484-dfc4-4a2b-986d-c49af025e5a8">

10. Implement Security Best Practices
Configure SELinux policies: Ensure SELinux is enforcing and configured for virtualization:

sudo setenforce 1
Review and configure firewall rules:

sudo firewall-cmd --list-all

<img width="359" alt="532" src="https://github.com/user-attachments/assets/7eeaebae-bd99-4a73-9159-687bc18f18d6">


Ensure VM isolation and security best practices:

Isolate VMs from each other where necessary.
Use secure communication channels for management tasks.

# KVM Virtualization Summary
Installation and Configuration:

Install KVM and related packages on CentOS, including virtualization tools and libraries.
Start and enable the libvirtd service to manage virtual machines.
Virtual Machine Creation and Configuration:

Use tools like virt-install or virt-manager to create virtual machines.
Configure VM settings such as CPU allocation, memory size, disk storage, and networking.
Network Configuration:

Set up a bridge interface to allow VMs to communicate with external networks.
Adjust firewall rules to ensure proper network access for virtual machines.
Storage Management:

Define and manage storage pools for VM disk images.
Allocate and manage storage volumes within the pools to ensure adequate capacity and performance.
Advanced Features:

Enable and configure features such as live migration to move VMs between hosts without downtime.
Set up snapshot management to capture the state of VMs for backup or testing.
Implement resource allocation techniques like CPU pinning and memory ballooning to optimize performance.
Security Measures:

Apply SELinux policies to enforce security contexts and protect the virtual environment.
Configure firewall rules to restrict access to only necessary services.
Ensure VM isolation to prevent unauthorized access and maintain security within the virtualized environment.
This summary covers the key aspects of setting up and managing KVM virtualization, including installation, VM configuration, network and storage management, advanced features, and security practices.




