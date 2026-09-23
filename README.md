# certified-data-driven-feedback-sim

This repository provides the simulation and evaluation artifacts for the numerical study of **certified data-driven feedback synthesis under hard constraints**.

The proposed controller separates control admissibility from behavior realization. A **certified set-valued feedback map** returns a finite set of admissible control candidates at the current operating condition, and a **single-valued feedback law** combines these candidates to determine the applied control input. Hard-constraint feasibility is therefore built into the feedback structure.

<p align="center">
  <img src="paper_lateral_figures/Framework.png"
       alt="Certified data-driven feedback structure"
       width="600">
</p>

<p align="center">
  <em>Certified feedback structure used in the closed-loop implementation.</em>
</p>

The **lateral-vehicle benchmark** is used as a finite-horizon constrained-control specialization of the general formulation. In this setting, admissibility of the current control input depends on the existence of a feasible future continuation. The benchmark therefore provides a complete closed-loop setting in which reference-behavior recovery, hard-constraint satisfaction, recursive feasibility, and online execution time can be evaluated together.

## Repository contents

```text
.
├─ paper_lateral_figures/                        # Figures for the lateral-vehicle benchmark
├─ simulate_lateral_controller.m                 # Runs the three-horizon simulations, reports statistics, and generates figures
├─ lateral_controller.mat                        # Precomputed controllers, certified law libraries, and deployed executor data
├─ lateral_simulation_results.mat                # Precomputed simulation results for the reported benchmark cases
├─ certnnmpc_executor_mex.mexw64                 # C-MEX implementation of the proposed and coverage-only controllers
├─ lateral_mpc_kwik_mex_Np15_Nc15.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 15
├─ lateral_mpc_kwik_mex_Np40_Nc40.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 40
└─ lateral_mpc_kwik_mex_Np66_Nc66.mexw64        # Condensed KWIK MPC baseline for Np = Nc = 66
```

The provided script and data files contain the vehicle-model, constraint, prediction-horizon, controller, and evaluation parameters required for the reported lateral-vehicle closed-loop simulations. No separate benchmark-parameter files are required.

## Tested environment

- **OS:** Microsoft Windows 11 Pro
- **CPU:** 12th Gen Intel(R) Core(TM) i7-12700KF (12 cores)
- **MATLAB:** MATLAB 25.1.0.2973910 (R2025a) Update 1
- **Required MATLAB toolboxes:** Model Predictive Control Toolbox; Optimization Toolbox; Statistics and Machine Learning Toolbox (used only for percentile-based timing and performance statistics)
- **MEX platform:** 64-bit Windows (`.mexw64`)

## Running the simulation

1. Clone or download the repository and set the MATLAB working directory to the repository root.
2. In `simulate_lateral_controller.m`, select:

```matlab
runMode = "simulate";
```

3. Run:

```matlab
simulate_lateral_controller
```

The script evaluates all three prediction horizons, prints the complete report, saves the results to `lateral_simulation_results.mat`, and exports the figures and summary files to `paper_lateral_figures/`.

To regenerate the reports and figures without rerunning the simulations, keep `lateral_simulation_results.mat` in the repository root and use:

```matlab
runMode = "report_and_plot";
```

## Comparison fairness

Both controllers are evaluated as **compiled MEX implementations under the same closed-loop benchmark conditions**, including the vehicle model, hard constraints, sampling time, prediction horizon, initial conditions, and simulation duration. The reference MPC is used without retuning and is formulated as a condensed quadratic program solved using the **active-set solver `mpcActiveSetSolver` from the MATLAB Model Predictive Control Toolbox**, with code generated using MATLAB Coder. The proposed controller is deployed as a **C-MEX implementation**, and the compiled reference controller is denoted **MPC (MEX)**.

For both controllers, **the current state is used to compute one control action at each sampling instant**. The same untimed warm-up procedure is applied before measurement, and execution time is measured using `tic`/`toc` around the **state-to-action MEX call**. Plant propagation, data storage, constraint diagnostics, and full-sequence reconstruction are excluded. The reported times therefore represent the **measured end-to-end state-to-action latency** of the two compiled implementations under matched benchmark conditions.

## Main benchmark results

The numerical evaluation considers three prediction horizons, `Np = Nc = 15, 40, 66`, using the same 50 held-out initial conditions. Panels (a)–(c) show the closed-loop lateral tracking error for the three horizons: the solid curves denote the median absolute error, and the shaded regions indicate the 10th–90th percentile range across the tested initial conditions. Panels (d)–(f) summarize the relative tracking-cost gap, online execution time, and deployed library size, respectively.

<p align="center">
  <img src="paper_lateral_figures/lateral_random_error_Np15.png"
       alt="Closed-loop tracking for Np = 15"
       width="32%">
  <img src="paper_lateral_figures/lateral_random_error_Np40.png"
       alt="Closed-loop tracking for Np = 40"
       width="32%">
  <img src="paper_lateral_figures/lateral_random_error_Np66.png"
       alt="Closed-loop tracking for Np = 66"
       width="32%">
</p>

<p align="center">
  <em>(a) Np = 15.</em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <em>(b) Np = 40.</em>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <em>(c) Np = 66.</em>
</p>

<p align="center">
  <img src="paper_lateral_figures/lateral_horizon_delta_Je.png"
       alt="Relative tracking-cost gap"
       width="32%">
  <img src="paper_lateral_figures/lateral_horizon_time.png"
       alt="Online execution time"
       width="32%">
  <img src="paper_lateral_figures/lateral_horizon_law_count.png"
       alt="Deployed library size"
       width="32%">
</p>

<p align="center">
  <em>(d) Relative tracking-cost gap.</em>&nbsp;&nbsp;
  <em>(e) Online execution time.</em>&nbsp;&nbsp;
  <em>(f) Deployed library size.</em>
</p>

**(i) Reference-behavior recovery.**  
Across the three prediction horizons, the proposed controller remains close to the reference MPC behavior. The median relative tracking-cost gap is `5.8%`, `1.3%`, and `1.0%` for `Np = 15, 40, 66`, respectively, compared with substantially larger gaps for the coverage-only controller.

**(ii) Online execution time.**  
The proposed controller maintains low measured online latency across all three horizons. Relative to **MPC (MEX)**, the mean state-to-action execution-time speedup is approximately `22.4×`, `32.1×`, and `85.3×` for `Np = 15, 40, 66`, respectively.

**(iii) Hard-constraint satisfaction and recursive feasibility.**  
No hard-constraint violations were observed over the tested initial conditions and prediction horizons, and recursive feasibility was maintained throughout the simulations.

Taken together, the benchmark demonstrates the proposed synthesis from offline construction to direct online feedback realization, with close reference-behavior recovery and low measured latency while satisfying the hard constraints and maintaining recursive feasibility. The proposed method is not tied to MPC or to this benchmark: here, MPC provides the reference behavior and compiled baseline, while the lateral-vehicle benchmark provides a finite-horizon closed-loop setting in which these properties can be evaluated together.

## Citation

This repository accompanies the manuscript **“Certified Data-Driven Feedback Synthesis under Lifted Hard Polyhedral Constraints,”** which is currently under review. Full bibliographic information and a persistent publication link will be added here once available.

If you use the code or numerical results in this repository, please cite the associated manuscript. Complete citation information will be updated upon publication.

## Additional information

For questions regarding reproduction of the benchmark or the provided implementation, please open an issue in this repository or contact the authors. For MATLAB toolbox installation, licensing, or platform-specific issues, please refer to the corresponding MathWorks documentation.

The provided MEX binaries target 64-bit Windows (`.mexw64`) and were generated and tested in the environment listed above.
