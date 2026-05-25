# A Web version of the Kefrens IFF-Converter.

<img width="1220" height="911" alt="image" src="https://github.com/user-attachments/assets/974bcb2b-cc77-4718-ab41-5282c0f6ff52" />

## How to use it
Press **LOAD IMG** and open a PNG/GIF/JPG — or a genuine .iff/.lbm from the Amiga days (the loader handles BMHD/CMAP/CAMG/BODY, decompresses ByteRun1, and even decodes EHB and HAM6/HAM8).
Drag the crosshair over the area you want, set DEPTH to the number of bitplanes (1–8, all the way up to AGA's 256 colors), and press **SAVE RAW**.

## What it produces
**SAVE RAW** gives you pure planar bitplane data, big-endian (68k), with each line word-aligned (16px) just the way an Amiga wants it. You choose the layout in the dropdown: "Planar" (plane-by-plane, standard RAW) or "Interleaved" (line-by-line,the way the BODY in an IFF is laid out).
**SAVE PAL** gives you the palette either as 12-bit $0RGB words (OCS/ECS) or 24-bit RGB triples (AGA), controlled by the PALETTE selector.
**SAVE IFF** writes a valid uncompressed ILBM back out, and COPY ASM puts the palette plus size constants (_WIDTH, _BPR, _PLANESZ, _RAWSIZE, dc.w/dc.l palette) on the clipboard as an assembler include.

## A few details worth knowing
**SNAP-16** rounds the width to the nearest 16 pixels automatically (turn it off if you want free width — the RAW gets padded to the word boundary regardless). Images with more colors than 2^DEPTH are quantized using median-cut; if the image already has few enough colors, the palette is kept exactly.
**PREVIEW** shows you the quantized result with a palette strip and RAW size before you save, and CMAP ON/OFF controls whether palette saving is active. The status line up top and the dims line at the bottom show rowbytes and total RAW size throughout.
**Two things I deliberately left out :** HAM/EHB reading works (so you can open old images), but there is no export to HAM/HAM8, since that encoding pipeline is a whole separate matter — The same goes for ByteRun1-compressed IFF saving (right now it writes uncompressed to be safe).


---
# CHANGELOG:

## Decoupled ASM export

- **`COPY ASM` now always emits 12-bit `dc.w $RGB`**, regardless of the PALETTE
  dropdown. Previously the ASM format followed the dropdown, which meant you could
  unintentionally get 24-bit `dc.l` output when you actually wanted 12-bit. That
  coupling is gone entirely — the ASM palette is Amiga color-register format every
  time.

```
name_pal:
    dc.w $fff  ; col 0
    dc.w $848  ; col 1
    ...
    dc.w $000  ; col 7
```

## Correct default palette format

- **12-bit `$RGB` (`dc.w`) is now the default choice** in the PALETTE dropdown.
  The 24-bit `dc.l` table (AGA) is still there as the second option for anyone who
  needs it.
- **Six hex digits instead of six** in the ASM output: `$fff` instead of `$ffffff`
  (the leading zero nibble is dropped — the assembled value is identical).

## Palette bug fixed

- **Fixed the 8-bit to 4-bit color-channel conversion.** The old code used
  `(v+8)>>4 & 0xf`, which assumes a /16 scale and masks with `& 0xf`. Values in the
  248–255 range then wrapped around to 0 — white (255,255,255) came out as `$000`
  (black). The Amiga channel is on a /17 scale (255/15 = 17), so the conversion was
  changed to `round(v/17)`, which caps naturally at 15 with no wraparound.
- The bug affected all three places that produce 12-bit values: `SAVE PAL`,
  `COPY ASM`, and the internal 12-bit preview. They now all use the same correct
  `rgb4()` helper function.

  ## Known / deliberately left out

- **HAM/EHB export** is not in yet — loading such images works, but output is
  written as ordinary bitplanes. A dedicated encoding pipeline is needed for export.
- **IFF saving is uncompressed** (no `ByteRun1` packing on the way out, to stay on
  the safe side).

## Possible next steps

- Sprite/blitter format with 16 px interleaving.
- EHB export.
- An "append several tiles into one RAW file" mode for tile/font data.
- `ByteRun1`-compressed IFF saving.
