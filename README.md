# certified-data-driven-feedback-sim

This work addresses **certified data-driven feedback synthesis under lifted hard polyhedral constraints**

`Sξ + Gu + Ew ≤ b`,

where `w` is an auxiliary decision variable. For a given operating condition `ξ`, a control action `u` is admissible if there exists some `w` satisfying the constraints. The corresponding admissible-control set is

`U_f(ξ) = {u | ∃ w: Sξ + Gu + Ew ≤ b}`.

Given finite reference data, the objective is to construct a feedback law that recovers the desired reference behavior while ensuring hard-constraint admissibility throughout a prescribed operating domain and a uniform bound on online execution time.

The resulting feedback controller follows two mappings:

`ξ_k ↠ {u_{k,j}}_{j=1}^{K_k} ⊆ U_f(ξ_k)`

`({u_{k,j}}_{j=1}^{K_k}, ξ_k, η_k) → u_k ∈ conv{u_{k,j}}_{j=1}^{K_k} ⊆ U_f(ξ_k)`.

The first mapping returns admissible control candidates under the hard constraints. The second combines these candidates using the current operating condition and reference or task information to determine the applied control input.

<p align="center">
  <img src="paper_lateral_figures/Framework.png"
       alt="Certified data-driven feedback structure"
       width="720">
</p>

<p align="center">
  <em>Certified feedback structure for direct online control evaluation.</em>
</p>

This repository considers one important specialization of the general formulation: **finite-horizon constrained control**. In this setting, `w` represents the future control sequence, so an applied input is admissible only if it admits a feasible continuation satisfying the remaining constraints.

The repository provides the simulation and evaluation code for the **lateral-vehicle benchmark** used in the paper. The benchmark implements the complete closed-loop setting and evaluates reference-behavior recovery, hard-constraint satisfaction, recursive feasibility, and online execution time.

## Repository contents

```text
.
├─ paper_lateral_figures/                        # Figures generated for the lateral-vehicle benchmark
├─ simulate_lateral_controller.m                 # Runs the three-horizon simulations, reports statistics, and generates figures
├─ lateral_controller.mat                        # Precomputed controllers, certified law libraries, and deployed executor data
├─ lateral_simulation_results.mat                # Precomputed simulation results for the reported benchmark cases
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

Both controllers are evaluated as **compiled MEX implementations under the same closed-loop benchmark conditions**, including the vehicle model, hard constraints, sampling time, prediction horizon, initial conditions, and simulation duration. The reference MPC is used without retuning.

The reference controller is formulated as a condensed quadratic program, solved using the **active-set solver `mpcActiveSetSolver` from the MATLAB Model Predictive Control Toolbox**, and compiled into MEX using MATLAB Coder. The proposed controller is deployed as a **C-MEX implementation**. The compiled reference controller is denoted **MPC (MEX)**.

For both controllers, **the current state is used to compute one control action at each sampling instant**. Before measurement, both implementations execute the same number of untimed warm-up calls and their internal logical states are reset. Execution time is measured using `tic`/`toc` around the **state-to-action MEX call**. Plant propagation, data storage, constraint diagnostics, and full-sequence reconstruction are excluded from the timed region.

Accordingly, the reported execution times represent the **measured end-to-end state-to-action latency** of the two compiled implementations under matched benchmark conditions.

## Running the simulation

1. Place all repository files in the same directory and set the MATLAB working directory to that folder.
2. In `simulate_lateral_controller.m`, select:

```matlab
runMode = "simulate";
```

3. Run:

```matlab
simulate_lateral_controller
```

The script evaluates all three prediction horizons, prints the complete report, saves the results to `lateral_simulation_results.mat`, and exports the figures and summary files to `paper_lateral_figures/`.

To regenerate reports and figures without rerunning the simulations, keep `lateral_simulation_results.mat` in the same directory and use:

```matlab
runMode = "report_and_plot";
```
