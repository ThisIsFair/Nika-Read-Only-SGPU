# IMPORTANT: The spoofing steps are creating an issue, which makes this guide pointless to follow as of right now. Please do not follow the guide, as it'll be a waste of time, at least until I or someone else can come up with a way to resolve it. This note will be replaced when the guide is good to follow again.

# Nika Read Only

- Apex Legends external cheat for Linux.
- As of Season 23, for QEMU/KVM (formerly for Proton).

```shell
+----------+    +----------+    +------------+    +--------------+
| Linux PC | -> | QEMU/KVM | -> | Windows VM | -> | Apex Legends |
+----------+    +----------+    +------------+    +--------------+
```

## Introduction

- The goal of this project is to have a working Linux cheat that can run alongside Apex Legends on my i5-6600K 4c/4t Linux PC.

![Screenshot.jpg](Screenshot.jpg)

## Features

* [x] Stable CR3 shuffle for [Windows 10 20H1](https://archive.org/details/win-10-2004-english-x-64_202010)
* [x] Overlay based ESP for players and items
* [x] Press 5 / 6 / 7 / 8 / 9 / 0 to cycle LIGHT / ENERGY / SHOTGUN / HEAVY / SNIPER / GEAR items
* [x] Map radar
* [x] Spectators list
* [x] Humanized aimbot
* [x] Inside FOV circle, hold RMB (Right Mouse Button) to aimbot **skynade** (even behind cover)
* [x] Hold SHIFT to **lock on target** and **triggerbot** auto fire
* [x] Toggle **aimbot** strength with CURSOR_LEFT; "**<**" symbol in the upper left corner of the screen
* [x] Toggle **ADS locking** with CURSOR_RIGHT; "**>**" symbol in the upper left corner of the screen
* [x] Disable/enable **triggerbot** auto fire with CURSOR_UP; "**^**" symbol in the upper left corner of the screen
- **Bind X in-game to fire, triggerbot will use that key** (default AIMBOT_FIRING_KEY)
- **Unbind LMB (Left Mouse Button) in-game from fire, so that the cheat will fire for you instead** (AIMBOT_ACTIVATED_BY_MOUSE default YES)
* [x] Toggle hitbox with CURSOR_DOWN; `body`/`neck`/`head` text in the upper left corner of the screen
* [ ] Hold CAPS_LOCK to **superglide**
* [x] Press F8 to dump **r5apex** and scan for offsets
* [x] Press F9 twice to terminate cheat

# The guide may contain a few scuffs, please let me know if you run into any issues when following the guide.

### 1. Environment set up in Linux

- Enter BIOS and enable Virtualization Technology:
  - VT-d for Intel (VMX)
  - AMD-Vi for AMD (SVM)
  - Enable "IOMMU"
  - Disable "Above 4G Decoding"
- This guide is created with the usage of another guide but merged into this with intentions of making it simplier.

### 2. Edit the bootloader and prerequisites 

- I did this with Fedora 41 KDE, this should work with other distros too, if you come up with any issues on other distros try finding the command that accomplishes the same thing but for that distro or use chatgpt to do it.
- you can find Fedora 41 KDE here: https://fedora.mirrorservice.org/fedora/linux/releases/41/Spins/x86_64/iso/Fedora-KDE-Live-x86_64-41-1.4.iso

- Make sure your OS is updated: ```sudo dnf update``` afterward reboot (amd doesn't need the next command) and run ```sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda```  
- Make sure you have a linux password set this will be VERY important for Putty
- Run ```sudo dnf group install --withoptional virtualization```

- in the terminal insert ```sudo vi /etc/default/grub```
- Press A to start typing
- Were going to be editing ```GRUB_CMDLINE_LINUX_DEFAULT```
- Keep anything previously there (Change the "intel_iommu=on" to "amd_iommu=on" in the following text if you have an AMD cpu) Add at the end: ```resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap intel_iommu=on quiet```
- exit the vi prompt by pressing esc, and typing :wq
- Update grub via ```sudo grub2-mkconfig -o /etc/grub2.cfg```  

### 2.1 Libvirtd changes

- ```sudo vi /etc/libvirt/libvirtd.conf``` or just go to the actual directory and use kwrite
- Edit `/etc/libvirt/qemu.conf` and uncomment (needed for **audio**):
```shell
#user = "libvirt-qemu"
user = "1000"
```
- remove the comment (the #) from the following: ```unix_sock_group = "libvirt"``` and ```unix_sock_rw_perms = "0770"``` (You can find these between line 80 and 105)
- add this to the end of the file ```log_filters="3:qemu 1:libvirt"
log_outputs="2:file:/var/log/libvirt/libvirtd.log"```
- Run ```sudo usermod -a -G kvm,libvirt $(whoami)``` in the terminal
- Run ```sudo systemctl enable libvirtd``` and afterwards run ```sudo systemctl start libvirtd```
- Run ```sudo groups $(whoami)``` and make sure your username appears, if it doesn't you did the steps wrong.
- Example: ![image](https://github.com/user-attachments/assets/0f69d679-5c14-423a-b17c-8fb90a337c0e)


### 2.2 Qemu chanes

- ```sudo vi /etc/libvirt/qemu.conf``` or go to the directory and use kwrite
- At lines 518 to 524 remove the # from ```user = "root"``` and ```group = "root"```
- Replace the root with your username keeping the quotation marks.
- Example: ![image](https://github.com/user-attachments/assets/cf6dbbd3-b7e1-417f-9391-9ab2419a7a6e)
- after you change ```root``` with your username, run ```sudo systemctl restart libvirtd```
- run ```sudo virsh net-autostart default```

# 2.3 VM Setup

- Virtual Machine Manager >> File >> New Virtual Machine

- Local install media (ISO image or CDROM) >> `Windows10.iso` >> Choose Memory and CPU settings >> **Disable** storage for this virtual machine >> Customize configuration before install
  - Overview >> Chipset: Q35, **Firmware**: `OVMF_CODE_4M.secboot` or `UEFI` >> [Apply]
  - [Add Hardware] >> Storage >> Device type: Disk device >> Bus type: SATA >> Create a disk image for the virtual machine: 240 GiB >> Advanced options >> Serial: B4NN3D53R14L >> [Finish]
  - [Begin Installation] >> Virtual Machine >> Shut Down >> Force Off

- Virtual Machine Manager >> [Open] >> View >> Details >> Video QXL >> Model: VGA >> [Apply]

- Virtual Machine Manager >> [Open] >> View >> Details >> NIC :xx:xx:xx >> XML


- Replace `<mac address="52:54:00:xx:xx:xx"/>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <mac address="xx:xx:xx:xx:xx:xx"/>
  ```
  </details>

- Virtual Machine Manager >> [Open] >> View >> Details >> SATA Disk 1 >> XML


- Replace `<driver name="qemu" type="raw"/>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <driver name="qemu" type="raw" cache="none" discard="ignore"/>
  ```
  </details>
  
### Configure VM XML

- Virtual Machine Manager >> [Open] >> View >> Details >> Overview >> XML


- Replace `<domain type="kvm">` and [Apply]:
  <details>
    <summary>Spoiler <b>(do NOT use this example, instead modify it with your own SMBIOS data; sudo dmidecode)</b></summary>

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

# Windows Setup

- Virtual Machine Manager >> [Open] >> View >> Details >> Boot Options >> Boot device order:
  * [x] SATA Disk 1 >> [Apply]

# Script installation

- Run ```git clone https://gitlab.com/risingprismtv/single-gpu-passthrough.git``` 
- Either CD into the folder or go into the folder and right click somewhere in a blank space and open terminal
- Run ```sudo chmod +x install_hooks.sh``` then run ```sudo ./install_hooks.sh```
- (Optional) Verify the files: 
- in /etc/systemd/system/ there should be "libvirt-nosleep@.service"
- in /usr/local/bin/ there should be vfio-startup and vfio-teardown
- in /etc/libvirt/hooks/ there should be "qemu"

- AMD GPU Potential Fix below

- Remove the files from the directory in "(Optional) Verify the files"
- Run ```git clone https://gitlab.com/akshaycodes/vfio-script```
- Open the terminal in the folder or cd into it
- ```sudo bash vfio_script_install.sh```

# Attaching the GPU

- Remove ```Display spice``` and remove ```Video QXL``` (If it is grey remove the other one first)

- Run 
```
#!/bin/bash
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
- One of the groups should contain your gpu (DO NOT ADD BRIDGES)
Example: ![image](https://github.com/user-attachments/assets/e78ebe2d-e975-408f-b44d-4f6e09a20769)
- So in that example you would add 07:00.0 and 07:00.1 you would NOT add the Host bridge nor PCI bridge.
- Now add hardware > PCI Host Device > and add whatever your gpu device id is

### Configure evdev passthrough (on Linux PC)

- Find your **mouse** and **keyboard** with:
```shell
ls -l /dev/input/by-id/

usb-COMPANY_USB_Device-event-if02 -> ../event7
usb-COMPANY_USB_Device-event-kbd -> ../event4
usb-COMPANY_USB_Device-if01-event-mouse -> ../event5
usb-COMPANY_USB_Device-if01-mouse -> ../mouse0
usb-COMPANY_USB_Device-if02-event-kbd -> ../event6
usb-SONiX_USB_DEVICE-event-if01 -> ../event9
usb-SONiX_USB_DEVICE-event-kbd -> ../event8
```

- By symlink `../mouse0` you find that `usb-COMPANY_USB_Device` is your **mouse**.

- You are looking for `event-mouse` and `event-kbd`:
  - `usb-COMPANY_USB_Device-if01-event-mouse -> ../event5` is your **mouse**.
  - `usb-SONiX_USB_DEVICE-event-kbd -> ../event8` is your **keyboard**.

- Edit `/etc/libvirt/qemu.conf` and uncomment:
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

- Include `cgroup_device_acl` as above, replacing `event-kbd`, `event-mouse`, and the path to each symlink `/dev/input/eventX`.

- Restart libvirtd:
```shell
sudo systemctl restart libvirtd
```

- Toggle input with LEFT_CTRL + RIGHT_CTRL when needed.

### Configure VM

- Virtual Machine Manager >> [Open] >> View >> Details >> Overview >> XML


- Replace `</qemu:commandline>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
    <qemu:arg value="-object"/>
    <qemu:arg value="input-linux,id=kbd1,evdev=/dev/input/by-id/usb-SONiX_USB_DEVICE-event-kbd,grab_all=on,repeat=on"/>
    <qemu:arg value="-object"/>
    <qemu:arg value="input-linux,id=mouse1,evdev=/dev/input/by-id/usb-COMPANY_USB_Device-if01-event-mouse"/>
  </qemu:commandline>
  ```
  </details>

- Join **input group**:
```shell
test $UID = 0 && exit
sudo usermod -aG input $USER
```


  <details>
    <summary>Manually stop `SELinux` every reboot on <b>Fedora Linux</b>:</summary>

    sudo setenforce 0
  </details>

  
  <details>
    <summary>Permanently disable `AppArmor` on <b>Debian Linux</b>:</summary>

    sudo systemctl stop apparmor
    sudo systemctl disable apparmor
  </details>

- Restart Linux PC.

- Open Nika.ini and change ```START_OVERLAY = YES``` to ```START_OVERLAY = NO```

### 7. Spoof qemu-system-x86_64 (mandatory)

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

### 7.1 Spoof OVMF (mandatory)

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

### 8. memflow-kvm (not required, ignore this)


  <details>
    <summary>Install <b>dkms</b> on <b>Fedora Linux</b>:</summary>

    sudo dnf install kernel-devel-$(uname -r)
    sudo dnf install kernel-devel-matched-$(uname -r)
    sudo dnf install dkms
  </details>


  <details>
    <summary>Install <b>dkms</b> on <b>Debian Linux</b>:</summary>

    sudo apt install linux-headers-amd64=6.12.38-1
    sudo apt install dkms
  </details>

- Download `memflow-0.2.1-source-only.dkms.tar.gz` from:
https://github.com/memflow/memflow-kvm/releases

- Install:
```shell
sudo dkms install --archive=memflow-0.2.1-source-only.dkms.tar.gz
```

- Run:
```shell
sudo modprobe memflow
cd path/to/extracted/repository
sudo -E ./nika
```

- If kvm does not get detected add this to your grub line `ibt=off`(or try `no_cet` if that doesn't work)
