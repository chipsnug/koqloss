# Chipsnug koqloss

How much more information Korean loses than English when an open model is quantized, measured on the same content.

- **What it measures:** KL divergence of Q4_K_M and Q3_K_M against Q8_0 on parallel Korean/English text, paired multiple-choice flip rates, and speed/memory. 3 models × 3 GGUF levels, llama.cpp on one Apple M4 Pro.
- **Main result (v0.1.1):** for the same content, Korean loses 1.47–3.45× more than English (6/6 combinations). Per token, kanana-1.5-2.1b-instruct shows no detectable extra loss (95% CI 0.84–1.14), so within this precision its gap is explained by token count (1.53×). Qwen3 adds 1.42–2.04× per token.
- **Full report:** [`koqloss-public.md`](koqloss-public.md) (English, then Korean).

## Data

| File | What |
|---|---|
| `data/kl.csv` | Q1: KO/EN per-document, per-token and per-byte KL ratios with 95% intervals |
| `data/mc.csv` | Q2: accuracy, flip rate and KO−EN flip gap with 95% intervals |
| `data/speed.csv` | Q3: prompt/generation tokens per second, max RSS |
| `data/models.csv` | GGUF repos, files, sizes and SHA-256 (weights not included) |
| `data/inputs.json` | Input IDs, build rules and SHA-256 (no source text) |
| `data/run-*.json` | Per-run numbers: per-chunk KL, per-item correct bits, bench output |
| `data/summary.json` | All of the above tables in one JSON |
| `data/SHA256SUMS` | `cd data && sha256sum -c SHA256SUMS` |

Each GitHub release attaches `koqloss-data-<version>.json`, one JSON bundle with all of the above, plus its `.sha256`. The files are also mirrored to the Hugging Face dataset `chipsnug/ko-quant-loss-v0`.

## Method and limits

Q8_0 is the reference, not BF16. One device, one session, 300 sentences (FLORES-101 devtest) and 300 item pairs (MMMLU KO_KR ↔ MMLU). Letter-choice scoring. Details and all caveats are in the report.

## Licenses

- Data and tables: CC BY 4.0 ([`LICENSE-DATA`](LICENSE-DATA)).
- Fields in `data/inputs.json` derived from FLORES-101 (sentence IDs, document hashes, build rule) also follow CC BY-SA 4.0. See [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
- Repository code (the release workflow): MIT ([`LICENSE`](LICENSE)).
- No model weights, logits or FLORES-101 text are included.

## Contact

hello@chipsnug.com · Updates: <https://chipsnug.com/?utm_source=gh-release&utm_medium=tool&utm_campaign=koqloss-v0.1.1>

---

## 한국어

같은 내용을 두고, 공개 모델을 양자화했을 때 한국어가 영어보다 정보를 얼마나 더 잃는지 쟀다.

- **무엇을 재나:** 세 가지를 잰다. 첫째, 한·영 병렬 텍스트에서 Q8_0 대비 Q4_K_M·Q3_K_M의 KL 발산. 둘째, 문항 쌍 다지선다 전환율. 셋째, 속도·메모리. 모델 3개 × GGUF 3수준이고, Apple M4 Pro 1대에서 llama.cpp로 쟀다.
- **결과(v0.1.1):** 같은 내용이면 한국어가 영어보다 1.47~3.45배 더 잃는다(6/6 조합). 토큰당으로 보면 kanana-1.5-2.1b-instruct는 검출할 만한 추가 손실이 없다(95% 구간 0.84~1.14). 이 정밀도 안에서 kanana의 격차는 토큰 수(1.53배)로 설명된다. Qwen3은 토큰당으로도 1.42~2.04배를 더 잃는다.
- **전체 보고서:** [`koqloss-public.md`](koqloss-public.md)(영어 다음 한국어).

## 데이터

- 파일 구성은 위 표와 같다. 검증은 `cd data && sha256sum -c SHA256SUMS`로 한다.
- GitHub 릴리스에는 위 내용을 모두 담은 JSON 묶음 `koqloss-data-<버전>.json`과 그 `.sha256`을 첨부한다. 같은 파일을 Hugging Face 데이터셋 `chipsnug/ko-quant-loss-v0`에도 올린다.

## 방법과 한계

- 기준은 BF16이 아니라 Q8_0이다.
- 측정 조건
  - 기기 1대, 측정 1회
  - FLORES-101 devtest 300문장
  - MMMLU KO_KR ↔ MMLU 300쌍
  - 글자 선택 채점
- 자세한 내용과 한계는 보고서에 있다.

## 라이선스

- 데이터·결과표: CC BY 4.0(`LICENSE-DATA`).
- `data/inputs.json`에서 FLORES-101에서 나온 항목(문장 ID·문서 해시·만드는 규칙): CC BY-SA 4.0 조건도 따른다(`THIRD_PARTY_NOTICES.md`).
- 저장소 코드(릴리스 워크플로): MIT(`LICENSE`).
- 모델 가중치·로짓·FLORES-101 원문은 넣지 않았다.

## 연락

hello@chipsnug.com · 소식 받기: <https://chipsnug.com/?utm_source=gh-release&utm_medium=tool&utm_campaign=koqloss-v0.1.1>
