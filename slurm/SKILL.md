---
name: slurm
description: Core reference for the SONIC SLURM cluster via phocus4 — architecture facts, filesystem tiers, venv strategy, launch patterns, dataset staging, and error catalog. Use as the foundational knowledge skill; for specific workflows use slurm-status (availability check), slurm-job (script creation), or slurm-debug (failure diagnosis).
---

# SONIC SLURM Cluster

## Overview

SONIC SLURM cluster is accessed exclusively via `ssh phocus4`. `squeue`/`sinfo`/`sbatch` need `SLURM_CONF` exported before every call because phocus4 does not set it in the default environment.

## Must-Do Before Any Submission

1. `ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sinfo -N -l"` — inspect which nodes are idle
2. `ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && squeue -u <username>"` — confirm current job count (keep ≤2 by default)
3. Pick a partition+node pair whose state is `idle` (not `allocated`, not `mixed` unless shared partition)

Skipping the `SLURM_CONF` export produces `Invalid user` / silent errors. Username on the cluster is the full login name (e.g. `arturxavier`), not the truncated `squeue` display.

## Partitions & GPU Architecture

| Partition | GPU | CUDA driver | Torch build | /tmp size | Notes |
|-----------|-----|-------------|-------------|-----------|-------|
| `gorgonas` | RTX 3090 Ti | CUDA 12.4+ | `cu124` | NVMe 200G+ | Copy datasets to `/tmp` for speed |
| `medusas` | RTX 5090 (Blackwell SM120) | 570.x / CUDA 12.8 | `cu128` (NOT cu130) | Only ~18G | Use `/snfs2` for datasets |
| `medusas_shr` / `gorgonas_dev` | Shared variants | Same as parent | Same | Same | Can run alongside other users |

Blackwell SM120 **requires** torch built for cu128 at minimum. `cu130` wheels fail with driver 570 because driver 570 only exposes CUDA 12.8. `cu124` wheels do not support SM120 at all.

## Filesystem Tiers (Speed-Ranked)

| Path | Type | Speed | Use For |
|------|------|-------|---------|
| `/tmp` (gorgonas) | Local NVMe | Fastest | Per-job dataset cache |
| `/snfs2` | BeeGFS shared | ~10× faster than sonic_home | Venvs, datasets shared across medusas |
| `/sonic_home` | NFS | Slow writes (~0.1 MB/s) | Code only, never datasets or venvs |

**Never** `uv sync` or unpack a wheel into `/sonic_home` — NFS writes will stall a 500MB torch wheel for 20+ minutes. Pre-build venvs on `/snfs2` or on a fast local disk and rsync/scp them into place.

## Venv Strategy

- One venv per GPU architecture. Select at job start via `hostname | grep medusa`.
- On medusas, point `VENV_DIR` at `/snfs2/<user>/.venv-cu128` (absolute path, pre-built).
- On gorgonas, use a relative `.venv` inside the repo root.
- **Never** run training through `uv run` on SLURM. `uv` detects the Python patch-version mismatch between the module system (`python/3.12.1` → `3.12.1`) and the cached environment (e.g. `3.12.3`/`3.12.13`), then deletes and recreates the venv on whichever filesystem it lives on. On NFS this is catastrophic.
- Invoke `"$VENV_DIR/bin/torchrun"` directly and set `export PYTHONPATH=$REPO_ROOT` since the venv is not editable-installed.
- Shebangs inside a relocated venv must point at its deployment path (`sed -i "1 s|^#!.*python.*|#!$VENV_DIR/bin/python3.12|"`). The python binary itself must be a symlink to the module-supplied interpreter (`/opt/python/3-12-1/bin/python3.12`).

## Launch Pattern (sbatch via phocus4)

1. `rsync` code to `/sonic_home/<user>/<repo>` with excludes for every venv (`.venv/`, `.venv-cu128/`, `.venv-cu130/`), `wandb/`, results, HDF5 files, `.git/`, `pyproject.toml`, `uv.lock`.
2. Generate an sbatch script on the remote: load `python/3.12.1` and `cuda/12.8.0`, export `HOME=/sonic_home/<user>`, set `WANDB_*`, `PYTHONPATH`, select `VENV_DIR` by hostname, build `LD_LIBRARY_PATH` from the venv's bundled NVIDIA libs (`find "$VENV_DIR" -path '*/nvidia/*/lib' -type d`).
3. Copy train/test HDF5 to the chosen dataset cache before `torchrun` — never let the dataloader read a multi-GB HDF5 directly from `/sonic_home` or `/snfs2` if `/tmp` fits it.
4. Submit: `ssh phocus4 "export SLURM_CONF=... && sbatch <script>"` and parse `Submitted batch job <ID>`.
5. Persist a manifest with `trial_idx`, `wandb_name`, `slurm_job_id`, `partition`, `node`, `started_at`.

## Dataset Staging Rules

- Parse **both** `train_hdf5` and `test_hdf5` from the config. Datasets that are not PTB-XL (code15, codetest, etc.) use a separate `test_hdf5`; PTB-XL uses `strat_fold` inside a single HDF5.
- Check `DST_SIZE > 0` to decide if the file already exists at the local cache; do not compare against `SRC_SIZE` from a location that may not hold the file (the pre-staged copy on `/snfs2` often has no sonic_home counterpart).
- Rewrite the yaml config in-place to absolute paths before training: `sed "s|train_hdf5:.*|train_hdf5: $DATASET_LOCAL/$HDF5_NAME|"` and the equivalent for `test_hdf5`.
- code15 train HDF5 is ~86G. On medusas `/tmp` cannot hold it; stage on `/snfs2` but expect DataLoader timeout unless `num_workers` is raised and `dataloader_timeout` is extended.

## Monitoring

Use a poll loop over `ssh phocus4 "grep ..."` of the log. Grep filter must be strict — `early_stop` appears inside the config dump and will falsely trigger "finished" detection. Match on the actual run-end markers:

```
grep -E '(Epoch [0-9]+/80|│ Val |│ Test |Training complete|EARLY STOP|exit=)'
```

Poll every 60 seconds, track the last emitted epoch, and emit only on increase. Treat `Training complete` or `EARLY STOP` as the termination signal; `exit=` appears at the end of the sbatch body.

## Error Catalog

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| `Invalid user: <name>` on `squeue` | Missing `SLURM_CONF` export | `export SLURM_CONF=/usr/local/slurm/etc/slurm.conf` |
| `DataLoader timed out after 120 seconds` | HDF5 read from slow shared FS | Stage to `/tmp` if space permits, else raise `dataloader_timeout`, raise `num_workers`, reduce `batch_size` |
| `Data leakage detected: split overlaps are non-empty` | Non-PTB-XL dataset needs `test_hdf5` | Add `test_hdf5:` to config; ensure launcher rewrites both paths |
| `No module named '<pkg>'` | `torchrun` invoked without editable install | `export PYTHONPATH=$REPO_ROOT` |
| `CUDA error: no kernel image` on medusa | torch built for wrong CUDA (cu130 with driver 570) | Rebuild venv with `cu128` wheels |
| sbatch hangs in `uv` phase on NFS | `uv run` is recreating the venv | Switch to direct `$VENV_DIR/bin/torchrun`; ensure `pyproject.toml` is excluded from rsync |
| Shebang `bad interpreter` | Venv copied from a different path | `sed` all bin/* shebangs to the new absolute path |
| `/tmp` out of space on medusa | 18G cap | Stage datasets on `/snfs2`, not `/tmp`, on medusas |

## Quick Reference

```bash
# Availability check (always first)
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sinfo -N -l && squeue -u <user>"

# Cancel a job
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && scancel <JOBID>"

# Tail a log
ssh phocus4 "tail -f /sonic_home/<user>/<repo>/autoresearch_runs/logs/<name>_<JOBID>.log"

# Inspect a job's assigned resources
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && scontrol show job <JOBID>"
```

## Red Flags — Stop and Rethink

- About to run `uv sync` or `uv run` inside an sbatch script → **use the venv directly**
- About to copy a venv or dataset into `/sonic_home` → **use `/snfs2`**
- About to submit without running `sinfo` → **check availability first**
- Using `cu130` wheels for medusas → **cu128 is the maximum supported**
- Grep filter for completion contains bare `early_stop` → **use `EARLY STOP` (uppercase, with space) to avoid config-dump false positives**
- Multiple user jobs already running and submitting more → **confirm policy allows it**
