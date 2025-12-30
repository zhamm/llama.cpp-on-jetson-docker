# llama.cpp-on-jetson-docker
A docker image for llama.cpp on Nvidia Jetson Orin Nano 8gb with Cuda

A **runtime-only** CUDA-enabled Docker image for running **llama.cpp** on **NVIDIA Jetson Orin Nano** with **JetPack 6 / CUDA 12.6**.

This image is intended for **inference only** and is built specifically for Jetson-class ARM64 devices using NVIDIA’s L4T stack.

---

## Docker Hub

Repository:
```
zahamm/llama-cpp-jetson
```

Docker Hub page:
```
https://hub.docker.com/r/zahamm/llama-cpp-jetson
```

---

## Available Tags

| Tag | Description |
|---|---|
| `0.1.0-runtime` | Versioned runtime release (recommended) |
| `0.1-runtime` | Compatibility alias |
| `jp6-cu126-runtime` | JetPack 6 / CUDA 12.6 runtime alias |

All runtime tags reference the **same immutable image**.

> ⚠️ This repository intentionally does **not** provide a `latest` tag.

---

## What This Image Is

- Prebuilt **llama.cpp runtime**
- CUDA backend enabled
- Optimized for Jetson Orin Nano
- Runs `llama-server`
- OpenAI-compatible HTTP API
- No compilers, no build toolchain

---

## What This Image Is NOT

- Not a development image
- Not a training image
- Not intended for x86 GPUs
- Not suitable for large models (>2GB GGUF)
- Not guaranteed to run on non-Jetson ARM devices

---

## Supported Hardware

- NVIDIA Jetson Orin Nano 8GB
- JetPack 6.x (L4T r36.x)
- CUDA 12.6
- Active cooling strongly recommended

> Note:  
> “Orin Nano Super” is not a separate SKU.  
> It refers to running the Orin Nano in MAX power mode.

---

## Quick Start (Docker Compose)

Create `docker-compose.yml`:

```yaml
services:
  llama:
    image: zahamm/llama-cpp-jetson:0.1.0-runtime
    container_name: llama_cpp
    runtime: nvidia
    network_mode: host
    restart: unless-stopped

    volumes:
      - ./models:/models:ro

    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility

    command: >
      llama-server
      --model /models/Llama-3.2-3B-Instruct-Q4_K_S.gguf
      --host 0.0.0.0
      --port 8080
      --n-gpu-layers 24
      --ctx-size 2048
      --mmap
      --temp 0.2
      --repeat_penalty 1.15
      --repeat_last_n 256
      --top_p 0.9
      --top_k 40
      --presence_penalty 0.6
      --frequency_penalty 0.8

```

Start the container:

```bash
docker compose up -d
```

Restart container after downloading new model and/or changing docker-compose.yml:

```bash
docker compose down && docker compose up -d && docker logs -f llama_cpp
```
---

## API Access

The server exposes an OpenAI-compatible API:

```
http://<jetson-ip>:8080/v1/chat/completions
```

Test endpoint:

```bash
curl http://localhost:8080/v1/models
```

Using the WebUI for Queries:

```bash
http://your-jetson-host-name:8080
```


---

## Recommended Models (GGUF)

### Known-good models for Orin Nano 8GB

**Llama 3.2 3B Instruct**
```
https://huggingface.co/bartowski/Llama-3.2-3B-Instruct-GGUF
```

Recommended quantization:
```
Q4_K_S
```

Download to models/ directory:
```
wget https://huggingface.co/bartowski/Llama-3.2-3B-Instruct-GGUF/resolve/main/Llama-3.2-3B-Instruct-Q4_K_S.gguf
```


---

**TinyLlama 1.1B Chat**
```
https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF
```

---

**unsloth/Qwen3-4B-GGUF**
```
https://huggingface.co/unsloth/Qwen3-4B-GGUF
```

Recommendations:
```
    Quantization: Q3_K_S
    GPU layers: ~22
    Context size: ≤ 2048
    Approximate RAM usage: 3–4 GB
```

---


## Models to Avoid

- GGUF files larger than ~2.2 GB
- 7B+ parameter models
- FP16 or unquantized models
- Context sizes >4096 on Nano 8GB

---

## Memory Constraints (Important)

Jetson Orin Nano uses **unified system memory** shared between CPU and GPU.

Key implications:

- CUDA allocations fragment memory quickly
- Swap does NOT help GPU allocations
- Model + KV cache + CUDA context must fit together

Practical limits on Nano 8GB:

- ~1.5–2.0 GB GGUF models
- Smaller context sizes greatly improve stability
- Reboot clears fragmented unified memory

---

## Performance Notes

Typical performance (MAX power mode):

- GPU utilization: ~95–100%
- CPU utilization: ~20–30%
- 1B models: ~20–35 tokens/sec
- Stable sustained inference

Enable MAX clocks:

```bash
sudo nvpmodel -m 0
sudo jetson_clocks
```

---

## Docker Networking Note

This container uses:

```
network_mode: host
```

Docker will **not** list exposed ports — this is expected.

---

## Tagging & Immutability Policy

- Versioned runtime tags are immutable
