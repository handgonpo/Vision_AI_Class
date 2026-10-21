part3_kakaomap

설치:
```bash
mkdir part3_kakaoMap
cd part3_kakaoMap
python3 -m venv venv
source venv/bin/activate
pip install notebook
```
---
쥬피터 노트북 공식웹사이트: [https://jupyter.org/](https://jupyter.org/)

###### Jupyter Notebook 장점 요약
| 💡 키워드             | 쉽게 설명하면…                                               |
| ------------------ | ------------------------------------------------------ |
| 셀 단위 실행            | 코드를 한 줄씩, 단계별로 실행해볼 수 있어요.오류 나도 그 셀만 다시 실행하면 돼요.       |
| 결과 즉시 확인           | 데이터를 불러오면 바로 보여줘요. <br>(VSCode는 print를 찍어야 하죠?)        |
| 시각화에 강함            | 그래프를 그리면 셀 아래 바로 보여줘요. <br>(`matplotlib`, `seaborn` 등) |
| 문서 작성 가능           | Markdown으로 설명을 같이 적을 수 있어서 보고서처럼 만들 수 있어요.             |
| 데이터 분석에 특화         | `pandas`, `scikit-learn` 같은 데이터 분석 도구랑 찰떡궁합이에요.        |
| 실험용으로 최고           | 여러 버전의 실험을 저장하고 비교하기가 쉬워요.                             |
| 설치 없이 브라우저에서 실행 가능 | 설치된 후엔 웹처럼 실행되기 때문에 편해요. (`localhost:8888`)            |
데이터 수집부터 분석, 모델 학습까지 한 눈에 보고, 한 줄씩 실행하면서, 그래프랑 설명까지 붙여가며 개발가능한  Jupyter Notebook은 친절한 도구입니다.

---
EXTENSIONS 에서 jupyter notebook을 설치한다.
![[Pasted image 20250705222235.png]]
![[Pasted image 20250705222241.png]]
![[Pasted image 20250705222247.png]]
![[Pasted image 20250705222252.png]]

| 확장명                            | 필요 여부    | 설명                                 |
| ------------------------------ | -------- | ---------------------------------- |
| **Jupyter**                    | ✅ **필수** | Jupyter 노트북(.ipynb) 실행 및 편집의 핵심 기능 |
| **Jupyter Keymap**             | ✅ 추천     | 단축키를 Jupyter Lab 스타일로 맞춰줌          |
| **Jupyter Cell Tags**          | ⭕ 선택     | Cell에 태그를 달아 구조화할 때 사용 (예: 슬라이드쇼용) |
| **Jupyter Notebook Renderers** | ⭕ 추천     | Plotly, Vega 등 시각화 라이브러리 잘 보이게 해줌  |
| **Jupyter Slide Show**         | ⭕ 선택     | 발표용 슬라이드 쇼 기능 (rare하게 사용)          |

---
이 명령은 브라우저를 자동으로 열지 않고 포트만 열어줍니다.
WSL의 IP 주소를 사용하여 브라우저에서 접속해야 합니다.
```bash
# wsl용
jupyter notebook --no-browser --port=8888

# macOS 
jupyter notebook 또는 
jupyter notebook --no-browser --port=9999
```
원격 서버에서 jupyter실행시 사용됩니다.

쥬피터 노트북과 연결된 링크:
```
http://localhost:8888/tree?token=65342e7337763d3b6ebbfe5b2351a513560534e33725bec6
http://127.0.0.1:8888/tree?token=65342e7337763d3b6ebbfe5b2351a513560534e33725bec6
```

Jupyter Notebook이 실행 중일 때 "어디서, 어떤 주소로 실행되고 있는지"를 확인하는 명령어
```bash
jupyter server list
```
Jupyter가 어느 폴더에서 열렸는지 확인 가능
결과확인:
```
Currently running servers:
http://localhost:8888/?token=4a21a0d3abbfc92f22487f3942865a3ba19f4885a625a117 :: /home/youjung/kakaoMap
```

토큰만 복사 Password or token 여기에 붙여넣기
http://localhost:8888/?token=abc1234567890abcdef...

---
새 파일 생성:
![[Pasted image 20250629083329.png]]

단축키
![[Pasted image 20250629083624.png]]A: 위에 셀추가
B: 아래 셀추가
D,D(빠르게 두번): 셀삭제
Z: 방금 지운 셀 되살리기
Y: 셀을 코드 셀로 전환
M: 셀을 마크다운 셀로 전환
C: 셀복사
V: 복사한 셀 아래에 붙이기
Shift+V: 복사한 셀 위에 붙이기
X: 잘라내기
셀 실행: `Shift + Enter`
실행 후 다음 셀 이동: `Shift + Enter`
셀 실행 후 새 셀 생성: `Alt + Enter`
실행만 하고 멈춤: `Ctrl + Enter`


jupyter notebook stop
pkill -f ipykernel_launcher

---
##### 웹드라이버 설치

```bash
# 크롬버전 확인
google-chrome --version
예: Google Chrome 138.0.7204.49

# python 라이브러리 설치
pip install selenium webdriver-manager

sudo apt update
sudo apt install -y chromium-browser
chromium-browser --version

```

selenium_utils.py 파일을 생성후 아래 코드를 작성합니다.
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

# Chrome 실행 옵션 설정
options = webdriver.ChromeOptions()
options.binary_location = "/usr/bin/chromium-browser"  # 시스템에 따라 경로 다를 수 있음
options.add_argument("--headless")  # UI 없이 실행
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")

# Chrome 드라이버 실행
service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)

# 페이지 열고 제목 출력
driver.get("https://google.com")
print(driver.title)

# 드라이버 종료
driver.quit()
```
크롬 드라이버를 수동 설치할 필요 없이 자동으로 다운로드하고, 셀레니움이 정상적으로 작동하는지 테스트하기 위한 코드

설치 확인
```bash
chromedriver --version

python selenium_utils.py
성공시
Google
```
---
실습해보기 : 구글에서 html가져오기
```python
pip install webdriver-manager
```
`webdriver-manager`는 Selenium에서 크롬 드라이버(또는 다른 브라우저 드라이버)를 자동으로 다운로드하고 설치·관리해주는 Python 라이브러리입니다.

vscode또는 쥬피터 노트북에 작성하세요.
`selenium_crawler.py`
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

def get_html(url: str):
    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service)

    driver.get(url)
    html = driver.page_source
    driver.quit()
    return html[:1000]
```

자동으로 연동된 Untilted.ipynb에 다음과 같이 작성합니다.
```ipynb
from selenium_crawler import get_html
get_html("https://google.com")
```

html Viewer: code Beauify
[https://codebeautify.org/htmlviewer](https://codebeautify.org/htmlviewer)
![[Pasted image 20250629095107.png]]
크로링한 html을 코드 뷰어로 펼쳐서 확인할수 있습니다.