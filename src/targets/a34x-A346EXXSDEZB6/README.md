# a34x-A346EXXSDEZB6

Exact target profile for Samsung Galaxy A34 5G:

```text
model: SM-A346E
device: a34x
firmware: A346EXXSDEZB6
display build: BP2A.250605.031.A3.A346EXXSDEZB6
fingerprint: samsung/a34xdxx/a34x:16/BP2A.250605.031.A3/A346EXXSDEZB6:user/release-keys
kernel: 6.6.82-android15-8-abA346EXXSDEZB6-4k
SoC: MediaTek MT6877
```

`target.h` and `p0_fingerprint.h` were derived from the exact boot image and
recovered `vmlinux.elf` for this firmware. The profile uses the physical P0
oracle path and converts raw oracle offsets to the real KASLR/P0 slide.

Hardware status:

- app-domain payload/root daemon: build complete, execution pending on the
  connected SM-A346E;
- KernelSU module: exact-vermagic `android15-6.6` build with KDP/RKP/DEFEX,
  target-symbol audit passed (0 CRC mismatches);
- KernelSU late-load: build complete, load test pending on hardware;
- see `docs/SM-A346E-A346EXXSDEZB6.md`.
