# GitHub Setup: Share Fire Analysis Project with Colleagues

**Purpose**: Set up a Git repository so colleagues can clone, reproduce, and collaborate on fire analysis  
**Time**: 10 minutes

---

## Option 1: Create Public GitHub Repository (Recommended)

### Step 1: Initialize Local Repository

```bash
cd ~/fire-analysis-project

# Initialize git
git init
git config user.name "Your Name"
git config user.email "your.email@organization.com"

# Create .gitignore (exclude large/temporary files)
cat > .gitignore << 'EOF'
# Large raster files (can be regenerated)
results/*.tif
results/*.tiff

# Downsampled visualizations (can be regenerated)
results/*color*.png
results/*map*.png

# System files
.DS_Store
Thumbs.db
__pycache__/
*.pyc
*.egg-info/

# Virtual environments
fire_analysis_env/
venv/

# IDE
.vscode/
.idea/
*.swp

# Temporary
*.tmp
*.log
EOF

# Add files
git add scripts/
git add *.md
git add .gitignore
git status  # Review what will be committed
```

### Step 2: Create Initial Commits

```bash
# Commit scripts and documentation
git commit -m "feat: Add Sentinel-2 burn severity analysis pipeline

- Implement NBR and dNBR calculations for fire extent mapping
- Support cloud-free Sentinel-2 multispectral imagery
- Generate georeferenced GeoTIFFs and visualizations
- Include comprehensive technical documentation
- Parameterized for reuse on different fire events"

# Add small output files (stats, report)
git add results/burn_severity_stats.json results/ANALYSIS_REPORT.md
git commit -m "docs: Add burn severity analysis report for Palisades & Eaton Fires

- Total burned area: 115.18 km² (28,462 acres)
- Severity breakdown by USGS classification
- Validation against official fire perimeters
- Limitations and recommendations for future work"
```

### Step 3: Create GitHub Repository

1. Go to https://github.com/new
2. **Repository name**: `fire-analysis` (or `palisades-eaton-fires-2025`)
3. **Description**: "Sentinel-2 optical analysis of fire extent and burn severity"
4. **Visibility**: Public (so colleagues can easily access)
5. **Initialize**: Do NOT initialize with README (we already have one)
6. Click "Create repository"

### Step 4: Push to GitHub

```bash
# Add remote (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/fire-analysis.git

# Rename branch if needed (GitHub defaults to 'main')
git branch -M main

# Push to GitHub
git push -u origin main
```

GitHub now displays:
- Source code (scripts/)
- Documentation (*.md files)
- Statistics (results/burn_severity_stats.json)
- Gitignore (excludes large files)

---

## Option 2: Upload to Organization GitHub

If your organization uses GitHub Enterprise or an org account:

```bash
# Add to org repository
git remote add origin https://github.com/YOUR_ORG/fire-analysis.git
git push -u origin main

# Colleagues clone with:
# git clone https://github.com/YOUR_ORG/fire-analysis.git
```

---

## Option 3: Use GitLab, Gitea, or Other Platforms

Same workflow, just different domain:

```bash
# GitLab
git remote add origin https://gitlab.com/YOUR_USERNAME/fire-analysis.git

# Gitea (self-hosted)
git remote add origin https://gitea.your-org.com/YOUR_USERNAME/fire-analysis.git

# Bitbucket
git remote add origin https://bitbucket.org/YOUR_USERNAME/fire-analysis.git

git push -u origin main
```

---

## Colleague Workflow: Clone & Run

### For a Colleague

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/fire-analysis.git
cd fire-analysis

# Set up environment
conda create -n fire-analysis python=3.11 -y
conda activate fire-analysis
conda install -c conda-forge rasterio numpy matplotlib gdal -y

# Run analysis
python3 scripts/analyze_burn_severity.py

# Results appear in results/ directory
# (Large GeoTIFFs not in repo, regenerated locally)
```

---

## Managing Updates & Contributions

### When Analysis is Updated

```bash
# You update the script to fix a bug or improve it
nano scripts/analyze_burn_severity.py

# Commit and push
git add scripts/analyze_burn_severity.py
git commit -m "fix: Improve cloud masking threshold for high-altitude clouds"
git push

# Colleague pulls updates
git pull origin main
```

### When Colleague Adapts for New Fire

```bash
# Colleague creates a new branch
git checkout -b feature/camp-fire-analysis

# Edits configuration
nano scripts/analyze_burn_severity.py

# Commits locally
git add scripts/
git commit -m "feat: Add Camp Fire (Nov 2018) configuration

- AOI: Paradise, CA
- Pre-fire: 2018-11-05 (3% cloud)
- Post-fire: 2018-11-20 (8% cloud)
- Total burned: ~240,000 acres"

# Pushes to GitHub
git push origin feature/camp-fire-analysis

# Opens Pull Request on GitHub for review
# After approval, merges to main
```

---

## Repository Structure in GitHub

```
fire-analysis/
├── README.md                              ← Main overview (GitHub shows this first)
├── .gitignore                             ← Excludes large files
├── scripts/
│   └── analyze_burn_severity.py           ← Main script
├── docs/
│   ├── SATELLITE_ANALYSIS_SUMMARY.md      ← Results overview
│   ├── ANALYSIS_REPORT.md                 ← Technical details
│   ├── PALISADES_EATON_FIRES_ANALYSIS.md  ← Background
│   ├── REUSABLE_FIRE_EVENT_WORKFLOW.md    ← Template
│   ├── HANDOFF_FOR_COLLEAGUE.md           ← Setup guide
│   └── GITHUB_SETUP.md                    ← This file
├── results/
│   ├── burn_severity_stats.json           ← Stats (small, included)
│   ├── ANALYSIS_REPORT.md                 ← Report (small, included)
│   └── *.tif (ignored, generated locally) ← Large files not in repo
└── environment.yml                        ← (Optional) Conda env file
```

---

## Create a Good README.md

This is what people see first on GitHub:

```markdown
# Fire Analysis: Sentinel-2 Burn Severity Mapping

Automated analysis of wildfire extent and burn severity using Sentinel-2 optical imagery.

## Features

- **Remote data access**: Fetch Sentinel-2 scenes directly from AWS via HTTPS COG
- **Cloud-free selection**: Automated cloud cover filtering
- **Burn indices**: Normalized Burn Ratio (NBR) and change detection (dNBR)
- **Severity classification**: USGS standard burn severity classes
- **Georeferenced outputs**: GeoTIFF + PNG visualizations + statistics
- **Reusable**: Easily adaptable to any fire event globally

## Quick Start

### Setup (One-time)
```bash
conda create -n fire-analysis python=3.11 -y
conda activate fire-analysis
conda install -c conda-forge rasterio numpy matplotlib gdal -y
git clone https://github.com/YOUR_USERNAME/fire-analysis.git
cd fire-analysis
```

### Run
```bash
python3 scripts/analyze_burn_severity.py
```

### Results
- GeoTIFFs: `results/*.tif` (10m resolution, EPSG:32611)
- Visualizations: `results/*.png` (pre/post true-color, severity map)
- Statistics: `results/burn_severity_stats.json`
- Report: `results/ANALYSIS_REPORT.md`

## Example: Palisades & Eaton Fires (Jan 2025)

**Location**: Los Angeles, CA  
**Date**: Jan 2 (pre) vs Jan 12 (post), 2025  
**Result**: 115.18 km² (28,462 acres) burned area identified

See [ANALYSIS_REPORT.md](docs/ANALYSIS_REPORT.md) for full methodology and validation.

## Adapt for Your Fire

Edit `scripts/analyze_burn_severity.py` configuration section:

```python
AOI_WGS84 = (-lon1, lat1, -lon2, lat2)  # Your fire bounding box
SCENE_PRE = {'scene_id': 'S2X_...', 'date': '2025-MM-DD', ...}
SCENE_POST = {'scene_id': 'S2X_...', 'date': '2025-MM-DD', ...}
```

Then run: `python3 scripts/analyze_burn_severity.py`

See [REUSABLE_FIRE_EVENT_WORKFLOW.md](docs/REUSABLE_FIRE_EVENT_WORKFLOW.md) for detailed instructions.

## Requirements

- Python 3.11+
- 16 GB RAM (8 GB minimum)
- 200 GB disk space
- Internet connection (AWS S3 data access)

## Dependencies

- `rasterio`: Geospatial raster I/O
- `numpy`: Array math
- `matplotlib`: Visualization
- `gdal`: Coordinate reference systems

Install: `conda install -c conda-forge rasterio numpy matplotlib gdal`

## Documentation

- [Setup Guide](docs/HANDOFF_FOR_COLLEAGUE.md)
- [Reusable Workflow](docs/REUSABLE_FIRE_EVENT_WORKFLOW.md)
- [Analysis Report](docs/ANALYSIS_REPORT.md)
- [GitHub Setup](docs/GITHUB_SETUP.md)

## Data Sources

- **Sentinel-2**: ESA Copernicus Open Access Hub
- **STAC Catalog**: Element84 Earth Search (AWS)
- **Fire Perimeters**: USGS NIFC InciWeb, MODIS FIRMS

All data is public and requires no authentication.

## Limitations

- Temporal: 5-6 day satellite revisit cycle
- Threshold-based: dNBR ≥0.1 is empirical; validation recommended
- Cloud-dependent: Method fails with >20% cloud cover
- Single sensor: Sentinel-2 optical only (Sentinel-1 SAR TBD)

## Cite This Work

```bibtex
@software{fire-analysis-2025,
  title={Fire Analysis: Sentinel-2 Burn Severity Mapping},
  author={Your Name},
  year={2025},
  url={https://github.com/YOUR_USERNAME/fire-analysis}
}
```

## License

MIT License (or choose another: CC-BY-4.0, GPL-3.0, Apache-2.0)

## Contributing

Contributions welcome! Please:
1. Fork the repo
2. Create a feature branch (`git checkout -b feature/your-fire`)
3. Commit changes (`git commit -m "Add Camp Fire analysis"`)
4. Push (`git push origin feature/your-fire`)
5. Open a Pull Request

## Issues & Support

- **Bug report**: Create GitHub Issue with error message + traceback
- **New fire event**: Share scene IDs + bounding box; we'll help adapt
- **Enhancement idea**: Open GitHub Discussion or Issue

## Changelog

### v1.0 (2025-08-26)
- Initial release with Sentinel-2 NBR/dNBR analysis
- Validated on Palisades & Eaton Fires (LA, Jan 2025)
- Parameterized for global reuse

---

**Maintainers**: Your Name (@github-username)  
**Last Updated**: 2025-08-26
```

Save as `README.md` in repo root:

```bash
# Create README
cat > README.md << 'EOF'
[paste above markdown]
EOF

git add README.md
git commit -m "docs: Add comprehensive README"
git push
```

---

## Optional: Add CI/CD Testing

To auto-test that the script works, add GitHub Actions:

```yaml
# .github/workflows/test.yml
name: Test Fire Analysis

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: conda-incubator/setup-miniconda@v2
        with:
          python-version: 3.11
          channels: conda-forge
          channel-priority: true
      - run: conda install rasterio numpy matplotlib gdal
      - run: python3 scripts/analyze_burn_severity.py
      - uses: actions/upload-artifact@v2
        with:
          name: results
          path: results/burn_severity_stats.json
```

---

## Summary: What Colleagues See

On GitHub, colleagues can:

1. **View code**: Browse `scripts/` and understand the methodology
2. **Read docs**: All *.md files rendered on GitHub
3. **Clone locally**: `git clone https://github.com/YOUR_USERNAME/fire-analysis.git`
4. **Run analysis**: Install dependencies, run script, regenerate results
5. **Adapt**: Edit configuration, create pull request with new fire event
6. **Contribute**: Report issues, suggest improvements, add features

**No additional setup needed beyond what's in HANDOFF_FOR_COLLEAGUE.md.**

---

*Created: 2026-08-26*
