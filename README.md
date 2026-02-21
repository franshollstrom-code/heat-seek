# Heat Seek – AI tool for mapping urban waste heat recovery potential

Final project for the Building AI course

## Summary
Heat Seek is an AI-driven analytical tool designed to map and prioritize industrial and commercial waste heat recovery. By generating a Heat Recovery Index (HRI), it identifies where excess thermal energy is most economically and technically viable to recycle into district heating networks.

## Background
* **The Problem:** Huge amounts of energy are wasted as heat in urban areas. The primary barrier to recovery is a lack of qualified, empirical data as cities often don't know which specific buildings offer the best return on investment for heat recovery.
* **Differentiation:** Unlike static tools like Hotmaps or Citiwatts that rely on statistical averages, Heat Seek uses live satellite thermal data to identify specific, high-potential sites that models based on building age alone overlook.
* **Motivation:** My motivation is to accelerate the circular economy. AI serves as the bridge between technical thermal data and actionable environmental investment, helping cities reach carbon neutrality faster.

## How is it used?
The tool bridges the gap between industrial waste and urban heat demand in various contexts:
* **Grid Cities (e.g., Nordics):** Utilities use the HRI to identify the most cost-effective connection points for pipeline extensions.
* **Off-grid Zones (e.g., Central Europe):** Planners use it to identify "Energy Clusters" for new localized 5th Generation District Heating (5GDH) networks.
* **Developers:** Used during the feasibility phase of new residential projects to find nearby "free" heat sources.

### Example Implementation (Python)
The following code demonstrates how we use linear regression to find buildings that "over-perform" in heat waste (high potential) compared to the neighborhood average:

```python
import numpy as np
from io import StringIO

def main():
    # Data: [Building_ID, Surface_Area, Ambient_Temp, Thermal_Intensity, Distance_to_Grid]
    data_string = """
    ID Area AmbTemp Intensity GridDist
    1 500 10 85 100
    2 1200 10 150 500
    3 800 10 210 50
    4 600 10 90 200
    5 2000 10 400 150
    """
    
    data = np.genfromtxt(StringIO(data_string), skip_header=1)
    
    # Features (X) and Target (y)
    X = data[:, 1:3]
    y = data[:, 3]
    grid_distances = data[:, 4]
    ids = data[:, 0]
    
    # Fit linear regression to find "expected" heat for a building size
    c = np.linalg.lstsq(X, y, rcond=None)[0]
    
    # Calculate Heat Recovery Index (HRI)
    predicted_intensity = X @ c
    waste_excess = y - predicted_intensity
    
    hri_scores = []
    for i in range(len(ids)):
        # Heuristic: Excess waste relative to grid distance
        score = waste_excess[i] / (grid_distances[i] + 1)
        hri_scores.append({"Building_ID": int(ids[i]), "HRI_Score": round(score, 3)})
    
    hri_scores.sort(key=lambda x: x['HRI_Score'], reverse=True)
    
    print("Top Heat Recovery Prospects:")
    for site in hri_scores:
        print(site)

main()
```

## Data sources and AI methods
The project combines multiple open data streams to create a reliable assessment of heat recovery potential.

| Source | Usage |
| ----------- | ----------- |
| [ESA Sentinel-3 / NASA Landsat-8](https://landsat.gsfc.nasa.gov/) | Thermal Infrared (TIR) data to detect surface temperature anomalies. |
| [OpenStreetMap (OSM)](https://www.openstreetmap.org/) | Building metadata to categorize sources and filter out non-building heat (e.g. asphalt). |
| [OpenWeatherMap](https://openweathermap.org/) | Real-time ambient temperature data to normalize thermal signatures. |
| [Hotmaps](https://www.hotmaps-project.eu/) | Used as a strategic baseline for existing heat density estimations. |

**AI Techniques:**
* **Computer Vision (U-Net):** Used for image segmentation to draw precise boundaries around thermal plumes and match them to building footprints.
* **Linear Regression:** To estimate the gap between "expected" heat loss (based on building size) and "observed" heat loss, identifying inefficient sites with high recovery potential.
* **Anomaly Detection:** To separate constant industrial process heat from temporary peaks or seasonal variations.

## Challenges
* **Resolution Limits:** Public satellite data has a limited pixel density (e.g., 30m-100m). Identifying small-scale heat sources like local bakeries or laundries may require high-resolution commercial data or drone-based scans.
* **Data Propriety:** Detailed maps of district heating networks are often proprietary for security reasons. Heat Seek addresses this by using a modular "private-data" layer where utilities can upload their own secure maps.
* **Physical Verification:** While AI can identify heat and proximity, it cannot determine if a building's internal HVAC system is technically compatible with heat recovery without an on-site audit.

## What next?
The project is designed to scale into a **Circular Energy Marketplace**. Future iterations could include:
1.  **Virtual Thermal Plants:** AI-driven bundling of multiple small heat producers (e.g., supermarkets and server rooms) to provide a steady, combined heat flow for an entire neighborhood.
2.  **Investment Simulator:** A tool for energy companies to calculate the ROI of new pipeline extensions based on the HRI scores of surrounding buildings.
3.  **Real-time Monitoring:** Integrating IoT sensors from industrial sites to provide real-time updates to the Heat Recovery
