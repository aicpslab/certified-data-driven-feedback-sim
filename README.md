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

Both controllers are evaluated as compiled MEX implementations under identical closed-loop conditions, including the vehicle model, constraints, sampling time, prediction horizon, initial states, and simulation duration. The reference MPC is used without retuning.

The reference MPC solves a condensed QP using the **`mpcActiveSetSolver` active-set solver from the MATLAB Model Predictive Control Toolbox** and is compiled with MATLAB Coder. The proposed controller is implemented as C-MEX. The compiled reference is denoted **MPC (MEX)**.

After equal untimed warm-up calls and internal-state resets, `tic`/`toc` measures each state-to-action MEX call. Plant propagation, data storage, diagnostics, and full-sequence reconstruction are excluded. The reported times therefore represent matched end-to-end implementation latency, not solver-independent complexity.
