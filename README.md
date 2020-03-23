# kvm-macOS

> Forked from [OSX-KVM](https://github.com/kholia/OSX-KVM).

macOS for Linux users.

## Quickstart

### Install QEMU

Install QEMU with your prefered way.

### Configure KVM

```sh
$ echo 1 > /sys/module/kvm/parameters/ignore_msrs
```

### Prepare macOS installer

Fetch macOS installer:

```sh
$ cd Installer
$ ./fetch-macOS.py
```

> Modern NVIDIA GPUs are supported on HighSierra but not on Mojave (yet).

After executing this step, `BaseSystem.dmg` is downloaded into current folder.

Next, convert this file into a usable format:

```sh
$ qemu-img convert BaseSystem.dmg -O raw BaseSystem.img
```

### Create a virtual HDD image for installing macOS

```sh
$ qemu-img create -f qcow2 storage-macOS.qcow2 50G
```

### Create a virtual HDD image for storage external data

```sh
$ qemu-img create -f qcow2 storage-external.qcow2 100G
```

### Edit and validate `macOS-libvirt.xml`

> `macOS-libvirt.xml` supposes that the root dir of macOS VM is `/vm/macOS`.

```sh
$ $EDITOR macOS-libvirt.xml
$ virt-xml-validate macOS-libvirt.xml
```

### Define a VM

```sh
$ virsh -c qemu:///system define macOS-libvirt.xml
```

### Start virt-manager

```sh
$ virt-manager -c qemu:///system
```

## Change Screen Resolution

### determine your desired resolution

The default resolution is 1440x900 which is widely supported. Suppose that your desired resolution is 1920x1080.

### change Clover resolution

Following steps will regenerate the corresponding `Clover*.qcow2` file.

Download ISO from [Clover EFI bootloader](https://sourceforge.net/projects/cloverefiboot/files/Bootable_ISO/) page, assume that `CloverISO-5070.tar.lzma` is downloaded:

```sh
$ tar xf CloverISO-50.tar.lzma
```

After extracting, we get `Clover-v2.5k-5070-X64.iso`.

Edit `config/config.plist.stripped.qemu`, change the resolution to your desired resolution.

Create a new Clover image:

```sh
$ rm -f Clover.qcow2
$ ./clover-image.sh --iso ./Clover-v2.5k-5070-X64.iso --cfg ./config/config.plist.stripped.qemu --img ./Clover.qcow2
```

> If you are using NixOS, ensure `libguestfs` and `libguestfs-appliance` are installed. Then, `export LIBGUESTFS_PATH=/nix/store/path-to-libguestfs-applicance-x.xx.x`. Finally, test `libguestfs` with `libguestfs-test-tool`.

### change OVMF resolution

Ensure that OVMF resolution is equal to Clover resolution which is set in above step.

OVMF resolution is changed by following steps:

1. Enter OVMF menu which can be reached with a press of <kbd>ESC</kbd> button during the OVMF boot logo (before Clover boot screen appears).
2. Set `Device Manager > OVMF Platform Configuration > Change Preferred Resolution` to desired resolution.
3. Commit changes and exit the OVMF menu.
4. Relaunch the KVM virtual machine.

## Change MAC address

```sh
# generates QEMU compatible mac addresses
$ printf '52:54:00:AB:%02X:%02X\n' $((RANDOM%256)) $((RANDOM%256))
```

## Update existing VM

Stop the VM, edit the XML file, then redefine the VM.

```sh
$ $EDITOR macOS-libvirt.xml
$ virt-xml-validate macOS-libvirt.xml
$ virsh -c qemu:///system define macOS-libvirt.xml
```

## License

MIT
