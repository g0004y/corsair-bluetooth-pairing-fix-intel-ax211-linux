# Intel AX211 Bluetooth — Corsair Pairing Fix

**Firmware fix for Intel AX211 CNVi Bluetooth failing to pair with Corsair peripherals on Linux and Windows.** Tested on Linux Mint 22.3 Zena (kernel 6.8). Issue and fix may apply to other distributions.

---

## Background

I own a Lenovo ThinkPad T14 Gen 3 (Intel) with an Intel AX211 CNVi wireless card. After buying Corsair peripherals (keyboard, mouse), I was unable to pair them over Bluetooth on Linux Mint. Pairing attempts would fail silently or drop immediately after connecting.

After months of troubleshooting, the fix turned out to be simple — replace the AX211 firmware files.

The same pairing issue also exists on Windows 10 with the default Intel BT driver. The driver version from Intel's website doesn't fix Corsair pairing either, but a specific driver version resolves it there as well.

---

## Symptoms

- Corsair peripherals (keyboard, mouse) fail to pair over Bluetooth
- Pairing appears to succeed but the device immediately disconnects
- `bluetoothctl pair <MAC>` times out or fails silently
- Other Bluetooth devices may work fine, but Corsair-specific devices fail consistently
- The problem persists across kernel versions and Bluetooth stack restarts

---

## Root Cause

The Intel AX211 requires two firmware files:

- `ibt-0040-0041.sfi` — main firmware blob
- `ibt-0040-0041.ddc` — device-specific calibration parameters (DDC)

The default firmware bundled with Linux Mint / Ubuntu for the AX211 is version 2023.13. This version works for basic Bluetooth operation but lacks the calibration parameters needed to maintain stable connections with certain devices — particularly Corsair peripherals which have stricter timing and power requirements.

Two possibilities for why this happens:

1. **Kernel + firmware version mismatch** — newer kernels (e.g. 6.x) enforce stricter AX211 firmware requirements, and the older bundled firmware doesn't meet them
2. **Hardware-specific calibration** — Corsair devices require exact DDC parameters that the default firmware doesn't provide

The fixed firmware files are version ~60 and include updated calibration data that resolves Corsair pairing.

---

## The Fix — Linux

### Step 1 — Get the Firmware Files

Download `linux-fix.tar.gz` from the [GitHub releases](https://github.com/g0004y/corsair-bluetooth-pairing-fix-intel-ax211-linux/releases).

Extract the archive to get `ibt-0040-0041.sfi` and `ibt-0040-0041.ddc`.

### Step 2 — Verify Current Firmware Version

Before replacing, check what firmware is currently loaded:

```bash
dmesg | grep -i "ibt-0040-0041"
```

You should see something like:

```
Bluetooth: hci0: Found device firmware: intel/ibt-0040-0041.sfi
Bluetooth: hci0: Firmware Version: 98-13.23
```

This confirms version 98-13.23 (the default) is in use.

### Step 3 — Backup Original Files

```bash
sudo cp /lib/firmware/intel/ibt-0040-0041.sfi /lib/firmware/intel/ibt-0040-0041.sfi.bak
sudo cp /lib/firmware/intel/ibt-0040-0041.ddc /lib/firmware/intel/ibt-0040-0041.ddc.bak
```

### Step 4 — Replace the Firmware Files

Copy the firmware files to `/lib/firmware/intel/`:

```bash
sudo cp ibt-0040-0041.sfi /lib/firmware/intel/
sudo cp ibt-0040-0041.ddc /lib/firmware/intel/
```

### Step 5 — Rebuild Initramfs

This ensures the new firmware is included in the initramfs and loaded at boot:

```bash
sudo update-initramfs -u -k all
```

### Step 6 — Reboot

```bash
sudo reboot
```

### Step 7 — Verify the Firmware Loaded

After reboot, check the loaded firmware version:

```bash
dmesg | grep -i bluetooth | grep -i "firmware\|fseq"
```

Expected output:

```
Bluetooth: hci0: Found device firmware: intel/ibt-0040-0041.sfi
Bluetooth: hci0: Firmware loaded in ... usecs
Bluetooth: hci0: Found Intel DDC parameters: intel/ibt-0040-0041.ddc
Bluetooth: hci0: Applying Intel DDC parameters completed
Bluetooth: hci0: Fseq status: Success (0x00)
```

The key indicators of success:

- `Fseq status: Success (0x00)` — firmware calibration loaded correctly
- `Applying Intel DDC parameters completed` — DDC file is being used
- `Firmware timestamp` should show a newer version

### Step 8 — Test Pairing

Pair your Corsair device again:

```bash
bluetoothctl
[bluetooth]# power on
[bluetooth]# scan on
[bluetooth]# pair <DEVICE_MAC>
[bluetooth]# connect <DEVICE_MAC>
[bluetooth]# trust <DEVICE_MAC>
```

The device should now pair and stay connected.

### Reverting the Fix

If something goes wrong, restore the original firmware:

```bash
sudo cp /lib/firmware/intel/ibt-0040-0041.sfi.bak /lib/firmware/intel/ibt-0040-0041.sfi
sudo cp /lib/firmware/intel/ibt-0040-0041.ddc.bak /lib/firmware/intel/ibt-0040-0041.ddc
sudo update-initramfs -u -k all
sudo reboot
```

---

## Firmware File Locations

| File | Purpose |
| --- | --- |
| `ibt-0040-0041.sfi` | Main AX211 firmware blob — contains Bluetooth baseband and radio firmware |
| `ibt-0040-0041.ddc` | Device-specific calibration data — tuning parameters for power, timing, and frequency response |

---

## Compatibility

This fix has been tested on:

| Component | Version |
| --- | --- |
| OS | Linux Mint 22.3 Zena (Ubuntu Noble base) |
| Kernel | 6.8.0-106-generic |
| Bluetooth adapter | Intel AX211 CNVi |
| Affected devices | Corsair peripherals (keyboard, mouse) |

The fix may apply to other distributions and kernel versions. The firmware is generally compatible with any Linux system using the standard firmware loading mechanism.

---

## The Fix — Windows 10

The same Corsair pairing issue also affects **Windows 10** with the Intel AX211 Bluetooth driver. The default driver from Intel's website or Windows Update does not fix Corsair device pairing.

The specific driver version `BT-22.130.0-32-64UWD-Win10-Win11.exe` resolves the issue.

### Step 1 — Uninstall the Current Driver

Before installing the new driver, remove the existing Intel Bluetooth driver completely:

1. Open **Device Manager**
2. Find **Bluetooth** → **Intel(R) Wireless Bluetooth(R)**
3. Right-click → **Uninstall device**
4. Check **"Attempt to remove the driver software for this device"** → click **Uninstall**
5. Repeat for any duplicate or residual Intel BT entries under Bluetooth
6. Reboot Windows

### Step 2 — Install the New Driver

After reboot:

1. Download `BT-22.130.0-32-64UWD-Win10-Win11.exe` from the [GitHub releases](https://github.com/g0004y/corsair-bluetooth-pairing-fix-intel-ax211-linux/releases)
2. Run the installer as **Administrator**
3. Follow the on-screen prompts to complete installation
4. Reboot if prompted

After installation, Corsair peripherals should pair and stay connected.

---

## Notes

- The firmware files are loaded by the kernel's `btintel` driver at boot. No userspace changes are needed.
- If your kernel uses compressed firmware (`.zst` files), ensure you also have those in place. The `update-initramfs` step handles this automatically.
- Bluetooth pairing behavior may improve for non-Corsair devices as well after the firmware update, as the DDC calibration data is more comprehensive.
- Both the Linux firmware fix and Windows driver fix address the same root cause — the default calibration data for the AX211 is insufficient for Corsair devices.

---

> **Tested on:** Linux Mint 22.3 Zena · kernel 6.8.0-106-generic · Intel AX211 CNVi · Windows 10
