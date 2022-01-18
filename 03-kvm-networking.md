# Part 3: Networking

<br>

## 1. The “default” network

When **libvirt** is in use and the **libvirtd** daemon is running, a default network is created. We can verify that this network exists by using the `virsh` utility, which on the majority of Linux distribution usually comes with the `libvirt-client` package. To invoke the utility so that it displays all the available virtual networks, we should include the `net-list` subcommand:
- `sudo virsh net-list --all`

Example

```bash
# sudo virsh net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   active     yes         yes

#
```

To obtain detailed information about the network, and eventually modify it, we can invoke virsh with the `edit` subcommand instead, providing the network name as argument:

- `sudo virsh net-edit default`

A temporary file containing the **xml** network definition will be opened in our favorite text editor. In this case the result is the following::

```bash
<network>
  <name>default</name>
  <uuid>6115b620-438d-44ad-9215-ce3ca396a890</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:d6:d9:21'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

As we can see, the default network is based on the use of the `virbr0` virtual bridge, and uses **NAT** based connectivity to connect the virtual machines which are part of the network to the outside world. We can verify that the bridge exists using the `ip` command:
- `ip link show type bridge`

In our case the command above returns the following output:

```bash
# ip link show type bridge
4: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:d6:d9:21 brd ff:ff:ff:ff:ff:ff
#
```

Alternative:

```bash
# brctl show  
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400d6d921	yes
#
```

To show the interfaces which are part of the bridge, we can use the `ip` command and query only for interfaces which have the `virbr0` bridge as master:

- `ip link show master virbr0`

The result of running the command is:

```bash
# ip link show master virbr0
7: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master virbr0 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:61:d5:93 brd ff:ff:ff:ff:ff:ff
#
```

As we can see, there is only one interface currently attached to the bridge, `vnet0`. The `vnet0` interface is a virtual ethernet interface: it is created and added to the bridge, and its purpose is to provide a **MAC** address for the bridge.

Other virtual interfaces will be added to the bridge when we create and launch virtual machines. For the sake of this tutorial I created and launched a new virtual machine; if we re-launch the command we used above to display the bridge slave interfaces, we can see a new one was added, `vnet1`:

```bash
# ip link show master virbr0
5: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master virbr0 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:4c:6c:6c brd ff:ff:ff:ff:ff:ff
6: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master virbr0 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:54:00:38:22:fe brd ff:ff:ff:ff:ff:ff
#
```

No physical interfaces should ever be added to the `virbr0` bridge, since it uses **NAT** to provide connectivity.

<br>

## 2. Use bridged networking for virtual machines

The default network provides a very straightforward way to achieve connectivity when creating virtual machines: everything is “ready” and works out of the box. Sometimes, however, we want to achieve a **full bridging** connection, where the guest devices are connected to the host **LAN**, without using **NAT**, we should create a new bridge and share one of the host physical ethernet interfaces. Let’s see how to do this step by step.

### Creating a new bridge

To create a new bridge, we can still use the `ip` command. Let’s say we want to name this bridge `br0`; we would run the following command:

- `sudo ip link add br0 type bridge`

To verify the bridge is created we do as before:

```bash
# sudo ip link show type bridge
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:d6:d9:21 brd ff:ff:ff:ff:ff:ff
8: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:f6:f5:c7:16:96 brd ff:ff:ff:ff:ff:ff
#
```

As expected, the new bridge, `br0` was created and is now included in the output of the command above. Now that the new bridge is created, we can proceed and add the physical interface to it.

### Adding a physical ethernet interface to the bridge

In this step we will add a host physical interface to the bridge. Notice that you can’t use your main ethernet interface in this case, since as soon as it is added to the bridge you would loose connectivity, since it will loose its IP address. In this case we will use an additional interface, `enp0s29u1u1`: this is an interface provided by an ethernet to usb adapter attached to my machine.

First we make sure the interface state is UP:
- `sudo ip link set enp0s29u1u1 up`

To add the interface to bridge, the command to run is the following:
- `sudo ip link set enp0s29u1u1 master br0`

To verify the interface was added to the bridge:
- `sudo ip link show master br0`

Example:
```
$ sudo ip link show master br0
3: enp0s29u1u1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br0 state UP mode DEFAULT group default qlen 1000
    link/ether 18:a6:f7:0e:06:64 brd ff:ff:ff:ff:ff:ff

```

### Assigning a static IP address to the bridge

At this point we can assign a static IP address to the bridge. Let’s say we want to use `192.168.0.90/24`; we would run:
- `sudo ip address add dev br0 192.168.0.90/24`

<br>

## 3. Creating a new virtual network

At this point we should define a new “network” to be used by our virtual machines. We open a file with our favorite editor and paste the following content inside of it, than save it as `bridged-network.xml`:

```bash
<network>
    <name>bridged-network</name>
    <forward mode="bridge" />
    <bridge name="br0" />
</network>
```

Once the file is ready we pass its position as argument to the `net-define` `virsh` subcommand:
- `sudo virsh net-define bridged-network.xml`

To activate the new network and make so that it is auto-started, we should run:
- `sudo virsh net-start bridged-network`
- `sudo virsh net-autostart bridged-network`

We can verify the network has been activated by running the `virsh net-list`command, again:

```
$ sudo virsh net-list
Name              State    Autostart   Persistent
----------------------------------------------------
bridged-network   active   yes         yes
default           active   yes         yes

```

We can now select the network by name when using the `--network` option:

```
$ sudo virt-install \
  --vcpus=1 \
  --memory=1024 \
  --cdrom=debian-10.8.0-amd64-DVD-1.iso \
  --disk size=7 \
  --os-variant=debian10 \
  --network network=bridged-network

```

Create a network10 network configuration file, using NAT:

```
<network>
  <name>net10</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='br10' stp='on' delay='0'/>
  <ip address='192.168.30.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.30.50' end='192.168.30.200'/>
    </dhcp>  </ip></network>
```

Define and start networks
- `sudo virsh net-define network10.xml`
- `sudo virsh net-start network10`
- `sudo virsh net-autostart network10`

Check network is created and active:
```
# sudo virsh net-list --all
[sudo] password for jmb: 
 Name      State      Autostart   Persistent
----------------------------------------------
 net10     inactive   no          yes
 default   active     yes         yes

#
```

In the offchance this lists nothing, define it and set it to autostart:
- `virsh net-autostart default`

Then try starting it up:
- `sudo virsh net-start default`

VM Interface association for my Guest machine:
```bash
# sudo virsh domiflist cedge
 Interface   Type      Source    Model    MAC
-------------------------------------------------------------
 vnet2       network   default   virtio   52:54:00:61:d5:93

#
```

<br>

## 4. More information

[How to use bridged networking with libvirt and kvm](https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm)

[Port forwarding](https://wiki.libvirt.org/page/Networking#Forwarding_Incoming_Connections)

[Change video adapter to virtio](https://gist.github.com/kalyco/afa1e18cd8c6c4ce1cab2ce6f8d24e49)