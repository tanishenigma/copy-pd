# pd-multimodal

Merged Parkinson's disease (PD) drawing and handwriting dataset: scanned drawings plus pen/tablet signals, built from public sources (HandPD, NewHandPD, UCI 395/358, kmader spiral/wave, PaHaW examples).

## Build

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install numpy pandas pillow pyarrow matplotlib
python scripts/build_dataset.py
```

Downloads go to `data/raw/` and outputs to `data/processed/` (both ignored by git).

## Contents

- `scripts/build_dataset.py`: download, normalize, render, dedupe, label.
- `data/processed/images_manifest.csv`: 1637 images (1296 scans, 341 rendered from signals).
- `data/processed/signals_manifest.csv`: 1133 pen/tablet signals.
- `data/processed/DATASET_CARD.md`: per-source counts, licenses, caveats.

Split train/test by `subject_group`, not `subject_id`: HandPD and NewHandPD share subjects.
