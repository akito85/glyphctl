### **Step 1: Load Module Interface on Kernel**

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

---

### **Step 2: Ensure Interface is Created and Available at Boot**

Create a udev rule to ensure the `kube0` dummy interface is created and available at boot.

#### **Create Udev Rule File**

```bash
sudo nano /etc/udev/rules.d/99-dummy-kube0.rules
```

#### **Add the Following Rule**

```bash
ACTION=="add", SUBSYSTEM=="net", KERNEL=="dummy*", ATTR{name}=="kube0", RUN+="/sbin/ip link add kube0 type dummy"
```

### **Step 3: Ensure Interface Creation at Boot**

Since `netplan` may not create the `kube0` dummy interface by itself, we can ensure it's created persistently using a systemd service.

Create a new systemd service to set up the dummy interface at boot:

```bash
sudo nano /etc/systemd/system/setup-dummy-kube0.service
```

Add the following content:

```bash
[Unit]
Description=Setup dummy network interface kube0
After=network-pre.target
Before=network-online.target

[Service]
# Cannot support mulitple ExecStart
# ExecStart=/sbin/ip link add kube0 type dummy
# ExecStart=/sbin/ip link set kube0 up
# ExecStart=/sbin/ip addr add 192.168.100.150/24 dev kube0
ExecStart=/bin/bash -c '/sbin/ip link add kube0 type dummy && /sbin/ip link set kube0 up && /sbin/ip addr add 192.168.100.150/24 dev kube0'

RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

---

[Unit]
Description=Setup dummy network interface kube0
After=network-pre.target
Before=network-online.target

[Service]
Type=oneshot # executed only once
# ExecStart=/bin/bash -c '/sbin/ip link add kube0 type dummy && /sbin/ip link set kube0 up && /sbin/ip addr add 192.168.100.150/24 dev kube0'
# ExecStart=/bin/bash -c '/sbin/ip link show kube0 > /dev/null 2>&1 && echo "Interface kube0 already exists, skipping creation." || (echo "Creating dummy interface kube0..." && /sbin/ip link add kube0 type dummy && /sbin/ip link set kube0 up && /sbin/ip addr add 192.168.100.150/24 dev kube0)'
ExecStart=/bin/bash -c '/sbin/ip link show kube0 > /dev/null 2>&1 || (/sbin/ip link add kube0 type dummy && /sbin/ip link set kube0 up && /sbin/ip addr add 192.168.100.150/24 dev kube0)'

RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable setup-dummy-kube0.service
sudo systemctl start setup-dummy-kube0.service
```

---