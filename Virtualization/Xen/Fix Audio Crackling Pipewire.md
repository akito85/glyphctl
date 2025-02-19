### Audio Server Linux Architecture
```bash
	Hardware → ALSA → PipeWire ← WirePlumber
                    ↓
            [Compatibility Layers]
                    ↓
    PulseAudio Apps  JACK Apps  ALSA Apps
```
---
### **1. Buffer Underruns or Overruns**

**Cause:** Low buffer sizes or high latency can cause crackling, popping, or distortion.
**Solution:** Adjust PipeWire’s buffer size and latency settings in `/etc/pipewire/pipewire.conf.d/` or `/usr/share/pipewire/pipewire.conf.d/` by adding:

```bash
context.properties = {
	default.clock.rate = 48000
	default.clock.quantum = 1024
	default.clock.min-quantum = 1024
	default.clock.max-quantum = 8192
}
```

Adjust RT (real time) limits settings: `/etc/security/limits.conf` or `/etc/security/limits.d/95-pipewire.conf`

```bash
@audio soft memlock unlimited 
@audio hard memlock unlimited 
@audio soft rtprio 95 
@audio hard rtprio 95
```

Reload limits `ulimit -r -l` then restart PipeWire:

```bash
systemctl --user restart pipewire
```

Check PipeWire is reading the config

```bash
PIPEWIRE_DEBUG=3 pipewire
```

Verify runtime settings

```bash
pw-dump | grep quantum
```

Check configuration error (view log)

```bash
journalctl --user -u pipewire --since today --no-pager
```

Verify module settings

```bash
pactl info pw-cli info all | grep -i clock
```

Check if there's memory allocation

```bash
journalctl --user -u pipewire --no-pager | grep -i "Cannot allocate memory"
```

### **2. Conflicting Services (PulseAudio / JACK / ALSA)**

**Cause:** Tumbleweed may have PulseAudio, ALSA, or JACK interfering with PipeWire.
**Solution:** Ensure PipeWire is handling all audio by checking:

```bash 
pactl info | grep "Server Name"

Server Name: PulseAudio (on PipeWire 1.3.82)
```

If it still shows `pulseaudio`, disable it:

```bash
systemctl --user mask pulseaudio.socket pulseaudio.service 
systemctl --user restart pipewire
```

### **3. Power Management Issues (CPU or Audio Card Throttling)**

**Cause:** Aggressive power-saving features may cause latency spikes.
**Solution:**
 
Check audio
```bash
lspci | grep -i audio
00:1f.3 Audio device: Intel Corporation Sunrise Point-LP HD Audio (rev 21)
```

Or
```bash
cat /proc/asound/cards
0 [PCH            ]: HDA-Intel - HDA Intel PCH
                     HDA Intel PCH at 0xd5228000 irq 143
```

Or
```bash
sudo lshw -C sound
*-usb:1
       description: Video
       product: Integrated Webcam
       vendor: CNFEH36m301030020880
       physical id: 5
       bus info: usb@1:5
       version: 65.10
       serial: 0x0001
       capabilities: usb-2.00
       configuration: driver=uvcvideo maxpower=500mA speed=480Mbit/s
*-multimedia
       description: Audio device
       product: Sunrise Point-LP HD Audio
       vendor: Intel Corporation
       physical id: 1f.3
       bus info: pci@0000:00:1f.3
       logical name: card0
       logical name: /dev/snd/controlC0
       logical name: /dev/snd/hwC0D0
       logical name: /dev/snd/hwC0D2
       logical name: /dev/snd/pcmC0D0c
       logical name: /dev/snd/pcmC0D0p
       logical name: /dev/snd/pcmC0D3p
       logical name: /dev/snd/pcmC0D7p
       logical name: /dev/snd/pcmC0D8p
       version: 21
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi bus_master cap_list
       configuration: driver=snd_hda_intel latency=32
       resources: irq:143 memory:d5228000-d522bfff memory:d5200000-d520ffff
```

Disable audio power management:
```bash
echo 'options snd_hda_intel power_save=0' | sudo tee /etc/modprobe.d/audio_powersave.conf 

sudo update-initramfs -u
```

Disable CPU frequency scaling (test temporarily):
```bash
sudo cpupower frequency-set -g performance
```

### **4. Hardware Driver Issues (ALSA or PipeWire Backend)**

**Cause:** Some audio devices have poor driver support.
**Solution:**

List available audio devices:

```bash
pw-cli ls Nodec
```

Check logs for errors:

```bash
journalctl --user -u pipewire --no-pager | grep -i error
```

If you are using a USB audio device, try switching the profile:

```bash
pactl list cards 
Card #42
	Name: alsa_card.pci-0000_00_1f.3
	Driver: alsa
	Owner Module: n/a
	Properties:
```

Card id #42

```bash
pactl set-card-profile 42 off 
pactl set-card-profile 42 pro-audio
```

Check active profile

```bash
pactl list cards | rg pro-audio
		pro-audio: Pro Audio (sinks: 4, sources: 1, priority: 1, available: yes)
	Active Profile: pro-audio
```

Check Dmesg Logs for Audio Errors

```bash
dmesg | grep -i audio journalctl --user -u pipewire | grep -i error
```

### **5. WirePlumber Conflicts (PipeWire Session Manager)**

**Cause:** WirePlumber might be misconfigured.
**Solution:** Restart WirePlumber:

```bash
systemctl --user restart wireplumber
```

### **6. Adjust CPU Pinning for Audio Processing**

If you must use Xen, **pin PipeWire to a specific vCPU** with:

```bash
taskset -c 2,3 pipewire
```

Or for persistent CPU affinity, modify `/etc/systemd/system/pipewire.service.d/override.conf`:

```
[Service] CPUAffinity=2 3
```

### **7. Boot with Correct Xen Parameters**

Since **Xen modifies IRQ handling and CPU scheduling**, ensure you optimize the boot parameters:

1. Open `/etc/default/grub` and add these parameters under `GRUB_CMDLINE_LINUX_DEFAULT`:

```bash
dom0_max_vcpus=2 dom0_vcpus_pin dom0_vcpu0_pin=1 dom0_mem=8G,max:8G dom0=pvh tsc=reliable clocksource=tsc sched=credit2 sync_console nosoftlockup=1 nmi_watchdog=0 processor.max_cstate=1 intel_idle.max_cstate=1
```
    
2. Regenerate GRUB:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
    
3. Reboot and check if the issue persists.

### **8. Prevent IRQ Sharing (Xen Issue)**

List IRQs:

```bash
cat /proc/interrupts | grep snd
143:        482          0 PCI-MSI-0000:00:1f.3   0-edge      snd_hda_intel:card0
```

Create a new file, for example, `/usr/local/bin/set_irq_affinity.sh`, with the following content:

```bash
#!/bin/bash

# IRQ number to pin
irq=143

# CPU(s) to pin to (e.g., 1 for CPU 0, 2 for CPU 1, 3 for CPU 0 and 1)
cpu_mask=1

# Check if the IRQ exists
if [ -f /proc/irq/$irq/smp_affinity ]; then
  echo "$cpu_mask" > /proc/irq/$irq/smp_affinity
  echo "IRQ $irq pinned to CPU(s) $(printf '0x%x' "$cpu_mask")"
else
  echo "IRQ $irq not found."
fi
```

Make the script executable

```bash
sudo chmod +x /usr/local/bin/set_irq_affinity.sh
```

Create a new file, for example, `/etc/systemd/system/set_irq_affinity.service`, with the following content:

```toml
[Unit]
Description=Set IRQ affinity for sound card
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/set_irq_affinity.sh

[Install]
WantedBy=multi-user.target
```

Enable and start the service

```bash
sudo systemctl enable set_irq_affinity.service
sudo systemctl start set_irq_affinity.service
```

Verify the service

```bash
sudo systemctl status set_irq_affinity.service
```

This should show that the service is active and has run successfully. You can also check the IRQ affinity:

```bash
cat /proc/irq/143/smp_affinity
```

### **9. Adjust ALSA Power Management**

Disable power saving for ALSA (to avoid latency spikes):

```bash
echo 'options snd_hda_intel power_save=0' | sudo tee /etc/modprobe.d/audio_powersave.conf sudo update-initramfs -u
```

Reboot the system.

### **10. Reduce PulseAudio Latency (If Still Enabled)**

Edit PulseAudio settings:

```bash
nano ~/.config/pulse/daemon.conf
```

Add:

```bash
default-fragments = 5 default-fragment-size-msec = 2 resample-method = speex-float-10 avoid-resampling = yes
```

Restart PulseAudio:

```bash
systemctl --user restart pulseaudio
```

### **11. Use Hugepages for Faster Memory Access**

Configure **hugepages** for VM memory to improve performance:

```bash
hugepages=1G
```

Set up persistent hugepages:
```bash
echo 16 | sudo tee /sys/kernel/mm/hugepages/hugepages1048576kB/nr_hugepages
```
