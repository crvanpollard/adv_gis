---
title: "Lab 2: ACS 2023 Philadelphia"
layout: default
parent: "Labs" # optional if you want a Labs parent page
nav_order: 2 # controls left‑nav order (lower = higher)
---

# ACS 2023 Philadelphia Lab (tidycensus + tidyverse)

This lab walks you through acquiring **ACS 2023 5‑year** data for **Philadelphia County, PA** at the **tract** level, reshaping to **wide** format, computing useful indicators (e.g., **% zero‑vehicle households**), and visualizing results with **maps** and a **scatter plot**.

> **Prerequisites**
>
> - An R installation (4.1+ recommended) and RStudio
> - Obtain a Census API key: <a href="https://api.census.gov/data/key_signup.html" target="_new">https://api.census.gov/data/key_signup.html</a>

---

> **Resources**
>
> - Getting Started with R Studio <a href="https://docs.posit.co/ide/user/ide/get-started/" target="_new">https://docs.posit.co/ide/user/ide/get-started/</a>
> - Analyzing US Census Data: Methods, Maps, and Models in R (Klye Walker, 2023) <a href="https://walker-data.com/census-r/index.html" target="_new">https://walker-data.com/census-r/index.html</a>

---

## Step 1 — Install & Load Packages

```r
install.packages(c(
  "tidycensus",
  "tidyverse",
  "sf",
  "ggplot2",
  "scales"
))

# Keep viridis separate if your environment sometimes fails on vector installs
install.packages("viridis")

library(tidycensus)
library(tidyverse)
library(sf)
library(ggplot2)
library(scales)   # label_percent(), label_dollar()
library(viridis)  # (optional) additional palette helpers
```

> **Troubleshooting:** If `install.packages()` fails for a single package, run `install.packages("thatPackage")` by itself and retry `library(thatPackage)`.

---

## Step 2 — Set Your Census API Key

Get your key at the link above and paste it below. Use `install = TRUE` to store it for future R sessions.

```r
census_api_key("YOUR_API_KEY_HERE", install = TRUE)
# Restart R or run: library(tidycensus)
```

---

## Step 3 — Define ACS Variables

First you will want to pull or get several variables from the census api:

- **Total population** (`B01003_001`)
- **Median household income** (`B19013_001`)
- **Zero‑vehicle households** (`B08201_002`) and **total households** (`B08201_001`) so we can compute a percentage.

```r
philly_vars <- c(
  total_pop        = "B01003_001",
  med_income       = "B19013_001",
  zero_car         = "B08201_002",
  total_households = "B08201_001"
)
```

---

## Step 4 — Download ACS 2023 5‑Year Tract‑Level Data (Long)

Next you will download the above variables that you identified and view the results

```r
philly_acs <- get_acs(
  geography = "tract",
  variables = philly_vars,
  state     = "PA",
  county    = "Philadelphia",
  year      = 2023,
  survey    = "acs5",
  geometry  = TRUE
)

View(philly_acs)
```

`philly_acs` is an **sf tibble** in **long** format (one row per variable per tract).

View(philly_acs) is just a quick way to automatically open the results.

## Step 5 — Convert LONG → WIDE (no geometry)

We pivot to a single row per tract with variables as columns and compute **% zero‑vehicle households**.

```r
wide_tmp <- philly_acs %>%
  st_drop_geometry() %>%
  select(GEOID, NAME, variable, estimate)

philly_wide <- wide_tmp %>%
  pivot_wider(names_from = variable, values_from = estimate) %>%
  mutate(pct_zero_car = 100 * zero_car / total_households)
```

---

## Step 6 — Reattach Geometry (once)

```r
geom_tbl <- philly_acs %>%
  st_as_sf() %>%
  select(GEOID, geometry)

philly_wide <- philly_wide %>%
  left_join(geom_tbl, by = "GEOID") %>%
  st_as_sf()
```

**Verify:**

```r
class(philly_wide)        # should include "sf"
st_geometry(philly_wide)  # should print POLYGON/MULTIPOLYGON
```

---

## Step 7 — Map: % Zero‑Vehicle Households

```r
ggplot(philly_wide) +
  geom_sf(aes(fill = pct_zero_car), color = NA) +
  scale_fill_viridis_c(
    option = "plasma",
    labels = label_percent(scale = 1),
    name   = "% Zero-Car"
  ) +
  labs(
    title   = "Philadelphia: % Zero‑Vehicle Households (ACS 2023 5‑Year)",
    caption = "Source: U.S. Census Bureau, ACS"
  ) +
  theme_minimal()
```

> If you see `stat_sf() requires missing aesthetics: geometry`, make sure you ran **Step 6** and that `philly_wide` still has a `geometry` column.

---

## Step 8 — Scatter Plot: Income vs % Zero‑Vehicle Households

```r
ggplot(philly_wide, aes(x = med_income, y = pct_zero_car)) +
  geom_point(aes(size = total_pop), alpha = 0.6) +
  scale_y_continuous(labels = label_percent(scale = 1)) +
  scale_size(range = c(1, 6), guide = "none") +
  labs(
    title = "Income vs % Zero‑Vehicle Households (ACS 2023)",
    x = "Median Household Income ($)",
    y = "% Zero‑Vehicle Households"
  ) +
  theme_minimal()
```

(Optional) Add a trend line:

```r
last_plot() + geom_smooth(method = "lm", se = TRUE, color = "steelblue")
```

---

## Step 9 — Map: Median Household Income

This map will use a different color scale gradient

```r
ggplot(philly_wide) +
  geom_sf(aes(fill = med_income), color = NA) +
  scale_fill_gradientn(
    colors  = c("yellow", "green", "blue"),
    labels  = label_dollar(accuracy = 1),
    na.value = "grey85",
    name    = "Income ($)"
  ) +
  labs(
    title   = "Median Household Income by Census Tract (ACS 2023 5‑Year)",
    caption = "Source: U.S. Census Bureau, ACS 5‑Year Estimates (2023)"
  ) +
  theme_minimal(base_size = 12) +
  theme(
    legend.position = "right",
    plot.title = element_text(face = "bold")
  )
```

---

## Step 10 — Export Files

You can directly export to a CSV as well as shapefile to a local folder.

> File Path Naming Tips (Mac + Windows)
> - Use forward slashes `/` in paths
>  PC : `"C:/Users/<yourname>/Documents/acs_lab/data.csv"`
>  MAC : `"/Users/<yourname>/Documents/acs_lab/data.csv"`
> - Avoid saving directly to OneDrive or Google Drive as cloud‑synced folders sometimes lock files, rename paths, or block R from writing.
> - Avoid spaces and special characters in folder / file names.

Make sure to create a new folder for your project beforehand, so that you can easily find your exports when needing to work with your data. Notice how the below exports are going to the `project_output` location for lab_2.

CSV export:

```r
readr::write_csv(philly_wide %>% st_drop_geometry(), "C:/Users/<YOURNAME>/Documents/Jefferson/adv_gis/labs/lab_2/project_output/philly_income_2023.csv")
```

Shapefile export:

```r
sf::st_write(philly_wide, "C:/Users/<YOURNAME>/Documents/Jefferson/adv_gis/labs/lab_2/project_output/philly_acs_2023_wide.shp", delete_layer = TRUE)
```
---

## Step 11 — Save your R Script

Save your R script to your Lab 2 <- scripts folder.

---

## Checks & Common Fixes

- **`could not find function 'label_percent'`** → `library(scales)` or prefix with `scales::label_percent()`.
- **`could not find function 'ggplot'`** → `library(ggplot2)`.
- **Geometry errors in `geom_sf()`** → ensure **Step 6** completed and `philly_wide` is `sf`.
- **`unexpected end of input`** → close all quotes and parentheses in `install.packages(...)`.

---

## Attribution

This lab uses the R packages **tidycensus**, **tidyverse**, **sf**, and **ggplot2** to access and visualize public ACS data made available by the **U.S. Census Bureau**.
