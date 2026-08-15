# Data Processing and Provenance

This document records how each released data level was obtained. It separates
the sensing data used by the trajectory method from the RTK data used only for
evaluation.

## 1. Signal acquisition

Three bistatic WiFi links were deployed. Each link consisted of a WiFi router,
a reference antenna directed toward that router, and a surveillance antenna
directed toward the flight region. The two receiver channels were sampled at
1 MS/s.

Every processing window contains 0.4 s of samples. Adjacent processing-window
centers are separated by 0.04 s, so consecutive windows overlap. The overlap
provides a denser temporal sequence of Doppler measurements while retaining a
long enough coherent interval for Doppler processing.

Raw complex IQ recordings are not part of this public package. The earliest
released sensing level is the time-Doppler/CAF map in `data/cfar_input`.

## 2. Time-Doppler map generation

For every link and processing window, reference-signal leakage and static
multipath in the surveillance signal were suppressed. The cross-ambiguity
function between the interference-suppressed surveillance signal and the
reference signal was evaluated over 37 Doppler bins from -45 Hz to 45 Hz.
Stacking the CAF magnitudes from successive windows produced the time-Doppler
map.

The released files contain:

- `raw_magnitude_map`: the time-Doppler magnitude map;
- `A_TD`: the map supplied to the subsequent Doppler-processing stage;
- `time_axis_s`: the processing-window center times;
- `doppler_axis_hz`: the Doppler-bin centers;
- `valid_window_mask`: acquisition-level window validity;
- `ref_power_mean`: mean reference-channel power per window.

The full-context maps are stored under `data/cfar_input/full_recordings`. The
released case segments are stored under `data/cfar_input/segments`.

## 3. Clutter suppression and denoising

The background of each Doppler bin was estimated using its 30th percentile
over time and subtracted from that bin. The nonnegative residual was divided by
a robust scale based on the median absolute deviation (MAD). Slowly varying
horizontal and temporal background components were then suppressed.

The release-generation pipeline contains direct and RPCA processing branches.
The branch selected for a recording is recorded by `selected_tracking_branch`
in the candidate MAT file. The RPCA branch separates the normalized map into a
low-rank background and a sparse target-related component before candidate
detection.

## 4. CFAR candidate extraction and DP ridge recovery

CFAR compares each time-Doppler cell with a local background estimated from
training cells outside its guard region. Cells passing the response,
prominence, and column-support checks form the candidate set.

A second-order dynamic-programming tracker then searches the full recording for
a continuous ridge. The path score rewards strong local responses and penalizes
large first- and second-order Doppler changes. This produces one continuous
estimate `main_track_doppler_hz` for every processing window.

Post-DP quality screening checks the ridge response, track score, prominence,
peak ratio, and minimum run length. Its output is
`main_track_valid_mask`. The corresponding retained observation is stored in
`main_track_doppler_hz_masked`; non-retained samples are represented by NaN.

The full-context DP outputs are stored under `doppler_candidates_full`. The
released candidate files were extracted only after processing their full
temporal context, because normalization, denoising, and DP are nonlocal
operations and processing an isolated segment can alter the estimates near its
boundaries.

## 5. Public case organization

The four released flight cases are identified as `uav1`, `uav2`, `uav3`, and
`uav4`. Each segment normally contains 501 processing-window centers. The time
axis of each released case is expressed relative to the beginning of that case.

## 6. RTK ground truth

The RTK trajectory associated with each released case was read from the flight
log, cleaned by removing nonfinite samples and duplicate
timestamps, and expressed in the experiment's local Cartesian coordinate
frame. The vertical coordinate was converted to ground-relative height.

The RTK coordinates were linearly interpolated at the released
processing-window centers. They are stored as `time_s` and
`position_xyz_m` in NPZ format and as `time_s,x_m,y_m,z_m` in CSV format.
Some boundary centers fall outside the available RTK interval; these centers
are left blank in the consolidated CSV files and are excluded from the error
metrics. RTK data were not used to initialize, weight, or optimize the
reconstructed trajectory.

## 7. Coarse trajectory reconstruction

For each processing window, the EKF measurement set contains only links whose
post-DP retention indicator equals one. If all three links are absent, the EKF
performs prediction without a measurement update. Candidate initial states are
evaluated, and the retained coarse sequence is processed by an RTS smoother.

The measurement-noise standard deviation is 5 Hz for all three links. The EKF
and EKF+RTS position sequences in the result CSV files are the two coarse-stage
baselines reported in the result table.

## 8. Reliability weighting and trajectory fine tuning

For a retained observation at window `k` and link `l`, the trajectory-level
weight is the product of:

- a link factor determined by the post-DP retention ratio and the mean absolute
  Doppler residual relative to the coarse trajectory;
- a local ridge-support factor computed over seven processing windows; and
- a Cauchy attenuation factor with a 20-Hz residual scale.

Control points sampled from the coarse trajectory are jointly optimized using
the weighted normalized Doppler residuals, a coarse-trajectory baseline term,
and a control-point smoothness term. Velocity- and acceleration-limit penalties
are not used in the released final results. Ordinary squared loss is used. The
optimized control points are linearly interpolated, and a 31-point moving
average is applied to the position sequence.


## 9. Error calculation

At every window with available RTK coordinates, the horizontal error is the
Euclidean distance between the reconstructed and RTK positions in the local
XY plane. The reported mean error is the arithmetic mean of these distances,
and RMSE is the square root of their mean squared value.

The complete pointwise values are under `results/trajectories`, while the
aggregate values are under `results/reproduced_summary.csv`.
