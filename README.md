# certified-data-driven-feedback-sim

Simulation and evaluation code for the lateral-vehicle benchmark of certified data-driven feedback control under hard constraints.

## Repository contents

```text
.
├─ simulate_lateral_controller.m                 # Runs the three-horizon simulations, reports statistics, and generates figures
├─ lateral_controller.mat                        # Precomputed controllers, certified law libraries, and deployed executor data
├─ certnnmpc_executor_mex.mexw64                 # C-MEX implementation of the proposed and coverage-only controllers
├─ lateral_mpc_kwik_mex_Np15_Nc15.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 15
├─ lateral_mpc_kwik_mex_Np40_Nc40.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 40
└─ lateral_mpc_kwik_mex_Np66_Nc66.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 66
```

## Tested environment

- **OS:** Microsoft Windows 11 Pro
- **CPU:** 12th Gen Intel(R) Core(TM) i7-12700KF (12 cores)
- **MATLAB:** MATLAB 25.1.0.2973910 (R2025a) Update 1
- **Required MATLAB toolboxes:** Model Predictive Control Toolbox; Optimization Toolbox; Statistics and Machine Learning Toolbox (used only for percentile-based timing and performance statistics).
- **MEX platform:** 64-bit Windows (`.mexw64`)

## Comparison fairness

Both controllers are evaluated as compiled MEX implementations within the same MATLAB closed-loop simulation environment. They use the same vehicle model, hard constraints, sampling time, prediction horizon, initial conditions, and simulation duration. The reference MPC is used without retuning.

The reference controller is formulated as a condensed quadratic program, solved using the **active-set solver `mpcActiveSetSolver` from the MATLAB Model Predictive Control Toolbox**, and compiled into MEX using MATLAB Coder. The proposed controller is deployed as a C-MEX implementation. The compiled reference controller is denoted **MPC (MEX)**.

Both implementations receive the current state and return one control action. Before measurement, they execute the same number of untimed warm-up calls and their internal logical states are reset. Execution time is measured using `tic`/`toc` around the state-to-action MEX call. Plant propagation, data storage, constraint diagnostics, and full-sequence reconstruction are excluded from the timed region and evaluated separately.

Therefore, the reported execution times represent end-to-end state-to-action latency under matched benchmark conditions, rather than solver-independent algorithmic complexity.
