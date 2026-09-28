# Third-party notices

This repository contains **no text, questions or model weights** from the sources below. It contains identifiers (row positions, sentence IDs), SHA-256 hashes, file names and measured numbers only.

## FLORES-101 (Q1 parallel text)

- Dataset used: `gsarti/flores_101` on Hugging Face, configs `kor` and `eng`, split `devtest`, sentences with id 1–300.
- Original work: Goyal et al., "The FLORES-101 Evaluation Benchmark for Low-Resource and Multilingual Machine Translation", 2021 (Meta AI).
- License: Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0), https://creativecommons.org/licenses/by-sa/4.0/
- What we include: sentence IDs, SHA-256 and byte counts of the two 300-sentence documents, and the rule for building them (`data/inputs.json` → `q1_kl_document`). These fields are shared under CC BY-SA 4.0. No changes were made to the sentences. They were joined with newlines.

## MMMLU (Q2 Korean items)

- Dataset used: `openai/MMMLU`, config `KO_KR`, split `test`.
- License: MIT.
- What we include: 300 row positions, subject, answer letter and the SHA-256 of each formatted prompt.

## MMLU (Q2 English items)

- Dataset used: `cais/mmlu`, config `all`, split `test`.
- Original work: Hendrycks et al., "Measuring Massive Multitask Language Understanding", ICLR 2021.
- License: MIT.
- What we include: the same 300 row positions, subject, answer letter and prompt SHA-256.

## Models (measured, not redistributed)

| Base model | GGUF files from | Base license |
|---|---|---|
| kakaocorp/kanana-1.5-2.1b-instruct-2505 | DevQuasar/kakaocorp.kanana-1.5-2.1b-instruct-2505-GGUF | Apache-2.0 |
| skt/A.X-4.0-Light | mykor/A.X-4.0-Light-gguf | Apache-2.0 |
| Qwen/Qwen3-1.7B | bartowski/Qwen_Qwen3-1.7B-GGUF | Apache-2.0 |
| Qwen/Qwen3-4B | bartowski/Qwen_Qwen3-4B-GGUF | Apache-2.0 |

`data/models.csv` lists the file names, sizes and SHA-256 we measured. Model names are used only to identify what was measured.

## llama.cpp

Measurements used llama.cpp 0.5.0 (build 11146, commit 7fe450e19), MIT license, https://github.com/ggml-org/llama.cpp. It is not included here.
