# MetaDistill

Official code and evaluation assets for **MetaDistill: Unlocking the Performance Ceiling for Pretrained Optimizers**.

This repository supports:

- BBOB evaluations for four learnable optimizers: LES, LDE, LGA, and POM, with optional SSFT variants.
- Control-task evaluations on LunarLander, BipedalWalker, and Acrobot.
- Supporting assets for task-diversity and teacher-diversity analysis.

The repository also includes pretrained checkpoints, fixed BBOB offsets, and scripts for regenerating teacher trajectories and distilled checkpoints.

Generated outputs such as plots, logs, and JSON summaries are not included. Running the evaluation scripts will create them locally.

## Repository Layout

```text
metadistill/
  checkpoints/        # baseline and MetaDistill checkpoints
  configs/            # optimizer and training configs
  meta_trainers/      # training logic for distillation and SSFT
  offsets/            # fixed BBOB offsets used in the paper
  optimizers/         # learnable and classical optimizer implementations
  scripts/            # evaluation and training entry points
  tasks/              # BBOB, CEC, EF, and control-task definitions
  tools/              # utility scripts
```

All commands below are run from the `metadistill/` directory unless otherwise stated.

## Setup

Clone the repository and fetch Git LFS assets:

```bash
git clone https://github.com/Roki-Xing/MetaDistill.git
cd MetaDistill

git lfs install
git lfs pull
```

Install Python dependencies:

```bash
cd metadistill
python3 -m pip install -r requirements.txt
```

The checkpoint and offset files are tracked with Git LFS. If any `.pt`, `.pth`, or `.pkl` file appears as a small text pointer instead of a binary file, run `git lfs pull` again from the repository root.

## Reproduce BBOB Evaluations

To reproduce one 7-seed window on BBOB:

```bash
bash scripts/repro_bbob_4alg_seedwindow.sh bbob_seedwin_0to6 0
```

To reproduce the three overlapping 7-seed windows used in the paper:

```bash
bash scripts/repro_bbob_4alg_seedwindow.sh bbob_seedwin_0to6 0
bash scripts/repro_bbob_4alg_seedwindow.sh bbob_seedwin_1to7 1
bash scripts/repro_bbob_4alg_seedwindow.sh bbob_seedwin_2to8 2
```

Outputs are written to:

```text
images/<RUN_ID>/
artifacts/eval_summaries/<RUN_ID>/
logs/<RUN_ID>/
```

The script evaluates dimensions `30`, `100`, `200`, `300`, and `500`. The main paper reports `30` to `300`; `d=500` is included as an extended evaluation.

## Reproduce Control-Task Evaluations

We evaluate MetaDistill on the three Gymnasium control tasks used in the paper: LunarLander, BipedalWalker, and Acrobot.

```bash
bash scripts/repro_control_lunarlander.sh
bash scripts/repro_control_bipedalwalker_acrobot.sh
```

Outputs are written to:

```text
images/<RUN_ID>/
artifacts/eval_summaries/<RUN_ID>/
logs/<RUN_ID>/
```

## Fixed BBOB Offsets

The directory `offsets/` contains the fixed BBOB offsets used in the paper. These files are part of the evaluation protocol and should not be regenerated when reproducing the reported BBOB numbers.

See:

```text
offsets/MANIFEST.json
```

The included offset files are stored in CPU-safe form. If offsets are regenerated on a GPU machine, rewrite them before sharing:

```bash
python3 tools/convert_offsets_to_cpu.py --offsets-dir offsets
```

## Checkpoints and Variants

Checkpoint paths and evaluation variants are documented in:

```text
../docs/MODEL_VARIANTS.md
checkpoints/MANIFEST.json
```

The main evaluation scripts compare each baseline checkpoint against the corresponding MetaDistill checkpoint with variants:

```text
md_j0, md_j1, md_j3, md_j5
```

Here, `md_j0` denotes MetaDistill without SSFT, while `md_j1`, `md_j3`, and `md_j5` apply SSFT with the corresponding interval.

## Regenerate Distilled Checkpoints

The evaluation scripts can use the included checkpoints directly. To regenerate distilled checkpoints, first collect and select teacher trajectories:

```bash
python3 scripts/collect_trajectories.py \
  --epochs 16 \
  --generations 100 \
  --interval 10 \
  --popsize 200 \
  --dim 10
```

This writes:

```text
data/teacher/teacher_cec2-6_raw_d10_pop200_gen100_7t_e16.pkl
data/teacher/teacher_cec2-6_sel_itvl10_d10_pop200_gen100_7t_e16.pkl
```

With the default five training tasks, `--epochs 16` gives `16 x 5 = 80` selected teacher trajectories, matching the knowledge-base size used in the paper.

Then train distilled optimizers:

```bash
python3 scripts/train_distill.py --config configs/distill_les.json
python3 scripts/train_distill.py --config configs/distill_lde.json
python3 scripts/train_distill.py --config configs/distill_lga.json
python3 scripts/train_distill.py --config configs/distill_pom.json
```

Generated checkpoints are written under:

```text
checkpoints/generated/
```

## Regenerate Baseline Checkpoints

Baseline checkpoints can also be regenerated:

```bash
python3 scripts/train_lde_pg.py \
  --config configs/lde_pg_pop200_d10_sample_h64.json \
  --function-set cec \
  --fids 2 3 4 5 6 \
  --ckpt-dir checkpoints/generated/lde_policy_gradient

python3 scripts/train_metabbo_les.py --outdir checkpoints/generated/les_metabbo
python3 scripts/train_metabbo_lga.py --outdir checkpoints/generated/lga_metabbo
```

## Maintainer Notes for Git LFS

The following patterns should be tracked with Git LFS:

```bash
git lfs track "metadistill/offsets/*.pkl"
git lfs track "metadistill/checkpoints/**/*.pt"
git lfs track "metadistill/checkpoints/**/*.pth"
```

These patterns are already recorded in `.gitattributes`.
