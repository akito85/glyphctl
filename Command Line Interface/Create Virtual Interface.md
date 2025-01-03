**The generic format to create interface**
```bash
$ ip link add link <physical_interface> name <virtual_interface_name> type <type>
```

## Creating Interface Type 'dummy'

The basic syntax is as follows:

```bash
$ sudo ip link add <interface_name> type dummy
```

**Step 1: Create a virtual interface**

```shell
$ sudo ip link add kube0 type dummy
```

**Step 2: Set a custom MAC and IP address**

```shell
$ sudo ip link set address 00:11:22:33:44:55 dev kube0
$ sudo ip addr add 192.168.100.150/24 dev kube0
```

**Step 3: Enable the interface**

```shell
$ sudo ip link set dev kube0 up
```

Your virtual interface is now ready to use. You can verify its status using the `ip link show` command.

___

## Creating Interface Type 'macvlan'

The basic syntax is as follows:

```bash
$ ip link add link <physical_interface> type macvlan
```

**Step 1: Create virtual interface on physical eth0**

```bash
$ ip link add link eth0 type macvlan
```

**Step 2: Set custom MAC address**

```bash
$ ip link set address 00:11:22:33:44:55 dev macvlan0
```

**Step 3: Enable the interface**

```bash
ip link set dev macvlan0 up
```

Your macvlan interface is now ready to use. You can verify its status using the `ip link show` command.
___

## Creating Interface Type 'vlan'

The basic syntax is as follows:

```bash
$ sudo ip link add <interface_name> name <interface_name>.<id> type vlan id <id>
```

**Step 1: Create virtual interface on physical enp0s3**

```bash
$ sudo ip link add link enp0s3 name enp0s3.100 type vlan id 100
```

**Step 2: Set custom IP address**

```bash
$ sudo ip addr add 192.168.0.200/24 dev enp0s3.100
```

**Step 3: Enable the interface**

```bash
$ sudo ip link set up enp0s3.100
```

Your vlan interface is now ready to use. You can verify its status using the `ip link show` command.
___

## Creating Interface Type 'bridge'

The basic syntax is as follows:

```bash
$ sudo brctl addbr <bridge_name>
```

**Step 1: Create virtual interface bridge**

```bash
$ sudo brctl addbr br0
```

**Step 2: Add bridge to vlan**

```bash
$ sudo brctl addif br0 enp0s3.100
```

**Step 3: Set custom IP address to bridge**

```bash
$ sudo ip addr add 192.168.0.1/24 dev br0
```

___

## Creating Interface Type 'bond'

Bond modes overview:

|**Mode**|**Description**|**Use Case**|
|--|--|--|
|`balance-rr`|Round-robin load balancing|High throughput|
|`active-backup`|Failover, only one active|Redundancy (default)|
|`balance-xor`|Transmit based on hashing|Performance & failover|
|`802.3ad`|LACP (Link Aggregation)|Switch support required|
|`broadcast`|Sends on all interfaces|High availability|
|`balance-tlb`|Adaptive transmit load|NIC driver-specific|
|`balance-alb`|Adaptive load balancing|NIC driver-specific|

The bond module must be loaded:
```bash
$ sudo modprobe bonding
$ echo "bonding" | sudo tee -a /etc/modulesecho "bonding" | sudo tee -a /etc/modules
```

The basic syntax is as follows:
```bash
$ ip link add <bond_interface_name> type bond
```

Create bond interface:
```bash
$ sudo ip link add bond0 type bond
```

Configure bonding mode and options:
```bash
$ echo "+eth0" | sudo tee /sys/class/net/bond0/bonding/slaves
$ echo "+eth1" | sudo tee /sys/class/net/bond0/bonding/slaves
$ echo "active-backup" | sudo tee /sys/class/net/bond0/bonding/mode
$ echo "100" | sudo tee /sys/class/net/bond0/bonding/miimon
```

Bring up interface and bond
```bash
$ sudo ip link set eth0 down
$ sudo ip link set eth1 down
$ sudo ip link set eth0 master bond0
$ sudo ip link set eth1 master bond0
$ sudo ip link set bond0 up
```

Assign Mac and IP address
```bash
$ sudo ip link set address 00:11:22:33:44:55 dev bond0
$ sudo ip addr add 192.168.100.150/24 dev bond0 sudo ip route add default via 192.168.100.1
```

Check bond status
```bash
$ cat /proc/net/bonding/bond0
```