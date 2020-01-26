# Install macOS with KVM

## Change Screen Resolution

### determine your desired resolution

The default resolution is 1024x768 which is widely supported. Suppose that your desired resolution is 1920x1080.

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
