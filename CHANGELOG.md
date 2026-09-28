# Changelog

## v0.1.0 (2026-09-28)

First public results, measured 2026-09-27.

- Q1: for the same content, Korean loses 1.47–3.45× more information than English under Q4_K_M/Q3_K_M quantization, relative to Q8_0 (per-document KL, 6/6 combinations).
- Per-token sensitivity differs by model: none for kanana-1.5-2.1b-instruct (intervals include 1.0), 1.42–2.04× for Qwen3-1.7B and Qwen3-4B.
- Q2: paired multiple-choice flip gap (KO−EN) is +2.8 to +18.5 points. Only Qwen3-1.7B's interval excludes 0.
- Q3: Q4_K_M generates fastest for all three models.
- Files: `kl.csv`, `mc.csv`, `speed.csv`, `models.csv`, `inputs.json`, `summary.json`, `run-*.json`, `SHA256SUMS`. Release asset: `koqloss-data-v0.1.0.json` (all tables in one file) and its `.sha256`. No weights, logits or FLORES-101 text.
