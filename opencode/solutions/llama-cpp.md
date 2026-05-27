# llama.cpp — 경량·최적화·범용 LLM 추론 엔진

> **난이도**: ⭐⭐ (쉬움) | **용도**: CPU/GPU 혼합 추론, GGUF 형식, 경량 서빙

## 개요

[llama.cpp](https://github.com/ggerganov/llama.cpp)는 C/C++로 작성된 **초경량 LLM 추론 엔진**입니다. Python 종속성이 전혀 없으며, GGUF 양자화 포맷의 사실상 표준 엔진입니다. CPU 최적화가 매우 뛰어나고 GPU 레이어 오프로드를 지원합니다.

## 주요 특징

- ✅ **순수 C/C++ 구현** — Python 의존성 없음, 가벼움
- ✅ **GGUF 포맷** — 가장 널리 사용되는 양자화 포맷
- ✅ **GPU 레이어 오프로드** — 일부 레이어만 GPU, 나머지는 CPU
- ✅ **CPU 최적화** — AVX2, AVX-512, NEON 등 SIMD 가속
- ✅ **KV 캐시 양자화** — Q8_0, Q4_0 등 KV 캐시 양자화 지원
- ✅ **OpenAI 호환 서버** — `llama-server` 바이너리 제공
- ✅ **스크래치 빌드** — 단일 바이너리 실행 가능
- ✅ **크로스 플랫폼** — Windows, Linux, macOS, Android (Termux)

## 설치 방법

### 1. Pre-built 바이너리

```bash
# GitHub Releases에서 최신 바이너리 다운로드
# llama-bench, llama-cli, llama-server 등 포함

# CUDA 지원 빌드
wget https://github.com/ggerganov/llama.cpp/releases/.../llama-cpp-cuda-linux-amd64.tar.xz
tar xf llama-cpp-cuda-linux-amd64.tar.xz
```

### 2. 소스 빌드 (CUDA 지원)

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# CUDA 지원 빌드
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release

# 빌드된 바이너리
./build/bin/llama-cli
./build/bin/llama-server
```

### 3. System-wide 설치

```bash
make clean && make LLAMA_CUDA=1 -j
sudo make install
```

## 기본 사용법

### CLI 모드

```bash
# GGUF 모델로 채팅
./llama-cli \
    -m models/qwen2.5-7b-instruct-q4_k_m.gguf \
    -p "Hello, how are you?" \
    -n 512 \
    -t 8 \
    -ngl 99

# -ngl 99: 모든 레이어를 GPU로 offload
# -t 8: 8개 CPU 스레드
# -n 512: 최대 512 토큰 생성
```

### 서버 모드 (OpenAI 호환)

```bash
# API 서버 실행
./llama-server \
    -m models/qwen2.5-7b-instruct-q4_k_m.gguf \
    --host 0.0.0.0 \
    --port 8080 \
    -ngl 99 \
    -c 32768

# API 호출
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5-7b-instruct",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## RTX 5090 최적 설정

```bash
# RTX 5090 32GB 최적화 실행
./llama-server \
    -m models/qwen2.5-32b-instruct-q4_k_m.gguf \
    -ngl 99 \
    -c 65536 \
    -ctk q8_0 \
    -ctv q8_0 \
    -t 8 \
    -tb 16 \
    --no-mmap \
    --n-gpu-layers 999
```

| 설정 | 설명 |
|------|------|
| `-ngl 99` | 모든 레이어 GPU offload |
| `-ctk q8_0` / `-ctv q8_0` | KV 캐시 Q8_0 양자화 (VRAM 절약) |
| `-c 65536` | 64K 컨텍스트 |
| `-t 8` | CPU 스레드 8개 |
| `--no-mmap` | 빠른 로딩 (VRAM에 직접 로드) |

### 70B 모델 부분 Offload

```bash
# 70B 모델: 32GB VRAM 부족 → 일부만 GPU
./llama-server \
    -m models/llama-3.1-70b-instruct-q4_k_m.gguf \
    -ngl 40 `# 40개 레이어만 GPU (약 24GB 사용)` \
    -c 16384 \
    -t 12
```

## 성능 (RTX 5090)

| 모델 | 양자화 | GPU 레이어 | 속도 (tok/s) |
|------|--------|-----------|-------------|
| Llama 3.1 8B | Q4_K_M | 전체 (99) | ~142 |
| Llama 3.1 8B | Q8_0 | 전체 (99) | ~118 |
| DeepSeek R1 32B | Q4_K_M | 전체 (99) | ~95 |
| Llama 3.1 70B | Q4_K_M | 40/80 레이어 | ~20-30 (GPU+CPU) |
| Llama 3.1 70B | Q2_K | 전체 (99) | ~25-35 |

## 장단점

### 장점
- **가장 가벼운 설치** — 단일 바이너리 실행 가능
- **CPU 성능 탁월** — GPU가 없어도 준수한 성능
- **GGUF 생태계** — 가장 많은 사전 양자화 모델
- **부분 오프로드** — VRAM 한계를 CPU로 극복 (70B 모델 등)
- **KV 캐시 양자화** — VRAM 부족 시 유용

### 단점
- **ExLlamaV2 대비 속도 열위** — 순수 GPU 추론에서 EXL2보다 15-25% 느림
- **동시 처리 부족** — vLLM의 Continuous Batching 없음
- **서버 기능 제한적** — API 기능이 vLLM에 비해 단순
- **양자화 품질** — 동일 비트율에서 EXL2/GPTQ 대비 미세 열위

## 언제 llama.cpp를 선택할까?

✅ **다음 경우 추천합니다:**
- CPU+GPU 혼합 환경에서 70B+ 모델을 돌려야 하는 경우
- 최소한의 의존성으로 실행해야 하는 경우
- GGUF 포맷이 가장 다양한 생태계를 가진 경우
- 서버/엣지 디바이스에서 LLM을 실행해야 하는 경우
- Ollama의 Go 래퍼가 불필요하게 느껴질 때 (직접 제어)

❌ **다음 경우 다른 솔루션을 고려하세요:**
- NVIDIA GPU에서 최고 속도가 필요할 때 (→ ExLlamaV2)
- 다중 사용자 프로덕션 환경 (→ vLLM)
- GUI 및 편의 기능이 필요할 때 (→ text-generation-webui)

## 참고 자료

- [GitHub](https://github.com/ggerganov/llama.cpp)
- [GGUF 포맷 문서](https://github.com/ggerganov/llama.cpp/tree/master/gguf)
- [TheBloke GGUF 모델](https://huggingface.co/TheBloke)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
