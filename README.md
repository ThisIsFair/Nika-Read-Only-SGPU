# This is a new format version, due to my other guide just being sloppy. The guide may have issues, but I have checked over it a few times, and it seemed fine. Additional note: The QEMU/KVM spoofing is causing issues for SGPU, so whenever the detection is fixed or when people no longer have issues with Nika, I'll work towards the fix.

# Nika Read Only: Apex Legends External Cheat for Linux

This guide provides step-by-step instructions for setting up an external cheat for Apex Legends on a Linux system using QEMU/KVM. It is designed for Season 23 and supports an i5-6600K 4c/4t Linux PC. The guide assumes familiarity with Linux and virtualization concepts.

> **Note**: This guide may have minor issues. If you encounter problems, please report them for clarification.

## Overview

Nika Read Only is an external cheat for Apex Legends, designed to run on a Linux host with a Windows virtual machine (VM) via QEMU/KVM. The setup allows the cheat to work alongside the game for enhanced gameplay features.

```shell
+----------+    +----------+    +------------+    +--------------+
| Linux PC | -> | QEMU/KVM | -> | Windows VM | -> | Apex Legends |
+----------+    +----------+    +------------+    +--------------+
```

![Screenshot.jpg](Screenshot.jpg)## Features

- [x] Stable CR3 shuffle for Windows 10 20H1

- [x] Overlay-based ESP for players and items

- [x] Cycle through LIGHT / ENERGY / SHOTGUN / HEAVY / SNIPER / GEAR items using keys `5` / `6` / `7` / `8` / `9` / `0`

- [x] Map radar

- [x] Spectators list

- [x] Humanized aimbot

- [x] Aimbot for **skynade** (works behind cover) when holding RMB inside FOV circle

- [x] **Lock on target** and **triggerbot** auto-fire when holding SHIFT

- [x] Toggle **aimbot strength** with CURSOR_LEFT (`<` symbol in top-left corner)

- [x] Toggle **ADS locking** with CURSOR_RIGHT (`>` symbol in top-left corner)

- [x] Enable/disable **triggerbot** auto-fire with CURSOR_UP (`^` symbol in top-left corner)

- [x] Toggle hitbox (`body`/`neck`/`head`) with CURSOR_DOWN (displayed in top-left corner)

- [ ] **Superglide** by holding CAPS_LOCK (not implemented)

- [x] Press `F8` to dump **r5apex** and scan for offsets

- [x] Press `F9` twice to terminate the cheat

- [ ] **In-game bindings**:

  - Bind `X` to fire (used by triggerbot as `AIMBOT_FIRING_KEY`)
  - Unbind LMB from fire (`AIMBOT_ACTIVATED_BY_MOUSE` default: YES)

## Prerequisites

This guide was tested on Fedora 41 KDE but should work on other Linux distributions with equivalent commands. For distro-specific issues, consult documentation or use tools like ChatGPT to adapt commands.

- **Download Fedora 41 KDE or Fedora 42 KDE**: [Fedora KDE 41](https://fedora.mirrorservice.org/fedora/linux/releases/41/Spins/x86_64/iso/Fedora-KDE-Live-x86_64-41-1.4.iso) or [Fedora KDE 42](https://download.fedoraproject.org/pub/fedora/linux/releases/42/KDE/x86_64/iso/Fedora-KDE-Desktop-Live-42-1.1.x86_64.iso)
- Ensure your system has a password set for use with PuTTY.
- Update your OS: `sudo dnf update` and reboot.
- Install NVIDIA drivers (skip for AMD GPUs): `sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda`
- Install virtualization tools: `sudo dnf group install --with-optional virtualization`

## Setup Instructions

### 1. Enable Virtualization in BIOS

1. Enter your BIOS and enable:
   - **Intel**: VT-d (VMX) and IOMMU
   - **AMD**: AMD-Vi (SVM) and IOMMU
2. Disable "Above 4G Decoding".

### 2. Configure Bootloader

1. Edit GRUB configuration:

   ```shell
   sudo vi /etc/default/grub
   ```
2. Press `A` to edit, and append to `GRUB_CMDLINE_LINUX_DEFAULT` (replace `intel_iommu=on` with `amd_iommu=on` for AMD CPUs):

   ```shell
   resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap intel_iommu=on quiet
   ```
3. Save and exit (`Esc`, then `:wq`).
4. Update GRUB: `sudo grub2-mkconfig -o /etc/grub2.cfg`

### 3. Configure Libvirt

1. Edit `/etc/libvirt/libvirtd.conf` (use `sudo vi` or `kwrite`):

   - Uncomment and set: `user = "1000"`
   - Uncomment: `unix_sock_group = "libvirt"` and `unix_sock_rw_perms = "0770"` (lines 80–105)
   - Append:

     ```shell
     log_filters="3:qemu 1:libvirt"
     log_outputs="2:file:/var/log/libvirt/libvirtd.log"
     ```

2. Add user to groups: `sudo usermod -a -G kvm,libvirt $(whoami)`

3. Enable and start libvirtd:

   ```shell
   sudo systemctl enable libvirtd
   sudo systemctl start libvirtd
   ```

4. Verify group membership: `sudo groups $(whoami)`

   - Example:

     ![image](https://github.com/user-attachments/assets/0f69d679-5c14-423a-b17c-8fb90a337c0e)

5. Edit `/etc/libvirt/qemu.conf`:

   - Uncomment and replace `user = "root"` and `group = "root"` with your username (e.g., `user = "yourusername"`).
   - Example:

     ![image](https://github.com/user-attachments/assets/cf6dbbd3-b7e1-417f-9391-9ab2419a7a6e)

6. Restart libvirtd: `sudo systemctl restart libvirtd`

7. Enable default network: `sudo virsh net-autostart default`

### 4. Set Up Virtual Machine

1. Open **Virtual Machine Manager** &gt; File &gt; New Virtual Machine.
2. Select **Local install media** and choose `Windows10.iso`.
3. Configure memory and CPU, then **disable** storage for this VM.
4. Select **Customize configuration before install**:
   - **Overview**: Set Chipset to `Q35` and Firmware to `OVMF_CODE_4M.secboot` or `UEFI`. Apply.
   - **Add Hardware**: Add a Disk device (SATA, 240 GiB, Serial: `B4NN3D53R14L`). Finish.
5. Begin installation, then shut down the VM (**Force Off**).
6. Modify settings:
   - Change **Video QXL** to Model: `VGA`. Apply.
   - Edit **NIC :xx:xx:xx** XML, replace `<mac address="52:54:00:xx:xx:xx"/>` with:

     ```shell
     <mac address="xx:xx:xx:xx:xx:xx"/>
     ```
   - Edit **SATA Disk 1** XML, replace `<driver name="qemu" type="raw"/>` with:

     ```shell
     <driver name="qemu" type="raw" cache="none" discard="ignore"/>
     ```

### 5. Configure VM XML

1. In **Virtual Machine Manager** &gt; Overview &gt; XML, make the following replacements:
   - Replace `<domain type="kvm">` with (use your own SMBIOS data from `sudo dmidecode`):

  ```shell
  <domain type="kvm" xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0">
    <qemu:commandline>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=0,vendor=ASUS,version=X.23,date=06/14/2024,release=12.34"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=1,manufacturer=ASUS,product=ASUS Zenbook 14X UM5401,version=23.41,serial=D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=2,manufacturer=ASUS,product=87FD,version=34.12,serial=B1C2D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=3,manufacturer=ASUS,version=23.41,serial=D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=4,sock_pfx=XPTO,manufacturer=Advanced Micro Devices,, Inc.,version=AMD Ryzen 9 6900HX with Radeon Graphics"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=8,internal_reference=J1A1,external_reference=Keyboard,connector_type=0x0F,port_type=0x0D"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=8,internal_reference=J1A1,external_reference=Mouse,connector_type=0x0F,port_type=0x0E"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=9,slot_designation=J6C1,slot_type=0xAA,slot_data_bus_width=0x0D,current_usage=0x04,slot_length=0x04,slot_id=0x01,slot_characteristics1=0x04,slot_characteristics2=0x03"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=17,manufacturer=Samsung,part=M425R1GB4BB0-CQK,speed=4800,serial=E4F50000"/>
    </qemu:commandline>
  ```
  </details>


- Replace `</metadata>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
    <vmware xmlns="http://www.vmware.com/schema/vmware.config">
      <config>
        <entry name="hypervisor.cpuid.v0" value="FALSE"/>
      </config>
    </vmware>
  </metadata>
  ```
  </details>


- Replace from `<memory unit="KiB">4194304</memory>` to `<vcpu placement="static">2</vcpu>` and [Apply]:
  <details>
    <summary>Spoiler <b>(use a commercial memory size like 8, 16, or 24 GiB; vcpu example for a 24 threads host CPU)</b></summary>

  ```shell
  <memory unit="GiB">24</memory>
  <currentMemory unit="GiB">24</currentMemory>
  <vcpu placement="static">24</vcpu>
  ```
  </details>


- Replace from `<features>` to `</clock>` and [Apply]:
  <details>
    <summary>Spoiler (example for <b>AMD</b> 12 cores 24 threads host CPU)</summary>

  ```shell
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="off"/>
      <vapic state="off"/>
      <spinlocks state="off"/>
      <vpindex state="off"/>
      <runtime state="off"/>
      <synic state="off"/>
      <stimer state="off"/>
      <reset state="off"/>
      <vendor_id state="on" value="AuthenticAMD"/>
      <frequencies state="off"/>
      <reenlightenment state="off"/>
      <tlbflush state="off"/>
      <ipi state="off"/>
      <evmcs state="off"/>
      <avic state="off"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <pmu state="on"/>
    <vmport state="off"/>
    <smm state="on"/>
    <ioapic driver="kvm"/>
    <msrs unknown="fault"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="off">
    <topology sockets="1" cores="12" threads="2"/>
    <cache mode="passthrough"/>
    <feature policy="disable" name="aes"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="require" name="svm"/>
    <feature policy="disable" name="vmx"/>
    <feature policy="disable" name="x2apic"/>
    <feature policy="require" name="topoext"/>
    <feature policy="require" name="invtsc"/>
    <feature policy="disable" name="amd-ssbd"/>
    <feature policy="disable" name="ssbd"/>
    <feature policy="disable" name="virt-ssbd"/>
    <feature policy="disable" name="rdpid"/>
    <feature policy="disable" name="rdtscp"/>
  </cpu>
  <clock offset="localtime">
    <timer name="tsc" present="yes" tickpolicy="discard" mode="native"/>
    <timer name="hpet" present="yes"/>
    <timer name="rtc" present="no"/>
    <timer name="pit" present="no"/>
    <timer name="kvmclock" present="no"/>
    <timer name="hypervclock" present="no"/>
  </clock>
  ```
  </details>


  <details>
    <summary>Spoiler (example for <b>Intel</b> 4 cores 4 threads host CPU)</summary>

  ```shell
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="off"/>
      <vapic state="off"/>
      <spinlocks state="off"/>
      <vpindex state="off"/>
      <runtime state="off"/>
      <synic state="off"/>
      <stimer state="off"/>
      <reset state="off"/>
      <vendor_id state="on" value="GenuineIntel"/>
      <frequencies state="off"/>
      <reenlightenment state="off"/>
      <tlbflush state="off"/>
      <ipi state="off"/>
      <evmcs state="off"/>
      <avic state="off"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <pmu state="on"/>
    <vmport state="off"/>
    <smm state="on"/>
    <ioapic driver="kvm"/>
    <msrs unknown="fault"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="off">
    <topology sockets="1" cores="4" threads="1"/>
    <cache mode="passthrough"/>
    <feature policy="require" name="aes"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="disable" name="svm"/>
    <feature policy="require" name="vmx"/>
    <feature policy="disable" name="x2apic"/>
    <feature policy="disable" name="topoext"/>
    <feature policy="require" name="invtsc"/>
    <feature policy="disable" name="amd-ssbd"/>
    <feature policy="disable" name="ssbd"/>
    <feature policy="disable" name="virt-ssbd"/>
    <feature policy="disable" name="rdpid"/>
    <feature policy="disable" name="rdtscp"/>
  </cpu>
  <clock offset="localtime">
    <timer name="tsc" present="yes" tickpolicy="discard" mode="native"/>
    <timer name="hpet" present="yes"/>
    <timer name="rtc" present="no"/>
    <timer name="pit" present="no"/>
    <timer name="kvmclock" present="no"/>
    <timer name="hypervclock" present="no"/>
  </clock>
  ```
  </details>


- Replace from `<memballoon model="virtio">` to `</memballoon>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <memballoon model="none"/>
  ```
  </details>


- Replace `<audio id="1" type="spice"/>` and [Apply]:
  <details>
    <summary>Spoiler <b>(for pipewire sound, not required)</b></summary>

  ```shell
  <audio id="1" type="pipewire" runtimeDir="/run/user/1000">
    <input name="qemuinput"/>
    <output name="qemuoutput"/>
  </audio>
  ```
  </details>

- Virtual Machine Manager >> [Open] >> View >> Details >> Tablet >> [Remove]

### 6. Configure Windows VM Boot

1. In **Virtual Machine Manager** &gt; Boot Options, set **SATA Disk 1** as the first boot device. Apply.

### 7. Install VFIO Scripts

1. Clone the repository: `git clone https://gitlab.com/risingprismtv/single-gpu-passthrough.git`

2. Navigate to the folder and run:

   ```shell
   sudo chmod +x install_hooks.sh
   sudo ./install_hooks.sh
   ```

3. Verify files:

   - `/etc/systemd/system/libvirt-nosleep@.service`
   - `/usr/local/bin/vfio-startup` and `vfio-teardown`
   - `/etc/libvirt/hooks/qemu`

4. **AMD GPU Fix (if needed)**:

   - Remove the above files.
   - Clone: `git clone https://gitlab.com/akshaycodes/vfio-script`
   - Run: `sudo bash vfio_script_install.sh`

### 8. Attach GPU to VM

1. Remove **Display spice** and **Video QXL** from VM settings.
2. Identify GPU devices:

   ```shell
   #!/bin/bash
   shopt -s nullglob
   for g in /sys/kernel/iommu_groups/*; do
       echo "IOMMU Group ${g##*/}:"
       for d in $g/devices/*; do
           echo -e "\t$(lspci -nns ${d##*/})"
       done;
   done;
   ```
   - Example:

     ![image](https://github.com/user-attachments/assets/e78ebe2d-e975-408f-b44d-4f6e09a20769)
   - Add GPU devices (e.g., `07:00.0` and `07:00.1`, exclude bridges) via **Add Hardware &gt; PCI Host Device**.

### 9. Configure Evdev Passthrough

1. Identify mouse and keyboard:

   ```shell
   ls -l /dev/input/by-id/
   ```
   - Example output:

     ```shell
     usb-COMPANY_USB_Device-event-if02 -> ../event7
     usb-COMPANY_USB_Device-event-kbd -> ../event4
     usb-COMPANY_USB_Device-if01-event-mouse -> ../event5
     usb-COMPANY_USB_Device-if01-mouse -> ../mouse0
     usb-COMPANY_USB_Device-if02-event-kbd -> ../event6
     usb-SONiX_USB_DEVICE-event-if01 -> ../event9
     usb-SONiX_USB_DEVICE-event-kbd -> ../event8
     ```
   - Mouse: `usb-COMPANY_USB_Device-if01-event-mouse -> ../event5`
   - Keyboard: `usb-SONiX_USB_DEVICE-event-kbd -> ../event8`
2. Edit `/etc/libvirt/qemu.conf` and uncomment/add:

   ```shell
   cgroup_device_acl = [
       "/dev/null", "/dev/full", "/dev/zero",
       "/dev/random", "/dev/urandom",
       "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
       "/dev/rtc", "/dev/hpet",
       "/dev/input/by-id/usb-COMPANY_USB_Device-if01-event-mouse",
       "/dev/input/by-id/usb-SONiX_USB_DEVICE-event-kbd",
       "/dev/input/event5",
       "/dev/input/event8",
       "/dev/userfaultfd"
   ]
   ```
3. Restart libvirtd: `sudo systemctl restart libvirtd`
4. Add evdev to VM XML:

   ```shell
   <qemu:arg value="-object"/>
   <qemu:arg value="input-linux,id=kbd1,evdev=/dev/input/by-id/usb-SONiX_USB_DEVICE-event-kbd,grab_all=on,repeat=on"/>
   <qemu:arg value="-object"/>
   <qemu:arg value="input-linux,id=mouse1,evdev=/dev/input/by-id/usb-COMPANY_USB_Device-if01-event-mouse"/>
   ```
5. Join input group: `sudo usermod -aG input $USER`
6. Toggle input with `LEFT_CTRL + RIGHT_CTRL`.

### 10. Disable Security Features

- **Fedora**: Disable SELinux each reboot: `sudo setenforce 0`
- **Debian**: Permanently disable AppArmor:

  ```shell
  sudo systemctl stop apparmor
  sudo systemctl disable apparmor
  ```

### 11. Spoof QEMU (Mandatory)

- This script is based on: [Scrut1ny/Hypervisor-Phantom](https://github.com/Scrut1ny/Hypervisor-Phantom)


  <details>
    <summary>Build on <b>Fedora Linux</b>:</summary>

  ```shell
  sudo dnf builddep qemu
  ```
  </details>


  <details>
    <summary>Build on <b>Debian Linux</b>:</summary>

  ```shell
  sudo apt build-dep qemu
  ```
  </details>

- Run `qemupatch.sh` to clone, patch, and build `qemu-system-x86_64` with generated data.
  - You can edit `default_models` with real data.

- Virtual Machine Manager >> [Open] >> View >> Details >> Overview >> XML


- Replace from `<pm>` to `</emulator>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <pm>
    <suspend-to-mem enabled="yes"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/local/bin/qemu-system-x86_64</emulator>
  ```
  </details>


- Replace `</qemu:commandline>` and [Apply]:
  <details>
    <summary>Spoiler <b>(not required, ignore this)</b></summary>

  ```shell
    <qemu:arg value="-acpitable"/>
    <qemu:arg value="file=/usr/local/bin/ssdt1.aml"/>
    <qemu:arg value="-acpitable"/>
    <qemu:arg value="file=/usr/local/bin/ssdt2.aml"/>
  </qemu:commandline>
  ```
  </details>

### 12. Spoof OVMF (Mandatory)

- This script is based on: [Scrut1ny/Hypervisor-Phantom](https://github.com/Scrut1ny/Hypervisor-Phantom)


  <details>
    <summary>Build on <b>Fedora Linux</b>:</summary>

  ```shell
  sudo dnf install acpica-tools
  sudo dnf install nasm
  ```
  </details>


  <details>
    <summary>Build on <b>Debian Linux</b>:</summary>

  ```shell
  sudo apt install acpica-tools
  sudo apt install nasm
  ```
  </details>

- Run `edk2patch.sh` to clone, patch, and build `OVMF` with generated data.

- Virtual Machine Manager >> [Open] >> View >> Details >> Overview >> XML


- Replace from `<os firmware="efi">` to `</os>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <os>
    <type arch="x86_64" machine="pc-q35-9.2">hvm</type>
    <loader readonly="yes" secure="yes" type="pflash" format="raw">/usr/share/edk2/ovmf/OVMF_CODE_4M.patched.fd</loader>
    <nvram format="raw">/usr/share/edk2/ovmf/OVMF_VARS_4M.patched.fd</nvram>
    <bootmenu enable="yes"/>
  </os>
  ```
  </details>

### 13. Configure Nika

1. Edit `Nika.ini` and set `START_OVERLAY = NO`.

### 14. No Longer Required: Memflow-KVM Setup (Ignore this step)

1. Install DKMS:
   - **Fedora**:

     ```shell
     sudo dnf install kernel-devel-$(uname -r)
     sudo dnf install kernel-devel-matched-$(uname -r)
     sudo dnf install dkms
     ```
   - **Debian**:

     ```shell
     sudo apt install linux-headers-amd64=6.12.38-1
     sudo apt install dkms
     ```
2. Download and install: memflow-0.2.1-source-only.dkms.tar.gz

   ```shell
   sudo dkms install --archive=memflow-0.2.1-source-only.dkms.tar.gz
   ```
3. Load module and run Nika:

   ```shell
   sudo modprobe memflow
   cd path/to/extracted/repository
   sudo -E ./nika
   ```

## Troubleshooting

- If KVM is not detected, add `ibt=off` (or `no_cet`) to GRUB: `sudo vi /etc/default/grub`, then `sudo grub2-mkconfig -o /etc/grub2.cfg`.

