# sunpy-lyra-data-pipeline
 
> A minimal working example of fetching and plotting LYRA solar irradiance data using SunPy — built as part of my GSoC application to OpenAstronomy.
 
---
 
## Overview
 
This repository demonstrates how to download and visualize data from the **LYRA** (Lyman Alpha Radiometer) instrument aboard the PROBA2 spacecraft using [SunPy](https://sunpy.org/). Along the way, I encountered and resolved several real-world environment issues — documented here to help others get started faster.
 
---
 
## What This Does
 
- Searches for LYRA data over a given time range using `Fido`
- Downloads the data file using `Fido.fetch`
- Loads it as a `TimeSeries` object
- Plots the solar irradiance over time
 
---
 
## Final Working Code
 
```python
from sunpy.net import Fido, attrs as a
import sunpy.timeseries as ts
 
# Search for LYRA data
result = Fido.search(a.Time('2012/3/4', '2012/3/6'), a.Instrument.lyra)
 
# Fetch the first result (overwrite=True ensures no truncated files)
files = Fido.fetch(result[0, 0], overwrite=True)
 
if len(files) > 0:
    try:
        lyra_ts = ts.TimeSeries(files[0])
        lyra_ts.plot()
    except Exception as e:
        print(f"Error loading file: {e}")
else:
    print("No LYRA data found")
```
 
---
 
## Installation
 
```bash
pip install sunpy h5netcdf
pip install "numpy<1.24"   # only if you hit the numpy.typeDict error (see below)
```
 
Or install all at once with a compatible set of versions:
 
```bash
pip install "sunpy>=4.1" "numpy>=1.24" h5netcdf
```
 
---
 
## Errors Encountered & Fixes
 
This section documents every error I hit during setup, and exactly how I fixed each one. This is the real value of this repo — a clean debugging trail.
 
### 1. `a.Instrument.lyra` is not a data reader
 
**Error:** Using `a.Instrument.lyra.get_files(files[0])` raised an `AttributeError`.
 
**Cause:** `a.Instrument.lyra` is a *search attribute*, not a file loader. It cannot read data files.
 
**Fix:** Use `sunpy.timeseries.TimeSeries` to load downloaded files:
 
```python
# Wrong
spec = a.Instrument.lyra.get_files(files[0])
 
# Correct
import sunpy.timeseries as ts
lyra_ts = ts.TimeSeries(files[0])
```
 
---
 
### 2. `ModuleNotFoundError: No module named 'h5netcdf'`
 
**Error:** Importing `sunpy.timeseries` failed because `h5netcdf` was missing.
 
**Fix:**
```bash
pip install h5netcdf
```
 
---
 
### 3. `AttributeError: module 'numpy' has no attribute 'typeDict'`
 
**Error:** `numpy.typeDict` was removed in NumPy 1.24, but an older SunPy version still referenced it.
 
**Fix (Option A):** Downgrade NumPy:
```bash
pip install "numpy<1.24"
```
 
---
 
### 4. `TypeError: buffer is too small for requested array`
 
**Error:** The downloaded FITS file was truncated (incomplete download), causing NumPy to fail when reading it.
 
**Warning that appeared:**
```
WARNING: File may have been truncated: actual file length (74793640) is smaller than the expected size (77627520)
```
 
**Fix:** Force a fresh download using `overwrite=True`:
```python
files = Fido.fetch(result[0, 0], overwrite=True)
```
 
Also make sure to pass `files[0]` (a single path string) to `TimeSeries`, not the full `files` list.
 
---
 
## Environment
 
| Package | Tested Version |
|---------|---------------|
| Python  | 3.8           |
| SunPy   | ≥ 4.1         |
| NumPy   | < 1.24 or ≥ 1.24 (with updated SunPy) |
| h5netcdf | any recent   |
 
---
 
## About
 
This project is a small contribution made while applying to [Google Summer of Code (GSoC)](https://summerofcode.withgoogle.com/) with [OpenAstronomy](https://openastronomy.org/). The goal was to go beyond tutorials — run real code, break things, fix them, and document the process clearly.
 
**Relevant GSoC organizations:**
- [SunPy](https://sunpy.org/) — Python library for solar physics
- [OpenAstronomy](https://openastronomy.org/) — umbrella org for open-source astronomy software
 
---
 
 ## Next Steps

- Extend this workflow to radio spectrogram data using radiospectra
- Explore integration with NDCube for better coordinate handling
- Contribute improvements to documentation and examples in SunPy/radiospectra

## Resources
 
- [SunPy Documentation](https://docs.sunpy.org/)
- [LYRA Instrument Overview](https://proba2.sidc.be/index.html/science/lyra)
- [OpenAstronomy GSoC Projects](https://openastronomy.org/gsoc/)