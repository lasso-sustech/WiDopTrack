# Released Results

## Aggregate metrics

`reproduced_summary.csv` and `reproduced_summary.md` contain the horizontal
mean error and RMSE of EKF, EKF+RTS, and JointOpt for the four released cases.
Errors are evaluated only at processing-window centers with available RTK
coordinates.

## Pointwise trajectories

`trajectories/uav*_trajectories.csv` contains:

- original and relative time;
- RTK XYZ position;
- EKF XY position and horizontal error;
- EKF+RTS XY position and horizontal error;
- JointOpt XYZ position and horizontal error.

The number of evaluated RTK points is 501, 501, 500, and 499 for UAV 1--4,
respectively. Blank RTK/error fields identify processing-window centers outside
the available RTK interval.

## Figures

`figures` contains one XY trajectory comparison and one horizontal-error CDF
for each released case.
