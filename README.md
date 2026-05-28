# Reproducibility materials

Data and a single notebook to reproduce the tables and figures of

> **Are Randomized Quantum Linear Systems Solvers Practical?**
> S. Hariprakash, R. Van Beeumen, K. Klymko, D. Camps.

The notebook performs **no simulation**. Every Monte-Carlo run was performed
offline and the results are stored in `data/`; `reproduce_figures.ipynb` reads
those files and regenerates the published artifacts.

## Quick start

```bash
pip install -r requirements.txt
jupyter notebook reproduce_figures.ipynb   # run all cells
```

Running all cells writes the four figures to `figures/` and prints the two
tables. No random seed is involved: the analytical bounds and Fourier-time
tail distributions are computed exactly, and the empirical RMSE curves are
read directly from the saved runs.

## What gets reproduced

| Paper artifact | Reference | Data source |
|---|---|---|
| Table I — Fourier error, $N=4$ | `Table 2` | `data/fourier_tables/table_N4_*.csv` |
| Table II — Fourier error, $N=16$ | `Table 3` | `data/fourier_tables/table_N16_*.csv` |
| Fig. — PF convergence, $\kappa=100$ | `Figure 3` | `data/convergence/pf_kappa100.csv` |
| Fig. — $\lvert\tau\rvert$ tail, $\kappa=100$ | `fig:fourier_time_distribution` | `data/fourier_nodes/kappa100/*.npz` |
| Fig. — PF vs RTE convergence, $\kappa=3$ | `Figure 4` | `data/convergence/pf_kappa3.csv`, `rte_kappa3_c1.csv` |
| Fig. — $\lvert\tau\rvert$ tail, $\kappa=3$ | `Figure 5` | `data/fourier_nodes/kappa3/*.npz` |

## Data layout

```
data/
├── fourier_tables/                 # Tables I & II (§6.1 of the paper)
│   ├── table_N4_summary.csv        #   per-cell (J,K), eps_mean, eps_max, ratios
│   ├── table_N4_raw.csv            #   per-trial errors (100 trials per cell)
│   ├── table_N16_summary.csv
│   └── table_N16_raw.csv
├── convergence/                    # RMSE-vs-N_S curves
│   ├── pf_kappa100.csv             #   PF, kappa=100, 10 instances, N_S up to 1e8
│   ├── pf_kappa3.csv               #   PF, kappa=3,  10 instances, N_S up to 1e6
│   └── rte_kappa3_c1.csv           #   RTE (c_RTE=1), kappa=3, same 10 instances
└── fourier_nodes/                  # Fourier nodes + sampling distributions
    ├── kappa100/instance_***.npz   #   y/z nodes, p_y, p_z for the tau distribution
    └── kappa3/instance_***.npz
```

### Column conventions (convergence CSVs)

Each row is one `(instance, N_S)` measurement.

- `rmse` — empirical RMSE of the estimator over 20 Monte-Carlo runs.
- `var_prefactor` — per-instance variance prefactor entering the analytical
  bound: `2 * exp(2/c_RTE) * (N_y N_z)^2 / lambda^2` (the `exp` factor is 1 for PF).
- `lambda`, `t_max`, `N_y`, `N_z`, `J`, `K` — instance constants.
- `target_value` — exact `<0|A^{-1}|0>` for that instance.

The max-across-instances analytical RMSE bound plotted in the paper is

```
max_i  sqrt( (eps_F/2)^2 + var_prefactor_i / N_S )  +  eps_F ,     eps_F = 2e-3 .
```

### `.npz` contents (Fourier nodes)

`y_nodes`, `z_nodes`, `p_y`, `p_z` give the product sampling distribution; the
sampled Fourier time is `|tau| = |y_j z_k|` with probability `p_y[j] * p_z[k]`.
Scalars `t_max`, `lambda`, `N_y`, `N_z`, `J`, `K`, `kappa`, `eps_F` are also stored.

## Notes

- The κ = 100 study is **PF only**: at κ = 100 the per-sample RTE work scales as
  `tau^2 ≈ 10^6`, making a classical RTE simulation (≈ 10^15 matrix
  multiplications across all samples and instances) intractable. The PF–RTE
  comparison is therefore carried out at κ = 3, where both are tractable on the
  *same* matrix instances.
- The PF empirical bundle at κ = 3 runs to `N_S = 1e6`; the analytical bounds
  are closed-form and are drawn across the full sample range.
