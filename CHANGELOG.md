
# Changelog

## v0.48.1

### Features

- Generalize `met.shift_longitude()` to translate longitude coordinates onto any domain bounds.
- Add `VectorDataset.to_dict()` methods to output Vector data as dictionary. This method enables `Flight.to_dict()` objects to be serialized for input to the [Contrails API](https://api.contrails.org).
- Add `VectorDataset.from_dict()` class method to create `VectorDataset` class from dictionary.
- Support more time formats, including timezone aware times, in `VectorDataset` creation. All timezone aware `"time"`` coordinates are converted to UTC and stripped of timezone identifier.

### Fixes

- Fix issue in the `wake_vortex.max_downward_displacement` function in which float32 dtypes were promoted to float64 dtypes in certain cases.
- Ignore empty vectors in `VectorDataset.sum`.

### Internals

- Set `frozen=True` on the `MetVariable` dataclass.
- Test against python 3.12 in the GitHub Actions CI. Use python 3.12 the docs and doctest workflows.

## v0.48.0

This release includes a number of breaking changes and new features. If upgrading from a previous version of `pycontrails`, please read the changelog carefully. Open an [issue](https://github.com/contrailcirrus/pycontrails/issues) if you experience problems.

### Breaking changes

- When running `Cocip` and other `pycontrails` models, the `met` and `rad` parameter must now contain predefined metadata attributes `provider`, `dataset`, and `product` describing the met source. An error will now be raised in `Cocip` if these attributes are not present.
- Deprecate passing arbitrary `kwargs` into the `MetDataArray` constructor.
- No longer convert accumulated radiation data to average instantaneous data in `ERA5` and `HRES` interfaces. This logic is now handled downstream by the model (e.g., `Cocip`). This change allows for more flexibility in the `rad` data passed into the model and avoids unnecessary computation in the `MetDataSource` interfaces.
- Add new `MetDataSource.set_met_source_metadata` abstract method. This should be called within the implementing class `open_metdataset` method.
- No longer take a finite difference in the time dimension for HRES radiation data. This is now also handled natively in `Cocip`.
- No longer convert relative humidity from a percentage to a fraction in `ERA5` and `HRES` interfaces.
- Require the `HRES` `stream` parameter to be one of `["oper", "enfo"]`. Require the `field_type` parameter to be one of `["fc", "pf", "cf", "an"]`.
- Remove the `steps` and `step_offset` properties in the `GFSForecast` interface. Now the `timesteps` attribute is the only source of truth for determining AWS S3 keys. Change the `filename` method to take in a `datatime` timestep instead of an `int` step. No longer assign the first step radiation data to the zeroth step.
- Change the return type of `ISSR.eval`, `SAC.eval`, and `PCR.eval` from `MetDataArray` to `MetDataset`. This is more consistent with the return type of other `pycontrails` models and more closely mirrors the behavior of vector models. Set output `attrs` metadata on the global `MetDataset` instead of the individual `MetDataArray` in each case.

### Features

- Rewrite parts of the `pycontrails.core.datalib` module for higher performance and readability.
- Add optional `attrs` and `attrs_kwargs` parameters to `MetDataset` constructor. This allows the user to customize the attributes on the underlying `xarray.Dataset` object. This update makes `MetDataset` more consistent with `VectorDataset`.
- Add three new properties `provider_attr`, `dataset_attr`, and `product_attr` to `MetDataset`. These properties give metadata describing the underlying meterological data source.
- Add new `Model.transfer_met_source_attrs` method for more consistent handling of met source metadata on the `source` parameter passed into `Model.eval`.
- No longer require `geopotential` data when computing `tau_cirrus`. If neither `geopotential` nor `geopotential_height` are available, geopotential is approximated from the geometric height. No longer require geopotential on the `met` parameter in `Cocip` or `CocipGrid`.
- Remove the `Cocip` `shift_radiation_time` parameter. This is now inferred directly from the `rad` metadata. An error is raised if the necessary metadata is not present.
- Allow `Cocip` to run with both instantaneous (`W m-2`) and accumulated (`J m-2`) radiation data.
- Allow `Cocip` to run with accumulated ECMWF HRES radiation data.

### Fixes

- Correct radiation unit in the `ACCF` wrapper model [#64]. Both instantaneous (`W m-2`) and accumulated (`J m-2`) radiation data are now supported, and the `ACCF` wrapper will handle each appropriately.
- Avoid unnecessary writing and reading of temporary files in `ERA5.cache_dataset` and `HRES.cache_dataset`.
- Fix timestep resolution bug in `GFSForecast`. When the `grid` parameter is 0.5 or 1.0, forecasts are only available every 3 hours. Previously, the `timesteps` property would define an hourly timestep.

### Internals

- Include `name` parameter in `MetDataArray` constructor.
- Make the `coordinates.slice_domain` function slightly more performant by explicitly dropping nan values from the `request` parameter.
- Round unwieldy floating point numbers in `GeoVectorDataset._display_attrs`.
- Remove the `ecmwflibs` package from the `ecmwf` optional dependencies.
- Add NPY to `ruff` rules.
- Add convenience `MetDataset.standardize_variables` method.
- Remove the `p_settings` attribute on the `ACCF` interface. This is now constructed internally within `ACCF.eval`. Replace the `ACCF._update_accf_config` method with a `_get_accf_config` function.

## v0.47.3

### Fixes

- Strengthen `correct_fuel_flow` in the `PSmodel` to account for descent conditions.
- Clip the denominator computed in `pycontrails.physics.jet.equivalent_fuel_flow_rate_at_cruise`.
- Ensure the token used within GitHub workflows has the fewest privileges required. Set top-level permissions to `none` in each workflow file. Remove unnecessary permissions previously used in the `google-github-actions/auth` action.
- Fix bug in `radiative_forcing.effective_tau_contrail` identified in [#99](https://github.com/contrailcirrus/pycontrails/issues/99).
- Fix the unit for `vertical_velocity` in `geo.advect_level`.
- Fix bug appearing in `Flight._geodesic_interpolation` in which a single initial large gap was not interpolated with a geodesic path.

### Internals

- Add `FlightPhase` to the `pycontrails` namespace.

## v0.47.2

### Features

- New experimental `GOES` interface for downloading and visualizing GOES-16 satellite imagery.
- Add new [GOES example notebook](https://py.contrails.org/examples/GOES.html) highlighting the interface.
- Build python 3.12 wheels for Linux, macOS, and Windows on release. This is in addition to the existing python 3.9, 3.10, and 3.11 wheels.

### Fixes

- Use the experimental version number parameter `E` in `pycontrails.ecmwf.hres.get_forecast_filename`. Update the logic involved in setting the dissemination data stream indicator `S`.
- Change the behavior of `_altitude_interpolation` method that is called within  `resample_and_fill`. Step climbs are now placed in the middle of long flight segments. Descents continue to occur at the end of segments.

### Internals

- Provide consistent `ModuleNotFoundError` messages when optional dependencies are not installed.
- Move the `synthetic_flight` module into the `pycontrails.ext` namespace.

## v0.47.1

### Fixes

- Fix bug in `PSGrid` in which the `met` data was assumed to be already loaded into memory. This caused errors when running `PSGrid` with a `MetDataset` source.
- Fix bug (#86) in which `Cocip.eval` loses the `source` fuel type. Instead of instantiating a new `Flight` or `Fleet` instance with the default fuel type, the `Cocip._bundle_results` method now overwrites the `self.source.data` attribute with the bundled predictions.
- Avoid a memory explosion when running `Cocip` on a large non-dask-backed `met` parameter. Previously the `tau_cirrus` computation would be performed in memory over the entire `met` dataset.
- Replace `datetime.utcfromtimestamp` (deprecated in python 3.12) with `datetime.fromtimestamp`.
- Explicitly support python 3.12 in the `pyproject.toml` build system.

### Internals

- Add `compute_tau_cirrus_in_model_init` parameter to `CocipParams`. This controls whether to compute the cirrus optical depth in `Cocip.__init__` or `Cocip.eval`. When set to `"auto"` (the default), the `tau_cirrus` is computed in `Cocip.__init__` if and only if the `met` parameter is dask-backed.
- Change data requirements for the `EmpiricalGrid` aircraft performance model.
- Consolidate `ERA5.cache_dataset` and `HRES.cache_dataset` onto common `ECMWFAPI.cache_dataset` method. Previously the child implementations were identical.
- No longer require the `pyproj` package as a dependency. This is now an optional dependency, and can be installed with `pip install pycontrails[pyproj]`.

## v0.47.0

Implement a Poll-Schumann (`PSGrid`) theoretical aircraft performance over a grid.

### Breaking changes

- Move the `pycontrails.models.aircraft_performance` module to `pycontrails.core.aircraft_performance`.
- Rename `PSModel` -> `PSFlight`.

### Fixes

- Use the instance `fuel` attribute in the `Fleet.to_flight_list` method. Previously, the default `JetA` fuel was always used.
- Ensure the `Fleet.fuel` attribute is inferred from the underlying sequence of flights in the `Fleet.from_seq` method.

### Features

- Implement the `PSGrid` model. For a given aircraft type and position, this model computes optimal aircraft performance at cruise conditions. In particular, this model can be used to estimate fuel flow, engine efficiency, and aircraft mass at cruise. In particular, the `PSGrid` model can now be used in conjunction with `CocipGrid` to simulate contrail formation over a grid.
- Refactor the `Emissions` model so that `Emissions.eval` runs with `source: GeoVectorDataset`. Previously, the `eval` method required a `Flight` instance for the `source` parameter. This change allows the `Emissions` model to run more seamlessly as a sub-model of a gridded model (ie, `CocipGrid`),
- No longer require `pycontrails-bada` to import or run the `CocipGrid` model. Instead, the `CocipGridParams.aircraft_performance` parameter can be set to any `AircraftPerformanceGrid` instance. This allows the `CocipGrid` model to run with any aircraft performance model that implements the `AircraftPerformanceGrid` interface.
- Add experimental `EmpiricalAircraftPerformanceGrid` model.
- Add convenience `GeoVectorDataset.T_isa` method to compute the ISA temperature at each point.

### Internals

- Add optional `climb_descend_at_end` parameter to the `Flight.resample_and_fill` method. If True, the climb or descent will be placed at the end of each segment rather than the start.
- Define `AircraftPerformanceGridParams`, `AircraftPerformanceGrid`, and `AircraftPerformanceGridData` abstract interfaces for gridded aircraft performance models.
- Add `set_attr` parameter to `Models.get_source_param`.
- Better handle `source`, `source.attrs`, and `params` customizations in `CocipGrid`.
- Include additional classes and functions in the `pycontrails.models.emissions` module.
- Hardcode the paths to the static data files used in the `Emissions` and `PSFlight` models. Previously these were configurable by model parameters.
- Add `altitude_ft` parameter to the `GeoVectorDataset` constructor. Warn if at least two of `altitude_ft`, `altitude`, and `level` are provided.
- Allow instantiation of `Model` instances with `params: ModelParams`. Previously, the `params` parameter was required to be a `dict`. The current implementation checks that the `params` parameter is either a `dict` or has type `default_params` on the `Model` class.

## v0.46.0

Support "dry advection" simulation.

### Features

- Add new `DryAdvection` model to simulate sediment-free advection of an aircraft's exhaust plume. This model is experimental and may change in future releases. By default, the current implementation simulates plume geometry as a cylinder with an elliptical cross section (the same geometry assumed in CoCiP). Wind shear perturbs the ellipse azimuth, width, and depth over the plume evolution. The `DryAdvection` model may also be used to simulate advection without wind-shear effects by setting the model parameters `azimuth`, `width`, and `depth` to None.
- Add new [Dry Advection example notebook](https://py.contrails.org/examples/advection.html) highlighting the new `DryAdvection` model and comparing it to the `Cocip` model.
- Add optional `fill_value` parameter to `VectorDataset.sum`.

### Fixes

- (#80) Fix unit error in `wake_vortex.turbulent_kinetic_energy_dissipation_rate`. This fix affects the estimate of wake vortex max downward displacement and slightly changes `Cocip` predictions.
- Change the implementation of `Flight.resample_and_fill` so that lat-lon interpolation is linear in time. Previously, the timestamp associated to a waypoint was floored according to the resampling frequency without updating the position accordingly. This caused errors in segment calculations (e.g., true airspeed).

### Internals

- Add optional `keep_original_index` parameter to the `Flight.resample_and_fill` method. If `True`, the time original index is preserved in the output in addition to the new time index obtained by resampling.
- Improve docstrings in `wake_vortex` module
- Rename `wake_vortex` functions to remove `get_` prefix at the start of each function name.
- Add pytest command line parameter `--regenerate-results` to regenerate static test fixture results. Automate in make recipe `make pytest-regenerate-results`.
- Update handling of `GeoVectorDataset.required_keys` the `GeoVectorDataset` constructor. Add class variable `GeoVectorDataset.vertical_keys` for handing the vertical dimension.
- Rename `CocipParam.max_contrail_depth` -> `CocipParam.max_depth`.
- Add `units.dt_to_seconds` function to convert `np.timedelta64` to `float` seconds.
- Rename `thermo.p_dz` -> `thermo.pressure_dz`.

## v0.45.0

Add experimental support for simulating radiative effects due to contrail-contrail overlap.

### Features

- Support simulating contrail contrail overlap when running the `Cocip` model with a `Fleet` source. The `contrail_contrail_overlapping` and `dz_overlap_m` parameters govern the overlap calculation. This mode of calculation is still experimental and may change in future releases.
- Rewrite the `pycontrails.models.cocip.output` modules into a single `pycontrails.cocip.output_formats` module. The new module supports flight waypoint summary statistics, contrail waypoints summary statistics, gridded outputs, and time-slice outputs.
- Add new `GeoVectorDataset.to_lon_lat_grid` method. This method can be thought of as a partial inverse to the `MetDataset.to_vector` method. The current implementation is brittle and may be revised in a future release.

### Fixes

- Extend `Models.set_source_met` to allow `interpolation_q_method="cubic-spline"` when working with `MetDataset` source (ie, so-called gridded models). Previously a `NotImplementedError` was raised.
- Ensure the `Flight.copy` implementation works with `Fleet` instances as well.
- Avoid looping over `keys` twice in `VectorDataset.broadcast_attrs`. This is a slight performance enhancement.
- Fix `Fleet` signature for compatibility with `Flight`.
- Fix a few hard-coded assumptions in broadcasting aircraft performance and emissions when running `Cocip` with a `Fleet` source. The previous implementation did not consider the possibility of aircraft performance variables on `Flight.data` and `Flight.attrs` separately.

### Internals

- Add optional `raise_error` parameter to the `VectorDataset.broadcast_attrs` method.
- Update `Fleet` internals.

## v0.44.2

### Fixes

- Narrow type hints on the ABC `AircraftPerformance` model. The `AircraftPerformance.eval` method requires a `Flight` object for the `source` parameter.
- In `PSFlight.eval`, explicitly set any aircraft performance data at waypoints with zero true airspeed to `np.nan`. This avoids numpy `RuntimeWarning`s without affecting the results.
- Fix corner-case in the `polygon.buffer_and_clean` function in which the polygon created by buffering the `opencv` contour is not valid. Now a second attempt to buffer the polygon is made with a smaller buffer distance.
- Ignore RuntimeError raised in `scipy.optimize.newton` if the maximum number of iterations is reached before convergence. This is a workaround for occasional false positive convergence warnings. The pycontrails use-case may be related to [this GitHub issue](https://github.com/scipy/scipy/issues/8904).
- Update the `Models.__init__` warnings when `humidity_scaling` is not provided. The previous warning provided an outdated code example.
- Ensure the `interpolation_q_method` used in a parent model is passed into the `humidity_scaling` child model in the `Models.__init__` method. If the two `interpolation_q_method` values are different, a warning is issued. This could be extended to other model parameters in the future.

### Features

- Enable `ExponentialBoostLatitudeCorrectionHumidityScaling` humidity scaling for the model parameter `interpolation_q_method="cubic_spline"`.
- Add [GFS notebook](https://py.contrails.org/examples/GFS.html) example.

### Breaking changes

- Remove `ExponentialBoostLatitudeCorrectionHumidityScalingParams`. These parameters are now hard-coded in the `ExponentialBoostLatitudeCorrectionHumidityScaling` model.

## v0.44.1

### Breaking changes

- By default, call `xr.open_mfdataset` with `lock=False` in the `MetDataSource.open_dataset` method. This helps alleviate a `dask` threading issue similar to [this GitHub issue](https://github.com/pydata/xarray/issues/4406).

### Fixes

- Support `MetDataset` source in the `HistogramMatching` humidity scaling model. Previously only `GeoVectorDataset` sources were explicitly supported.
- Replace `np.gradient` with `dask.array.gradient` in the `tau_cirrus` module. This ensures that the computation is done lazily for dask-backed arrays.
- Round to 6 digits in the `polygon.determine_buffer` function. This avoid unnecessary warnings for rounding errors.
- Fix type hint for `opencv-python` 4.8.0.74.

### Internals

- Take more care with `float` and `int` types in the `contrail_properties` module. Prefer `np.clip` to `np.where` or `np.maximum` for clipping values.
- Support `air_temperature` in `CocipGrid` verbose formation outputs.
- Remove `pytest-timeout` dev dependency.

## v0.44.0

Support for the [Poll-Schumann aircraft performance model](https://doi.org/10.1017/aer.2020.62).

### Features

- Implement a basic working version of the Poll-Schumann (PS) aircraft performance model. This is experimental and may undergo revision in future releases. The PS Model currently supports the following 53 aircraft types:
  - A30B
  - A306
  - A310
  - A313
  - A318
  - A319
  - A320
  - A321
  - A332
  - A333
  - A342
  - A343
  - A345
  - A346
  - A359
  - A388
  - B712
  - B732
  - B733
  - B734
  - B735
  - B736
  - B737
  - B738
  - B739
  - B742
  - B743
  - B744
  - B748
  - B752
  - B753
  - B762
  - B763
  - B764
  - B77L
  - B772
  - B77W
  - B773
  - B788
  - B789
  - E135
  - E145
  - E170
  - E195
  - MD82
  - MD83
  - GLF5
  - CRJ9
  - DC93
  - RJ1H
  - B722
  - A20N
  - A21N

  The "gridded" version of this model is not yet implemented. This will be added in a future release.
- Improve the runtime of instantiating the `Emissions` model by a factor of 10-15x. This translates to a time savings of several hundred milliseconds on modern hardware. This improvement is achieved by more efficient parsing of the underlying static data and by deferring the construction of the interpolation artifacts until they are needed.
- Automatically use a default engine type from the aircraft type in the `Emissions` model if an `engine_uid` parameter is not included on the `source`. This itself is configurable via the `use_default_engine_uid` parameter on the `Emissions` model. The default mappings from aircraft types to engines is included in `pycontrails/models/emissions/static/default-engine-uids.csv`.

### Breaking changes

- Remove the `Aircraft` dataclass from `Flight` instantiation. Any code previously using this should instead directly pass additional `attrs` to the `Flight` constructor.
- The `load_factor` is now required in `AircraftPerformance` models. The global `DEFAULT_LOAD_FACTOR` constant in `pycontrails.models.aircraft_performance` provides a reasonable default. This is currently set to 0.7.
- Use a `takeoff_mass` parameter in `AircraftPerformance` models if provided in the `source.attrs` data.
- No longer use a reference mass `ref_mass` in `AircraftPerformance` models. This is replaced by the `takeoff_mass` parameter if provided, or calculated from operating empty operating mass, max payload mass, total fuel consumption mass, reserve fuel mass, and the load factor.

### Fixes

- Remove the `fuel` parameter from the `Emissions` model. This is inferred directly from the `source` parameter in `Emissions.eval`.
- Fix edge cases in the `jet.reserve_fuel_requirements` implementation. The previous version would return `nan` for some combinations of `fuel_flow` and `segment_phase` variables.
- Fix a spelling mistake: `units.kelvin_to_celcius` -> `units.kelvin_to_celsius`.

### Internals

- Use `ruff` in place of `pydocstyle` for linting docstrings.
- Use `ruff` in place of `isort` for sorting imports.
- Update the `AircraftPerformance` template based on the patterns used in the new `PSFlight` class. This may change again in the future.

## v0.43.0

Support experimental interpolation against gridded specific humidity. Add new data-driven humidity scaling models.

### Features

- Add new experimental `interpolation_q_method` field to the `ModelParams` data class. This parameter controls the interpolation methodology when interpolation against gridded specific humidity. The possible values are:
  - `None`: Interpolate linearly against specific humidity. This is the default behavior and is the same as the previous behavior.
  - `"cubic-spline"`: Apply cubic-spline scaling to the interpolation table vertical coordinate before interpolating linearly against specific humidity.
  - `"log-q-log-p"`: Interpolate in the log-log domain against specific humidity and pressure.

  This interpolation parameter is used when calling `pycontrails.core.models.interpolate_met`. It can also be used directly with the new lower-level `pycontrails.core.models.interpolate_gridded_specific_humidity` function.
- Add new experimental `HistogramMatching` humidity scaling model to match RHi values against IAGOS observations. The previous `HistogramMatchingWithEckel` scaling is still available when working with ERA5 ensemble members.
- Add new [tutorial](https://py.contrails.org/tutorials/interpolating-specific-humidity.html) discussing the new specific humidity interpolation methodology.

### Breaking changes

- Add an optional `q_method` parameter to the `pycontrails.core.models.interpolate_met` function. The default value `None` agrees with the previous behavior.
- Change function signatures in the `cocip.py` module for consistency. The `interp_kwargs` parameter is now unpacked in the `calc_timestep_meterology` signature. Rename `kwargs` to `interp_kwargs` where appropriate.
- Remove the `cache_size` parameter in `MetDataset.from_zarr`. Previously this parameter allowed the user to wrap the Zarr store in a `LRUCacheStore` to improve performance. Changes to Zarr internals have broken this approach. Custom Zarr patterns should now be handled outside of `pycontrails`.

### Fixes

- Recompute and extend quantiles for histogram matching humidity scaling. Quantiles are now available for each combination of `q_method` and the following ERA5 data products: reanalysis and ensemble members 0-9. This data is available as a parquet file and is packaged with `pycontrails`.
- Fix the precomputed Eckel coefficients. Previous values where computed for different interpolation assumptions and were not correct for the default interpolation method.
- Clip the scaled humidity values computed by the `humidity_scaling.eckel_scaling` function to ensure that they are non-negative. Previously, both relative and specific humidity values arising from Eckel scaling could be negative.
- Handle edge case of all NaN values in the `T_critical_sac` function in the `sac.py` module.
- Avoid extraneous copy when calling `VectorDataset.sum`.
- Officially support `numpy` v1.25.0.
- Set a `pytest-timeout` limit for tests in `tests/unit/test_ecmwf.py` to avoid hanging tests.
- Add `forecast_step` parameter to the `ACCF` model.

### Internals

- Refactor auxillary functions used by `HistogramMatchingWithEckel` to better isolated histogram matching from Eckel scaling.
- Refactor `intersect_met` method in `pycontrails.core.models` to handle experimental `q_method` parameter.
- Include a `q_method` field in `Model.interp_kwargs`.
- Include precomputed humidity lapse rate values in the new `pycontrails.core.models._load_spline` function.
- Move the `humidity_scaling.py` module into its own subdirectory within `pycontrails/models`.

## v0.42.2

Re-release of [v0.42.1](#v0421).

## v0.42.1

### Features

- Add new `HistogramMatchingWithEckel` experimental humidity scaling model. This is still a work in progress.
- Add new `Flight.fit_altitude` method which uses piecewise linear fitting to smooth a flight profile.
- Add new `pycontrails.core.flightplan` module for parsing ATC flight plans between string and dictionary representations.
- Add new [airports](docs/examples/airports.ipynb) and [flightplan](docs/examples/flightplan.ipynb) examples.

### Breaking changes

- No longer attach empty fields "sdr", "rsr", "olr", "rf_sw", "rf_lw", "rf_net" onto the `source` parameter in `Cocip.eval` when the flight doesn't generate any persistent contrails.
- Remove params `humidity_scaling`, `rhi_adj_uncertainty`, and `rhi_boost_exponent_uncertainty` from `CocipUncertaintyParams`.
- Change the default value for `parallel` from True to False in `xr.open_mfdataset`. This can be overridden by setting the `xr_kwargs` parameter in `ERA5.open_metdataset`.

### Fixes

- Fix a unit test (`test_dtypes.py::test_issr_sac_grid_output`) that occasionally hangs. There may be another test in `test_ecmwf.py` that suffers from the same issue.
- Fix issue encountered in `Cocip.eval` when concatenating contrails with inconsistent values for `_out_of_bounds`. This is only relevant when running the model with the experimental parameter `interpolation_use_indices=True`.
- Add a `Fleet.max_distance_gap` property. The previous property on the `Flight` class was not applicable to `Fleet` instances.
- Fix warning in `Flight` class to correctly suggest adding kwarg `drop_duplicated_times`.
- Fix an issue in the `VectorDataset` constructor with a `data` parameter of type `pd.DataFrame`. Previously, time data was rewritten to the underlying DataFrame. This could cause copy-on-write issues if the DataFrame was a view of another DataFrame. This is now avoided.

### Internals

- When possible, replace type hints `np.ndarray` -> `np.typing.NDArray[np.float_]` in the `cocip`, `cocip_params`, `cocip_uncertainty`, `radiative_forcing`, and `wake_vortex` modules.
- Slight performance enhancements in the `radiative_forcing` module.
- Change the default value of `u_wind` and `v_wind` from None to 0 in `Flight.segment_true_airspeed`. This makes more sense semantically.

## v0.42.0

Phase 1 of the Spire datalib, which contains functions to identify unique flight trajectories from the raw Spire ADS-B data.

### Features

- Add a `pycontrails.core.airport` module to read and process the global airport database, which can be used to identify the nearest airport to a given coordinate.
- Add a `pycontrails.datalib.spire.clean` function to remove and address erroneous waypoints in the raw Spire ADS-B data.
- Add a `pycontrails.datalib.spire.filter_altitude` function to remove noise in cruise altitude.
- Add a `pycontrails.datalib.spire.identify_flights` function to identify unique flight trajectories from ADS-B messages.
- Add a `pycontrails.datalib.spire.validate_trajectory` function to check the validity of the identified trajectories from ADS-B messages.
- Add a `FlightPhase` integer `Enum` in the `flight` module. This includes a new `level_flight` flight phase.

### Internals

- Add unit tests providing examples to identify unique flights.
- Rename `flight._dt_waypoints` -> `flight.segment_duration`.
- Move `jet.rate_of_climb_descent` -> `flight.segment_rocd`.
- Move `jet.identify_phase_of_flight` -> `flight.segment_phase`.
- Update `FlightPhase` to be a dictionary enumeration of flight phases.
- Add references to [`traffic` library](https://traffic-viz.github.io/).

## v0.41.0

Improve polygon algorithms.

### Features

- Rewrite the `polygon` module to run computation with [opencv](https://docs.opencv.org/4.x/index.html) in place of [scikit-image](https://scikit-image.org/) for finding contours. This change improves the algorithm runtime and fixes some previous unstable behavior in finding nested contours. For an introduction to the methodology, see the [OpenCV contour tutorial](https://docs.opencv.org/master/d4/d73/tutorial_py_contours_begin.html).

### Breaking changes

- Completely rewrite the `polygon` module. Replace the "main" public function `polygon.find_contours_to_depth` with `polygon.find_multipolygon`. Replace the `polygon.contour_to_lat_lon` function with `polygon.multipolygon_to_geojson`. Return `shapely` objects when convenient to do so.
- Convert continuous data to binary for polygon computation.
- Remove parameters `min_area_to_iterate` and `depth` in the `MetDataArray.to_polygon_feature` method. The `depth` parameter has been replaced by the boolean `interiors` parameter. Add a `properties` parameter for adding properties to the `Polygon` and `MultiPolygon` features. The `max_area` and `epsilon` parameters are now expressed in terms of latitude-longitude degrees.

### Internals

- Add `opencv-python-headless>=4.5` as an optional "vis" dependency. Some flavor of `opencv` is required for the updated polygon algorithms.

## v0.40.1

### Fixes

- Use [oldest-supported-numpy](https://pypi.org/project/oldest-supported-numpy/) for building pycontrails wheels. This allows pycontrails to be compatible with environments that use old versions of numpy. The [pycontrails v0.40.0 wheels](https://pypi.org/project/pycontrails/0.40.0/#files) are not compatible with numpy 1.22.

## v0.40.0

Support scipy 1.10, improve interpolation performance, and fix many windows issues.

### Features

- Improve interpolation performance by cythonizing linear interpolation. This extends the approach taken in [scipy 1.10](https://github.com/scipy/scipy/pull/17291). The pycontrails [cython routines](pycontrails/core/rgi_cython.pyx) allow for both float64 and float32 grids via cython fused types (the current scipy implementation assumes float64). In addition, interpolation up to dimension 4 is supported (the scipy implementation supports dimension 1 and 2).
- Officially support [scipy 1.10](https://scipy.github.io/devdocs/release/1.10.0-notes.html).
- Officially test on windows in the GitHub Actions CI.
- Build custom wheels for python 3.9, 3.10, and 3.11 for the following platforms:
  - Linux (x86_64)
  - macOS (arm64 and x86_64)
  - Windows (x86_64)

### Breaking changes

- Change `MetDataset` and `MetDataArray` conventions: underlying dimension coordinates are automatically promoted to float64.
- Change how datetime arrays are converted to floating values for interpolation. The new approach introduces small differences compared with the previous implementation. These differences are significant enough to see relative differences in CoCiP predictions on the order of 1e-4.

### Fixes

- Unit tests no longer raise errors when the `pycontrails-bada` package is not installed. Instead, some tests are skipped.
- Fix many numpy casting issues encountered on windows.
- Fix temp file issues encountered on windows.
- Officially support changes in `xarray` 2023.04 and `pandas` 2.0.

### Internals

- Make the `interpolation` module more aligned with [scipy 1.10 enhancements](https://docs.scipy.org/doc/scipy/release.1.10.0.html#scipy-interpolate-improvements) to the `RegularGridInterpolator`. In particular, grid coordinates now must be float64.
- Use [cibuildwheel](https://cibuildwheel.readthedocs.io/en/stable/) to build wheels for Linux, macOS (arm64 and x86_64), and Windows on [release](.github/workflows/release.yaml) in Github Actions. Allow this workflow to be triggered manually to test the release process without actually publishing to PyPI.
- Simplify interpolation with pre-computed indices (invoked with the model parameter `interpolation_use_indices`) via a `RGIArtifacts` interface.
- Overhaul much of the interpolation module to improve performance.
- Slight performance enhancements to the `met` module.

## v0.39.6

### Features

- Add `geo.azimuth` and `geo.segment_azimuth` functions to calculate the azimuth between coordinates. Azimuth is the angle between coordinates relative to true north on the interval `[0, 360)`.

### Fixes

- Fix edge case in polygon algorithm by utilizing the `fully_connected` parameter in `measure.find_contours`. This update leads to slight changes in interior contours in some cases.
- Fix hard-coded POSIX path in `conftest.py` for windows compatibility.
- Address `PermissionError` raised by `shutil.copy` when the destination file is open in another thread. The copy is skipped and a warning is logged.
- Fix some unit tests in `test_vector.py` and `test_ecmwf.py` for windows compatibility. There are still a few tests that fail on windows (unrelated to changes in this release) that will be fixed in v0.40.0.
- Allow `cachestore=None` to skip caching in `ERA5`, `HRES`, and `GFS` interfaces. Previously a default `DiskCacheStore` was created even when `cachestore=None`. By default, caching is still enabled.

### Internals

- Clean up `geo` module docstrings.
- Add Wikipedia reference for [Azimuth](https://en.wikipedia.org/wiki/Azimuth).
- Convert `MetBase._load` to from a class method to a function.

## v0.39.5

### Fixes

- Fix `docs/examples/CoCiP.ipynb` example demonstrating aircraft performance integration
- Fix unit test caused by breaking change in [pyproj 3.5.0](https://github.com/pyproj4/pyproj/releases/tag/3.5.0)

### Internals

- Add additional Zenodo metadata
- Execute notebook examples in [Docs Action](https://github.com/contrailcirrus/pycontrails/actions/workflows/docs.yaml)

## v0.39.4

### Internals

- Add additional Zenodo metadata
- Add [Doc / Notebook test Action](https://github.com/contrailcirrus/pycontrails/actions/workflows/doctest.yaml) to run notebook test (`make nb-test`) and doctests (`make doctest`) on pull requests.
- Update [Docs Action](https://github.com/contrailcirrus/pycontrails/actions/workflows/docs.yaml) to use python 3.11.

## v0.39.3

### Fixes

- Update links in the [README](README.md).
- Update the [release guide](RELEASE.md) to include a checklist of steps to follow when cutting a release.

## v0.39.2

### Fixes

- Fix links in documentation website.
- Deploy build to [PyPI](https://pypi.org/project/pycontrails/) (in addition to Test PyPI) on release.

## v0.39.1

### Features

- Use [setuptools_scm](https://github.com/pypa/setuptools_scm) to manage the `pycontrails` version.

## v0.39.0

### Features

- Add Apache-2 LICENSE and NOTICE files (#21)
- Add [CONTRIBUTING](https://github.com/contrailcirrus/pycontrails/blob/main/CONTRIBUTING.md) document
- Update documentation website [py.contrails.org](https://py.contrails.org).
  Includes `install` and `develop` guides, update citations
  and many other small improvements (#19).
- Initiate the [Github Discussion](https://github.com/contrailcirrus/pycontrails/discussions) forum (#22).

### Fixes

- Fix erroneous docstrings in `emissions.py` (#25)

### Internals

- Add Github action to push to `pypi` on tag (#3)
- Replace `flake8` with `ruff` linter
- Add `nb-clean` to pre-commit hooks for example notebooks
- Add `doc8` rst linter and pre-commit hook

## v0.38.0

### Breaking changes

- Change default value of `epsilon` parameter in method `MetDataArray.to_polygon_feature` from 0.15 to 0.0.
- Change the polygon simplification algorithm. The new implementation uses `shapely.buffer` and doesn't try to preserve the topology of the simplified polygon. This change may result in slightly different polygon geometries.

### Internals

- Add `depth` parameter to `MetDataArray.to_polygon_feature` to control the depth of the contour searching.
- Add experimental `convex_hull` parameter to `MetDataArray.to_polygon_feature` to control whether to take the convex hull of each contour.
- Warn if `iso_value` is not specified in `MetDataArray.to_polygon_feature`.

### Fixes

- Consolidate three redundant implementations of standardizing variables into a single `met.standardize_variables`.
- Ensure simplified polygons returned by `MetDataArray.to_polygon_feature` are disjoint. While non-disjoint polygons don't violate the GeoJSON spec, they can cause problems in some applications.

## v0.37.3

### Internals

- Add citations for ISA calculations
- Abstract functionality to convert a Dataset or DataArray to longitude coordinates `[-180, 180)` into `core.met.shift_longitude`. Add tests for method.
- Add auto-formatting checks to CI testing

## v0.37.2

ACCF integration updates

### Fixes

- Fixes ability to evaluate ACCF model over a `MetDataset` grid by passing in tuple of (dims, data) when assigning data
- Fixes minor issues with ACCF configuration and allows more configuration options
- Updates example ACCF notebook with example of how to set configuration options when evaluating ACCFs over a grid

## v0.37.1

### Features

- Include "rhi" and "iwc" variables in `CocipGrid` verbose outputs.

## v0.37.0

### Breaking changes

- Update CoCiP unit test static results for breaking changes in tau cirrus calculation. The relative difference in pinned energy forcing values is less than 0.001%.

### Fixes

- Fix geopotential height gradient calculation in the `tau_cirrus` module. When calculating finite differences along the vertical axis, the tau cirrus model previously divided the top and bottom differences by 2. To numerically approximate the derivative at the top and bottom levels, these differences should have actually been divided by 1. The calculation now uses `np.gradient` to calculate the derivative along the vertical axis, which handles this correctly.
- Make tau cirrus calculation slightly more performant.
- Include a warning in the suspect `IFS._calc_geopotential` implementation.

### Internals

- Remove `_deprecated.tau_cirrus_alt`. This function is now the default `tau_cirrus` calculation. The original `tau_cirrus` calculation is still available in the `_deprecated` module.
- Run `flake8` over test modules.
