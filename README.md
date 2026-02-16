# Local_LLM
# 로컬 LLM 설치 가이드 (Claude AI 대안)

Claude AI를 로컬 환경에서 실행할 수 있는 대안 솔루션 가이드입니다.

## 목차
- [개요](#개요)
- [Ollama 설치](#ollama-설치)
- [추천 모델](#추천-모델)
- [실행 및 테스트](#실행-및-테스트)
- [웹 인터페이스 (Open WebUI)](#웹-인터페이스-open-webui)
- [GPU 가속 설정](#gpu-가속-설정)
- [하드웨어 권장사양](#하드웨어-권장사양)
- [실전 활용 예시](#실전-활용-예시)
- [모델 커스터마이징](#모델-커스터마이징)
- [트러블슈팅](#트러블슈팅)

## 개요

Claude AI는 클라우드 기반 서비스로만 제공되어 로컬에서 직접 실행할 수 없습니다. 
이 가이드는 임베디드 시스템, FPGA, AI 개발 환경에 최적화된 로컬 LLM 솔루션을 제시합니다.

**주요 장점:**
- 인터넷 연결 불필요
- 민감한 코드/데이터 외부 전송 없음
- 무제한 사용 (API 비용 없음)
- 커스터마이징 가능

## Ollama 설치

### Linux
```bash
# 자동 설치 스크립트
curl -fsSL https://ollama.com/install.sh | sh

# 설치 확인
ollama --version
```

### macOS
```bash
# Homebrew 사용
brew install ollama

# 또는 공식 다운로드
# https://ollama.com/download
```

### Windows
```powershell
# 공식 설치 프로그램 다운로드
# https://ollama.com/download/windows
```

## 추천 모델

### 1. DeepSeek-R1 (추론 능력 우수)
```bash
# 7B 버전 (일반 사용)
ollama pull deepseek-r1:7b

# 양자화 버전 (메모리 절약)
ollama pull deepseek-r1:7b-q4_K_M
```

**특징:**
- 코딩 및 기술 분석 강점
- 논리적 추론 능력 뛰어남
- STM32, FPGA 프로젝트에 적합

### 2. Qwen2.5-Coder (코딩 특화)
```bash
ollama pull qwen2.5-coder:7b
```

**특징:**
- 코드 생성 및 디버깅 최적화
- 임베디드 C/C++ 코드 작성
- 하드웨어 레지스터 설정 이해

### 3. Llama 3.3 (범용)
```bash
# 8B 버전 (일반 GPU)
ollama pull llama3.3:8b

# 70B 버전 (고성능 GPU)
ollama pull llama3.3:70b
```

**특징:**
- 균형잡힌 범용 성능
- 기술 문서 작성
- 복잡한 프로젝트 분석

## 실행 및 테스트

### 기본 실행
```bash
# 대화형 모드
ollama run deepseek-r1:7b

# 종료: /bye 입력
```

### API 서버 모드
```bash
# 백그라운드 서버 실행
ollama serve

# 기본 포트: http://localhost:11434
```

### Python 사용
```bash
# Python 패키지 설치
pip install ollama
```
```python
import ollama

# 간단한 대화
response = ollama.chat(
    model='deepseek-r1:7b',
    messages=[{
        'role': 'user',
        'content': 'STM32F103에서 I2C로 MPU6050 센서 읽는 코드 작성해줘'
    }]
)

print(response['message']['content'])
```

### cURL 테스트
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "deepseek-r1:7b",
  "prompt": "STM32 UART 초기화 코드"
}'
```

## 웹 인터페이스 (Open WebUI)

Claude AI와 유사한 웹 인터페이스를 제공합니다.

### Docker 설치
```bash
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### 접속
```
http://localhost:3000
```

**기능:**
- 대화 히스토리 저장
- 여러 모델 동시 사용
- 파일 업로드 지원
- 마크다운 렌더링

## GPU 가속 설정

### NVIDIA GPU 확인
```bash
# GPU 상태 확인
nvidia-smi

# CUDA 버전 확인
nvcc --version
```

### Ollama GPU 사용 확인
```bash
# 실행 시 GPU 사용 모니터링
ollama run deepseek-r1:7b --verbose
```

### 메모리 최적화
```bash
# 양자화 모델 사용 (VRAM 절약)
ollama pull deepseek-r1:7b-q4_K_M  # 4-bit 양자화
ollama pull qwen2.5-coder:7b-q5_K_M  # 5-bit 양자화
```

## 하드웨어 권장사양

### 임베디드/FPGA 개발 환경 기준

| 구성 | CPU | RAM | GPU | 사용 모델 |
|------|-----|-----|-----|-----------|
| **최소** | 8코어 | 16GB | - | 7B Q4 |
| **권장** | 12코어 | 32GB | RTX 3060 12GB | 7B, 13B |
| **최적** | 16코어+ | 64GB | RTX 4090 24GB | 70B |

### 모델별 메모리 요구사항
- 7B 모델: RAM 8GB 또는 VRAM 6GB
- 13B 모델: RAM 16GB 또는 VRAM 10GB
- 70B 모델: RAM 64GB 또는 VRAM 40GB

## 실전 활용 예시

### 1. STM32 코드 자동 생성
```python
import ollama

def generate_stm32_code(sensor_type, mcu="STM32F411"):
    prompt = f"""
    {mcu} HAL 라이브러리 사용해서 {sensor_type} 센서
    - 초기화 함수
    - 데이터 읽기 함수
    - 에러 처리 포함
    - 주석 자세히 작성
    README.md 형식으로 작성
    """
    
    response = ollama.chat(
        model='qwen2.5-coder:7b',
        messages=[{'role': 'user', 'content': prompt}]
    )
    
    return response['message']['content']

# 사용 예시
code = generate_stm32_code("MPU6050")
with open("mpu6050_driver.md", "w") as f:
    f.write(code)
```

### 2. FPGA Verilog 코드 리뷰
```python
def review_verilog_code(code_file):
    with open(code_file, 'r') as f:
        code = f.read()
    
    prompt = f"""
    다음 Verilog 코드를 리뷰해주세요:
    
    {code}
    
    - 문법 오류
    - 타이밍 문제
    - 리소스 최적화
    - 베스트 프랙티스 위반
    """
    
    response = ollama.chat(
        model='deepseek-r1:7b',
        messages=[{'role': 'user', 'content': prompt}]
    )
    
    return response['message']['content']
```

### 3. 기술 문서 자동 생성
```python
def create_project_readme(project_name, description):
    prompt = f"""
    임베디드 프로젝트 README.md 작성:
    
    프로젝트명: {project_name}
    설명: {description}
    
    포함 내용:
    - 프로젝트 개요
    - 하드웨어 구성
    - 소프트웨어 아키텍처
    - 빌드 방법
    - 사용 방법
    - 트러블슈팅
    """
    
    response = ollama.chat(
        model='deepseek-r1:7b',
        messages=[{'role': 'user', 'content': prompt}]
    )
    
    return response['message']['content']
```

### 4. 배치 코드 생성 스크립트
```python
import ollama
import os

sensors = ["MPU6050", "VL53L0X", "DHT11", "BMP280"]

for sensor in sensors:
    print(f"Generating code for {sensor}...")
    
    response = ollama.chat(
        model='qwen2.5-coder:7b',
        messages=[{
            'role': 'user',
            'content': f'STM32F103 HAL {sensor} 드라이버 코드 작성'
        }]
    )
    
    output_dir = f"drivers/{sensor}"
    os.makedirs(output_dir, exist_ok=True)
    
    with open(f"{output_dir}/{sensor.lower()}.c", "w") as f:
        f.write(response['message']['content'])
    
    print(f"✓ {sensor} driver created")
```

## 모델 커스터마이징

### Modelfile 생성
```bash
cat > Modelfile << 'EOF'
FROM deepseek-r1:7b

# 시스템 프롬프트 설정
SYSTEM """
당신은 27년 경력의 임베디드 시스템 전문가입니다.
전문 분야:
- STM32 마이크로컨트롤러 개발
- FPGA 디지털 설계 (Verilog/VHDL)
- 로봇 제어 시스템
- 센서 인터페이싱
- 실시간 운영체제 (RTOS)

응답 규칙:
- 코드는 HAL 라이브러리 기반
- 주석은 한글로 상세히
- 에러 처리 필수 포함
- README.md 형식 선호
"""

# 파라미터 설정
PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER top_k 40
PARAMETER num_ctx 4096
EOF
```

### 커스텀 모델 생성 및 실행
```bash
# 모델 생성
ollama create embedded-expert -f Modelfile

# 실행
ollama run embedded-expert

# 테스트
# >>> STM32 타이머 인터럽트 설정 방법 알려줘
```

### 다양한 전문가 모델 예시

#### FPGA 전문가
```bash
cat > Modelfile.fpga << 'EOF'
FROM deepseek-r1:7b
SYSTEM """
FPGA 디지털 설계 전문가입니다.
- Verilog/VHDL 코딩
- Vivado/Quartus 사용
- 타이밍 제약 및 최적화
- IP 코어 통합
"""
PARAMETER temperature 0.5
EOF

ollama create fpga-expert -f Modelfile.fpga
```

#### 로봇 제어 전문가
```bash
cat > Modelfile.robot << 'EOF'
FROM deepseek-r1:7b
SYSTEM """
로봇 제어 시스템 전문가입니다.
- ROS/ROS2 프로그래밍
- 모터 제어 (BLDC, Stepper)
- 센서 융합 및 칼만 필터
- 경로 계획 알고리즘
"""
PARAMETER temperature 0.6
EOF

ollama create robot-expert -f Modelfile.robot
```

## 트러블슈팅

### 1. Ollama 서비스 시작 안됨
```bash
# 서비스 상태 확인
systemctl status ollama

# 수동 시작
ollama serve

# 로그 확인
journalctl -u ollama -f
```

### 2. GPU 인식 안됨
```bash
# NVIDIA 드라이버 확인
nvidia-smi

# CUDA 재설치 (필요시)
# https://developer.nvidia.com/cuda-downloads

# Ollama 재시작
systemctl restart ollama
```

### 3. 메모리 부족 오류
```bash
# 양자화 모델 사용
ollama pull deepseek-r1:7b-q4_K_M

# 또는 더 작은 모델
ollama pull qwen2.5-coder:3b
```

### 4. 느린 응답 속도
```bash
# CPU 코어 확인
nproc

# 환경변수 설정 (CPU 코어 수 지정)
export OLLAMA_NUM_PARALLEL=4

# GPU 사용 강제
export CUDA_VISIBLE_DEVICES=0
```

### 5. 포트 충돌
```bash
# 기본 포트 변경
export OLLAMA_HOST=0.0.0.0:11435

ollama serve
```

### 6. 모델 다운로드 실패
```bash
# 프록시 설정 (필요시)
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# 재시도
ollama pull deepseek-r1:7b
```

## 유용한 명령어
```bash
# 설치된 모델 목록
ollama list

# 모델 삭제
ollama rm deepseek-r1:7b

# 실행 중인 모델 확인
ollama ps

# 모델 정보 확인
ollama show deepseek-r1:7b

# 로그 레벨 설정
OLLAMA_DEBUG=1 ollama serve
```

## 참고 자료

- [Ollama 공식 문서](https://github.com/ollama/ollama)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [DeepSeek 모델](https://huggingface.co/deepseek-ai)
- [Qwen2.5-Coder](https://huggingface.co/Qwen/Qwen2.5-Coder)

## 라이선스

이 가이드는 MIT 라이선스로 배포됩니다.

## 기여

이슈 및 풀 리퀘스트를 환영합니다!

---

**작성자**: 임베디드 시스템 전문가  
**최종 수정**: 2026-02-16
