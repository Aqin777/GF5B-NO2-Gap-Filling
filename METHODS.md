# Methods note

## Research scope

The project estimates near-surface NO₂ around the GF-5B overpass time and studies how to maintain useful spatial coverage when same-day satellite retrievals are missing. The public description separates methodological implementation from confirmed evaluation results.

## Target construction

Ground-monitor NO₂ at 10:00 and 11:00 local time is paired at the same location and averaged to approximate a 10:30 target. Satellite and meteorological fields are matched to the station-time records. The early workflow uses GF-5B tropospheric vertical column density and meteorological predictors; later experiments add static and spatial-support features.

## Prediction settings

Two feature regimes address different scientific questions:

- **Satellite-assisted estimation** includes GF-5B tropospheric column information when a usable retrieval exists.
- **Gap filling without same-day satellite NO₂** excludes the satellite column and relies on meteorology, calendar features, geographic coordinates, population, elevation, land cover, and related static or support features where available.

The two regimes should be compared on the same evaluation rows when estimating the incremental value of satellite retrievals.

## Spatial-density reweighting

Let the training domain be partitioned into fixed 2° × 2° latitude-longitude blocks. For training sample \(i\), let \(b(i)\) denote its block and \(n_{b(i)}\) the number of training samples in that block. The unnormalized weight is

\[
\tilde{w}_i = \frac{1}{n_{b(i)}}.
\]

The implementation rescales these inverse-frequency weights, constrains them to the range 0.25–4.0, and normalizes them to keep their mean near one. The resulting vector is supplied to LightGBM as `sample_weight`.

The intended effect is to reduce the over-dominance of dense eastern urban monitoring clusters and increase the reasonable influence of sparse regions in a national mapping objective. Appropriate names for this method are:

- spatial-density reweighting;
- inverse-frequency weighting across spatial blocks; and
- spatial-structure-informed sample weighting.

It should not be called a spatial-autocorrelation model. The method does not directly calculate Moran's I, a semivariogram, a spatial covariance model, or a data-driven correlation length.

## Baselines and implementation status

The archived early benchmark is unweighted. Its best random-cross-validation result is LightGBM R² = 0.7765. The v3–v8 experimental LightGBM scripts are the later experiments that supply spatial weights through `sample_weight`.

The current two-year and daily production-baseline code performs a random 80:20 split without spatial sample weighting. Therefore, the weighting design is **implemented and currently being evaluated**, not a confirmed component of the final production model.

## Evaluation protocols

### Random rows

Random row-level validation measures interpolation within the observed sample pool. Repeated observations from a station can occur on both sides of the split, so this protocol is useful for debugging and baseline comparison but is not sufficient for geographic generalization claims.

### Held-out stations

All observations from selected stations are excluded from training. This tests transfer to unseen monitoring locations and reduces direct station identity leakage.

### Held-out spatial blocks

Entire 2° spatial blocks are assigned to the test partition. This is stricter than held-out-station evaluation because nearby stations do not straddle the train-test boundary within the same block.

### Future dates

The latest dates form the test partition and are excluded from training. This tests temporal transfer and prevents later observations from informing the fitted model.

## Metric interpretation

R², RMSE, MAE, and mean bias are reported separately for each split. High-concentration bias and block-level macro summaries are planned or implemented as diagnostics. A metric is not transferred across a different model, feature set, weighting scheme, or split.

## Disclosure boundary

This methods note intentionally omits data paths, station identifiers, restricted records, model files, code, credentials, infrastructure details, and unpublished weighted-result tables. The current public evidence supports the early unweighted random and station-grouped baselines; it does not yet support a numeric claim for the weighted spatial-block or future-date experiments.
