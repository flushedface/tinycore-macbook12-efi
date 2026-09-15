# TinyCore EFI image for Intel MacBooks

This project creates a portable GPT/UEFI USB image for Intel Macs. It downloads the official 64-bit `TinyCorePure64-17.1.iso`, extracts its kernel and core filesystem, wraps them in a standalone x86_64 GRUB EFI loader, and creates a USB image with an EFI System Partition plus a persistent `tce` partition.

## Keyboard support

TinyCore 17's x86_64 kernel configuration enables `CONFIG_KEYBOARD_APPLESPI=y`. That is the in-kernel driver for the integrated keyboard and trackpad on 12-inch Retina MacBooks (`MacBook8,1`, `MacBook9,1`, and `MacBook10,1`). No out-of-tree Apple SPI driver repository or submodule is required.

The image has `tce=LABEL=tce` in its GRUB configuration. Installed TinyCore extensions go under `/tce/optional`; add package names to `/tce/onboot.lst` and use `filetool.sh -b` to persist configuration changes.

## Build with GitHub Actions

1. Open **Actions** in this repository.
2. Select **Build TinyCore EFI image**.
3. Select **Run workflow**.
4. When it completes, download the `tinycore-macbook-efi` artifact.

The artifact contains:

- `tinycore-macbook-efi.img.gz` — the compressed USB disk image.
- `tinycore-macbook-efi.img.gz.sha256` — its SHA-256 checksum.

## Flash the image

Flashing overwrites the selected USB drive. Use a USB drive of at least 1 GB.

### Linux

```bash
gunzip -c tinycore-macbook-efi.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
sync
```

Replace `/dev/sdX` with the whole USB device, not a partition such as `/dev/sdX1`.

### macOS

```bash
gunzip -c tinycore-macbook-efi.img.gz | sudo dd of=/dev/rdiskN bs=4m
sync
```

First identify the removable target using `diskutil list`, unmount it with `diskutil unmountDisk /dev/diskN`, then replace `rdiskN` carefully. `dd` erases the target drive.

## Boot on a MacBook

1. Insert the USB image.
2. Start the Mac while holding **Option** (Alt).
3. Choose **EFI Boot**.
4. GRUB starts TinyCore with the persistent partition selected by its `tce` filesystem label.

## Repository dependencies

No git submodules are needed. The Apple SPI driver is part of modern Linux/TinyCore kernel configuration, and GRUB is built from the Ubuntu GitHub Actions runner package. The workflow uses the TinyCore project's official ISO as the runtime source.

## Scope

This is intended for 64-bit Intel EFI MacBooks. It does not apply to Apple-silicon Macs, which need ARM64 Linux distributions and a different boot path.
