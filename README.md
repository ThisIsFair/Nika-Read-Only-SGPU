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
- Download Stable virtio-win ISO from https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md
- Download W10 iso https://www.microsoft.com/en-us/software-download/windows10ISO

# 2.3 VM Setup

- Create the vm, add the w10 iso.
- Uncheck the automatically detect box and change it to windows 10 (this is to make sure that the vm is called w10)
- Set the storage to 200 gb | Set the Memory to whatever you want (I use half) and CPU will be changed later
- Check "Customize configuration before install" and click finish
- Now the important stuff
- Set the chipset to Q35 and Firmware to UEFI
- Check your cpu info via google or ```lscpu | egrep 'Model name|Socket|Thread|NUMA|CPU\(s\)'```
- I set mine to 8 cores, 1 socket, and 1 thread.
- Now go to disk drive, and set it to VirIO and click advanced options > Cache Mode > Write back > change the serial to whatever nika uses, at this time it is "B4NN3D53R14L" > apply
- Go to boot options > check SATA CDROM 1 > put it to the top of priority > apply
- Add hardware > (It should already be on storage) > device type > change to "CDROM device" > Be sure the bus type is SATA > Finish

# Windows Setup

- After you accept the terms and service click "Custom: install windows only (advanced)"
- load driver > ok > ```E:\amd64\w10\viostor.inf``` > next (it will load for a bit) > it should show your storage drive just click next
- Just go thorugh the w10 installation like normal
- (OPTIONAL) when you get to the point of signing into or creating a microsoft account click "create an account", disconnect yourself from the wifi > click the back arrow > and you can just name yourself and reconnnect to the wifi.
- once you finish and get into the actual w10 just turn off the vm

# Script installation

- Run ```git clone https://gitlab.com/risingprismtv/single-gpu-passthrough.git``` 
- Either CD into the folder or go into the folder and right click somewhere in a blank space and open terminal
- Run ```sudo chmod +x install_hooks.sh``` then run ```sudo ./install_hooks.sh```
- (Optional) Verify the files: 
- in /etc/systemd/system/ there should be "libvirt-nosleep@.service"
- in /usr/local/bin/ there should be vfio-startup and vfio-teardown
- in /etc/libvirt/hooks/ there should be "qemu"

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
      <qemu:arg value="type=1,manufacturer=ASUS,product=ASUS Zenbook 14X UX1337,version=23.41,serial=D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=2,manufacturer=ASUS,product=87FD,version=34.12,serial=B1C2D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=3,manufacturer=ASUS,version=23.41,serial=D3E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=17,manufacturer=Samsung,loc_pfx=BANK,speed=4800,serial=E4F56789"/>
      <qemu:arg value="-smbios"/>
      <qemu:arg value="type=4,manufacturer=Intel(R) Corporation,version=13th Gen Intel(R) Core(TM) i9-13900H @ 2.60GHz,max-speed=5400,current-speed=2600"/>
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

- The following command may give you an error when you try to save it, the fix is changing the vcpu amount to the amount you put originally

- Replace from `<memory unit="KiB">4194304</memory>` to `<vcpu placement="static">2</vcpu>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <memory unit="KiB">12582912</memory>
  <currentMemory unit="KiB">12582912</currentMemory>
  <vcpu placement="static">4</vcpu>
  ```
  </details>


- Replace from `<features>` to `</clock>` and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <synic state="on"/>
      <stimer state="on">
        <direct state="on"/>
      </stimer>
      <reset state="on"/>
      <vendor_id state="on" value="GenuineIntel"/>
      <frequencies state="on"/>
      <reenlightenment state="off"/>
      <tlbflush state="on"/>
      <ipi state="on"/>
      <evmcs state="off"/>
      <avic state="on"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <vmport state="off"/>
    <smm state="on"/>
    <ioapic driver="kvm"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="off">
    <topology sockets="1" dies="1" cores="4" threads="1"/>
    <cache mode="passthrough"/>
    <feature policy="require" name="hypervisor"/>
    <feature policy="disable" name="vmx"/>
    <feature policy="disable" name="svm"/>
    <feature policy="disable" name="aes"/>
    <feature policy="disable" name="x2apic"/>
    <feature policy="require" name="ibpb"/>
    <feature policy="require" name="invtsc"/>
    <feature policy="require" name="pdpe1gb"/>
    <feature policy="require" name="ssbd"/>
    <feature policy="require" name="amd-ssbd"/>
    <feature policy="require" name="stibp"/>
    <feature policy="require" name="amd-stibp"/>

    <feature policy="require" name="tsc-deadline"/>
    <feature policy="require" name="tsc_adjust"/>
    <feature policy="require" name="arch-capabilities"/>
    <feature policy="require" name="rdctl-no"/>
    <feature policy="require" name="skip-l1dfl-vmentry"/>
    <feature policy="require" name="mds-no"/>
    <feature policy="require" name="pschange-mc-no"/>

    <feature policy="require" name="cmp_legacy"/>
    <feature policy="require" name="xsaves"/>
    <feature policy="require" name="perfctr_core"/>
    <feature policy="require" name="clzero"/>
    <feature policy="require" name="xsaveerptr"/>
  </cpu>
  <clock offset="timezone" timezone="Europe/London">
    <timer name="rtc" present="no" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="discard"/>
    <timer name="hpet" present="no"/>
    <timer name="kvmclock" present="no"/>
    <timer name="hypervclock" present="yes"/>
    <timer name="tsc" present="yes" mode="native"/>
  </clock>
  ```
  </details>

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

### Configure VM

- Virtual Machine Manager >> [Open] >> View >> Details >> Overview >> XML


- Replace `</domain>` (it should be at the bottom of the xml) and [Apply]:
  <details>
    <summary>Spoiler</summary>

  ```shell
      <qemu:arg value="-object"/>
      <qemu:arg value="input-linux,id=kbd1,evdev=/dev/input/by-id/usb-SONiX_USB_DEVICE-event-kbd,grab_all=on,repeat=on"/>
      <qemu:arg value="-object"/>
      <qemu:arg value="input-linux,id=mouse1,evdev=/dev/input/by-id/usb-COMPANY_USB_Device-if01-event-mouse"/>
    </qemu:commandline>
  </domain>
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

### 6. Nika Read Only (on Linux PC)

- Install:
```shell
cd path/to/extracted/repository
chmod +x nika
```

- Run:
```shell
cd path/to/extracted/repository
sudo -E ./nika
```
- Open Nika.ini and change ```START_OVERLAY = YES``` to ```START_OVERLAY = NO```

### 7. Disable hypervisor (mandatory)

- Edit XML for Intel (on Linux PC):
```shell
<feature policy="disable" name="hypervisor"/>
<feature policy="require" name="vmx"/>
```

- Edit XML for AMD (on Linux PC):
```shell
<feature policy="disable" name="hypervisor"/>
<feature policy="require" name="svm"/>
```

### 8. memflow-kvm (faster VMREAD)


  <details>
    <summary>Install <b>dkms</b> on <b>Fedora Linux</b>:</summary>

    sudo dnf install kernel-devel-$(uname -r)
    sudo dnf install kernel-devel-matched-$(uname -r)
    sudo dnf install dkms
  </details>


  <details>
    <summary>Install <b>dkms</b> on <b>Debian Linux</b>:</summary>

    sudo apt install linux-headers-amd64=6.1.123-1
    sudo apt install dkms
  </details>

- Download `memflow-0.2.1-source-only.dkms.tar.gz` from:
https://github.com/memflow/memflow-kvm/releases

- Install:
```shell
sudo dkms install --archive=memflow-0.2.1-source-only.dkms.tar.gz
```

- IMPORTANT AFTER EVERY REBOOT:
<details>
<summary>Do the Following after every reboot</summary>

- Run `sudo setenforce 0`  
- Run `sudo systemctl start sshd`  

</details>

- If you do not know your IP run `ip addr`  
- It may be different for you but I can find mine in 2: enp5s0 (It should be purple)
- Now install gpu drivers > install putty > connect to putty > open apex > run the command below
- You may not have audio, the fix is adding your audio controller. 
- Example: My audio device is called ```Audio device: Advanced Micro Devices, Inc. [AMD] Starship/Matisse HD Audio Controller``` which I just added as a PCI device and it fixed.

- Run:
```shell
sudo modprobe memflow
cd path/to/extracted/repository
sudo -E ./nika
```
