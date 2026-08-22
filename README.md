# OMX-AI_ROBOTIS
# # chapter 1.  OMX-AI Orin Nano Super 초기 설정 (2025-08-15)

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


# OMX 리더-팔로워 텔레오퍼레이션 테스트 절차

## 1단계: 포트 번호 확인

리더와 팔로워 USB를 하나씩 순서대로 꽂고, 터미널에서 다음 명령으로 어느 포트가 어느 로봇인지 확인하세요:

```bash
ls -l /dev/serial/by-id/
```

각 장치의 시리얼 ID를 보고 `ttyACM0`/`ttyACM1` 중 어느 쪽이 리더인지 판단하세요.
(재부팅하면 번호가 바뀔 수 있으니 매번 확인)

---

## 2단계: 모터 ID 확인/수정 (필요시)

이전에 ID가 **11~16**으로 잘못 설정되어 있었습니다. Windows의 **DYNAMIXEL Wizard**로 각 모터를 **1~6**번으로 바꿔야 합니다 (리더/팔로워 둘 다 확인 필요).

이미 하셨거나 진행 중이면 이 단계는 건너뛰세요.

---

## 3단계: 팔로워 메인 전원 확인

팔로워(로봇팔 자체)에 별도 DC 전원(어댑터나 배터리)이 있다면 연결되어 있고 켜져있는지 확인하세요.

토크(Torque)가 안 켜지는 문제가 있었다면 이 전원이 문제일 가능성이 높습니다.

---

## 4단계: conda 환경 활성화

터미널에서:

```bash
conda activate lerobot
```

프롬프트 앞에 `(lerobot)`이 나타나는지 확인하세요.

---

## 5단계: 텔레오퍼레이션 실행

1단계에서 확인한 포트 번호로 아래 명령을 실행하세요.
(포트 번호는 매번 다를 수 있으니 1단계에서 확인한 값으로 교체해서 입력)

```bash
lerobot-teleoperate \
  --robot.type=omx_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=omx_follower_arm \
  --teleop.type=omx_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=omx_leader_arm
```

---

## 6단계: 동작 확인

에러 없이 실행되면 화면에 `shoulder_pan.pos`, `gripper.pos` 등 실시간 값이 나옵니다.

- 리더를 움직여보면서 값이 바뀌는지 확인
- 팔로워가 실제로 따라 움직이는지 확인

**종료:** `Ctrl + C`
        ↓
리더/팔로워 포트 확인 ✅

```
# OMX-AI 트러블슈팅 정리 (2026-08-15 ~ 16)

재조립한 로봇 세트를 텔레오퍼레이션 테스트하는 과정에서 겪었던 문제와 해결 과정 기록.
비슷한 증상을 겪는 학생들을 위한 참고용.
모터킷 구입해서 조립할 때 제대로 안하면 다음과같은 증상이 생김

원래 세팅된 상태에서 조립만 잘하면 아래와같은 일은 생기지 않는다

---

## 문제 1. 리더 USB 포트가 계속 끊겼다 붙었다 함

### 증상
```bash
ls -l /dev/serial/by-id/
```
을 반복 실행하면:
- 장치가 있다가 사라졌다 함
- 심지어 순간적으로 **0개**(`No such file or directory`)까지 나옴
- 재연결할 때마다 시리얼 ID가 다르게 나오는 것처럼 보임

### 원인
**USB 케이블 불량**. 겉보기엔 멀쩡해 보여도 내부 단선이 있으면 접촉이 불안정해서 계속 연결이 끊긴다.

### 해결
케이블을 다른 것으로 교체 → 즉시 안정적으로 잡힘.

### 확인 방법
```bash
watch -n 1 'ls -l /dev/serial/by-id/'
```
10~15초 지켜보면서 계속 안정적으로 유지되는지, ID가 계속 같은지 확인. 흔들리면 케이블 의심.

---

## 문제 2. `lerobot-teleoperate` 실행 시 모터를 하나도 못 찾음

### 증상
```
RuntimeError: DynamixelMotorsBus motor check failed on port '/dev/ttyACM1':
Missing motor IDs:
  - 1 (expected model: 1200)
  - 2 (expected model: 1200)
  ...
Full found motor list (id: model_number):
{}
```
 - 1 (expected model: 1200)
 - 2 (expected model: 1200)
모터가 0개(`{}`)로 나옴. 케이블을 바꿔도, 보드를 바꿔도 동일한 증상.

### 원인
**모터 ID가 lerobot이 기대하는 값(1~6)과 다르게 설정되어 있었음.**
확인해보니 실제 모터 ID가 **11, 12, 13, 14, 15, 16**으로 설정되어 있었음.
lerobot은 정확히 1~6번 ID를 찾기 때문에, 모터가 살아있고 정상 작동해도 ID가 다르면 "없음"으로 처리됨.
그래도 에러 생김 그래서 나오는 모터 부분을 재 조립하니 해결됨

### 진단 방법
1. **DYNAMIXEL Wizard 2.0**을 PC(Windows 권장, Orin Nano에는 설치 시 시스템이 불안정해질 수 있어 주의)에 설치
2. 문제의 보드를 PC에 직접 연결
3. **Scan** 실행 (가능하면 "Scan for all baudrates" 옵션 사용)
4. 발견된 모터의 ID 목록 확인 → 1~6이 아닌 다른 번호면 원인 확정

### 해결
DYNAMIXEL Wizard에서 각 모터를 클릭 → 오른쪽 상세 설정의 **ID** 드롭다운에서 값 변경 → **저장** 버튼 클릭.
```
ID 11 → 1
ID 12 → 2
ID 13 → 3
ID 14 → 4
ID 15 → 5
ID 16 → 6 (그리퍼, 모델 XL330-M077)
```
하나씩 순서대로 변경 후 재스캔하여 1~6이 모두 나오는지 확인.

> ⚠️ 리더/팔로워 둘 다 확인 필요. 한쪽만 고치고 넘어가지 말 것.

---

## 문제 3. 팔로워 그리퍼 모터가 뜨거워짐 / 값이 이상하게 나옴 (= 16번 모터 조립 오류)

### 증상
텔레오퍼레이션 중 리더 그리퍼(손잡이, 16번 모터) 모터가 뜨거워짐.

### 원인
**모터 조립 방향이 반대로 되어 있었음.** 그리퍼가 실제로 완전히 닫히는 지점과, 소프트웨어가 캘리브레이션 상 "닫힘"으로 인식하는 지점이 어긋나서, 모터가 계속 더 닫으려고 힘을 주다가 과부하로 발열.

이게 바로 **16번 모터의 조립 오류**였음 — 그리퍼 모터가 반대 방향으로 조립되어 있었던 것.

### 해결
모터 방향을 반대로(정방향으로) 재조립 → 정상화.

### 진단 팁
- 모터가 뜨거워지는 건 "계속 안 되는 방향으로 힘을 주고 있다"는 신호. 조립 방향/기구적 걸림을 의심할 것.
- `gripper.pos` 값 자체가 마이너스로 나오는 것은 정상 동작(그리퍼로 물체를 세게 쥘 때 목표 위치가 캘리브레이션 zero점을 넘어가면서 나오는 값)이니 값 자체로는 혼동하지 말 것. 문제의 핵심 신호는 값이 아니라 **모터 발열**이었음.

---

## 문제 4. 다이나믹셀 모터에 빨간불이 잠깐 켜졌다 꺼짐

### 증상
전원 인가 시 특정 모터(예: 12번, 14번)에 빨간 LED가 잠깐 들어왔다 사라짐.

### 원인
**정상적인 부팅 시퀀스**. 전원이 켜지는 순간 모터들이 자체 점검용으로 잠깐 반짝이는 것으로, 에러가 아님.

### 판단 기준
- 전원 켤 때 한 번만 반짝이고 꺼짐 → 정상
- 계속 깜빡이거나, 특정 동작 중에만 켜짐 → 과부하/과열 등 실제 알람일 수 있음 (전원 재시작으로 해결 안 되면 원인 추가 조사 필요)

---

## 문제 5. 팔로워 토크(Torque)가 안 켜짐

### 증상
팔로워 로봇팔에 전원은 들어오는데 Torque Enable이 안 됨/유지가 안 됨.

### 원인 (추정, 확인 필요)
팔로워팔은 실제로 힘을 내서 움직여야 하므로 USB 전원만으로는 부족할 수 있음. 별도 DC 메인 전원(어댑터/배터리)이 필요한 구조인지 확인 필요.

### 확인할 것
- 팔로워팔에 USB 외 별도 전원 포트가 있는지
- 있다면 연결 및 전원 On 여부

> **상태: 아직 미해결 — 다음 세션에서 이어서 확인 필요**

---

## 참고: 원인별 분류 — 조립 오류 vs 조립과 무관한 문제

같은 "안 움직인다", "빨간불 켜진다" 같은 증상이어도 원인은 다를 수 있음. 구분해두면 진단이 빨라짐.

### 조립(하드웨어 결합) 오류로 인한 문제
- **문제 3 (16번 모터 그리퍼 과열)**: 모터 방향을 반대로 조립 → 재조립하면서 방향 맞춰야 해결됨
- 관절이 뻑뻑하거나 걸리는 느낌, 기구물끼리 간섭하는 경우도 이 범주

### 조립과 무관한 문제 (설정값·부품 노후화 등)
- **문제 1 (USB 불안정)**: 순수 케이블 노후화/불량, 조립과 무관
- **문제 2 (모터 ID 11~16)**: 소프트웨어 설정값 문제. 기구적으로는 멀쩡히 조립되어 있어도 통신이 안 될 수 있음
- **문제 4 (전원 켤 때 반짝임)**: 정상 동작,애초에 "문제"가 아니었음
- **문제 5 (팔로워 토크)**: 전원 공급 문제로 추정 (조립보다는 배선/전원 계통)

> 즉 "모터 자체가 정상으로 응답하고 잘 움직이는데도 lerobot에서 안 잡힌다"면 조립을 다시 뜯기 전에 **먼저 ID·포트·전원 같은 설정값부터 확인**하는 게 시간을 아끼는 길임.

---

## 참고: 하드웨어 문제 진단 시 체크리스트 (막힐 때 순서대로)
##하드웨어 조립이 잘못되면 에러가 생김

1. `ls -l /dev/serial/by-id/` 로 포트/장치 인식 여부 확인
2. 케이블 교체해서 재현되는지 확인
3. 보드(컨트롤러) 교체해서 재현되는지 확인
4. DYNAMIXEL Wizard로 직접 스캔 (lerobot보다 하위 레벨에서 확인 가능)
   - 모터 개수, ID, 보드레이트 확인
   - 모터가 잡히면 Torque Enable + Goal Position으로 직접 구동 테스트
5. 위 4단계에서 모터가 정상 작동하면 → 소프트웨어 설정 문제 (ID, 포트 매핑 등)
6. 4단계에서도 모터가 하나도 안 잡히면 → 배선/전원 문제, 조립 재점검 필요


# chapter 2.  LeRobot 개발환경 세팅 정리 (노트북 x86_64)

## 0. 기본 개념 정리
- `~` 는 항상 홈 디렉토리(`/home/orin`)를 가리키는 절대경로 축약어. 어디에 있든 `rm -rf ~/miniconda3` 는 항상 홈의 miniconda3를 삭제함.
- 경로 헷갈릴 때: `pwd`(현재 위치), `ls ~`(홈 내용 확인)

---

## 1. Miniconda 설치

### 주의: 아키텍처 확인 필수
Jetson Orin(ARM64)용 설치파일(`Miniconda3-latest-Linux-aarch64.sh`)을 x86_64 노트북에 실행하면 에러 발생:
```
aarch64-binfmt-P: Could not open '/lib/ld-linux-aarch64.so.1': No such file or directory
```
→ 노트북은 x86_64 아키텍처이므로 x86_64용 설치파일을 받아야 함.

### 잘못 설치된 것 정리
```bash
rm -rf ~/miniconda3
```

### 올바른 설치파일 다운로드 및 설치
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```
- 라이센스 동의: `yes` 입력 (한글 자판 상태 확인!)
- 설치 경로: 기본값(`/home/orin/miniconda3`) 사용 시 Enter

### conda 초기화 (셸에 인식 안 될 경우)
```bash
~/miniconda3/bin/conda init bash
source ~/.bashrc
```

---

## 2. conda 이용약관(ToS) 동의
최근 conda 정책상 기본 채널 사용 전 ToS 동의 필요 (아키텍처 무관, 공통 절차):
```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

---

## 3. LeRobot용 Conda 가상환경 생성
```bash
conda create -y -n lerobot python=3.10
conda activate lerobot
python --version   # Python 3.10.20 확인
```

---

## 4. ffmpeg 설치
```bash
conda install -c conda-forge ffmpeg=6.1.1 -y
```
- 설치 중 `conda-pypi` 관련 WARNING은 무시해도 됨.

---

## 5. Git 설치 (사전 설치 안 되어 있던 경우)
```bash
sudo apt update
sudo apt install git -y
git --version
```
- `git clone`은 홈 디렉토리 내 작업이면 `sudo` 불필요.

---

## 6. LeRobot 저장소 클론
```bash
git clone https://github.com/ROBOTIS-GIT/lerobot.git
cd lerobot
```

---

## 7. 패키지 설치
```bash
pip install -e .
pip install -e ".[dynamixel]"
```
- 이 단계는 순수 소프트웨어(파이썬 패키지) 설치 과정으로, **로봇/다이나믹셀 하드웨어 연결 불필요**.
- 실제 포트(`/dev/ttyUSB0` 등) 연결 및 통신 테스트는 이후 단계에서 진행.

---

## 다음 할 일 (TODO)
- [ ] 하드웨어(로봇, 다이나믹셀) 연결 후 포트 인식 테스트
- [ ] LeRobot 예제 스크립트 실행 확인
