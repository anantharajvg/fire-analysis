# Reusable Workflow: Fire Event Satellite Analysis with GeoCroissant

## Overview
This document describes the complete workflow used to create a GeoCroissant dataset for the Palisades & Eaton Fires. You can adapt this for any fire event globally using publicly available Sentinel data.

---

## Quick Start Template

Copy this prompt and fill in the bracketed parameters for your fire event:

```
Let us investigate a fire event, [FIRE_NAME], in the [LOCATION], [COUNTRY/STATE]. 
We need to identify publicly available satellite images and the fire extent, 
before and after the event. The images should be cloud free, and the temporal 
and spatial extents should match the event, before and after.
```

**Example fills:**
- `[FIRE_NAME]` → "Camp Fire", "Dixie Fire", "Maui Lahaina Fire"
- `[LOCATION]` → "Northern California", "Maui/Hawaii", "Paradise County"
- `[COUNTRY/STATE]` → "California", "Hawaii", "Greece"

---

## Step-by-Step Workflow

### Phase 1: Location & Event Research

**What I did:**
1. Used `geocode_place` to convert human place name → bounding box
2. Identified fire dates (Palisades: ~Jan 7-9, 2025; Eaton: ~Jan 8-9, 2025)
3. Defined temporal windows:
   - Pre-event: 15 days before fire start
   - Post-event: 15-30 days after fire start

**How to adapt:**
```
Questions to answer for your fire event:
- Exact fire start date(s)? (check news, USGS NIFC, InciWeb, NASA FIRMS)
- Geographic center & approximate size?
- Region name for geocoding? (city, county, state)
- Expected cloud cover at that time of year?
```

**Tools used:**
```python
mcp__geocr__geocode_place(
    place_name="Los Angeles, California",
    limit=3  # Get top 3 candidates to choose from
)
```

---

### Phase 2: Data Catalog Discovery

**What I did:**
1. Listed all EO catalogs: `list_eo_catalogs()`
   - Found: Earth Search (Element84/AWS), NASA VEDA
2. Searched available datasets: `search_eo_datasets(limit=50)`
   - Identified: Sentinel-2 (optical), Sentinel-1 (SAR), Landsat

**How to adapt:**
```python
# Always start with catalog discovery
mcp__geocr__list_eo_catalogs()  # See what's available

# Then search datasets in your region
mcp__geocr__search_eo_datasets(
    catalog_id="earth-search",  # or "veda" for NASA data
    limit=50  # List all collections
)
```

**Key datasets for fire analysis:**
| Sensor | Type | Resolution | Revisit | Cloud? | Best For |
|--------|------|-----------|---------|--------|----------|
| Sentinel-2 | Optical | 10-20m | 5 days | ✗ Limited | Burn severity, NBR indices |
| Sentinel-1 | SAR Radar | 10-25m | 6 days | ✓ Yes | Change detection, coherence |
| Landsat 8/9 | Optical | 30m | 16 days | ✗ Limited | Coarser resolution alternative |

---

### Phase 3: Scene Availability Assessment

**What I did:**
1. Counted available scenes: `count_eo_scenes()` for each temporal window
   - Pre-fire Sentinel-2: 2 scenes (cloud ≤10%)
   - Post-fire Sentinel-2: 2 scenes (cloud ≤10%)
   - Pre-fire Sentinel-1: 11 scenes (no cloud constraint)
   - Post-fire Sentinel-1: 7 scenes

**How to adapt:**
```python
# Count scenes BEFORE searching (saves time)
mcp__geocr__count_eo_scenes(
    bbox=[-118.67, 33.66, -118.16, 34.34],  # Your bounding box
    collections=["sentinel-2-c1-l2a"],
    datetime_range="2024-12-20/2025-01-05",  # Pre-event window
    max_cloud_cover=10  # Cloud threshold for optical
)

# If count is 0, try:
# 1. Extend temporal window (±30 days instead of ±15)
# 2. Increase cloud_cover threshold (15-20%)
# 3. Switch to Sentinel-1 (no cloud issues) or Landsat
# 4. Try NASA VEDA catalog instead of Earth Search
```

---

### Phase 4: Scene Discovery & Selection

**What I did:**
1. Searched for actual scenes: `search_eo_scenes()`
2. Selected cloud-free acquisitions
3. Recorded scene IDs, dates, cloud cover, sun elevation

**How to adapt:**
```python
# Search with same parameters as count
mcp__geocr__search_eo_scenes(
    bbox=[-118.67, 33.66, -118.16, 34.34],
    collections=["sentinel-2-c1-l2a"],
    datetime_range="2024-12-20/2025-01-05",
    max_cloud_cover=10,
    limit=5  # Get top 5 scenes
)

# Look for scenes with:
# ✓ Cloud cover < 5% (ideal) or < 10% (acceptable)
# ✓ High sun elevation (>25°) for better imagery
# ✓ Close to fire dates (not too far before/after)
```

---

### Phase 5: Multi-Source GeoCroissant Generation

**What I did:**
1. Created multi-source dataset combining:
   - Sentinel-2 pre-fire scenes
   - Sentinel-2 post-fire scenes
   - Sentinel-1 pre-fire scenes
   - Sentinel-1 post-fire scenes
2. Added comprehensive RAI (Responsible AI) disclosures

**How to adapt:**

```python
mcp__geocr__create_geocroissant_from_stac_sources(
    name="[FIRE_NAME] - Before & After Analysis",
    description="[Your description]",
    sources=[
        {
            "source_id": "sentinel2-before",
            "catalog_id": "earth-search",
            "collection_id": "sentinel-2-c1-l2a",
            "bbox": [-118.67, 33.66, -118.16, 34.34],
            "datetime_range": "PRE_EVENT_DATES",
            "max_cloud_cover": 10,
            "limit": 5
        },
        {
            "source_id": "sentinel2-after",
            "catalog_id": "earth-search",
            "collection_id": "sentinel-2-c1-l2a",
            "bbox": [-118.67, 33.66, -118.16, 34.34],
            "datetime_range": "POST_EVENT_DATES",
            "max_cloud_cover": 10,
            "limit": 5
        },
        {
            "source_id": "sentinel1-before",
            "catalog_id": "earth-search",
            "collection_id": "sentinel-1-grd",
            "bbox": [-118.67, 33.66, -118.16, 34.34],
            "datetime_range": "PRE_EVENT_DATES",
            "limit": 5
        },
        {
            "source_id": "sentinel1-after",
            "catalog_id": "earth-search",
            "collection_id": "sentinel-1-grd",
            "bbox": [-118.67, 33.66, -118.16, 34.34],
            "datetime_range": "POST_EVENT_DATES",
            "limit": 5
        }
    ],
    creators=["European Space Agency (ESA)", "NASA", "USGS"],
    license="https://creativecommons.org/licenses/by/4.0/",
    
    # RAI Disclosures - CRITICAL
    data_use_cases=[
        "Fire extent and burn severity mapping",
        "Disaster response and emergency management",
        "Pre/post-event damage assessment",
        "Land use and land cover change analysis",
        "Vegetation recovery monitoring"
    ],
    data_limitations=[
        # Fill based on your findings
    ],
    data_biases=[
        # Document known biases
    ],
    data_collection="[Describe how data was collected/selected]",
    data_social_impact="[Discuss positive/negative impacts]",
    personal_sensitive_information="[Note privacy considerations]",
    sampling_strategy="[Explain temporal/spatial sampling]",
    spatial_bias="[Discuss spatial representativeness]",
    
    output_filename="[fire_name_geocroissant.json"
)
```

---

## Key Parameters to Adapt

### 1. Location (Bounding Box)
```
The geocode_place tool returns a bounding box automatically:
[min_lon, min_lat, max_lon, max_lat]

Example: Los Angeles = [-118.67, 33.66, -118.16, 34.34]

For different regions:
- California coastal: [-125, 32, -117, 42]
- Greece (Mediterranean): [20, 36, 28, 41]
- Australia (e.g., Victoria): [141, -39, 150, -34]
```

### 2. Temporal Windows
```
PRE-EVENT:
- Start: 15-30 days before fire (depending on season/cloud patterns)
- End: 1-2 days before fire onset

POST-EVENT:
- Start: 1-3 days after fire (allow time for satellite acquisition)
- End: 30-60 days after (for recovery/damage assessment)

Format: "YYYY-MM-DD/YYYY-MM-DD"
Example: "2024-12-20/2025-01-05"
```

### 3. Cloud Cover Thresholds
```
Optical data only (Sentinel-2, Landsat):
- Ideal: ≤5% cloud cover
- Acceptable: ≤10% cloud cover
- Compromised: 10-20% cloud cover
- Unusable: >20% cloud cover

SAR data (Sentinel-1):
- No cloud cover constraint (active microwave)
- Use max_cloud_cover: null or omit parameter
```

### 4. Responsible AI Disclosures

For each fire event, document:

**Data Limitations:**
- Cloud cover availability in that region/season
- Revisit cycles (Sentinel-2: 5 days, Sentinel-1: 6 days)
- Whether fire perimeters are user-derived
- Atmospheric conditions during fire period
- Data processing level and corrections applied

**Known Biases:**
- Seasonal atmospheric aerosols/water vapor
- SAR sensitivity to surface roughness
- Orbital coverage gaps (if applicable)
- Processing artifacts or sensor characteristics

**Use Cases:**
- Document intended uses (disaster response, research, insurance, etc.)
- Note who benefits from this analysis

**Social Impact:**
- Positive: Rapid assessment, transparency, emergency support
- Caution: Privacy of affected residents, socioeconomic sensitivity

**Personal Sensitive Information:**
- Does the bounding box include private homes/property?
- What are privacy implications of sharing derivative products?

---

## File Organization Template

```
fire-analysis-[EVENT-NAME]/
├── README.md                                  # Event overview
├── geocroissant_[event].json                 # Main GeoCroissant file
├── analysis_[event].md                        # Full documentation
├── WORKFLOW.md                                # Steps taken
├── data/
│   ├── sentinel2_pre/                        # Downloaded optical pre-fire
│   ├── sentinel2_post/                       # Downloaded optical post-fire
│   ├── sentinel1_pre/                        # Downloaded SAR pre-fire
│   └── sentinel1_post/                       # Downloaded SAR post-fire
├── results/
│   ├── nbr_pre.tif                          # Calculated indices
│   ├── nbr_post.tif
│   ├── dnbr_change.tif
│   ├── burn_severity_map.tif
│   └── fire_extent.shp                      # Final vector output
└── scripts/
    ├── calculate_nbr.py                      # Reusable analysis code
    ├── coherence_analysis.py
    └── generate_report.py
```

---

## Reproducing Your Workflow

### Option 1: Use the Same Prompt Template

Copy this exact prompt for a new fire event (just change the bracketed parts):

```
Let us investigate a fire event, [FIRE_NAME], in the [LOCATION], [COUNTRY]. 
We need to identify publicly available satellite images and the fire extent, 
before and after the event. The images should be cloud free, and the temporal 
and spatial extents should match the event, before and after.
```

### Option 2: Create a Python Script

```python
from geocroissant_client import GeoCroissantMCP

def create_fire_event_dataset(
    fire_name,
    location,
    fire_start_date,
    fire_end_date,
    pre_event_days=15,
    post_event_days=30
):
    """
    Reusable function to create GeoCroissant fire datasets
    
    Args:
        fire_name: "Palisades Fire", "Camp Fire", etc.
        location: "Los Angeles, California"
        fire_start_date: "2025-01-07"
        fire_end_date: "2025-01-09"
        pre_event_days: Days before fire to include
        post_event_days: Days after fire to include
    """
    
    # Step 1: Geocode location
    bbox = geocode_place(location)
    
    # Step 2: Define temporal windows
    pre_start = subtract_days(fire_start_date, pre_event_days)
    pre_end = subtract_days(fire_start_date, 1)
    post_start = add_days(fire_end_date, 1)
    post_end = add_days(fire_end_date, post_event_days)
    
    # Step 3: Count scenes
    s2_pre_count = count_eo_scenes(
        bbox, ["sentinel-2-c1-l2a"], 
        f"{pre_start}/{pre_end}", 
        max_cloud_cover=10
    )
    
    # Step 4: Create GeoCroissant
    create_geocroissant_from_stac_sources(
        name=f"{fire_name} - Before & After Analysis",
        sources=[
            {"source_id": "s2-before", "collection_id": "sentinel-2-c1-l2a",
             "bbox": bbox, "datetime_range": f"{pre_start}/{pre_end}"},
            {"source_id": "s2-after", "collection_id": "sentinel-2-c1-l2a",
             "bbox": bbox, "datetime_range": f"{post_start}/{post_end}"},
            {"source_id": "s1-before", "collection_id": "sentinel-1-grd",
             "bbox": bbox, "datetime_range": f"{pre_start}/{pre_end}"},
            {"source_id": "s1-after", "collection_id": "sentinel-1-grd",
             "bbox": bbox, "datetime_range": f"{post_start}/{post_end}"}
        ]
    )

# Usage:
create_fire_event_dataset(
    fire_name="Palisades Fire",
    location="Los Angeles, California",
    fire_start_date="2025-01-07",
    fire_end_date="2025-01-09"
)
```

### Option 3: Save as Claude Memory

I can save this workflow to my memory system so I remember your preferences for future fire event analyses. Just ask: **"Remember my fire event analysis workflow"** and I'll store key preferences like:
- Preferred temporal windows
- Cloud cover thresholds
- RAI disclosure preferences
- Output file naming conventions

---

## Troubleshooting Guide

| Issue | Solution |
|-------|----------|
| **No cloud-free scenes found** | Extend temporal window (±30 days), increase cloud threshold (15-20%), try SAR data (Sentinel-1) |
| **Bounding box too large** | Reduce to specific county/region instead of entire state |
| **Dates are uncertain** | Search NASA FIRMS (Fire Information for Resource Management System) or NIFC InciWeb |
| **Different country** | Same workflow works globally; Sentinel covers 56°S to 84°N |
| **Need finer detail** | Try NAIP (1m res, USA only), Planet Labs (commercial) |
| **RAI requirements unclear** | Document: what data you used, where it came from, what you didn't include, known limitations |

---

## Advanced: Regional Variations

### Mediterranean Fires (Greece, Spain, Turkey)
```
Challenges:
- High summer temperatures → dust/aerosol haze
- Frequent cloud cover Aug-Sep
- Solution: Use SAR (Sentinel-1) as primary, optical as secondary

Temporal window: ±20 days (shorter season)
Cloud threshold: 15-20% (may need to accept haze)
```

### Australian Bushfires
```
Challenges:
- Large extent (analyze by region)
- Dry season (May-Nov) → better cloud cover
- Solution: Landsat 8/9 for coarser 30m resolution over large areas

Temporal window: ±30 days
Cloud threshold: ≤10% (austral winter is clear)
```

### Amazon Wildfires
```
Challenges:
- Persistent cloud cover (wet season)
- Rapid regrowth (vegetation recovery)
- Solution: SAR as primary (Sentinel-1), extended temporal windows

Temporal window: ±45 days (account for clouds)
Cloud threshold: 20%+ (accept some haze)
```

---

## Next Steps After GeoCroissant Creation

Once you have the GeoCroissant file, you can:

1. **Download satellite data** from asset URLs in the JSON
2. **Calculate burn indices** (NBR, dNBR) on Sentinel-2
3. **Perform coherence analysis** on Sentinel-1 SAR
4. **Generate fire extent maps** using spectral thresholds
5. **Validate** against official fire perimeters (NIFC, FIRMS)
6. **Share** GeoCroissant file with researchers/agencies

---

## Resources

- **USGS NIFC**: https://www.nifc.blm.gov/ (fire perimeters, statistics)
- **NASA FIRMS**: https://firms.modaps.eosdis.nasa.gov/ (active fire detection)
- **InciWeb**: https://inciweb.nwcg.gov/ (incident information)
- **GeoCroissant Spec**: MLCommons croissant-geo v1.0
- **Sentinel-2 Handbook**: ESA official documentation
- **SAR Analysis**: ESA Sentinel-1 toolbox guide

---

*Workflow Template v1.0 | Created: 2026-08-26 | For fire events globally*
