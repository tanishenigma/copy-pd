# PD Multimodal Handwriting/Spiral Dataset: Dataset Card

Built by `scripts/build_dataset.py` (run `.venv/bin/python -I scripts/build_dataset.py`; `--skip-download` reuses raw data).
Label convention: `1` = Parkinson's disease (PD), `0` = healthy control (HC).
Full count tables are in `counts_images.csv` and `counts_signals.csv`.

## Files
- `images/scan/*.png`: scanned or exported drawings, grayscale, 256x256, aspect kept, white padding.
- `images/rendered/*.png`: x,y pen-down trajectories rendered from the tablet signals (256x256, y axis up).
- `images_manifest.csv` columns: `sample_id, path, source_dataset, task, label, subject_id, subject_group, original_path, modality (scan|rendered), split_hint, overlap_flag, note, src_md5`.
- `signals/*.parquet`: one file per sample, columns `t` (seconds), `x`, `y`, `pressure`, `pen_down`, plus source-specific extras.
- `signals_manifest.csv` columns: the same as the image manifest, with `modality=signal`, plus `n_samples` and `sampling`.
- `images_dropped_duplicates.csv`, `signals_skipped_duplicates.csv`, `subject_groups_merged.csv`: the deduplication audit trail.
- **Use `subject_group`, not `subject_id`, for train/test splits.** It merges subject IDs that share identical images (see the caveats below).

## Totals
- Images: 1637 (1296 scan + 341 rendered), 255 subject groups (179 PD, 76 HC).
- Signals: 1133, 141 subject groups (89 PD, 52 HC).

## Scanned images after deduplication (samples / subjects)
| source | task | HC | PD |
|---|---|---|---|
| handpd | spiral | 44 / 11 | 200 / 50 |
| handpd | meander | 44 / 11 | 204 / 51 |
| newhandpd | spiral | 140 / 35 | 104 / 26 |
| newhandpd | meander | 140 / 35 | 100 / 25 |
| newhandpd | circle | 35 / 35 | 31 / 31 |
| kmader_drawings | spiral | 51 / 13 | 51 / 15 |
| kmader_drawings | wave | 51 / 13 | 51 / 15 |
| uci395 | static_spiral | 0 | 25 / 25 |
| uci395 | dynamic_spiral | 0 | 25 / 25 |

## Signals (samples / subjects). Every signal with x,y also has a `<task>_rendered` image.
| source | task | HC | PD |
|---|---|---|---|
| uci395 | static_spiral | 15 / 15 | 61 / 61 |
| uci395 | dynamic_spiral | 15 / 15 | 57 / 57 |
| uci395 | circle (STCP) | 7 / 7 | 47 / 47 |
| uci358 (non-identical files only) | static + dynamic spiral | 5+5 / 5 | 0 |
| newhandpd | spiral, meander | 140 / 35 each | 124 / 31 each |
| newhandpd | circle, circle_air, diadochokinesis_a, diadochokinesis_b | 35 / 35 each | 31 / 31 each |
| pahaw_examples | spiral + handwriting_t2 to t14 | 65 / 2 | 64 / 2 |

## Per source

### UCI 395: Parkinson Disease Spiral Drawings Using Digitized Graphics Tablet
- Data: `hw_dataset/control` (15 HC), `hw_dataset/parkinson` (25 PD), `new_dataset/parkinson` (37 PD).
- Format: `X;Y;Z;Pressure;GripAngle;Timestamp;TestID`. TestID 0 is the static spiral, 1 the dynamic spiral, 2 the circular-motion (STCP) test. Each file is split into one sample per test.
- `t` is the timestamp in ms divided by 1000. Extra columns: `z`, `grip_angle`. `pen_down` means pressure > 0.
- `hw_drawings`: 25 static and 25 dynamic PNGs. **They are labelled PD by inference:** the count matches the 25 PWP and UCI describes them as patient drawings. They cannot be linked to the signal files, so they get their own subject IDs (`uci395_drawing_NN`).
- Labels come from the folder names.
- License: CC BY 4.0. Cite: Isenkul, Sakar, Kursun, "Improved spiral test using digitized graphics tablet for monitoring Parkinson's disease", ICEHTM 2014.
- Caveat: the STCP circle renders look staircase-like because the coordinates are coarse integers over a small area.

### UCI 358: Improved Spiral Test (same authors)
- Its 40 files (15 Healthy, 25 PWP) are the same cohort as UCI 395 `hw_dataset`.
- 35 files are byte-identical copies and were skipped (see `signals_skipped_duplicates.csv`).
- The other 5 Healthy files start with the same first sample as UCI395 C_0001, C_0002, C_0005, C_0006 and C_0007. They are kept, mapped to those subject IDs, and flagged in `note`.
- Its drawings are byte-identical to UCI 395's, so they were not added again.
- License: CC BY 4.0, same citation as UCI 395.

### HandPD (original, 2016): Spiral_HandPD and Meander_HandPD
- 92 subjects (18 HC, 74 PD), 4 spirals and 4 meanders each, from Botucatu Medical School (UNESP).
- Labels come from the `*Control` / `*Patients` folders. The subject is the leading number of the filename (`0044-4.jpg`).
- Cite: Pereira et al., "A new computer vision-based approach to aid the diagnosis of Parkinson's disease", Computer Methods and Programs in Biomedicine, 2016.
- License: none stated. Academic use, with citation.

### NewHandPD (2016)
- 35 HC and 31 PD subjects. Each subject has 4 spirals, 4 meanders and 1 circle as images, plus 12 BiSP smart-pen signals.
- Labels come from the archive (Healthy* or Patient*), checked against the H/P tag in the filename.
- Filename anomalies:
  - The HealthyCircle archive names its files `circA-P<n>.jpg`. They are treated as HC subject H<n>.
  - `HealthySignal/sigMea1-28.txt` has no H/P tag. It is treated as H28.
- Signals have 6 channels, assigned in the order described in the literature (Pereira et al., SIBGRAPI 2016): microphone, finger_grip, axial_pressure (stored as `pressure`), tilt_accel_x, tilt_accel_y, tilt_accel_z. Sampling is 1000 Hz, read from the header.
- **The pen gives no x,y position**, so `x` and `y` are NaN and these signals have no rendered images.
- Task mapping:
  - circA: circle on paper (the matching image exists).
  - circB: circle in the air.
  - sigDiaA/B: diadochokinesis.
- Cite: Pereira et al., "Deep learning-aided Parkinson's disease diagnosis from handwritten dynamics", SIBGRAPI 2016.

### Kaggle kmader/parkinsons-drawings (spiral + wave)
- Taken from the GitHub mirror `widushan/Parkinson-Disease-Detection-from-Spiral-and-Wave-Drawings` (`data/raw/repos/widushan`). The Kaggle CLI was not installed and needs a login.
- Filename `V<nn><H|P><E|O><rep>`: E = spiral, O = wave. The H/P tag was asserted against the folder label.
- The subject is `V<nn>` plus the H/P tag (28 subjects).
- `split_hint` holds the original train/test split. **That split leaks subjects**, so do not use it.
- Origin: Zham et al., "Distinguishing different stages of Parkinson's disease using composite index of speed and pen-pressure of sketching a spiral", Front. Neurol. 2017.

### PaHaW: only the 4 public example subjects
- Taken from `BDALab/handwriting-features/examples/data` (MIT license). There are 2 HC and 2 PD subjects; labels come from the folder names.
- SVC columns, per the bundled `info.json`: Y, X, timestamp, button, azimuth, altitude, pressure. Extras kept: `azimuth`, `altitude`, `button`. `pen_down` means button = 1.
- Task 1 is the Archimedean spiral. Tasks 2 to 14 are kept as `handwriting_tN`.
- The full PaHaW database (75 subjects) needs a signed license request to BDALab (https://bdalab.utko.feec.vutbr.cz/), so it was not downloaded.
- Cite: Drotar et al., "Evaluation of handwriting kinematics and pressure for differential diagnosis of Parkinson's disease", Artif. Intell. Med. 2016.

## Subject-overlap caveats (important)
- **HandPD and NewHandPD overlap heavily.** 288 NewHandPD/HandPD scans are byte-identical:
  - 30 HandPD subjects are re-shipped as NewHandPD subjects, for example handpd_0098 as newhandpd_H11 and handpd_0261 as newhandpd_P10.
  - Inside NewHandPD, some patient IDs share identical images, for example P1, P4 and P25, or P2 and P26.
  - The labels always agree.
- How this was handled:
  - Exact duplicate images were dropped, keeping the NewHandPD copy (list in `images_dropped_duplicates.csv`).
  - Linked IDs were merged into `subject_group` (`subject_groups_merged.csv`).
- 8 further near-duplicates (32x32 thumbnail correlation of at least 0.95) are flagged in `overlap_flag`, but not dropped.
- The `ID_PATIENT` column of NewSpiral.csv is inconsistent (for example, healthy subjects carry IDs that HandPD uses for patients), so it was not used.
- The remaining HandPD and NewHandPD subjects may still be the same people. Treat the two as one cohort when splitting.
- UCI 395 and UCI 358 are one cohort. The UCI drawings may be the same people as the UCI PD signal subjects.

## Failed or not obtained
- Full PaHaW: needs a license or login.
- Kaggle kmader via the Kaggle CLI: not installed and needs a login. The GitHub mirror was used instead.
- `sayakpaul/Parkinson-s-Disease-Classifier`: only the kmader wave images, which are redundant.
- `musaru/PD_PaHaW`: only derived feature CSVs, no raw data.
- `khareyash05/...`: only models and notebooks, no data.
- `IoannisKansizoglou/DrawingAwarePDdetection`: only figures.
- `ManasaGadepalli/...`: only a manifest of NewHandPD.
- All the NewHandPD sibling archives (Meander, Circle, Signal for both groups) downloaded fine.
