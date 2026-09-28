# How much more does quantization cost Korean? (koqloss v0.2.0)

Measured by Chipsnug on one Apple M4 Pro (24 GB, Metal) with llama.cpp 0.5.0 (build 11146, commit `7fe450e19`). Four open models, three GGUF levels each (Q8_0, Q4_K_M, Q3_K_M). Three models were measured on 2026-09-27; A.X-4.0-Light was added on 2026-09-28 with the same inputs (checked by SHA-256), settings and build. All numbers are relative to **Q8_0**, not to the original BF16 weights.

[한국어는 아래에 있습니다.](#한국어)

## Headline

1. **Loss a user sees.** For the same content, a Korean document loses **1.09–3.45×** more information than the English one under quantization. In 7 of 8 model × level combinations the 95% interval is above 1.0. The exception is A.X-4.0-Light at Q3_K_M (1.09, 0.97–1.23).
2. **Two things multiply into that number:** how many more tokens the tokenizer needs for Korean, and how much more each token loses.
   - Token count, Korean ÷ English: kanana 1.53×, Qwen3 1.69×, **A.X-4.0-Light 0.93×** (fewer tokens for Korean).
   - Per-token loss, Korean ÷ English: **kanana shows no detectable difference** (0.96–1.01×; 95% CIs 0.84–1.14 include 1.0). **A.X-4.0-Light 1.17–1.32×** (CIs 1.05–1.50). Qwen3 1.42–2.04×.
3. **Tested hypothesis, rejected.** Before measuring A.X-4.0-Light, we wrote down the rule that would decide the hypothesis "Korean-focused models have no per-token gap". A.X's per-token intervals sit above 1.0 at both levels (lower bounds 1.17 and 1.05; widths 0.34 and 0.28), so the rule rejects it. kanana's result is a property of kanana, not of Korean-focused models in general.
4. A.X-4.0-Light still has the **smallest user-facing gap** of the four (1.22 at Q4_K_M), because its tokenizer needs fewer Korean tokens. Quantization cost Korean more in every model we measured, but for different reasons and by very different amounts.

## Models

| Model | GGUF repo (same repo for all three levels) | Base license | Measured |
|---|---|---|---|
| kakaocorp/kanana-1.5-2.1b-instruct-2505 | DevQuasar/kakaocorp.kanana-1.5-2.1b-instruct-2505-GGUF | apache-2.0 | 2026-09-27 |
| Qwen/Qwen3-1.7B | bartowski/Qwen_Qwen3-1.7B-GGUF | apache-2.0 | 2026-09-27 |
| Qwen/Qwen3-4B | bartowski/Qwen_Qwen3-4B-GGUF | apache-2.0 | 2026-09-27 |
| skt/A.X-4.0-Light | mykor/A.X-4.0-Light-gguf | apache-2.0 | 2026-09-28 |

- We did not quantize anything ourselves. File names, sizes and SHA-256 of every GGUF are in `data/models.csv`. The weights are not in this repository.
- A.X-4.0-Light was the first model in a candidate list we fixed in advance. It is permissively licensed, has no access gate, and has all three levels in one repository. No bartowski or mradermacher conversion existed, so we used the most-downloaded repository that met the rule.

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
| A.X-4.0-Light | Q4_K_M | **1.22** | 1.08–1.39 | 0.93 | **1.32** | 1.17–1.50 | 1.01 |
| A.X-4.0-Light | Q3_K_M | **1.09** | 0.97–1.23 | 0.93 | **1.17** | 1.05–1.33 | 0.90 |

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
| A.X-4.0-Light | Q4_K_M | 60.7% · 70.7% | 58.7% · 72.3% | 7.1% · 2.8% | **+4.3 pt** | +0.2 to +8.7 |
| A.X-4.0-Light | Q3_K_M | 60.7% · 70.7% | 60.0% · 66.3% | 7.7% · 9.4% | **−1.7 pt** | −6.9 to +3.8 |

- The point estimate is higher in Korean in 7 of 8 combinations. The intervals exclude 0 for Qwen3-1.7B at both levels and for A.X-4.0-Light at Q4_K_M, which only just excludes 0 (lower bound +0.2).
- With 300 items the 95% half-width is about ±4.8 points at a 10% flip rate, and the minimum detectable difference (80% power) is about ±6.9 points. Gaps around 5 points cannot be separated from noise. There is no correction for multiple comparisons across the 8 combinations.
- Some English accuracies go *up* at a lower level (for example Qwen3-1.7B at Q4_K_M, +3.7 points). That is within 300-item noise.

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
| A.X-4.0-Light | Q8_0 | 404.1 | 28.1 | 7.88 |
| A.X-4.0-Light | Q4_K_M | 436.3 | 46.2 | 4.60 |
| A.X-4.0-Light | Q3_K_M | 422.5 | 34.4 | 3.74 |

- Q4_K_M generates fastest for all four models. Q3_K_M files are smaller but generate more slowly than Q4_K_M.
- The A.X-4.0-Light Q8_0 bench ran right after the model's own multiple-choice run. The 1-minute load average just before it was 5.69, higher than for the other two levels (2.88, 3.04), so treat that row as approximate.

## Cost per combination (new in v0.2.0)

`data/cost.csv` records, for each model × level:
- **Measurement time**: KL in both languages plus multiple choice in both languages, and the bench where it was timed.
- **File size.**
- **Download time**: A.X-4.0-Light only.

| | Measurement per combination | File | Download |
|---|---|---|---|
| kanana, Qwen3 (2026-09-27) | 1.1–2.6 min (bench not timed) | 1.1–4.3 GB | not recorded |
| A.X-4.0-Light (2026-09-28) | 3.8–3.9 min (bench included) | 3.6–7.7 GB | 4.5–8.5 min |

- A.X-4.0-Light end to end took 30 minutes of wall-clock time: 3 downloads, 12 KL and multiple-choice runs, and 3 benches.
- Its Q8_0 download overlapped with other CPU work on the same machine, so that download time may be inflated. The note in `cost.csv` marks it.
- All benches ran after that window.

## Method

- **Q1.** `llama-perplexity -c 512 --kl-divergence-base <file>` with Q8_0 on each language's document, then `llama-perplexity -c 512 --kl-divergence-base <file> --kl-divergence` with each lower level. llama.cpp scores the last `n_ctx − 1 − n_ctx/2` tokens of each chunk. Document token counts come from `llama-tokenize` (BOS included). The unscored tail (under 512 tokens) is assumed to have the same mean.
- **Q2.** MMLU letter format. The four choices are written into the question, and the answer is a single letter A–D. The Korean header is `질문`/`정답`, the English one is `Question`/`Answer`. Run with `llama-perplexity -c 2048 --multiple-choice` over all 300 items with no task limit. Pairs are matched by row position. Subject and answer letter agreed for 300/300 pairs.
- **Inputs, without text.** `data/inputs.json` contains everything you need to rebuild the inputs from the public datasets and check them:
  - the FLORES-101 sentence IDs and the exact rule for building each document
  - the 300 MMLU row positions, with subject and answer
  - the prompt template
  - a SHA-256 for every input
  The input files used for A.X-4.0-Light matched these hashes.
- **Per-run numbers.** `data/run-*.json` holds per-chunk KL, per-item correct/incorrect bits (in item order), token counts and bench output. Every interval in this report can be recomputed from these files.

## Limits

- One device and one run per model. Speed was not controlled for temperature or background load. Max RSS is macOS `time -l`, and we did not check whether it counts all unified memory used by Metal.
- The reference is Q8_0, not BF16. Loss against the original weights is larger. If Q8_0 itself loses differently by language, the ratios can be biased.
- Q1 uses 300 sentences from Wikinews, Wikijunior and Wikivoyage (FLORES-101). Other domains may give different ratios. The chunk bootstrap (15–27 chunks) can make intervals slightly narrow.
- Q2 has 300 items and no multiple-comparison correction across 8 combinations. Letter-choice scoring is not the same as the quality of generated answers.
- Two Korean-focused models are not enough to say what Korean-focused training does in general. They disagree on per-token sensitivity.
- Automatic metrics (KL, multiple choice) can understate what people notice. Results do not transfer to other chips, NPUs or models.

## Files and licenses

| File | Contents | License |
|---|---|---|
| `data/kl.csv`, `data/mc.csv`, `data/speed.csv`, `data/cost.csv`, `data/summary.json` | Result tables | CC BY 4.0 |
| `data/models.csv` | GGUF repo, file, size, SHA-256, base license | CC BY 4.0 (facts about third-party files) |
| `data/run-*.json` | Per-run numbers | CC BY 4.0 |
| `data/inputs.json` | Input IDs, build rules and hashes. The FLORES-101 part is derived from a CC BY-SA 4.0 dataset. See `THIRD_PARTY_NOTICES.md` | CC BY 4.0, plus CC BY-SA 4.0 terms for the FLORES-101-derived fields |

No model weights, logits or FLORES-101 text are included.

---

## 한국어

# 양자화하면 한국어는 얼마나 더 잃는가 (koqloss v0.2.0)

Chipsnug가 Apple M4 Pro(24GB, Metal) 1대에서 llama.cpp 0.5.0(build 11146, commit `7fe450e19`)으로 쟀다. 공개 모델 4개를 각각 GGUF 3수준(Q8_0·Q4_K_M·Q3_K_M)으로 쟀다. 3개는 2026-09-27에, A.X-4.0-Light는 2026-09-28에 같은 입력(SHA-256으로 확인)·설정·빌드로 더했다. 모든 값은 원본 BF16이 아니라 **Q8_0 대비**다.

## 요약

1. **사용자가 겪는 손실.** 같은 내용이면 한국어 문서가 양자화로 잃는 정보가 영어보다 **1.09~3.45배** 크다.
   - 모델·수준 8조합 중 7조합에서 95% 구간이 1.0보다 크다.
   - 예외는 A.X-4.0-Light Q3_K_M(1.09, 0.97~1.23)이다.
2. **이 값은 두 가지의 곱이다**: 토크나이저가 한국어를 몇 배 많은 토큰으로 쪼개는가, 토큰 하나가 몇 배 더 잃는가.
   - 토큰 수(한÷영): kanana 1.53배, Qwen3 1.69배, **A.X-4.0-Light 0.93배**(한국어를 더 적게 쪼갬).
   - 토큰당 손실(한÷영)
     - **kanana**: 검출할 만한 차이가 없다(0.96~1.01배, 95% 구간 0.84~1.14가 1.0 포함).
     - **A.X-4.0-Light**: 1.17~1.32배(구간 1.05~1.50).
     - Qwen3: 1.42~2.04배.
3. **검증한 가설: 기각.**
   - 가설은 "한국어 중심 모델은 토큰당 격차가 없다"이다. A.X-4.0-Light를 재기 전에 판정 규칙을 먼저 적어 두었다.
   - A.X의 토큰당 구간은 두 수준 모두 1.0보다 위에 있다(하한 1.17·1.05, 폭 0.34·0.28). 규칙대로 기각이다.
   - kanana의 결과는 한국어 중심 모델 일반의 성질이 아니라 kanana의 성질이다.
4. 그래도 A.X-4.0-Light는 네 모델 중 **사용자가 겪는 격차가 가장 작다**(Q4_K_M 1.22). 토크나이저가 한국어를 적게 쪼개기 때문이다. 잰 모든 모델에서 양자화는 한국어에 더 큰 손실을 냈지만, 이유와 크기는 모델마다 크게 다르다.

## 모델

- 위 영문 표와 같다: kanana-1.5-2.1b-instruct(DevQuasar GGUF), Qwen3-1.7B·Qwen3-4B(bartowski GGUF), A.X-4.0-Light(mykor GGUF). 원본 라이선스는 모두 apache-2.0이다.
- 세 수준은 같은 저장소에서 받았고, 직접 양자화하지 않았다.
- A.X-4.0-Light는 미리 고정한 후보 목록의 1순위였다. 조건은 허용 라이선스, 게이트 없음, 한 저장소에 3수준이다. bartowski·mradermacher 변환본이 없어 조건을 만족하는 저장소 중 다운로드가 가장 많은 곳을 썼다.
- 파일 이름·크기·SHA-256은 `data/models.csv`에 있다. 가중치는 이 저장소에 없다.

## Q1 — 병렬 텍스트 KL 발산(Q8_0 대비)

- 텍스트: FLORES-101 devtest 앞 300문장, 한국어와 영어(같은 뜻).
- 판정: 문서당 KL(토큰당 평균 KL × 문서 전체 토큰 수)의 한/영 비.

| 모델 | 수준 | 한/영 문서당 비 | 토큰 수 비 | 토큰당 비 |
|---|---|---|---|---|
| kanana | Q4_K_M · Q3_K_M | 1.54 · 1.47 | 1.53 | 1.01 · 0.96 (구간이 1.0 포함) |
| Qwen3-1.7B | Q4_K_M · Q3_K_M | 2.93 · 3.45 | 1.69 | 1.73 · 2.04 |
| Qwen3-4B | Q4_K_M · Q3_K_M | 2.40 · 2.75 | 1.69 | 1.42 · 1.63 |
| A.X-4.0-Light | Q4_K_M · Q3_K_M | 1.22 · 1.09 (Q3 구간이 1.0 포함) | 0.93 | 1.32 · 1.17 (구간이 1.0보다 위) |

- 문서당 비 = 토큰 수 비 × 토큰당 비. 구간은 위 영문 표에 있다.
- 바이트당 비는 참고용이다. UTF-8에서 한글은 3바이트, 영문은 1바이트라 인코딩 효과가 섞인다.

## Q2 — 문항 쌍 다지선다(MMMLU 한국어 ↔ MMLU 영어 300쌍, Q8_0 대비)

- 전환율: P(낮은 수준에서 오답 | Q8_0에서 정답).
- 8조합 중 7조합에서 점추정은 한국어가 크다.
- 구간이 0을 넘는 조합은 Qwen3-1.7B 두 수준과 A.X-4.0-Light Q4_K_M이다. A.X는 하한이 +0.2로 간신히 넘는다.
- 300문항이면 전환율 10%에서 95% 반폭이 약 ±4.8%p이고, 최소 검출 차이(검정력 80%)는 약 ±6.9%p다. 8조합을 다중 비교 보정 없이 본다.

## Q3 — 속도·메모리

- 네 모델 모두 Q4_K_M의 생성 속도가 가장 빠르다. Q3_K_M은 파일이 작지만 생성은 Q4_K_M보다 느리다.
- A.X-4.0-Light Q8_0 bench는 같은 모델의 다지선다 실행 바로 뒤에 돌았다. 직전 1분 부하 평균이 5.69로 다른 두 수준(2.88·3.04)보다 높았으므로 근삿값으로 본다.

## 조합당 비용 (v0.2.0 새 표)

- `data/cost.csv`에 모델·수준마다 다음을 적었다.
  - 측정 시간: KL 2언어 + 다지선다 2언어. 시간을 잰 경우 bench 포함.
  - 파일 크기.
  - 받기 시간: A.X-4.0-Light만.
- 값
  - kanana·Qwen3(9/27): 조합당 1.1~2.6분(bench 시간은 기록 안 함), 파일 1.1~4.3GB.
  - A.X-4.0-Light(9/28): 조합당 3.8~3.9분(bench 포함), 파일 3.6~7.7GB, 받기 4.5~8.5분. 3수준 전체는 벽시계 30분이다.
- A.X Q8_0 받기 시간은 같은 기기의 다른 CPU 작업과 겹쳐 부풀었을 수 있다(`cost.csv` 메모). bench는 모두 그 뒤에 돌았다.

## 방법

- Q1·Q2 명령과 채점 범위는 위 영문 "Method"와 같다.
- `data/inputs.json`에는 입력 ID·만드는 규칙·해시가 있다. 공개 자료에서 입력을 다시 만들어 해시로 맞춰 볼 수 있다. A.X-4.0-Light 측정에 쓴 입력 파일은 이 해시와 같았다.
- `data/run-*.json`에는 청크별 KL, 문항별 정오 비트, 토큰 수, bench 출력이 있다. 보고서의 모든 구간을 이 파일로 다시 계산할 수 있다.

## 한계

- 기기 1대, 모델당 측정 1회다. 속도는 온도·부하를 통제하지 않았다.
- 기준이 BF16이 아니라 Q8_0이다. 원본 대비 손실은 이보다 크다.
- Q1은 FLORES-101의 Wikinews·Wikijunior·Wikivoyage 300문장뿐이다. 분야가 바뀌면 비가 달라질 수 있다.
- Q2는 300문항이고 8조합을 다중 비교 보정 없이 본다. 글자 선택 채점이라 생성 답변의 품질과는 다르다.
- 한국어 중심 모델 2개로는 한국어 중심 학습의 일반적 효과를 말할 수 없다. 두 모델은 토큰당 민감도에서 서로 다르다.
- 자동 지표는 사람이 느끼는 손실보다 작게 잡을 수 있다. 다른 칩·NPU·모델로 일반화할 수 없다.

## 파일과 라이선스

- 결과표·실행별 수치: CC BY 4.0.
- `data/inputs.json`의 FLORES-101 파생 항목(문장 ID·문서 해시·만드는 규칙)은 CC BY-SA 4.0 조건도 따른다(`THIRD_PARTY_NOTICES.md`).
- 모델 가중치, 로짓, FLORES-101 원문은 넣지 않았다.
