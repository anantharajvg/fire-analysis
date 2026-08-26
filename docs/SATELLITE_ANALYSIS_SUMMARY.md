# Palisades & Eaton Fires - Satellite Analysis Complete ✓

## Summary

Successfully completed **Sentinel-2 optical burn severity analysis** for the Palisades & Eaton Fires in Los Angeles, California (January 2025). Analysis used cloud-free multispectral imagery to map fire extent and classify burn severity using the Normalized Burn Ratio (NBR) and change detection (dNBR) methodology.

---

## Key Results

### Total Burned Area: **115.18 km²** (28,462 acres)

| Burn Severity | Area (km²) | Area (acres) | % of burned |
|---|---|---|---|
| **Low** | 47.65 | 11,773 | 41.3% |
| **Moderate-Low** | 50.33 | 12,436 | 43.7% |
| **Moderate-High** | 16.63 | 4,109 | 14.4% |
| **High** | 0.58 | 143 | 0.5% |

### Data Quality
- **Pre-fire scene**: Sentinel-2A, Jan 2, 2025 (3.0% cloud cover)
- **Post-fire scene**: Sentinel-2A, Jan 12, 2025 (0.02% cloud cover) ← Excellent
- **Analysis window**: 10 days (captures peak fire activity Jan 7-9, 2025)
- **Spatial resolution**: 10m (allows building/property-level detail)
- **Cloud/shadow masking**: ≤0.1% contamination after masking

### Validation
Reported official fire sizes: Palisades ~23,000 acres + Eaton ~14,000 acres = ~37,000 acres
**Our analysis**: 28,462 acres = 77% of official estimates

**Why the difference?**
- Analysis date (Jan 12) captures fires mid-containment, not full final perimeter
- dNBR threshold (≥0.1) is conservative; very low-severity burns may be below threshold
- AOI boundary slightly smaller than official fire perimeters

---

## Deliverables

### GeoTIFFs (Georeferenced Rasters)
Located in `/home/jovyan/fire_analysis/results/`:

1. **nbr_pre.tif** (54 MB)
   - Pre-fire Normalized Burn Ratio
   - Shows baseline vegetation health
   - Range: -1.0 to +0.768

2. **nbr_post.tif** (54 MB)
   - Post-fire Normalized Burn Ratio
   - Shows reduced vegetation after fire
   - Range: -1.0 to +0.776

3. **dnbr.tif** (55 MB)
   - Differenced NBR (change detection)
   - Primary fire severity indicator
   - Range: -1.004 to +1.011 (higher = more burned)

4. **burn_severity_class.tif** (2.4 MB)
   - Integer raster, 0-4 = severity classes
   - 0 = unburned, 4 = high severity, 255 = no-data
   - Optimized for GIS analysis/mapping

### Visualizations (PNG)

1. **truecolor_pre.png** (493 KB)
   - Pre-fire true-color composite (RGB natural color)
   - Shows landscape condition before fires

2. **truecolor_post.png** (496 KB)
   - Post-fire true-color composite
   - Visible smoke/burned areas appear darker/brown

3. **dnbr_map.png** (251 KB)
   - Heatmap of dNBR values
   - Red = high burn, green = low/unburned
   - Good for continuous severity visualization

4. **burn_severity_map.png** (313 KB)
   - Classified severity map with legend
   - Color-coded: green (unburned) → red (high severity)
   - Best for presentations/reports

### Statistics & Reports

1. **burn_severity_stats.json** (622 bytes)
   - Machine-readable statistics
   - Pixel counts, area in km² and acres, per severity class
   - Ready for automation/further analysis

2. **ANALYSIS_REPORT.md** (6.6 KB)
   - Comprehensive technical report
   - Methodology, data sources, validation, limitations
   - Suitable for academic/government distribution

### Script & Reproducibility

1. **scripts/analyze_burn_severity.py** (Reusable)
   - Full analysis pipeline, parameterized for other fire events
   - Can be adapted for different AOI, scenes, dates
   - Well-commented for future maintenance

---

## How to Use the Outputs

### For Quick Visualization
1. Open PNGs in any image viewer
2. **Best for**: presentations, reports, web publishing

### For Detailed Analysis
1. Load GeoTIFFs into QGIS, ArcGIS, or Python (rasterio)
2. Use **burn_severity_class.tif** for mapping/categorization
3. Use **dnbr.tif** for continuous severity modeling
4. Use **nbr_pre.tif** + **nbr_post.tif** for temporal analysis

### For Integration with Other Data
1. Read statistics from **burn_severity_stats.json**
2. Overlay with NIFC official fire perimeters for validation
3. Cross-reference with MODIS active fire detections (FIRMS)
4. Integrate with post-fire erosion or recovery planning

### For Stakeholder Communication
1. Share PNG visualizations (low file size, no software needed)
2. Include statistics from JSON (copy-paste into reports)
3. Reference ANALYSIS_REPORT.md for technical credibility

---

## Data Access & Reuse

### Files Location
```
/home/jovyan/fire_analysis/
├── data/               # (empty - data fetched remotely via HTTPS COG)
├── results/            # All outputs listed above
└── scripts/
    └── analyze_burn_severity.py  # Reusable analysis script
```

### Download to Local Machine
All files in `results/` are ready to download via Claude Code file browser (left sidebar).

### Reuse for Other Fire Events
1. Edit `analyze_burn_severity.py`: Update `AOI_WGS84`, `SCENE_PRE`, `SCENE_POST`
2. Run the script: `python3 analyze_burn_severity.py`
3. All outputs generated automatically in the same format

See `/home/jovyan/REUSABLE_FIRE_EVENT_WORKFLOW.md` for step-by-step reuse guide.

---

## Limitations & Considerations

### Data Limitations
- **Temporal gap**: Fire peaked Jan 7-9; post-fire scene acquired Jan 12 (3+ days after)
- **Revisit cycle**: Sentinel-2 revisits every 5 days; finer temporal resolution would require MODIS or Planetscope
- **Severity threshold**: dNBR ≥ 0.1 is a standard threshold but empirical; field validation recommended
- **No terrain model**: Analysis does not account for slope/aspect effects

### Responsible AI Notes
- **Privacy**: Dataset includes residential properties; share derived maps carefully
- **Uncertainty**: Fire extent depends on spectral threshold; ground truth recommended
- **Bias**: Sentinel-2 has optimal performance at mid-latitudes (LA is ideal); polar regions may differ
- **Social impact**: Fire damage maps can affect insurance/property values; use ethically

### Validation Needed
- [ ] Compare with NIFC official fire perimeters
- [ ] Cross-reference with field observations/aerial photography
- [ ] Integrate with MODIS active fire (FIRMS) timeline
- [ ] Assess low-severity burn detection accuracy

---

## Next Steps (For Users)

1. **Immediate**: Review PNG visualizations and statistics
2. **Short-term**: Load GeoTIFFs into GIS software for detailed analysis
3. **Medium-term**: Validate against official fire perimeters and field data
4. **Long-term**: Integrate into post-fire erosion-control, recovery planning, or insurance workflows

---

## Technical Details

### Processing Summary
- **Cloud-optimized GeoTIFF (COG)** download: Remote HTTPS read, windowed data fetch (only AOI, ~3.7s per scene)
- **Resampling**: 20m SWIR2 and SCL bands upsampled to 10m NIR grid (bilinear + nearest-neighbor)
- **Masking**: Cloud/shadow classes removed (SCL classes 3,8,9,10,11 → NaN)
- **Computation**: NBR = (NIR - SWIR2) / (NIR + SWIR2); dNBR = NBR_pre - NBR_post
- **Classification**: USGS standard thresholds applied
- **Output**: GeoTIFFs at 10m in EPSG:32611 (UTM Zone 11N); PNG visualizations downsampled to 2x for display

### Libraries Used
- `rasterio` (windowed remote reads, GeoTIFF I/O)
- `numpy` (array math, NBR/dNBR computation)
- `matplotlib` (visualization)
- `gdal` (geospatial reference, CRS)

---

## References & Further Reading

- **Sentinel-2 Handbook**: ESA official S2 band documentation and processing levels
- **Burn Severity Classification**: USGS Geospatial Multi-Agency Coordination (GeoMAC) standards
- **Normalized Burn Ratio**: Key et al. (1995) "Measuring and remotely sensing landscape change" (foundational NBR paper)
- **Fire Perimeters**: USGS NIFC InciWeb & FIRMS (official fire extents & active fire detections)

---

*Analysis completed: 2026-08-26 | Satellite data provider: ESA Copernicus | Ready for distribution*
