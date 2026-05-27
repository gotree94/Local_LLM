# Ollama — 가장 간편한 로컬 LLM 실행 도구

> **난이도**: ⭐ (매우 쉬움) | **용도**: 개인 사용, 개발 테스트, 빠른 시작

## 개요

[Ollama](https://ollama.com)는 로컬에서 LLM을 실행하는 **가장 대중적이고 간편한** 도구입니다. llama.cpp 기반으로 Go 래퍼 위에 구축되어 있으며, 단일 명령어로 모델을 다운로드하고 실행할 수 있습니다.

## 주요 특징

- ✅ **30초 설치** — 단일 스크립트로 설치 완료
- ✅ **방대한 모델 라이브러리** — 공식 및 커뮤니티 모델 100+종
- ✅ **OpenAI 호환 API** — `http://localhost:11434/v1`로 OpenAI 클라이언트와 호환
- ✅ **Modelfile** — Dockerfile 스타일의 커스텀 모델 빌드
- ✅ **자동 GPU 감지** — NVIDIA GPU 자동 탐지 및 활용
- ✅ **Docker 지원** — 공식 Docker 이미지 제공
- ✅ **다중 플랫폼** — Linux, macOS, Windows 지원

## 설치 방법

### Linux (권장)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Docker

```bash
docker run -d --gpus all -v ollama:/root/.ollama -p 11434:11434 \
  --name ollama ollama/ollama
```

## 기본 사용법

```bash
# 모델 다운로드 및 실행
ollama pull llama3.1:8b
ollama run llama3.1:8b

# 모델 리스트 확인
ollama list

# API 서버 상태 확인
curl http://localhost:11434/api/tags
```

### Python에서 Ollama 사용

```python
import ollama

# 채팅
response = ollama.chat(model='llama3.1:8b', messages=[
    {'role': 'user', 'content': 'Hello!'},
])
print(response['message']['content'])

# 스트리밍
for chunk in ollama.chat(model='llama3.1:8b', messages=[
    {'role': 'user', 'content': 'Hello!'}
], stream=True):
    print(chunk['message']['content'], end='', flush=True)
```

## 커스텀 모델 만들기 (Modelfile)

```dockerfile
# Modelfile
FROM llama3.1:8b

# 시스템 프롬프트 설정
SYSTEM "You are a helpful AI assistant specialized in Python programming."

# 온도 설정
PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER num_ctx 8192
```

```bash
ollama create my-custom-model -f Modelfile
ollama run my-custom-model
```

## RTX 5090에서의 성능

| 모델 | 양자화 | 속도 (tok/s) | 최대 컨텍스트 |
|------|--------|-------------|-------------|
| Llama 3.1 8B | Q4_K_M | ~142 | 256K+ |
| Llama 3.1 8B | Q8_0 | ~118 | 256K+ |
| DeepSeek R1 32B | Q4_K_M | ~95 | 128K+ |
| Qwen 2.5 32B | Q4_K_M | ~90 | 128K+ |
| Mixtral 8x7B | Q4_K_M | ~78 | 64K |

## 장단점

### 장점
- **설치-실행 1분** — 진입장벽 최소
- **활발한 커뮤니티** — 문제 해결이 쉬움
- **Open WebUI와 완벽 연동** — 웹 UI 즉시 사용 가능
- **모델 관리 자동화** — 풀/업데이트 모두 간단

### 단점
- **단일 요청 처리** — 순차 처리만 가능 (동시 요청 시 대기)
- **고급 설정 부족** — 세부 추론 파라미터 조정 제한적
- **메모리 효율 낮음** — vLLM 대비 KV 캐시 효율 2-3배 낮음
- **프로덕션 부적합** — 모니터링, 로드밸런싱 등 미지원

## 언제 Ollama를 선택할까?

✅ **다음 경우 Ollama를 추천합니다:**
- LLM을 처음 시작하는 경우
- 개인적인 채팅/코딩 어시스턴트가 필요한 경우
- 빠른 프로토타이핑이 필요한 경우
- Open WebUI와 함께 사용할 경우

❌ **다음 경우 다른 솔루션을 고려하세요:**
- 다중 사용자 서빙이 필요한 경우
- 프로덕션 환경에 배포해야 하는 경우
- 최고 추론 속도가 필요한 경우 (→ ExLlamaV2)
- 세부 모델 실험이 필요한 경우 (→ text-generation-webui)

## 참고 자료

- [공식 문서](https://github.com/ollama/ollama)
- [모델 라이브러리](https://ollama.com/library)
- [GitHub](https://github.com/ollama/ollama)

---

→ [메인 페이지로 돌아가기](../README.md)
→ [솔루션 비교 보기](comparison.md)
