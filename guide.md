# Membrane Science Analyzer v2.0
## User Guide

> **SEM / TEM · Porosity · Fouling · Active Site Analysis**  
> Version: 2.0 · Python 3.8+

---

## Table of Contents

1. [Overview](#1-overview)
2. [Installation](#2-installation)
3. [Interface Layout](#3-interface-layout)
4. [Loading Images](#4-loading-images)
5. [Scale Calibration](#5-scale-calibration)
6. [Analysis Modes](#6-analysis-modes)
   - 6.1 [Porosity Analysis](#61-porosity-analysis)
   - 6.2 [Fouling Layer Analysis](#62-fouling-layer-analysis)
   - 6.3 [Active Site / Catalyst Analysis](#63-active-site--catalyst-analysis)
   - 6.4 [Comparative Analysis](#64-comparative-analysis)
7. [Parameters](#7-parameters)
8. [Results and Charts](#8-results-and-charts)
9. [Export](#9-export)
10. [Keyboard Shortcuts](#10-keyboard-shortcuts)
11. [Supported File Formats](#11-supported-file-formats)
12. [Metric Reference](#12-metric-reference)
13. [Troubleshooting](#13-troubleshooting)
14. [Tips and Best Practices](#14-tips-and-best-practices)

---

## 1. Overview

**Membrane Science Analyzer** is an open-source desktop application for quantitative analysis of SEM (Scanning Electron Microscopy) and TEM (Transmission Electron Microscopy) images.

### Key Features

| Feature | Description |
|---|---|
| Porosity Analysis | Pore detection via Otsu, Adaptive, and Triangle thresholding |
| Fouling Detection | Fouling layer thickness using Sobel + Canny edge mapping |
| Active Site Analysis | K-Means clustering + CLAHE normalization |
| Comparison | Delta porosity calculation between two images |
| Scale Calibration | px/µm input for diameter reporting in micrometres |
| Export | CSV, JSON, PDF (with embedded figure) |
| Bilingual UI | Turkish / English interface |

### Technical Stack

```
Python ─── OpenCV ─── Thresholding, Sobel, Canny, K-Means
        ├── scikit-image ── Regionprops, CLAHE, Canny
        ├── SciPy ──────── Gaussian filter, gradient
        ├── Matplotlib ─── Charts, PDF figure
        ├── ReportLab ──── PDF report generation
        └── Tkinter ────── GUI
```

---

## 2. Installation

### 2.1 Requirements

- Python **3.8** or higher
- Operating system: Windows 10+, macOS 11+, Ubuntu 20.04+

### 2.2 Install Dependencies

```bash
pip install opencv-python pillow scikit-image numpy scipy matplotlib reportlab
```

### 2.3 Optional: Drag & Drop Support

```bash
pip install tkinterdnd2
```

> If this package is not installed, the application runs normally — only the drag-and-drop feature is disabled.

### 2.4 Launch the Application

```bash
python membrane_analyzer_v2.py
```

---

## 3. Interface Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  TOPBAR  │ App name · Version · Language button · Status indicator  │
├──────────┼──────────────────────────────────────┬──────────────────┤
│          │                                      │                  │
│ SIDEBAR  │         IMAGE CANVAS                 │  METRIC          │
│          │   (Zoom · Pan · Drop support)        │  CARDS           │
│  Image   │                                      │  (%  count  px)  │
│  Loading ├──────────────────────────────────────┴──────────────────┤
│          │  [ Results ] [ Charts ] [ Activity Log ]                │
│  Scale   │                                                          │
│  Mode    │  Analysis output and matplotlib charts                   │
│  Params  │                                                          │
│  Actions │                                                          │
└──────────┴──────────────────────────────────────────────────────────┘
```

### Panel Descriptions

**Sidebar (left)**
- **Image Input** — Load primary and comparison images
- **Scale Calibration** — Enter px/µm value
- **Analysis Mode** — Choose from 4 modes
- **Parameters** — Threshold method, pore size, fouling side, cluster count
- **Actions** — Run, Export (CSV/JSON/PDF), Reset

**Image Canvas (centre-top)**
- Displays the loaded image
- Shows the processed image after analysis (e.g. pore mask overlay)
- Supports zoom and pan

**Metric Cards (right)**
- Porosity %, Pore count, Mean diameter, Fouling thickness
- Automatically updated after each analysis

**Tabs (bottom)**
- **Results** — All numerical output
- **Charts** — Matplotlib visualisations
- **Activity Log** — Timestamped session log

---

## 4. Loading Images

Three methods are available for loading images:

### Method 1: Open Button
Click the **Open** button under **"Primary Image"** in the sidebar and select your file using the file picker.

### Method 2: Drag & Drop
Drag a file directly onto the image canvas area.  
*(Requires the tkinterdnd2 package)*

### Method 3: Recent Files Menu
Navigate to **File → Recent Files** in the top menu to reopen one of the last 8 files.  
Recent files are stored in `~/.membrane_analyzer_recent.json`.

### Keyboard Shortcut
`Ctrl + O` opens the file picker.

### Image Metadata Bar
After loading, the following information appears above the canvas:

```
filename.tif  ·  2048×1536  ·  1ch  ·  8bit  ·  3.2 MB
```

### Supported Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| PNG | `.png` | Recommended lossless format |
| JPEG | `.jpg`, `.jpeg` | Lossy compression — use with care |
| TIFF | `.tif`, `.tiff` | 16-bit supported, auto-converted to 8-bit |
| BMP | `.bmp` | — |
| AVIF | `.avif` | — |
| WebP | `.webp` | — |

> ⚠️ **Note:** JPEG compression artefacts can reduce thresholding accuracy. For scientific analysis, **TIFF** or **PNG** is strongly recommended.

---

## 5. Scale Calibration

Scale calibration allows pore diameters to be reported in **micrometres (µm)** instead of pixels (px).

### Setting the Calibration

1. Locate the **"Scale Calibration"** section in the sidebar
2. Enter the **px / µm** value from your SEM/TEM scale bar
3. Leave at `0` to report results in pixels

### Finding the px/µm Value

Refer to the scale bar in your SEM image:

```
Example: Scale bar = 500 nm = 0.5 µm spans 100 px
→ px/µm = 100 px / 0.5 µm = 200 px/µm
```

### Effect of Calibration

When calibration is set, the following line is added to Porosity Analysis results:

```
Mean Equiv. Diameter (µm):   0.847
```

---

## 6. Analysis Modes

### 6.1 Porosity Analysis

Detects pores in the membrane image and produces morphometric statistics.

**How it works:**
1. Image is converted to greyscale
2. Binary mask is created using the selected threshold method
3. Morphological opening removes noise
4. Connected components are labelled; those below the minimum size are discarded
5. Area, diameter and eccentricity are calculated for each pore

**Output Metrics:**

| Metric | Description |
|--------|-------------|
| Porosity (%) | Pore area / total area × 100 |
| Total Pore Count | Number of pores above the minimum size |
| Mean Equiv. Diameter (px) | Diameter computed as 2 × √(area / π) |
| Std. Deviation (px) | Standard deviation of diameter distribution |
| Min / Max Diameter | Smallest and largest pore diameter |
| D10 / D50 / D90 | Percentile diameter values |
| Mean Eccentricity | 0 = perfect circle, 1 = line |

**Visual Output:**  
Detected pores are shown as a **green** overlay on the original image in the canvas.

---

### 6.2 Fouling Layer Analysis

Measures the thickness and intensity of the fouling (contamination) layer on the membrane surface.

**How it works:**
1. A row or column intensity profile is extracted along the chosen direction
2. Sobel + Canny edge detection is applied
3. The intensity gradient is smoothed with a Gaussian filter
4. The peak gradient point is identified as the fouling/membrane interface
5. The interface splits the image into two regions: fouling layer and membrane region

**Fouling Side Selection:**

| Option | Description |
|--------|-------------|
| `top` | Fouling layer is at the top of the image |
| `bottom` | Fouling layer is at the bottom of the image |
| `left` | Fouling layer is on the left side |
| `right` | Fouling layer is on the right side |

**Output Metrics:**

| Metric | Description |
|--------|-------------|
| Interface (row px) | Pixel position of the fouling/membrane boundary |
| Fouling Thickness (px) | Pixel thickness of the fouling layer |
| Fouling Thickness (%) | Ratio to total image height/width |
| Fouling Mean Intensity | Mean grey value of the fouling region |
| Membrane Mean Intensity | Mean grey value of the membrane region |
| Contrast (σ) | Standard deviation of the full image |
| **SNR (dB)** | Signal-to-Noise Ratio — contrast quality between fouling and membrane |

> **SNR interpretation:**  
> `> 20 dB` → Strong contrast, reliable interface detection  
> `10–20 dB` → Moderate contrast, manually verify results  
> `< 10 dB` → Low contrast, analysis reliability is reduced

---

### 6.3 Active Site / Catalyst Analysis

Detects high-intensity (bright) and low-intensity (dark) active regions on the membrane surface.

**How it works:**
1. CLAHE (Contrast Limited Adaptive Histogram Equalization) enhances contrast
2. Pixels above the 90th percentile are labelled bright; pixels below the 10th percentile are labelled dark
3. Morphological opening removes noise
4. K-Means clustering divides the image into `k` groups
5. A segmentation map is produced for each cluster

**Output Metrics:**

| Metric | Description |
|--------|-------------|
| K-Means Cluster Count | Number of clusters used (2–12) |
| Bright Site Count | Number of high-intensity active regions |
| Dark Site Count | Number of low-intensity active regions |
| Total Active Sites | Bright + Dark |
| Bright Coverage (%) | Fraction of image area covered by bright regions |
| Dark Coverage (%) | Fraction of image area covered by dark regions |
| Total Coverage (%) | Combined active site coverage |

---

### 6.4 Comparative Analysis

Performs a porosity comparison between two membrane images (e.g. pristine vs. fouled).

**How to use:**
1. Load the first image in the **Primary Image** field
2. Load the second image in the **Comparison Image** field
3. Select **"Comparative Analysis"** mode
4. Run the analysis

**Output Metrics:**

| Metric | Description |
|--------|-------------|
| Image 1 Porosity (%) | Porosity of the first image |
| Image 2 Porosity (%) | Porosity of the second image |
| Delta Porosity | Image 2 − Image 1 (`+` = Image 2 is more porous) |
| Img 1 / Img 2 Pore Count | Pore count per image |
| Img 1 / Img 2 Mean Diam. | Mean pore diameter per image |

> **∆ Porosity interpretation:**  
> `> +5%` → Significant difference (amber warning displayed)  
> `0 ± 5%` → Similar porosity  
> `< −5%` → First image is more porous

---

## 7. Parameters

### Threshold Method

| Method | Description | Best Used When |
|--------|-------------|----------------|
| **Otsu** | Globally optimal threshold; ideal for bimodal histograms | Homogeneous background, high contrast |
| **Adaptive** | Local window-based thresholding | Uneven illumination, local contrast variation |
| **Triangle** | Zack method for unimodal histograms | Sparse pore distribution, low porosity |

### Min. Pore Size (px)

Regions smaller than this value are excluded from pore counting. Used to filter noise and artefacts.

- **Small value (1–10 px):** All microstructure detected; sensitive to noise
- **Medium value (10–50 px):** Balanced — recommended for most SEM analyses
- **Large value (50–200 px):** Only large pores are counted

### Fouling Side

Specifies where the fouling layer appears in the image. An incorrect selection will cause a wrong interface detection. Inspect your image carefully before choosing.

### K-Means Cluster Count (2–12)

Determines how many intensity groups the image is divided into during Active Site analysis.

- **2–3:** Simple two-phase structures
- **4–6:** Moderately complex membranes (default: 5)
- **7–12:** Multi-phase or highly heterogeneous materials

> Increasing cluster count raises computation time.

---

## 8. Results and Charts

### Results Tab

After analysis completes, the **Results** tab is selected automatically. Colour coding:

| Colour | Meaning |
|--------|---------|
| 🟢 Teal | Normal value |
| 🟡 Amber | Value requiring attention |
| 🔵 Blue | Information / parameter |
| 🔴 Red | Error |

### Charts Tab

Each analysis mode produces different charts:

**Porosity:**
- Pore mask image
- Size distribution histogram (with D10/D50/D90 lines)
- Eccentricity distribution histogram

**Fouling:**
- Edge map + interface line
- Intensity profile + fouling region fill area

**Active Sites:**
- Normalised image
- Bright site mask
- Dark site mask
- K-Means segmentation map

**Comparison:**
- Porosity comparison bar chart (with value labels)
- Pore count comparison bar chart

### Canvas Zoom & Pan

| Action | Gesture |
|--------|---------|
| Zoom in | Scroll wheel up |
| Zoom out | Scroll wheel down |
| Pan | Hold left-click + drag |
| Reset view | Double-click |

---

## 9. Export

### CSV Export

Saves all numerical results in tabular format.

**Columns:**
```
Parameter | Value | Module | Image File | Exported At | App Version
```

**Shortcut:** `Ctrl + S`

---

### JSON Export

Machine-readable, structured format. Ideal for programmatic analysis or pipeline integration.

**Example output:**
```json
{
  "app_version": "2.0",
  "exported_at": "2025-03-15T14:32:11",
  "image": {
    "path": "/data/sem_001.tif",
    "width": 2048,
    "height": 1536,
    "file_kb": 3276.8,
    "scale_um_per_px": 200.0
  },
  "modules": {
    "porosity": {
      "method": "otsu",
      "porosity_pct": 12.847,
      "pore_count": 341,
      "mean_diam_px": 18.23,
      "mean_diam_um": 0.0911,
      "d10_px": 11.4,
      "d50_px": 17.8,
      "d90_px": 26.3
    }
  }
}
```

---

### PDF Export

Generates a professional A4 report containing an embedded chart figure and all numerical results.

**PDF contents:**
- Report title, image filename and timestamp
- Embedded matplotlib figure (120 DPI)
- Results table with parameters for each module

**Shortcut:** `Ctrl + Shift + S`

> ⚠️ PDF export requires the `reportlab` package:
> ```bash
> pip install reportlab
> ```

---

## 10. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + O` | Open image |
| `F5` or `Ctrl + R` | Run analysis |
| `Ctrl + S` | Export CSV |
| `Ctrl + Shift + S` | Export PDF |
| `Ctrl + Z` | Reset application |

---

## 11. Supported File Formats

| Format | Read | Recommendation |
|--------|------|----------------|
| TIFF (8/16-bit) | ✅ | ⭐ Best choice for scientific analysis |
| PNG | ✅ | ⭐ Lossless, noise-free |
| BMP | ✅ | Large file size |
| JPEG | ✅ | ⚠️ Lossy — use with caution |
| AVIF | ✅ | — |
| WebP | ✅ | — |

---

## 12. Metric Reference

### Equivalent Diameter

The diameter of a circle with the same area as the detected pore:

```
d_eq = 2 × √(Area / π)
```

### Eccentricity

Measures how much a pore shape deviates from a circle:

```
0.0 → Perfect circle
0.5 → Moderate ellipse
1.0 → Line (degenerate ellipse)
```

### D10 / D50 / D90 Percentiles

- **D10:** 10% of pores have a diameter below this value
- **D50:** Median diameter — half of pores are smaller, half are larger
- **D90:** 90% of pores have a diameter below this value

A wide D10–D90 spread indicates a heterogeneous pore size distribution.

### SNR (Signal-to-Noise Ratio)

In fouling analysis, SNR measures the contrast quality between the membrane and fouling regions in decibels:

```
SNR = 20 × log10(|Membrane Intensity − Fouling Intensity| / Noise)
```

A higher SNR value indicates more reliable interface detection.

### Porosity (%)

```
Porosity = (Pore Pixel Count / Total Pixel Count) × 100
```

---

## 13. Troubleshooting

### Image Fails to Load

**Symptom:** Error message or blank canvas  
**Solution:**
- Check for special characters or spaces in the file path
- Verify the file is not corrupted
- 16-bit TIFF is automatically converted; 32-bit float formats are not supported

---

### Too Few or Too Many Pores Detected

**Too few pores:**
- Decrease the Min. Pore Size value
- Switch threshold method to `adaptive`
- Increase image contrast in your SEM software

**Too many pores (noise):**
- Increase the Min. Pore Size value (e.g. 20–50 px)
- Try the `otsu` or `triangle` method

---

### Fouling Interface Detected in the Wrong Location

**Symptom:** Interface line appears at an incorrect position  
**Solution:**
- Verify the Fouling Side parameter matches your image
- Ensure the fouling layer forms a distinct boundary in the image
- Very low SNR values (< 10 dB) indicate unreliable detection

---

### PDF Export Not Working

**Symptom:** "Missing Package" error  
**Solution:**
```bash
pip install reportlab
```

---

### Drag & Drop Not Working

**Solution:**
```bash
pip install tkinterdnd2
```
Restart the application after installing.

---

### Analysis Takes Too Long

K-Means (Active Sites) can be slow on large images.  
**Solution:**
- Reduce the cluster count (2–4)
- Resize the image before analysis

---

## 14. Tips and Best Practices

### Image Quality

- Ensure your SEM image is **in focus and contrast-optimised** before analysis
- **16-bit TIFF** provides the highest dynamic range
- If you must use JPEG, select the lowest compression level available

### Choosing a Threshold Method

```
Homogeneous background + high contrast  →  Otsu
Uneven illumination / vignetting        →  Adaptive
Sparse pores / low porosity             →  Triangle
```

### Workflow for Reproducibility

1. Use the same image format for all analyses (preferably TIFF)
2. Set and record the scale calibration at the start
3. Save parameters alongside results using JSON export (metadata is included)
4. Analyse multiple images with the same parameter set

### Parameter Change Reminder

When you modify any parameter after running an analysis, an **amber warning** appears in the sidebar:

```
⚠  Parameters changed — re-run analysis
```

When this message is visible, the displayed results still correspond to the previous parameters. Re-run the analysis to update them.

### For Large Datasets

- In comparative analysis, ensure both images have the same dimensions and resolution
- The Activity Log tab maintains a session log; archive CSV/JSON exports for long-term data storage

---

## Licence and Contribution

This application is developed as open-source software. For bug reports, feature requests or contributions, please contact the project repository.

---

*Membrane Science Analyzer v2.0 · User Guide*  
*Last updated: 2025*
