# OpenShift Dev Spaces Workspace for AI LLM Development with Intel GPU

Initial Attempt to leverage native AI capabilities in the Intel Untra Core 7 chipset

__Note:__ NPU support is not working yet.  llama-cpp does not support the NPU


## Machine Config to leak GPU into a Pod

__Note:__ `/dev/net/tun` and `/dev/fuse` are enabled OOTB.  They are included here for compatibility.  

```bash
cat << EOF | butane | oc apply -f -
variant: openshift
version: 4.18.0
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: enable-gpu
storage:
  files:
  - path: /etc/crio/crio.conf.d/99-intel-gpu
    mode: 0644
    overwrite: true
    contents:
      inline: |
        [crio.runtime]
        allowed_devices = [
          "/dev/fuse",
          "/dev/net/tun",
          "/dev/dri/renderD128"
        ]
EOF
```

## Intel GPU Devices

[https://dgpu-docs.intel.com/devices/hardware-table.html](https://dgpu-docs.intel.com/devices/hardware-table.html)

```bash
cat /sys/bus/pci/drivers/i915/0000:00:02.0
```
