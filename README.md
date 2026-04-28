# CLP-CSGM

Conditional Latent-Prior Compressed Sensing with Generative Models
(CLP-CSGM) for petrophysical property estimation from dense logs and sparse
target measurements.

This repository contains the code needed to reproduce the CLP-CSGM experiments
reported in the paper. It intentionally excludes raw well-log data and large
experiment outputs.

## Method

CLP-CSGM learns an autoencoder decoder \(G(z)\) on target-property windows and
a conditional latent prior \(z_0=h(u)\) from dense log windows. At validation and
test time, the latent code is refined by

```text
z_hat = argmin_z ||M G(z) - b||_2^2 + lambda ||z - z0(u)||_2^2
y_hat = G(z_hat)
```

The released benchmark scripts use Adam in latent space. They do not use FISTA
or LFISTA when executed with `--no-lfista`.

## Repository Contents

- `csgm_m2_module.py`: CLP-CSGM model, autoencoder, Ridge/MLP latent priors,
  latent optimization, and ablation branches.
- `sir_cs_benchmark_multi_well_vc.py`: cross-well Vc benchmark launcher.
- `sir_cs_benchmark_real_well_direct_ub.py`: F03-4 GR-only porosity launcher.
- `sir_cs_benchmark_direct_ub.py`: shared direct `[u,b] -> y` benchmark
  infrastructure and baseline runner.
- `sir_cs_pipeline_optimized.py`: shared configuration, metrics, plotting, and
  measurement utilities used by the benchmark launchers.
- `direct_ub_baselines.py`: direct MLP, PCA, and AE `[u,b]` baselines.
- `external_benchmarks.py`: shared metric-row utilities and legacy sparse
  baselines used by the benchmark infrastructure.
- `multi_well_vc.py`: cross-well Vc dataset loader/window builder.
- `real_well_f03.py`: F03-4 porosity dataset loader/window builder.
- `scripts/`: CLP-CSGM result consolidation and reproduction helpers.

Some filenames retain historical `sir_cs_*` names because the CLP-CSGM paper
was developed on top of a broader compressed-sensing benchmark codebase. In
this repository, use the commands below with `--run-csgm-m2` and `--no-lfista`
to run only the CLP-CSGM branch and its direct baselines.

## Data

Raw data files are not included. Create a `data/` directory and place the
required private/local files there. See `data/README.md` for filenames and
expected schemas.

Required files for the paper experiments:

- `data/F02-1,F03-2,F06-1_6logs_30dB.txt`
- `data/F03-4_6logs_30dB.txt`
- `data/F03-4_AC+GR+Porosity.txt`

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Reproduce The Paper Runs

The full paper grid can be launched with:

```bash
bash scripts/run_reproduce_clp_csgm.sh
```

The script expects the private data files to be present under `data/`.

## Individual Commands

Cross-well Vc, Ridge prior with structural ablations:

```bash
python sir_cs_benchmark_multi_well_vc.py \
  --train-path data/F02-1,F03-2,F06-1_6logs_30dB.txt \
  --test-path data/F03-4_6logs_30dB.txt \
  --channels sonic,rhob,gr,ai,vp \
  --target vc \
  --step 8 \
  --measurement-ratios 0.05,0.10,0.20 \
  --seeds 7,23,41 \
  --run-csgm-m2 \
  --run-csgm-ablations \
  --csgm-prior-type ridge \
  --no-lfista
```

Repeat the same command with `--step 16` and `--step 32`.

Cross-well Vc, MLP prior sensitivity:

```bash
python sir_cs_benchmark_multi_well_vc.py \
  --train-path data/F02-1,F03-2,F06-1_6logs_30dB.txt \
  --test-path data/F03-4_6logs_30dB.txt \
  --channels sonic,rhob,gr,ai,vp \
  --target vc \
  --step 8 \
  --measurement-ratios 0.05,0.10,0.20 \
  --seeds 7,23,41 \
  --run-csgm-m2 \
  --csgm-prior-type mlp \
  --no-lfista
```

Repeat with `--step 16` and `--step 32`.

F03-4 GR-only porosity, Ridge prior with structural ablations:

```bash
python sir_cs_benchmark_real_well_direct_ub.py \
  --data-path data/F03-4_AC+GR+Porosity.txt \
  --u-channels gr \
  --measurement-ratios 0.20,0.30,0.40,0.50,0.60 \
  --seeds 7,23,41 \
  --run-csgm-m2 \
  --run-csgm-ablations \
  --csgm-prior-type ridge \
  --no-lfista
```

F03-4 GR-only porosity, MLP prior sensitivity:

```bash
python sir_cs_benchmark_real_well_direct_ub.py \
  --data-path data/F03-4_AC+GR+Porosity.txt \
  --u-channels gr \
  --measurement-ratios 0.20,0.30,0.40,0.50,0.60 \
  --seeds 7,23,41 \
  --run-csgm-m2 \
  --csgm-prior-type mlp \
  --no-lfista
```

## Fast Smoke Tests

After placing the data files, use `--fast` for a short validation run:

```bash
python sir_cs_benchmark_multi_well_vc.py \
  --train-path data/F02-1,F03-2,F06-1_6logs_30dB.txt \
  --test-path data/F03-4_6logs_30dB.txt \
  --channels sonic,rhob,gr,ai,vp \
  --target vc \
  --step 32 \
  --run-csgm-m2 \
  --run-csgm-ablations \
  --csgm-prior-type ridge \
  --no-lfista \
  --fast
```

```bash
python sir_cs_benchmark_real_well_direct_ub.py \
  --data-path data/F03-4_AC+GR+Porosity.txt \
  --u-channels gr \
  --run-csgm-m2 \
  --run-csgm-ablations \
  --csgm-prior-type ridge \
  --no-lfista \
  --fast
```

## Paper Asset Scripts

After the full runs are complete, generate CLP-CSGM tables and figures with:

```bash
python scripts/clp_csgm_ablation_assets.py
python scripts/clp_csgm_paper_assets.py
python scripts/clp_csgm_quick_figures.py
```

These scripts read benchmark CSV outputs and write paper-ready assets under
`paper_clp_csgm/`. The LaTeX manuscript itself is not included in this minimal
code repository.

## Notes On Reproducibility

- Main seeds: `7,23,41`.
- Cross-well low-data steps: `8,16,32`.
- Cross-well measurement ratios: `0.05,0.10,0.20`.
- F03-4 measurement ratios: `0.20,0.30,0.40,0.50,0.60`.
- CLP-CSGM Ridge is the main paper-facing variant.
- CLP-CSGM MLP is included only as a prior-class sensitivity check.

