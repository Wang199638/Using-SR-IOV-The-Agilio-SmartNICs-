# Using-SR-IOV-The-Agilio-SmartNICs
Record the operation

Can be combined with my previous operations： https://github.com/Wang199638/Passing-the-Netronome-Smartnic-to-a-KVM-vm-guest/blob/main/README.md

1. Follow the link: https://help.netronome.com/support/solutions/articles/36000049975-basic-firmware-user-guide#appendix-b-installing-the-out-of-tree-nfp-driver (Section **Using SR-IOV**)

2. After the finish the installation of vfs, use **ip link**, you will see the vfs and the mac address of the vfs:
   
```
   p4@testbed:~$ ip link
50: ens801np0np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:4d:13:39:a0 brd ff:ff:ff:ff:ff:ff
    vf 0 MAC 86:ef:3a:39:61:c6, spoof checking off, link-state auto, trust off, query_rss off
    vf 1 MAC d6:eb:33:39:d4:67, spoof checking off, link-state auto, trust off, query_rss off
51: ens801np1np1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:4d:13:39:a4 brd ff:ff:ff:ff:ff:ff
    vf 0 MAC 86:ef:3a:39:61:c6, spoof checking off, link-state auto, trust off, query_rss off
    vf 1 MAC d6:eb:33:39:d4:67, spoof checking off, link-state auto, trust off, query_rss off
 ```

3. you can also use **lspci -kd 19ee** to see the PCI number of the vfs, so here the **81:08.0**, and **81:08.1** are the pci for my vfs.

```
p4@testbed:~$ lspci -kd 19ee:
81:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
	Subsystem: Netronome Systems, Inc. Device 4000
	Kernel driver in use: nfp
	Kernel modules: nfp
81:08.0 Ethernet controller: Netronome Systems, Inc. Device 6003
	Subsystem: Netronome Systems, Inc. Device 4000
	Kernel driver in use: vfio-pci
	Kernel modules: nfp
81:08.1 Ethernet controller: Netronome Systems, Inc. Device 6003
	Subsystem: Netronome Systems, Inc. Device 4000
	Kernel driver in use: vfio-pci
	Kernel modules: nfp
83:00.0 Ethernet controller: Netronome Systems, Inc. Device 4000
	Subsystem: Netronome Systems, Inc. Device 0611
	Kernel driver in use: nfp
	Kernel modules: nfp
```

4. use **virsh nodedev-list | grep pci_0000_81** to see the pci.
```
p4@testbed:~$ virsh nodedev-list | grep pci_0000_81
pci_0000_81_00_0
pci_0000_81_08_0
pci_0000_81_08_1
```

5. let host to leave pci_0000_81_08_0 and pci_0000_81_08_1 alone.
```
p4@testbed:~$ sudo virsh nodedev-dettach pci_0000_81_08_0
Device pci_0000_81_08_0 detached
```

6. use **virsh edit xxxx** to edit your VM, change or add something like, **notice in the interface domain**, **the mac is the vf`s mac address**.
```
<interface type='hostdev' managed='yes'>
  <source>
    <address type='pci' domain='0x0000' bus='0x81' slot='0x08' function='0x0'/>
  </source>
  <mac address='86:ef:3a:39:61:c6'/>
</interface>
```

7. **Now you can virsh console your VM, and use ip link to see the interface.**
