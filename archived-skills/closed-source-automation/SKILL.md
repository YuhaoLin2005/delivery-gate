---
name: closed-source-automation
description: 闭源科学软件自动化通用方法论——ENVI/ArcGIS/ERDAS等GUI软件的全自动管道
---

# Closed-Source Scientific Software Automation Skill

> 从遥感实习（ENVI 5.3 + ArcGIS 10.8）全自动化实战中提炼的通用模式。适用于任何闭源、GUI-centric科学软件。

## When to Use
When you need to automate a task involving closed-source, GUI-centric scientific/engineering software (ENVI, ArcGIS, ERDAS, AutoCAD, etc.) that doesn't have a well-documented public API.

## Core Methodology (6 Steps)

### 1. Discover the Hidden CLI
Every GUI software has a back door:
- **Batch/headless mode**: Look for `-batch`, `-cli`, `-e`, or `--headless` flag
- **Scripting console**: Many tools have built-in scripting (IDL console, ArcPy Python window, MATLAB engine)
- **COM/OLE automation**: Windows software often exposes COM interfaces (`win32com`)
- **DLL/SO libraries**: The GUI is just a wrapper — find the underlying `.dll` or `.so`

### 2. Bridge Missing Features with Open-Source
When the CLI lacks a feature the GUI has:
- **GDAL/OGR**: Universal raster/vector processing — `gdalwarp`, `gdal_translate`, `ogr2ogr`, `gdal_calc.py`
- **numpy/scipy**: Array processing, statistics, classification
- **Orfeo Toolbox (OTB)**: Remote sensing classification
- **SAGA/GRASS**: GIS analysis
- Build chains: `ENVI(load+sensor) → GDAL(reproject+clip) → ArcPy(warp+stats)`

### 3. Platform-Specific Fixes
**Encoding & Path Issues:**
- Windows CLI often garbles Unicode paths → use ASCII-only paths, copy files to temp
- Python 2.7 (ArcGIS): requires `# -*- coding: utf-8 -*-`, no f-strings, forward slashes
- IDL/Python 2.7: avoid `r'path\中文'`, use `'path'` with forward slashes

**Memory Management:**
- Always check disk space first
- Process in tiles/strips, never load full image at once
- Free memory explicitly: `del var; gc.collect()` in Python, `var = !NULL` in IDL
- Use `BIGTIFF=YES` for >4GB outputs

### 4. Auto-Iterate Until Success
```
LOOP:
  run script → capture stderr+stdout → parse errors →
  fix the specific issue → rerun → repeat until success
```
**Never ask the user to fix errors.** Read the error, understand it, fix it, rerun.

### 5. Build Fallback Chains
Always have Plan B, C, D:
```
Tool A (primary) → if fails → Tool B → if fails → Tool C
```
Example: `ENVI IDL → Python GDAL → Python numpy → ArcPy`

### 6. API Discovery for Undocumented Tools
- **ENVI IDL**: `HELP, task, /STRUCTURE` shows all parameters
- **ArcPy**: `dir(arcpy.management)`, `arcpy.Usage('tool_name')`
- **GDAL**: `gdalinfo --help`, `gdalwarp --help`
- When parameter names differ between versions, probe with a test script

## Tool-Specific Cheat Sheet

### ENVI 5.3 + IDL 8.5
```
idl.exe script.pro                    # Batch execution (not envi_idl.exe)
ENVI(/HEADLESS)                       # No-GUI mode
e.OpenRaster('file')                  # Open raster
ENVITask('TaskName')                  # Create task
task.PARAMETER_NAME = value           # Set parameters (check exact names!)
HELP, task, /STRUCTURE                # Discover parameters
raster.GetData(BAND=n)                # Read single band (memory efficient)
raster.Export, 'path', 'ENVI'         # Save result
```

### ArcGIS 10.8 / ArcPy (Python 2.7)
```
"C:/Python27/ArcGIS10.8/python.exe" script.py
arcpy.Exists(path)                    # Check if dataset exists
arcpy.ListFields(path)                # Discover field names
arcpy.da.SearchCursor(path, fields)   # Read attributes
arcpy.management.WarpFromFile(...)    # GCP-based warp (POLYORDER1)
arcpy.sa.MajorityFilter(...).save()   # Clump equivalent
```

### GDAL (QGIS)
```
gdalwarp -t_srs prj -cutline shp -crop_to_cutline -r cubic
gdal_translate -of ENVI
gdal_sieve.py -st 9                   # Sieve filter
gdal_polygonize.py -f "ESRI Shapefile"
gdallocationinfo -valonly             # Extract pixel values at coordinates
```

## Anti-Patterns
- ❌ Asking the user to interact with GUI ("now click File → Open...")
- ❌ Giving up after one tool fails — always have a fallback
- ❌ Loading full raster into memory (use band-by-band or tiled processing)
- ❌ Hardcoding paths with Chinese/Unicode characters in string literals
- ❌ Using `r'C:\path'` raw strings with backslashes in Python 2.7 arcpy
