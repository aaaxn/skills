---
name: slurm-job
description: Create or modify sbatch scripts for the SONIC SLURM cluster. Handles partition selection, CUDA/venv setup, dataset staging, and proper resource allocation for gorgonas (RTX 3090 Ti) and medusas (RTX 5090 Blackwell) nodes.
---

# SONIC SLURM Job Script Builder

## Step 1: Gather Requirements

Ask the user (or infer from context):
- **Job type**: training, evaluation, inference, preprocessing
- **GPU preference**: gorgonas (RTX 3090 Ti, cu124) or medusas (RTX 5090, cu128)
- **Wall time**: default 24:00:00
- **Dataset(s)**: which HDF5 files, sizes, staging needs
- **Job name**: descriptive slug (lowercase, hyphens)

## Step 2: Select Resources

| Parameter | Gorgonas | Medusas |
|-----------|----------|---------|
| Partition | `gorgonas` (exclusive) or `gorgonas_dev` (shared) | `medusas` (exclusive) or `medusas_shr` (shared) |
| GPU | RTX 3090 Ti | RTX 5090 (Blackwell SM120) |
| CUDA module | `cuda/12.4.0` | `cuda/12.8.0` |
| Torch wheels | `cu124` | `cu128` (NOT cu130) |
| Dataset staging | `/tmp` (200G+ NVMe, fastest) | `/snfs2` (BeeGFS, 10x faster than sonic_home) |
| Venv location | `.venv` in repo root (local disk ok) | `/snfs2/<user>/.venv-cu128` |

## Step 3: Generate Script

Use this template, filling in the specifics:

```bash
#!/bin/bash
#SBATCH --job-name=<JOB_NAME>
#SBATCH --partition=<PARTITION>
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --gres=gpu:1
#SBATCH --time=<WALLTIME>
#SBATCH --output=/sonic_home/<user>/<repo>/autoresearch_runs/logs/<name>_%j.log
#SBATCH --error=/sonic_home/<user>/<repo>/autoresearch_runs/logs/<name>_%j.log

set -euo pipefail

# --- Environment ---
export SLURM_CONF=/usr/local/slurm/etc/slurm.conf
export HOME=/sonic_home/<user>
module load python/3.12.1

# --- GPU Architecture Detection ---
if hostname | grep -q medusa; then
    module load cuda/12.8.0
    VENV_DIR="/snfs2/<user>/.venv-cu128"
    DATASET_CACHE="/snfs2/<user>/datasets"
else
    module load cuda/12.4.0
    VENV_DIR="/sonic_home/<user>/<repo>/.venv"
    DATASET_CACHE="/tmp"
fi

# --- Venv Activation (direct, no uv) ---
export PYTHONPATH=/sonic_home/<user>/<repo>
export PATH="$VENV_DIR/bin:$PATH"

# Build LD_LIBRARY_PATH from bundled NVIDIA libs
LD_PATHS=$(find "$VENV_DIR" -path '*/nvidia/*/lib' -type d | tr '\n' ':')
export LD_LIBRARY_PATH="${LD_PATHS}${LD_LIBRARY_PATH:-}"

# --- W&B ---
export WANDB_PROJECT="<project>"
export WANDB_RUN_NAME="<name>-${SLURM_JOB_ID}"
export WANDB_DIR="/sonic_home/<user>/<repo>"

# --- Dataset Staging ---
REPO_ROOT="/sonic_home/<user>/<repo>"
# Copy HDF5 to fast storage if not already there
TRAIN_HDF5="<train_hdf5_filename>"
if [ ! -f "$DATASET_CACHE/$TRAIN_HDF5" ]; then
    echo "Staging $TRAIN_HDF5 to $DATASET_CACHE..."
    cp "$REPO_ROOT/data/$TRAIN_HDF5" "$DATASET_CACHE/$TRAIN_HDF5"
fi

# --- Training ---
"$VENV_DIR/bin/torchrun" \
    --nproc_per_node=1 \
    --master_port=$((29500 + SLURM_JOB_ID % 1000)) \
    <training_script> \
    --config <config_path> \
    --train_hdf5 "$DATASET_CACHE/$TRAIN_HDF5"

echo "exit=0"
```

## Critical Rules

1. **NEVER** use `uv run` or `uv sync` in sbatch scripts — uv will detect Python version mismatches and destroy the venv
2. **NEVER** stage datasets to `/sonic_home` — NFS writes at ~0.1 MB/s will stall
3. **NEVER** use cu130 wheels on medusas — driver 570 only exposes CUDA 12.8
4. **ALWAYS** export `SLURM_CONF=/usr/local/slurm/etc/slurm.conf`
5. **ALWAYS** export `PYTHONPATH=$REPO_ROOT` since venvs are not editable-installed
6. **ALWAYS** set unique `--master_port` for torchrun to avoid collisions
7. **ALWAYS** build LD_LIBRARY_PATH from the venv's bundled NVIDIA libs

## Step 4: Submission

After writing the script:

```bash
# Rsync code (exclude heavy/generated files)
rsync -az --exclude='.venv*' --exclude='wandb/' --exclude='*.hdf5' \
    --exclude='.git/' --exclude='pyproject.toml' --exclude='uv.lock' \
    ./ phocus4:/sonic_home/<user>/<repo>/

# Submit
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sbatch /sonic_home/<user>/<repo>/<script>.sh"
```

Parse the output for `Submitted batch job <ID>` and record the job ID.

## Step 5: Verify

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && squeue -j <JOBID>"
```
