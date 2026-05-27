# Open WebUI — 로컬 LLM을 위한 ChatGPT 스타일 인터페이스

> **난이도**: ⭐⭐ (쉬움) | **용도**: Ollama/vLLM용 웹 UI, RAG, 지식베이스

## 개요

[Open WebUI](https://openwebui.com)는 로컬 LLM을 위한 **가장 세련된 웹 인터페이스**입니다. Ollama, OpenAI 호환 서버(vLLM, LocalAI 등)와 연동하여 ChatGPT와 유사한 사용자 경험을 제공합니다. RAG(Retrieval-Augmented Generation), 지식베이스 구축, 다중 사용자 관리까지 지원합니다.

## 주요 특징

- ✅ **직관적인 UI** — ChatGPT와 유사한 대화 인터페이스
- ✅ **Ollama 완벽 연동** — Ollama 모델 관리도 UI에서 가능
- ✅ **OpenAI 호환 API 연동** — vLLM, LocalAI, 원격 API 연결
- ✅ **RAG 내장** — 문서 업로드 → 임베딩 → 검색 기반 응답
- ✅ **지식베이스** — PDF, Word, TXT 등 문서 관리
- ✅ **멀티 사용자** — 사용자 관리, 권한 제어
- ✅ **모델 관리** — Ollama 모델 Pull/삭제를 UI에서 직접
- ✅ **역할 놀이** — 시스템 프롬프트 템플릿
- ✅ **Docker 원클릭** — 1분 배포
- ✅ **다국어 지원** — 한국어 UI 지원

## 설치 방법

### Docker (권장 — 1분 설치)

```bash
# Ollama와 함께 실행 (단일 컨테이너)
docker run -d \
  --gpus all \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main

# 또는 Ollama와 별도 실행 (권장)
docker run -d \
  -p 3000:8080 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

### Docker Compose

```yaml
# docker-compose.yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    volumes:
      - ollama:/root/.ollama
    ports:
      - "11434:11434"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - open-webui:/app/backend/data
    depends_on:
      - ollama
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  ollama:
  open-webui:
```

```bash
docker compose up -d
```

## 사용법

### 접속

`http://localhost:3000` 으로 접속 → 계정 생성 (최초 생성자가 관리자)

### 주요 기능

1. **채팅** — 모델 선택 후 대화
2. **모델 관리** — Admin → Connections → Ollama에서 모델 Pull/삭제
3. **지식베이스** — Workspace → Knowledge → 문서 업로드
4. **RAG** — 채팅에서 문서 참조하여 질문
5. **템플릿** — 자주 사용하는 시스템 프롬프트 저장
6. **다중 모델** — 하나의 채팅에서 여러 모델 전환

### vLLM 연동 설정

```
관리자 설정 → Connections → OpenAI API
- API URL: http://host.docker.internal:8000/v1
- API Key: (빈 값 또는 임의 값)
- 모델 ID: (vLLM에 로드된 모델명)
```

## 고급 기능

### RAG (Retrieval-Augmented Generation)

```python
# Open WebUI에 문서 업로드 시:
# 1. 문서 텍스트 추출 (PDF, Word, TXT)
# 2. 청크 분할 (기본 500자)
# 3. 임베딩 생성 (Ollama 임베딩 모델)
# 4. 벡터 DB에 저장 (ChromaDB 내장)
# 5. 질문 시 유사 청크 검색 → 컨텍스트에 추가
```

### 환경 변수 설정

```bash
# 주요 환경 변수
OLLAMA_BASE_URL=http://ollama:11434  # Ollama 서버 주소
WEBUI_SECRET_KEY=your-secret-key    # 세션 암호화 키
ANONYMIZED_TELEMETRY=false          # 텔레메트리 비활성화
ENABLE_SIGNUP=false                 # 회원가입 비활성화
DEFAULT_MODELS=llama3.1:8b          # 기본 모델
```

## RTX 5090 + Open WebUI 구성

```
┌─────────────┐     ┌──────────────┐     ┌───────────┐
│  Open       │────▶│  Ollama      │────▶│  RTX 5090 │
│  WebUI      │     │  (서버)       │     │  (GPU)    │
│  (port 3000)│     │  (port 11434)│     │  32GB     │
└─────────────┘     └──────────────┘     └───────────┘
```

## 장단점

### 장점
- **프로페셔널한 UI** — ChatGPT와 동등한 사용자 경험
- **RAG 내장** — 별도 도구 없이 문서 기반 질의 가능
- **다중 백엔드 지원** — Ollama, vLLM, LocalAI 동시 연결
- **모델 관리 UI** — 터미널 없이 모델 다운로드/삭제
- **한국어 지원** — 네이티브 한국어 UI

### 단점
- **Docker 의존성** — 네이티브 설치는 상대적 복잡
- **시스템 리소스** — Node.js 백엔드 + Python RAG로 인한 오버헤드
- **Ollama 의존** — 최적의 경험을 위해 Ollama 필요
- **고급 기능 학습 곡선** — RAG, 지식베이스 등 설정 이해 필요

## 언제 Open WebUI를 선택할까?

✅ **다음 경우 추천합니다:**
- 로컬 LLM을 ChatGPT처럼 사용하고 싶을 때
- 문서 기반 질문(RAG)이 필요한 경우
- 팀/가족과 LLM을 공유해야 하는 경우
- 비개발자도 쉽게 사용할 수 있는 UI가 필요한 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- API 기반 서빙만 필요하고 UI는 불필요한 경우
- 최소한의 리소스 사용이 중요한 경우
- CLI 기반 워크플로우에 익숙한 경우

## 참고 자료

- [공식 문서](https://docs.openwebui.com)
- [GitHub](https://github.com/open-webui/open-webui)
- [Docker Hub](https://hub.docker.com/r/ghcr.io/open-webui/open-webui)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
