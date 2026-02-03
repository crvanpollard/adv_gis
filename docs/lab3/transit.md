---
title: "Lab 3: SEPTA Transit Stops within 500ft of a Park"
layout: default
parent: "Labs" # optional if you want a Labs parent page
nav_order: 3 # controls left‑nav order (lower = higher)
---

# Lab 3 — Identify SEPTA Transit Stops Within 500 Feet of Philadelphia Parks

Advanced GIS for Urban Planning & Development (Week 4 — Vector Analysis)

In this lab, you will perform a spatial proximity analysis in QGIS to determine which SEPTA bus stops are located within 500 feet of a Philadelphia park. You will use public spatial datasets from OpenDataPhilly, including SEPTA Transit stops and Philadelphia Parks & Recreation (PPR) properties.

---

# Required Datasets

- **PPR Properties:** https://opendataphilly.org/datasets/ppr-properties/
- **SEPTA Transit Stops:** https://opendataphilly.org/datasets/septa-routes-stops-locations/

---

# PART 1 — Project Setup

## 1. Create Project Workspace

Make a new folder for Lab 3.

## 2. New QGIS Project

Save as `Lab4_Parks_Transit_YourLastName.qgz`.

## 3. Download and Extract Data

Extract all shapefiles before loading.

## 4. Add Layers to QGIS

Use **Layer → Add Layer → Add Vector Layer**.

**Placeholder Image:**
![Loaded Layers](placeholder_loaded_layers.png)

---

# PART 2 — Coordinate System Alignment

## 5. Set Project CRS

Use **EPSG: 2272 — NAD83 / Pennsylvania South (ftUS)**.

**Placeholder Image:**
![CRS Window](placeholder_crs.png)

---

# PART 3 — Filtering

## 6. Inspect Attribute Tables

Review fields in SEPTA and PPR layers.

## 7. Filter Out Operational-Internal PPR Sites

Expression:

```
NOT "property_c" = 'OPERATIONAL_INTERNAL'
```

**Placeholder Image:**
![Filter Dialog](placeholder_filter.png)

## 8. Verify Filter Worked

Zoom to Fairmount Park.

**Placeholder Image:**
![Filtered Park](placeholder_filtered_park.png)

---

# PART 4 — Buffering Parks

## 9. Create a 500-ft Buffer

Use **Vector → Geoprocessing Tools → Buffer**.

**Placeholder Image:**
![Park Buffer](placeholder_buffer.png)

---

# PART 5 — Prevent Double Counting Bus Stops

## 10. Dissolve Bus Stops by StopId

Use **Dissolve** with `StopId`.

**Placeholder Image:**
![Dissolve Stops](placeholder_dissolve.png)

---

# PART 6 — Select Stops Within 500 Feet

## 11. Create Spatial Indexes

Right-click → Properties → Source → Create Spatial Index.

**Placeholder Image:**
![Spatial Index](placeholder_spatial_index.png)

## 12. Select by Location

Select stops _within_ `ppr_500ft_buffer`.

**Placeholder Image:**
![Select By Location](placeholder_select_location.png)

## 13. Export Selected Stops

Save as:

```
septa_stops_within_500ft_parks.shp
```

---

# PART 7 — Summaries & Mapping

## 14. Count Results

View attribute table.

## 15. Symbolize Layers

- Parks = Green
- Buffer = Light green
- All stops = Gray
- Selected stops = Blue

**Placeholder Image:**
![Final Map](placeholder_final_map.png)

## 16. Export Screenshot

Zoom to Center City and screenshot.

---

# Deliverables

- `septa_stops_within_500ft_parks.shp`
- Screenshot of results
- 1–2 paragraph summary
