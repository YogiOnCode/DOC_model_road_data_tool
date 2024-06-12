# Vehicle Path Estimation with Geospatial Data (DOC Model)

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![HERE API](https://img.shields.io/badge/HERE-Route%20Matching%20v8-00AFAA)
![Pandas](https://img.shields.io/badge/pandas-150458?logo=pandas&logoColor=white)

Code from my Master's thesis, **"Advancing Vehicle Path Estimation Using Geospatial Data Analysis"**.
The tool turns raw GPS traces from a vehicle into a **Deterministic Operating Cycle (DOC)** model: a
distance-indexed description of the road (elevation, gradient, curvature, speed limits, traffic signs,
weather) that can be used for energy-consumption analysis and residual range estimation.

---

## How it works

```
GPS trace (.txt)  ──►  HERE Route Matching API  ──►  JSON road attributes
                                                          │
                     DOC model (.csv) + path plot  ◄──  clean, align, compute distances
```

1. **Read** latitude/longitude points from a text file.
2. **Match** the trace to the road network with the HERE Route Matching v8 API, requesting ADAS,
   speed-limit, traffic-sign, traffic-pattern and archived-weather attributes.
3. **Parse** the JSON response (`jsonpath-ng`) and snap the returned link geometry to the input points.
4. **Compute** cumulative distance between consecutive points (Haversine and geodesic/Vincenty via `pyproj`).
5. **Clean** missing values (e.g. fill speed limits from neighbouring links and free-flow speed).
6. **Export** the DOC model as CSV, along with a report and a plot of the vehicle path.

## Getting started

```bash
git clone https://github.com/YogiOnCode/DOC_model_road_data_tool.git
cd DOC_model_road_data_tool
pip install -r requirements.txt
```

You need a [HERE developer](https://developer.here.com/) API key.

1. In `DOC_Model.py`, set `api_key` to your HERE API key.
2. Set `input_directory` (at the bottom of the file) to a folder containing your `.txt` GPS traces.
   Output is written to the same folder by default (`output_directory`).
3. Run:

```bash
python DOC_Model.py
```

Every `.txt` file in the input folder is processed.

### Input format

```text
Latitude, Longitude
47.37532819752522, 8.588794536964125
45.458435679351616, 9.185317763784466
```

See [`sample_input.txt`](sample_input.txt).

## DOC output

| Attribute | Description |
| --- | --- |
| Distance (m) | Cumulative geodesic distance along the route. |
| Latitude / Longitude (deg) | High-precision WGS84 coordinates along each link. |
| Elevation (m) | Height above the WGS84 ellipsoid. |
| Gradient (deg) | Vertical road direction; missing values are set to 0. |
| Heading (deg) | Horizontal road heading; missing values are set to 0. |
| Curvature (1/m) | 1 / radius at each point; missing values are set to 0. |
| Speed limit (m/s) | Applicable speed limit; gaps filled from adjacent links and free-flow speed. |
| Free-flow speed (m/s) | Static average travel speed for the link. |
| Traffic signal, Stop, Yield, Pedestrian crossing | Binary flags for signs present on the link. |
| Wind direction (deg) / Wind velocity (m/s) | From archived weather data (nullable). |

[`dOCformat.pdf`](dOCformat.pdf) documents the format, and [`Illustration.pdf`](Illustration.pdf) shows an example.

## Applications

- **Residual range estimation:** how far the vehicle can travel on its remaining battery or fuel.
- **Energy consumption analysis:** using accurate distance plus road gradient, curvature and speed profile.
- **Simulation and testing:** realistic road conditions for vehicle-dynamics simulations.

## Tech stack

Python · pandas · NumPy · pyproj · haversine · jsonpath-ng · Matplotlib · Requests · HERE Route Matching API

## Repository structure

```
├── DOC_Model.py        # Main pipeline: API call, parsing, distance calc, export
├── requirements.txt
├── sample_input.txt    # Example GPS trace
├── dOCformat.pdf       # DOC format specification
└── Illustration.pdf    # Example output / illustration
```

## Author

**Yogeswaran Amsavalli** · [GitHub](https://github.com/YogiOnCode)
