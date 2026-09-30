# **학습 목표**

- Windows 환경에서 **WSL2(Ubuntu)** 기반 개발환경 완전 구축
- VSCode와 WSL2 **속성 연동**
- Python 설치 / 버전 확인
- **uv(초고속 패키지 관리자)** 설치 및 기본 명령어 실습
- 첫 Python 코드 실행
- “WSL + VSCode + uv + Python” 개발 흐름 이해

---
`1)`  개발환경 기본 이해
https://learn.microsoft.com/ko-kr/windows/wsl/install-manual

Windows 10에서 WSL2 설치전 사전 확인
![[Pasted image 20250524093630.png]]

| 상황                                   | 설명                                                |
| ------------------------------------ | ------------------------------------------------- |
| Windows11 최신 버전(2022 이후)             | WSL2가 기본 설치 + 기본값입니다. `wsl --install`만 해도 자동 설치됨. |
| Windows 11 초기 버전 또는 `WSL` 사용 안 했던 경우 | 기능은 포함되어 있지만 수동 활성화 또는 커널 업데이트 필요할 수 있음.          |
| `wsl` 명령어 실행 시 오류 발생                 | WSL2가 미설치 상태일 수 있으며, 설치 또는 커널 업데이트가 필요합니다.        |
❌ macOS에서는 별도로 리눅스 환경으로 만들 필요가 없음

|항목|설명|
|---|---|
|MacOS는 Unix 기반|macOS는 Darwin이라는 BSD 계열 유닉스 기반 운영체제입니다. 리눅스와 유사한 CLI와 파일시스템 구조를 갖고 있음.|
|Python, pip, venv 사용 가능|Django 개발에 필요한 핵심 도구들이 모두 macOS에 내장되거나 쉽게 설치 가능|
|Homebrew 등 패키지 관리자 지원|리눅스처럼 소프트웨어 패키지를 손쉽게 설치하고 관리 가능|

---
### 1. Windows 업데이트 확인

- `Win + R` → 실행창에 `winver` 입력 → Windows 버전 확인
- 최신 업데이트 적용 권장

---
### 2. 파일 확장명 및 숨김 항목 표시 설정

- `Win + E` → 탐색기 실행
- `[보기]` 탭 클릭 → `[파일 확장명]` 및 `[숨긴 항목]` 체크

이유:
- 파일 확장명: 악성코드 방지, 파일 타입 식별, 버전 구분
- 숨김 항목: `.env`, `.gitignore`, `.config` 등 프로젝트 필수 파일 확인

![[Pasted image 20250524092517.png]]

---
### 3. Windows Terminal 설치
WSL 설치, 설정, 커널 업데이트 등은 반드시 Windows 쪽 터미널 (PowerShell 또는 Windows Terminal)에서 해야 합니다.
- `[시작] `→ Microsoft Store 검색 → "Windows Terminal" 설치
- 설치 후 작업표시줄 고정 권장

---
### 4. WSL 기능 활성화
- `Win` 키 옆 검색창에 `Windows 기능 켜기/끄기` 입력 → 실행
- 다음 항목 체크:
    - `[가상 머신 플랫폼]`
    - `[리눅스용 Windows 하위 시스템]`
- 확인 후 재부팅

![[Pasted image 20250524092902.png]]

---
###  5. WSL2 커널 업데이트 설치
- Microsoft 공식 WSL2 커널 업데이트 설치 파일 다운로드:  
    🔗 https://aka.ms/wsl2kernel
    
- 다운로드한 `.msi` 파일을 더블 클릭 → 설치 진행  
    (관리자 권한 필요할 수 있음)
![[Pasted image 20250524100544.png]]
---
### 6. WSL2 기본 설정
- **PowerShell (관리자 권한)** 실행 후 아래 명령어 입력:

```powershell
wsl --set-default-version 2
```


```bash
wsl --install
```

---
### 7. Ubuntu 설치
- Microsoft Store → "Ubuntu 22.04 LTS" 또는 "Ubuntu 20.04 LTS" 검색 및 설치
- 설치 완료 후 실행 → 초기 설정(사용자명/비밀번호) 입력

[!마이크로소프트 스토어](https://apps.microsoft.com/search?query=Ubuntu+22.04+LTS&hl=ko-KR&gl=KR)
![[Pasted image 20250524093245.png]]

---
### 8. Ubuntu에서 시스템 패키지 업데이트

```bash
sudo apt update && sudo apt upgrade -y
```
---
### 9. VSCode와 WSL2 연동
1. VSCode 확장 → "WSL" 검색 → 설치
2. Ubuntu 터미널에서 `code .` 입력 → VSCode 실행
3. VSCode 하단 번개 아이콘 → "Connect to WSL" 선택

![[Pasted image 20250524101025.png]]

---
### 10. WSL 내에서 경로 확인 (리눅스 터미널)
- vscode 터미널에서 확인해도 됩니다.

```bash
pwd

# 결과 예시
/home/youjung/django_projects/myproject
```

![[Pasted image 20250524102439.png]]


확인 방법:
```powershell
wsl -l -v
```

예시 출력:
```powershell
  NAME              STATE           VERSION
* Ubuntu-24.04      Running         2
```

경로는 이렇게 작성해야 합니다:
``` bash
\\wsl$\Ubuntu-24.04\home\youjung\
```

---
### WSL 오류 해결
#### ❗ 오류 메시지: `0x8007019e: WSL이 설치되지 않았거나 비활성화된 경우`
WSL 관련 Windows 기능이 꺼져 있을 때

1. PowerShell(관리자 권한)에서 다음 명령 실행:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all
# WSL 1 기능 활성화 (리눅스 하위 시스템)

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all
# WSL 2를 위한 가상 머신 플랫폼 활성화
```

이후: 반드시 재부팅
다시 PowerShell(관리자) 실행:
```powershell
wsl --set-default-version 2
```
필요시 Ubuntu 재설치 또는 초기 실행

---
### BIOS 설정 (필요한 경우)
- 재부팅 후 BIOS 진입
- 가상화 기술(Virtualization Technology, SVM Mode 등) "Enabled"로 설정
- 설정 저장 후 재부팅

WSL2 설치나 실행이 안 되는 경우, 특히 WSL2에서 “가상화 오류(0x80370102 등)”가 발생할 때,  
➡️ BIOS 설정에서 가상화 기능(Virtualization)을 켜야 합니다.


BIOS에서 Virtualization 설정 방법

##### 1. 컴퓨터 재부팅 후 BIOS 진입
| 제조사                       | BIOS 진입 키                      |
| ------------------------- | ------------------------------ |
| ASUS, ACER, MSI, GIGABYTE | `Del` 또는 `F2`                  |
| Lenovo                    | `F1` 또는 `F2` 또는 `Enter` + `F1` |
| HP                        | `F10`                          |
| Dell                      | `F2`                           |
컴퓨터 부팅 직후 로고 화면에서 반복적으로 해당 키 누르기

##### 2. BIOS 메뉴에서 가상화 기능 찾기
| BIOS 항목 이름                                 | 설명                          |
| ------------------------------------------ | --------------------------- |
| **Intel Virtualization Technology (VT-x)** | 인텔 CPU용 가상화 기능              |
| **Intel VT-d**                             | 가상화된 장치 I/O 기능 (사용 가능 시 ON) |
| **AMD-V (SVM Mode)**                       | AMD CPU용 가상화 기능             |
| **Virtualization Technology**              | 브랜드 공통 항목                   |
| **Hyper-V Support** (간혹 있음)                | Hypervisor와 관련된 항목          |

---
##### 3. 해당 기능들을 모두 **Enabled**로 설정
- 선택 → `Enter` → `Enabled`
- 변경 저장 후 → `F10` → `Yes`로 저장하고 재부팅

---
✅ 이후 해야 할 작업
1. BIOS에서 가상화 기능 활성화 후 Windows 부팅
2. 다음 명령 실행:
```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all
wsl --set-default-version 2
```
Ubuntu 실행 → 더 이상 오류가 없어야 함

---
### VSCode 설치
WSL 개발은 반드시 Windows에 VSCode가 먼저 설치되어 있어야 합니다.

### 🔗 설치 링크
👉 [https://code.visualstudio.com/Download](https://code.visualstudio.com/Download)

### 설치 시 체크할 옵션
- Add to PATH (recommended)
- Register as default editor for supported file types
- Add “Open with Code” to Explorer context menu
    
### 설치 후 확인
- VSCode 실행 → 화면 정상 뜨면 OK
- 업데이트가 있다면 즉시 업데이트 권장

---
### VSCode 필수 확장 설치 (WSL + Python)
VSCode 좌측 Extensions(확장) → 아래 4개 검색하여 설치

|확장|설명|
|---|---|
|**WSL**|VSCode를 WSL 환경과 완벽하게 연결|
|**Python**|Python 개발 기본 확장 (Pylance 포함됨)|
|**Pylance**|Python 코드 자동완성/오류 분석|
|**GitLens**|Git 변경 내역 추적|
### 설치 링크(직접 열기 가능)

- WSL: [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
- Python: [https://marketplace.visualstudio.com/items?itemName=ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
- Pylance: [https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)
- GitLens: [https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

---
### Python 설치 (WSL Ubuntu 내부)
WSL Ubuntu는 기본 Python이 있지만 최신 버전 설치를 권장합니다.

버전 확인
```bash
python3 --version
```

최신 Python 설치
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

uv 방식:
```bash
uv add requests
uv add fastapi
```
✔ 더 빠름  
✔ 캐시 관리가 자동  
✔ 버전 충돌 방지 시스템 탁월  
✔ Poetry 같은 프로젝트 의존성 관리 기능 내장

uv 설치 (pip/venv 대체)
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

설치 확인
```bash
uv --version
```

기본 명령어
```bash
uv venv          # 가상환경 생성
uv run main.py   # 가상환경 자동 실행 + 코드 실행
uv pip install fastapi
uv add requests
uv remove requests
```

---
### Git 설치 (Windows / WSL 둘 다 필요)
Git은 Windows와 WSL 둘 다 설치하는 것이 좋습니다.  
하지만 WSL 안에서 개발할 것이므로 주력은 Ubuntu Git입니다.

✔ (A) Windows용 Git 설치

🔗 설치 링크
👉 https://git-scm.com/download/win

설치 옵션:
- “Use Visual Studio Code as Git’s default editor”
- “Git from the command line and also from 3rd-party software”
- CRLF 옵션은 기본 권장 선택 사용
    
---
✔ (B) WSL Ubuntu Git 설치
```bash
sudo apt update 
sudo apt install -y git
```

Git 버전 확인
```bash
git --version
```

---
### Git 기본 설정 (WSL 내부)
```bash
git config --global user.name "Eunice Lee" 
git config --global user.email "your-email@example.com" 
git config --global init.defaultBranch main
```

---
### GitHub 인증 (SSH 키 생성 → GitHub 등록)

`1)` SSH 키 생성
```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```
경로는 기본값 엔터 3번.

`2)` SSH 에이전트 실행
```bash
eval "$(ssh-agent -s)" 
ssh-add ~/.ssh/id_ed25519
```

`3)` 공개키 복사
```bash
cat ~/.ssh/id_ed25519.pub
```
출력된 내용을 그대로 복사

`4)` GitHub → Settings → SSH keys → New SSH key → 붙여넣기

![[Pasted image 20260106141028.png]]

`5)` 테스트
```bash
ssh -T git@github.com
```

성공 메시지:
```bash
Hi Eunice! You've successfully authenticated...
```

---
### VSCode와 WSL2 연동
Ubuntu 터미널에서 원하는 폴더로 이동:

```bash
mkdir dev 
cd dev 
code .
```
VSCode 창이 자동으로 **WSL(ubuntu)** 모드로 실행되면 성공.
VSCode 하단에 다음처럼 보임:
`WSL: Ubuntu-22.04`

---
### 첫 Python 코드 실행
예제 파일 생성
```bash
echo 'print("Hello WSL + Python + uv!")' > hello.py
```

실행
```bash
uv run hello.py
```

출력:
```bash
Hello WSL + Python + uv!
```
