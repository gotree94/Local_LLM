# text-generation-webui (oobabooga) — 최대 자유도의 LLM 실험실

> **난이도**: ⭐⭐⭐ (중간) | **용도**: 모델 실험, 파인튜닝, 최대 제어

## 개요

[text-generation-webui](https://github.com/oobabooga/text-generation-webui) (일명 oobabooga)는 LLM을 실행하고 실험하기 위한 **가장 기능이 풍부한 웹 인터페이스**입니다. 모든 추론 파라미터를 세밀하게 제어할 수 있고, LoRA 파인튜닝, 채팅, 노트북 모드 등 다양한 기능을 제공합니다.

## 주요 특징

- ✅ **다중 백엔드** — Transformers, ExLlamaV2, llama.cpp, GPTQ 등
- ✅ **LoRA/QLoRA 파인튜닝** — 모델 미세 조정 지원
- ✅ **세밀한 파라미터 제어** — temperature, top_p, top_k, repetition_penalty, etc.
- ✅ **채팅/노트북/기본 모드** — 3가지 인터페이스 모드
- ✅ **멀티모달** — LLaVA, BakLLaVA 등 비전 모델 지원
- ✅ **확장 시스템** — 사용자 확장 기능 지원
- ✅ **OpenAI API** — OpenAI 호환 API 모드 지원
- ✅ **캐릭터 프롬프트** — 캐릭터 카드 기반 롤플레이 지원
- ✅ **GGUF/EXL2/AWQ/GPTQ** — 거의 모든 양자화 형식 지원

## 설치 방법

```bash
# Clone 및 설치
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui

# Linux 설치 스크립트
./start_linux.sh

# 또는 pip 직접 설치
pip install -r requirements.txt
pip install -r requirements_cuda.txt
```

## 기본 사용법

### 실행

```bash
# 기본 실행 (채팅 모드)
python server.py --listen

# ExLlamaV2 백엔드 사용 (최고 속도)
python server.py --listen \
    --model deepseek-r1-32b-q4 \
    --loader exllamav2 \
    --max_seq_len 32768

# AutoGPTQ 백엔드
python server.py --listen \
    --model model-name \
    --loader autogptq \
    --gpu-split 32
```

### 웹 인터페이스

실행 후 `http://localhost:7860` 접속:

- **Chat 탭** — 일반 채팅 인터페이스 (캐릭터 설정 가능)
- **Notebook 탭** — 연속 텍스트 생성 (소설/에세이 작성에 적합)
- **Default 탭** — 단일 프롬프트 응답
- **Parameters 패널** — temperature, top_p, top_k, mirostat 등 전역 제어
- **Model 탭** — 모델 로드/언로드, 백엔드 전환
- **Training 탭** — LoRA/QLoRA 파인튜닝
- **Extensions** — 확장 기능 관리

## 주요 기능 상세

### LoRA 파인튜닝 (4bit QLoRA)

```python
# Training 탭에서 설정 가능한 주요 파라미터:
# - 학습률: 1e-4 ~ 2e-4
# - LoRA Rank: 8 ~ 128
# - LoRA Alpha: 16 ~ 256
# - 에포크: 1 ~ 5
# - 배치 크기: 2 ~ 8 (RTX 5090 기준 8 충분)
# - 데이터셋: Alpaca 형식 JSON
```

### 캐릭터 프롬프트 예시

```yaml
name: "Assistant"
context: |
  You are a helpful, harmless, and honest AI assistant.
  You respond in Korean when the user speaks Korean.
greeting: "안녕하세요! 무엇을 도와드릴까요?"
```

## RTX 5090 최적 설정

```bash
# ExLlamaV2 (최고 속도)
python server.py --listen \
    --loader exllamav2 \
    --gpu-split 32 \
    --max_seq_len 65536 \
    --cache_4bit \
    --cache_8bit \
    --no_flash_attn
```

| 설정 | 설명 |
|------|------|
| `loader exllamav2` | NVIDIA GPU 최고 속도 추론 |
| `gpu-split 32` | 32GB 전체 GPU 할당 |
| `max_seq_len 65536` | 64K 컨텍스트 |
| `--listen` | 외부 접속 허용 |

## 장단점

### 장점
- **가장 풍부한 기능** — 파인튜닝, 임베딩, 확장 등 올인원
- **모든 백엔드 지원** — EXL2, GGUF, GPTQ, AWG 등 상황에 맞게 선택
- **완전한 제어** — 모든 추론 파라미터 세밀 조정
- **캐릭터/롤플레이** — 고급 프롬프트 엔지니어링 지원
- **학습 기능 내장** — 별도 학습 파이프라인 없이 LoRA 학습 가능

### 단점
- **설치 복잡** — 의존성 지옥 (CUDA, PyTorch 버전 호환성)
- **UI 과부하** — 기능이 많아 초보자에게 복잡
- **메모리 효율** — vLLM 대비 동시 처리 효율 낮음
- **업데이트 주기** — 활발하나 가끔 호환성 깨짐

## 언제 text-generation-webui를 선택할까?

✅ **다음 경우 추천합니다:**
- 다양한 모델/양자화 형식을 실험해야 하는 경우
- LoRA 파인튜닝이 필요한 경우
- 캐릭터 기반 롤플레이/스토리텔링
- 최대한의 제어가 필요한 연구/실험
- GGUF와 EXL2를 상황에 따라 번갈아 사용해야 하는 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- 간단한 LLM 사용만 필요한 경우 (→ Ollama)
- 프로덕션 서빙이 목적인 경우 (→ vLLM)
- 설치를 최대한 간단하게 하고 싶은 경우 (→ Ollama)

## 참고 자료

- [GitHub](https://github.com/oboobaga/text-generation-webui)
- [위키](https://github.com/oboobaga/text-generation-webui/wiki)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
