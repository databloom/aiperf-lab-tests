# Benchmark Results

This file contains the extracted performance numbers from the latest test runs.

## Latest Runs

| Run | Host(s) | Model | Request Count | Error Count | Request Throughput (req/s) | Output Token Throughput (tokens/s) | TTFT Avg (ms) | Request Latency Avg (ms) | Duration (s) |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Wintermute single-host | `192.168.19.150:8080` | `qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1` | 15 | 5 | 0.05 | 12.95 | 516.39 | 8066.00 | 274.14 |
| Molly single-host | `192.168.19.9:8080` | `qwen_qwen3.5-4b-gguf:q4_k_m` | 0 | 20 | N/A | N/A | N/A | N/A | N/A |
| Dual-host | `192.168.19.150:8080` + `192.168.19.9:8080` | `llama-3.2-3b-instruct-gguf:q4_k_m` | 10 | 10 | 0.05 | 0.58 | 389.57 | 617.37 | 191.24 |

## Artifact Sources

Metrics above are taken from the exported CSV/JSON files:

- `artifacts/qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1-openai-chat-concurrency1/profile_export_aiperf.csv`
- `artifacts/qwen_qwen3.5-4b-gguf:q4_k_m-openai-chat-concurrency1/profile_export_aiperf.csv`
- `artifacts/llama-3.2-3b-instruct-gguf:q4_k_m-openai-chat-concurrency1/profile_export_aiperf.csv`

For full metric distributions and logs, use:

- `profile_export_aiperf.json`
- `profile_export.jsonl`
- `logs/aiperf.log`

