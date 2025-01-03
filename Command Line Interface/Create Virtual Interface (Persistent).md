Module purpose

|**Module**|**Description**|**Use Case**|
|---|---|---|
|**dummy**|Creates virtual NICs|Testing and mock networks|
|**8021q**|VLAN tagging (IEEE 802.1Q)|VLAN management|
|**macvlan**|Virtual MAC NICs|Container networking|
|**bridge**|Bridging support|Virtual network switches|
|**bonding**|NIC teaming and bonding|Redundancy and failover|

Check module info
```bash
sudo modinfo dummy      # Dummy interface
sudo modinfo 8021q      # VLAN support
sudo modinfo macvlan    # MACVLAN support
sudo modinfo bridge     # Bridging support
sudo modinfo bonding    # Interface bonding
```

Load the kernel module
```bash
# Load networking modules
sudo modprobe dummy      # Dummy interface
sudo modprobe 8021q      # VLAN support
sudo modprobe macvlan    # MACVLAN support
sudo modprobe bridge     # Bridging support
sudo modprobe bonding    # Interface bonding
```

Check if module is loaded
```bash
lsmod | grep dummy
lsmod | grep 8021q
lsmod | grep macvlan
lsmod | grep bridge
lsmod | grep bonding
```

Load module at boot (persistent)
```bash
echo "dummy"   | sudo tee -a /etc/modules-load.d/dummy.conf
echo "8021q"   | sudo tee -a /etc/modules-load.d/8021q.conf
echo "macvlan" | sudo tee -a /etc/modules-load.d/macvlan.conf
echo "bridge"  | sudo tee -a /etc/modules-load.d/bridge.conf
echo "boding"  | sudo tee -a /etc/modules-load.d/bonding.conf

or

cat <<EOF | sudo tee /etc/modules-load.d/custom-interface.conf
dummy
8021q
macvlan
bridge
boding
EOF 
```

Check module error
```bash
dmesg | grep dummy
dmesg | grep 8021q
dmesg | grep macvlan
dmesg | grep bridge
dmesg | grep bonding
```