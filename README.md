# ERA5 forcing data for the Weather Research and Forecasting (WRF) model

## Summary

This dataset provides ERA5 reanalysis product forcing parameters suitable for immediate ingestion into the Weather Research and Forecasting (WRF) model, i.e., in WRF intermediate data format. The dataset also includes global timeseries data from 2000-01 to present for key meteorological variables for the wind industry. Data is updated monthly.

## About

The ERA5 reanalyis product (https://www.ecmwf.int/en/forecasts/dataset/ecmwf-reanalysis-v5) is the leading global reanalysis product available today. For the wind energy industry in particular, ERA5 consistenly outperforms other products when compared against observations. Furthermore, ERA5 is most commonly used to provide boundary forcings to mesoscale modeling, notably for the open-source, community-supported, and industry-trusted Weather and Research Forecasting (WRF) model. Processing the ERA5 data into the format required by WRF can be onerous. With this open dataset, we aim to greatly simplify the process and allow users to more quickly and efficiently launch WRF simulations using ERA5 reanalysis as forcing

## Data

### Overview

Two scenario's are considered in the WINS50 project:

- **2020**: the 2020 scenario includes all wind turbines operational within the simulated domain in the year 2020. This scenario serves as a baseline/reference case and can be used for validation and verification against real world observations of meteorological and wind turbine measurements.
- **2050**: the 2050 scenario includes all wind turbines operational in the 2020 scenario, plus hypothetical windfarms consistent withh future energy ambitions. The rationale for this scenario can be found at the [project website](https://www.wins50.nl/data).

The figure below shows the spatial extent of the simulation domain, which measures 328 x 492 km with a grid spacing of 128 m. The example wind field around hub heigh shows extensive wakes downstream the various wind farms. The turbines of the the 2020 scenario are represented by the black dots, the turbines of the 2050 scenario by the magenta dots.

![](images/wins50_map_domain_with_windfarms.png)

The data is organized into four types of output for the 2020 and 2050 scenarios:

- **Turbines**: simulated time series of wind turbine-related variables (e.g. power, wind speed, air density, etc.) on the individual wind turbine level at a 5 min. time resolution. 
- **Virtual meteorological (met.) masts**: simulated meteorological quantities like wind speed, wind direction and turbulence intensity at 10 min. time resolution for around 60 locations corresponding to real world measurement locations (automatic weather stations, etc.) within the simulated domain.
- **Profiles**: profiles of simulated meteorological quantities (including velocity and temperature fluxes) for all levels in the model up to 6 km height for a grid of 600 locations over the simulated domain on 10 min. time resolution. 
- **Fields**: Two-dimensional cross sections of the entire simulated domain of several meteorological quantities at 9 heights on hourly time resolution. 

Further more, rendered images of the 100m wind field and liquid and ice cloud field are available as separate images and concatenated animations.

For examples of working with the data and finding your location/windfarm of interest see [Tutorials](#tutorials) below.

### Format and structure

The numerical data is provided in [Zarr](https://zarr.dev) format with one file (or "store") per output type and per scenario:

```
s3://whiffle-wins50-data/data
	whiffle_wins50_2020_scenario_fields.zarr
	whiffle_wins50_2020_scenario_profiles.zarr
	whiffle_wins50_2020_scenario_turbines.zarr
	whiffle_wins50_2020_scenario_virtual_metmasts.zarr
	whiffle_wins50_2050_scenario_fields.zarr
	whiffle_wins50_2050_scenario_profiles.zarr
	whiffle_wins50_2050_scenario_turbines.zarr
	whiffle_wins50_2050_scenario_virtual_metmasts.zarr
```

The PNG image data is stored in Tar files per day, per rendered field type and per GPU sub-domain following the pattern:

```
s3://whiffle-wins50-data/data
	images/{2020,2050}_scenario/2020/{01..12}/{01..31}/00/graspOutView{,-2}.001-00[01]-00[0123].tar
```

where `graspOutView` contains the rendered cloud field and `graspOutView-2` the rendered wind speed field.

Since the image data is rather large and not so straightforward to interact with, animated PNG's are stored in MP4 format following the pattern: 

```
s3://whiffle-wins50-data/data
	movies/whiffle_wins50_{2020,2050}_scenario_{clouds,wind,clouds_wind_mosaic}_2020{01..12}{01..31}.mp4
```

where `clouds` contains the rendered cloud field, `wind` the rendered wind field and `clouds_wind_mosaic` both rendered fields side-by-side.

An example animation can be found [online](https://vimeo.com/883270904/ee4a2f6e56?share=copy).

## Data Access

### Python

We recommended to use the [Xarray](https://docs.xarray.dev/en/stable/) Python package, which works very well together with the Zarr format, for an easy and efficient way to interact with the ~40 TB of WINS50 data without having to copy large data files to your local computer. Xarray provides a "lazy" way of loading the data, where only the metadata is copied when opening a file and the actual data will only be loaded when required. 

First setup your Python environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
```

To open, e.g., the Turbines data for the 2050 scenario with Xarray in Python:

```python
import xarray as xr

ds = xr.open_dataset(
    "s3://whiffle-wins50-data/data/whiffle_wins50_2050_scenario_turbines.zarr",
    engine="zarr",
)
```

For more examples see the Tutorials below.

### AWS CLI

The [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) can also be used to interact with the data, e.g.:

```
aws s3 ls --no-sign-request whiffle-wins50-data/data/
	PRE images/
	PRE movies/
	PRE whiffle_wins50_2020_scenario_fields.zarr/
	PRE whiffle_wins50_2020_scenario_profiles.zarr/
	PRE whiffle_wins50_2020_scenario_turbines.zarr/
	PRE whiffle_wins50_2020_scenario_virtual_metmasts.zarr/
	PRE whiffle_wins50_2050_scenario_fields.zarr/
	PRE whiffle_wins50_2050_scenario_profiles.zarr/
	PRE whiffle_wins50_2050_scenario_turbines.zarr/
	PRE whiffle_wins50_2050_scenario_virtual_metmasts.zarr/
```

## Tutorials

- [Plot wind farm locations and wind turbine time series for a particular wind farm](tutorials/turbines.py)
- [Plot met. mast locations and plot time series of particular met. mast](tutorials/masts.py)
- [Plot profile locations and plot daily mean profile for particular location](tutorials/profiles.py)
- [Plot (wind) fields](tutorials/fields.py)

## License

CC BY-SA 4.0

