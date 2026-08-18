# Doppler-based Drone Trajectory Reconstruction via Bistatic WiFi Sensing

## Overview

This dataset contains the intermediate sensing data, post-DP Doppler estimates,
RTK ground truth, and trajectory-reconstruction results obtained in outdoor
experiments with three distributed bistatic WiFi links. It is intended for
studying Doppler-only drone trajectory reconstruction when the starting
position is unknown and the Doppler estimates from different WiFi links have
unequal reliability.

## Keywords

Passive WiFi sensing, bistatic Doppler, drone tracking, trajectory
reconstruction, distributed sensing, low-altitude networks.

## Experiment Setup

- **Transmitters:** Three commercial WiFi routers operating on different WiFi
  channels.
- **Receivers:** Three USRP X310 software-defined radios. Each receiver uses
  one directional antenna for the reference channel and one for the
  surveillance channel.
- **Target:** DJI M400 drone.
- **Sampling rate:** 1 MSa/s.
- **Processing-window duration:** 0.4 s.
- **Interval between adjacent processing-window centers:** 0.04 s.
- **Doppler grid:** 37 bins from -45 Hz to 45 Hz.
- **Transmitter--receiver baselines:** 2.48 m, 2.57 m, and 1.80 m for the three
  bistatic links.
- **Pairwise transmitter separations:** 13.63 m, 16.52 m, and 12.10 m.

<p align="center">
  <img src="Imgs/system_model.png" width="42%" alt="System model">
  <img src="Imgs/experiment_setup.png" width="42%" alt="Experiment setup">
</p>

## Dataset Contents
Due to its large size, the dataset cannot be directly uploaded to GitHub. Please download it from the following link:
https://lasso525.quickconnect.cn/d/s/19TlCJasfD3vkoonw6ZUy76dRSd06vAb/B8HsRs41ZX_PNLwRdcxgN0lUWN4JcDBc-p7Ggyry3bQ0

The release contains four flight cases. Each case includes data from three
surveillance links, denoted by `sur1`, `sur2`, and `sur3`.

| Released case |
|---|
| `uav1` |
| `uav2` |
| `uav3` |
| `uav4` |

```text
data/
  cfar_input/
    full_recordings/       full time-Doppler maps used as processing context
    segments/              released 20-s time-Doppler segments
  doppler_candidates_full/ full-context CFAR/DP outputs
  doppler_candidates/      cropped CFAR/DP outputs for the four cases
  processed/               readable per-window Doppler and RTK CSV files
  ground_truth/            RTK trajectories in NPZ and CSV formats
  manifest.json            public case metadata
results/
  trajectories/            per-window EKF, EKF+RTS, JointOpt, and RTK positions
  figures/                 trajectory and error-CDF figures
  reproduced_summary.csv   numerical error summary
Imgs/
  raw_doppler/             unprocessed time-Doppler spectrograms
  recovered_doppler/       recovered Doppler-ridge figures
```

The MAT files in `cfar_input` contain time-Doppler/CAF maps rather than raw
complex baseband IQ samples. See [DATA_PROCESSING.md](DATA_PROCESSING.md) for
the complete data lineage and [data/README.md](data/README.md) for the field
definitions.

## Data Processing Pipeline

1. The reference and surveillance baseband signals were divided into 0.4-s
   processing windows with a 0.04-s center interval.
2. Reference leakage and static multipath were suppressed before the CAF was
   evaluated over the Doppler grid. Aggregating all processing windows produced
   one time-Doppler map per bistatic link.
3. Each Doppler bin was normalized using percentile-background subtraction and
   a MAD-based robust scale. Clutter suppression and denoising were then applied.
4. CFAR produced Doppler candidates, and second-order dynamic programming
   recovered a continuous Doppler ridge. A post-DP quality check generated the
   retained-point indicator for every link and processing window.
5. The retained Doppler estimates were used by the EKF and RTS smoother to
   construct a coarse trajectory. Reliability-weighted control-point
   optimization and moving-average post-processing produced the final JointOpt
   trajectory.
6. RTK positions were transformed to the experiment's local Cartesian frame
   and interpolated at the processing-window centers. RTK data were used only
   for evaluation and plotting.

### Unprocessed Time-Doppler Spectrograms

These input time-Doppler maps are shown before clutter suppression, denoising,
CFAR detection, and DP ridge recovery.

#### UAV 1

<p align="center">
  <img src="Imgs/raw_doppler/uav1_link1_raw_doppler.png" width="32%" alt="UAV 1 Link 1 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav1_link2_raw_doppler.png" width="32%" alt="UAV 1 Link 2 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav1_link3_raw_doppler.png" width="32%" alt="UAV 1 Link 3 unprocessed time-Doppler spectrogram">
</p>

#### UAV 2

<p align="center">
  <img src="Imgs/raw_doppler/uav2_link1_raw_doppler.png" width="32%" alt="UAV 2 Link 1 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav2_link2_raw_doppler.png" width="32%" alt="UAV 2 Link 2 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav2_link3_raw_doppler.png" width="32%" alt="UAV 2 Link 3 unprocessed time-Doppler spectrogram">
</p>

#### UAV 3

<p align="center">
  <img src="Imgs/raw_doppler/uav3_link1_raw_doppler.png" width="32%" alt="UAV 3 Link 1 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav3_link2_raw_doppler.png" width="32%" alt="UAV 3 Link 2 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav3_link3_raw_doppler.png" width="32%" alt="UAV 3 Link 3 unprocessed time-Doppler spectrogram">
</p>

#### UAV 4

<p align="center">
  <img src="Imgs/raw_doppler/uav4_link1_raw_doppler.png" width="32%" alt="UAV 4 Link 1 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav4_link2_raw_doppler.png" width="32%" alt="UAV 4 Link 2 unprocessed time-Doppler spectrogram">
  <img src="Imgs/raw_doppler/uav4_link3_raw_doppler.png" width="32%" alt="UAV 4 Link 3 unprocessed time-Doppler spectrogram">
</p>

### Recovered Doppler Ridges

The figures below show the continuous DP ridge, the ridge points retained by
the post-DP quality check, and the RTK-derived Doppler for all three links in
each released case.

#### UAV 1

<p align="center">
  <img src="Imgs/recovered_doppler/uav1_link1_recovered_doppler_ridge.png" width="32%" alt="UAV 1 Link 1 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav1_link2_recovered_doppler_ridge.png" width="32%" alt="UAV 1 Link 2 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav1_link3_recovered_doppler_ridge.png" width="32%" alt="UAV 1 Link 3 recovered Doppler ridge">
</p>

#### UAV 2

<p align="center">
  <img src="Imgs/recovered_doppler/uav2_link1_recovered_doppler_ridge.png" width="32%" alt="UAV 2 Link 1 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav2_link2_recovered_doppler_ridge.png" width="32%" alt="UAV 2 Link 2 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav2_link3_recovered_doppler_ridge.png" width="32%" alt="UAV 2 Link 3 recovered Doppler ridge">
</p>

#### UAV 3

<p align="center">
  <img src="Imgs/recovered_doppler/uav3_link1_recovered_doppler_ridge.png" width="32%" alt="UAV 3 Link 1 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav3_link2_recovered_doppler_ridge.png" width="32%" alt="UAV 3 Link 2 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav3_link3_recovered_doppler_ridge.png" width="32%" alt="UAV 3 Link 3 recovered Doppler ridge">
</p>

#### UAV 4

<p align="center">
  <img src="Imgs/recovered_doppler/uav4_link1_recovered_doppler_ridge.png" width="32%" alt="UAV 4 Link 1 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav4_link2_recovered_doppler_ridge.png" width="32%" alt="UAV 4 Link 2 recovered Doppler ridge">
  <img src="Imgs/recovered_doppler/uav4_link3_recovered_doppler_ridge.png" width="32%" alt="UAV 4 Link 3 recovered Doppler ridge">
</p>

## Results

Horizontal errors are evaluated against the RTK trajectory. Detailed
per-window trajectories and errors are available under `results/trajectories`.

| Case | Method | Mean error (m) | RMSE (m) |
|---|---|---:|---:|
| UAV 1 | EKF | 1.994 | 2.334 |
| UAV 1 | EKF+RTS | 1.596 | 1.902 |
| UAV 1 | JointOpt | 0.913 | 0.945 |
| UAV 2 | EKF | 2.338 | 2.741 |
| UAV 2 | EKF+RTS | 2.098 | 2.390|
| UAV 2 | JointOpt | 0.739 | 0.782 |
| UAV 3 | EKF | 1.500 | 1.663 |
| UAV 3 | EKF+RTS | 1.461 | 1.634 |
| UAV 3 | JointOpt | 0.691 | 0.785 |
| UAV 4 | EKF | 1.923 | 2.077 |
| UAV 4 | EKF+RTS | 1.626 | 1.763 |
| UAV 4 | JointOpt | 0.957 | 1.049 |

The mean JointOpt error over the two manuscript cases, UAV 1 and UAV 2, is
approximately 0.83 m.

<p align="center">
  <img src="results/figures/uav1_trajectory.png" width="42%" alt="UAV 1 trajectory">
  <img src="results/figures/uav2_trajectory.png" width="42%" alt="UAV 2 trajectory">
</p>

## How to Use the Dataset

- Start with `data/processed/uav*_window_data.csv` for a readable table of the
  continuous DP estimates, post-DP retention indicators, retained Doppler
  observations, and RTK positions.
- Use `data/cfar_input/segments` to inspect the released time-Doppler maps.
- Use `data/cfar_input/full_recordings` when the surrounding temporal context
  is required to analyze normalization, denoising, CFAR, or DP boundary effects.
- Use `results/trajectories/uav*_trajectories.csv` to compare the reconstructed
  trajectories point by point.
- Use `results/reproduced_summary.csv` for the reported aggregate errors.

## License

A public data license has not yet been specified. Please add the intended
license before publishing the repository.

## Contact

For questions about the dataset, please contact Xianger Li at
`lixe2025@mail.sustech.edu.cn`.
