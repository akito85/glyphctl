The vm disk is located in:
```bash
/var/lib/libvirt/images/
```

The XML configuration is located in:
```bash
# copy
/etc/libvirt/qemu/

# dump
virsh dumpxml <vm_name> <path_to_xml_file>

```

To import XML configuration:
```bash
virsh define <path_to_xml_file>
```