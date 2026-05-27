# vLLM — 프로덕션급 고성능 LLM 서빙 엔진

> **난이도**: ⭐⭐⭐⭐ (중상) | **용도**: 고처리량 프로덕션 서빙, 다중 사용자

## 개요

[vLLM](https://docs.vllm.ai)은 UC Berkeley에서 개발된 **고성능 LLM 추론 엔진**입니다. 핵심 기술인 **PagedAttention**을 통해 VRAM 활용도를 극대화하고, 지속적 배치(Continuous Batching)로 동시 요청 처리량을 획기적으로 개선했습니다.

## 주요 특징

- ✅ **PagedAttention** — KV 캐시를 페이지 단위로 관리, VRAM 단편화 제거
- ✅ **Continuous Batching** — 동시 요청을 동적으로 배치 처리
- ✅ **OpenAI 호환 API** — `/v1/chat/completions` 완벽 지원
- ✅ **Prefix Caching** — 공통 프롬프트 접두사 캐싱
- ✅ **양자화 지원** — AWQ, GPTQ, FP8 (RTX 5090 네이티브)
- ✅ **멀티 GPU** — Tensor Parallelism 지원
- ✅ **스트리밍** — SSE 기반 스트리밍 응답
- ✅ **모니터링** — Prometheus 메트릭 연동

## 설치 방법

```bash
# pip 설치 (CUDA 12.4 기준)
pip install vllm

# 또는 소스 빌드 (최신 기능)
git clone https://github.com/vllm-project/vllm.git
cd vllm
pip install -e .
```

## 기본 사용법

### Python API

```python
from vllm import LLM, SamplingParams

# 모델 로드
llm = LLM(
    model="meta-llama/Meta-Llama-3.1-8B-Instruct",
    tensor_parallel_size=1,  # 단일 GPU
    max_model_len=32768,
    gpu_memory_utilization=0.95,
)

# 추론 파라미터 설정
sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.9,
    max_tokens=1024,
)

# 추론 실행
outputs = llm.generate(["Hello, how are you?"], sampling_params)
for output in outputs:
    print(output.outputs[0].text)
```

### OpenAI 호환 서버

```bash
# OpenAI 호환 API 서버 실행
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Meta-Llama-3.1-8B-Instruct \
    --port 8000 \
    --max-model-len 32768 \
    --gpu-memory-utilization 0.95

# API 호출
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

## 고급 설정

### 양자화 모델 사용 (FP8 — RTX 5090 최적)

```bash
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Meta-Llama-3.1-8B-Instruct \
    --quantization fp8 \
    --kv-cache-dtype fp8 \
    --max-model-len 65536
```

### Docker 배포

```bash
docker run --gpus all \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -p 8000:8000 \
  vllm/vllm-openai:latest \
  --model meta-llama/Meta-Llama-3.1-8B-Instruct
```

## 성능 (Ollama vs vLLM)

| 지표 | Ollama | vLLM | 향상 |
|------|--------|------|------|
| 단일 요청 (7B) | 45 t/s | 48 t/s | +7% |
| **10 동시 요청** | ~6 t/s **각** | ~35 t/s **각** | **+483%** |
| 시간당 처리량 | ~500 요청 | ~5,000+ 요청 | 10x |
| 메모리 효율 KV Cache | 기준 | 2.5-3x | +150-200% |
| 최대 컨텍스트 길이 | VRAM 제한 | 2-3x 더 김 | +100-200% |

## RTX 5090 최적 설정

RTX 5090의 32GB VRAM과 FP8 Tensor Core를 최대한 활용하는 설정:

```bash
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Meta-Llama-3.1-8B-Instruct \
    --quantization fp8 \
    --kv-cache-dtype fp8 \
    --max-model-len 131072 \
    --gpu-memory-utilization 0.96 \
    --max-num-seqs 256 \
    --enable-prefix-caching \
    --enable-chunked-prefill
```

| 설정 | 설명 |
|------|------|
| `quantization fp8` | FP8 양자화 (RTX 5090 Tensor Core 최적화) |
| `kv-cache-dtype fp8` | KV 캐시도 FP8 (VRAM 2배 절약) |
| `gpu-memory-utilization 0.96` | VRAM 96% 사용 (약 30.7GB) |
| `max-num-seqs 256` | 최대 동시 시퀀스 수 |
| `enable-prefix-caching` | 시스템 프롬프트 재사용 시 캐싱 |

## 장단점

### 장점
- **압도적인 동시 처리량** — 단일 GPU로 수십 명 동시 서빙
- **메모리 효율** — PagedAttention으로 같은 VRAM으로 2-3배 긴 컨텍스트
- **프로덕션 레디** — 모니터링, 로드밸런싱, 서빙 메트릭
- **FP8 최적화** — RTX 5090 Tensor Core 완벽 활용

### 단점
- **설정 복잡** — 다양한 플래그와 옵션 이해 필요
- **설치 시간** — 빌드 의존성, CUDA 버전 호환성 주의
- **오버헤드** — 단일 사용자 단순 추론에는 Ollama보다 느릴 수 있음
- **모델 호환성** — 모든 모델이 vLLM에서 지원되지는 않음

## 언제 vLLM을 선택할까?

✅ **다음 경우 vLLM을 추천합니다:**
- 다중 사용자/동시 요청이 필요한 경우
- 프로덕션 환경 배포가 목적인 경우
- 제한된 VRAM으로 최대 컨텍스트를 확보해야 하는 경우
- 확장성을 고려한 아키텍처가 필요한 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- 단순한 개인 채팅만 필요한 경우 (→ Ollama)
- 모델 탐색/실험이 주목적인 경우 (→ text-generation-webui)
- CPU-only 환경인 경우 (→ llama.cpp)

## 참고 자료

- [공식 문서](https://docs.vllm.ai)
- [GitHub](https://github.com/vllm-project/vllm)
- [PagedAttention 논문](https://arxiv.org/abs/2309.06180)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
