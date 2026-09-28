# Changelog

## v0.2.0 (2026-09-28)

- New model: skt/A.X-4.0-Light (GGUF from mykor/A.X-4.0-Light-gguf), measured with the same inputs (checked by SHA-256), settings and llama.cpp build.
- Pre-registered test of "Korean-focused models have no per-token gap": **rejected**. A.X-4.0-Light's per-token KO/EN ratio is 1.32 (1.17–1.50) at Q4_K_M and 1.17 (1.05–1.33) at Q3_K_M.
- Headline updated: Korean loses 1.09–3.45× more than English for the same content; the 95% interval is above 1.0 in 7 of 8 combinations (was 6/6). A.X-4.0-Light has the smallest user-facing gap because its tokenizer needs 0.93× as many Korean tokens.
- New `data/cost.csv`: measurement time and file size per combination (download time for A.X-4.0-Light). `data/inputs.json` now also lists the SHA-256 of the multiple-choice input binaries.
- Issue-form labels are now `topic:request`, `topic:question` and `topic:correction`; the release workflow creates them.
- Release asset: `koqloss-data-v0.2.0.json` and its `.sha256`.

## v0.1.2 (2026-09-28)

Repository additions only. Results and data files are unchanged.

- Issue forms: measurement request (requester type, target, optional budget range), question, data correction. Every form asks for no confidential information, personal data or contact details. Requests carry no promise of a reply, price or schedule.
- `CITATION.cff`.
- Release asset: `koqloss-data-v0.1.2.json` and its `.sha256`.

## v0.1.1 (2026-09-28)

Wording fixes only. The numbers and data files are unchanged. Only the release bundle name changes.

- kanana-1.5-2.1b-instruct per-token result is now stated with its interval: no detectable difference (95% CIs 0.84–1.14), not "no difference". Up to about +14% per token is not ruled out.
- FLORES-101 source is described precisely: Wikinews, Wikijunior and Wikivoyage sentences, not "Wikipedia-style".
- Release asset: `koqloss-data-v0.1.1.json` and its `.sha256`.

## v0.1.0 (2026-09-28)

First public results, measured 2026-09-27.

- Q1: for the same content, Korean loses 1.47–3.45× more information than English under Q4_K_M/Q3_K_M quantization, relative to Q8_0 (per-document KL, 6/6 combinations).
- Per-token sensitivity differs by model: none for kanana-1.5-2.1b-instruct (intervals include 1.0), 1.42–2.04× for Qwen3-1.7B and Qwen3-4B.
- Q2: paired multiple-choice flip gap (KO−EN) is +2.8 to +18.5 points. Only Qwen3-1.7B's interval excludes 0.
- Q3: Q4_K_M generates fastest for all three models.
- Files: `kl.csv`, `mc.csv`, `speed.csv`, `models.csv`, `inputs.json`, `summary.json`, `run-*.json`, `SHA256SUMS`. Release asset: `koqloss-data-v0.1.0.json` (all tables in one file) and its `.sha256`. No weights, logits or FLORES-101 text.
