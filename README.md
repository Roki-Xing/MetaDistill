# MetaDistill

This repository contains the code, fixed evaluation assets, and checkpoints needed to reproduce:

- BBOB evaluations for four optimizers: LES, LDE, LGA, POM (with optional SSFT).
- Neuroevolution tasks: robot tasks (tasks 3/4/5/6, gen=100) and LunarLander (task 2, gen=500).
- The MetaDistill training path from teacher trajectory collection to distilled checkpoints.

Not included:

- Any experimental outputs (plots, logs, JSON summaries). Running the scripts will generate new outputs locally.

Large evaluation assets are intended to be tracked with Git LFS:

```bash
git lfs install
git lfs track "metadistill/offsets/*.pkl"
git lfs track "metadistill/checkpoints/**/*.pt"
git lfs track "metadistill/checkpoints/**/*.pth"
```

## Quick start

```bash
cd metadistill
python3 -m pip install -r requirements.txt
```

## Reproduce From Scratch

The evaluation scripts can use the included checkpoints directly. To regenerate distilled checkpoints from scratch, first collect and select teacher trajectories:

```bash
python3 scripts/collect_trajectories.py \
  --epochs 16 \
  --generations 100 \
  --interval 10 \
  --popsize 200 \
  --dim 10
```

This writes:

- `data/teacher/teacher_cec2-6_raw_d10_pop200_gen100_7t_e16.pkl`
- `data/teacher/teacher_cec2-6_sel_itvl10_d10_pop200_gen100_7t_e16.pkl`

Then train a distilled optimizer:

```bash
python3 scripts/train_distill.py --config configs/distill_les.json
python3 scripts/train_distill.py --config configs/distill_lde.json
python3 scripts/train_distill.py --config configs/distill_lga.json
python3 scripts/train_distill.py --config configs/distill_pom.json
```

Generated checkpoints are written under `checkpoints/generated/`.

Baseline checkpoints can also be regenerated from scratch:

```bash
python3 scripts/train_lde_pg.py \
  --config configs/lde_pg_pop200_d10_sample_h64.json \
  --function-set cec \
  --fids 2 3 4 5 6 \
  --ckpt-dir checkpoints/generated/lde_policy_gradient

python3 scripts/train_metabbo_les.py --outdir checkpoints/generated/les_metabbo
python3 scripts/train_metabbo_lga.py --outdir checkpoints/generated/lga_metabbo
```

## Fixed Offsets

`offsets/` contains the fixed BBOB offsets used for the reported evaluation protocol. These files are part of the protocol assets and should not be regenerated when reproducing the reported BBOB numbers. See `metadistill/offsets/MANIFEST.json`.

The included `.pkl` files are stored in CPU-safe form. If offsets are regenerated on a GPU machine, rewrite them before sharing:

```bash
python3 tools/convert_offsets_to_cpu.py --offsets-dir offsets
```

### BBOB (four optimizers, one seed window)

```bash
bash scripts/repro_bbob_4alg_seedwindow.sh bbob_seedwin_0to6 0
```

Outputs:

- `images/<RUN_ID>/...`
- `artifacts/eval_summaries/<RUN_ID>/...`
- `logs/<RUN_ID>/...`

### NeuroEvo robot tasks (gen=100)

```bash
bash scripts/repro_neuroevo_robot_tasks_gen100.sh
```

### NeuroEvo LunarLander (gen=500)

```bash
bash scripts/repro_neuroevo_lunarlander_gen500.sh
```

## Checkpoints / variants

See `docs/MODEL_VARIANTS.md`.
