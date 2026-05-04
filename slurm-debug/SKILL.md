---
name: slurm-debug
description: Diagnose failed, killed, timed-out, or stuck SLURM jobs on the SONIC cluster. Covers OOM, CUDA errors, DataLoader timeouts, venv corruption, driver mismatches, and pending-forever jobs.
---

# SONIC SLURM Job Debugger

## Step 1: Identify the Job

If a job ID is provided, use it directly. Otherwise, find recent failures:

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sacct -u arturxavier --starttime=now-7days --format=JobID,JobName%30,Partition,State,ExitCode,Elapsed,MaxRSS,NodeList -X"
```

Focus on jobs with state: FAILED, OUT_OF_MEMORY, TIMEOUT, CANCELLED, NODE_FAIL.

## Step 2: Get Job Details

```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && scontrol show job <JOBID>"
```

If already completed (scontrol won't have it):
```bash
ssh phocus4 "export SLURM_CONF=/usr/local/slurm/etc/slurm.conf && sacct -j <JOBID> --format=JobID,JobName%30,Partition,State,ExitCode,Elapsed,MaxRSS,MaxVMSize,NodeList,Submit,Start,End,AllocTRES%60"
```

## Step 3: Read Logs

Find log paths (usually follows pattern):
```bash
ssh phocus4 "ls -la /sonic_home/arturxavier/*/autoresearch_runs/logs/*_<JOBID>.log 2>/dev/null || ls -la /sonic_home/arturxavier/*/logs/*<JOBID>* 2>/dev/null"
```

Read the end (crash usually at bottom):
```bash
ssh phocus4 "tail -80 <LOG_PATH>"
```

Read the beginning (early config/import errors):
```bash
ssh phocus4 "head -30 <LOG_PATH>"
```

## Step 4: Diagnose by Pattern

### Exit Codes
| Exit Code | Signal | Meaning |
|-----------|--------|---------|
| 0:0 | - | Success |
| 1:0 | - | Python exception / script error |
| 137:0 or 0:9 | SIGKILL | OOM killed by kernel or SLURM |
| 139:0 or 0:11 | SIGSEGV | Segfault (corrupted memory, bad CUDA) |
| 2:0 | - | Python KeyboardInterrupt / scancel |

### SONIC-Specific Error Patterns

| Log Pattern | Root Cause | Fix |
|------------|-----------|-----|
| `CUDA error: no kernel image is available` | cu130 wheels on medusa (driver 570 = CUDA 12.8 max) | Rebuild venv with cu128 wheels |
| `CUDA error: no kernel image` on gorgona | cu128 wheels missing sm86 support | Use cu124 wheels for gorgonas |
| `DataLoader timed out after 120 seconds` | Reading HDF5 from slow NFS/BeeGFS | Stage to /tmp (gorgona) or /snfs2 (medusa); raise `dataloader_timeout`; increase `num_workers` |
| `Data leakage detected: split overlaps` | Non-PTB-XL dataset missing `test_hdf5` | Add separate `test_hdf5` path to config |
| `No module named '<pkg>'` | Missing PYTHONPATH | Add `export PYTHONPATH=$REPO_ROOT` |
| `bad interpreter` / shebang error | Venv moved without fixing shebangs | `sed -i "1 s\|^#!.*python.*\|#!$VENV_DIR/bin/python3.12\|"` on all bin/* |
| `Invalid user` from squeue/sinfo | Missing SLURM_CONF | Add `export SLURM_CONF=/usr/local/slurm/etc/slurm.conf` |
| `RuntimeError: Address already in use` | torchrun port collision | Use `--master_port=$((29500 + SLURM_JOB_ID % 1000))` |
| `Killed` (bare, in output) | Kernel OOM | Reduce batch size, or request more memory |
| `oom-kill` in dmesg / slurmstepd | SLURM memory limit exceeded | Increase `--mem` or reduce batch size |
| `uv` messages about recreating venv | uv detected Python version mismatch | NEVER use uv in sbatch; call venv binaries directly |
| `No space left on device` on medusa | /tmp only 18G | Move staging to /snfs2 |
| `NCCL error` / `NCCL timeout` | Multi-GPU communication failure | Check if nodes have working IB; try `NCCL_DEBUG=INFO` |

### Pending Job Diagnosis

If job is stuck PENDING, check the Reason field from `scontrol show job`:

| Reason | Meaning | Action |
|--------|---------|--------|
| Resources | Waiting for nodes to free up | Wait, or try a different partition |
| Priority | Lower priority than other jobs | Wait |
| ReqNodeNotAvail | Requested node is down | Remove node constraint or pick another |
| PartitionNodeLimit | Too many nodes requested | Reduce --nodes |
| InvalidAccount | Account/partition mismatch | Check partition name |

## Step 5: Report

Format your diagnosis as:

### What Happened
One sentence: "Job <ID> (<name>) failed with <STATE> after <ELAPSED> on <NODE>."

### Evidence
Quote 2-3 relevant log lines that confirm the diagnosis.

### How to Fix
Specific, actionable steps with exact values/commands. Reference the correct filesystem tier:
- Gorgonas: stage to `/tmp` (200G+ NVMe)
- Medusas: stage to `/snfs2` (BeeGFS, ~10x faster than sonic_home)
- NEVER suggest staging to `/sonic_home`

### Resubmit Command
Provide the exact fixed sbatch command or the specific changes to make to the script.

## Red Flags — SONIC-Specific

- If suggesting cu130 → STOP. Medusas max is cu128.
- If suggesting dataset staging to /sonic_home → STOP. Use /tmp or /snfs2.
- If suggesting `uv run` or `uv sync` in sbatch → STOP. Call venv directly.
- If fix involves running heavy computation on phocus4 → STOP. That's the login node.
