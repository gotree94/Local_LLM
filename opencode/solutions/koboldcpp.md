# KoboldCPP — 스토리텔링과 롤플레이에 특화된 LLM 런처

> **난이도**: ⭐ (매우 쉬움) | **용도**: 창작, 스토리텔링, 롤플레이

## 개요

[KoboldCPP](https://github.com/LostRuins/koboldcpp)는 llama.cpp 기반의 **올인원 LLM 실행 번들**입니다. 단일 .exe 파일(Windows) 또는 단일 스크립트(Linux)로 실행되며, 창작/스토리텔링에 최적화된 KoboldAI 기반 UI를 제공합니다.

## 주요 특징

- ✅ **단일 파일 실행** — Windows는 .exe 하나, Linux는 파이썬 스크립트
- ✅ **GGUF 네이티브** — 가장 풍부한 모델 생태계
- ✅ **KoboldAI Lite UI 내장** — 스토리텔링에 특화된 웹 인터페이스
- ✅ **GPU 레이어 오프로드** — llama.cpp 기반 GPU 가속
- ✅ **RVC/음성 지원** — 캐릭터 음성 출력
- ✅ **World Info** — 설정/지식베이스 관리
- ✅ **Text-to-Speech** — TTS 엔진 연동
- ✅ **클라우드 세이브** — Google Drive 연동 가능

## 설치 방법

### Linux

```bash
# Python 스크립트 실행
git clone https://github.com/LostRuins/koboldcpp
cd koboldcpp

# (선택) CUDA 지원 빌드
python koboldcpp.py --help

# GGUF 모델과 함께 실행
python koboldcpp.py /path/to/model.gguf
```

### Windows

1. [GitHub Releases](https://github.com/LostRuins/koboldcpp/releases)에서 `koboldcpp.exe` 다운로드
2. GGUF 모델 다운로드 (Hugging Face)
3. `koboldcpp.exe` 실행 → 모델 선택 → Start

## 기본 사용법

```bash
# 기본 실행 (GGUF 모델)
python koboldcpp.py ~/models/qwen2.5-7b-q4_k_m.gguf

# GPU 가속 실행 (RTX 5090)
python koboldcpp.py \
    ~/models/qwen2.5-7b-q4_k_m.gguf \
    --gpulayers 99 \
    --contextsize 32768

# API 서버 모드
python koboldcpp.py \
    ~/models/qwen2.5-7b-q4_k_m.gguf \
    --gpulayers 99 \
    --port 5001 \
    --host 0.0.0.0
```

### KoboldAI Lite UI

실행 후 `http://localhost:5001` 접속:

- **Story 탭** — 새로운 이야기 작성
- **Adventure 탭** — AI 던전 마스터 모드
- **Chat 탭** — 캐릭터 채팅
- **Settings** — 생성 파라미터, World Info 등

## 고급 기능

### World Info

```json
[
  {
    "name": "Character_Alice",
    "content": "Alice is a 25-year-old software engineer...",
    "keys": ["Alice", "alice", "engineer"],
    "selective": true
  },
  {
    "name": "World_Settings",
    "content": "The story takes place in a cyberpunk city...",
    "keys": ["city", "cyberpunk", "future"],
    "selective": true
  }
]
```

### API 사용

```python
import requests

API_URL = "http://localhost:5001/api/v1/generate"

response = requests.post(API_URL, json={
    "prompt": "Once upon a time,",
    "max_context_length": 8192,
    "max_length": 512,
    "temperature": 0.9,
    "top_p": 0.95,
    "rep_pen": 1.1,
})
print(response.json()["results"][0]["text"])
```

## RTX 5090 최적 설정

```bash
python koboldcpp.py \
    ~/models/deepseek-r1-32b-q4_k_m.gguf \
    --gpulayers 99 \
    --contextsize 65536 \
    --useclblast 0 0 \
    --blasthreads 8
```

## 장단점

### 장점
- **가장 쉬운 시작** — 단일 파일 + 모델만 있으면 즉시 실행
- **스토리텔링 최적화** — World Info, 캐릭터 설정 등 창작 도구
- **가벼움** — Python 환경만 있으면 실행 가능
- **RVC/TTS 지원** — 캐릭터 음성 구현 가능

### 단점
- **단일 사용자** — 다중 사용자/동시 요청 미지원
- **속도** — ExLlamaV2 대비 느림 (llama.cpp 기반)
- **창작 특화** — 일반 채팅/API 용도로는 Open WebUI가 더 나음
- **기능 제한** — vLLM의 배치 처리, 프로덕션 기능 없음

## 언제 KoboldCPP를 선택할까?

✅ **다음 경우 추천합니다:**
- AI 스토리텔링/소설 창작이 주목적인 경우
- 롤플레이 AI 던전 마스터가 필요한 경우
- 가장 간단하게 LLM을 시작하고 싶은 경우
- 캐릭터 기반 대화가 필요한 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- 일반적인 챗봇/코딩 어시스턴트 (→ Ollama + Open WebUI)
- 프로덕션 API 서빙 (→ vLLM)
- 빠른 추론 속도 (→ ExLlamaV2)

## 참고 자료

- [GitHub](https://github.com/LostRuins/koboldcpp)
- [KoboldAI Lite](https://lite.koboldai.net/)
- [KoboldAI 커뮤니티](https://koboldai.org/)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
