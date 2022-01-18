# README

Launch Cat8000v
```bash
virt-install \
    --name=cedge \
    --os-type=linux \
    --os-variant=rhel4.0 \
    --arch=x86_64 \
    --cpu host \
    --vcpus=2 \
    --hvm \
    --ram=4096 \
    --disk path=/home/jmb/virt-manager/c8000v.qcow2,size=16,device=disk,bus=ide,format=qcow2 \
    --disk path=config.iso,device=cdrom \
    --network=network:default,model=virtio \
    --graphics none \
    --import

```

VMs:
- `sudo virsh domiflist cedge` - VM Interface association (network)
- `virsh --connect qemu:///system console cedge` - Connect to console


Bridge:
- `ip link show type bridge` - Show bridge
- `brctl show ` - show bridge (alternative)
- `ip link show master virbr0` - interfaces that belongs to bridge
- `sudo ip link add br0 type bridge` - Create a new bridge
- `sudo ip link set enp0s29u1u1 up` - enable interface
- `sudo ip link set enp0s29u1u1 master br0` - Add interface to bridge
- `sudo ip address add dev br0 192.168.0.90/24` - Assigning a static IP address to the bridge


Networks:
- `sudo virsh net-list --all`
- `sudo virsh net-edit default`

Create file `network.xml`:
```bash
<network>
    <name>network1</name>
    <forward mode="bridge" />
    <bridge name="br0" />
</network>
```

- `sudo virsh net-define network.xml` - Add new network to bridge
- `sudo virsh net-start network` - To start a (previously defined) inactive network,
- `sudo virsh net-autostart network` - set network to autostart at service start,


C8000v
- `show platform software cpu alloc` - Gives CPU allocation
- `show platform software vnic-if interface-mapping` - Gives interface mapping
-
-

