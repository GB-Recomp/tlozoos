# tlozoos

Static recompilation of **The Legend of Zelda: Oracle of Seasons** (USA,
Australia) into portable C, built with
[GB-Recomp/gb-recompiled](https://github.com/GB-Recomp/gb-recompiled).

Symbol names come from the [Stewmath/oracles-disasm](https://github.com/Stewmath/oracles-disasm)
WLA-DX disassembly.

## Build

```sh
mkdir build && cd build
cmake ..
cmake --build . -j$(nproc)
```

Produces a `tlozoos` executable (~560 KB with the default MinSizeRel +
dead-strip profile).

## Required ROM

This build is locked to a single revision:

| Region | Filename in the wild | MD5 |
|--------|----------------------|-----|
| USA / Australia | Legend of Zelda, The - Oracle of Seasons (USA, Australia).gbc | `f2dc6c4e093e4f8c6cbea80e8dbd62cb` |

The asset loader verifies the SHA-1 on first launch and refuses to run
on any other dump.

## Run

Drop your Oracle of Seasons ROM next to the executable as `roms/tlozoos.gbc`:

```sh
mkdir -p roms
cp '/path/to/Legend of Zelda, The - Oracle of Seasons (USA, Australia).gbc' roms/tlozoos.gbc
./tlozoos
```

First boot extracts the ROM into `assets/tlozoos/rom.bin`; the source ROM
isn't needed after that. Press Esc for the runtime menu.

## Notes

- The decomp's Seasons build doesn't produce a byte-matching ROM (empty-
  fill quirks); symbols still align with the official ROM because code
  layout is identical. SHA-1 in the asset loader is computed from the
  user-supplied ROM, so the manifest check works either way.
- The asset manifest is a single opaque section for now. The decomp has
  pret-style asset directories that a future pass can split out.
- The companion game (Oracle of Ages) lives at
  [GB-Recomp/tlozooa](https://github.com/GB-Recomp/tlozooa).
