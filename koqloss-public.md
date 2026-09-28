# How much more does quantization cost Korean? (koqloss v0.1.1)

Measured 2026-09-27 by Chipsnug. Three open models, three GGUF levels each (Q8_0, Q4_K_M, Q3_K_M), on one Apple M4 Pro (24 GB, Metal) with llama.cpp 0.5.0 (build 11146, commit `7fe450e19`). All numbers are relative to **Q8_0**, not to the original BF16 weights.

[한국어는 아래에 있습니다.](#한국어)

## Headline

1. **Loss a user sees.** For the same content, a Korean document loses **1.47–3.45×** more information than the English one under quantization. This holds for all 6 model × level combinations, and no 95% interval includes 1.0.
2. **Per-token sensitivity depends on the model.** kanana-1.5-2.1b-instruct shows no detectable per-token difference (0.96–1.01×; 95% CIs 0.84–1.14 include 1.0, so up to about +14% per token is not ruled out). Within this precision, its Korean gap is explained by the tokenizer splitting Korean into 1.53× more tokens. Qwen3 adds 1.42–2.04× per token on top of 1.69× more tokens.
3. So "quantization hurts Korean more" is **not** true for every model. The explanation that a Korean-focused tokenizer and training remove the per-token gap is a hypothesis: we only tested two model families.

## Models

| Model | GGUF repo (same repo for all three levels) | Base license |
|---|---|---|
| kakaocorp/kanana-1.5-2.1b-instruct-2505 | DevQuasar/kakaocorp.kanana-1.5-2.1b-instruct-2505-GGUF | apache-2.0 |
| Qwen/Qwen3-1.7B | bartowski/Qwen_Qwen3-1.7B-GGUF | apache-2.0 |
| Qwen/Qwen3-4B | bartowski/Qwen_Qwen3-4B-GGUF | apache-2.0 |

We did not quantize anything ourselves. File names, sizes and SHA-256 of every GGUF are in `data/models.csv`. The weights are not in this repository.

## Q1 — KL divergence on parallel text (vs Q8_0)

Text: the first 300 sentences of FLORES-101 devtest, in Korean and in English (same meaning). The score is **per-document KL** = mean per-token KL × total tokens in the document. Its Korean/English ratio answers "how much more does the same content lose in Korean".

| Model | Level | **KO/EN per-document** | 95% CI | Token count ratio | **Per-token ratio** | 95% CI | Per-byte ratio |
|---|---|---|---|---|---|---|---|
| kanana-1.5-2.1b-instruct | Q4_K_M | **1.54** | 1.37–1.74 | 1.53 | **1.01** | 0.89–1.14 | 1.27 |
| kanana-1.5-2.1b-instruct | Q3_K_M | **1.47** | 1.28–1.68 | 1.53 | **0.96** | 0.84–1.10 | 1.21 |
| Qwen3-1.7B | Q4_K_M | **2.93** | 2.55–3.39 | 1.69 | **1.73** | 1.51–2.01 | 2.42 |
| Qwen3-1.7B | Q3_K_M | **3.45** | 3.07–3.89 | 1.69 | **2.04** | 1.82–2.30 | 2.85 |
| Qwen3-4B | Q4_K_M | **2.40** | 2.13–2.68 | 1.69 | **1.42** | 1.26–1.59 | 1.98 |
| Qwen3-4B | Q3_K_M | **2.75** | 2.44–3.12 | 1.69 | **1.63** | 1.45–1.85 | 2.27 |

- Per-document ratio = token count ratio × per-token ratio.
- Per-byte ratio is shown for reference only. Hangul is 3 bytes per character in UTF-8 and English letters are 1, so per-byte values mix in the encoding.
- Intervals: percentile bootstrap over llama.cpp chunks (15–27 chunks per language).

## Q2 — Paired multiple choice (300 MMMLU KO ↔ MMLU EN items, vs Q8_0)

Flip rate = P(wrong at the lower level | right at Q8_0). Using Q8_0-correct items as the denominator keeps the two languages' different base accuracy out of the comparison.

| Model | Level | Q8_0 acc KO · EN | Level acc KO · EN | Flip rate KO · EN | **Flip gap (KO−EN)** | 95% CI |
|---|---|---|---|---|---|---|
| kanana-1.5-2.1b-instruct | Q4_K_M | 47.7% · 51.3% | 44.3% · 50.3% | 12.6% · 9.1% | **+3.5 pt** | −3.3 to +10.0 |
| kanana-1.5-2.1b-instruct | Q3_K_M | 47.7% · 51.3% | 39.0% · 46.0% | 25.9% · 18.8% | **+7.0 pt** | −2.6 to +16.9 |
| Qwen3-1.7B | Q4_K_M | 47.0% · 53.7% | 44.7% · 57.3% | 18.4% · 4.3% | **+14.1 pt** | +7.5 to +21.2 |
| Qwen3-1.7B | Q3_K_M | 47.0% · 53.7% | 39.3% · 56.3% | 34.0% · 15.5% | **+18.5 pt** | +9.7 to +27.4 |
| Qwen3-4B | Q4_K_M | 56.7% · 69.3% | 54.3% · 70.3% | 10.0% · 7.2% | **+2.8 pt** | −3.0 to +8.1 |
| Qwen3-4B | Q3_K_M | 56.7% · 69.3% | 53.3% · 66.0% | 17.6% · 12.5% | **+5.1 pt** | −1.7 to +12.0 |

- The point estimate is higher in Korean in all 6 combinations. Only Qwen3-1.7B's interval excludes 0.
- With 300 items the 95% half-width is about ±4.8 points at a 10% flip rate, and the minimum detectable difference (80% power) is about ±6.9 points. Gaps around 5 points cannot be separated from noise.
- Some English accuracies go *up* at Q4_K_M (for example Qwen3-1.7B, +3.7 points). That is within 300-item noise.

## Q3 — Speed and memory (`llama-bench -p 512 -n 128 -r 3`, Metal)

| Model | Level | Prompt tok/s | Generate tok/s | Max RSS (GB) |
|---|---|---|---|---|
| kanana-1.5-2.1b-instruct | Q8_0 | 1572.6 | 90.7 | 2.70 |
| kanana-1.5-2.1b-instruct | Q4_K_M | 1541.6 | 123.9 | 1.75 |
| kanana-1.5-2.1b-instruct | Q3_K_M | 1444.6 | 95.9 | 1.46 |
| Qwen3-1.7B | Q8_0 | 2160.1 | 109.1 | 2.36 |
| Qwen3-1.7B | Q4_K_M | 2029.0 | 153.7 | 1.48 |
| Qwen3-1.7B | Q3_K_M | 1975.3 | 123.8 | 1.28 |
| Qwen3-4B | Q8_0 | 759.0 | 50.0 | 4.50 |
| Qwen3-4B | Q4_K_M | 544.5 | 73.0 | 2.73 |
| Qwen3-4B | Q3_K_M | 735.8 | 56.3 | 2.31 |

Q4_K_M generates fastest for all three models. Q3_K_M files are smaller but generate more slowly than Q4_K_M.

## Method

- **Q1.** `llama-perplexity -c 512 --kl-divergence-base <file>` with Q8_0 on each language's document, then `llama-perplexity -c 512 --kl-divergence-base <file> --kl-divergence` with each lower level. llama.cpp scores the last `n_ctx − 1 − n_ctx/2` tokens of each chunk. Document token counts come from `llama-tokenize` (BOS included). The unscored tail (under 512 tokens) is assumed to have the same mean.
- **Q2.** MMLU letter format. The four choices are written into the question, and the answer is a single letter A–D. The Korean header is `질문`/`정답`, the English one is `Question`/`Answer`. Run with `llama-perplexity -c 2048 --multiple-choice` over all 300 items with no task limit. Pairs are matched by row position. Subject and answer letter agreed for 300/300 pairs.
- **Inputs, without text.** `data/inputs.json` lists the FLORES-101 sentence IDs, the exact rule for building each document, the 300 MMLU row positions with subject and answer, the prompt template, and a SHA-256 for every input. You can rebuild the inputs from the public datasets and check that they match.
- **Per-run numbers.** `data/run-*.json` holds per-chunk KL, per-item correct/incorrect bits (in item order), token counts and bench output. Every interval in this report can be recomputed from these files.

## Limits

- One device, one session, one run. Speed was not controlled for temperature or background load. Max RSS is macOS `time -l`, and we did not check whether it counts all unified memory used by Metal.
- The reference is Q8_0, not BF16. Loss against the original weights is larger. If Q8_0 itself loses differently by language, the ratios can be biased.
- Q1 uses 300 sentences from Wikinews, Wikijunior and Wikivoyage (FLORES-101). Other domains may give different ratios. The chunk bootstrap (15–27 chunks) can make intervals slightly narrow, but the lowest Qwen3 per-token bound (1.26) leaves room.
- Q2 has 300 items and no multiple-comparison correction across 6 combinations. Letter-choice scoring is not the same as the quality of generated answers.
- Automatic metrics (KL, multiple choice) can understate what people notice. Results do not transfer to other chips, NPUs or models.

## Files and licenses

| File | Contents | License |
|---|---|---|
| `data/kl.csv`, `data/mc.csv`, `data/speed.csv`, `data/summary.json` | Result tables | CC BY 4.0 |
| `data/models.csv` | GGUF repo, file, size, SHA-256, base license | CC BY 4.0 (facts about third-party files) |
| `data/run-*.json` | Per-run numbers | CC BY 4.0 |
| `data/inputs.json` | Input IDs, build rules and hashes. The FLORES-101 part is derived from a CC BY-SA 4.0 dataset. See `THIRD_PARTY_NOTICES.md` | CC BY 4.0, plus CC BY-SA 4.0 terms for the FLORES-101-derived fields |

No model weights, no logits and no FLORES-101 text are included.

---

## 한국어

# 양자화하면 한국어는 얼마나 더 잃는가 (koqloss v0.1.1)

2026-09-27에 Chipsnug가 측정했다. 공개 모델 3개를 각각 GGUF 3수준(Q8_0·Q4_K_M·Q3_K_M)으로 쟀다. 기기는 Apple M4 Pro(24GB, Metal) 1대이고, llama.cpp는 0.5.0(build 11146, commit `7fe450e19`)이다. 모든 값은 원본 BF16이 아니라 **Q8_0 대비**다.

## 요약

1. **사용자가 겪는 손실.** 같은 내용이면 한국어 문서가 양자화로 잃는 정보가 영어보다 **1.47~3.45배** 크다. 모델·수준 6조합 모두 그렇고, 95% 구간이 1.0을 포함한 조합은 없다.
2. **토큰 하나당 민감도는 모델마다 다르다.** kanana-1.5-2.1b-instruct는 검출할 만한 차이가 없다(0.96~1.01배, 95% 구간 0.84~1.14가 1.0을 포함한다. 토큰당 최대 약 +14%까지는 배제하지 못한다). 이 정밀도 안에서 kanana의 한국어 격차는 토크나이저가 한국어를 1.53배 더 많은 토큰으로 쪼개는 것으로 설명된다. Qwen3은 토큰이 1.69배 많은 데다 토큰당으로도 1.42~2.04배를 더 잃는다.
3. 그래서 "양자화가 한국어에 더 해롭다"는 모든 모델에 맞는 말이 **아니다**. "한국어 중심 토크나이저·학습이 토큰당 격차를 없앤다"는 설명은 가설이다. 모델군을 둘만 쟀다.

## 모델

위 영문 표와 같다. kanana-1.5-2.1b-instruct(DevQuasar GGUF), Qwen3-1.7B·Qwen3-4B(bartowski GGUF)이고, 원본 라이선스는 모두 apache-2.0이다. 세 수준은 같은 저장소에서 받았고, 직접 양자화하지 않았다. 파일 이름·크기·SHA-256은 `data/models.csv`에 있다. 가중치는 이 저장소에 없다.

## Q1 — 병렬 텍스트 KL 발산(Q8_0 대비)

- 텍스트: FLORES-101 devtest의 앞 300문장을 한국어와 영어로 쓴다(같은 뜻).
- 판정: **문서당 KL**(토큰당 평균 KL × 문서 전체 토큰 수)의 한/영 비.
- 결과는 위 영문 표와 같다.

| 모델 | 수준 | 한/영 문서당 비 | 토큰당 비 |
|---|---|---|---|
| kanana | Q4_K_M · Q3_K_M | 1.54 · 1.47 | 1.01 · 0.96 (구간이 1.0 포함) |
| Qwen3-1.7B | Q4_K_M · Q3_K_M | 2.93 · 3.45 | 1.73 · 2.04 |
| Qwen3-4B | Q4_K_M · Q3_K_M | 2.40 · 2.75 | 1.42 · 1.63 |

- 문서당 비 = 토큰 수 비 × 토큰당 비.
- 바이트당 비는 참고용이다. UTF-8에서 한글은 3바이트, 영문은 1바이트라 인코딩 효과가 섞인다.

## Q2 — 문항 쌍 다지선다(MMMLU 한국어 ↔ MMLU 영어 300쌍, Q8_0 대비)

- 전환율: P(낮은 수준에서 오답 | Q8_0에서 정답).
- 6조합 모두 점추정은 한국어가 크다(+2.8~+18.5%p). 다만 구간이 0을 넘는 모델은 Qwen3-1.7B뿐이다.
- 300문항이면 전환율 10%에서 95% 반폭이 약 ±4.8%p다. 최소 검출 차이(검정력 80%)는 약 ±6.9%p다. 5%p 안팎의 차이는 잡음과 구분하지 못한다.

## Q3 — 속도·메모리

- 세 모델 모두 Q4_K_M의 생성 속도가 가장 빠르다.
- Q3_K_M은 파일이 작지만 생성은 Q4_K_M보다 느리다. 표는 위 영문 절에 있다.

## 방법

- **Q1**: 명령과 채점 범위는 위 영문 "Method"와 같다. 문서 토큰 수는 `llama-tokenize`(BOS 포함)로 셌다. 채점하지 않은 꼬리 토큰도 같은 평균이라고 가정한다.
- **Q2**: MMLU 글자 형식이다.
  - 보기 A~D를 질문 본문에 넣고, 답은 글자 하나다.
  - 머리말은 `질문`/`정답`과 `Question`/`Answer`다.
  - 300문항 전체를 `-c 2048 --multiple-choice`로 돌렸다.
  - 짝은 행 위치로 맞췄다. 과목·정답 기호는 300쌍 모두 일치했다.
- **입력(원문 없이)**: `data/inputs.json`에 다음을 담았다. 공개 자료에서 입력을 다시 만들어 해시로 맞춰 볼 수 있다.
  - FLORES-101 문장 ID
  - 문서를 만드는 규칙
  - MMLU 행 위치 300개(과목·정답 포함)
  - 프롬프트 형식
  - 입력별 SHA-256
- **실행별 수치**: `data/run-*.json`에 다음을 담았다. 보고서의 모든 구간을 이 파일로 다시 계산할 수 있다.
  - 청크별 KL
  - 문항별 정오 비트
  - 토큰 수
  - bench 출력

## 한계

- 기기 1대에서 한 번만 측정했다. 속도는 온도·부하를 통제하지 않았다.
- 기준이 BF16이 아니라 Q8_0이다. 원본 대비 손실은 이보다 크다.
- Q1은 FLORES-101의 Wikinews·Wikijunior·Wikivoyage 300문장뿐이다. 분야가 바뀌면 비가 달라질 수 있다.
- Q2는 300문항이고, 6조합을 다중 비교 보정 없이 본다. 글자 선택 채점이라 생성 답변의 품질과는 다르다.
- 자동 지표는 사람이 느끼는 손실보다 작게 잡을 수 있다. 다른 칩·NPU·모델로 일반화할 수 없다.

## 파일과 라이선스

- 결과표·실행별 수치: CC BY 4.0.
- `data/inputs.json`의 FLORES-101 파생 항목(문장 ID·문서 해시·만드는 규칙): CC BY-SA 4.0 조건도 따른다(`THIRD_PARTY_NOTICES.md`).
- 모델 가중치, 로짓, FLORES-101 원문은 넣지 않았다.
