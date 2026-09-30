### 이해를 돕기 

| 구분                | Tkinter               | Streamlit                      |
| ----------------- | --------------------- | ------------------------------ |
| 종류                | 데스크톱 GUI              | 웹 앱 프레임워크                      |
| 실행 결과             | PC 프로그램 창             | 브라우저 화면                        |
| 배우기 난이도           | 이벤트/위젯 구조 때문에 조금 더 복잡 | 비교적 쉬움                         |
| 마우스 드래그/Canvas    | 강함                    | 기본만으로는 제한적                     |
| 이미지 위 BBox 직접 그리기 | 적합                    | 가능하지만 별도 컴포넌트 필요할 수 있음         |
| 상태 관리             | 직접 관리                 | Streamlit 방식의 session_state 사용 |
| 배포                | 데스크톱 앱 형태             | 웹으로 공유하기 좋음                    |
| 데이터 대시보드          | 보통                    | 매우 좋음                          |

### 같은 화면 기능을 세 방식으로 만들면 문법이 어떻게 다른가

|기능|Tkinter|Streamlit|HTML|
|---|---|---|---|
|제목 표시|`Label(root, text="라벨링 도구")`|`st.title("라벨링 도구")`|`<h1>라벨링 도구</h1>`|
|버튼|`Button(root, text="저장")`|`st.button("저장")`|`<button>저장</button>`|
|입력창|`Entry(root)`|`st.text_input("이름")`|`<input type="text">`|
|선택 목록|`ttk.Combobox(...)`|`st.selectbox("Class", options)`|`<select><option>...</option></select>`|
|이미지 표시|`Canvas` 또는 `Label`에 이미지 배치|`st.image(img)`|`<img src="image.jpg">`|
|화면 배치|`pack()`, `grid()`, `place()`|위에서 아래로 자동 배치|HTML 태그 + CSS|
|클릭 이벤트|`command=함수명`|`if st.button(...):`|JavaScript의 `onclick`|
|마우스 드래그|`canvas.bind("<B1-Motion>", func)`|기본 기능만으로는 제한적|JavaScript 이벤트 필요|
|실행 방식|`root.mainloop()`|`streamlit run app.py`|브라우저에서 HTML 실행|

### 버튼을 눌렀을 때 동작

|Tkinter|Streamlit|HTML + JS|
|---|---|---|
|`Button(root, text="저장", command=save)`|`if st.button("저장"):`|`<button onclick="save()">저장</button>`|

쉽게 비교하면 다음과 같습니다.

```
Tkinter
버튼에 실행할 함수를 연결합니다.

Streamlit
버튼이 눌렸는지 if문으로 확인합니다.

HTML
JavaScript 함수를 onclick에 연결합니다.
```

Tkinter에서는 다음처럼 먼저 함수를 만들고,

```python
def save():
    print("저장합니다.")
```

버튼에 연결합니다.

```python
save_button = Button(
    root,
    text="저장",
    command=save
)
```

여기서 중요한 것은 다음 차이입니다.

```python
command=save
```

는 버튼을 눌렀을 때 `save()` 함수를 실행하라는 뜻입니다.

반면

```python
command=save()
```

처럼 괄호를 붙이면 버튼을 누르기 전에 함수가 먼저 실행될 수 있으므로 주의합니다.

---

### 2. 이미지 표시 방식은 어떻게 다른가요?

세 방식 모두 이미지를 화면에 보여줄 수 있지만, **이미지 위에서 마우스로 BBox를 직접 그리는 작업**에서는 차이가 큽니다.

|Tkinter|Streamlit|HTML|
|---|---|---|
|`Canvas` 또는 `Label` 사용|`st.image()` 사용|`<img>` 사용|
|이미지 위에 직접 사각형 그리기 쉬움|기본 기능만으로는 제한적|JavaScript 또는 Canvas 필요|
|마우스 이벤트 연결 쉬움|별도 컴포넌트가 필요할 수 있음|JavaScript 필요|

단순히 이미지만 보여준다면 Streamlit이 매우 쉽습니다.

```python
st.image(image)
```

HTML도 간단합니다.

```html
<img src="image.jpg">
```

하지만 이번 프로젝트에서는 단순히 이미지를 보여주는 것보다

```
이미지 표시
→ 기존 BBox 표시
→ 마우스로 새 BBox 작성
→ BBox 선택
→ 삭제
→ Class 변경
```

을 해야 합니다.

그래서 Tkinter에서는 `Canvas`가 핵심입니다.

```python
canvas = tk.Canvas(
    root,
    width=900,
    height=700
)
```

---

### 3. Canvas는 무엇인가요?

`Canvas`는 화면에 이미지, 선, 사각형, 글자 등을 직접 그릴 수 있는 공간입니다.

이번 프로젝트에서는 Canvas를 다음 용도로 사용합니다.

```
Canvas
├─ 이미지 표시
├─ 기존 BBox 표시
├─ 새 BBox 작성
├─ Class 이름 표시
├─ BBox 선택
└─ Zoom / Pan
```

예를 들어 사각형을 하나 그리려면 다음처럼 작성합니다.

```python
canvas.create_rectangle(
    100,
    100,
    300,
    250,
    outline="red",
    width=2
)
```

좌표는 다음 의미입니다.

```
(x1, y1)
   ┌──────────────┐
   │              │
   │              │
   └──────────────┘
             (x2, y2)
```

즉,

```python
canvas.create_rectangle(
    x1,
    y1,
    x2,
    y2
)
```

형태로 사용합니다.

---

### 4. 마우스로 BBox를 그리려면 어떻게 하나요?

이번 프로젝트에서 가장 중요한 Tkinter 기능 중 하나입니다.

사용자가 마우스를 눌렀다가 드래그하고 놓으면 BBox를 만들 수 있습니다.

필요한 이벤트는 다음 세 가지입니다.

|이벤트|의미|
|---|---|
|`<ButtonPress-1>`|마우스 왼쪽 버튼을 누름|
|`<B1-Motion>`|왼쪽 버튼을 누른 상태로 움직임|
|`<ButtonRelease-1>`|왼쪽 버튼을 놓음|

전체 흐름은 다음과 같습니다.

```
마우스 누름
→ 시작 좌표 저장

마우스 드래그
→ 임시 사각형 크기 변경

마우스 놓음
→ 끝 좌표 저장
→ BBox 확정
```

예:

```python
canvas.bind("<ButtonPress-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_move)
canvas.bind("<ButtonRelease-1>", on_mouse_up)
```

이처럼 Tkinter에서는 `bind()`를 이용해 마우스 동작과 함수를 연결합니다.

---

### 5. Streamlit과 Tkinter의 가장 큰 차이는 무엇인가요?

Streamlit은 **데이터 결과를 빠르게 보여주는 웹 화면**을 만드는 데 강합니다.

예:

```
표
그래프
이미지
모델 결과
대시보드
```

반면 Tkinter는 **마우스로 직접 조작하는 데스크톱 프로그램**을 만들 때 유리합니다.

이번 프로젝트를 기준으로 보면 다음처럼 이해하면 됩니다.

```
Streamlit
→ 결과를 보여주는 데 강함

Tkinter
→ 사용자가 직접 화면을 조작하는 데 강함
```

예를 들면:

```
Streamlit
모델 추론 결과
그래프
통계
로그 표시

Tkinter
이미지 열기
BBox 그리기
BBox 수정
Class 선택
저장
```

---

### 6. 화면에 여러 요소는 어떻게 배치하나요?

Tkinter에서는 위젯을 만든 뒤 화면에 배치해야 합니다.

대표적인 방법은 다음 세 가지입니다.

|방법|특징|
|---|---|
|`pack()`|위·아래·좌·우로 간단하게 배치|
|`grid()`|표처럼 행·열 기준으로 배치|
|`place()`|좌표로 직접 배치|

예:

```python
button.pack()
```

좌우 배치:

```python
prev_button.pack(side="left")
next_button.pack(side="left")
save_button.pack(side="left")
```

표처럼 배치:

```python
label.grid(row=0, column=0)
entry.grid(row=0, column=1)
```

처음에는 `pack()`과 `grid()`만 이해하면 충분합니다.

---

### 7. Frame은 왜 사용하나요?

화면이 복잡해질수록 기능별로 영역을 나누는 것이 좋습니다.

예를 들어 라벨링 프로그램은 다음처럼 나눌 수 있습니다.

```
┌──────────────────────────────┐
│ 상단 버튼 영역               │
├─────────────────────┬────────┤
│                     │ Class  │
│                     │        │
│      이미지          │ 상태   │
│      Canvas         │        │
│                     │ Note   │
│                     │        │
├─────────────────────┴────────┤
│ 상태 표시 영역               │
└──────────────────────────────┘
```

이를 만들 때 `Frame`을 사용합니다.

```python
top_frame = tk.Frame(root)
main_frame = tk.Frame(root)
right_frame = tk.Frame(root)
```

즉, `Frame`은 여러 위젯을 묶어 관리하는 **화면 구역**이라고 이해하면 됩니다.

---

### 8. Class 선택은 어떻게 하나요?

이번 프로젝트에서는 BBox를 그린 뒤 해당 객체의 Class를 선택해야 합니다.

이때 `Combobox`를 사용할 수 있습니다.

```python
from tkinter import ttk
```

예:

```python
class_box = ttk.Combobox(
    root,
    values=[
        "0 - 나뭇잎·종이류",
        "1 - 플라스틱·돌·금속",
        "2 - 나뭇가지류",
        "3 - 벌레류",
        "4 - 고무장갑",
        "5 - 병해·갈변",
        "6 - 파·고추"
    ],
    state="readonly"
)
```

현재 선택한 값을 확인하려면

```python
selected = class_box.get()
```

을 사용합니다.

---

### 9. 입력창은 어떻게 만드나요?

Issue, Note 같은 내용을 입력받을 때는 `Entry`를 사용할 수 있습니다.

```python
note_entry = tk.Entry(root)
```

입력한 값을 읽을 때:

```python
note = note_entry.get()
```

내용을 지울 때:

```python
note_entry.delete(0, tk.END)
```

---

### 10. 경고창은 어떻게 띄우나요?

저장하지 않은 상태에서 다음 이미지로 넘어가려고 할 때처럼 경고가 필요할 수 있습니다.

```python
from tkinter import messagebox
```

경고창:

```python
messagebox.showwarning(
    "경고",
    "저장하지 않은 변경사항이 있습니다."
)
```

예/아니오 확인:

```python
answer = messagebox.askyesno(
    "확인",
    "저장하지 않고 이동하시겠습니까?"
)
```

---

### 11. 이번 프로젝트에서는 무엇을 먼저 기억하면 되나요?

Tkinter 문법 전체를 처음부터 외울 필요는 없습니다.

이번 프로젝트에서는 다음 순서로 이해하면 충분합니다.

```
Tk()
→ 창 만들기

Frame
→ 화면 영역 나누기

Canvas
→ 이미지와 BBox 표시

Button
→ 이전 / 다음 / 저장

Combobox
→ Class 선택

bind()
→ 마우스 클릭 / 드래그

Entry
→ Note 입력

messagebox
→ 저장 경고

mainloop()
→ 프로그램 실행 유지
```

---

### 12. 이번 프로젝트에서 가장 중요한 Tkinter 기능

|기능|Tkinter 문법|
|---|---|
|메인 창|`tk.Tk()`|
|화면 영역|`tk.Frame()`|
|글자|`tk.Label()`|
|버튼|`tk.Button()`|
|이미지·BBox 영역|`tk.Canvas()`|
|Class 선택|`ttk.Combobox()`|
|입력창|`tk.Entry()`|
|버튼 함수 연결|`command=함수명`|
|마우스 이벤트|`bind()`|
|BBox 그리기|`create_rectangle()`|
|객체 삭제|`canvas.delete()`|
|값 변경|`config()`|
|경고창|`messagebox`|
|프로그램 실행|`mainloop()`|

---

### 13. 이번 프로젝트에서는 Tkinter 문법보다 더 중요한 것이 있습니다

Tkinter 문법을 외우는 것이 목표는 아닙니다.

실제 프로젝트에서 더 중요한 것은 다음 흐름을 정확하게 만드는 것입니다.

```
이미지 Load
        ↓
기존 TXT Load
        ↓
BBox 표시
        ↓
사용자 수정
        ↓
Class 선택
        ↓
좌표 변환
        ↓
YOLO TXT 저장
        ↓
다시 Load
        ↓
같은 위치에 BBox 복원
```

따라서 문법이 기억나지 않으면 찾아봐도 됩니다.

중요한 것은

```
왜 이 Widget을 사용하는지
어떤 데이터가 들어오는지
무엇을 화면에 보여주는지
수정한 결과를 어디에 저장하는지
```

를 이해하는 것입니다.

> **Tkinter는 이번 프로젝트의 목표가 아니라, 라벨링 프로그램을 만들기 위해 사용하는 도구입니다.**