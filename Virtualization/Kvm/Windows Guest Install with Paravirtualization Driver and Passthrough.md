
### Configure OS bootable firmware

**1. Configure loader**
```xml
<loader readonly="yes" type="pflash">/usr/share/OVMF/OVMF_CODE_4M.fd</loader>
```

**2. Configure nvram**
```xml
<nvram template="/usr/share/OVMF/OVMF_VARS_4M.fd">/var/lib/libvirt/qemu/nvram/Windows_Server_2025_VARS.fd</nvram>
```

**3. Final Configuration**
```xml
<os firmware="efi">
<type arch="x86_64" machine="pc-q35-8.2">hvm</type>
<firmware>
  <feature enabled="no" name="enrolled-keys"/>
  <feature enabled="no" name="secure-boot"/>
</firmware>
<loader readonly="yes" type="pflash">/usr/share/OVMF/OVMF_CODE_4M.fd</loader>
<nvram template="/usr/share/OVMF/OVMF_VARS_4M.fd">/var/lib/libvirt/qemu/nvram/Windows_Server_2025_VARS.fd</nvram>
<bootmenu enable="yes"/>
</os>
```

---
### Configure Storage Passthrough (SATA SSD)

**1. Add SCSI Controller**
```xml
<controller type="scsi" index="0" model="virtio-scsi">
  <address type="pci" domain="0x0000" bus="0x00" slot="0x0a" function="0x0"/>
</controller>
```

**2. Storage Selection with SCSI Controller**
```xml
<disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
  <source dev="/dev/disk/by-id/ata-Patriot_P210_512GB_P210EDCB2109152394"/>
  <target dev="sda" bus="scsi"/>
  <boot order="1"/>
  <address type="drive" controller="0" bus="0" target="0" unit="0"/>
</disk>
```

---
### Configure Paravirtualization Driver (virtio)

**Load Guess OS Paravirtualization Driver (windows)**
```xml
<disk type="file" device="cdrom">
  <driver name="qemu" type="raw"/>
  <source file="/home/akito/Iso/virtio-win-0.1.266.iso"/>
  <target dev="sdc" bus="sata"/>
  <readonly/>
  <address type="drive" controller="0" bus="0" target="0" unit="2"/>
</disk>
```

**1. Set network driver to virtio**
```xml
<interface type="network">
  <mac address="52:54:00:32:8c:99"/>
  <source network="default"/>
  <model type="virtio"/>
  <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
</interface>
```

**2. Set video driver to spice virtio**

Add controller before setting up channel
```xml
<controller type="virtio-serial" index="0">
  <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
</controller>
```

Setup channel (spice), display spice, video (virtio)
```xml
<!--
This defines a SPICE channel for device communication (spicevmc) using virtio-serial transport. This channel handles non-graphical data like clipboard sharing, USB redirection, etc.
-->
<channel type="spicevmc">
  <target type="virtio" name="com.redhat.spice.0"/>
  <address type="virtio-serial" controller="0" bus="0" port="1"/>
</channel>

<!-- 
This defines the SPICE server display protocol using specific gpu render node (amd rx570). This channel handles graphical data. 
-->
<graphics type="spice">
  <listen type="none"/>
  <image compression="off"/>
  <gl enable="yes" rendernode="/dev/dri/by-path/pci-0000:0a:00.0-render"/>
</graphics>

<!--
This defines the virtual GPU using virtio-gpu driver with 3D acceleration enabled and specifies its PCI address.
-->
<video>
  <model type="virtio" heads="1" primary="yes">
    <acceleration accel3d="yes"/>
  </model>
  <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
</video>
```

Advantage of this setup:
- **Dynamic Memory Management**: The virtio-gpu driver uses a more modern approach where memory for graphics is allocated dynamically from the host system's memory pool as needed, rather than requiring a fixed reservation upfront.
- **Shared Memory Architecture**: When using virtio with SPICE and 3D acceleration enabled (`accel3d="yes"`), the system can leverage the host GPU's memory and processing capabilities more directly through the specified render node (`/dev/dri/by-path/pci-0000:0a:00.0-render`).
- **Efficient Resource Utilization**: Instead of emulating a complete graphics card with dedicated memory, virtio creates an efficient paravirtualized interface between the guest and host, reducing overhead.
- **Host-Directed Rendering**: With GL enabled (`gl enable="yes"`), much of the 3D rendering workload is offloaded to the host GPU, which manages its own memory allocation.