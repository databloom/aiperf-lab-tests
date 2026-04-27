# AIPerf Test Reproduction Guide

This document captures exactly how benchmark tests were run for this repo.

## Test Environment

- Host running AIPerf: `heimdall`
- Inference hosts:
  - `wintermute` -> `192.168.19.150:8080`
  - `molly` -> `192.168.19.9:8080`
- AIPerf source checkout: `/home/mike/performancetest/aiperf`
- Artifact repo: `/home/mike/performancetest/aiperf-lab-tests`

## Prerequisites

```bash
cd /home/mike/performancetest
git clone https://github.com/ai-dynamo/aiperf.git
curl -LsSf https://astral.sh/uv/install.sh | sh
~/.local/bin/uv python install 3.10
~/.local/bin/uv venv --clear --python 3.10 /home/mike/performancetest/aiperf/.venv
~/.local/bin/uv pip install --python /home/mike/performancetest/aiperf/.venv/bin/python -e /home/mike/performancetest/aiperf
```

## Input Dataset Used

Create `inputs.jsonl`:

```bash
cat > /home/mike/performancetest/aiperf/inputs.jsonl <<'EOF'
{"texts":["Hello! Please respond with a short sentence."]}
{"texts":["What is 2 + 2? Reply with one number only."]}
{"texts":["Write a one-line haiku about GPUs."]}
{"texts":["Return the single word: ready"]}
EOF
```

## Endpoint and Cluster Health Checks

```bash
kubectl get nodes -o wide
curl -sS --max-time 10 http://192.168.19.150:8080/v1/models
curl -sS --max-time 10 http://192.168.19.9:8080/v1/models
```

## Full Commands Used for Benchmarks

All runs below use `--ui-type simple` to force consistent CLI progress formatting and final summary output.

### 1) Wintermute-only (Qwen3.6 distilled, GGUF q4_1)

```bash
/home/mike/performancetest/aiperf/.venv/bin/aiperf profile \
  --model qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1 \
  --tokenizer Qwen/Qwen3-4B \
  --endpoint-type chat \
  --endpoint /v1/chat/completions \
  --streaming \
  --ui-type simple \
  --input-file /home/mike/performancetest/aiperf/inputs.jsonl \
  --custom-dataset-type single_turn \
  --request-count 20 \
  --request-timeout-seconds 30 \
  --url http://192.168.19.150:8080
```

### 2) Molly-only (Qwen3.5 GGUF q4_k_m)

```bash
/home/mike/performancetest/aiperf/.venv/bin/aiperf profile \
  --model qwen_qwen3.5-4b-gguf:q4_k_m \
  --tokenizer Qwen/Qwen3-4B \
  --endpoint-type chat \
  --endpoint /v1/chat/completions \
  --streaming \
  --ui-type simple \
  --input-file /home/mike/performancetest/aiperf/inputs.jsonl \
  --custom-dataset-type single_turn \
  --request-count 20 \
  --request-timeout-seconds 30 \
  --url http://192.168.19.9:8080
```

### 3) Dual-host run (Llama 3.2 3B)

```bash
/home/mike/performancetest/aiperf/.venv/bin/aiperf profile \
  --model llama-3.2-3b-instruct-gguf:q4_k_m \
  --tokenizer gpt2 \
  --endpoint-type chat \
  --endpoint /v1/chat/completions \
  --streaming \
  --ui-type simple \
  --input-file /home/mike/performancetest/aiperf/inputs.jsonl \
  --custom-dataset-type single_turn \
  --request-count 20 \
  --request-timeout-seconds 20 \
  --url http://192.168.19.150:8080 \
  --url http://192.168.19.9:8080
```

## Where to Find Results

AIPerf writes outputs under:

```text
/home/mike/performancetest/aiperf/artifacts/<run-name>/
```

Each run includes:

- `profile_export_aiperf.csv`
- `profile_export_aiperf.json`
- `logs/aiperf.log`

## Extended: RAG Endpoint Testing

AIPerf supports RAG-style benchmarking in two useful ways:

1. Built-in `solido_rag` endpoint type
2. Generic `template` endpoint type for custom APIs (for example, chains servers)

### A) Built-in SOLIDO RAG Endpoint

`solido_rag` is defined in AIPerf plugins with default path `/rag/api/prompt`.

```bash
/home/mike/performancetest/aiperf/.venv/bin/aiperf profile \
  --model your-rag-model-name \
  --endpoint-type solido_rag \
  --url http://<rag-host>:<port> \
  --request-count 20 \
  --streaming \
  --input-file /home/mike/performancetest/aiperf/inputs.jsonl \
  --custom-dataset-type single_turn
```

Optional request fields can be passed via `--extra-inputs`, for example:

```bash
--extra-inputs filters:'{"family":"Solido","tool":"SDE"}'
```

### B) Custom RAG / Chains API via Template Endpoint

Use `--endpoint-type template` and provide at least:

- `payload_template` (Jinja2 JSON payload template)
- optional `response_field` (JMESPath selector to extract generated text)

Example command:

```bash
/home/mike/performancetest/aiperf/.venv/bin/aiperf profile \
  --model my-chains-model \
  --endpoint-type template \
  --endpoint /v1/chains/query \
  --url http://<chains-host>:<port> \
  --request-count 20 \
  --streaming \
  --input-file /home/mike/performancetest/aiperf/inputs.jsonl \
  --custom-dataset-type single_turn \
  --extra-inputs payload_template:'{"query": {{ text|tojson }}, "stream": {{ stream|tojson }}, "model": {{ model|tojson }}}' \
  --extra-inputs response_field:'answer'
```

Another common response selector pattern:

```bash
--extra-inputs response_field:'result.text'
```

### RAG Validation Tips

- Start with `--request-count 2` to validate payload/response parsing.
- If responses are non-OpenAI JSON, use `template` + `response_field`.
- Keep timeout explicit for long retrieval/inference chains:
  - `--request-timeout-seconds 30` (or higher)
- Run single-host first, then multi-url once stable.

