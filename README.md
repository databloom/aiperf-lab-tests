# aiperf-lab-tests

Benchmark artifacts collected from AIPerf runs against lab inference endpoints.

## Included runs

- `qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1` on `wintermute` (`192.168.19.150:8080`)
- `llama-3.2-3b-instruct-gguf:q4_k_m` multi-endpoint run (`wintermute` + `molly`)

## Host Performance Summary

Latest single-host benchmark runs using `aiperf profile --request-count 20 --streaming --ui-type simple`:

| Host | Endpoint | Model | Successful Requests | Error Requests | Request Throughput (req/s) | Output Token Throughput (tokens/s) | TTFT Avg (ms) | Request Latency Avg (ms) | Benchmark Duration (s) |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `wintermute` | `192.168.19.150:8080` | `qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1` | 15 | 5 | 0.05 | 12.95 | 516.39 | 8066.00 | 274.14 |
| `molly` | `192.168.19.9:8080` | `qwen_qwen3.5-4b-gguf:q4_k_m` | 0 | 20 | N/A | N/A | N/A | N/A | N/A |

## Contents

- `artifacts/`: AIPerf exports (`profile_export_aiperf.csv`, `profile_export_aiperf.json`, logs)
- `inputs.jsonl`: prompts used for custom single-turn dataset runs
- `TESTING.md`: full reproduction commands, environment setup, and extended RAG testing notes
- `RESULTS.md`: extracted benchmark performance metrics table with artifact pointers

