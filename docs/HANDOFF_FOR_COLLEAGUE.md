# Session Handoff: Palisades & Eaton Fires Satellite Analysis

**For**: Colleague reproducing this analysis on a different computer  
**Purpose**: Complete guide to set up environment, reproduce results, and adapt for other fire events  
**Time estimate**: 30-45 minutes (including environment setup)

---

## Part 1: Environment Setup

### 1.1 System Requirements

**Operating System**: Linux, macOS, or Windows (WSL2)  
**Disk space**: 200 GB free (for temporary working files + outputs)  
**RAM**: 16 GB minimum (8 GB acceptable with patience; 32 GB ideal)  
**Network**: Stable internet connection (downloads from AWS S3, no authentication needed)

### 1.2 Install Python & Dependencies

**Option A: Using Conda (Recommended)**

```bash
# Create a new conda environment
conda create -n fire-analysis python=3.11 -y

# Activate environment
conda activate fire-analysis

# Install geospatial stack
conda install -c conda-forge rasterio numpy matplotlib gdal -y

# Install additional packages
pip install requests

# Verify installation
python3 -c "import rasterio, numpy, matplotlib; print('✓ All imports successful')"
```

**Option B: Using pip (Standalone)**

```bash
# Create virtual environment
python3 -m venv fire_analysis_env
source fire_analysis_env/bin/activate  # On Windows: fire_analysis_env\Scripts\activate

# Install dependencies
pip install rasterio numpy matplotlib requests

# On macOS/Linux, if GDAL issues occur:
# brew install gdal (macOS) or apt-get install gdal-bin (Ubuntu)
```

**Option C: Using Docker (Most Reproducible)**

```dockerfile
FROM osgeo/gdal:latest

RUN apt-get update && apt-get install -y \
    python3-pip python3-dev && \
    pip install numpy matplotlib requests rasterio

WORKDIR /fire-analysis
COPY . .
CMD ["python3", "scripts/analyze_burn_severity.py"]
```

Build and run:
```bash
docker build -t fire-analysis .
docker run -v $(pwd)/results:/fire-analysis/results fire-analysis
```

---

## Part 2: File Setup

### 2.1 Download All Files from Session

Your colleague needs these files (available from Claude Code file browser):

**Core files** (required):
```
fire_analysis/
├── scripts/
│   └── analyze_burn_severity.py          [Main analysis script - 25 KB]
├── data/                                  [Empty - data fetched remotely]
└── results/                               [Will be created by script]
```

**Documentation** (required for reference):
```
├── SATELLITE_ANALYSIS_SUMMARY.md          [Quick overview]
├── ANALYSIS_REPORT.md                     [Technical details]
├── PALISADES_EATON_FIRES_ANALYSIS.md      [Background on data]
├── REUSABLE_FIRE_EVENT_WORKFLOW.md        [Template for other fires]
└── HANDOFF_FOR_COLLEAGUE.md               [This file]
```

**Outputs from original session** (optional - for comparison):
```
├── palisades_eaton_fires_geocroissant.json [Metadata - 91 KB]
└── fire_analysis/results/
    ├── *.tif (GeoTIFFs)                   [Optional - can regenerate]
    └── *.png (visualizations)             [Optional - can regenerate]
```

### 2.2 Directory Structure

Create this structure on the new computer:

```
~/fire-analysis-project/
├── scripts/
│   └── analyze_burn_severity.py
├── data/
│   └── (empty - data fetched from AWS)
├── results/
│   └── (will be populated by script)
├── docs/
│   ├── SATELLITE_ANALYSIS_SUMMARY.md
│   ├── ANALYSIS_REPORT.md
│   ├── PALISADES_EATON_FIRES_ANALYSIS.md
│   ├── REUSABLE_FIRE_EVENT_WORKFLOW.md
│   └── HANDOFF_FOR_COLLEAGUE.md
└── README.md (create locally with project overview)
```

---

## Part 3: Run the Analysis

### 3.1 Quick Start (Palisades & Eaton Fires)

```bash
cd ~/fire-analysis-project

# Activate environment
conda activate fire-analysis  # or: source fire_analysis_env/bin/activate

# Run analysis
python3 scripts/analyze_burn_severity.py

# Expected output:
# - Console: Progress messages + statistics
# - results/: GeoTIFFs (163 MB), PNGs (1.6 MB), JSON + markdown reports
# - Total time: ~5-10 minutes (depends on disk speed and network)
```

### 3.2 Expected Output

After running, check that `results/` contains:

```
results/
├── nbr_pre.tif                  ✓ 54 MB
├── nbr_post.tif                 ✓ 54 MB
├── dnbr.tif                     ✓ 55 MB
├── burn_severity_class.tif      ✓ 2.4 MB
├── truecolor_pre.png            ✓ 493 KB
├── truecolor_post.png           ✓ 496 KB
├── dnbr_map.png                 ✓ 251 KB
├── burn_severity_map.png        ✓ 313 KB
├── burn_severity_stats.json     ✓ 622 B
└── ANALYSIS_REPORT.md           ✓ 6.6 KB
```

**Total size**: ~166 MB (GeoTIFFs are large; PNG visualizations can be deleted to save space)

### 3.3 Validate Results

Check the statistics are reasonable:

```bash
# Read statistics
cat results/burn_severity_stats.json | python3 -m json.tool

# Expected total burned area: ~115 km² (28,462 acres)
# Should be close to original session
```

Validate GeoTIFFs with GDAL:

```bash
gdalinfo results/dnbr.tif | head -20
# Should show:
# - Driver: GTiff/GeoTIFF
# - Size: 5294 x 2952
# - Projection: EPSG:32611 (UTM Zone 11N)
```

---

## Part 4: Adapt for a Different Fire Event

### 4.1 Edit Configuration

Open `scripts/analyze_burn_severity.py` and modify the **CONFIGURATION** section (lines ~30-50):

```python
# ============================================================================
# CONFIGURATION: Parameterized for reuse on other fire events
# ============================================================================

# Step 1: Find the fire location
AOI_WGS84 = (-118.62, 34.00, -118.05, 34.26)  # ← Change this
# Format: (min_lon, min_lat, max_lon, max_lat)
# Get bounding box from:
#   - Google Maps: right-click, get lat/lon
#   - NIFC InciWeb: view fire boundary
#   - Natural Earth viewer

# Step 2: Find Sentinel-2 scene IDs and dates
SCENE_PRE = {
    'scene_id': 'S2A_T11SLT_20250102T183754_L2A',  # ← Change this
    'date': '2025-01-02',                          # ← Change this
    'cloud_pct': 3.0,
    'base_url': 'https://e84-earth-search-sentinel-data.s3.us-west-2.amazonaws.com/sentinel-2-c1-l2a/11/S/LT/2025/1'
}

SCENE_POST = {
    'scene_id': 'S2A_T11SLT_20250112T183727_L2A',  # ← Change this
    'date': '2025-01-12',                          # ← Change this
    'cloud_pct': 0.02,
    'base_url': 'https://e84-earth-search-sentinel-data.s3.us-west-2.amazonaws.com/sentinel-2-c1-l2a/11/S/LT/2025/1'
}
```

### 4.2 How to Find Scene IDs for a Different Fire

**Step A: Get fire location**

```bash
# Option 1: USGS NIFC InciWeb
# https://inciweb.nwcg.gov/
# Search fire name → view boundary → note lat/lon

# Option 2: NASA FIRMS active fire map
# https://firms.modaps.eosdis.nasa.gov/
# View fire, note lat/lon from browser

# Result: AOI_WGS84 = (min_lon, min_lat, max_lon, max_lat)
```

**Step B: Find Sentinel-2 scenes**

Use Earth Search browser or Python:

```python
import requests

# Query Sentinel-2 scenes
bbox = (-118.62, 34.00, -118.05, 34.26)  # Your fire AOI
datetime_range = "2025-01-02/2025-01-12"  # Pre/post fire

url = "https://earth-search.aws.element84.com/v1/collections/sentinel-2-c1-l2a/items"
params = {
    "bbox": f"{bbox[0]},{bbox[1]},{bbox[2]},{bbox[3]}",
    "datetime": datetime_range,
    "limit": 100
}

response = requests.get(url, params=params)
items = response.json()['features']

for item in items:
    print(f"Scene: {item['id']}")
    print(f"Date: {item['properties']['datetime']}")
    print(f"Cloud cover: {item['properties'].get('eo:cloud_cover', 'N/A')}%")
    print()
```

**Step C: Select best pre/post pair**

Pick scenes with:
- ≤10% cloud cover for optical (Sentinel-2)
- Pre-fire: 7-14 days before fire start
- Post-fire: 3-10 days after fire peak
- Same Sentinel-2 tile (if possible) to avoid reprojection

**Step D: Update script**

```python
SCENE_PRE = {
    'scene_id': 'S2B_T...Y..._L2A',  # From search results
    'date': 'YYYY-MM-DD',            # From search results
    'cloud_pct': X.X,                # From search results
    'base_url': 'https://e84-earth-search-sentinel-data.s3.us-west-2.amazonaws.com/sentinel-2-c1-l2a/xx/X/XX/YYYY/MM'
}
```

### 4.3 Example: Adapt for a Different Fire

Say colleague wants to analyze the **Camp Fire** (California, Nov 2018):

```python
# Camp Fire AOI (Paradise, CA)
AOI_WGS84 = (-121.25, 39.70, -120.90, 39.95)

# Query Sentinel-2 near Nov 2018 (pre: early Nov, post: late Nov)
SCENE_PRE = {
    'scene_id': 'S2A_TILEID_20181105T..._L2A',  # ~Nov 5 (pre-fire)
    'date': '2018-11-05',
    'cloud_pct': 5.0,
    'base_url': 'https://...'
}

SCENE_POST = {
    'scene_id': 'S2B_TILEID_20181120T..._L2A',  # ~Nov 20 (post-fire)
    'date': '2018-11-20',
    'cloud_pct': 8.0,
    'base_url': 'https://...'
}

# Run: python3 scripts/analyze_burn_severity.py
# → Results in results/ directory with same format
```

---

## Part 5: Troubleshooting

### Error: "Module not found: rasterio"

```bash
# Reinstall geospatial stack
conda install -c conda-forge rasterio gdal -y
# or
pip install rasterio --upgrade
```

### Error: "Connection refused" or "Timeout"

```bash
# Sentinel-2 data is hosted on AWS S3 (public, no auth)
# If download is slow, try:
# 1. Check internet connection
# 2. Run during off-peak hours (US West Coast typically has lower load 2-6 AM UTC)
# 3. Increase timeout in script: rasterio.open(url, driver='GTiff', timeout=60)
```

### Error: "Killed" or "Out of memory"

```bash
# Script uses ~4 GB RAM for 2952x5294 pixel arrays
# Solutions:
# 1. Close other applications
# 2. Increase virtual memory / swap
# 3. Edit script to downsample input: e.g., read every 2nd pixel
```

### PNG visualizations are low-resolution

This is intentional (downsampled 2x for memory efficiency during display generation).  
For high-res maps, use **GeoTIFFs** in QGIS/ArcGIS instead.

---

## Part 6: Share Results with the Team

### 6.1 What to Share

**For stakeholders** (non-technical):
- `SATELLITE_ANALYSIS_SUMMARY.md` (overview)
- PNG visualizations (burn_severity_map.png, truecolor_pre/post.png)
- Statistics (copy from burn_severity_stats.json)

**For researchers/analysts**:
- GeoTIFFs (all .tif files)
- ANALYSIS_REPORT.md (technical details)
- GeoCroissant metadata (palisades_eaton_fires_geocroissant.json)

**For colleagues reproducing analysis**:
- This entire handoff guide
- analyze_burn_severity.py script
- All documentation

### 6.2 Package for Distribution

```bash
# Create a portable archive
cd ~/fire-analysis-project

tar -czf palisades_eaton_fires_analysis.tar.gz \
  scripts/ \
  docs/ \
  results/burn_severity_stats.json \
  results/ANALYSIS_REPORT.md \
  results/*.png

# Share palisades_eaton_fires_analysis.tar.gz (~2 MB)
# (Exclude large GeoTIFFs unless needed; they're 163 MB)

# Or just share the script + docs + script:
zip -r fire-analysis-portable.zip \
  scripts/analyze_burn_severity.py \
  docs/*.md \
  results/burn_severity_stats.json
```

### 6.3 Git Repository Setup (Recommended)

```bash
# Initialize git repo
cd ~/fire-analysis-project
git init
git add scripts/ docs/ HANDOFF_FOR_COLLEAGUE.md
git commit -m "Initial: Palisades & Eaton Fires satellite analysis pipeline"

# Create .gitignore to exclude large files
cat > .gitignore << EOF
results/*.tif
results/*.tiff
results/*color*.png
.DS_Store
__pycache__/
*.pyc
EOF

git add .gitignore
git commit -m "Add gitignore for large rasters"

# Push to GitHub/GitLab for team access
git remote add origin https://github.com/yourorg/fire-analysis.git
git push -u origin main
```

Colleague clones with:
```bash
git clone https://github.com/yourorg/fire-analysis.git
cd fire-analysis
python3 scripts/analyze_burn_severity.py
```

---

## Part 7: Verification Checklist

After colleague sets up and runs, verify:

- [ ] Environment setup: `python3 -c "import rasterio, numpy, matplotlib"` ✓
- [ ] Script runs without errors
- [ ] Output files exist in `results/` directory
- [ ] Total burned area ~115 km² (±10%)
- [ ] GeoTIFFs have correct CRS (EPSG:32611)
- [ ] PNG visualizations show fire-affected areas
- [ ] JSON statistics are readable and numerical
- [ ] Script can be re-run on same data without errors (idempotent)
- [ ] Colleague can adapt script for a different fire event

---

## Part 8: Support Resources

### For Colleagues

**If they need to understand the methodology:**
→ Read `ANALYSIS_REPORT.md` (methodology section)

**If they want to adapt for other fires:**
→ Follow Part 4 of this guide + `REUSABLE_FIRE_EVENT_WORKFLOW.md`

**If they want to integrate with GIS:**
→ Load GeoTIFFs into QGIS/ArcGIS; use `burn_severity_class.tif` for mapping

**If they want to cite this work:**
→ Cite as: "Sentinel-2 Normalized Burn Ratio analysis of [Fire Name], [Date], [Location]"

### Key References

- **Sentinel-2 Documentation**: https://sentinel.esa.int/web/sentinel/technical-guides/sentinel-2-msi
- **Burn Severity Classification**: USGS GeoMAC (geomac.usgs.gov)
- **Rasterio Docs**: https://rasterio.readthedocs.io/
- **Earth Search STAC**: https://earth-search.aws.element84.com/v1/

---

## Part 9: Known Limitations (For Colleague Awareness)

1. **Temporal**: Analysis captures fires 3+ days after peak; doesn't track real-time progression
2. **Threshold-based**: dNBR ≥0.1 classification misses very low-severity burns
3. **Single satellite**: Relies on Sentinel-2 only; Sentinel-1 SAR not yet integrated
4. **Resolution**: 10m pixels; sub-meter details (individual properties) not visible
5. **Cloud dependency**: Method fails with >20% cloud cover; requires clear conditions

---

## Part 10: Quick Reference Card

**For your colleague's desk:**

```
═════════════════════════════════════════════════════════════════════
PALISADES & EATON FIRES SATELLITE ANALYSIS - QUICK REFERENCE
═════════════════════════════════════════════════════════════════════

SETUP (once):
  1. conda create -n fire-analysis python=3.11 -y
  2. conda activate fire-analysis
  3. conda install -c conda-forge rasterio numpy matplotlib gdal -y
  4. Download scripts/ and docs/ from Claude Code session

RUN (every time):
  1. cd ~/fire-analysis-project
  2. conda activate fire-analysis
  3. python3 scripts/analyze_burn_severity.py
  4. Results appear in results/ directory (~5-10 min)

ADAPT FOR NEW FIRE:
  1. Open scripts/analyze_burn_severity.py
  2. Edit CONFIGURATION section (lines 30-50):
     - AOI_WGS84: fire bounding box
     - SCENE_PRE/SCENE_POST: Sentinel-2 scene IDs & dates
  3. Run script → new results generated automatically

OUTPUTS:
  - *.tif: Georeferenced rasters for GIS analysis
  - *.png: Visualizations for presentations
  - *.json: Statistics for reports
  - *.md: Technical documentation

KEY FILES:
  scripts/analyze_burn_severity.py     ← Main script
  docs/ANALYSIS_REPORT.md              ← Technical details
  docs/REUSABLE_FIRE_EVENT_WORKFLOW.md ← How to adapt

TROUBLESHOOT:
  - Module not found? → conda install -c conda-forge rasterio
  - Out of memory? → Close other apps or increase swap
  - Timeout? → Run during off-peak hours (2-6 AM UTC)

═════════════════════════════════════════════════════════════════════
```

---

## Conclusion

Your colleague now has:

✓ **Environment setup** instructions (conda/pip/Docker)  
✓ **Complete reproducibility** (all code, docs, parameterized for any fire)  
✓ **Adaptation guide** (how to run for different fire events)  
✓ **Troubleshooting** (common issues + solutions)  
✓ **Distribution guide** (how to share results with teams)  

**Expected outcome**: Colleague can reproduce the Palisades & Eaton analysis in 30-45 minutes, then adapt for their own fire event in 15 minutes.

---

*Version 1.0 | Created: 2026-08-26 | Last updated: [colleague should update this]*
