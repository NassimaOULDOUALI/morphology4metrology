# Branch `aikon-api`

This branch is the version of morphology4metrology (M4M) used by the AIKON platform API
(`aikon-api`, module `app/paleography`, git submodule `app/paleography/lib/m4m`).

It starts from `main` of `raphael-baena/morphology4metrology` and adds the commits below.
The upstream `README.md` is unchanged.

## Changes with respect to upstream `main`

| Commit | File | Change | Why |
|---|---|---|---|
| `12df923` | `datasets/dataset.py` | Non-breaking spaces (U+00A0) replaced by regular spaces in `convert_str_to_tensor` | With them, Python could not parse the file |
| `0b55a90` | `.gitignore` | `weights/` | The step-0 checkpoint is stored outside the repository |
| `64b6d8b` | `reconstruction.py` | Epoch loop `range(start_epoch, max_e)` instead of `range(start_epoch, max_e + 1)`; progress-bar total adjusted | `--max_e N` ran N + 1 epochs, and the final sprite and reconstruction saves (`e == max_e - 1`) never hit the last epoch. Now N epochs, and the final saves happen at the last one |
| `55844e5` | `reconstruction.py` | Optional `--seed` (default `None`): seeds `random`, `numpy`, `torch` and `torch.cuda` | No seed was set. Note: with a seed, runs are still not bitwise identical (likely non-deterministic CUDA ops) |
| `6c773bd` | `predict.py` | Fall back on `datasets/config.json` when `./config.json` is absent | Superseded by the next commit, kept for history |
| `3c7d54d` | `predict.py` | Restore the version that precedes upstream `b0e0cf3` | See below |
| `1d80d21` | `.gitignore` | `/logs_reconstruction` instead of `logs_reconstruction/` | The API replaces `logs_reconstruction` with a link to its data folder; the new pattern also ignores a link |

## Not included: upstream `predict.py` from `b0e0cf3`

Upstream commit `b0e0cf3` (14/10/2026) rewrites `predict.py`. With it, `export_paleography_input.py`
no longer finds what it reads: `sprite_final` is empty, there is no per-folio folder in
`<step 2>/predict`, and no `global_summary.json`. The export then writes no measurement JSON and
an empty `baseline_with_ar`, while exiting with code 0. The new version also reads a
`config.json` at the repository root instead of `datasets/config.json`.

This branch therefore keeps the previous `predict.py`. Adapting the export to the new
`predict.py` is an open decision.

## How the API calls M4M

The API runs each step as a subprocess from the repository root, with its own interpreter:

```
step 0  reconstruction.py --step 0 --init --model_checkpoint_path <checkpoint> ...
step 1  reconstruction.py --step 1 --model_checkpoint_path <s0>/model.pth --reconstructor_path <s0>/reconstructor.pth ...
step 2  reconstruction.py --step 2 --documents --mask_sprite --prototypes_only_path <s1>/reconstructor_unfrozen.pth
                          --model_checkpoint_path <s1>/model.pth --annotation_file <annotation.json> ...
export  export_paleography_input.py --step1_dir <s1> --finetune_dir <s2> ...
```

Common arguments: `--dataset_file dataset --space_index 0 --model_config_path config/Latin_accent.py
--num_fine_classes 2 --loss L1 --tag <run>_s<step> --max_e <N> --batch_size <B> --seed <seed>`.

The API writes `datasets/config.json` (ignored by git) and replaces `logs_reconstruction` with a
link to its own data folder.

## Keeping the branch up to date

```bash
git fetch origin
git log --oneline aikon-api..origin/main      # upstream commits not in this branch
# rebase or merge, then run the API end-to-end test before pushing
git push fork aikon-api
# in aikon-api: pin the submodule on the new commit
git -C app/paleography/lib/m4m fetch && git -C app/paleography/lib/m4m checkout <sha>
git add app/paleography/lib/m4m && git commit -m "[paleography] bump M4M submodule"
```
