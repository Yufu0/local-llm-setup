# local-llm-setup

# Hardware
OS: fedora 44
CPU: Intel Core i5-14600KF
GPU: NVIDIA GeForce RTX 4070 SUPER
RAM: 32Go DDR5 6000hz

## Install llamacpp
```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build   -DGGML_CUDA=ON  -DCMAKE_CUDA_FLAGS="--allow-unsupported-compiler"
cmake --build build -j$(nproc)
```
The binary will be in ./build/bin/*

## Download the model




## Bench the model

```bash
./llama-bench \
  -p 2048 -n 512 \
  -m path/to/model/Qwen3.6-35B-A3B-MTP-UD-Q4_K_M.gguf \
  <config>
```

###
| Configuration                          | PP (t/s)         | TG (t/s)        |
| -------------------------------------- | ---------------- | --------------- |
| `-ngl 999`                            | X                | X               |
| `-ngl 999 -ncmoe 999`                 | 519.15 ± 4.04    | 50.98 ± 0.83    |
| `-ngl 999 -ncmoe 27`                  | 690.45 ± 3.18    | 61.87 ± 0.84    |
| `-ngl 999 -ncmoe 30 -ub 2048`         | 1671.66 ± 6.41   | 59.74 ± 0.65    |
| `-ngl 999 -ncmoe 30 -ub 2048 --mmp 0` | 2196.72 ± 10.19  | 58.95 ± 1.07    |




## Run Llama server

```bash
./llama-server \
  --host 0.0.0.0 \
  --port 8000 \
  -m path/to/model/Qwen3.6-35B-A3B-MTP-UD-Q4_K_M.gguf \
  --no-mmap --mlock \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --n-gpu-layers 999 \
  -fa on \
  --n-cpu-moe 40 \
  -ub 2048 \
  -np 1 \
  --ctx-size 262144 \
  --cache-reuse 512 --cache-ram -1 --ctx-checkpoints 256 \
  --temp 0.6 --top-p 0.95 --top-k 20 --min-p 0.00 \
  --chat-template-kwargs '{"enable_thinking":false}' \
  --spec-type draft-mtp,ngram-mod \
  --spec-draft-n-max 3 --spec-draft-p-min 0.75 \
  --spec-ngram-mod-n-match 24 --spec-ngram-mod-n-min 32 --spec-ngram-mod-n-max 64 
```

