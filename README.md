# device_xiaomi_redwood-kernel-6.18

A ready-to-use **Linux 6.18** kernel for the **Redmi Note 12 Pro 5G / POCO
X5 Pro 5G** (`redwood`, SM7325), for building a ROM. Nothing here has to be
compiled: the ROM build just copies these files in.

The kernel is Android Common Kernel `android17-6.18` (`6.18.33-Suvkernel`)
with Qualcomm's lahaina platform carried over from 6.12, built from the
`main` branch of
[redwood-6.18-kernel](https://github.com/suvojeet-sengupta/redwood-6.18-kernel).
The vendor modules come from
[redwood-6.18-modules](https://github.com/suvojeet-sengupta/redwood-6.18-modules).
`build-info.txt` says exactly which build this is. What works and what
doesn't is tracked in
[redwood-6.18-devicetrees](https://github.com/suvojeet-sengupta/redwood-6.18-devicetrees).

## What's inside

| File | What it is |
|---|---|
| `Image` | The kernel |
| `dtb/` | The SoC device tree (the same one the 6.12 prebuilt ships) |
| `dtbo.img` | redwood's own hardware description, laid over the SoC one |
| `vendor_ramdisk/` | Every kernel module, plus `modules.load`, the order they load in |
| `kernel-headers/` | The 5.4 kernel headers. On purpose: the phone's vendor userspace was built against them |
| `build-info.txt` | Kernel version, build date and module count |

## How to use it

Use the `6.18` branch of the redwood device tree. It already points here,
and the `6.18-setup` branch has a manifest that syncs everything for you.

To wire it into another redwood tree:

```make
KERNEL_PATH := device/xiaomi/redwood-kernel-6.18
TARGET_PREBUILT_KERNEL := $(KERNEL_PATH)/Image
TARGET_BOARD_KERNEL_HEADERS := $(KERNEL_PATH)/kernel-headers
BOARD_PREBUILT_DTBOIMAGE := $(KERNEL_PATH)/dtbo.img
BOARD_PREBUILT_DTBIMAGE_DIR := $(KERNEL_PATH)/dtb
BOARD_VENDOR_RAMDISK_KERNEL_MODULES := $(wildcard $(KERNEL_PATH)/vendor_ramdisk/*.ko)
BOARD_VENDOR_RAMDISK_KERNEL_MODULES_LOAD := $(strip $(shell cat $(KERNEL_PATH)/vendor_ramdisk/modules.load))
BOARD_KERNEL_CMDLINE += binder.impl=rust iommu.strict=1 deferred_probe_timeout=-1 redwood.nlmsg_legacy
```

- `binder.impl=rust`: the Rust Binder module is in `modules.load`.
- `iommu.strict=1`: the SMMU flushes its TLB on every unmap until the
  deferred path is fixed.
- `redwood.nlmsg_legacy`: vendor SELinux policy written for 6.12 lacks the
  netlink `allowxperm` rules 6.18 checks; without it netmgrd can't create
  the mobile data links.

## Before you flash

- `/data` is shared between the two slots. Back up your data before trying
  it.
- Don't mix it with a 6.12 or 5.4 boot or vendor_boot image: the modules
  only load on this exact kernel build.
- KernelSU Next's own `kernelsu.ko` doesn't load on this kernel (other
  vermagic); it has to be built against `redwood-6.18-kernel`.
