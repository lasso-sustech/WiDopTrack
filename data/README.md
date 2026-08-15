# Released Data

## Directory structure

```text
cfar_input/
  full_recordings/       full-context time-Doppler maps for the released cases
  segments/              released case segments
doppler_candidates_full/ full-context CFAR, DP, and quality-screening outputs
doppler_candidates/      outputs cropped to the released cases
processed/               consolidated per-window CSV tables
ground_truth/            interpolated RTK trajectories in NPZ and CSV formats
manifest.json            public case metadata
```

Every sensing directory contains `sur1`, `sur2`, and `sur3`, corresponding to
the three bistatic WiFi links.

## Recommended starting point

The easiest files to inspect are `processed/uav*_window_data.csv`. Each row is
one processing-window center and contains:

| Column | Meaning |
|---|---|
| `time_s` | Time relative to the beginning of the released case |
| `sur*_input_valid` | Acquisition-level validity of the input window |
| `sur*_dp_doppler_hz` | Continuous Doppler estimate recovered by DP |
| `sur*_retained` | Post-DP quality-screening indicator |
| `sur*_retained_doppler_hz` | DP estimate retained for trajectory reconstruction; blank otherwise |
| `rtk_x_m`, `rtk_y_m`, `rtk_z_m` | RTK position in the local Cartesian frame |

## Time-Doppler MAT files

The most relevant fields under `cfar_input` are:

| Field | Meaning |
|---|---|
| `A_TD` | Time-Doppler map, stored as time by Doppler bin |
| `raw_magnitude_map` | CAF/time-Doppler magnitude map |
| `normalized_db_map` | Display-normalized map in decibels |
| `time_axis_s` | Processing-window center times relative to the released case |
| `doppler_axis_hz` | Doppler-bin centers |
| `valid_window_mask` | Acquisition-level window validity |
| `ref_power_mean` | Mean reference-channel power per window |
| `relative_time_axis_s` | Duplicate relative time axis retained for compatibility; segment files only |

For the released segments, `A_TD`, `raw_magnitude_map`, and
`normalized_db_map` have 501 rows and 37 Doppler columns. The Doppler axis spans
-45 Hz to 45 Hz, and the median center interval is 0.04 s.

## Candidate MAT files

The main fields under `doppler_candidates` and
`doppler_candidates_full` are:

| Field | Meaning |
|---|---|
| `time_axis_s` | Processing-window center times |
| `doppler_axis_hz` | Doppler-bin centers |
| `candidate_time_index` | Time index of every stored CFAR candidate |
| `candidate_doppler_hz` | Doppler value of every candidate |
| `candidate_response` | Candidate response strength |
| `candidate_track_score` | Candidate score used by ridge tracking |
| `candidate_support_flag` | Candidate local-support flag |
| `main_track_doppler_hz` | Continuous DP ridge estimate |
| `main_track_valid_mask` | Post-DP retention indicator |
| `main_track_doppler_hz_masked` | Retained DP estimate, with NaN elsewhere |
| `selected_tracking_branch` | Release-generation processing branch (`direct` or `rpca`) |

In the released candidate MAT files, use the time-aligned `main_track_*`,
`candidate_*`, and `time_axis_s` fields for the released interval. The
diagnostic matrices `candidate_mask` and `candidate_energy_mask` retain the
full-recording context and can therefore be wider than the cropped interval.

## Ground-truth files

Each NPZ file contains:

- `time_s`: relative RTK times aligned with valid processing-window centers;
- `position_xyz_m`: local Cartesian RTK positions in meters.

The CSV files contain the same values under `time_s,x_m,y_m,z_m`.

## Post-DP retention ratios

| Case | Link 1 | Link 2 | Link 3 |
|---|---:|---:|---:|
| UAV 1 | 77.2% | 84.8% | 77.8% |
| UAV 2 | 82.8% | 78.8% | 75.4% |
| UAV 3 | 86.8% | 84.4% | 18.6% |
| UAV 4 | 72.1% | 82.4% | 50.7% |

The data lineage is described in detail in
[`../DATA_PROCESSING.md`](../DATA_PROCESSING.md).
