# Port to Linux Kernel 6.18 HDA Codec API

This patch updates the out-of-tree CS8409 codec driver to compile against
Linux kernel 6.18+, which introduced breaking changes to the HDA codec API.

## Changes

### API Removals in Kernel 6.18

- **`hda_codec_ops.free`** field was removed; replaced by **`.remove`**
- **`hda_codec.patch_ops`** field was removed; codec ops are now set at the
  driver level via `hda_codec_driver.ops`
- **`HDA_CODEC_ENTRY()`** macro was removed; replaced by **`HDA_CODEC_ID()`**
- **`snd_hda_gen_free()`** was renamed to **`snd_hda_gen_remove()`**

### How This Patch Addresses Them

- A new `codec_ops` pointer was added to `struct cs8409_spec` to store
  per-codec ops (Dell vs Apple paths use different callbacks)
- Driver-level dispatch functions (`cs8409_dispatch_*`) forward calls to the
  per-codec ops stored in `spec->codec_ops`
- A `.probe` callback wraps the existing `patch_cs8409()` function
- All `.free` references updated to `.remove`
- All `snd_hda_gen_free` calls updated to `snd_hda_gen_remove`

## Build Instructions

The kernel must be built with the LLVM toolchain (CachyOS default). Use:

```bash
make CC=clang LD=ld.lld
sudo make CC=clang LD=ld.lld install
```

## Tested On

- CachyOS with kernel 6.18.8-3-cachyos
- MacBook Pro 13,1 (CS8409 + CS42L83)
