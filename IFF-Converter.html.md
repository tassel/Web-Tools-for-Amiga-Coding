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
