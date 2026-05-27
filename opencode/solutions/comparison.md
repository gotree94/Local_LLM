# 솔루션 종합 비교

## 1분 요약

| 용도 | 추천 솔루션 | 이유 |
|------|-----------|------|
| 🚀 **처음 시작** | **Ollama** | 30초 설치, 1분 내 실행 |
| 🖥️ **웹 UI가 필요하다면** | **Ollama + Open WebUI** | ChatGPT 스타일 인터페이스 |
| ⚡ **최고 속도** | **ExLlamaV2** | 단일 사용자 기준 가장 빠름 |
| 🏢 **다중 사용자/프로덕션** | **vLLM** | PagedAttention, Continuous Batching |
| 🔬 **실험/파인튜닝** | **text-generation-webui** | LoRA 학습, 모든 백엔드 |
| 🔄 **OpenAI 드롭인** | **LocalAI** | LLM + 이미지 + 음성 + 비전 |
| 📖 **스토리텔링** | **KoboldCPP** | World Info, 캐릭터 설정 |
| 💻 **CPU/GPU 혼합** | **llama.cpp** | 70B+ 모델 부분 오프로드 |

---

## 상세 비교표

### 기본 정보

| 항목 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI | Open WebUI |
|------|--------|------|---------------------|-----------|-----------|---------|------------|
| **언어** | Go | Python/C++ | Python | C/C++ | Python/CUDA | Go | Python/Svelte |
| **설치 난이도** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **설정 난이도** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **실행 파일 크기** | ~50MB | 수백 MB | ~2GB+ | ~10MB (바이너리) | ~200MB | ~100MB (Docker) | ~1GB (Docker) |
| **시작 시간** | 즉시 | 10-30초 | 10-30초 | 즉시 (바이너리) | 5-15초 | 10-20초 | 5-10초 |

### 성능

| 항목 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI |
|------|--------|------|---------------------|-----------|-----------|---------|
| **단일 추론 속도** | ★★★★ | ★★★★★ | ★★★★ (EXL2) / ★★★ (GGUF) | ★★★★ | **★★★★★** | ★★★ |
| **동시 처리량** | ★★ | **★★★★★** | ★★ | ★★ | ★★ | ★★ |
| **TTFT (첫 토큰)** | ★★★ | ★★★★ | ★★★★ (EXL2) / ★★★ (GGUF) | ★★★ | **★★★★★** | ★★★ |
| **메모리 효율** | ★★★ | **★★★★★** | ★★★ | ★★★★ | ★★★★ | ★★★ |
| **최대 컨텍스트** | VRAM 제한 | **PagedAttention** | 모델/백엔드 의존 | VRAM+KV 캐시 양자화 | VRAM 제한 | 모델 의존 |

### 기능

| 항목 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI | Open WebUI |
|------|--------|------|---------------------|-----------|-----------|---------|------------|
| **OpenAI API 호환** | ✅ 부분 | ✅ | ✅ 부분 | ✅ 부분 | ✅ (TabbyAPI) | **✅ 완전** | N/A (프론트) |
| **웹 UI 내장** | ❌ | ❌ | ✅ | ❌ (별도 서버) | ✅ (TabbyAPI) | ✅ 기본 | **✅ ChatGPT급** |
| **모델 다운로드** | ✅ `ollama pull` | ❌ (Hugging Face) | ✅ UI | ❌ (직접 다운로드) | ❌ | ✅ Gallery | ✅ (Ollama 연동) |
| **RAG** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | **✅ 내장** |
| **파인튜닝** | ❌ | ❌ | **✅ LoRA/QLoRA** | ❌ | ❌ | ❌ | ❌ |
| **멀티모달** | ✅ Vision | ✅ Vision | ✅ LLaVA | ❌ | ❌ | **✅ LLM+Img+Audio** | ✅ (백엔드 연동) |
| **스트리밍** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### 백엔드/포맷 지원

| 포맷 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI |
|------|--------|------|---------------------|-----------|-----------|---------|
| **GGUF** | ✅ (llama.cpp) | ❌ | ✅ | **✅ 네이티브** | ❌ | ✅ |
| **EXL2** | ❌ | ❌ | ✅ | ❌ | **✅ 네이티브** | ❌ |
| **AWQ** | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **GPTQ** | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **FP8** | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Hugging Face** | ❌ | ✅ | ✅ | ✅ (변환 필요) | ❌ | ✅ |

### RTX 5090 최적화

| 항목 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI |
|------|--------|------|---------------------|-----------|-----------|---------|
| **Blackwell 지원** | ✅ | ✅ (최신) | ✅ (EXL2) | ✅ | ✅ | ✅ |
| **FP8 Tensor Core** | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **최대 VRAM 활용** | 28-30GB | **30-31GB** | 28-30GB | 28-30GB | 28-30GB | 28-30GB |
| **70B 모드 지원** | ⚠️ Offload | ❌ | ⚠️ Offload | ✅ Offload | ❌ | ⚠️ Offload |

### 생태계

| 항목 | Ollama | vLLM | text-generation-webui | llama.cpp | ExLlamaV2 | LocalAI | Open WebUI |
|------|--------|------|---------------------|-----------|-----------|---------|------------|
| **GitHub Stars** | 130k+ | 60k+ | 50k+ | 80k+ | 10k+ | 30k+ | 80k+ |
| **커뮤니티 규모** | ★★★★★ | ★★★★ | ★★★★ | ★★★★★ | ★★★ | ★★★ | ★★★★★ |
| **문서화 수준** | ★★★★★ | ★★★★ | ★★★ | ★★★★ | ★★★ | ★★★★ | ★★★★★ |
| **업데이트 빈도** | 주간 | 주간 | 주간 | 거의 매일 | 격주 | 주간 | 주간 |
| **모델 공급** | Ollama Library | Hugging Face | Hugging Face | Hugging Face GGUF | HF EXL2 | Gallery | Ollama 연동 |

---

## 사용 사례별 추천

### 🟢 초보자 (LLM 처음)

```
1. Ollama 설치
2. ollama pull llama3.1:8b
3. ollama run llama3.1:8b
```

→ 이유: 설치 난이도 최하, 문서 방대, 문제 해결 쉬움

### 🟢 개인용 AI 어시스턴트

```
1. Ollama 설치
2. Open WebUI (Docker) 설치 → http://localhost:3000
3. Ollama에서 원하는 모델 Pull
4. 채팅 시작
```

→ 이유: ChatGPT와 동등한 UX, 모델 관리 UI 제공

### 🟡 코딩 어시스턴트 (개인)

```
1. Ollama 또는 ExLlamaV2 설치
2. Continue.dev (VS Code 확장) + Ollama 연동
3. Qwen 2.5 Coder 7B 또는 DeepSeek Coder 33B 모델 사용
```

→ 이유: 빠른 코드 생성 + IDE 통합

### 🟠 다중 사용자 서빙

```
1. vLLM 설치
2. OpenAI 호환 API 서버 실행
3. Open WebUI 또는 사용자 앱에서 API 연동
4. Continuous Batching으로 동시 처리
```

→ 이유: PagedAttention으로 10+ 동시 요청 처리 가능

### 🔬 연구/실험

```
1. text-generation-webui 설치
2. EXL2 + GGUF + AWQ 모델 번갈아 실험
3. LoRA 파인튜닝으로 모델 커스터마이징
```

→ 이유: 최대 제어, 파인튜닝, 모든 포맷 지원

### 🔄 OpenAI 마이그레이션

```
1. LocalAI 설치 (Docker)
2. 기존 OpenAI 코드의 base_url만 변경
3. LLM + 이미지 + 음성 모두 로컬에서 처리
```

→ 이유: 코드 변경 최소화, 모든 OpenAI API 호환

### 📖 창작/스토리텔링

```
1. KoboldCPP 다운로드
2. GGUF 모델 로드
3. World Info 설정 → 스토리 생성
```

→ 이유: 스토리텔링에 특화된 UI와 World Info 시스템

---

## RTX 5090 32GB 환경 추천 구성

### 구성 A: 올라운드 (추천)

```
Ollama + Open WebUI
├── Llama 3.1 8B (Q8_0)  — 빠른 일상 채팅
├── Qwen 2.5 32B (Q4_K_M) — 고품질 추론
└── DeepSeek R1 32B (Q4_K_M) — 복잡한 문제 해결
```

**설정: Docker Compose, 10분, 일상 사용에 최적**

### 구성 B: 개발자/프로덕션

```
vLLM
├── Llama 3.1 8B (FP8) — API 서빙, 다중 사용자
└── Qwen 2.5 32B (AWQ/GPTQ) — 고품질 API
```

**설정: 30분, 확장성 최우선**

### 구성 C: 연구/최고 속도

```
text-generation-webui + ExLlamaV2
├── DeepSeek R1 32B (EXL2 4.0 bpw) — 최고 속도
├── Qwen 3.6 27B (EXL2 4.0 bpw) — 장문 처리
└── Gemma 4 26B-A4B (EXL2 4.0 bpw) — 가장 긴 컨텍스트
```

**설정: 1시간+, 속도와 제어 최우선**

---

## 결정 트리

```
LLM을 로컬에서 실행하려고?
│
├── 처음 시작하는가?
│   ├── YES → Ollama
│   └── NO
│
├── 웹 UI가 필요한가?
│   ├── YES → Open WebUI (+ Ollama/vLLM)
│   └── NO
│
├── 다중 사용자/프로덕션?
│   ├── YES → vLLM
│   └── NO
│
├── 최고 추론 속도?
│   ├── YES → ExLlamaV2
│   └── NO
│
├── 실험/파인튜닝?
│   ├── YES → text-generation-webui
│   └── NO
│
├── 스토리텔링?
│   ├── YES → KoboldCPP
│   └── NO
│
└── OpenAI 드롭인?
    ├── YES → LocalAI
    └── NO → (위 중 가장 가까운 용도 선택)
```

---

## 참고 자료

- [RTX 5090 AI LLM Performance Guide (quantized.fyi)](https://quantized.fyi/hardware/rtx-5090-32gb-ai-llm-performance-guide-2026-benchmarks/)
- [Ollama vs vLLM vs Text Generation WebUI (ML Journey)](https://mljourney.com/ollama-vs-vllm-vs-text-generation-webui-which-should-you-use/)
- [GGUF vs EXL2 vs AWQ (quantized.fyi)](https://quantized.fyi/performance/gguf-vs-exl2-vs-awq-which-is-fastest-on-nvidia-in-2026/)

---

→ [메인 페이지로 돌아가기](../README.md)
