우리가 최종적으로 만들 프로그램은 이런 모습입니다.

![[Pasted image 20261002080044.png]]

처음 보면 복잡해 보이지만 실제 프로그램은 몇 가지 부품으로 나누어 생각할 수 있습니다.

```
변수
+
함수
+
화면 부품(Widget)
+
이벤트 연결
+
mainloop()
```

이 다섯 가지를 이해하면 Tkinter 프로그램의 큰 구조를 이해한 것입니다.

---

# 1. 가장 먼저 Tkinter 프로그램의 뼈대를 이해합니다

가장 작은 Tkinter 프로그램은 다음과 같습니다.

```python
import tkinter as tk

root = tk.Tk()

root.title("라벨링 연습")
root.geometry("800x650")

root.mainloop()
```

이 코드는 아직 아무 기능도 없습니다.

단지 **프로그램 창 하나를 만드는 코드**입니다.

흐름으로 보면 다음과 같습니다.

```
Tkinter 가져오기
        ↓
프로그램 창 만들기
        ↓
창 제목 설정
        ↓
창 크기 설정
        ↓
프로그램 실행 유지
```

각 코드를 하나씩 보면 어렵지 않습니다.

```python
import tkinter as tk
```

Tkinter라는 GUI 도구를 가져옵니다.

앞으로 Tkinter 기능을 사용할 때 `tk.`를 붙입니다.

그래서 다음과 같은 코드들이 등장합니다.

```python
tk.Tk()
tk.Label()
tk.Button()
tk.Canvas()
```

---

# 2. root는 무엇인가요?

```python
root = tk.Tk()
```

`root`는 우리가 만든 **메인 프로그램 창**을 기억하는 변수입니다.

쉽게 생각하면 다음과 같습니다.

```
root
 ↓
┌──────────────────────────┐
│                          │
│      프로그램 전체 창      │
│                          │
└──────────────────────────┘
```

그리고

```c
root.title("라벨링 연습")
```

은 그 창의 제목을 설정합니다.

```python
root.geometry("800x650")
```

은 창 크기를 설정합니다.

따라서 `root`는 앞으로 우리가 만들 버튼, 글자, Canvas 등을 담는 **가장 큰 상자**라고 생각하면 됩니다.

---

# 3. Tkinter에서 가장 먼저 기억할 구조

Tkinter 프로그램은 크게 다음처럼 생각하면 됩니다.

```
root
│
├── Label
│
├── Button
│
├── Canvas
│
└── Label
```

예를 들어:

![[Pasted image 20261002080555.png]]

여기서 `Label`, `Button`, `Canvas` 같은 것을 **Widget(위젯)**이라고 합니다.

즉,

```
Widget
= 화면에 배치하는 부품
```

입니다.

---

# 4. HTML을 배웠으니 이렇게 연결해서 생각하면 쉽습니다

| 웹에서 배운 것    | Tkinter에서 비슷한 역할 |
| ----------- | ---------------- |
| HTML        | 화면 부품 만들기        |
| CSS         | 색상·크기·배치 설정      |
| JavaScript  | 버튼 클릭·마우스 이벤트 처리 |
| `<button>`  | `tk.Button()`    |
| `<canvas>`  | `tk.Canvas()`    |
| 글자 표시       | `tk.Label()`     |
| `onclick`   | `command=`       |
| mouse event | `bind()`         |

예를 들어 웹에서

```html
<button onclick="clearBBox()">
    BBox 삭제
</button>
```

같은 생각을 했다면 Tkinter에서는

```python
tk.Button(
    root,
    text="BBox 삭제",
    command=clear_bbox
)
```

처럼 생각하면 됩니다.

완전히 같은 문법은 아니지만 **사고방식은 상당히 비슷합니다.**

---

# 5. 첫 번째 화면 부품을 하나 추가해봅니다

기존 코드에 제목을 하나 넣어보겠습니다.

```python
import tkinter as tk

root = tk.Tk()

root.title("라벨링 연습")
root.geometry("800x650")

title_label = tk.Label(
    root,
    text="BBox 라벨링 연습"
)

title_label.pack()

root.mainloop()
```

여기서

```python
tk.Label()
```

은 화면에 글자를 만드는 기능입니다.

그리고

```python
title_label
```

이라는 변수에 그 Label을 기억시켰습니다.

즉,

```
title_label
     ↓
"BBox 라벨링 연습"이라는 Label
```

입니다.

---

# 6. 여기서 변수가 왜 필요한지 이해합니다

Python을 막 배웠다면 변수부터 어렵게 느껴질 수 있습니다.

변수는 일단 **어떤 값을 기억해 두는 이름표**라고 생각하면 됩니다.

예를 들어:

```python
start_x = 100
start_y = 80
```

이라고 하면

```
start_x ──→ 100
start_y ──→ 80
```

입니다.

라벨링 프로그램에서는 특히 다음 값을 기억해야 합니다.

```
마우스를 어디서 눌렀는가?
→ start_x
→ start_y

현재 그리고 있는 사각형은 무엇인가?
→ current_rect
```

그래서 다음 변수가 등장합니다.

```python
start_x = 0
start_y = 0

current_rect = None
```

아직 마우스를 누르지 않았기 때문에 시작 좌표는 우선 `0`으로 놓습니다.

아직 사각형도 없으므로

```python
current_rect = None
```

으로 놓습니다.

`None`은 여기에서는 간단히

> **아직 아무것도 없다**

라고 이해하면 충분합니다.

---

# 7. 이제 Canvas를 하나 만들어봅니다

라벨링 프로그램에서 가장 중요한 공간입니다.

```python
canvas = tk.Canvas(
    root,
    width=700,
    height=450,
    bg="white"
)

canvas.pack()
```

Canvas는 **그림을 그릴 수 있는 도화지**라고 생각하면 됩니다.

![[Pasted image 20261002081014.png]]

나중에는 이 Canvas에 이미지를 보여주고 그 위에 BBox를 그리게 됩니다.

---

# 8. BBox는 사실 사각형 하나입니다

라벨링에서 사용하는 Bounding Box도 처음에는 어렵게 생각할 필요가 없습니다.

```
마우스를 누른 위치
(start_x, start_y)

       ↓

       ●─────────────┐
       │             │
       │    BBox     │
       │             │
       └─────────────●
                  현재 마우스 위치
```

Tkinter에서는 사각형을 다음처럼 만들 수 있습니다.

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

의미는

```
왼쪽 위        오른쪽 아래

(100,100) ───────────┐
    │                │
    │                │
    │                │
    └─────────── (300,250)
```

입니다.

BBox도 결국 이 좌표 네 개를 구하는 작업입니다.

![[Pasted image 20261002081222.png]]

---

# 9. 그런데 마우스 좌표는 누가 알려줄까요?

Tkinter가 알려줍니다.

사용자가 Canvas를 클릭하면 Tkinter가 그 정보를 `event`라는 객체에 담아 함수에 전달합니다.

```python
def on_mouse_down(event):

    print(event.x)
    print(event.y)
```

여기에서

```python
event.x
event.y
```

는 Canvas 안에서 마우스를 클릭한 위치입니다.

예를 들어 사용자가 여기에서 클릭했다면

```
Canvas

(0,0)
┌──────────────────────────────
│
│
│        ● ← 클릭
│      (120,80)
│
│
```

Tkinter가 대략 다음 정보를 전달합니다.

```
event.x → 120
event.y → 80
```

그래서 함수 안에서

```python
start_x = event.x
start_y = event.y
```

라고 저장합니다.

---

# 10. 함수는 무엇인가요?

일단 함수를 **특정 일이 생겼을 때 실행할 작업 묶음**이라고 생각하면 됩니다.

예를 들어:

```python
def clear_bbox():
    canvas.delete("all")
```

은

```
clear_bbox

하는 일
   ↓
Canvas의 그림을 지운다
```

라는 기능 하나를 만들어 놓은 것입니다.

중요한 것은 함수를 만들었다고 바로 실행되는 것이 아니라는 점입니다.

```python
def clear_bbox():
    canvas.delete("all")
```

이것은

> "`clear_bbox`라는 기능을 준비해 두었다."

는 뜻입니다.

---

# 11. 그러면 함수는 언제 실행될까요?

버튼과 연결해 봅니다.

```python
clear_button = tk.Button(
    root,
    text="BBox 삭제",
    command=clear_bbox
)

clear_button.pack()
```

여기가 Tkinter에서 매우 중요한 부분입니다.

```
Button
"BBox 삭제"
       │
       │ command
       ↓
clear_bbox()
       │
       ↓
Canvas 삭제
```

즉,

```
사용자
  ↓
버튼 클릭
  ↓
Tkinter가 클릭 감지
  ↓
command=clear_bbox
  ↓
clear_bbox 함수 실행
  ↓
BBox 삭제
```

입니다.

이 구조를 이해하면 GUI 프로그램이 훨씬 쉬워집니다.

---

# 12. 주의 — `command=clear_bbox()`가 아닙니다

초보자가 정말 많이 실수하는 부분입니다.

버튼에는

```python
command=clear_bbox
```

라고 작성합니다.

다음처럼 작성하지 않습니다.

```python
command=clear_bbox()
```

왜냐하면

```python
command=clear_bbox

→ 버튼을 눌렀을 때 실행해 주세요.
```

라는 의미이기 때문입니다.

반면

```
command=clear_bbox()

→ 프로그램을 만드는 지금 당장 함수를 실행
```

이라는 의미가 됩니다.

처음에는 이렇게 기억해도 됩니다.

```
Button command에서는

함수 이름만 전달

command=clear_bbox
```

---

# 13. 마우스는 Button의 command와 조금 다릅니다

BBox는 버튼 클릭이 아니라 **Canvas에서 마우스를 누르고 움직이고 놓는 동작**을 사용합니다.

그래서 `bind()`를 사용합니다.

```python
canvas.bind("<Button-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_drag)
canvas.bind("<ButtonRelease-1>", on_mouse_up)
```

---

`bind()`는 Tkinter에서 **“이 이벤트가 발생하면 이 함수를 실행하도록 연결한다”**는 뜻입니다.

예를 들어:

```python
root.bind("<Button-1>", on_mouse_down)
```

이 코드는 이렇게 읽으면 됩니다.

```
<Button-1>
마우스 왼쪽 클릭 이벤트

        ↓ bind로 연결

on_mouse_down
실행할 함수
```

즉,

```
이벤트
+
함수

를 연결한다
```

라고 이해하면 됩니다.

다만 더 정확히 말하면 `bind()`는 그냥 함수와 함수를 연결하는 것이 아니라,

> **이벤트와 함수(Event Handler)를 연결하는 기능**

입니다.

그래서 이렇게 이해하면 쉽습니다.

```
.bind()
= 사용자 행동과 함수를 연결한다
```

예를 들면:

```python
canvas.bind("<Button-1>", on_mouse_down)
```

```
Canvas에서 왼쪽 마우스를 누름
        ↓
on_mouse_down 함수 실행
```

그리고

```python
canvas.bind("<B1-Motion>", on_mouse_drag)
```

은

```
왼쪽 버튼을 누른 채 마우스를 움직임
        ↓
on_mouse_drag 함수 실행
```

입니다.

버튼의 `command`와 비교하면 더 쉽게 기억할 수 있습니다.

```
Button
command=함수
→ 버튼 클릭과 함수 연결

Canvas
bind(이벤트, 함수)
→ 마우스/키보드 이벤트와 함수 연결
```

그래서 한 문장으로 정리하면:

> **`bind()`는 “이 행동이 발생하면 이 함수를 실행해”라고 연결해 주는 기능입니다.**

---

각각 다음 뜻입니다.

```
<Button-1>
왼쪽 마우스를 누름

<B1-Motion>
왼쪽 마우스를 누른 채 이동

<ButtonRelease-1>
왼쪽 마우스를 놓음
```

```
<Button-1>  → 마우스 왼쪽 버튼 클릭
<Button-2>  → 마우스 가운데 버튼 클릭
<Button-3>  → 마우스 오른쪽 버튼 클릭
```

따라서 프로그램은 이렇게 연결됩니다.

```
마우스 누름
    ↓
on_mouse_down()
    ↓
시작 좌표 저장


마우스 이동
    ↓
on_mouse_drag()
    ↓
사각형 크기 변경


마우스 놓음
    ↓
on_mouse_up()
    ↓
BBox 좌표 확정
```

이것이 라벨링 프로그램의 핵심입니다.

---

# 14. 아주 작은 BBox 프로그램을 만들어봅니다

처음부터 복잡한 라벨링 도구를 만들지 않고 다음 정도만 만들어봅니다.

```python
import tkinter as tk


# -------------------------
# 변수
# -------------------------

start_x = 0
start_y = 0

current_rect = None


# -------------------------
# 함수
# -------------------------

def on_mouse_down(event):

    global start_x, start_y, current_rect

    start_x = event.x
    start_y = event.y

    current_rect = canvas.create_rectangle(
        start_x,
        start_y,
        start_x,
        start_y,
        outline="red",
        width=2
    )


def on_mouse_drag(event):

    if current_rect is None:
        return

    canvas.coords(
        current_rect,
        start_x,
        start_y,
        event.x,
        event.y
    )


def on_mouse_up(event):

    end_x = event.x
    end_y = event.y

    status_label.config(
        text=f"BBox: ({start_x}, {start_y}) ~ ({end_x}, {end_y})"
    )


def clear_bbox():

    global current_rect

    canvas.delete("all")

    current_rect = None

    status_label.config(
        text="BBox를 다시 그려보세요."
    )


# -------------------------
# 화면 만들기
# -------------------------

root = tk.Tk()

root.title("BBox 라벨링 연습")
root.geometry("800x650")


title_label = tk.Label(
    root,
    text="마우스로 BBox를 그려보세요."
)

title_label.pack()


canvas = tk.Canvas(
    root,
    width=700,
    height=450,
    bg="white"
)

canvas.pack()


clear_button = tk.Button(
    root,
    text="BBox 삭제",
    command=clear_bbox
)

clear_button.pack()


status_label = tk.Label(
    root,
    text="아직 BBox가 없습니다."
)

status_label.pack()


# -------------------------
# 마우스 이벤트 연결
# -------------------------

canvas.bind("<Button-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_drag)
canvas.bind("<ButtonRelease-1>", on_mouse_up)


# -------------------------
# 프로그램 실행
# -------------------------

root.mainloop()
```

```
프로그램이 기억해야 하는 값
        ↓
BBox 시작 위치
현재 Rectangle
```

# 15. 이 프로그램을 다섯 덩어리로 읽습니다

### ① 변수 — 무엇을 기억해야 하는가?

```python
start_x = 0
start_y = 0
current_rect = None
```

```
프로그램이 기억해야 하는 값
        ↓
BBox 시작 위치
현재 Rectangle
```

### ② 함수 — 프로그램이 할 수 있는 일은 무엇인가?

```python
on_mouse_down()
on_mouse_drag()
on_mouse_up()
clear_bbox()
```

```
마우스를 누르면?
마우스를 움직이면?
마우스를 놓으면?
삭제 버튼을 누르면?
```

각 상황에서 해야 할 일을 미리 만들어 놓습니다.

### ③ Widget — 화면에 무엇이 보이는가?

```python
tk.Label()
tk.Canvas()
tk.Button()
tk.Label()
```

즉,

```
제목
Canvas
삭제 버튼
상태 메시지
```

를 화면에 만듭니다.

### ④ 연결 — 어떤 행동이 어떤 함수를 실행하는가?

```python
command=clear_bbox
```

```python
canvas.bind("<Button-1>", on_mouse_down)
```

```python
canvas.bind("<B1-Motion>", on_mouse_drag)
```

```python
canvas.bind("<ButtonRelease-1>", on_mouse_up)
```

즉,

```
사용자의 행동
        ↓
함수
```

를 연결합니다.

### ⑤ mainloop — 이제 사용자의 행동을 기다립니다

```python
root.mainloop()
```

여기까지 만들어 놓으면 프로그램은 종료되지 않고 기다립니다.

```
"버튼 누르나?"

"마우스 클릭하나?"

"마우스 움직이나?"

"창 닫나?"
```

를 계속 기다리는 것입니다.

---

# 16. 그래서 Tkinter 프로그램의 핵심 구조는 이것입니다

```
┌─────────────────────────────────────────┐
│          Tkinter 프로그램                │
└─────────────────────────────────────────┘

        ① 변수
          │
          │ 프로그램 상태 기억
          ↓
 start_x / start_y / current_rect


        ② 함수
          │
          │ 해야 할 행동 정의
          ↓
 on_mouse_down()
 on_mouse_drag()
 on_mouse_up()
 clear_bbox()


        ③ Widget
          │
          │ 화면 구성
          ↓
 Label
 Canvas
 Button


        ④ 이벤트 연결
          │
          ├── Button command
          │
          └── Canvas bind
          ↓

 사용자의 행동 ─────────────→ 함수 실행


        ⑤ mainloop()
          │
          ↓
 사용자 입력을 계속 기다림
```

---

# 17. 실제 BBox 하나가 만들어지는 순간을 따라가 봅니다

사용자가 `(100, 80)`에서 마우스를 누릅니다.

```
마우스 DOWN
(100,80)
     ↓
on_mouse_down(event)
     ↓
start_x = 100
start_y = 80
     ↓
Rectangle 생성
```

그리고 `(300, 250)`까지 마우스를 움직입니다.

```
마우스 DRAG
(300,250)
     ↓
on_mouse_drag(event)
     ↓
canvas.coords()
     ↓

(100,80)
     ●──────────────┐
     │              │
     │    BBox      │
     │              │
     └──────────────●
                 (300,250)
```

그리고 마우스를 놓습니다.

```
마우스 UP
     ↓
on_mouse_up(event)
     ↓
end_x = 300
end_y = 250
     ↓
BBox 좌표 확정
```

결국 행동은 단순합니다.

```
누른다
  ↓
움직인다
  ↓
놓는다
```

프로그램에서는 이것을

```python
on_mouse_down()
       ↓
on_mouse_drag()
       ↓
on_mouse_up()
```

세 함수로 나눈 것입니다.

---

# 18. `global`

다음 코드가 나옵니다.

```python
global start_x, start_y, current_rect
```

```
함수 밖에서 만든 변수

start_x
start_y
current_rect

        ↓

함수 안에서 그 값을
변경해서 계속 사용하고 싶다.

        ↓

global
```

즉,

> "`global`은 여기서는 여러 이벤트 함수가 같은 BBox 상태를 함께 사용하기 위해 사용한다."

정도로 먼저 이해하면 됩니다.

---

# 19. `status_label.config()`는 무엇인가요?

처음에 Label을 만들었습니다.

```python
status_label = tk.Label(
    root,
    text="아직 BBox가 없습니다."
)
```

그런데 프로그램 실행 중에 글자를 바꾸고 싶습니다.

그래서

```python
status_label.config(
    text="BBox가 만들어졌습니다."
)
```

를 사용합니다.

즉,

```
Label 처음 생성

"아직 BBox가 없습니다."

       ↓

BBox 생성

       ↓

config()

       ↓

"BBox가 만들어졌습니다."
```

입니다.

여기에서 중요한 GUI 개념 하나를 배울 수 있습니다.

```
Widget 생성
     ↓
사용자가 행동
     ↓
함수 실행
     ↓
Widget 상태 변경
```

---

# 20. 결국 GUI 프로그램은 이것의 반복입니다

사실 복잡한 라벨링 프로그램도 본질적으로는 같습니다.

```
화면을 만든다
        ↓
사용자가 무엇인가 한다
        ↓
이벤트가 발생한다
        ↓
연결된 함수가 실행된다
        ↓
변수가 바뀐다
        ↓
화면이 바뀐다
        ↓
다시 사용자의 행동을 기다린다
```

이 구조를 이해하면 Tkinter를 통째로 외울 필요가 없습니다.

---

# 21. HTML/CSS/JS 경험과 다시 연결해봅니다

웹에서 버튼을 눌렀을 때:

```
HTML Button
     ↓
onclick
     ↓
JavaScript 함수
     ↓
화면 변경
```

Tkinter에서는:

```
tk.Button
     ↓
command
     ↓
Python 함수
     ↓
Widget 변경
```

Canvas 마우스 이벤트라면:

```
Canvas
   ↓
bind()
   ↓
Mouse Event
   ↓
Python 함수
   ↓
BBox 변경
```

따라서 **전혀 새로운 사고방식을 배우는 것이 아닙니다.**

이미 웹에서 배운

> **"사용자 행동 → 이벤트 → 함수 실행 → 화면 변경"**

이라는 구조를 Python/Tkinter 방식으로 다시 사용하는 것입니다.

---

# 22. 라벨링 프로그램도 결국 이 구조가 커지는 것입니다

오늘 작은 프로그램에서는

```
Canvas
  ↓
BBox 그리기
  ↓
BBox 삭제
```

만 만들었습니다.

여기에 기능을 하나씩 붙이면 됩니다.

```
Tkinter 기본 창
       ↓
Canvas 추가
       ↓
마우스 좌표 확인
       ↓
BBox 그리기
       ↓
BBox 삭제
       ↓
이미지 열기
       ↓
이미지 Canvas에 표시
       ↓
클래스 선택
       ↓
BBox + 클래스 연결
       ↓
여러 BBox 관리
       ↓
BBox 수정
       ↓
BBox 삭제
       ↓
라벨 저장
       ↓
저장 라벨 다시 읽기
       ↓
이전 이미지
       ↓
다음 이미지
       ↓
진행 상태 표시
       ↓

┌─────────────────────────┐
│   실제 라벨링 프로그램     │
└─────────────────────────┘
```

실제 교과 7의 프로젝트 역시 이미지 표시, 이전·다음 이동, BBox 작성, 
클래스 선택, 여러 BBox 관리, 수정·삭제, 저장·재로딩, 진행상태 표시로 확장되는 구조입니다.

---

# 23. 그래서 지금 외워야 하는 것은 Tkinter 문법이 아닙니다

다음 6문장만 먼저 기억합니다.

1. `root`는 프로그램의 **전체 창**이다.
2. `Label`, `Button`, `Canvas`는 화면을 만드는 **부품(Widget)**이다.
3. 변수는 좌표나 현재 BBox처럼 프로그램이 **기억해야 할 값**이다.
4. 함수는 클릭이나 드래그가 발생했을 때 **실행할 작업**이다.
5. `command`와 `bind`는 **사용자의 행동과 함수를 연결**한다.
6. `mainloop()`는 프로그램이 종료되지 않고 **사용자의 다음 행동을 기다리게 한다.**

이 여섯 가지를 이해했다면 Tkinter의 가장 중요한 구조는 이미 이해한 것입니다.

---

## 마지막으로 보여줄 한 장짜리 핵심 그림

```
                [ 변수 ]
                  │
        프로그램의 상태를 기억
                  │
      start_x / start_y / BBox
                  │
                  ▼
┌─────────────────────────────────────┐
│             Tkinter 화면            │
│                                     │
│  Label                              │
│                                     │
│  ┌────────── Canvas ────────────┐   │
│  │                             │   │
│  │       ┌────────────┐        │   │
│  │       │    BBox    │        │   │
│  │       └────────────┘        │   │
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
│          [ BBox 삭제 ]              │
└─────────────────────────────────────┘
          │               │
          │               │
       Canvas           Button
       bind()           command
          │               │
          ▼               ▼
   마우스 이벤트       버튼 클릭
          │               │
          ▼               ▼
 on_mouse_down()      clear_bbox()
 on_mouse_drag()
 on_mouse_up()
          │               │
          └───────┬───────┘
                  ▼
              변수 변경
                  +
              화면 변경
                  │
                  ▼
              mainloop()
                  │
                  ▼
         다음 사용자 행동 기다림
```

![[Pasted image 20261002082458.png]]

**이 그림이 이해되면 라벨링 프로그램 전체를 배울 준비가 된 것입니다.**

---

라벨링 프로그램에서 함수로 만들 수 있는 행동은 이런 것들입니다.

- **이미지를 연다**  
    사용자가 이미지 열기 버튼을 눌렀을 때 파일을 선택하고 화면에 이미지를 보여주는 행동입니다.
    
- **다음 이미지로 이동한다**  
    현재 이미지를 확인한 뒤 다음 파일을 불러오는 행동입니다.
    
- **이전 이미지로 이동한다**  
    잘못 넘어갔을 때 이전 이미지로 돌아가는 행동입니다.
    
- **마우스를 누른 위치를 기억한다**  
    BBox를 그리기 시작한 첫 좌표를 저장하는 행동입니다.
    
- **마우스를 드래그하면서 사각형을 늘린다**  
    시작 좌표에서 현재 마우스 위치까지 임시 BBox를 계속 바꾸는 행동입니다.
    
- **마우스를 놓으면 BBox를 확정한다**  
    시작점과 끝점을 이용해 실제 Bounding Box 하나를 완성하는 행동입니다.
    
- **BBox를 삭제한다**  
    잘못 그린 사각형을 지우는 행동입니다.
    
- **BBox를 수정한다**  
    이미 그린 사각형의 위치나 크기를 바꾸는 행동입니다.
    
- **Class를 선택한다**  
    현재 BBox가 어떤 종류의 객체인지 지정하는 행동입니다.
    
- **BBox와 Class를 연결한다**  
    단순히 사각형만 만드는 것이 아니라, 이 사각형이 어떤 객체를 의미하는지 함께 기억하는 행동입니다.
    
- **현재 좌표를 화면에 표시한다**  
    BBox의 시작점, 끝점, 너비, 높이 같은 정보를 Label에 보여주는 행동입니다.
    
- **라벨을 저장한다**  
    화면에서 만든 BBox와 Class 정보를 파일로 저장하는 행동입니다.
    
- **기존 라벨을 불러온다**  
    이미지와 함께 이미 저장된 라벨 파일을 읽어서 기존 BBox를 화면에 다시 그리는 행동입니다.
    
- **저장하지 않고 이동하려 할 때 경고한다**  
    수정한 내용이 있는데 저장하지 않고 다음 이미지로 가면 사용자에게 확인하는 행동입니다.
    

정리하면:

```
함수 = 프로그램이 할 수 있는 행동

이미지 열기
다음 이미지
이전 이미지
마우스 누르기
마우스 드래그
마우스 놓기
BBox 만들기
BBox 삭제
Class 선택
저장
불러오기
경고하기
상태 표시
```

그리고 더 중요한 것은 **행동마다 함수 하나를 만든다고 생각하게 하는 것**입니다.

```
사용자 행동
        ↓
프로그램이 해야 할 일
        ↓
함수
```

예를 들어,

```
이미지 열기 버튼 클릭
→ 이미지를 연다
→ 이미지 열기 함수

BBox 삭제 버튼 클릭
→ 선택된 BBox를 지운다
→ BBox 삭제 함수

Canvas에서 마우스를 누름
→ 시작 좌표를 기억한다
→ 마우스 Down 함수

마우스를 드래그
→ 사각형 크기를 계속 바꾼다
→ Drag 함수

마우스를 놓음
→ BBox를 확정한다
→ Mouse Up 함수
```

이렇게 보면 함수 이름부터 짓기 쉬워집니다.

즉, 프로젝트에서 먼저 해야 할 일은 **코드를 짜는 것이 아니라 “이 프로그램이 어떤 행동을 해야 하는가?”를 목록으로 만드는 것**입니다.

---

Tkinter 구현이 어렵다면, 먼저 팀에서 만들고자 하는 인터페이스를 Figma로 설계합니다. 각 화면 요소와 버튼의 역할, 사용자 동작, 필요한 기능을 구체적으로 작성합니다. 

이후 설계한 UI 이미지와 기능 설명을 무료 버전의 Gemini와 같은 생성형 AI에 입력하여 Tkinter 코드 초안을 생성할 수 있습니다. 

단, 생성된 코드를 그대로 사용하는 것이 아니라 변수, 함수, 이벤트 연결, 데이터 흐름을 충분히 리뷰하고 직접 실행·수정·검증한 뒤 프로젝트에 적용합니다.