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

**Maintainers**: Valentine Anantharaj (@anantharajvg)
**Last Updated**: 2025-08-26
```
