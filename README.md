# aiperf-lab-tests

Benchmark artifacts collected from AIPerf runs against lab inference endpoints.

## Included runs

- `qwen3-4b-qwen3.6-plus-reasoning-distilled-gguf:q4_1` on `wintermute` (`192.168.19.150:8080`)
- `llama-3.2-3b-instruct-gguf:q4_k_m` multi-endpoint run (`wintermute` + `molly`)

## Contents

- `artifacts/`: AIPerf exports (`profile_export_aiperf.csv`, `profile_export_aiperf.json`, logs)
- `inputs.jsonl`: prompts used for custom single-turn dataset runs

