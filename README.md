# Heat Seek – AI-powered urban heat recovery index

Final project for the Building AI course

## Summary
Heat Seek is an AI-driven tool designed to map and prioritize industrial waste heat recovery. By analyzing satellite thermal imagery and building metadata, it generates a **Heat Recovery Index (HRI)** to pinpoint where excess energy can be most efficiently recycled into district heating networks.

## Background
* **The Problem:** Massive amounts of energy are wasted as heat in urban areas, but we lack granular, empirical data to identify which specific buildings offer the best return on investment for heat recovery.
* **The Challenge of "False Positives":** Urban environments are full of warm surfaces, like asphalt parking lots or dark roofs. AI is needed to distinguish between active industrial waste and passive solar absorption.
* **Motivation:** To accelerate the circular economy. I see AI as the bridge between technical thermal data and actionable environmental investment.
* **Importance:** Heat recovery is often the most cost-effective way to reduce urban carbon emissions, requiring a "bird's-eye view" that humans cannot process manually.

## How is it used?
The tool follows a clear three-step workflow for city planners and energy utility managers:

1. **Regional Selection:** The user selects a geographic area for analysis.
2. **AI Analysis:** The system extracts **thermal gradients** from satellite data and cross-references them with OpenStreetMap metadata to filter out sun-heated surfaces (like asphalt).
3. **Prioritization:** The tool generates a ranked list of "Top Prospects" based on the HRI score (High waste + Proximity to existing grids).

### Example Implementation (Python)
The following code demonstrates the core logic of calculating the HRI, including basic normalization against ambient temperatures:

```python
import numpy as np

# Data format: [Building_ID, Area, Surface_Temp, Ambient_Temp, Distance_to_Grid]
# Note: In a real scenario, Surface_Temp is derived from thermal gradients.
data = np.array([
    [1, 500, 85, 10, 100],  # Industrial site
    [2, 1200, 45, 10, 500], # Large warehouse
    [3, 800, 95, 10, 50]    # Data center near grid
]) 

X = data[:, 1:2] # Building Area
# We normalize intensity by subtracting ambient temp to find the "active" heat plume
y = data[:, 2] - data[:, 3] 

# Linear regression to find "expected" heat loss based on size
c = np.linalg.lstsq(X, y, rcond=None)[0]
predicted = X @ c
excess_waste = y - predicted

# Heat Recovery Index (HRI): High excess waste / distance to grid
hri = excess_waste / (data[:, 4] + 1)

print("Priority Sites (HRI Score):")
for i, score in enumerate(hri):
    print(f"Building ID {int(data[i,0])}: {score:.3f}")
```

## Data sources and AI methods
**Data sources:**
* **ESA Sentinel-3 / NASA Landsat-8:** Public thermal infrared (TIR) data used to detect surface temperature anomalies.
* **OpenStreetMap (OSM):** Used to verify building footprints and filter out "false positives" like sun-heated asphalt parking lots.
* **Weather APIs:** Provides real-time ambient temperature data to normalize heat signatures against seasonal and daily variations.

**AI Techniques:**
* **Computer Vision (Segmentation):** Used to identify the sharp edges of thermal plumes. High **thermal gradients** (rapid temperature changes at building boundaries) help the AI distinguish active industrial processes from passive solar gain.
* **Linear Regression:** Used to estimate the "unnatural" heat output by comparing observed thermal signatures to a building's physical size and category.

## Challenges
* **Resolution Limits:** Public satellites have limited pixel density (30m–100m). Smaller heat sources, such as local bakeries or laundries, may require high-resolution commercial data or drone scans.
* **The "Asphalt Problem":** Dark urban surfaces absorb solar heat, which can mimic waste heat. Heat Seek addresses this by using building metadata to ensure analysis is focused only on rooftops and exhaust points.
* **Technical Compatibility:** While AI can identify heat and proximity to grids, it cannot determine if a factory's internal plumbing is compatible with recovery without an on-site engineering audit.

## What next?
The project could evolve into a **Circular Energy Marketplace**. Future iterations could include:
1. **Generative Pipeline Routing:** Automatically suggesting the most cost-efficient routes to connect high-HRI buildings to existing heat networks.
2. **Virtual Power Plants:** Using AI to bundle multiple small-scale heat producers (e.g., supermarkets and server rooms) into a steady, combined energy flow for entire neighborhoods.

## Acknowledgments
* Inspired by the [Building AI course](https://buildingai.elementsofai.com/) by Reaktor and the University of Helsinki.
* Methodology influenced by open-source projects like [Hotmaps](https://www.hotmaps-project.eu/) and [Citiwatts](https://citiwatts.eu/).
