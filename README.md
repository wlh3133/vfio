# My VFIO Setup
This document describes how to recreate my personal VFIO setup.

## Hardware
- CPU: Ryzen 7 1700
- Motherboard: ASUS ROG STRIX X370-F GAMING
- RAM: 16 GB
- Host GPU: RX 570
- Guest GPU: GTX 980 Ti

## Prerequesites
#### Set Kernel Parameters 
Edit **/etc/default/grub**

Before:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```
After:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash amd_iommu=on iommpu=pt video=efifb:off vfio-pci.ids=10de:17c8,10de:0fb0 isolcpus=8-15 nohz_full=8-15 rcu_nocbs=8-15"
```
- `amd_iommu=on`: Enable IOMMU
- `iommu=pt`: Improves performance. See [this](http://mails.dpdk.org/archives/dev/2014-October/007411.html) for a (very) detailed description.
- `video=efifb:off`: No display output from guest GPU without this. See [this post on The Passthrough POST](https://passthroughpo.st/explaining-csm-efifboff-setting-boot-gpu-manually/)
- `vfio-pci.ids=10de:17c8,10de:0fb0`: Use vfio-pci driver for guest GPU
- `isolcpus=8-15`: Isolates and pins CPU cores so that they are exclusively used by the guest OS. **Caution:** Disables host OS to use these CPU cores
- `nohz_full=8-15` and `rcu_nocbs=8-15`: Removes microstutters in the guest OS. I don't know exactly why this works. See [this old Reddit post](https://www.removeddit.com/r/VFIO/comments/6vgtpx/high_dpc_latency_and_audio_stuttering_on_windows/dm0sfto/)

#### Blacklist GPU to prevent driver loading
Create **/etc/modprobe.d/vfio.conf** and add these parameters:

```
  options vfio-pci ids=10de:17c8,10de:0fb0 
  options kvm ignore_msrs=1
```
- `options vfio-pci ids=10de:17c8,10de:0fb0`: Further blacklisting of guest GPU
- `options kvm ignore_msrs=1`: Windows will not boot without this. See [this post on the Level1Techs forum](https://forum.level1techs.com/t/windows-10-1803-as-guest-with-qemu-kvm-bsod-under-install/127425/13)

#### Load VFIO kernel modules
Edit **/etc/initramfs-tools/modules** and append these parameters:
```
vfio_pci
vfio
vfio_iommu_type1
vfio_virqfd
```
