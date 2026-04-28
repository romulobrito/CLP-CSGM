# Data Files

Raw well-log data are not included in this public repository.

Place the following files in this directory before running the reproduction
commands:

```text
data/
  F02-1,F03-2,F06-1_6logs_30dB.txt
  F03-4_6logs_30dB.txt
  F03-4_AC+GR+Porosity.txt
```

## Cross-Well Vc Files

Used by `sir_cs_benchmark_multi_well_vc.py`:

- `F02-1,F03-2,F06-1_6logs_30dB.txt`
- `F03-4_6logs_30dB.txt`

Expected content:

- Tabular text readable by `pandas.read_csv(..., sep=None, engine="python")` or
  by the delimiter fallback in `multi_well_vc.py`.
- Columns resolvable to:
  - `depth`
  - `sonic`
  - `rhob`
  - `gr`
  - `ai`
  - `vp`
  - `vc`

The train file contains F02-1, F03-2, and F06-1. The test file contains F03-4.
Well boundaries in concatenated files are inferred from depth discontinuities
and filename well names.

## F03-4 Porosity File

Used by `sir_cs_benchmark_real_well_direct_ub.py`:

- `F03-4_AC+GR+Porosity.txt`

Expected content:

- Tab-separated table.
- Columns resolvable to:
  - `Depth`
  - `AC`
  - `GR`
  - `Porosity`

The paper experiment uses `--u-channels gr`, so only GR is used as the dense
input even though AC is present in the file.

## Privacy

Do not commit raw data files. The repository `.gitignore` excludes data tables
by default.

