# LocalAI — OpenAI 완전 호환 로컬 드롭인 대체

> **난이도**: ⭐⭐⭐ (중간) | **용도**: OpenAI 드롭인 대체, 멀티모달, 올인원

## 개요

[LocalAI](https://localai.io)는 OpenAI API의 **완전한 드롭인 대체**를 목표로 하는 로컬 AI 서버입니다. 텍스트 LLM뿐 아니라 이미지 생성(Stable Diffusion), 음성(STT/TTS), 임베딩, 비전(LLaVA)까지 하나의 서버로 제공합니다.

## 주요 특징

- ✅ **OpenAI API 완전 호환** — 코드 한 줄 변경 없이 전환
- ✅ **멀티모달** — 텍스트 + 이미지 생성 + 음성 + 비전
- ✅ **다중 백엔드** — llama.cpp, transformers, diffusers 등
- ✅ **바이너리 배포** — Docker 또는 단일 바이너리 실행
- ✅ **RAG 지원** — 벡터 DB 연동 (LlamaIndex, Chroma 등)
- ✅ **Galería** — 모델 스토어 (1클릭 모델 설치)
- ✅ **GPU 가속** — CUDA, OpenCL, Vulkan 지원
- ✅ **WebUI 내장** — 기본 채팅 UI 제공

## 설치 방법

### Docker (권장)

```bash
# CUDA 지원 Docker 실행
docker run -d \
    --gpus all \
    -p 8080:8080 \
    -v $PWD/models:/build/models \
    -v $PWD/images:/build/images \
    -e DEBUG=true \
    -e CUDA=true \
    localai/localai:latest-gpu-cuda-12

# 또는 All-in-One (모델 포함)
docker run -d \
    --gpus all \
    -p 8080:8080 \
    localai/localai:latest-aio-gpu-cuda-12
```

### 바이너리 설치

```bash
# Linux 직접 설치
curl -o- https://localai.io/install.sh | bash

# CUDA 지원 바이너리 실행
./localai
```

## 기본 사용법

### API 호출 (OpenAI 호환)

```python
from openai import OpenAI

# LocalAI는 OpenAI 호환 API 제공
client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="not-needed",
)

# 채팅
response = client.chat.completions.create(
    model="gpt-4",  # LocalAI 모델 이름 (자유롭게 매핑)
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)

# 이미지 생성
response = client.images.generate(
    model="stablediffusion",
    prompt="a cat wearing a hat",
    n=1,
    size="1024x1024",
)

# 음성-텍스트 (Whisper)
with open("audio.mp3", "rb") as f:
    transcript = client.audio.transcriptions.create(
        model="whisper-1",
        file=f,
    )
```

### Model Gallery

```bash
# 모델 목록 확인
curl http://localhost:8080/v1/models

# CLI로 모델 설치
localai models install llama-3.1-8b-instruct
localai models install stablediffusion
localai models install whisper-1
```

## 설정 파일 (models.yaml)

```yaml
# models.yaml - 모델 설정
name: llama-3.1-8b-instruct
backend: llama-cpp
parameters:
  model: /build/models/llama-3.1-8b-instruct-q4_k_m.gguf
  context_size: 32768
  n_gpu_layers: 99
  f16: true
  mmap: true
---
name: stablediffusion
backend: diffusers
parameters:
  model: stabilityai/stable-diffusion-xl-base-1.0
---
name: whisper-1
backend: whisper
parameters:
  model: base
```

## RTX 5090 최적 설정

```yaml
# docker-compose.yml
services:
  localai:
    image: localai/localai:latest-gpu-cuda-12
    ports:
      - "8080:8080"
    environment:
      - DEBUG=true
      - CUDA=true
      - REBUILD=true
      - THREADS=16
    volumes:
      - ./models:/build/models
      - ./images:/build/images
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

실행:

```bash
docker compose up -d
```

## 장단점

### 장점
- **올인원 AI 서버** — LLM + 이미지 + 음성 + 비전
- **OpenAI 100% 호환** — 기존 OpenAI 코드를 수정 없이 사용
- **모델 스토어** — GUI로 모델 관리
- **컨테이너화** — Docker로 완전히 격리된 환경

### 단점
- **백엔드 오버헤드** — 다중 백엔드 추상화로 인한 속도 저하 (vLLM 대비 ~200ms 레이턴시)
- **복잡한 설정** — 다양한 모델 관리가 까다로움
- **처리량 제한** — Ollama와 유사한 수준 (vLLM에 비해 낮음)
- **리소스 사용** — 모든 기능 활성화 시 상당한 디스크 공간 필요

## 언제 LocalAI를 선택할까?

✅ **다음 경우 추천합니다:**
- 기존 OpenAI 기반 앱을 로컬로 전환해야 하는 경우
- LLM + 이미지 + 음성 등 멀티모달이 필요한 경우
- 코드 변경 없이 OpenAI → 로컬 전환이 목적인 경우
- RAG 시스템 구축이 필요한 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- LLM 추론 속도만 최우선인 경우 (→ ExLlamaV2 / vLLM)
- 간단한 LLM 채팅만 필요한 경우 (→ Ollama + Open WebUI)
- 프로덕션 고처리량 서빙 (→ vLLM)

## 참고 자료

- [공식 문서](https://localai.io)
- [GitHub](https://github.com/mudler/LocalAI)
- [Model Gallery](https://github.com/mudler/LocalAI/tree/master/models)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
