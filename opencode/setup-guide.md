# Ubuntu 22.04 LLM 환경 설치 가이드

> ASUS ROG Strix SCAR 16 (G635LX) + RTX 5090 32GB + Ubuntu 22.04.5 LTS

---

## 목차

1. [NVIDIA 드라이버 설치](#1-nvidia-드라이버-설치)
2. [CUDA Toolkit 설치](#2-cuda-toolkit-설치)
3. [Docker + NVIDIA Container Toolkit](#3-docker--nvidia-container-toolkit)
4. [Ollama 설치](#4-ollama-설치)
5. [Open WebUI 설치 (Docker)](#5-open-webui-설치-docker)
6. [모델 다운로드 및 실행](#6-모델-다운로드-및-실행-첫-실행)
7. [vLLM 설치 (고급)](#7-vllm-설치-고급)
8. [llama.cpp 설치](#8-llamacpp-설치)
9. [성능 확인](#9-성능-확인-방법)
10. [문제 해결](#10-문제-해결)
11. [시스템 부팅 시 자동 시작](#11-시스템-부팅-시-자동-시작)

---

## 1. NVIDIA 드라이버 설치

### 1.1 현재 드라이버 확인

```bash
nvidia-smi
```

만약 드라이버가 없다면 출력되지 않습니다.

### 1.2 드라이버 설치 (권장: 550+)

```bash
# NVIDIA GPU 드라이버 PPA 추가
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

# 드라이버 자동 설치 (권장)
sudo ubuntu-drivers autoinstall

# 또는 수동으로 특정 버전 설치
sudo apt install nvidia-driver-550

# 재부팅
sudo reboot
```

### 1.3 설치 확인

```bash
nvidia-smi
```

출력 예시:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 550.xx.xx    Driver Version: 550.xx.xx    CUDA Version: 12.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name                      TCC/WDDM  | Bus-Id        |
| 0    NVIDIA GeForce RTX 5090 ... WDDM    | 00000000:01:00.0 |
+-------------------------------+----------------------+----------------------+
```

---

## 2. CUDA Toolkit 설치

> Ubuntu 22.04 + RTX 5090 (Blackwell) 환경에서는 **CUDA 12.4** 이상을 권장합니다.
> CUDA 13.0이 출시되었지만, Ubuntu 22.04에서의 검증은 CUDA 12.4/12.8이 더 안정적입니다.

### 방법 A: NVIDIA 공식 runfile 설치 (권장)

```bash
# CUDA 12.4 다운로드
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run
sudo sh cuda_12.4.0_550.54.14_linux.run

# 설치 시 주의:
# - "Driver"는 이미 설치했으므로 체크 해제
# - "CUDA Toolkit"만 선택
# - "CUDA Samples" 옵션
```

### 방법 B: apt 패키지 설치

```bash
# NVIDIA CUDA apt 저장소 설정
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update

# CUDA 12.4 설치
sudo apt install cuda-toolkit-12-4
```

### 2.1 환경 변수 설정

```bash
# ~/.bashrc에 추가
echo 'export PATH=/usr/local/cuda-12.4/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

### 2.2 설치 확인

```bash
nvcc --version
```

출력:
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on ...
Cuda compilation tools, release 12.4, V12.4.xx
```

---

## 3. Docker + NVIDIA Container Toolkit

### 3.1 Docker 설치

```bash
# Docker 공식 저장소 설정
sudo apt update
sudo apt install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 현재 사용자를 docker 그룹에 추가 (sudo 없이 실행)
sudo usermod -aG docker $USER
newgrp docker  # 또는 로그아웃 후 재로그인
```

### 3.2 NVIDIA Container Toolkit 설치

```bash
# NVIDIA Container Toolkit 저장소 설정
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
sudo apt install nvidia-container-toolkit

# Docker에 NVIDIA 런타임 설정
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### 3.3 GPU 접근 확인

```bash
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

출력에 RTX 5090이 보이면 성공!

---

## 4. Ollama 설치

### 방법 A: 직접 설치 (권장)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 방법 B: Docker 설치

```bash
docker run -d --gpus all \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama
```

### 설치 확인

```bash
ollama --version
```

### GPU 활용 확인

```bash
ollama run llama3.1:8b "Hello, are you using GPU?"

# 또는 별도 터미널에서
nvidia-smi
```

Ollama 프로세스가 GPU 메모리를 사용 중인지 확인합니다.

---

## 5. Open WebUI 설치 (Docker)

### Ollama와 함께 단일 서버로 구성

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

### 접속

브라우저에서 `http://localhost:3000` 열기
→ 최초 접속 시 계정 생성 (첫 계정 = 관리자)

---

## 6. 모델 다운로드 및 실행 (첫 실행)

### Ollama로 모델 다운로드

```bash
# 필수 추천 모델
ollama pull llama3.1:8b

# 추가 옵션
ollama pull deepseek-r1:8b       # 경량 추론
ollama pull qwen2.5-coder:7b    # 코딩

# 고급 (32GB VRAM 활용)
ollama pull deepseek-r1:32b     # 강력 추론
ollama pull qwen2.5:32b         # 고품질

# 한국어
ollama pull exaone3.5:8b

# 임베딩 (RAG용)
ollama pull nomic-embed-text
```

### 실행

```bash
# CLI 채팅
ollama run llama3.1:8b

# API 서버 (이미 백그라운드 실행 중)
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Hello!",
  "stream": false
}'
```

---

## 7. vLLM 설치 (고급)

```bash
# 기본 pip 설치
pip install vllm

# 또는 특정 CUDA 버전
pip install vllm==0.9.0

# (선택) 소스 빌드
git clone https://github.com/vllm-project/vllm.git
cd vllm
pip install -e .
```

* vLLM은 의존성 패키지(PyTorch, Triton 등)가 무겁고 복잡하기 때문에
* Anaconda base 환경에 직접 빌드하는 것은 나중에 다른 프로젝트와 패키지 충돌을 일으킬 확률이 매우 높습니다.
* 가상환경을 분리하는 것을 권장합니다.

```Bash
# 1. vLLM 전용 콘다 가상환경 생성 (Python 3.10 또는 3.11 추천)
conda create -n vllm python=3.10 -y
conda activate vllm

# 2. 가상환경 내 최신 빌드 도구 및 필수 패키지 설치
pip install --upgrade pip setuptools wheel

# 3. vLLM 디렉토리로 이동 후 다시 빌드
cd ~/llm/vllm
pip install -e .
```



### OpenAI 호환 서버 실행

```bash
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Meta-Llama-3.1-8B-Instruct \
    --port 8000 \
    --max-model-len 32768 \
    --gpu-memory-utilization 0.95 \
    --dtype auto
```

### API 테스트

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## 8. llama.cpp 설치

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# CUDA 지원 빌드
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j

# GGUF 모델 다운로드 (Hugging Face)
# 예: wget https://huggingface.co/TheBloke/Llama-3.1-8B-Instruct-GGUF/resolve/main/llama-3.1-8b-instruct-q4_k_m.gguf

# 서버 실행
./build/bin/llama-server \
    -m /path/to/model.gguf \
    --host 0.0.0.0 \
    --port 8080 \
    -ngl 99 \
    -c 32768

# CLI 채팅
./build/bin/llama-cli \
    -m /path/to/model.gguf \
    -p "Hello!" \
    -n 256 \
    -ngl 99
```

---

## 9. 성능 확인 방법

### 9.1 Ollama 속도 테스트

```bash
# 특정 모델의 추론 속도 측정
ollama run llama3.1:8b

# 채팅 내에서 정보: "/show info" 로 모델 정보 확인
# tok/s는 응답 생성 시 하단에 표시됨
```

### 9.2 nvidia-smi 모니터링

```bash
# 실시간 GPU 모니터링
watch -n 1 nvidia-smi

# 상세 정보
nvidia-smi -l 1
```

### 9.3 llama.cpp 벤치마크

```bash
# llama.cpp 벤치마크 도구
./build/bin/llama-bench \
    -m /path/to/model.gguf \
    -ngl 99 \
    -n 128
```

* 실제 GGUF 모델 파일 다운로드
   - Hugging Face에서 공식 아키텍처에 맞춰 잘 컨버팅된 Llama 3.1 8B Instruct Q4_K_M 파일을 다운로드합니다.
   - 관리하기 편하게 models/ 폴더 안에 저장하도록 경로를 지정했습니다.

```
wget -O models/Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf https://huggingface.co/bartowski/Meta-Llama-3.1-8B-Instruct-GGUF/resolve/main/Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf
```


출력 예시:
```
| model                          | size | backend | ngl | t/s |
| ------------------------------ | ---- | ------- | --- | --- |
| llama 8B Q4_K_M                | 4.9 GB| CUDA    | 99  | 142.45 ± 2.34 |
```

### 9.4 예상 성능 (RTX 5090 기준)

| 모델 | 양자화 | ollama | vLLM (FP8) | llama.cpp | ExLlamaV2 |
|------|--------|--------|------------|-----------|-----------|
| Llama 3.1 8B | Q4_K_M | ~142 | ~165 | ~142 | ~205 (EXL2) |
| Llama 3.1 8B | Q8_0 | ~118 | - | ~118 | - |
| DeepSeek R1 32B | Q4_K_M | ~95 | - | ~95 | ~105 (EXL2) |
| Qwen 2.5 32B | Q4_K_M | ~90 | - | ~90 | ~100 (EXL2) |
| Llama 3.1 70B | Q4_K_M (offload) | ~20-30 | ❌ | ~25-35 | ❌ |

---

## 10. 문제 해결

### GPU 인식 문제

```bash
# 증상: Ollama가 GPU를 사용하지 않음
# 해결:
sudo systemctl stop ollama
sudo systemctl start ollama
ollama run llama3.1:8b

# 다른 터미널에서 GPU 사용 확인
nvidia-smi

# 여전히 안되면
sudo systemctl restart ollama
sudo journalctl -u ollama -n 50  # 로그 확인
```

### Docker GPU 접근 불가

```bash
# 증상: docker run --gpus all ... 이 동작하지 않음
# 해결:
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

### CUDA 버전 불일치

```bash
# 증상: "CUDA driver version is insufficient" 에러
# 해결: NVIDIA 드라이버 업데이트
sudo apt install nvidia-driver-550  # 또는 최신
sudo reboot
```

### Ollama 설치 실패 (Linux)

```bash
# 증상: curl 설치 실패
# 해결:
sudo apt update
sudo apt install curl
# 재시도
curl -fsSL https://ollama.com/install.sh | sh
```

### Open WebUI 연결 안됨

```bash
# 증상: Open WebUI가 Ollama에 연결 안됨
# 해결:
# Docker Compose 환경 변수 확인
# OLLAMA_BASE_URL이 올바른지 확인
docker logs open-webui  # 로그 확인
```

---

## 11. 시스템 부팅 시 자동 시작

### Ollama systemd 서비스 (이미 기본 설정됨)

```bash
# Ollama는 설치 시 자동으로 systemd 서비스로 등록됨
sudo systemctl status ollama
sudo systemctl enable ollama  # 부팅 시 자동 시작
```

### Docker 컨테이너 자동 시작

```bash
# Docker 컨테이너에 restart policy 추가
docker update --restart unless-stopped ollama
docker update --restart unless-stopped open-webui
```

### Docker Compose 서비스

```bash
# docker-compose.yml에 restart 추가
services:
  ollama:
    restart: unless-stopped
  open-webui:
    restart: unless-stopped
```

---

## 전체 설치 요약

### 최소 설치 (10분)

```bash
# 1. NVIDIA 드라이버
sudo apt install nvidia-driver-550
sudo reboot

# 2. NVIDIA Container Toolkit
sudo apt install nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# 3. Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 4. Open WebUI
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main

# 5. 모델
ollama pull llama3.1:8b

# 6. 접속!
# http://localhost:3000
```

### 전체 설치 (권장, 30분)

```bash
# 1. NVIDIA 드라이버
# 2. CUDA Toolkit 12.4
# 3. Docker + NVIDIA Container Toolkit
# 4. Ollama
# 5. Open WebUI (Docker Compose)
# 6. 모델: llama3.1:8b, deepseek-r1:8b, nomic-embed-text
# 7. (선택) vLLM
# 8. (선택) llama.cpp
```

---

→ [메인 페이지로 돌아가기](README.md)
→ [하드웨어 상세 분석](hardware-specs.md)
→ [모델 추천 보기](model-recommendations.md)
