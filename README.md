# OMX-AI_ROBOTIS
# OMX-AI Orin Nano Super 초기 설정 (2025-08-15)

## 개요

- **디바이스**: Jetson Orin Nano Super
- **역할**: 로봇 컨트롤러 (OMX-L, OMX-F 연결, ROS 2 / Docker, Leader/Follower 동기화, 카메라 데이터 송신, 로봇 상태/관절값 송신)
- **참고 문서**: https://docs.robotis.com/docs/systems/omx/video_gallery

---

## Step 1. Miniconda 설치

```bash
rm -rf ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
bash Miniconda3-latest-Linux-aarch64.sh
```

> Jetson Orin Nano Super는 ARM64(aarch64) 아키텍처이므로 aarch64용 Miniconda를 설치합니다.

---

## Step 2. Anaconda 채널 TOS(이용약관) 동의

설치 후 채널 관련 오류가 발생하면 아래 명령으로 TOS에 동의합니다.

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

---

## Step 3. LeRobot용 Conda 가상환경 생성

```bash
conda create -y -n lerobot python=3.10
```

환경 활성화:

```bash
conda activate lerobot
```

버전 확인 (활성화 전에는 base 환경의 Python이 잡힐 수 있으니, 활성화 후 다시 확인):

```bash
python --version
# Python 3.10.20
```

---

## Step 4. ffmpeg 설치

```bash
conda install -c conda-forge ffmpeg=6.1.1 -y
```

> 설치 중 `conda-pypi` 관련 안내 메시지(WARNING)는 정상이며 무시해도 됩니다.

---

## Step 5. LeRobot 저장소 클론

```bash
git clone https://github.com/ROBOTIS-GIT/lerobot.git
cd lerobot
```

---

## Step 6. LeRobot 설치

```bash
pip install -e .
pip install -e ".[dynamixel]"
```

- `dynamixel-sdk`, `lerobot` 패키지가 설치됨
- 기존 `lerobot 0.3.4` 설치본이 있었다면 자동으로 제거 후 재설치됨

---

## Step 7. 리더/팔로워 로봇 포트 확인

연결된 시리얼 포트 확인:

```bash
ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null
```

- **리더+팔로워 모두 연결 시**: `/dev/ttyACM0 /dev/ttyACM1`
- **팔로워만 연결 시**: `/dev/ttyACM0`

> 확인 결과 **`/dev/ttyACM1`이 리더 로봇**의 포트로 확인됨.

---

## 진행 순서 요약

```
Jetson Orin Nano Super
        ↓
ARM64 (aarch64) 환경 확인
        ↓
Miniconda ARM64 설치 ✅
        ↓
conda 명령 정상 작동 ✅
        ↓
TOS 동의 ✅
        ↓
lerobot conda 환경 생성 ✅
        ↓
ffmpeg 설치 ✅
        ↓
LeRobot 저장소 클론 및 설치 ✅
        ↓
리더/팔로워 포트 확인 ✅
        ↓
(다음 단계) PyTorch / CUDA 설치
```
