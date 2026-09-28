---
license: cc-by-4.0
language:
- ko
- en
pretty_name: Chipsnug koqloss — Korean vs English quantization loss
size_categories:
- n<1K
tags:
- quantization
- gguf
- llama.cpp
- kl-divergence
- korean
configs:
- config_name: kl
  data_files: data/kl.csv
- config_name: multiple_choice
  data_files: data/mc.csv
- config_name: speed
  data_files: data/speed.csv
- config_name: models
  data_files: data/models.csv
---

# Chipsnug koqloss (v0.1.1)

This dataset measures how much more information Korean loses than English when an open model is quantized, using the same content in both languages. It is a mirror of the result tables in <https://github.com/chipsnug/koqloss>. The full report, in English and then Korean, is in `koqloss-public.md`.

- **Q1 — KL vs Q8_0 on parallel text** (FLORES-101 devtest, sentences 1–300). For the same content, Korean loses **1.47–3.45×** more than English (6/6 model × level combinations, no 95% interval includes 1.0). Per token, kanana-1.5-2.1b-instruct shows no detectable extra loss (95% CIs 0.84–1.14 include 1.0), so within this precision its gap is explained by token count (1.53×). Qwen3-1.7B and Qwen3-4B add 1.42–2.04× per token.
- **Q2 — paired multiple choice** (300 MMMLU KO_KR ↔ MMLU items). The KO−EN flip gap is +2.8 to +18.5 points. Only Qwen3-1.7B's interval excludes 0.
- **Q3 — speed and memory** on an Apple M4 Pro (Metal). Q4_K_M generates fastest for all three models.

Models (GGUF, three levels from the same repo, no self-quantization):
- kakaocorp/kanana-1.5-2.1b-instruct-2505 (DevQuasar)
- Qwen/Qwen3-1.7B (bartowski)
- Qwen/Qwen3-4B (bartowski)

Q8_0 is the reference, not BF16. One device, one session, 300 sentences, 300 item pairs.

## Files

| File | Contents |
|---|---|
| `data/kl.csv` | Q1 ratios with 95% intervals |
| `data/mc.csv` | Q2 accuracy, flip rates and gap with 95% intervals |
| `data/speed.csv` | Q3 tokens/s and max RSS |
| `data/models.csv` | GGUF repos, files, sizes and SHA-256 |
| `data/inputs.json` | Input IDs, build rules and SHA-256. No source text |
| `data/run-*.json` | Per-chunk KL, per-item correct bits, bench output |
| `data/summary.json` | All tables in one file |

## License

- Tables and documentation: CC BY 4.0.
- The FLORES-101-derived fields in `data/inputs.json` (`q1_kl_document`: sentence IDs, hashes, build rule) also follow **CC BY-SA 4.0**, the license of FLORES-101. See `THIRD_PARTY_NOTICES.md`.
- MMMLU and MMLU are MIT. Only row positions and hashes are included.
- No model weights, logits or FLORES-101 text are included.

## Contact

hello@chipsnug.com · Updates: <https://chipsnug.com/?utm_source=hf-dataset&utm_medium=tool&utm_campaign=koqloss-v0.1.1>

## 한국어

같은 내용을 두고, 공개 모델을 양자화했을 때 한국어가 영어보다 정보를 얼마나 더 잃는지 잰 결과표다. 원본 저장소는 <https://github.com/chipsnug/koqloss>이다.

- 같은 내용이면 한국어가 영어보다 1.47~3.45배 더 잃는다(Q8_0 대비, 6/6 조합).
- 토큰당으로 보면 kanana는 검출할 만한 차이가 없고(95% 구간 0.84~1.14), Qwen3은 1.42~2.04배를 더 잃는다.
- 전체 보고서는 `koqloss-public.md`에 있다.

라이선스는 결과표 CC BY 4.0이다. FLORES-101에서 나온 항목은 CC BY-SA 4.0 조건도 따른다. 모델 가중치·로짓·FLORES-101 원문은 없다.
