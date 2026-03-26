# AFM Tip-Cell Segmentation Pipeline

Fine-tuning Cellpose on AFM T-cell images with probe-aware tip-cell selection and Dice/IoU evaluation across 8 experimental subsets.

## What this is

This notebook covers the full model development workflow for automated AFM tip-cell segmentation:
- Fine-tuning a Cellpose model on labeled AFM T-cell images
- Running batch inference with automatic cantilever tip detection and probe-aware cell selection
- Evaluating performance using closest-to-probe Dice/IoU across 216 frames (DN1–DN4 × rapid/rate)

**Ground truth masks** used for training were generated separately using [afm-annotation-pipeline](https://github.com/Jeremy-Kwok/afm-annotation-pipeline).

## Results

| Metric | Value |
|---|---|
| Global mean Dice | 0.889 |
| Global mean IoU | 0.813 |
| Pred-empty frames | 0 / 216 |

Per-subset performance ranges from Dice 0.816 (DN2-rate) to 0.913 (DN1-rapid, DN2-rapid, DN4-rate).

## Notebook structure

| Section | Cells | Description |
|---|---|---|
| Setup | 1–6 | Install, mount Drive, stage data, sanity check, train/test split, preprocessing helpers |
| Training | 7–9 | Backup, fine-tune Cellpose, optional resume |
| Inference | 10–11 | Smoke test, full batch inference with probe-aware selection and fallback paths |
| Evaluation | 12–14 | Closest-to-probe metrics, subgroup breakdown, optional best-instance comparison |
| Debug tools | D0–D8 | Failure diagnosis, re-inference on failing stems, single-stem selector diagnostics |
| Appendix | — | Pipeline diagram and qualitative examples |

## Key design decisions

**Probe-aware selection:** The model outputs candidate cells across the full frame. A separate geometry rule — right-of-probe filter, vertical corridor, weighted distance — selects the tip-adjacent cell. This eliminated wrong-cell picks that baseline largest-cell selection failed on.

**Closest-to-probe evaluation:** Metrics are computed using the same selection rule used at inference, not best-instance matching. This gives an honest measure of end-to-end pipeline performance.

**Structured fallback paths:** If the main inference pass produces no usable mask, the pipeline retries with softer thresholds, then falls back to a local crop. Every frame's path (normal, retry, fallback) is logged to JSON. Zero pred-empty frames across all 216 test frames.

## Requirements

- Google Colab with GPU runtime (T4 or better)
- Google Drive with AFM dataset mounted at `data/afm_dataset`
- Ground truth masks from [afm-annotation-pipeline](https://github.com/Jeremy-Kwok/afm-annotation-pipeline)

## Acknowledgments

AFM images and ground truth annotations provided by Hoseyn Amiri, graduate researcher at the [Sulchek Lab](https://www.sulchek2.gatech.edu/), Georgia Tech.

## Author

Jeremy Kwok — Georgia Tech Mechanical Engineering (CS minor)