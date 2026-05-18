# mkv-strip

A fast, pure-Rust command-line utility to strip, extract, and add subtitle tracks in MKV (Matroska) files — **no FFmpeg or external tools required**.

Built on the [`mkv-element`](https://crates.io/crates/mkv-element) crate for native EBML/Matroska parsing and writing.

```
mkv-strip 0.1.0 (2026-05-18)

Created by Digital Futures Consultancy LLP (Singapore) - https://DigitalFutures.Asia
```

## Features

| Command | Description |
|---------|-------------|
| `list` | Inspect an MKV file — show all tracks with type, language, codec, and flags |
| `strip` | Remove audio/subtitle tracks by language or type, output a clean `.mkv` |
| `extract` | Pull subtitle tracks out to `.srt` files |
| `add` | Inject an `.srt` file as a new subtitle track into an MKV |

## Download

Pre-built binaries for **Linux x64** and **Windows x64** are available in the [`binaries/`](https://github.com/stevenyy88/mkv-strip/tree/main/binaries) directory.

| File | Platform | Size |
|------|----------|------|
| `mkv-strip-linux-x64` | Linux (x86-64) | ~1.8 MB |
| `mkv-strip-windows-x64.exe` | Windows (x86-64) | ~2.2 MB |

No installer needed — just download and run.

## Quick Start

### List tracks
```bash
mkv-strip list movie.mkv
```
```
  # │ Type      │ Lang │ Flags            │ Name │ Codec
────┼───────────┼──────┼──────────────────┼──────┼──────────────
  1 │ video     │ und  │ enabled          │      │ V_MPEG4/ISO/AVC
  2 │ audio     │ eng  │ default, enabled │      │ A_AC3
  3 │ audio     │ jpn  │ enabled          │      │ A_AC3
  4 │ subtitle  │ eng  │ default, enabled │      │ S_TEXT/UTF8
  5 │ subtitle  │ spa  │ enabled          │      │ S_TEXT/UTF8
```

### Strip tracks
Keep only English and Japanese audio, remove all subtitles:
```bash
mkv-strip strip -i movie.mkv -o movie_stripped.mkv --keep-audio eng,jpn --no-subtitle
```
Remove specific subtitle languages:
```bash
mkv-strip strip -i movie.mkv -o movie_stripped.mkv --remove-subtitle spa
```

### Extract subtitles to SRT
Extract all subtitle tracks:
```bash
mkv-strip extract -i movie.mkv
```
Filter by language or track number:
```bash
mkv-strip extract -i movie.mkv --lang eng,spa
mkv-strip extract -i movie.mkv -t 4,5
mkv-strip extract -i movie.mkv -o ./subs
```
Output files are named like `movie.4.eng.srt`, `movie.5.spa.srt`.

### Add an SRT subtitle track
```bash
mkv-strip add -i movie.mkv -s subs.srt -o movie_with_subs.mkv --lang eng
mkv-strip add -i movie.mkv -s subs.srt --lang eng --name "English (SDH)" --default
mkv-strip add -i movie.mkv -s forced.srt --lang eng --forced
```

## Supported Subtitle Codecs

| Extraction | Add |
|-----------|-----|
| `S_TEXT/UTF8` → `.srt` | `.srt` → `S_TEXT/UTF8` |
| `S_TEXT/SSA` → `.srt` | |
| `S_TEXT/ASS` → `.srt` | |

Image-based subtitles (VobSub, HDMV PGS) are not supported for extraction.

## How It Works

- **list / extract** — Uses `MatroskaView` to parse metadata without loading entire clusters into memory
- **strip** — Two-pass: metadata scan first, then full re-read with block-level track filtering
- **add** — Parses SRT timestamps, converts to MKV segment ticks, builds SimpleBlock elements, appends new TrackEntry + clusters

## Limitations

- **Memory**: Clusters are loaded into memory during processing; very large files may use significant RAM
- **SeekHead / Cues**: Dropped from output — most players rebuild these automatically
- **Multi-segment files**: Not yet supported (rare in practice)
- **Track renumbering**: Track numbers are preserved as-is

## Build from Source

```bash
git clone https://github.com/stevenyy88/projects.git
cd projects/mkv-strip  # or your local source directory
cargo build --release
```

Cross-compile for Windows:
```bash
cargo zigbuild --release --target x86_64-pc-windows-gnu
```

## License

MIT — Created by [Digital Futures Consultancy LLP (Singapore)](https://digitalfutures.asia)
