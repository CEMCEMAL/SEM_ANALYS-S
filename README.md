# SEM_ANALYS-S
Membrane Science Analyzer

Membrane Science Analyzer is a desktop application for analyzing SEM/TEM images of membrane materials. It provides automatic measurements of porosity, fouling layer thickness, and active site/catalyst distribution with an intuitive dark‑themed GUI. The interface supports both Turkish and English, and results can be exported to CSV or PDF.

Features
Porosity Analysis – automatic thresholding (Otsu, adaptive, triangle), pore counting, size distribution, eccentricity.

Fouling Layer Analysis – detects layer thickness from any side, intensity profile, contrast ratio.

Active Site / Catalyst Analysis – identifies bright/dark regions using percentiles and K‑Means clustering.

Comparative Analysis – compares porosity and pore statistics between two images.

Interactive GUI – real‑time image preview, metric cards, detailed text results, charts, and activity log.

Bilingual – switch between Turkish and English instantly.

Export – save numerical data to CSV, generate professional PDF reports.

Quick Start
Requirements
Python 3.8+

Install dependencies:

bash
pip install opencv-python pillow scikit-image numpy scipy matplotlib reportlab
Run
bash
python membrane_analyzer.py
Usage
Load an image (PNG, JPG, TIF, BMP, AVIF, WEBP).

Choose analysis mode (Porosity, Fouling, Active Sites, or Comparison).

Adjust parameters if needed.

Click RUN ANALYSIS.

View results in metric cards, text tab, and charts tab.

License
MIT © Cem Köroğlu

Citation
If you use this software in your research, please cite:

Membrane Science Analyzer v1.0, 2026, GitHub repositor https://github.com/CEMCEMAL/SEM_ANALYS-S
