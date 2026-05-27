# ExLlamaV2 — NVIDIA GPU에서 가장 빠른 LLM 추론

> **난이도**: ⭐⭐⭐⭐ (중상) | **용도**: 속도 최우선 추론, NVIDIA GPU 최적화

## 개요

[ExLlamaV2](https://github.com/turboderp/exllamav2)는 NVIDIA GPU에서 **단일 사용자 추론 속도가 가장 빠른** LLM 추론 엔진입니다. EXL2 양자화 포맷을 사용하며, CUDA 커널이 NVIDIA GPU에 극도로 최적화되어 있습니다. 최근 v0.5.x부터 Blackwell (RTX 50 시리즈) 공식 지원이 추가되었습니다.

## 주요 특징

- ✅ **최고 추론 속도** — GGUF 대비 15-25%, AWQ 대비 30%+ 빠름
- ✅ **EXL2 양자화** — 레이어별 민감도 기반 가변 비트율 (bpw)
- ✅ **낮은 TTFT (Time To First Token)** — 1K 프롬프트에서 ~138ms
- ✅ **FlashAttention** — 긴 컨텍스트에서 빠른 어텐션
- ✅ **FP8 KV 캐시** — VRAM 절약 및 속도 향상
- ✅ **Blackwell 최적화** — RTX 50 시리즈 전용 CUDA 커널
- ✅ **OpenAI 호환** — TabbyAPI 등 연동으로 API 서빙 가능

## 설치 방법

```bash
git clone https://github.com/turboderp/exllamav2
cd exllamav2
pip install -r requirements.txt
pip install -e .
```

## 기본 사용법

### Python API

```python
import torch
from exllamav2 import (
    ExLlamaV2Config,
    ExLlamaV2Tokenizer,
    ExLlamaV2,
    ExLlamaV2Cache_8bit,
)
from exllamav2.generator import (
    ExLlamaV2StreamingGenerator,
    ExLlamaV2Sampler,
)

# 모델 설정
config = ExLlamaV2Config()
config.model_dir = "/path/to/exl2-model"
config.prepare()

# 모델 로드 (8bit 캐시 = VRAM 절약)
model = ExLlamaV2(config)
cache = ExLlamaV2Cache_8bit(model)
model.load_autosplit(cache)

# 토크나이저
tokenizer = ExLlamaV2Tokenizer(config)

# 생성기
generator = ExLlamaV2StreamingGenerator(model, cache)

# 샘플링 설정
settings = ExLlamaV2Sampler.Settings()
settings.temperature = 0.7
settings.top_k = 50
settings.top_p = 0.9
settings.token_repetition_penalty = 1.1

# 추론
prompt = "Hello, how are you?"
input_ids = tokenizer.encode(prompt)
generator.set_stop_conditions([])

# 스트리밍 생성
generator.begin_stream(input_ids, settings)
for i in range(512):
    chunk, eos, _ = generator.stream()
    print(chunk, end="", flush=True)
    if eos: break
```

### TabbyAPI를 통한 OpenAI 호환 서버

TabbyAPI는 ExLlamaV2 기반의 OpenAI 호환 API 서버입니다.

```bash
# TabbyAPI 설치
git clone https://github.com/theroyallab/tabbyAPI
cd tabbyAPI
pip install -r requirements.txt

# 설정 (config.yml)
# 모델 경로, 캐시 설정 등

# 실행
python main.py
```

## 성능 (RTX 5090, Qwen3-8B 4bit 기준)

| 포맷 | 엔진 | 속도 (t/s) | TTFT (1K) | GGUF 대비 |
|------|------|-----------|-----------|-----------|
| **EXL2 4.0 bpw** | **ExLlamaV2** | **205.4** | **138ms** | **+24%** |
| GPTQ-Marlin | vLLM | 178.2 | 149ms | +8% |
| GGUF Q4_K_M | llama.cpp | 165.1 | 248ms | 기준 |
| AWQ | vLLM | 155.3 | 176ms | -6% |

### 모델별 성능

| 모델 | 양자화 | VRAM | 속도 (tok/s) |
|------|--------|------|-------------|
| Qwen3-8B | EXL2 4.0 bpw | ~6GB | ~205 |
| DeepSeek R1 32B | EXL2 4.0 bpw | ~20GB | ~105 |
| Mistral 7B | EXL2 4.0 bpw | ~5GB | ~220 |
| Gemma 4 26B-A4B | EXL2 4.0 bpw | ~16GB | ~175 |

## EXL2 모델 구하기

```bash
# Hugging Face에서 EXL2 양자화 모델 검색
# https://huggingface.co/models?search=exl2

# 또는 직접 양자화 (긴 과정, 경험 필요)
python convert.py -i /path/to/hf-model -o /path/to/output -b 4.0
```

## 장단점

### 장점
- **단일 사용자 기준 최고 속도** — NVIDIA에서 가장 빠름
- **낮은 TTFT** — 첫 토큰까지 대기 시간 최소
- **정교한 양자화** — 레이어별 민감도 기반 비트 할당
- **Blackwell 지원** — RTX 5090 완전 활용

### 단점
- **설치 복잡** — CUDA 커널 빌드 필요
- **모델 공급 제한** — EXL2 양자화 모델이 GGUF보다 적음
- **신규 모델 지원 지연** — 새 모델 EXL2 양자화까지 수 주 소요
- **단일 사용자 한정** — vLLM의 동시 처리 기능 없음
- **NVIDIA 전용** — AMD/Intel GPU 미지원

## 언제 ExLlamaV2를 선택할까?

✅ **다음 경우 추천합니다:**
- 단일 사용자 기준 최고 추론 속도가 필요한 경우
- 연구/실험에서 빠른 피드백이 중요한 경우
- NVIDIA GPU 전용 환경인 경우
- TTFT가 중요한 실시간 애플리케이션

❌ **다음 경우 다른 솔루션을 고려하세요:**
- 다중 사용자 서빙 환경 (→ vLLM)
- 다양한 모델을 자주 바꿔가며 사용해야 할 때 (→ Ollama/llama.cpp)
- CPU에서도 실행해야 하는 경우 (→ llama.cpp)
- 설치 용이성이 가장 중요한 경우 (→ Ollama)

## 참고 자료

- [GitHub](https://github.com/turboderp/exllamav2)
- [TabbyAPI](https://github.com/theroyallab/tabbyAPI)
- [EXL2 모델 (Hugging Face)](https://huggingface.co/models?search=exl2)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
