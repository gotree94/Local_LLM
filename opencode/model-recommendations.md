# 모델 추천 — RTX 5090 32GB + 64GB RAM 최적화

> RTX 5090의 32GB VRAM과 64GB 시스템 RAM을 최대한 활용하는 모델 선정 가이드

---

## 핵심 원칙

1. **32GB VRAM 한계 인식** — Q4 양자화 기준 35B 모델까지 전체 GPU offload 가능
2. **70B+ 모델은 CPU offload 필요** — GGUF 포맷으로 GPU-레이어 일부 오프로드
3. **FP8 Tensor Core 활용** — RTX 5090 Blackwell은 FP8 추론 최적화
4. **MoE(Mixture of Experts) 모델 선호** — 활성 파라미터 수가 작아 VRAM 효율 우수

---

## 🏆 최고 추천 조합

### 1. 일상용 (추천)

| 순위 | 모델 | 크기 | 양자화 | VRAM | 속도 | 용도 |
|------|------|------|--------|------|------|------|
| **🥇** | **Llama 3.1 8B** | 8B | Q8_0 | ~9GB | ~118 tok/s | 일반 채팅, 빠른 작업 |
| **🥈** | **Qwen 2.5 32B** | 32B | Q4_K_M | ~20GB | ~90 tok/s | 고품질 추론 |
| **🥉** | **DeepSeek R1 32B** | 32B | Q4_K_M | ~20GB | ~95 tok/s | 복잡한 문제 해결 |

### 2. 코딩용

| 순위 | 모델 | 크기 | 양자화 | VRAM | 속도 |
|------|------|------|--------|------|------|
| **🥇** | **Qwen 2.5 Coder 32B** | 32B | Q4_K_M | ~20GB | ~90 tok/s |
| **🥈** | **DeepSeek Coder V2 Lite** | 16B | Q4_K_M | ~11GB | ~110 tok/s |
| **🥉** | **CodeLlama 34B** | 34B | Q4_K_M | ~22GB | ~85 tok/s |

### 3. 추론/분석용

| 순위 | 모델 | 크기 | 양자화 | VRAM | 속도 | 특징 |
|------|------|------|--------|------|------|------|
| **🥇** | **DeepSeek R1 32B** | 32B | Q4_K_M | ~20GB | ~95 tok/s | 강력한 추론, 사고 체인 |
| **🥈** | **Qwen 3.6 27B** | 27B | Q4 | ~31GB | ~38 tok/s | **180K 긴 컨텍스트** |
| **🥉** | **Gemma 4 26B-A4B (MoE)** | 26B/4B 활성 | Q4 | ~30GB | **~62 tok/s** | **262K 초장문 컨텍스트** |

### 4. 장문 처리용 (130K+ 컨텍스트)

| 모델 | 최대 컨텍스트 | VRAM (Q4) | 속도 |
|------|-------------|-----------|------|
| **Gemma 4 26B-A4B** | **262K (전체)** | ~30GB | ~62 tok/s |
| **Qwen 3.6 35B-A3B (MoE)** | ~210K | ~31GB | ~51 tok/s |
| **Qwen 3.6 27B** | ~180K | ~31GB | ~38 tok/s |
| **Gemma 4 31B** | ~140K | ~31GB | ~34 tok/s |

---

## VRAM 사용량 가이드

### 완전 GPU 오프로드 가능 (32GB VRAM 이내)

| 모델 규모 | Q2_K | Q3_K_M | Q4_K_M | Q5_K_M | Q8_0 |
|-----------|------|--------|--------|--------|------|
| **7B-8B** | ~3GB | ~4GB | ~5GB | ~6GB | ~9GB |
| **13B-14B** | ~5GB | ~7GB | ~9GB | ~11GB | ~15GB |
| **30B-34B** | ~12GB | ~16GB | ~20GB | ~24GB | ~36GB ❌ |
| **70B-72B** | ~24GB ⚠️ | ~32GB ❌ | ~42GB ❌ | ~52GB ❌ | ❌ |
| **MoE 8x7B** | ~10GB | ~13GB | ~18GB | ~22GB | ~30GB |
| **MoE 8x22B** | ~16GB | ~22GB | ~28GB ⚠️ | ❌ | ❌ |

> ✅ = 여유 있음, ⚠️ = 간신히 Fit, ❌ = VRAM 초과

### 부분 GPU 오프로드 (70B 모델)

70B급 모델을 RTX 5090 32GB에서 실행하려면 GPU 레이어를 제한하고 일부를 CPU에 offload 해야 합니다:

| 모델 | GPU 레이어 수 | GPU VRAM 사용 | CPU RAM 사용 | 예상 속도 |
|------|-------------|--------------|-------------|-----------|
| Llama 3.1 70B Q4_K_M | 30/80 | ~20GB | ~20GB | ~25 tok/s |
| Llama 3.1 70B Q4_K_M | 40/80 | ~26GB | ~14GB | ~30 tok/s |
| Llama 3.1 70B Q4_K_M | 50/80 | ❌ VRAM 초과 | - | - |
| Llama 3.1 70B Q2_K | 80/80 (전체) | ~24GB ✅ | ~0GB | ~30 tok/s |

---

## 양자화 포맷 선택 가이드

### GGUF (권장 — Ollama, llama.cpp, KoboldCPP)

| 수준 | 비트율 | 품질 | VRAM | 추천 용도 |
|------|--------|------|------|----------|
| Q2_K | 2.5 bpw | 낮음 | ~24GB (70B) | 70B+ 모델 간신히 Fit |
| Q3_K_M | 3.5 bpw | 중간 | 균형 | VRAM 절약 필요 시 |
| **Q4_K_M** | **4.5 bpw** | **좋음** | **균형** | **✅ 가장 추천** |
| Q5_K_M | 5.5 bpw | 매우 좋음 | 높음 | 품질 우선, VRAM 여유 있을 때 |
| Q8_0 | 8 bpw | 거의 무손실 | 매우 높음 | 최고 품질, 7B-8B 모델에 적합 |

### EXL2 (ExLlamaV2 — 최고 속도)

| 비트율 | 품질 | VRAM | GGUF Q레벨 대응 |
|--------|------|------|----------------|
| 4.0 bpw | 좋음 | ~5GB (8B) / ~20GB (32B) | Q4_K_M |
| 4.65 bpw | 매우 좋음 | ~6GB (8B) / ~24GB (32B) | Q4_K_S ~ Q5_0 |
| 5.0 bpw | 최상 | ~7GB (8B) / ~28GB (32B) | Q5_K_M |

### FP8 (vLLM — RTX 5090 Tensor Core 최적)

| 모델 | FP8 VRAM | 속도 이점 |
|------|---------|-----------|
| Llama 3.1 8B | ~10GB | Tensor Core 가속, 30-40% 속도 향상 |
| Qwen 2.5 32B | ~35GB ❌ | VRAM 부족 |

> FP8은 RTX 5090의 5세대 Tensor Core에서 최고 성능을 발휘하지만, 7-14B급 모델에 적합합니다.

---

## 특수 목적 모델

### 한국어 특화

| 모델 | 크기 | 특징 |
|------|------|------|
| **EXAONE 3.5** | 7.8B / 32B | LG AI 연구소, 한국어 특화 |
| **Llama 3.1 8B** | 8B | 한국어 성능 우수 (번역 데이터 학습) |
| **Qwen 2.5 32B** | 32B | 한국어 포함 다국어 지원 우수 |

### 멀티모달 (비전)

| 모델 | 크기 | 설명 |
|------|------|------|
| **LLaVA 1.6** | 7B / 13B / 34B | 이미지 이해 + 텍스트 추론 |
| **Qwen 2.5 VL** | 7B / 32B / 72B | 비전-언어 모델 |
| **Gemma 4 Vision** | 26B-A4B / 31B | Google 멀티모달 |

### 임베딩 (RAG 용)

| 모델 | 크기 | 설명 |
|------|------|------|
| **BGE-M3** | 567M | 다국어 임베딩, 8192 컨텍스트 |
| **jina-embeddings-v3** | 570M | 8192 컨텍스트, 태스크 특화 |
| **nomic-embed-text-v1.5** | 137M | 경량, 빠름 |

---

## Ollama 모델 설치 명령어

```bash
# 일상용
ollama pull llama3.1:8b
ollama pull llama3.3:70b  # Q2_K 양자화 (간신히 Fit)

# 코딩용
ollama pull qwen2.5-coder:7b
ollama pull deepseek-coder-v2

# 추론용
ollama pull deepseek-r1:32b
ollama pull deepseek-r1:8b  # 경량 추론

# 한국어
ollama pull exaone3.5:8b
ollama pull exaone3.5:32b

# 임베딩 (RAG)
ollama pull nomic-embed-text
ollama pull bge-m3

# 멀티모달
ollama pull llava:13b
ollama pull qwen2.5-vl:7b
```

---

## 요약: 지금 당장 시작하려면

### 초보자 추천

```bash
# 1. Ollama 설치
curl -fsSL https://ollama.com/install.sh | sh

# 2. 모델 2개만 다운로드
ollama pull llama3.1:8b     # 빠른 일상용
ollama pull deepseek-r1:8b  # 추론용

# 3. Open WebUI 설치 (Docker)
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main

# 4. http://localhost:3000 접속!
```

### 고급 사용자 추천

```bash
# Ollama + 고성능 모델
ollama pull deepseek-r1:32b   # 강력한 추론
ollama pull qwen2.5-coder:7b  # 코딩
ollama pull llama3.1:8b       # 일상
```

---

→ [메인 페이지로 돌아가기](README.md)
→ [하드웨어 상세 분석](hardware-specs.md)
→ [Ubuntu 22.04 설치 가이드](setup-guide.md)
