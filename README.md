# ExifScrub

![CI Status](https://github.com/16lv/exifscrub/actions/workflows/ci.yml/badge.svg)

**ExifScrub** is a Python 3.12 utility built with **Pillow (PIL)** that inspects image dimensions, formats, colour modes, EXIF tags, and embedded GPS IFD blocks (`34853`), and creates scrubbed copies with metadata removed.

## Why I Built It

Photos taken on phones and digital cameras routinely embed device serial numbers, timestamps, software strings, and GPS coordinates in EXIF headers. Many simple scripts overwrite files in place or accidentally carry cached metadata dictionaries over when calling `img.save()`. I built ExifScrub to inspect standard and GPS EXIF tags cleanly and write pixel data onto a fresh image canvas while refusing to overwrite the original file.

## Key Features

- **Structured EXIF & GPS Inspection (`inspect_image`)**: Reads image dimensions, colour mode, format, file size, decoded `ExifTags.TAGS`, and nested `ExifTags.GPSTAGS` (IFD `34853`) without treating missing EXIF blocks as an error.
- **Non-Destructive Metadata Stripping (`strip_metadata_to_copy`)**: Copies raw pixel values onto a fresh `Image.new(img.mode, img.size)` canvas so cached EXIF/GPS dictionaries are never written to the output file.
- **Source File Protection**: Compares resolved canonical paths (`source_path.resolve() == output_path.resolve()`) and raises a `ValueError` if the caller attempts to overwrite the original image in place.

## Usage Example

```python
from pathlib import Path
from src.image_metadata.inspector import inspect_image, strip_metadata_to_copy

report = inspect_image(Path("photo.jpg"))
print(f"{report.format} {report.width}x{report.height} | EXIF: {report.has_exif} | GPS: {report.has_gps_info}")

summary = strip_metadata_to_copy(Path("photo.jpg"), Path("photo_clean.jpg"))
print(f"Removed {summary['removed_exif_field_count']} EXIF fields -> {summary['output_path']}")
```

## Running Tests & Linter

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```

## Known Limitations & Future Improvements

- `strip_metadata_to_copy` currently re-encodes raster images via `Image.new` + `paste`, which strips EXIF/GPS tags reliably but recompresses lossy JPEGs; lossless JPEG APP1 marker stripping would avoid re-encoding pixel blocks.
