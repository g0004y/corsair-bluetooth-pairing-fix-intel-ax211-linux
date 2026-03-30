# Intel AX211 Bluetooth (Corsair Pairing Fix)

> Firmware fix for Intel AX211 CNVi Bluetooth failing to pair with Corsair peripherals on Linux.
> Tested on Linux Mint 22.3 Zena (kernel 6.8). Issue and fix may apply to other distributions.

---

## Background

I own a Lenovo ThinkPad T14 Gen 3 (Intel) with an Intel AX211 CNVi wireless card. After buying Corsair peripherals (keyboard, mouse), I was unable to pair them over Bluetooth on Linux Mint. Pairing attempts would fail silently or drop immediately after connecting.

After months of troubleshooting, the fix turned out to be simple — replace the AX211 firmware files.

The same pairing issue also exists on **Windows 10** with the default Intel BT driver. The driver version from Intel's website doesn't fix Corsair pairing either, but a specific driver version resolves it there as well.

---

## Symptoms

- Corsair peripherals (keyboard, mouse) fail to pair over Bluetooth
- Pairing appears to succeed but the device immediately disconnects
- `bluetoothctl pair <MAC>` times out or fails silently
- Other Bluetooth devices may work fine, but Corsair-specific devices fail consistently
- The problem persists across kernel versions and Bluetooth stack restarts


#### To view the FIX and GUIDE, [download the HTML file](https://github.com/g0004y/corsair-bluetooth-pairing-fix-intel-ax211-linux/releases/download/v1.0/bt-ax211-firmware.html) and open with your preferred browser
