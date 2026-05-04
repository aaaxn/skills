---
name: slurm-status
description: Quick SONIC SLURM cluster status check — shows GPU availability, running jobs, node states, and recommends where to submit next. Use before any job submission.
---

# SONIC Cluster Status Check

Run these commands **sequentially** (each requires SLURM_CONF) via `ssh phocus4`. Present the results in a unified report.

## Step 1: GPU Partition Overview

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sinfo -p gorgonas,gorgonas_dev,medusas,medusas_shr -o '%12P %8a %6D %6t %12G %10m %15N'"
```

## Step 2: Per-Node Detailed State

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sinfo -N -l -p gorgonas,gorgonas_dev,medusas,medusas_shr"
```

## Step 3: Who Is Using GPUs Right Now

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && squeue -p gorgonas,gorgonas_dev,medusas,medusas_shr -o '%10i %12u %12P %8T %10M %6D %R'"
```

## Step 4: User's Current Jobs (All Partitions)

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && squeue -u arturxavier -o '%10i %30j %12P %8T %12M %12l %6D %R'"
```

## Step 5: Pending Jobs

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && squeue -p gorgonas,gorgonas_dev,medusas,medusas_shr -t PENDING -o '%10i %12u %12P %10Q %R'"
```

## Report Format

Present a single structured report:

### Cluster Status

| Partition | Nodes Idle | Nodes Alloc | GPUs Available | GPU Type |
|-----------|-----------|-------------|----------------|----------|
| gorgonas | ... | ... | ... | RTX 3090 Ti |
| gorgonas_dev | ... | ... | ... | RTX 3090 Ti |
| medusas | ... | ... | ... | RTX 5090 |
| medusas_shr | ... | ... | ... | RTX 5090 |

### My Running Jobs
(table or "None")

### Recommendation
Based on availability, recommend:
- **Best partition right now**: (whichever has idle nodes)
- **Dataset staging**: `/tmp` for gorgonas (200G NVMe), `/snfs2` for medusas (18G /tmp limit)
- **Warnings**: If user already has 2+ jobs running, note the soft limit

## Important Reminders
- Every `ssh phocus4` command needs `export SLURM_CONF=/usr/local/slurm/etc/slurm.conf`
- Gorgonas = RTX 3090 Ti, cu124, big /tmp
- Medusas = RTX 5090 Blackwell, cu128 ONLY (not cu130), tiny /tmp → use /snfs2
- /snfs2 is ~10x faster than /sonic_home for I/O
- Never run heavy compute directly on phocus4 (it's the login node)
