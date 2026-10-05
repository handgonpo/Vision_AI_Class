
> **목표**  
> 이 자료는 Tkinter를 처음 접하는 학생이 **가상환경 생성 → 프로젝트 폴더 생성 → 파일 작성 → 실행 → 결과 확인**까지 그대로 따라 하면서 GUI 프로그래밍의 기본 구조를 이해하도록 만든 입문 실습입니다.
>
> 오늘은 복잡한 라벨링 도구를 바로 만들지 않습니다.  
> 먼저 다음 세 가지를 순서대로 익힙니다.
>
> 1. **예제 1 — 버튼이 있는 기본 화면 만들기**
> 2. **예제 2 — 입력 폼과 클래스 선택 화면 만들기**
> 3. **예제 3 — 마우스로 BBox 사각형 그리기**
>
> 세 번째 예제까지 이해하면 이후 **교과 7 Mini Labeling Tool**에서 사용하는 `Canvas`, 마우스 이벤트, BBox의 기본 구조를 이해할 수 있습니다.

---

# 0. 오늘 수업의 전체 흐름

```text
프로젝트 폴더 생성
        ↓
Python 가상환경 생성
        ↓
Tkinter 실행 확인
        ↓
예제 1
기본 Window + Label + Button
        ↓
예제 2
Entry + Combobox + Button
        ↓
예제 3
Canvas + Mouse Event + BBox
        ↓
세 예제의 공통 구조 정리
        ↓
라벨링 도구와 연결 이해
```

---

# 1. 오늘 사용할 기술

| 기술 | 오늘 하는 일 |
|---|---|
| Python | GUI 프로그램 코드 작성 |
| Tkinter | Python 기본 GUI 라이브러리 |
| ttk | 조금 더 정돈된 버튼·입력창·Combobox 사용 |
| Canvas | 도형과 BBox를 그리는 화면 |
| Mouse Event | 클릭·드래그·놓기 동작 처리 |
| `.venv` | 프로젝트별 Python 실행환경 분리 |

---

# 2. Tkinter는 무엇인가요?

Tkinter는 Python에서 GUI 프로그램을 만들 때 사용하는 기본 라이브러리입니다.

예를 들어 다음과 같은 프로그램을 만들 수 있습니다.

```text
Python 코드
    ↓
Tkinter
    ↓
Window
 ├─ Label
 ├─ Button
 ├─ Entry
 ├─ Combobox
 └─ Canvas
```

교과 7의 라벨링 도구를 생각하면 다음과 연결됩니다.

```text
라벨링 프로그램

Window
 ├─ 이미지 표시 영역      → Canvas
 ├─ Class 선택           → Combobox
 ├─ 다음 이미지 버튼      → Button
 ├─ 현재 파일명 표시      → Label
 └─ BBox 작성            → Mouse Event
```

즉 오늘 배우는 작은 예제들이 나중에 하나의 라벨링 도구로 합쳐집니다.


## HTML · CSS · JavaScript를 배웠다면 이렇게 연결해서 생각합니다

Tkinter가 완전히 새로운 개념은 아닙니다. 웹 프론트엔드를 배웠다면 이미 비슷한 구조를 알고 있습니다.

| 웹 개발                 | Tkinter            | 역할                |
| -------------------- | ------------------ | ----------------- |
| HTML 문서              | `Tk()` Window      | 화면의 가장 바깥 구조      |
| `<div>`              | `Frame`            | 여러 UI를 묶는 영역      |
| `<p>`, `<span>`      | `Label`            | 글자 표시             |
| `<button>`           | `Button`           | 버튼                |
| `<input>`            | `Entry`            | 문자 입력             |
| `<select>`           | `ttk.Combobox`     | 목록 선택             |
| `<canvas>`           | `Canvas`           | 그림·도형 표시 영역       |
| CSS 배치               | `pack()`, `grid()` | 화면 배치             |
| `onclick`            | `command=`         | 버튼 클릭 처리          |
| `addEventListener()` | `.bind()`          | 마우스·키보드 Event 연결  |
| `input.value`        | `.get()`           | 입력값 읽기            |
| DOM 내용 변경            | `.config()`        | 화면 내용 변경          |
| 브라우저 Event Loop      | `mainloop()`       | GUI Event를 계속 기다림 |

웹에서는 보통 다음처럼 역할을 나눕니다.

```text
HTML
→ 화면 구조

CSS
→ 화면 배치와 모양

JavaScript
→ 클릭, 입력, Event 처리
```

Tkinter에서는 이 세 역할을 Python 코드 안에서 함께 처리합니다.

```text
Tkinter Widget 생성
→ HTML과 비슷한 역할

pack() / grid()
→ CSS Layout과 비슷한 역할

command / bind / Python 함수
→ JavaScript Event 처리와 비슷한 역할
```

따라서 오늘은 Tkinter 문법을 외우기보다 **“웹에서 하던 일을 Python GUI에서는 어떻게 표현하는가?”**를 비교하면서 학습합니다.

---

# PART 1. 개발환경을 준비합니다

---

## 먼저 VS Code가 WSL에 연결되어 있는지 확인합니다

이번 실습은 **Windows PowerShell이 아니라 WSL2 Ubuntu 터미널 기준**으로 진행합니다.

VS Code 왼쪽 아래에 다음과 비슷한 표시가 있는지 확인합니다.

```text
WSL: Ubuntu-22.04
```

VS Code Terminal에서 다음 명령을 입력합니다.

```bash
pwd
```

WSL이라면 다음처럼 Linux 경로가 나옵니다.

```text
/home/user1
```

또는 프롬프트가 다음처럼 보입니다.

```text
user1@DESKTOP-XXXX:~$
```

반대로 다음처럼 보이면 Windows PowerShell입니다.

```text
PS C:\\Users\\student>
```

이 경우 먼저 VS Code를 WSL Window로 다시 엽니다.

> **기억하기**  
> `PS C:\\...>` → Windows PowerShell  
> `/home/...` 또는 `user@DESKTOP:~$` → WSL Ubuntu

현재 위치가 `/mnt/c/WINDOWS/...`라면 프로젝트를 만들기 전에 Linux 홈으로 이동합니다.

```bash
cd ~
pwd
```

예상 결과:

```text
/home/user1
```

---

# 3. 프로젝트 폴더를 만듭니다

터미널을 엽니다.

Ubuntu, WSL, Linux 기준으로 다음 명령을 실행합니다.

```bash
mkdir -p ~/subject07_tkinter_intro
cd ~/subject07_tkinter_intro
```

현재 위치를 확인합니다.

```bash
pwd
```

예상 결과는 다음과 비슷합니다.

```text
/home/사용자이름/subject07_tkinter_intro
```

폴더 안의 내용을 확인합니다.

```bash
ls
```

처음에는 아무 파일도 없어도 정상입니다.

---

# 4. Python 버전을 확인합니다

```bash
python3 --version
```

예:

```text
Python 3.12.3
```

Python 3.x가 표시되면 됩니다.

---

# 5. Tkinter 설치 여부를 확인합니다

먼저 다음 명령을 실행합니다.

```bash
python3 -m tkinter
```

작은 Tkinter 테스트 창이 나타나면 설치되어 있는 것입니다.

창을 닫고 다음 단계로 넘어갑니다.

## 만약 오류가 발생한다면

Ubuntu 계열에서는 다음과 같이 설치할 수 있습니다.

```bash
sudo apt update
sudo apt install -y python3-tk
sudo apt install -y fonts-noto-cjk
```

설치 후 다시 확인합니다.

```bash
python3 -m tkinter
```

> Windows에 Python을 일반 설치한 경우 Tkinter가 함께 설치되어 있는 경우가 많습니다.

---

# 6. 가상환경을 만듭니다

프로젝트 폴더 안에서 실행합니다.

```bash
python3 -m venv .venv
```

폴더를 확인합니다.

```bash
ls -a
```

다음과 같이 `.venv`가 보이면 됩니다.

```text
.
..
.venv
```

---

# 7. 가상환경을 활성화합니다

Ubuntu / WSL / Linux:

```bash
source .venv/bin/activate
```

터미널 앞에 다음과 같이 표시되면 정상입니다.

```text
(.venv) user@computer:~/subject07_tkinter_intro$
```

Python 위치를 확인합니다.

```bash
which python
```

예상 결과:

```text
/home/사용자이름/subject07_tkinter_intro/.venv/bin/python
```

---

# 8. 오늘 사용할 폴더를 만듭니다

```bash
mkdir -p example01_basic
mkdir -p example02_form
mkdir -p example03_bbox
```

확인합니다.

```bash
tree
```

`tree` 명령이 없다면 다음 명령을 사용해도 됩니다.

```bash
find . -maxdepth 2 -type d
```

현재 구조는 다음과 같습니다.

```text
subject07_tkinter_intro/
├── .venv/
├── example01_basic/
├── example02_form/
└── example03_bbox/
```

---

# PART 2. 예제 1 — 기본 Window와 Button

---

# 9. 예제 1에서 무엇을 만드나요?


## HTML · JavaScript와 먼저 비교합니다

웹에서 같은 동작을 만든다면 다음과 비슷합니다.

```html
<p id="message">아래 버튼을 눌러보세요.</p>
<button onclick="changeMessage()">메시지 변경</button>
```

```javascript
function changeMessage() {
    document.getElementById("message").innerText =
        "버튼을 눌렀습니다!";
}
```

Tkinter에서는 다음처럼 연결됩니다.

```text
HTML <p>          → Tkinter Label
HTML <button>     → Tkinter Button
onclick           → command
JavaScript 함수   → Python 함수
innerText 변경    → Label.config(text=...)
```

즉 예제 1의 핵심은 **“Button Event가 발생하면 Python 함수가 실행되고, 그 함수가 화면을 바꾼다”**입니다.

첫 번째 예제에서는 가장 기본적인 GUI 화면을 만듭니다.

```text
Window
 ├─ 제목 Label
 ├─ 설명 Label
 └─ Button
```

버튼을 누르면 화면의 문장이 바뀝니다.

이 예제에서 다음을 배웁니다.

- `Tk()` : 프로그램의 기본 창 만들기
- `Label` : 글자 표시
- `Button` : 버튼 만들기
- `command` : 버튼을 눌렀을 때 실행할 함수 연결
- `mainloop()` : GUI 프로그램을 계속 실행

---

# 10. 예제 1의 의사코드

코드를 작성하기 전에 흐름부터 봅니다.

```text
Tkinter 불러오기

        ↓

기본 Window 생성

        ↓

Window 제목과 크기 설정

        ↓

Label 생성

        ↓

Button 생성

        ↓

버튼 클릭 함수 작성

        ↓

Button과 함수 연결

        ↓

mainloop 실행
```

---

# 11. 예제 1 파일을 만듭니다

```bash
cd ~/subject07_tkinter_intro/example01_basic
touch app.py
```

확인합니다.

```bash
ls
```

```text
app.py
```

VS Code를 사용한다면 다음과 같이 열 수 있습니다.

```bash
code app.py
```

---

# 12. 예제 1 코드를 작성합니다

`app.py`에 다음 코드를 작성합니다.

```python
import tkinter as tk  


def change_message():
    """
    버튼을 눌렀을 때 실행되는 함수입니다.
    message_label의 글자를 새로운 문장으로 바꿉니다.
    """
    message_label.config(text="버튼을 눌렀습니다!")


# 1. 프로그램의 기본 Window를 만듭니다.
root = tk.Tk()

# 2. Window 제목을 설정합니다.
root.title("예제 1 - Tkinter 기본 화면")

# 3. Window 크기를 설정합니다.
root.geometry("500x300")

# 4. 제목을 표시하는 Label을 만듭니다.
title_label = tk.Label(
    root,
    text="교과 7 Tkinter 첫 번째 실습",
    font=("Arial", 18)
)
title_label.pack(pady=30)

# 5. 상태 메시지를 표시하는 Label을 만듭니다.
message_label = tk.Label(
    root,
    text="아래 버튼을 눌러보세요.",
    font=("Arial", 12)
)
message_label.pack(pady=10)

# 6. Button을 만듭니다.
change_button = tk.Button(
    root,
    text="메시지 변경",
    command=change_message,
    width=15,
    height=2
)
change_button.pack(pady=20)

# 7. GUI 프로그램을 계속 실행합니다.
root.mainloop()
```

---

# 13. 예제 1을 실행합니다

프로젝트의 가상환경이 활성화되어 있는지 확인합니다.

```text
(.venv)
```

예제 폴더에서 실행합니다.

```bash
python app.py
```

화면에 다음과 같은 GUI가 나타납니다.

```text
+--------------------------------------+
|     교과 7 Tkinter 첫 번째 실습      |
|                                      |
|        아래 버튼을 눌러보세요.       |
|                                      |
|          [ 메시지 변경 ]             |
+--------------------------------------+
```

`메시지 변경` 버튼을 눌러봅니다.

화면의 문장이 다음과 같이 바뀌면 성공입니다.

```text
버튼을 눌렀습니다!
```

---

# 14. 예제 1 코드를 이해합니다

## `tk.Tk()`

```python
root = tk.Tk()
```

GUI 프로그램의 가장 바깥 창을 만듭니다.

쉽게 생각하면 다음과 같습니다.

```text
Tk()
=
빈 프로그램 창 하나 만들기
```

---

## `Label`

```python
title_label = tk.Label(
    root,
    text="교과 7 Tkinter 첫 번째 실습"
)
```

화면에 글자를 표시합니다.

`root`는 이 Label이 어느 Window에 들어가는지를 의미합니다.

---

## `pack()`

```python
title_label.pack()
```

만든 위젯을 실제 화면에 배치합니다.

위젯을 만들기만 하고 `pack()`하지 않으면 화면에 나타나지 않습니다.

---

## `Button`

```python
change_button = tk.Button(
    root,
    text="메시지 변경",
    command=change_message
)
```

버튼을 만듭니다.

여기에서 중요한 부분은 다음입니다.

```python
command=change_message
```

버튼을 눌렀을 때 `change_message()` 함수를 실행하라는 의미입니다.

---

## `config()`

```python
message_label.config(
    text="버튼을 눌렀습니다!"
)
```

이미 만들어져 있는 Label의 설정을 변경합니다.

---



## `mainloop()`는 브라우저의 Event Loop와 비슷합니다

웹 브라우저는 페이지가 열린 동안 사용자의 클릭, 입력, 마우스 이동을 계속 기다립니다. Tkinter에서는 `mainloop()`가 이 역할을 합니다.

```text
브라우저
→ 클릭을 기다림
→ Event가 발생하면 JavaScript 함수 실행

Tkinter
→ mainloop()가 Event를 기다림
→ Event가 발생하면 Python 함수 실행
```

따라서 `root.mainloop()`는 단순히 “창을 띄우는 명령”이라기보다 **GUI 프로그램이 사용자의 행동을 계속 기다리게 하는 핵심 반복 구조**라고 이해합니다.

## `command=change_message`에 괄호를 붙이지 않는 이유

올바른 코드:

```python
command=change_message
```

잘못된 코드:

```python
command=change_message()
```

`command`에는 지금 당장 실행한 결과가 아니라 **나중에 버튼을 눌렀을 때 실행할 함수 자체**를 전달해야 합니다.

---

# 15. 예제 1 Mini Challenge

다음 세 가지를 직접 수정해봅니다.

1. Window 제목을 자신의 이름으로 바꿉니다.
2. 버튼 이름을 `실습 완료`로 바꿉니다.
3. 버튼을 눌렀을 때 `교과 7 실습 준비 완료`가 표시되도록 수정합니다.

예:

```text
이유정 - Tkinter 연습
        ↓
[ 실습 완료 ]
        ↓
교과 7 실습 준비 완료
```

---

# 16. 예제 1 체크리스트

- [ ] Tkinter Window가 실행된다.
- [ ] Label이 화면에 표시된다.
- [ ] Button이 화면에 표시된다.
- [ ] 버튼을 누르면 함수가 실행된다.
- [ ] Label의 문장이 변경된다.
- [ ] `mainloop()`의 역할을 설명할 수 있다.

---

# PART 3. 예제 2 — 입력 폼과 클래스 선택

---

# 17. 예제 2에서 무엇을 만드나요?


## HTML Form과 비교합니다

웹에서는 다음과 같은 구조입니다.

```html
<input id="fileName">

<select id="className">
    <option>leaf</option>
    <option>plastic</option>
    <option>metal</option>
</select>

<button onclick="showResult()">입력 확인</button>
```

JavaScript에서는 입력값을 이렇게 읽습니다.

```javascript
const fileName = document.getElementById("fileName").value;
const className = document.getElementById("className").value;
```

Tkinter에서는 다음처럼 생각합니다.

```text
HTML <input>       → Entry
HTML <select>      → Combobox
input.value        → StringVar.get()
CSS Grid           → grid(row=..., column=...)
Validation         → if + return
```

즉 예제 2는 **웹의 Form을 Python GUI로 다시 만드는 실습**입니다.

라벨링 프로그램에서는 보통 다음 정보가 필요합니다.

```text
현재 이미지
Class
저장
상태 메시지
```

두 번째 예제에서는 이를 단순화한 화면을 만듭니다.

```text
파일명 입력
Class 선택
        ↓
저장 버튼
        ↓
현재 입력 내용을 화면에 표시
```

실제 데이터 파일을 저장하지는 않습니다.

오늘의 목적은 **입력값을 GUI에서 가져오는 방법**을 이해하는 것입니다.

---

# 18. 예제 2에서 배울 내용

- `Entry` : 사용자가 글자를 입력하는 칸
- `StringVar` : GUI 값 보관
- `ttk.Combobox` : 목록에서 Class 선택
- `grid()` : 행과 열로 화면 배치
- 버튼을 눌러 입력값 읽기

---

# 19. 예제 2 의사코드

```text
Window 생성

        ↓

파일명 Label 생성
Entry 생성

        ↓

Class Label 생성
Combobox 생성

        ↓

저장 Button 생성

        ↓

버튼 클릭

        ↓

Entry 값 읽기

        ↓

Combobox 값 읽기

        ↓

결과 Label에 표시
```

---

# 20. 예제 2 파일을 만듭니다

```bash
cd ~/subject07_tkinter_intro/example02_form
touch app.py
```

VS Code:

```bash
code app.py
```

---

# 21. 예제 2 코드를 작성합니다

```python
import tkinter as tk
from tkinter import ttk


def show_result():
    """
    Entry와 Combobox에서 현재 값을 읽어서
    result_label에 표시합니다.
    """

    file_name = file_name_var.get()
    class_name = class_var.get()

    if file_name == "":
        result_label.config(
            text="파일명을 입력하세요."
        )
        return

    if class_name == "":
        result_label.config(
            text="Class를 선택하세요."
        )
        return

    result_text = (
        f"파일명: {file_name}\n"
        f"선택 Class: {class_name}"
    )

    result_label.config(text=result_text)


# --------------------------------------------------
# 1. Window 생성
# --------------------------------------------------

root = tk.Tk()
root.title("예제 2 - 라벨 정보 입력")
root.geometry("550x350")


# --------------------------------------------------
# 2. GUI에서 사용할 변수
# --------------------------------------------------

file_name_var = tk.StringVar()
class_var = tk.StringVar()


# --------------------------------------------------
# 3. 제목
# --------------------------------------------------

title_label = tk.Label(
    root,
    text="라벨링 정보 입력 연습",
    font=("Arial", 18)
)

title_label.grid(
    row=0,
    column=0,
    columnspan=2,
    pady=25
)


# --------------------------------------------------
# 4. 파일명 입력
# --------------------------------------------------

file_label = tk.Label(
    root,
    text="이미지 파일명:"
)

file_label.grid(
    row=1,
    column=0,
    padx=20,
    pady=10,
    sticky="e"
)


file_entry = tk.Entry(
    root,
    textvariable=file_name_var,
    width=30
)

file_entry.grid(
    row=1,
    column=1,
    padx=20,
    pady=10
)


# --------------------------------------------------
# 5. Class 선택
# --------------------------------------------------

class_label = tk.Label(
    root,
    text="Class:"
)

class_label.grid(
    row=2,
    column=0,
    padx=20,
    pady=10,
    sticky="e"
)


class_combo = ttk.Combobox(
    root,
    textvariable=class_var,
    values=[
        "leaf",
        "plastic",
        "metal",
        "disease"
    ],
    state="readonly",
    width=27
)

class_combo.grid(
    row=2,
    column=1,
    padx=20,
    pady=10
)


# --------------------------------------------------
# 6. 확인 버튼
# --------------------------------------------------

save_button = tk.Button(
    root,
    text="입력 확인",
    command=show_result,
    width=20,
    height=2
)

save_button.grid(
    row=3,
    column=0,
    columnspan=2,
    pady=20
)


# --------------------------------------------------
# 7. 결과 표시 Label
# --------------------------------------------------

result_label = tk.Label(
    root,
    text="파일명과 Class를 입력해 주세요.",
    font=("Arial", 11)
)

result_label.grid(
    row=4,
    column=0,
    columnspan=2,
    pady=10
)


# --------------------------------------------------
# 8. GUI 실행
# --------------------------------------------------

root.mainloop()
```

---

# 22. 예제 2를 실행합니다

```bash
python app.py
```

다음과 비슷한 화면이 나타납니다.

```text
+------------------------------------------------+
|              라벨링 정보 입력 연습             |
|                                                |
| 이미지 파일명: [ sample_001.jpg            ]  |
|                                                |
|         Class: [ plastic ▼                  ]  |
|                                                |
|                [ 입력 확인 ]                  |
|                                                |
|        파일명: sample_001.jpg                  |
|        선택 Class: plastic                     |
+------------------------------------------------+
```

---

# 23. 예제 2 코드를 이해합니다

## `StringVar`

```python
file_name_var = tk.StringVar()
```

GUI 위젯의 값을 Python에서 쉽게 읽고 바꾸기 위한 변수입니다.

Entry에 연결합니다.

```python
file_entry = tk.Entry(
    root,
    textvariable=file_name_var
)
```

값을 읽을 때는 다음처럼 합니다.

```python
file_name = file_name_var.get()
```

---

# 24. Combobox는 왜 사용하나요?

라벨링에서는 Class명을 학생이 마음대로 입력하면 문제가 생길 수 있습니다.

예를 들어 같은 플라스틱인데 아래처럼 작성할 수 있습니다.

```text
plastic
Plastic
plastics
플라스틱
```

AI 학습에서는 서로 다른 Class로 인식될 수 있습니다.

따라서 미리 정한 Class 목록에서 선택하게 하는 것이 좋습니다.

```python
values=[
    "leaf",
    "plastic",
    "metal",
    "disease"
]
```

---

# 25. `grid()`는 무엇인가요?

첫 번째 예제에서는 `pack()`을 사용했습니다.

두 번째 예제에서는 입력 폼이기 때문에 `grid()`를 사용합니다.

```text
row 0
┌──────────────────────────────┐
│           제목               │
└──────────────────────────────┘

row 1
┌─────────────┬────────────────┐
│ 파일명      │ Entry          │
└─────────────┴────────────────┘

row 2
┌─────────────┬────────────────┐
│ Class       │ Combobox       │
└─────────────┴────────────────┘
```

즉 `grid()`는 표처럼 생각하면 됩니다.

---



## `StringVar`는 GUI와 연결된 상태값이라고 생각합니다

```python
file_name_var = tk.StringVar()
```

웹에서 JavaScript 변수나 상태값을 사용했던 것처럼 Tkinter에서도 화면의 값을 Python에서 읽고 변경해야 합니다.

```python
file_name = file_name_var.get()
```

JavaScript의 다음 코드와 비슷합니다.

```javascript
const fileName = input.value;
```

## `grid()`는 CSS Grid처럼 행과 열로 생각합니다

```python
file_label.grid(row=1, column=0)
file_entry.grid(row=1, column=1)
```

다음 표처럼 배치됩니다.

```text
row=1
┌───────────────┬────────────────────┐
│ column=0      │ column=1           │
│ 파일명 Label  │ Entry              │
└───────────────┴────────────────────┘
```

정확히 CSS Grid와 같은 기술은 아니지만, **행(row)과 열(column)을 기준으로 배치한다는 사고방식**은 같습니다.

---

# 26. 예제 2 Mini Challenge

Class 목록을 다음처럼 변경해봅니다.

```text
leaf
plastic
metal
branch
disease
pepper
```

그리고 버튼을 눌렀을 때 다음 문장이 나오도록 수정합니다.

```text
sample_001.jpg 파일의 Class는 metal입니다.
```

---

# 27. 예제 2 체크리스트

- [ ] Entry에 문자를 입력할 수 있다.
- [ ] Combobox에서 Class를 선택할 수 있다.
- [ ] `StringVar`의 역할을 설명할 수 있다.
- [ ] `.get()`을 사용해 GUI 값을 읽을 수 있다.
- [ ] `grid()`에서 `row`, `column`의 의미를 안다.
- [ ] Class를 자유입력이 아니라 목록 선택으로 만드는 이유를 설명할 수 있다.

---

# PART 4. 예제 3 — Canvas에서 마우스로 BBox 그리기

---

# 28. 예제 3이 중요한 이유


## HTML Canvas와 JavaScript Mouse Event를 떠올립니다

웹 Canvas에서는 다음처럼 Event를 연결할 수 있습니다.

```javascript
canvas.addEventListener("mousedown", onMouseDown);
canvas.addEventListener("mousemove", onMouseMove);
canvas.addEventListener("mouseup", onMouseUp);
```

Tkinter에서는 다음처럼 연결합니다.

```python
canvas.bind("<ButtonPress-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_drag)
canvas.bind("<ButtonRelease-1>", on_mouse_up)
```

비교하면 다음과 같습니다.

```text
JavaScript mousedown  → Tkinter <ButtonPress-1>
JavaScript mousemove  → Tkinter <B1-Motion>
JavaScript mouseup    → Tkinter <ButtonRelease-1>
addEventListener()    → bind()
MouseEvent 좌표       → event.x, event.y
```

예제 3의 핵심은 **“마우스 Event에서 좌표를 얻고, 그 좌표를 이용해 Rectangle을 계속 다시 그린다”**입니다.

세 번째 예제는 교과 7 라벨링 도구와 직접 연결됩니다.

라벨링 프로그램의 핵심 기능은 다음입니다.

```text
이미지 표시
        ↓
마우스 누르기
        ↓
마우스 드래그
        ↓
마우스 놓기
        ↓
BBox 확정
```

이번 예제에서는 실제 이미지는 불러오지 않고 흰색 Canvas 위에 BBox를 그립니다.

그 이유는 먼저 **마우스 이벤트와 BBox 좌표 계산**에 집중하기 위해서입니다.

---

# 29. 예제 3에서 배울 기술

- `Canvas`
- 마우스 이벤트
- `<ButtonPress-1>`
- `<B1-Motion>`
- `<ButtonRelease-1>`
- 시작 좌표
- 종료 좌표
- `create_rectangle()`
- 기존 사각형 삭제

---

# 30. 예제 3의 핵심 개념

마우스로 사각형을 그리려면 두 점이 필요합니다.

```text
마우스를 누른 위치

(x1, y1)
    ●
    │
    │
    │
    └──────────────●
                 (x2, y2)

마우스를 놓은 위치
```

BBox는 다음 네 값으로 만들 수 있습니다.

```text
x1
y1
x2
y2
```

---

# 31. 예제 3 의사코드

```text
Window 생성

        ↓

Canvas 생성

        ↓

마우스를 누른다

        ↓

시작 좌표 저장
start_x
start_y

        ↓

마우스를 드래그한다

        ↓

현재 위치까지
임시 Rectangle 표시

        ↓

마우스를 놓는다

        ↓

종료 좌표 저장
end_x
end_y

        ↓

최종 BBox 좌표 표시
```

---

# 32. 예제 3 파일을 만듭니다

```bash
cd ~/subject07_tkinter_intro/example03_bbox
touch app.py
```

VS Code:

```bash
code app.py
```

---

# 33. 예제 3 코드를 작성합니다

```python
import tkinter as tk


# --------------------------------------------------
# 마우스 시작 좌표와 Rectangle ID를 저장할 변수
# --------------------------------------------------

start_x = 0
start_y = 0

current_rect = None


def on_mouse_down(event):
    """
    마우스 왼쪽 버튼을 누른 순간 실행됩니다.

    event.x
    event.y

    는 Canvas 안에서 현재 마우스 좌표를 의미합니다.
    """

    global start_x, start_y, current_rect

    start_x = event.x
    start_y = event.y

    # 이전에 만들던 임시 Rectangle이 있다면 삭제합니다.
    if current_rect is not None:
        canvas.delete(current_rect)

    # 처음에는 크기가 0인 Rectangle을 만듭니다.
    current_rect = canvas.create_rectangle(
        start_x,
        start_y,
        start_x,
        start_y,
        outline="red",
        width=2
    )

    status_label.config(
        text=f"시작 좌표: ({start_x}, {start_y})"
    )


def on_mouse_drag(event):
    """
    마우스 왼쪽 버튼을 누른 채 움직일 때 실행됩니다.

    마우스의 현재 위치에 맞춰 Rectangle 크기를 계속 변경합니다.
    """

    if current_rect is None:
        return

    current_x = event.x
    current_y = event.y

    canvas.coords(
        current_rect,
        start_x,
        start_y,
        current_x,
        current_y
    )


def on_mouse_up(event):
    """
    마우스 버튼을 놓았을 때 실행됩니다.

    이 순간을 BBox가 확정된 시점으로 생각합니다.
    """

    end_x = event.x
    end_y = event.y

    # 사용자가 오른쪽 → 왼쪽,
    # 아래쪽 → 위쪽으로 드래그할 수도 있기 때문에
    # min, max를 사용해 좌표를 정리합니다.

    x1 = min(start_x, end_x)
    y1 = min(start_y, end_y)

    x2 = max(start_x, end_x)
    y2 = max(start_y, end_y)

    bbox_width = x2 - x1
    bbox_height = y2 - y1

    result_text = (
        f"BBox 좌표: "
        f"({x1}, {y1}) ~ ({x2}, {y2})\n"
        f"Width: {bbox_width}, "
        f"Height: {bbox_height}"
    )

    status_label.config(text=result_text)


def clear_bbox():
    """
    Canvas의 모든 Rectangle을 삭제합니다.
    """

    global current_rect

    canvas.delete("all")
    current_rect = None

    status_label.config(
        text="BBox를 다시 그려보세요."
    )


# --------------------------------------------------
# 1. Window 생성
# --------------------------------------------------

root = tk.Tk()

root.title("예제 3 - BBox 그리기")
root.geometry("800x650")


# --------------------------------------------------
# 2. 제목
# --------------------------------------------------

title_label = tk.Label(
    root,
    text="마우스로 BBox를 그려보세요.",
    font=("Arial", 18)
)

title_label.pack(pady=15)


# --------------------------------------------------
# 3. Canvas
# --------------------------------------------------

canvas = tk.Canvas(
    root,
    width=700,
    height=450,
    bg="white"
)

canvas.pack(pady=10)


# --------------------------------------------------
# 4. Mouse Event 연결
# --------------------------------------------------

canvas.bind(
    "<ButtonPress-1>",
    on_mouse_down
)

canvas.bind(
    "<B1-Motion>",
    on_mouse_drag
)

canvas.bind(
    "<ButtonRelease-1>",
    on_mouse_up
)


# --------------------------------------------------
# 5. 상태 Label
# --------------------------------------------------

status_label = tk.Label(
    root,
    text="흰색 영역에서 마우스를 드래그하세요.",
    font=("Arial", 11)
)

status_label.pack(pady=10)


# --------------------------------------------------
# 6. 삭제 Button
# --------------------------------------------------

clear_button = tk.Button(
    root,
    text="BBox 지우기",
    command=clear_bbox,
    width=20,
    height=2
)

clear_button.pack(pady=10)


# --------------------------------------------------
# 7. GUI 실행
# --------------------------------------------------

root.mainloop()
```

---

# 34. 예제 3을 실행합니다

```bash
python app.py
```

흰색 영역이 나타납니다.

그 안에서 마우스를 누른 상태로 드래그합니다.

```text
┌──────────────────────────────────────────┐
│                                          │
│       ┌────────────────┐                 │
│       │                │                 │
│       │      BBox      │                 │
│       │                │                 │
│       └────────────────┘                 │
│                                          │
└──────────────────────────────────────────┘
```

마우스를 놓으면 아래에 좌표가 표시됩니다.

예:

```text
BBox 좌표: (105, 72) ~ (380, 256)
Width: 275, Height: 184
```

---

# 35. Mouse Event를 이해합니다

Tkinter에서는 특정 행동이 발생하면 함수를 실행할 수 있습니다.

이를 Event라고 합니다.

오늘 사용하는 세 이벤트는 다음과 같습니다.

| Event | 의미 |
|---|---|
| `<ButtonPress-1>` | 마우스 왼쪽 버튼 누르기 |
| `<B1-Motion>` | 왼쪽 버튼을 누른 채 이동 |
| `<ButtonRelease-1>` | 왼쪽 버튼 놓기 |

라벨링 도구에서는 거의 그대로 사용합니다.

```text
ButtonPress
→ BBox 시작점

B1-Motion
→ BBox 크기 변경

ButtonRelease
→ BBox 확정
```

---

# 36. `event.x`, `event.y`는 무엇인가요?

다음 코드를 봅니다.

```python
start_x = event.x
start_y = event.y
```

마우스를 누른 지점의 Canvas 좌표입니다.

예를 들어 다음 지점을 클릭했다고 가정합니다.

```text
Canvas

(0, 0)
  ┌──────────────────────────────
  │
  │       ●
  │     (120, 80)
  │
  │
```

그러면

```python
event.x == 120
event.y == 80
```

이 됩니다.

---

# 37. `create_rectangle()`은 무엇인가요?

```python
canvas.create_rectangle(
    x1,
    y1,
    x2,
    y2
)
```

두 점을 이용하여 사각형을 만듭니다.

```text
(x1, y1)
    ●──────────────┐
    │              │
    │              │
    └──────────────●
                (x2, y2)
```

이 구조가 Object Detection의 BBox와 연결됩니다.

---

# 38. 왜 `min()`과 `max()`를 사용하나요?

학생이 항상 왼쪽 위에서 오른쪽 아래로 드래그한다는 보장은 없습니다.

아래처럼 반대로 그릴 수도 있습니다.

```text
오른쪽 아래에서 시작
        ↓
왼쪽 위로 드래그
```

따라서 좌표를 정리합니다.

```python
x1 = min(start_x, end_x)
x2 = max(start_x, end_x)

y1 = min(start_y, end_y)
y2 = max(start_y, end_y)
```

이렇게 하면 항상 다음 관계를 유지할 수 있습니다.

```text
x1 < x2
y1 < y2
```

---

# 39. BBox의 Width와 Height를 계산합니다

```python
bbox_width = x2 - x1
bbox_height = y2 - y1
```

예:

```text
x1 = 100
x2 = 300

width = 300 - 100
      = 200
```

---

# 40. 이 좌표가 나중에 YOLO 좌표로 변환됩니다

지금까지 만든 값은 Pixel 좌표입니다.

```text
x1
y1
x2
y2
```

YOLO에서는 다음 값을 사용합니다.

```text
class_id
x_center
y_center
width
height
```

그리고 좌표와 크기를 이미지 전체 크기로 나누어 0~1 값으로 바꿉니다.

예:

```text
원본 이미지

Width  = 1000
Height = 500


BBox

x1 = 100
y1 = 50
x2 = 300
y2 = 250
```

먼저 중심점을 계산합니다.

```text
x_center_pixel
= (100 + 300) / 2
= 200

y_center_pixel
= (50 + 250) / 2
= 150
```

BBox 크기:

```text
width_pixel
= 300 - 100
= 200

height_pixel
= 250 - 50
= 200
```

YOLO Normalized 좌표:

```text
x_center
= 200 / 1000
= 0.2

y_center
= 150 / 500
= 0.3

width
= 200 / 1000
= 0.2

height
= 200 / 500
= 0.4
```

Class가 2라면 TXT는 다음처럼 됩니다.

```text
2 0.2 0.3 0.2 0.4
```

이것이 Roboflow에서 Export했던 YOLO Label의 기본 원리입니다.

---

## GUI에 보이는 Rectangle과 실제 Label 데이터는 다릅니다

이 부분은 라벨링 도구에서 매우 중요합니다.

```text
Canvas Rectangle
→ 화면에 보이는 사각형

(x1, y1, x2, y2)
→ Python이 가지고 있는 BBox 데이터

YOLO TXT
→ 디스크에 저장되는 실제 Label 파일
```

화면에 Rectangle이 보인다고 해서 TXT가 자동으로 저장되는 것은 아닙니다. 최종 라벨링 도구에서는 **화면 표시 → 좌표 데이터 → 저장 파일**을 연결하는 코드가 추가됩니다.

## `command`와 `bind`를 구분합니다

```text
Button처럼 단순 클릭
→ command=함수

Canvas에서 마우스 좌표가 필요한 Event
→ widget.bind(Event, 함수)
```

예:

```python
save_button = tk.Button(
    root,
    text="저장",
    command=save_label
)
```

```python
canvas.bind(
    "<ButtonPress-1>",
    on_mouse_down
)
```

---

# 41. 예제 3 Mini Challenge

`status_label`에 BBox의 중심 좌표도 표시하도록 수정해봅니다.

현재:

```text
BBox 좌표: (100, 50) ~ (300, 250)
Width: 200, Height: 200
```

목표:

```text
BBox 좌표: (100, 50) ~ (300, 250)
Center: (200, 150)
Width: 200, Height: 200
```

힌트:

```python
center_x = (x1 + x2) / 2
center_y = (y1 + y2) / 2
```

---

# 42. 예제 3 체크리스트

- [ ] Canvas가 무엇인지 설명할 수 있다.
- [ ] 마우스 클릭 좌표를 읽을 수 있다.
- [ ] 드래그하면서 Rectangle을 그릴 수 있다.
- [ ] BBox의 `(x1, y1, x2, y2)`를 설명할 수 있다.
- [ ] BBox Width와 Height를 계산할 수 있다.
- [ ] YOLO 좌표와 Pixel 좌표의 차이를 설명할 수 있다.
- [ ] Roboflow가 자동으로 처리하던 작업과 연결해 설명할 수 있다.

---

# PART 5. 세 예제를 연결합니다

---

# 43. 지금까지 만든 기능을 비교합니다

## 예제 1

```text
Window
+
Label
+
Button
```

배운 것:

```text
GUI 프로그램의 기본 구조
```

---

## 예제 2

```text
Entry
+
Combobox
+
Button
+
입력값 읽기
```

배운 것:

```text
파일명과 Class 같은
사용자 입력 처리
```

---

## 예제 3

```text
Canvas
+
Mouse Event
+
BBox
```

배운 것:

```text
라벨링 도구의 핵심 동작
```

---

# 44. 세 예제를 합치면 라벨링 Tool이 됩니다

```text
[예제 1]
Window · Button
        +
[예제 2]
Class 선택
        +
[예제 3]
Canvas · BBox
        ↓

Mini Labeling Tool
```

조금 더 확장하면 다음 구조가 됩니다.

```text
Tkinter Window
        │
        ├── 이미지 파일 열기
        │
        ├── Canvas에 이미지 표시
        │
        ├── 마우스로 BBox 작성
        │
        ├── Class 선택
        │
        ├── 이전 이미지
        │
        ├── 다음 이미지
        │
        ├── BBox 삭제
        │
        └── YOLO TXT 저장
```

---

# 45. Roboflow와 오늘 코딩의 관계

기존에 Roboflow에서 다음 과정을 수행했다면

```text
이미지 Upload
        ↓
BBox 작성
        ↓
Class 지정
        ↓
Label 저장
        ↓
YOLO Export
```

오늘은 그 안에서 어떤 일이 발생하는지 일부를 직접 구현했습니다.

```text
Roboflow의 UI
        ↓
Tkinter Window

Roboflow의 BBox
        ↓
Canvas Rectangle

Roboflow의 Class 선택
        ↓
Combobox

Roboflow의 Label Export
        ↓
다음 단계에서 Python으로 TXT 저장
```

따라서 Roboflow와 Tkinter는 서로 반대되는 내용이 아닙니다.

```text
Roboflow
→ 완성된 라벨링 도구를 사용하는 경험

Tkinter 실습
→ 라벨링 도구 내부 원리를 이해하는 경험
```

---

# 46. 오늘의 전체 프로젝트 구조

실습이 끝나면 다음과 같은 구조가 됩니다.

```text
subject07_tkinter_intro/
│
├── .venv/
│
├── example01_basic/
│   └── app.py
│
├── example02_form/
│   └── app.py
│
└── example03_bbox/
    └── app.py
```

확인합니다.

```bash
cd ~/subject07_tkinter_intro

find . -maxdepth 2 -type f
```

예:

```text
./example01_basic/app.py
./example02_form/app.py
./example03_bbox/app.py
```

---

# 47. 세 프로그램을 다시 실행해봅니다

예제 1:

```bash
cd ~/subject07_tkinter_intro/example01_basic
python app.py
```

예제 2:

```bash
cd ~/subject07_tkinter_intro/example02_form
python app.py
```

예제 3:

```bash
cd ~/subject07_tkinter_intro/example03_bbox
python app.py
```

세 프로그램이 모두 정상 실행되는지 확인합니다.

---

# 48. 오류가 발생했을 때 확인합니다

## 오류 1 — `No module named tkinter`

예:

```text
ModuleNotFoundError: No module named 'tkinter'
```

Ubuntu:

```bash
sudo apt update
sudo apt install -y python3-tk
```

설치 후 다시 실행합니다.

---

## 오류 2 — 화면이 나타나지 않습니다

WSL을 사용한다면 먼저 다음을 확인합니다.

```bash
echo $DISPLAY
```

Windows 11 + 최신 WSL에서는 WSLg를 통해 GUI가 실행되는 경우가 많습니다.

그래도 GUI가 나오지 않는다면 수업 PC의 GUI 지원 상태를 강사에게 확인합니다.

---

## 오류 3 — 가상환경이 활성화되지 않았습니다

다시 프로젝트 Root로 이동합니다.

```bash
cd ~/subject07_tkinter_intro
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

확인합니다.

```bash
which python
```

---

## 오류 4 — 프로그램이 종료되지 않습니다

Tkinter Window의 X 버튼을 눌러 종료합니다.

터미널에서 강제로 종료해야 한다면

```text
Ctrl + C
```

를 사용할 수 있습니다.

---

# PART 6. 오늘 수업 마무리

---



# 49. HTML · CSS · JavaScript와 Tkinter 최종 대응표

| 이미 알고 있는 웹 개념 | Tkinter에서의 대응 | 라벨링 도구에서 하는 일 |
|---|---|---|
| HTML 문서 | `Tk()` | 전체 프로그램 Window |
| `<p>` / `<span>` | `Label` | 파일명·상태 표시 |
| `<button>` | `Button` | 저장·이전·다음·삭제 |
| `<input>` | `Entry` | 문자열 입력 |
| `<select>` | `Combobox` | Class 선택 |
| `<canvas>` | `Canvas` | 이미지와 BBox 표시 |
| CSS 배치 | `pack()`, `grid()` | Widget 배치 |
| `onclick` | `command=` | 버튼 클릭 함수 연결 |
| `addEventListener()` | `.bind()` | 마우스 Event 연결 |
| `event.offsetX/Y` | `event.x/y` | Canvas 좌표 읽기 |
| DOM 상태 변경 | `.config()` | Label 내용 변경 |
| 브라우저 Event Loop | `mainloop()` | GUI Event 대기 |

핵심적으로 Tkinter GUI도 다음 구조를 반복합니다.

```text
화면 요소를 만든다
        ↓
화면에 배치한다
        ↓
Event를 연결한다
        ↓
사용자가 행동한다
        ↓
Python 함수가 실행된다
        ↓
화면 또는 데이터가 변경된다
```

이 흐름을 이해하면 Tkinter 전체 문법을 외우지 않아도 교과 7 Mini Labeling Tool을 따라갈 수 있습니다.

---

# 50. 오늘 꼭 이해해야 하는 핵심

오늘은 Tkinter 문법 전체를 배우는 것이 목적이 아닙니다.

다음 흐름을 이해하는 것이 중요합니다.

```text
GUI Window
        ↓
Widget 배치
        ↓
사용자 입력
        ↓
Event 발생
        ↓
Python 함수 실행
        ↓
화면 또는 데이터 변경
```

그리고 라벨링 도구에서는 다음처럼 연결됩니다.

```text
마우스 Event
        ↓
BBox 좌표
        ↓
Class
        ↓
Label Data
        ↓
YOLO Dataset
        ↓
교과 8 Object Detection
```

---

# 51. 오늘의 최종 자가 체크리스트

아래 항목을 직접 확인합니다.

- [ ] `.venv` 가상환경을 만들 수 있다.
- [ ] 가상환경을 활성화할 수 있다.
- [ ] Tkinter Window를 만들 수 있다.
- [ ] Label과 Button을 화면에 배치할 수 있다.
- [ ] Button 클릭 시 Python 함수를 실행할 수 있다.
- [ ] Entry에서 입력값을 가져올 수 있다.
- [ ] Combobox에서 Class를 선택할 수 있다.
- [ ] Canvas의 역할을 설명할 수 있다.
- [ ] 마우스 Event 세 가지를 설명할 수 있다.
- [ ] 마우스로 BBox를 그릴 수 있다.
- [ ] BBox의 `(x1, y1, x2, y2)`를 설명할 수 있다.
- [ ] BBox의 Width와 Height를 계산할 수 있다.
- [ ] Pixel BBox와 YOLO Normalized Label의 차이를 설명할 수 있다.
- [ ] Roboflow의 BBox 작업과 오늘 작성한 코드의 관계를 설명할 수 있다.

---

# 52. 다음 실습 예고

다음 단계에서는 오늘 만든 세 가지 예제를 하나로 합칩니다.

```text
이미지 폴더 열기
        ↓
이미지 Canvas 표시
        ↓
마우스로 BBox 작성
        ↓
Class 선택
        ↓
YOLO TXT 저장
        ↓
이전 · 다음 이미지
        ↓
저장 Label 다시 불러오기
```

최종적으로 다음과 같은 **Mini Labeling Tool**을 직접 만듭니다.

```text
+------------------------------------------------------+
| Image: sample_001.jpg          [이전] [다음]         |
|                                                      |
|  +-----------------------------------------------+   |
|  |                                               |   |
|  |              Image Canvas                     |   |
|  |                                               |   |
|  |       ┌─────────────┐                         |   |
|  |       │    BBox     │                         |   |
|  |       └─────────────┘                         |   |
|  |                                               |   |
|  +-----------------------------------------------+   |
|                                                      |
| Class: [ plastic ▼ ]   [BBox 삭제]   [Label 저장]   |
+------------------------------------------------------+
```

이 단계까지 진행하면 교과 7의 라벨링 도구 구조와 교과 8의 Object Detection 데이터가 어떻게 연결되는지 훨씬 명확하게 이해할 수 있습니다.
