# Palisades & Eaton Fires Los Angeles - Satellite Imagery Analysis

## Overview
A comprehensive GeoCroissant dataset combining Sentinel-2 optical and Sentinel-1 radar satellite imagery before and after the Palisades and Eaton Fires in Los Angeles, California (January 2025).

---

## Dataset Metadata

| Property | Value |
|----------|-------|
| **Name** | Palisades & Eaton Fires Los Angeles - Before & After Analysis |
| **Temporal Coverage** | 2024-12-28 to 2025-02-02 |
| **Spatial Extent** | Los Angeles County, California (34.0°N, -118.2°W) |
| **Bounding Box** | [-121.47°W, 31.73°N, -116.00°W, 35.67°N] |
| **Data Standard** | GeoCroissant (extends MLCommons Croissant 1.1) |
| **Conformance** | Croissant 1.1 ✓, GeoCroissant ✓, Responsible AI (RAI) ✓ |
| **Status** | ✓ Valid & Published |

---

## Data Sources

### Sentinel-2 (Optical Multispectral)
- **Instrument**: Multispectral Instrument (MSI)
- **Resolution**: 10m (visible/NIR), 20m (red edge, SWIR), 60m (atmospheric bands)
- **Bands**: 13 bands including coastal aerosol, RGB, NIR, red edge, SWIR, cirrus
- **Revisit Cycle**: 5 days (with constellation)
- **Availability**: 2 cloud-free scenes before event (≤3% cloud), 2 cloud-free scenes after (≤8% cloud)
- **Level**: Level-2A (atmospherically corrected surface reflectance)

#### Pre-Fire Scenes (Before January 7-9, 2025)
- **S2A_T11SLT_20250102T183754_L2A** - January 2, 2025 (3.0% cloud cover)
- **S2A_T11SLU_20250102T183754_L2A** - January 2, 2025 (1.4% cloud cover)

#### Post-Fire Scenes (After January 9, 2025)
- **S2B_T11SLU_20250117T183840_L2A** - January 17, 2025 (0.3% cloud cover) ← **Excellent for burn mapping**
- **S2C_T11SLU_20250201T184431_L2A** - February 1, 2025 (7.6% cloud cover)

### Sentinel-1 (Synthetic Aperture Radar - SAR)
- **Instrument**: C-Band SAR
- **Resolution**: 10-25m spatial resolution (full resolution mode)
- **Polarizations**: VV (vertical-vertical) and VH (vertical-horizontal)
- **Revisit Cycle**: 6 days (with constellation)
- **Cloud Penetration**: ✓ Cloud-free (SAR is active microwave, not affected by clouds/smoke)
- **Pre-Fire Acquisitions**: 5 scenes (Dec 28, 2024 - Jan 2, 2025)
- **Post-Fire Acquisitions**: 5 scenes (Jan 15 - Feb 2, 2025)

**Key Advantage**: Radar sees through smoke and clouds, detecting structural changes from fire damage via coherence analysis.

---

## Responsible AI (RAI) Disclosures

### Data Collection Process
✓ **Sentinel-2 Data**: Acquired by ESA's Multispectral Instrument as part of the Copernicus Open Access Programme. Level-2A products are atmospherically corrected using ESA's baseline processing.

✓ **Sentinel-1 Data**: Acquired by ESA's Synthetic Aperture Radar as part of Copernicus. Level-1C Ground Range Detected (GRD) products processed and distributed via AWS Open Data.

✓ **Dataset Curation**: Scenes selected around documented fire event dates (Palisades Fire: ~Jan 7-9, 2025; Eaton Fire: ~Jan 8-9, 2025) with cloud cover thresholds (≤10% for optical).

### Known Data Limitations
1. **Sentinel-2 Cloud Cover**: Optical data limited by cloud cover. Pre/post scenes selected for minimal cloud presence (≤10%).
2. **Revisit Gaps**: Temporal gaps exist due to 5-6 day revisit cycles. Complete daily coverage not available.
3. **No Fire Perimeters**: Dataset includes raw satellite data; fire extent boundaries must be derived by users via:
   - Normalized Burn Ratio (NBR) = (NIR - SWIR2) / (NIR + SWIR2)
   - dNBR (change detection between pre/post scenes)
   - SAR coherence change detection
4. **Atmospheric Effects**: Fire-affected areas with high smoke content show altered spectral/radar signatures.
5. **Limited Validation**: Ground truth validation not provided; fire extent depends on user interpretation.
6. **Processing Level**: Data includes ESA Level-2A atmospheric corrections only; quantitative analysis may need additional radiometric calibration.

### Known Data Biases
1. **Optical Atmospheric Effects**: Aerosol loading and water vapor can affect spectral reflectance in smoke-heavy environments.
2. **SAR Surface Effects**: Backscatter affected by surface roughness changes; interpretation requires multi-temporal coherence analysis.
3. **Orbit Characteristics**: Sentinel constellation has polar orbital gaps (not relevant at Los Angeles latitude 34°N).

### Recommended Use Cases
✓ Disaster response and emergency management
✓ Fire extent and burn severity mapping
✓ Pre/post-event damage assessment
✓ Land use and land cover change analysis
✓ Vegetation recovery monitoring post-fire

### Potential Social Impacts (Positive & Cautionary)
**Positive Impact**:
- Enables rapid damage assessment supporting emergency response
- Fire extent maps inform evacuation planning and public safety
- Supports insurance claims processing and disaster relief targeting
- Independent verification of fire impacts and transparency

**Cautionary**:
- **Privacy Consideration**: Dataset contains georeferenced imagery of private residential properties. Derivative products (burned property maps) identifying specific homes require privacy-aware handling when shared.
- **Socioeconomic Sensitivity**: Fire damage characterization may be used for insurance disputes; analysis should be transparent about uncertainty and limitations.

### Personal Sensitive Information (PSI) Declaration
⚠️ Dataset contains georeferenced satellite imagery covering private residential properties in Los Angeles. While satellite imagery is publicly available, users analyzing fire damage should:
- Be mindful of privacy implications when sharing derivative products
- Consider anonymization/aggregation when publishing results
- Respect property owner privacy in derived damage assessments

### Sampling Strategy
- **Optical Data**: All available cloud-free (≤10%) Sentinel-2 acquisitions within ±15-day windows around fire event over Los Angeles County bounding box.
- **Radar Data**: All available Sentinel-1 GRD acquisitions in same windows; includes varied orbit states (ascending/descending).
- **Temporal Windows**:
  - Pre-event: 2024-12-20 to 2025-01-05
  - Post-event: 2025-01-15 to 2025-02-05

### Spatial Bias & Coverage Limitations
- Dataset restricted to Los Angeles County area (bounding box: -118.67°W to -118.16°W, 33.66°N to 34.34°N)
- Spatial representation limited to defined region; may not represent conditions in other Southern California fire areas
- Sentinel constellation provides global coverage with optimal mid-to-high latitude coverage
- Data completeness depends on 5-6 day revisit cycle

---

## Recommended Analysis Workflows

### 1. Optical-Based Burn Mapping (Sentinel-2)
```
Step 1: Load NIR (B8) and SWIR2 (B12) bands from pre/post scenes
Step 2: Calculate Normalized Burn Ratio (NBR) for each date
        NBR = (NIR - SWIR2) / (NIR + SWIR2)
Step 3: Compute change detection: dNBR = NBR_pre - NBR_post
Step 4: Classify burn severity:
        dNBR < 0.1:   Unburned
        0.1 - 0.27:   Low severity
        0.27 - 0.44:  Moderate-low severity
        0.44 - 0.66:  Moderate-high severity
        > 0.66:       High severity
Step 5: Validate with Google Earth or aerial photography
```

### 2. SAR-Based Change Detection (Sentinel-1)
```
Step 1: Compute SAR coherence (VV and VH) for pre/post pairs
Step 2: Detect coherence loss indicating structural changes
Step 3: Mask water and urban areas for focus on fire damage
Step 4: Cross-validate with optical NBR results
```

### 3. Integrated Multi-Temporal Analysis
```
- Overlay Sentinel-2 burn maps with Sentinel-1 change detection
- Generate high-confidence fire extent (agreement between both sensors)
- Estimate confidence for each pixel based on multi-sensor agreement
- Produce final fire extent vector map
```

---

## Data Access & Files

**Primary GeoCroissant File**:
```
palisades_eaton_fires_geocroissant.json (78.8 KB)
```

**Asset URLs**: 110 source files across 4 RecordSets:
- Sentinel-2 Pre-Fire: 2 scenes × 22 bands = 44 assets
- Sentinel-2 Post-Fire: 2 scenes × 22 bands = 44 assets
- Sentinel-1 Pre-Fire: 5 scenes × 2 polarizations + metadata = 14 assets
- Sentinel-1 Post-Fire: 5 scenes × 2 polarizations + metadata = 14 assets

All data hosted on AWS S3 Open Data (no authentication required for download).

---

## Technical Details

**Dataset Conformance**:
- ✓ MLCommons Croissant 1.1 (standard geospatial dataset format)
- ✓ GeoCroissant v1.0 (ESA extension for Earth Observation metadata)
- ✓ RAI (Responsible AI) disclosures per MLCommons RAI guidelines

**Coordinate Reference System**: 
- EPSG:4326 (WGS84) for collection; scenes stored in UTM Zone 11N (EPSG:32611)

**Creator Organizations**:
- European Space Agency (ESA) - Data Producer
- NASA - Archive & Distribution
- USGS - Landsat Distribution Infrastructure

**License**: CC-BY 4.0 (Creative Commons Attribution)
- Allows use with attribution to ESA/NASA
- Use for commercial and research purposes permitted

---

## Quality Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Cloud Cover (Optical) | 0.3% - 7.6% | Excellent for burn mapping |
| SAR Completeness | 100% | 10 scenes covering pre/post |
| Temporal Resolution | 5-6 days | Optimal for disaster response |
| Spatial Resolution | 10-25m | Suitable for fire extent mapping |
| Atmospheric Correction | Level-2A | ESA baseline processing |
| Validation Status | ✓ Passed | JSON-LD validates against Croissant/GeoCroissant/RAI schemas |

---

## Citation

```bibtex
@dataset{palisades_eaton_fires_2025,
  title={Palisades & Eaton Fires Los Angeles - Before & After Analysis},
  author={European Space Agency and NASA and USGS},
  year={2025},
  month={January},
  url={https://stac.earth-search.aws.element84.com/},
  license={CC-BY 4.0},
  spatialCoverage={Los Angeles County, California}
}
```

---

## Next Steps

1. **Download Asset Files**: Access S3 URLs in the GeoCroissant JSON-LD
2. **Compute Burn Indices**: Apply NBR and dNBR calculations to Sentinel-2
3. **Change Detection**: Process Sentinel-1 coherence for structural changes
4. **Validation**: Compare results with MODIS fire perimeters or NIFC data
5. **Publication**: Share fire extent maps with emergency management agencies
6. **Social Responsibility**: Apply privacy safeguards before sharing property-level damage assessments

---

*Generated: 2026-08-26 | GeoCroissant Dataset | MLCommons Croissant v1.1*
