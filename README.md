---
license: cc-by-4.0
language:
- en
pretty_name: NanoBook AI Cost Studies (2026-11-05)
size_categories:
- n<1K
tags:
- llm
- local-llm
- vram
- gguf
- kv-cache
- prompt-caching
- claude
- rag
- embeddings
- cost-estimation
- tabular
- synthetic
configs:
- config_name: vram_model_quant_context
  data_files: vram-model-quant-context-2026-11-05.csv
  default: true
- config_name: claude_cache_break_even
  data_files: claude-cache-break-even-2026-11-05.csv
- config_name: rag_average_length_comparison
  data_files: rag-average-length-comparison-2026-11-05.csv
- config_name: rag_document_lengths
  data_files: rag-document-lengths-2026-11-05.csv
---

# NanoBook AI Cost Studies (2026-11-05)

Four small CSV tables produced by the AI cost calculators on [NanoBook](https://nanobook.cc/). Each table sweeps one calculator across a fixed grid of inputs so that the effect of a single assumption (model, weight quantization, context length, cache TTL, request gap, chunk overlap, document length mix) can be read row by row.

- **Study date:** 2026-11-05 (the VRAM model data was checked on 2026-11-04)
- **Total rows:** 874 (100 + 720 + 45 + 9), plus one header row per file
- **License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) for NanoBook's calculated values, schemas and documentation (see [Rights and attribution](#rights-and-attribution))
- **All values are calculated, not measured.** No model was run, no API was called and no bill was checked.

## Files

| File | Rows | Calculator | What it shows |
|---|---:|---|---|
| `vram-model-quant-context-2026-11-05.csv` | 100 | [Local LLM VRAM calculator](https://nanobook.cc/vram-calculator/) | Estimated GPU memory for 5 models × 4 weight formats × 5 context lengths |
| `claude-cache-break-even-2026-11-05.csv` | 720 | [Claude prompt caching calculator](https://nanobook.cc/prompt-caching-calculator/) | Cost of no cache vs. 5-minute vs. 1-hour cache across request counts and request gaps |
| `rag-average-length-comparison-2026-11-05.csv` | 45 | [RAG chunk & embedding calculator](https://nanobook.cc/rag-embedding-calculator/) | Error from estimating chunk counts with the average document length instead of per-document lengths |
| `rag-document-lengths-2026-11-05.csv` | 9 | [RAG chunk & embedding calculator](https://nanobook.cc/rag-embedding-calculator/) | The synthetic document length mixes used by the RAG study |

The same files are published on the calculator pages, where each study is explained in its own section: [VRAM study](https://nanobook.cc/vram-calculator/#vram-study), [cache study](https://nanobook.cc/prompt-caching-calculator/#cache-study), [RAG length study](https://nanobook.cc/rag-embedding-calculator/#length-study).

SHA-256 of each file (identical to the copies served on nanobook.cc, checked 2026-11-27):

| File | SHA-256 |
|---|---|
| `vram-model-quant-context-2026-11-05.csv` | `dc12cdcdbcf1a415defed30be09edc210972f6e68215d9f820d9174a94e01cab` |
| `claude-cache-break-even-2026-11-05.csv` | `d2a84dc37b46e82ae53122f087ea14904a2fd96c73cb1a6c0c8dbca79e88f5d3` |
| `rag-average-length-comparison-2026-11-05.csv` | `8e96552f11c374704b354c40876354a464b80cb8fb7894264eaf58bcad919948` |
| `rag-document-lengths-2026-11-05.csv` | `1f56716f20b1a64c4a1c776d3ffee06bb8f68d071e47488cde6e7243322d34ad` |

Format: UTF-8, comma-separated, every field double-quoted, CRLF line endings, one header row. Empty fields mean "not applicable" (see the `status` column of the VRAM table).

## 1. Local LLM VRAM: model × quantization × context

`vram-model-quant-context-2026-11-05.csv` — 100 rows: 5 models × 4 weight formats (`Q4_K_M`, `Q5_K_M`, `Q8_0`, `FP16`) × 5 context lengths (2,048 / 4,096 / 8,192 / 16,384 / 32,768 tokens). 90 rows are calculated; 10 rows (Qwen3 4B and Qwen3 8B at `Q5_K_M`) are marked `unavailable exact Ollama tag` and have empty size columns.

Models: Llama 3.2 3B Instruct, Qwen3 4B, Qwen3 8B, Qwen2.5 7B Instruct, Qwen2.5-Coder 7B Instruct.

Formula (bytes):

```
f16_kv_bytes         = 2 × layers × kv_heads × head_dimension × context_tokens × 2
planning_allowance   = 1.5 GiB (1,610,612,736 bytes)
total_estimate_bytes = gguf_weight_proxy_bytes + f16_kv_bytes + planning_allowance_bytes
comparison_tier_gib  = smallest of 8, 12, 16, 24, 32, 48, 64, 80 GiB that is ≥ the total
```

| Column | Description |
|---|---|
| `analysis_date` | Date the study was calculated (2026-11-05) |
| `model_data_date` | Date the model metadata was checked (2026-11-04) |
| `model_id` | NanoBook model key |
| `model` | Model display name |
| `quant` | Weight format of the Ollama tag (`Q4_K_M`, `Q5_K_M`, `Q8_0`, `FP16`) |
| `context_tokens` | Context length used for the KV cache estimate |
| `status` | `calculated, not measured`, or `unavailable exact Ollama tag` when no matching tag was found |
| `gguf_weight_proxy_bytes` | Size of the GGUF model file in bytes, used as a proxy for weight memory |
| `f16_kv_bytes` | KV cache size at F16 (2 bytes per value) for one sequence |
| `planning_allowance_bytes` | Fixed 1.5 GiB runtime allowance; uncalibrated, not a measured overhead |
| `total_estimate_bytes` | Sum of the three values above |
| `total_estimate_gib` | Total in GiB (1 GiB = 1,073,741,824 bytes), 9 decimals |
| `comparison_tier_gib` | Smallest listed GPU memory size that fits the total |
| `layers` | Number of transformer layers |
| `kv_heads` | Number of key/value attention heads |
| `head_dimension` | Dimension per attention head |
| `kv_type` | KV cache precision assumed (`F16`) |
| `sequences` | Parallel sequences assumed (`1`) |
| `gguf_digest` | SHA-256 digest of the GGUF file as listed in the Ollama manifest |
| `artifact_manifest` | Ollama registry manifest URL for the tag |
| `structure_source` | Where the layer/head values were read: the official Qwen `config.json` on Hugging Face, or for Llama 3.2 the GGUF metadata shown by Ollama |
| `scope` | Assumptions that apply to the row |

Scope: single NVIDIA GPU, full offload, dense full attention. GGUF file size is a proxy for weight memory, the allowance is uncalibrated, and there is no execution or quality guarantee. Memory used by other applications is not included.

## 2. Claude prompt caching break-even

`claude-cache-break-even-2026-11-05.csv` — 720 rows: 5 pricing variants × 6 request counts (1, 2, 3, 6, 10, 20) × 8 request gaps (1, 4.9, 5, 10, 30, 59.9, 60, 70 minutes) × 3 scenarios (`No cache`, `5m cache`, `1h cache`).

Pricing variants: `claude-sonnet-5-5`, `claude-opus-5-5` and `claude-haiku-4-5-20251001` at the standard rate with a 20,000-token prefix; `claude-haiku-5-5` with a 20,000-token prefix (`up to 100000 input tokens`) and with a 100,000-token prefix (`over 100000 input tokens`). Every request also has 1,000 new input tokens and 500 output tokens.

Model of the requests: requests are sequential and evenly spaced by `gap_minutes`, starting cold. The first request writes the fixed prefix to the cache. A later request reads the cache only if its start is strictly less than the TTL after the previous cache access; each access refreshes the TTL. Otherwise it writes again. Costs are summed in integer nano-dollars and rounded only for output.

| Column | Description |
|---|---|
| `study_date` | Date the study was calculated (2026-11-05) |
| `rate_version` | Date of the price snapshot used (2026-11-05) |
| `model` | Claude model ID |
| `price_tier` | `standard`, or the long-prompt price tier (`up to 100000 input tokens` / `over 100000 input tokens`) |
| `prefix_tokens` | Fixed prompt prefix that can be cached |
| `new_input_tokens` | Uncached input tokens per request |
| `output_tokens` | Output tokens per request |
| `requests` | Number of requests in the sequence |
| `gap_minutes` | Minutes between request starts |
| `scenario` | `No cache`, `5m cache` or `1h cache` |
| `total_usd` | Total cost of the sequence in USD (9 decimals), input + cache + output |
| `cache_writes` | Requests that wrote the prefix to the cache |
| `cache_reads` | Requests that read the prefix from the cache |
| `lowest_cost` | `true` if this scenario is the cheapest (or tied) for that model, tier, request count and gap |
| `assumption` | Assumptions that apply to the row |
| `pricing_source` | Official price page used as input |

## 3. RAG chunking: average length vs. per-document lengths

`rag-average-length-comparison-2026-11-05.csv` — 45 rows: 3 embedding models (`text-embedding-3-small`, `text-embedding-3-large`, `voyage-4`) × 5 synthetic corpora × 3 overlaps (0, 100, 200 tokens). Chunk size is 1,000 tokens and every corpus has 1,000 documents.

Each document is split independently into fixed token windows:

```
chunks           = max(1, ceil((tokens − chunk_tokens) / (chunk_tokens − overlap_tokens)) + 1)
processed_tokens = tokens + (chunks − 1) × overlap_tokens
```

The table compares the result of applying this to the exact mean length of the corpus (`average_*`) with applying it to each document (`per_document_*`).

| Column | Description |
|---|---|
| `study_date` | Date the study was calculated (2026-11-05) |
| `rate_version` | Date of the price snapshot used (2026-11-05) |
| `case_id` | Corpus ID; lengths are in `rag-document-lengths-2026-11-05.csv` |
| `model` | Embedding model ID |
| `documents` | Documents in the corpus |
| `original_tokens` | Sum of document lengths |
| `exact_mean_tokens` | Mean document length (an exact integer in every case) |
| `chunk_tokens` | Chunk window size |
| `overlap_tokens` | Overlap between consecutive chunks |
| `average_chunks` | Chunks estimated from the mean length × documents |
| `per_document_chunks` | Chunks counted per document and summed |
| `average_processed_tokens` | Tokens embedded, estimated from the mean length |
| `per_document_processed_tokens` | Tokens embedded, counted per document |
| `average_cost_usd` | Initial embedding cost from the mean-length estimate (12 decimals) |
| `per_document_cost_usd` | Initial embedding cost from per-document counting (12 decimals) |
| `signed_token_difference` | `average_processed_tokens − per_document_processed_tokens` |
| `signed_error_percent_vs_per_document` | The difference as a percentage of per-document tokens (negative = the mean underestimates) |
| `assumption` | Assumptions that apply to the row |
| `pricing_source` | Official price page used as input |

## 4. Synthetic document lengths

`rag-document-lengths-2026-11-05.csv` — 9 rows describing the 5 corpora used above. A corpus with two length groups has two rows.

| `case_id` | Label | Groups (documents × tokens) |
|---|---|---|
| `uniform` | Equal lengths | 1,000 × 1,000 |
| `balanced` | Balanced mix | 500 × 900, 500 × 1,100 |
| `long-tail` | Short majority, long tail | 900 × 100, 100 × 9,100 |
| `boundary-under` | Across the first boundary | 500 × 999, 500 × 1,001 |
| `boundary-over` | Mean just above the boundary | 500 × 1,000, 500 × 1,002 |

| Column | Description |
|---|---|
| `case_id` | Corpus ID |
| `case_label` | Short description of the corpus |
| `document_count` | Documents in this length group |
| `tokens_per_document` | Token length of each document in the group |
| `data_type` | Always `synthetic token lengths` |

## Limitations

- **Calculated, not measured.** The VRAM table is arithmetic on published file sizes and model structure values. No model was loaded on a GPU, so real memory use can differ from these estimates.
- **Synthetic token lengths.** The RAG corpora are constructed length mixes, not real documents. Token counts are not produced by any tokenizer.
- **No live API or billing test.** Cache and embedding costs are computed from official price pages as of 2026-11-05. No API request was sent and no invoice was compared. Padding, provider-added tokens and discounts are not included.
- **Point-in-time prices.** Provider prices and models can change after the study date. Check the `pricing_source` page before relying on a value.
- **Limited coverage.** Five open models, five Claude pricing variants and three embedding models. Rows are not a ranking or a recommendation.

## Rights and attribution

CC BY 4.0 applies to NanoBook's calculated results, the CSV schemas and this documentation. It does not cover material owned by others:

- Model names and trademarks (Claude, Llama, Qwen, OpenAI and Voyage model names) belong to their owners. They are used only to identify the models.
- Upstream model metadata (layer counts, KV heads, head dimensions, GGUF file sizes, digests, manifest and config URLs) comes from the model providers and from Ollama and Hugging Face. Rights in that metadata and in the model files stay with them.
- Official prices used as inputs belong to the publishing providers (Anthropic, OpenAI, Voyage AI).

No model weights, provider documents or price lists are included; only calculated values and source URLs. See [LICENSE](LICENSE) for the full license text.

Suggested citation:

> NanoBook (2026). NanoBook AI Cost Studies (study date 2026-11-05). https://nanobook.cc/ — CC BY 4.0.

Per-study citations used on the calculator pages:

- NanoBook (2026). Local LLM VRAM & F16 KV Cache Calculator and 100-Combination Benchmark Dataset (v2026.11). https://nanobook.cc/vram-calculator/
- NanoBook (2026). Claude Prompt Caching Break-Even Calculator and 720-Scenario Cost Matrix Dataset (v2026.11). https://nanobook.cc/prompt-caching-calculator/
- NanoBook (2026). RAG Chunk Overlap & Document Length Distribution Cost Calculator and Dataset (v2026.11). https://nanobook.cc/rag-embedding-calculator/

Contact: support@nanobook.cc
