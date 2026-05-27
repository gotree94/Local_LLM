# 🤖 Local LLM — 로컬 LLM 구축 및 운용 프로젝트

> **ASUS ROG Strix SCAR 16 (G635LX) + Ubuntu 22.04 + NVIDIA RTX 5090 32GB**

고성능 로컬 환경에서 LLM(Large Language Model)을 구축, 실행, 운영하기 위한 종합 가이드 및 솔루션 분석 프로젝트입니다.

---

## 📋 하드웨어 사양

| 구성 요소 | 사양 |
|-----------|------|
| **CPU** | Intel Core Ultra 9 285HX (16코어 / 22스레드) |
| **GPU** | NVIDIA GeForce RTX 5090 **32GB** GDDR7 (Blackwell, 21,760 CUDA 코어) |
| **RAM** | 64GB DDR5 |
| **스토리지** | 4TB NVMe SSD |
| **OS** | Ubuntu 22.04.5 LTS (Jammy) |

> ⚠️ **참고**: RTX 5090의 실제 VRAM은 **32GB GDDR7**입니다. (명세서에 24GB로 기재되어 있다면 5080(16GB)과 혼동되었을 수 있습니다.)

→ [하드웨어 상세 분석](hardware-specs.md)

---

## 🚀 지원 가능한 모델 범위

RTX 5090 32GB는 아래 규모의 모델을 **전체 GPU 오프로드**로 구동할 수 있습니다:

| 모델 규모 | 양자화 | VRAM 사용량 | 예상 속도 |
|-----------|--------|-------------|-----------|
| 7B ~ 8B | Q8_0 | ~9GB | 110~140 tok/s |
| 13B ~ 14B | Q4_K_M | ~9GB | 100~120 tok/s |
| 30B ~ 35B | Q4_K_M | ~20GB | 85~100 tok/s |
| 70B ~ 72B | Q4_K_M | ~42GB | ❌ VRAM 부족 (CPU Offload 필요) |
| 70B ~ 72B | Q2_K | ~24GB | 25~40 tok/s (절반 CPU offload) |
| MoE 8x7B | Q4_K_M | ~18GB (active) | 70~80 tok/s |
| MoE 8x22B | Q4_K_M | ~28GB (active) | 30~40 tok/s |
| **DeepSeek R1 32B** | Q4_K_M | ~20GB | ~95 tok/s |
| **DeepSeek R1 70B** | Q4_K_M | ~42GB | ❌ (32GB 초과) |
| **Qwen 3.6 27B** | Q4 | ~31GB | ~38 tok/s @ 180K ctx |
| **Gemma 4 26B-A4B** | Q4 | ~30GB | ~62 tok/s @ 262K ctx |

→ [모델 추천 상세 보기](model-recommendations.md)

---

## 🧩 로컬 LLM 솔루션 비교

| 솔루션 | 설치 난이도 | 속도 | 동시 요청 | API 호환 | 추천 용도 |
|--------|-----------|------|----------|----------|----------|
| **[Ollama](solutions/ollama.md)** | ⭐ (매우 쉬움) | ★★★★ | 단일 사용자 | OpenAI 유사 | 개인 사용, 개발 테스트, 시작하기 |
| **[vLLM](solutions/vllm.md)** | ⭐⭐⭐⭐ | ★★★★★ | **대규모** (PagedAttention) | OpenAI 유사 | **프로덕션 서빙**, 다중 사용자 |
| **[text-generation-webui](solutions/oobabooga.md)** | ⭐⭐⭐ | ★★★★ | 단일~소규모 | OpenAI 유사 | 실험, 파인튜닝, 모델 탐색 |
| **[llama.cpp](solutions/llama-cpp.md)** | ⭐⭐ | ★★★★ | 단일~소규모 | OpenAI Lite | **CPU/GPU 혼합**, GGUF, 경량 |
| **[ExLlamaV2](solutions/exllamav2.md)** | ⭐⭐⭐⭐ | ★★★★★ | 단일 (최고 속도) | OpenAI 유사 | **추론 속도 최우선**, NVIDIA 전용 |
| **[LocalAI](solutions/localai.md)** | ⭐⭐⭐ | ★★★ | 소규모 | **OpenAI 완전 호환** | OpenAI 드롭인 대체, 멀티모달 |
| **[Open WebUI](solutions/open-webui.md)** | ⭐⭐ | N/A (프론트엔드) | N/A | Ollama/OAI 연동 | **챗봇 UI**, 지식베이스(RAG) |
| **[KoboldCPP](solutions/koboldcpp.md)** | ⭐ (매우 쉬움) | ★★★★ | 단일 | KoboldAI | **스토리텔링/게이밍**, 롤플레이 |

→ [상세 비교 보기](solutions/comparison.md)

---

## 📁 프로젝트 구조

```
D:\github\Local_LLM\
├── opencode/
│   ├── README.md                    # 메인 페이지 (현재 파일)
│   ├── hardware-specs.md            # 하드웨어 상세 분석
│   ├── setup-guide.md               # Ubuntu 22.04 CUDA + Docker + Ollama 설치
│   ├── model-recommendations.md     # RTX 5090 맞춤 모델 추천
│   │
│   └── solutions/
│       ├── comparison.md            # 솔루션 종합 비교
│       ├── ollama.md                # Ollama 상세
│       ├── vllm.md                  # vLLM 상세
│       ├── oobabooga.md             # text-generation-webui 상세
│       ├── llama-cpp.md             # llama.cpp 상세
│       ├── exllamav2.md             # ExLlamaV2 상세
│       ├── localai.md               # LocalAI 상세
│       ├── open-webui.md            # Open WebUI 상세
│       └── koboldcpp.md             # KoboldCPP 상세
│
└── (향후 실제 설치/설정 파일들)
```

---

## 🎯 시작 가이드

### 초보자 추천 경로
```
Ollama 설치 → 모델 다운로드 → Open WebUI 연결
```

### 개발자 추천 경로
```
Ollama (빠른 프로토타입) → vLLM (프로덕션)
```

### 연구자/실험자 추천 경로
```
text-generation-webui + ExLlamaV2
```

→ [Ubuntu 22.04 설치 가이드](setup-guide.md)

---

## 📊 주요 성능 지표 (RTX 5090 기준)

| 모델 | 추론 속도 (tok/s) | 최대 컨텍스트 |
|------|------------------|-------------|
| Llama 3.1 8B (Q4_K_M) | ~142 | 256K+ |
| Llama 3.1 70B (Q4_K_M, CPU offload) | ~20-40 | 128K |
| DeepSeek R1 32B (Q4_K_M) | ~95 | 128K+ |
| DeepSeek R1 70B (Q4_K_M, CPU offload) | ~15-30 | 64K |
| Mixtral 8x7B (Q4_K_M) | ~78 | 64K |
| Qwen 3.6 27B (Q4) | ~38 | ~180K |
| Gemma 4 26B-A4B (Q4) | ~62 | ~262K |

> 벤치마크 출처: quantized.fyi, localaimaster.com (2026년实测 데이터)

---

## 🔗 참고 자료

- [NVIDIA RTX 5090 공식 스펙](https://www.nvidia.com/en-gb/geforce/graphics-cards/50-series/rtx-5090/)
- [Ollama 공식 문서](https://github.com/ollama/ollama)
- [vLLM 공식 문서](https://docs.vllm.ai/)
- [llama.cpp GitHub](https://github.com/ggerganov/llama.cpp)
- [Open WebUI](https://openwebui.com/)
- [Hugging Face Models](https://huggingface.co/models)

---

## 📝 라이선스

이 프로젝트는 학습 및 연구 목적으로 작성된 문서입니다. 각 솔루션의 라이선스는 각 프로젝트의 라이선스를 따릅니다.
