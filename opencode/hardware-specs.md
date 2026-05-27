# 하드웨어 상세 분석

## ASUS ROG Strix SCAR 16 (G635LX)

### 전체 시스템 사양

| 구성 요소 | 사양 | 비고 |
|-----------|------|------|
| **모델명** | ROG Strix SCAR 16 G635LX | 2025년형 플래그십 게이밍 노트북 |
| **CPU** | Intel Core Ultra 9 285HX | 16코어 (8P + 8E) / 22스레드, 최대 5.5GHz |
| **GPU** | NVIDIA GeForce RTX 5090 Laptop GPU | Blackwell 아키텍처 |
| **VRAM** | **32GB GDDR7** | 512-bit, 1,792 GB/s 대역폭 |
| **CUDA 코어** | 21,760 | FP8 Tensor Core 지원 |
| **RAM** | 64GB DDR5 | 듀얼 채널 |
| **스토리지** | 4TB NVMe SSD | PCIe Gen5 지원 |
| **OS** | Ubuntu 22.04.5 LTS (Jammy) | WSL2 아님, 네이티브 설치 |

### ⚠️ VRAM 관련 중요 참고사항

**RTX 5090의 실제 VRAM은 32GB입니다.**

- NVIDIA GeForce RTX 5090 공식 사양: **32GB GDDR7**
- NVIDIA GeForce RTX 5080: **16GB GDDR7**
- NVIDIA GeForce RTX 4090 Laptop: **16GB GDDR6**

노트북 변형이라고 해도 RTX 5090 Laptop GPU는 32GB GDDR7을 탑재합니다. (일부 OEM 구성에 따라 차이가 있을 수 있으나, ASUS ROG SCAR 16 G635LX는 32GB 모델입니다.)

**24GB로 표기되었다면 잘못된 정보이므로 32GB로 정정하시기 바랍니다.**

### GPU 상세 (RTX 5090 Laptop)

| 항목 | 수치 |
|------|------|
| 아키텍처 | Blackwell |
| CUDA 코어 | 21,760 |
| Tensor 코어 | 5세대 (3352 AI TOPS) |
| RT 코어 | 4세대 (318 TFLOPS) |
| VRAM | 32GB GDDR7 |
| 메모리 대역폭 | 1,792 GB/s |
| TGP | 95~150W (Dynamic Boost 포함 최대 150W) |
| FP8 지원 | 네이티브 |
| PCI Express | Gen 5 |
| NVENC | 3x 9세대 |
| NVDEC | 2x 6세대 |

### CPU 상세 (Intel Core Ultra 9 285HX)

| 항목 | 수치 |
|------|------|
| 아키텍처 | Arrow Lake (2025) |
| P-코어 | 8개 (최대 5.5GHz) |
| E-코어 | 8개 (최대 4.6GHz) |
| 총 스레드 | 22 |
| L3 캐시 | 30MB |
| TDP | 55W (기본) / 160W (최대) |
| 메모리 지원 | DDR5-6400 |

### 로컬 LLM 운용에 대한 시사점

#### 강점
1. **32GB VRAM** — 7B~35B급 모델을 전체 GPU 오프로드로 구동 가능
2. **GDDR7 고대역폭** — 1,792 GB/s로 추론 속도 우수 (4090 Desktop의 1,008 GB/s 대비 78% 향상)
3. **FP8 Tensor Core** — FP8 양자화 모델에서 네이티브 성능 발휘
4. **64GB 시스템 RAM** — 70B급 모델의 CPU offload에도 충분한 메모리
5. **16코어 CPU** — LLM 외 병렬 작업(데이터 전처리, 토크나이징)에 유리

#### 한계
1. **노트북 TGP 제한 (150W)** — 데스크탑 RTX 5090 (575W) 대비 추론 속도 열위
2. **70B+ 모델 전체 오프로드 불가** — Q4 양자화 기준 42GB 필요 → CPU offload 필요
3. **발열 관리** — 장시간 추론 시 쓰로틀링 가능성 (ROG 특성상 쿨링은 양호한 편)
4. **메모리 대역폭** — 데스크탑 RTX 5090 (1,792 GB/s) 동일하나 TGP 제한으로 실제 성능은 낮음

#### 적합 모델 범위 (100% GPU)

| 등급 | 예시 모델 | 양자화 | GPU 전용 가능? |
|------|----------|--------|---------------|
| 소형 (7B) | Llama 3.1 8B, Qwen 2.5 7B | Q8_0 | ✅ 여유 |
| 중형 (13B) | Mistral Small, CodeLlama 13B | Q4_K_M | ✅ 여유 |
| 중대형 (30-35B) | Qwen 3.6 27B, DeepSeek R1 32B | Q4_K_M | ✅ 가능 |
| 대형 (70B) | Llama 3.1 70B, Qwen 2.5 72B | Q4_K_M | ❌ (32GB 초과) |
| 대형 (70B) | Llama 3.1 70B | Q2_K | ⚠️ 간신히 Fit |
| MoE 대형 | Mixtral 8x22B | Q4_K_M | ⚠️ Offload 필요 |

---

## Ubuntu 22.04 환경

현재 시스템은 Ubuntu 22.04.5 LTS를 네이티브로 구동 중입니다.

| 항목 | 상태 |
|------|------|
| OS | Ubuntu 22.04.5 LTS (Jammy) |
| 설치 방식 | 네이티브 (듀얼 부트 또는 단독) |
| Docker | 설치 가능 (nvidia-container-toolkit 필요) |
| CUDA Driver | NVIDIA 공식 드라이버 필요 (550+ 권장) |

> **Ubuntu 22.04는 CUDA 12.x와 완벽 호환**되며, Ollama, Docker, PyTorch 등 모든 주요 AI 도구를 지원합니다.
> 단, 최신 CUDA 13.0은 Ubuntu 24.04를 권장하므로, 22.04에서는 CUDA 12.4/12.8 사용을 권장합니다.

→ [Ubuntu 22.04 CUDA + Docker + Ollama 설치 가이드](setup-guide.md)
→ [모델 추천 보기](model-recommendations.md)
→ [메인 페이지로 돌아가기](README.md)
