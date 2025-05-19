# Building TWRP for Poco X3 Pro (vayu) on Ubuntu

This guide provides step-by-step instructions to build TWRP (Team Win Recovery Project) for the Poco X3 Pro (codename: vayu) on a Ubuntu system. TWRP is a custom recovery that allows flashing custom ROMs, creating backups, and performing advanced operations on Android devices.

## Prerequisites

- **Operating System**: 64-bit Ubuntu (20.04 or newer recommended).
- **Hardware**: At least 8GB RAM, 100GB free disk space, and a decent internet connection.
- **Device**: Poco X3 Pro with an **unlocked bootloader**. Unlocking wipes data, so back up your device.
- **Knowledge**: Familiarity with Linux terminal commands and Android building processes.

## Step 1: Install Required Packages

Update your system and install the necessary tools for building TWRP.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl bison build-essential ccache flex libesd0-dev libssl-dev libxml2-utils openjdk-8-jdk python3 python3-distutils python3-dev python3-pip rsync schedtool squashfs-tools uuid-dev zlib1g-dev
```

Install the `repo` tool to manage source code:

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

## Step 2: Clone TWRP Source Code

Create a working directory and clone the minimal TWRP manifest for Android 11 (twrp-11.0 branch).

```bash
mkdir ~/twrp
cd ~/twrp
repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-11.0
repo sync -c -j8 --no-clone-bundle --no-scheduler
```

**Note**: The `repo sync` command may take time depending on your internet speed. The `--no-clone-bundle` flag helps reduce download issues.

## Step 3: Add Device Tree for Poco X3 Pro (vayu)

Clone the device tree for vayu from the TeamWin repository.

```bash
mkdir -p device/xiaomi
git clone https://github.com/TeamWin/android_device_xiaomi_vayu.git device/xiaomi/vayu
```

Verify that the `device/xiaomi/vayu` directory contains files like `Android.mk`, `BoardConfig.mk`, and `device.mk`.

## Step 4: Set Up Build Environment

Initialize the build environment and select the vayu configuration.

```bash
source build/envsetup.sh
lunch omni_vayu-eng
```

The `lunch` command sets up the build for Poco X3 Pro in engineering mode (`-eng`).

## Step 5: Build TWRP Recovery

Compile the recovery image using the following command:

```bash
mka recoveryimage
```

This process may take 30 minutes to several hours, depending on your system's performance.

## Step 6: Locate the Built Recovery Image

Once the build completes successfully, the recovery image will be located at:

```
out/target/product/vayu/recovery.img
```

Verify its presence:

```bash
ls out/target/product/vayu/recovery.img
```

## Step 7: Install TWRP on Your Device

1. **Ensure Bootloader is Unlocked**: If not, enable Developer Options, USB Debugging, and OEM Unlock in your device settings, then run:

   ```bash
   adb reboot bootloader
   fastboot flashing unlock
   ```

   **Warning**: Unlocking the bootloader wipes all data on the device.

2. **Flash the Recovery Image**:

   ```bash
   cd ~/twrp/out/target/product/vayu
   fastboot flash recovery recovery.img
   fastboot reboot recovery
   ```

Your device should boot into TWRP recovery. Test basic functions like backup and file management.

## Troubleshooting

- **Build Errors**: Check terminal logs for missing dependencies or incorrect device tree files. Ensure all packages are installed and the device tree is properly cloned.
- **TWRP Not Booting**: Verify the recovery image was flashed correctly and the bootloader is unlocked.
- **Slow Sync**: If `repo sync` is slow, reduce the `-j` value (e.g., `-j4`) or check your internet connection.

## Additional Notes

- **Backup**: Always back up your device data before flashing.
- **Updates**: To use a newer TWRP version, check for updated branches in the [TWRP manifest repository](https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni).
- **Support**: For issues, refer to the [TeamWin vayu device tree](https://github.com/TeamWin/android_device_xiaomi_vayu) or XDA forums for community support.

## References

- [TWRP Minimal Manifest](https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni)
- [Poco X3 Pro Device Tree](https://github.com/TeamWin/android_device_xiaomi_vayu)
- [TWRP Build Guide](https://github.com/twrpdtgen/twrpdtgen/wiki/4.-Build-TWRP-from-source)
