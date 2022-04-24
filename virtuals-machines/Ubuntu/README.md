# Ubuntu VM
## Unraid configuration
```
CPU: AMD Ryzen 7 3700X
GPU: NVIDIA GeForce RTX 2070 SUPER
RAM: 32Go @ 3200MHz
SSD: NVME (M.2) Sabrent 2To
```
### VFIO
- pass through NVME drive group
- pass through all graphics card groups
```
Group 14	 01:00.0			1987:5012	Non-Volatile memory controller: Phison Electronics Corporation E12 NVMe Controller (rev 01)
Group 22	 07:00.0			10de:1e84	VGA compatible controller: NVIDIA Corporation TU104 [GeForce RTX 2070 SUPER] (rev a1)
Group 23	 07:00.1			10de:10f8	Audio device: NVIDIA Corporation TU104 HD Audio Controller (rev a1)
Group 24	 07:00.2			10de:1ad8	USB controller: NVIDIA Corporation TU104 USB 3.1 Host Controller (rev a1)
                                                    *USB devices attached to controllers bound to vfio are not visible to unRAID*
Group 25	 07:00.3			10de:1ad9	Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller (rev a1)
```
### CPU Pinning
- isolate needed CPUs
```
UNRAID:
0-8
1-9
```
```
ISOLATED:
2-10
3-11
4-12
5-13
6-14
7-15
```
### VM Settings
- **CPU Mode**: `Host Passthrough`
- **Logical CPUs**: `Select isolated CPUs`
- **Machine**: `Q35-5.1` (for NVIDIA)
- **BIOS**: `SeaBIOS`
- **OS Install ISO**: `Path to Ubuntu ISO file`
- **OS Install CRRom BUS**: `SATA`
- **Graphics card**: `NVIDIA Card`
- **Other PCI Devices**: `NVME Disk`
- **Primary vDisk**: Remove vDisk from XML. Don't need vDisk because of NVME pass through.
```xml
  <devices>
    <emulator>/usr/local/sbin/qemu</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='raw' cache='writeback'/>
      <source file='/mnt/user/domains/ssd/vdisk1.img'/>
      <target dev='hdc' bus='virtio'/>
      <boot order='1'/>
      <address type='pci' domain='0x0000' bus='0x03' slot='0x00' function='0x0'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/mnt/user/isos/ubuntu-20.04.3-desktop-amd64.iso'/>
      <target dev='hda' bus='sata'/>
      <readonly/>
      <boot order='2'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
```
to
```xml
  <devices>
    <emulator>/usr/local/sbin/qemu</emulator>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/mnt/user/isos/ubuntu-20.04.3-desktop-amd64.iso'/>
      <target dev='hda' bus='sata'/>
      <readonly/>
      <boot order='2'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
```
#### More CPU settings
- Add `emulatorpin` setting to `cputune` xml section.
> For exemple: Assign VM Management to Core 0 and 1 of the CPU.
> (Depend of isolated CPUs you wan to use.)
```xml
<emulatorpin cpuset='0-8,1-9'/>
```
