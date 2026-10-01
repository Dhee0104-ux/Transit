# TransitLens — Public Transport Analytics

TransitLens is a Google Colab project for exploring public-transport network data. It accepts either a **GTFS ZIP file**, an **Excel workbook**, or built-in sample data, then generates interactive maps, route statistics, passenger-demand forecasts, and downloadable outputs.

## Features

- **Choose your data format:** GTFS ZIP, Excel (`.xlsx` / `.xls`), or sample data.
- **Load GTFS ZIP files:** extracts the archive and searches nested folders for GTFS files.
- **Load Excel workbooks:** reads recognized sheets such as `stops`, `routes`, `trips`, `stop_times`, and `ridership`.
- **Clean stop and route data:** standardizes column names and validates coordinates.
- **Interactive maps:** displays bus stops and draws route lines when route-to-stop data is available.
- **Route statistics:** counts stops and estimates route distance from consecutive stop coordinates.
- **Passenger-demand forecasting:** trains a Random Forest model and forecasts demand for the next seven days when usable ridership data is available.
- **Underserved-area indicators:** highlights grid cells with relatively few supplied stops.
- **Journey-time estimates:** estimates travel time using a configurable average-speed assumption.
- **Exports:** saves CSV tables, a GeoJSON stop file, and interactive HTML maps, then packages the outputs into a ZIP file.

## Run in Google Colab

1. Open [Google Colab](https://colab.research.google.com/).
2. Create a new notebook.
3. Paste the complete TransitLens code into a single code cell.
4. Run the cell and wait for the libraries to install.
5. Select one of the data-source options:
   - **GTFS ZIP file (.zip)**
   - **Excel workbook (.xlsx / .xls)**
   - **Use built-in sample data**
6. Click **Choose Data and Render**.
7. If you selected ZIP or Excel, upload the matching file when prompted.
8. Review the maps, tables, charts, and model metrics. The generated results ZIP will be offered for download.

## Data formats

### 1. GTFS ZIP

A GTFS ZIP commonly contains these files:

| File | Purpose |
|---|---|
| `stops.txt` | Stop IDs, names, and coordinates |
| `routes.txt` | Route IDs and route names |
| `trips.txt` | Trips associated with routes |
| `stop_times.txt` | Stops and their sequence within trips |
| `shapes.txt` | Optional geographic route shapes |

The code requires `stops.txt` and `routes.txt`. Route lines and route-level statistics need usable trip and stop-time relationships. `shapes.txt` is optional; the current implementation connects stop coordinates in sequence rather than drawing GTFS shape geometry.

### 2. Excel workbook

The workbook can contain multiple sheets. Use these recommended sheet names and columns:

| Sheet | Recommended columns |
|---|---|
| `stops` | `stop_id`, `stop_name`, `stop_lat`, `stop_lon` |
| `routes` | `route_id`, `route_short_name`, `route_long_name` |
| `trips` | `route_id`, `trip_id`, `service_id` |
| `stop_times` | `trip_id`, `stop_id`, `stop_sequence`, `arrival_time` |
| `ridership` (optional) | `date`, `route_id`, `passengers` |

Column names are normalized to lowercase and spaces are replaced with underscores. The stops sheet must include latitude and longitude fields. Common alternatives such as `latitude`, `longitude`, `lat`, `lon`, and `lng` are supported.

For route analysis, the workbook should include `trips` and `stop_times` sheets. If your data uses a different structure, adapt the loading and route-joining functions to match it.

### 3. Ridership data

For actual passenger-demand forecasting, provide historical ridership data with:

- `date`: date of observation
- `route_id`: route identifier matching the route data
- `passengers`: passenger count for that date and route

GTFS usually describes the transit network and schedule; it does **not** normally provide actual passenger counts. If no usable ridership sheet is supplied, the notebook generates synthetic passenger data for demonstration.

## Outputs

The code writes results to `/content/transitlens_outputs/` and packages them as `TransitLens_Results.zip`.

| Output | Description |
|---|---|
| `cleaned_stops.csv` | Cleaned stop records |
| `cleaned_routes.csv` | Cleaned route records |
| `route_statistics.csv` | Stop counts and estimated route distances |
| `ridership_data.csv` | Ridership data used by the analysis, if available |
| `demand_forecast.csv` | Forecast passenger counts for the next seven days |
| `underserved_areas.csv` | Grid cells flagged as potentially underserved |
| `bus_stops.geojson` | Stop coordinates in GeoJSON format |
| `bus_stop_map.html` | Interactive bus-stop map |
| `route_map.html` | Interactive route map |
| `underserved_areas.html` | Interactive map of flagged areas, when any are found |

## Methodology and limitations

- **Distance:** estimated by summing straight-line distances between sequential stop coordinates. This can differ substantially from the actual road distance.
- **Journey time:** estimated from distance and an assumed average speed of 20 km/h. It does not include live traffic, dwell time, transfers, or waiting.
- **Demand forecasting:** uses a Random Forest model with route and calendar features. Results depend on the quantity and quality of historical data. Synthetic data produces demonstration results, not real-world predictions.
- **Evaluation:** MAE and RMSE are calculated when enough records are available. These metrics are only meaningful when the evaluation data represents the intended use case.
- **Underserved areas:** the current method flags geographic grid cells with relatively few stops among the supplied stop points. It does not account for population, walking distance, road barriers, service frequency, or travel demand.
- **Route geometry:** the current map connects stop coordinates in sequence. It does not use `shapes.txt` to trace the actual road path.
- **Geographic assumptions:** the sample data is centered around Hyderabad. If adapting projected-distance analysis to another city, use an appropriate local coordinate reference system.

## Troubleshooting

**The notebook says `stops.txt` was not found**
- Confirm that you selected the ZIP option and uploaded a valid GTFS ZIP.
- Check that the ZIP contains `stops.txt` and `routes.txt`, possibly inside a nested folder.

**The stop map is empty or stops are missing**
- Check that the stop sheet/file has valid latitude and longitude values.
- Confirm coordinates are in decimal degrees, not projected coordinates.

**Route lines do not appear**
- Confirm that route, trip, trip ID, stop ID, and stop-sequence fields match across the data.
- Route lines cannot be inferred reliably from a stops-only dataset.

**Forecasts look unrealistic**
- Check that `date`, `route_id`, and `passengers` are correct and that passenger counts are real observations.
- Ensure the route IDs in ridership match the route IDs in the network data.

**Excel data does not load as expected**
- Use the recommended sheet names and column names above.
- Make sure the workbook is a valid `.xlsx` or `.xls` file.

## Technology stack

- Python
- pandas and NumPy
- scikit-learn (Random Forest regression)
- Folium
- Plotly
- Matplotlib
- ipywidgets
- openpyxl / xlrd

## Project structure

This project is currently designed to run as a single Google Colab cell. The notebook creates its output directory at runtime:

```text
/content/transitlens_outputs/
├── cleaned_stops.csv
├── cleaned_routes.csv
├── route_statistics.csv
├── ridership_data.csv
├── demand_forecast.csv
├── underserved_areas.csv
├── bus_stops.geojson
├── bus_stop_map.html
├── route_map.html
└── underserved_areas.html
```

## Future improvements

- Use GTFS `shapes.txt` for route geometry.
- Add real road-network routing and traffic-aware journey times.
- Calculate stop accessibility using population and walking-distance data.
- Incorporate real passenger counts, ticketing, or boarding/alighting data.
- Add route-frequency and service-span analysis.
- Support custom column mapping for non-standard Excel workbooks.

## License

No license is specified yet. Add a `LICENSE` file before distributing the project if you want to define reuse and contribution terms.
