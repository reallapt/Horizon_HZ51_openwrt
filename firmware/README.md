# Firmware components

`Horizon_HZ51_OpenWrt_firmware-components.zip` contains four volumes extracted from the inactive `rootfs_1` UBI partition:

- `kernel.bin`
- `ubi_rootfs.bin`
- `wifi_fw.bin`
- `bt_fw.bin`

The writable `rootfs_data` volume and every device-specific partition were excluded because the source image contains live configuration, credentials and SSH host keys.

These files are raw recovery/development components. They are not a LuCI or `sysupgrade` image. Do not flash them without a verified partition map, a complete backup and a working serial recovery path.
