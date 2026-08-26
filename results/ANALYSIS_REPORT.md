# Palisades & Eaton Fires - Burn Severity Analysis Report

**Analysis Date**: 2026-08-26 20:05:32 UTC

## Executive Summary
Sentinel-2 optical analysis of the Palisades and Eaton Fires in Los Angeles, CA.
Analysis uses Normalized Burn Ratio (NBR) and differenced NBR (dNBR) to map fire
extent and classify burn severity across two cloud-free multispectral scenes
acquired before and 10 days after the fire onset.

## Methodology

### Data Sources
- **Pre-fire**: Sentinel-2A, scene S2A_T11SLT_20250102T183754_L2A, 2025-01-02 (3.0% cloud cover)
- **Post-fire**: Sentinel-2A, scene S2A_T11SLT_20250112T183727_L2A, 2025-01-12 (0.02% cloud cover)
- **Sensor**: Multispectral Instrument (MSI), 13 bands
- **Spatial Resolution**: 10m (resampled from native 10m/20m/60m per band)
- **Analysis Area**: (-118.62, 34.0, -118.05, 34.26)

### Processing Steps
1. **Windowed Remote Read**: Both scenes loaded via HTTPS COG (cloud-optimized GeoTIFF),
   only AOI window fetched to minimize download (~3.7s per scene).
2. **Resampling**: SWIR2 (native 20m) and SCL (native 20m) resampled to 10m using
   bilinear and nearest-neighbor interpolation, respectively.
3. **Cloud/Shadow Masking**: Scene Classification Layer (SCL) classes 3, 8, 9, 10, 11
   (cloud shadow, cloud medium/high probability, cirrus, snow) masked to NaN.
4. **Normalized Burn Ratio (NBR)**:
   - Formula: `NBR = (NIR - SWIR2) / (NIR + SWIR2)`
   - Healthy vegetation → high NBR; burned areas → low NBR
5. **Change Detection (dNBR)**:
   - Formula: `dNBR = NBR_pre - NBR_post`
   - dNBR > 0.1 indicates burned area; magnitude correlates with burn severity
6. **Severity Classification**: USGS standard thresholds applied:
   - Unburned: dNBR < 0.10
   - Low severity: 0.10 ≤ dNBR < 0.27
   - Moderate-low: 0.27 ≤ dNBR < 0.44
   - Moderate-high: 0.44 ≤ dNBR < 0.66
   - High severity: dNBR ≥ 0.66

## Results

### Cloud Coverage & Data Quality
- Pre-fire masked pixels: 668 (0.0%)
- Post-fire masked pixels: 4324 (0.0%)
- Data quality: Excellent (≤3.0% cloud cover for both scenes)

### Burn Severity Distribution
| Severity Class | Area (km²) | Area (acres) | Pixels |
|---|---|---|---|
| Unburned | 669.95 | 165549 | 6699524 |
| Low | 47.65 | 11773 | 476452 |
| Moderate-Low | 50.33 | 12436 | 503283 |
| Moderate-High | 16.63 | 4109 | 166304 |
| High | 0.58 | 143 | 5783 |
| **Total Burned** | **115.18** | **28462** | **1151822** |

### Scene Selection Note (Important for Reproducibility)
The original GeoCroissant dataset included post-fire scenes from Sentinel-2 tile
`T11SLU` (dates Jan 17 & Feb 1, 2025). However, tile `T11SLU` begins at latitude
34.23°N, which covers only the northern portion of the Eaton Fire and **completely
misses the Palisades Fire** (~34.03–34.11°N).

For this analysis, we selected a better-matched post-fire scene from the **same tile
as the pre-fire scene (`T11SLT`)** to ensure both fires are fully captured within
a single consistent projection. Scene `S2A_T11SLT_20250112T183727_L2A` (Jan 12, 2025)
is nearly cloud-free (0.02%) and provides a tight 10-day pre/post window.

### Output Files
- `nbr_pre.tif` – Pre-fire Normalized Burn Ratio
- `nbr_post.tif` – Post-fire Normalized Burn Ratio
- `dnbr.tif` – Differenced NBR (change detection)
- `burn_severity_class.tif` – Severity class raster (0=unburned through 4=high)
- `truecolor_pre.png` – Pre-fire true-color reference image
- `truecolor_post.png` – Post-fire true-color reference image
- `dnbr_map.png` – dNBR heatmap visualization
- `burn_severity_map.png` – Classified severity map with legend
- `burn_severity_stats.json` – Machine-readable statistics (area per class)

## Validation & Limitations

### Validation
- **Area Sanity Check**: Reported fire sizes are Palisades Fire ~23,000 acres, Eaton
  Fire ~14,000 acres, totaling ~37,000 acres. Our analysis estimates 28462 acres
  of burned area, which is 77% of official estimates. Differences expected due to:
  - AOI boundary clipping
  - dNBR threshold (0.1) may miss low-severity burns
  - Analysis date (Jan 12) is after fire onset but before full containment

- **Spectral Consistency**: NBR values consistent with Sentinel-2 vegetation indices
  across healthy vs. burned pixels.

### Known Limitations
1. **Temporal Window**: Analysis uses single pre/post pair (Jan 2 → Jan 12). Fire
   activity peaked Jan 7–9, so 10-day window captures peak burn but misses very
   early stages and ongoing containment.
2. **Residual Smoke**: High-altitude aerosols/smoke at fire onset (Jan 8–10) may
   alter spectral signatures; post-fire scene (Jan 12) acquired after atmospheric
   clearing.
3. **SCL Classification**: Scene Classification relies on ESA's baseline pixel-level
   classifier; edge pixels near clouds may be misclassified.
4. **Severity Interpretation**: dNBR thresholds are empirical; severity classification
   is probabilistic and should be cross-validated with field assessments or
   higher-resolution imagery (aerial photography, Planetscope).
5. **No Terrain Topography**: Analysis does not account for slope/aspect, which
   affects fire spread and severity in mountainous terrain.

## Responsible AI Notes

### Data Collection
Sentinel-2 data collected by ESA's Copernicus program as open-access global archive.
Scenes selected based on cloud-cover constraints and temporal proximity to fire dates.

### Known Biases & Gaps
- Sentinel revisit cycle (5 days constellation) creates temporal sampling gaps.
- Post-fire scene acquired 3 days after fires peaked, missing initial burn progression.
- Smoke presence during fire peak (Jan 7–9) not captured in dataset.
- Coastal/urban area water bodies (Pacific Ocean, urban structures) may confound
  spectral indices; SCL masking helps but is imperfect.

### Use Cases
- Fire extent mapping and burn severity assessment
- Insurance claims and disaster relief targeting
- Post-fire recovery monitoring
- Research on fire behavior and suppression effectiveness

### Social Impact & Privacy
Fire damage maps (especially at property scale) can impact insurance claims,
property values, and residents' privacy. Derivative products should:
- Aggregate to city/county level for public reporting
- Obtain consent before sharing property-specific damage assessments
- Acknowledge uncertainty and limitations in any published analysis

## Future Work

- Integrate Sentinel-1 SAR coherence for structural-change validation
- Combine with MODIS active fire detections (FIRMS) for temporal progression
- Compare with official NIFC fire perimeters for accuracy assessment
- Implement automated severity-based targeting for post-fire erosion-control resources

---

*Report generated by `analyze_burn_severity.py` | Reusable for other fire events*
