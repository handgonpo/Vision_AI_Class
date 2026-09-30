# 교과 7 프로젝트용 Tkinter 기초 문법 가이드

내일 교과 7 프로젝트에서 바로 찾아보며 쓸 수 있도록 **“Tkinter 입문서”가 아니라 “라벨링 도구 구현에 필요한 Tkinter 실전 문법”** 기준으로 정리합니다.

현재 프로젝트에서 요구하는 핵심 GUI 기능은 이미지 표시, 이전/다음, Zoom/Pan, 기존 TXT Load, BBox 표시·추가·삭제, Class 선택·변경, 다중 BBox, PASS/EDITED/REVIEW, 저장, 미저장 경고, Validation 등입니다.

따라서 아래 내용도 이 기능을 구현할 수 있는 범위에 맞춥니다.

---

# 0. Tkinter에서 가장 먼저 이해해야 할 것

Tkinter는 Python으로 **데스크톱 GUI 프로그램**을 만드는 라이브러리입니다.

일반 Python 프로그램은 보통 위에서 아래로 실행됩니다.

```python
print("시작")
name = input("이름: ")
print(name)
print("끝")
```

하지만 GUI 프로그램은 조금 다릅니다.

```text
프로그램 실행
        ↓
창 생성
        ↓
버튼, 이미지, 입력창 배치
        ↓
사용자가 행동할 때까지 기다림
        ↓
클릭
드래그
키보드
        ↓
연결된 함수 실행
```

이것을 **이벤트 기반 프로그래밍(Event-driven Programming)**이라고 합니다.

Tkinter에서 가장 중요한 구조는 다음입니다.

```python
import tkinter as tk

root = tk.Tk()

# 화면 구성

root.mainloop()
```

`mainloop()`가 실행되면 프로그램은 종료되는 것이 아니라 **사용자의 이벤트를 계속 기다립니다.**

---

# 1. 가장 기본적인 창 만들기

```python
import tkinter as tk

root = tk.Tk()

root.title("조각김치 라벨링 도구")
root.geometry("1200x800")

root.mainloop()
```

각 코드의 의미입니다.

| 코드 | 의미 |
|---|---|
| `tk.Tk()` | 메인 창 생성 |
| `title()` | 창 제목 |
| `geometry()` | 창 크기 |
| `mainloop()` | GUI 이벤트 대기 시작 |

---

# 2. Widget이란 무엇인가?

화면을 구성하는 부품을 **Widget**이라고 합니다.

이번 프로젝트에서 특히 많이 사용하는 것은 다음입니다.

| Widget | 프로젝트에서 하는 일 |
|---|---|
| `Label` | 파일명, 상태, 번호 표시 |
| `Button` | 이전, 다음, 저장 |
| `Canvas` | 이미지 표시 + BBox 그리기 |
| `Entry` | Issue/Note 입력 |
| `Combobox` | Class 선택 |
| `Frame` | 화면 영역 분리 |
| `Scrollbar` | 필요 시 스크롤 |

가장 중요한 것은 **Canvas**입니다.

---

# 3. Label — 글자 표시

```python
label = tk.Label(root, text="현재 이미지: sample001.jpg")

label.pack()
```

예:

```python
file_label = tk.Label(
    root,
    text="파일 없음",
    font=("Arial", 12)
)

file_label.pack()
```

내용 변경:

```python
file_label.config(text="sample002.jpg")
```

또는

```python
file_label["text"] = "sample002.jpg"
```

프로젝트에서 다음처럼 사용합니다.

```text
현재 파일
125 / 900
현재 Class
PASS / EDITED / REVIEW
```

---

# 4. Button — 버튼

가장 기본적인 버튼입니다.

```python
button = tk.Button(
    root,
    text="저장"
)

button.pack()
```

하지만 버튼은 보통 **눌렀을 때 실행할 함수**를 연결해야 합니다.

```python
def save_label():
    print("저장합니다.")

button = tk.Button(
    root,
    text="저장",
    command=save_label
)

button.pack()
```

중요합니다.

```python
command=save_label
```

이지

```python
command=save_label()
```

이 아닙니다.

차이는 다음입니다.

```text
command=save_label
→ 버튼을 누르면 함수 실행

command=save_label()
→ 프로그램 시작할 때 바로 함수 실행
```

---

# 5. 이전 / 다음 버튼

이번 프로젝트에서 반드시 사용합니다.

```python
images = [
    "001.jpg",
    "002.jpg",
    "003.jpg"
]

current_index = 0
```

다음 이미지:

```python
def next_image():
    global current_index

    if current_index < len(images) - 1:
        current_index += 1

    print(images[current_index])
```

이전 이미지:

```python
def prev_image():
    global current_index

    if current_index > 0:
        current_index -= 1

    print(images[current_index])
```

버튼 연결:

```python
prev_btn = tk.Button(
    root,
    text="이전",
    command=prev_image
)

next_btn = tk.Button(
    root,
    text="다음",
    command=next_image
)
```

---

# 6. Frame — 화면을 영역별로 나누기

라벨링 프로그램은 화면을 한 덩어리로 만들면 복잡해집니다.

예를 들어 다음처럼 나눌 수 있습니다.

```text
┌───────────────────────────────┐
│ Toolbar                       │
├───────────────────────┬───────┤
│                       │ Class │
│                       │       │
│        Image          │ 상태   │
│        Canvas         │       │
│                       │ Note  │
│                       │       │
├───────────────────────┴───────┤
│ Status                        │
└───────────────────────────────┘
```

코드:

```python
top_frame = tk.Frame(root)
top_frame.pack(side="top", fill="x")

main_frame = tk.Frame(root)
main_frame.pack(fill="both", expand=True)

left_frame = tk.Frame(main_frame)
left_frame.pack(side="left", fill="both", expand=True)

right_frame = tk.Frame(main_frame, width=250)
right_frame.pack(side="right", fill="y")
```

---

# 7. pack / grid / place 차이

Tkinter에서는 Widget을 생성만 하면 화면에 나타나지 않습니다.

화면 배치가 필요합니다.

대표적으로 세 가지가 있습니다.

```text
pack()
grid()
place()
```

## pack()

간단한 배치에 좋습니다.

```python
button.pack()
```

좌우 배치:

```python
prev_btn.pack(side="left")
next_btn.pack(side="left")
save_btn.pack(side="left")
```

## grid()

표처럼 배치할 때 좋습니다.

```python
tk.Label(root, text="Class").grid(row=0, column=0)

class_box.grid(row=0, column=1)
```

## 중요한 규칙

**같은 부모 Frame 안에서는 `pack()`과 `grid()`를 섞지 않는 것이 안전합니다.**

잘못된 예:

```python
label = tk.Label(root)
label.pack()

button = tk.Button(root)
button.grid(row=0, column=0)
```

이런 구조는 문제를 만들 수 있습니다.

Frame을 나누고 각각 다른 방식을 쓰는 것은 괜찮습니다.

---

# 8. Canvas — 이번 프로젝트의 핵심

Canvas는 이번 라벨링 프로그램에서 가장 중요합니다.

Canvas에서는

```text
이미지
사각형
선
텍스트
```

등을 자유롭게 그릴 수 있습니다.

생성:

```python
canvas = tk.Canvas(
    root,
    width=900,
    height=700,
    bg="black"
)

canvas.pack()
```

---

# 9. Canvas에 사각형 그리기

BBox 기본입니다.

```python
bbox_id = canvas.create_rectangle(
    100,
    100,
    300,
    250,
    outline="red",
    width=2
)
```

좌표 의미:

```text
(x1, y1)
   ┌──────────────┐
   │              │
   │              │
   └──────────────┘
             (x2, y2)
```

즉:

```python
canvas.create_rectangle(
    x1,
    y1,
    x2,
    y2
)
```

---

# 10. Canvas에 Class 이름 표시

BBox만 그리면 어떤 Class인지 알 수 없습니다.

```python
canvas.create_text(
    100,
    90,
    text="Class 2",
    anchor="sw"
)
```

예:

```python
canvas.create_rectangle(
    100,
    100,
    300,
    250,
    outline="red",
    width=2
)

canvas.create_text(
    100,
    100,
    text="2: 나뭇가지",
    anchor="sw"
)
```

---

# 11. Canvas에 있는 객체 삭제

Canvas에서 생성한 객체에는 ID가 반환됩니다.

```python
bbox_id = canvas.create_rectangle(
    100, 100, 300, 250
)
```

삭제:

```python
canvas.delete(bbox_id)
```

Canvas 전체 삭제:

```python
canvas.delete("all")
```

주의하십시오.

이미지를 바꿀 때

```python
canvas.delete("all")
```

등으로 이전 이미지의 BBox를 지우지 않으면,

> 다음 이미지에 이전 이미지의 BBox가 남는 문제

가 발생할 수 있습니다.

---

# 12. 마우스 클릭 이벤트

Tkinter에서는 `bind()`를 사용합니다.

```python
canvas.bind(
    "<Button-1>",
    on_click
)
```

함수:

```python
def on_click(event):
    print(event.x, event.y)
```

사용자가 클릭하면

```text
event.x
event.y
```

에 Canvas 좌표가 들어옵니다.

---

# 13. 마우스로 BBox 만들기

BBox는 보통 다음 이벤트를 사용합니다.

```text
<ButtonPress-1>
마우스 왼쪽 버튼 누름

<B1-Motion>
누른 상태로 이동

<ButtonRelease-1>
마우스 버튼 놓음
```

기본 구조:

```python
start_x = 0
start_y = 0
current_rect = None
```

마우스 누르기:

```python
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
```

드래그:

```python
def on_mouse_move(event):
    if current_rect is not None:
        canvas.coords(
            current_rect,
            start_x,
            start_y,
            event.x,
            event.y
        )
```

마우스 놓기:

```python
def on_mouse_up(event):
    global current_rect

    end_x = event.x
    end_y = event.y

    print(
        start_x,
        start_y,
        end_x,
        end_y
    )
```

연결:

```python
canvas.bind("<ButtonPress-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_move)
canvas.bind("<ButtonRelease-1>", on_mouse_up)
```

이것이 **BBox 작성의 가장 기본적인 원리**입니다.

---

# 14. BBox 데이터는 Canvas ID만 저장하면 안 됩니다

중요합니다.

예를 들어 이렇게만 관리하면 안 됩니다.

```python
bbox_ids = [
    1,
    2,
    3
]
```

Canvas ID는 화면 객체 번호일 뿐입니다.

실제 데이터는 별도로 보관해야 합니다.

예:

```python
bboxes = [
    {
        "class_id": 2,
        "x1": 100,
        "y1": 120,
        "x2": 300,
        "y2": 280
    }
]
```

더 좋은 구조:

```python
bboxes = []
```

새 BBox:

```python
bbox = {
    "class_id": 2,
    "x1": x1,
    "y1": y1,
    "x2": x2,
    "y2": y2
}

bboxes.append(bbox)
```

---

# 15. 가장 중요한 원칙 — 좌표는 원본 이미지 기준

프로젝트에서 가장 중요하게 지켜야 할 원칙입니다.

> **정답 BBox 상태는 항상 원본 이미지 기준으로 관리합니다.**

예를 들어 원본 이미지가

```text
4848 × 2704
```

인데 화면에는

```text
900 × 500
```

으로 줄여서 보여줄 수 있습니다.

그러면 Canvas 좌표를 그대로 YOLO TXT에 저장하면 안 됩니다.

구조는:

```text
Canvas
   ↓
화면 좌표
   ↓
원본 좌표
   ↓
YOLO 좌표
```

입니다.

---

# 16. 화면 좌표 → 원본 이미지 좌표

예를 들어

```python
scale = 0.25
```

라면

```python
original_x = canvas_x / scale
original_y = canvas_y / scale
```

입니다.

예:

```python
canvas_x = 200
scale = 0.25

original_x = 200 / 0.25
```

결과:

```text
800 pixel
```

---

# 17. 원본 Pixel 좌표 → YOLO 좌표

YOLO는 다음 형식입니다.

```text
class_id x_center y_center width height
```

모든 좌표는 `0~1` 사이 정규화 값입니다.

BBox:

```text
x1, y1
x2, y2
```

라고 하면:

```python
bbox_width = x2 - x1
bbox_height = y2 - y1

center_x = (x1 + x2) / 2
center_y = (y1 + y2) / 2
```

YOLO:

```python
yolo_x = center_x / image_width
yolo_y = center_y / image_height

yolo_w = bbox_width / image_width
yolo_h = bbox_height / image_height
```

저장:

```python
line = (
    f"{class_id} "
    f"{yolo_x:.6f} "
    f"{yolo_y:.6f} "
    f"{yolo_w:.6f} "
    f"{yolo_h:.6f}\n"
)
```

---

# 18. YOLO 좌표 → Pixel 좌표

기존 TXT를 화면에 그릴 때 반대 변환이 필요합니다.

YOLO:

```text
class_id x_center y_center width height
```

Pixel 변환:

```python
center_x = yolo_x * image_width
center_y = yolo_y * image_height

bbox_w = yolo_w * image_width
bbox_h = yolo_h * image_height
```

그리고:

```python
x1 = center_x - bbox_w / 2
y1 = center_y - bbox_h / 2

x2 = center_x + bbox_w / 2
y2 = center_y + bbox_h / 2
```

---

# 19. 파일 열기 — 이미지 선택

Tkinter의 `filedialog`를 사용합니다.

```python
from tkinter import filedialog
```

파일 선택:

```python
file_path = filedialog.askopenfilename(
    title="이미지 선택",
    filetypes=[
        ("Image Files", "*.jpg *.jpeg *.png")
    ]
)
```

폴더 선택:

```python
folder_path = filedialog.askdirectory(
    title="이미지 폴더 선택"
)
```

---

# 20. 폴더의 JPG 목록 가져오기

```python
from pathlib import Path

folder = Path(folder_path)

images = sorted(
    list(folder.glob("*.jpg"))
)
```

여러 확장자:

```python
images = []

for ext in ["*.jpg", "*.jpeg", "*.png"]:
    images.extend(folder.glob(ext))

images = sorted(images)
```

---

# 21. 같은 이름의 TXT 찾기

예:

```text
ABC001.jpg
ABC001.txt
```

Path를 사용하면 편합니다.

```python
image_path = Path("ABC001.jpg")

label_path = image_path.with_suffix(".txt")
```

폴더가 다른 경우:

```python
image_path = Path(
    "images/train/ABC001.jpg"
)

label_dir = Path(
    "labels/train"
)

label_path = (
    label_dir /
    image_path.with_suffix(".txt").name
)
```

---

# 22. TXT 파일 읽기

```python
with open(
    label_path,
    "r",
    encoding="utf-8"
) as f:

    lines = f.readlines()
```

각 줄:

```python
for line in lines:
    parts = line.strip().split()

    print(parts)
```

예:

```text
['2', '0.34', '0.47', '0.04', '0.25']
```

변환:

```python
class_id = int(parts[0])

x_center = float(parts[1])
y_center = float(parts[2])
width = float(parts[3])
height = float(parts[4])
```

---

# 23. Empty TXT 처리

이번 데이터에서 매우 중요합니다.

```python
if label_path.exists():

    text = label_path.read_text(
        encoding="utf-8"
    ).strip()

    if text == "":
        print("Empty label")
```

하지만

```text
Empty TXT = 오류
```

가 아닙니다.

정상 김치 이미지라면 정상 Negative Sample일 수 있습니다.

---

# 24. Combobox — Class 선택

Tkinter 기본 모듈 외에 `ttk`를 사용합니다.

```python
from tkinter import ttk
```

Class 목록:

```python
classes = [
    "0 - 나뭇잎·종이류",
    "1 - 플라스틱·돌·금속",
    "2 - 나뭇가지류",
    "3 - 벌레류",
    "4 - 고무장갑",
    "5 - 병해·갈변",
    "6 - 파·고추"
]
```

Combobox:

```python
class_combo = ttk.Combobox(
    root,
    values=classes,
    state="readonly"
)

class_combo.current(0)

class_combo.pack()
```

현재 선택 값:

```python
selected = class_combo.get()
```

Class ID:

```python
class_id = int(
    selected.split(" ")[0]
)
```

---

# 25. StringVar — 화면 값과 변수 연결

Tkinter에는 특별한 상태 변수가 있습니다.

```python
status_var = tk.StringVar()
```

값 지정:

```python
status_var.set("PASS")
```

읽기:

```python
print(status_var.get())
```

Label과 연결:

```python
status_label = tk.Label(
    root,
    textvariable=status_var
)
```

그러면

```python
status_var.set("EDITED")
```

만 해도 화면이 자동으로 바뀝니다.

---

# 26. Entry — Note 입력

```python
note_entry = tk.Entry(
    root,
    width=40
)

note_entry.pack()
```

내용 읽기:

```python
note = note_entry.get()
```

내용 삭제:

```python
note_entry.delete(
    0,
    tk.END
)
```

값 넣기:

```python
note_entry.insert(
    0,
    "BBox 수정 필요"
)
```

---

# 27. messagebox — 경고창

미저장 상태에서 다음 이미지로 넘어가면 안 됩니다.

```python
from tkinter import messagebox
```

경고:

```python
messagebox.showwarning(
    "경고",
    "저장하지 않은 변경사항이 있습니다."
)
```

확인:

```python
answer = messagebox.askyesno(
    "확인",
    "저장하지 않고 이동하시겠습니까?"
)
```

사용:

```python
if answer:
    next_image()
```

---

# 28. 수정 여부 Dirty Flag

프로젝트에서 매우 중요합니다.

```python
is_dirty = False
```

BBox 수정:

```python
is_dirty = True
```

저장 성공:

```python
is_dirty = False
```

다음 이미지:

```python
def next_image():

    if is_dirty:
        messagebox.showwarning(
            "미저장",
            "먼저 저장하세요."
        )
        return

    # 다음 이미지
```

이 방식으로 작업 유실을 막습니다.

---

# 29. 이미지 읽기 — Pillow 사용 권장

Tkinter만으로 JPG 처리가 불편하므로 일반적으로 Pillow를 함께 사용합니다.

```python
from PIL import Image, ImageTk
```

이미지 열기:

```python
image = Image.open(
    "sample.jpg"
)
```

크기:

```python
width, height = image.size
```

---

# 30. 이미지 Resize

4K 이미지를 그대로 Canvas에 넣으면 화면보다 큽니다.

```python
display_image = image.copy()

display_image.thumbnail(
    (900, 700)
)
```

원본은 유지해야 합니다.

```python
original_image = Image.open(path)

display_image = original_image.copy()

display_image.thumbnail(
    (900, 700)
)
```

---

# 31. PIL Image → Tkinter Image

```python
tk_image = ImageTk.PhotoImage(
    display_image
)
```

Canvas 표시:

```python
canvas.create_image(
    0,
    0,
    anchor="nw",
    image=tk_image
)
```

---

# 32. 매우 자주 발생하는 문제 — 이미지가 안 보임

이 코드는 문제가 생길 수 있습니다.

```python
tk_image = ImageTk.PhotoImage(image)

canvas.create_image(
    0,
    0,
    image=tk_image
)
```

함수가 끝나면 Python이 `tk_image`를 제거해서 화면에서 이미지가 사라질 수 있습니다.

따라서 참조를 유지해야 합니다.

```python
canvas.image = tk_image
```

예:

```python
tk_image = ImageTk.PhotoImage(
    display_image
)

canvas.create_image(
    0,
    0,
    anchor="nw",
    image=tk_image
)

canvas.image = tk_image
```

---

# 33. 이미지 표시 함수는 하나로 묶는 것이 좋습니다

예:

```python
def show_image():

    global tk_image

    canvas.delete("all")

    image_path = images[current_index]

    image = Image.open(image_path)

    display = image.copy()
    display.thumbnail((900, 700))

    tk_image = ImageTk.PhotoImage(
        display
    )

    canvas.create_image(
        0,
        0,
        anchor="nw",
        image=tk_image
    )
```

그리고

```python
next_image()
prev_image()
```

에서 모두 이 함수를 호출합니다.

---

# 34. 프로그램 상태를 여러 global 변수로 흩뜨리지 않는 것이 좋습니다

초기 실습에서는 다음처럼 해도 됩니다.

```python
current_index = 0
bboxes = []
is_dirty = False
```

하지만 프로젝트가 커지면 Class 구조가 훨씬 안전합니다.

예:

```python
class LabelingApp:

    def __init__(self, root):

        self.root = root

        self.images = []
        self.current_index = 0

        self.bboxes = []

        self.is_dirty = False
```

이후:

```python
def next_image(self):
    ...
```

처럼 작성합니다.

---

# 35. Tkinter Class 구조 기본

학생들이 Class가 너무 어렵다면 처음부터 반드시 이렇게 만들 필요는 없습니다.

하지만 최종 프로그램에는 권장합니다.

```python
import tkinter as tk


class LabelingApp:

    def __init__(self, root):

        self.root = root

        self.root.title(
            "YOLO Labeling Tool"
        )

        self.canvas = tk.Canvas(
            root,
            width=900,
            height=700,
            bg="black"
        )

        self.canvas.pack()


root = tk.Tk()

app = LabelingApp(root)

root.mainloop()
```

---

# 36. 프로젝트에서 권장하는 상태 변수

최소 다음 정도는 관리하는 것이 좋습니다.

```python
self.images = []

self.current_index = 0

self.current_image_path = None

self.original_image = None

self.display_image = None

self.scale = 1.0

self.offset_x = 0
self.offset_y = 0

self.bboxes = []

self.selected_bbox = None

self.current_class = 0

self.is_dirty = False

self.status = "PENDING"
```

---

# 37. BBox 하나의 권장 구조

```python
bbox = {
    "class_id": 2,

    "x1": 100,
    "y1": 120,

    "x2": 300,
    "y2": 280,

    "canvas_id": None
}
```

여기서 중요한 것은

```text
x1, y1, x2, y2
```

는 **원본 이미지 기준**으로 보관하는 것입니다.

---

# 38. 화면 다시 그리기 방식

GUI 프로그램에서는 데이터와 화면을 분리하는 것이 좋습니다.

```text
self.bboxes
= 실제 정답 상태

Canvas
= 화면 표시
```

따라서 화면을 다시 그릴 때:

```python
def redraw():

    canvas.delete("all")

    draw_image()

    for bbox in bboxes:
        draw_bbox(bbox)
```

이런 구조가 안전합니다.

---

# 39. 왜 이렇게 해야 하나?

Zoom이 바뀌어도:

```text
원본 BBox
↓
현재 scale 적용
↓
다시 화면에 그림
```

하면 됩니다.

반대로 Canvas 객체 자체를 정답 데이터로 사용하면

```text
Zoom
Pan
Resize
```

가 들어가는 순간 좌표가 꼬이기 쉽습니다.

---

# 40. Zoom 기본 원리

예:

```python
zoom = 1.0
```

확대:

```python
zoom *= 1.2
```

축소:

```python
zoom /= 1.2
```

화면 위치 계산:

```python
screen_x = original_x * zoom
screen_y = original_y * zoom
```

원본 복원:

```python
original_x = screen_x / zoom
original_y = screen_y / zoom
```

---

# 41. Pan까지 들어가면

화면 좌표:

```python
screen_x = (
    original_x * zoom
    + offset_x
)

screen_y = (
    original_y * zoom
    + offset_y
)
```

반대로:

```python
original_x = (
    screen_x - offset_x
) / zoom

original_y = (
    screen_y - offset_y
) / zoom
```

이 수식이 **교과 7에서 상당히 중요한 부분**입니다.

---

# 42. 저장 함수 기본

```python
def save_labels():

    with open(
        output_path,
        "w",
        encoding="utf-8"
    ) as f:

        for bbox in bboxes:

            line = convert_to_yolo(
                bbox
            )

            f.write(line)
```

저장 후:

```python
is_dirty = False
```

---

# 43. RAW에는 저장하지 않습니다

프로젝트 구조상

```text
RAW
→ 읽기 전용 원본

WORK
→ 수정본

FINAL
→ QA 완료본
```

이어야 합니다.

즉 다음처럼 작성하면 위험합니다.

```python
save_path = original_label_path
```

대신

```python
save_path = work_label_path
```

로 저장해야 합니다.

---

# 44. 키보드 단축키

선택 기능이지만 매우 편합니다.

```python
root.bind(
    "<Right>",
    lambda event: next_image()
)

root.bind(
    "<Left>",
    lambda event: prev_image()
)
```

저장:

```python
root.bind(
    "<Control-s>",
    lambda event: save_labels()
)
```

---

# 45. 프로그램 종료 이벤트

창의 X 버튼을 눌렀을 때도 미저장 여부를 확인해야 합니다.

```python
def on_close():

    if is_dirty:

        answer = messagebox.askyesno(
            "종료",
            "저장하지 않은 변경사항이 있습니다. 종료할까요?"
        )

        if not answer:
            return

    root.destroy()
```

연결:

```python
root.protocol(
    "WM_DELETE_WINDOW",
    on_close
)
```

---

# 46. try / except — GUI 프로그램에서는 특히 중요

파일이 없거나 TXT가 깨져도 프로그램 전체가 바로 종료되면 안 됩니다.

```python
try:

    image = Image.open(
        image_path
    )

except Exception as e:

    messagebox.showerror(
        "오류",
        str(e)
    )
```

TXT:

```python
try:

    labels = load_label(
        label_path
    )

except Exception as e:

    print(
        "Label Load Error:",
        e
    )
```

---

# 47. 함수는 기능별로 분리합니다

한 함수에 전부 넣지 않습니다.

좋은 구조:

```python
open_folder()

load_image()

load_yolo_label()

yolo_to_pixel()

draw_image()

draw_bboxes()

create_bbox()

delete_bbox()

change_class()

save_yolo_label()

validate_label()

next_image()

prev_image()
```

이렇게 나누면 팀 프로젝트에서 역할을 나누기도 쉽습니다.

---

# 48. 이번 프로젝트에서 학생들이 최소한 찾아볼 수 있어야 하는 Tkinter 이벤트

| 이벤트 | 의미 |
|---|---|
| `<Button-1>` | 왼쪽 클릭 |
| `<ButtonPress-1>` | 왼쪽 버튼 누름 |
| `<ButtonRelease-1>` | 왼쪽 버튼 놓음 |
| `<B1-Motion>` | 왼쪽 버튼 누른 상태로 이동 |
| `<Motion>` | 마우스 이동 |
| `<MouseWheel>` | 휠 |
| `<Key>` | 키보드 |
| `<Left>` | 왼쪽 방향키 |
| `<Right>` | 오른쪽 방향키 |
| `<Control-s>` | Ctrl+S |

---

# 49. 프로젝트 1일차에는 이것만 성공해도 됩니다

내일 첫날에는 Tkinter 전체를 만들 필요가 없습니다.

첫 End-to-End 목표는 다음입니다.

```text
JPG 1장
        ↓
Tkinter Canvas 표시
        ↓
같은 이름 TXT 찾기
        ↓
TXT 읽기
        ↓
YOLO → Pixel
        ↓
BBox 표시
        ↓
BBox 하나 추가/삭제
        ↓
Class 지정
        ↓
WORK에 Save
        ↓
프로그램 종료
        ↓
재실행
        ↓
같은 위치에 BBox
```

이것만 되면 **1일차 성공**으로 보셔도 됩니다.

---

# 50. 2일차부터 붙이는 기능

1일차가 성공한 뒤:

```text
이전 / 다음
        ↓
여러 BBox
        ↓
Class 변경
        ↓
BBox 삭제
        ↓
Zoom
        ↓
Pan
        ↓
미저장 경고
        ↓
PASS / EDITED / REVIEW
        ↓
Validation
```

순서가 좋습니다.

---

# 51. 학생용 초간단 Tkinter 치트시트

모르면 아래 표부터 찾아보면 됩니다.

| 하고 싶은 것 | 문법 |
|---|---|
| 창 만들기 | `root = tk.Tk()` |
| 제목 | `root.title("제목")` |
| 실행 | `root.mainloop()` |
| 글자 | `tk.Label(...)` |
| 버튼 | `tk.Button(...)` |
| 버튼 함수 | `command=함수명` |
| 화면 영역 | `tk.Frame(...)` |
| 이미지/BBox | `tk.Canvas(...)` |
| 사각형 | `canvas.create_rectangle(...)` |
| 글자 그리기 | `canvas.create_text(...)` |
| 객체 삭제 | `canvas.delete(id)` |
| 전체 삭제 | `canvas.delete("all")` |
| 클릭 이벤트 | `canvas.bind("<Button-1>", func)` |
| 드래그 | `"<B1-Motion>"` |
| 마우스 놓기 | `"<ButtonRelease-1>"` |
| Class 선택 | `ttk.Combobox(...)` |
| 입력 | `tk.Entry(...)` |
| 폴더 선택 | `filedialog.askdirectory()` |
| 파일 선택 | `filedialog.askopenfilename()` |
| 경고 | `messagebox.showwarning()` |
| 확인창 | `messagebox.askyesno()` |
| 값 연결 | `tk.StringVar()` |
| 좌우 배치 | `.pack(side="left")` |
| 표 배치 | `.grid(row=, column=)` |
| 종료 감지 | `root.protocol("WM_DELETE_WINDOW", func)` |

---

# 52. 이번 프로젝트에서 가장 중요한 10가지

학생들에게는 이 10개만 반드시 기억하게 하셔도 됩니다.

```text
① Tk()
→ 프로그램 창

② Frame
→ 화면 영역 구분

③ Canvas
→ 이미지 + BBox

④ Button
→ 이전 / 다음 / 저장

⑤ Combobox
→ Class

⑥ bind()
→ 마우스 이벤트

⑦ Canvas 좌표와 원본 좌표를 구분

⑧ BBox 상태는 원본 이미지 기준으로 저장

⑨ RAW에 저장하지 않고 WORK에 저장

⑩ Save → Reload 했을 때
   같은 위치에 BBox가 나와야 함
```

마지막 10번이 특히 중요합니다.

```text
기존 YOLO
→ Pixel
→ 화면
→ 저장
→ 재실행
→ 같은 위치 BBox
```

이 Round-trip이 실패하면 본 라벨링을 진행하면 안 됩니다.

---

# 학생들에게 이렇게 안내하면 좋습니다

> Tkinter 문법을 전부 외울 필요는 없습니다.  
> 필요한 기능이 생기면 이 가이드에서 `Button`, `Canvas`, `bind`, `Combobox`, `messagebox`처럼 기능 이름을 찾아 사용하면 됩니다.
>
> 이번 프로젝트에서 중요한 것은 Tkinter 문법을 암기하는 것이 아니라 **이미지 → BBox → Class → 좌표 → 저장 → 복원이라는 데이터 흐름을 정확하게 만드는 것**입니다.

그리고 첫날이라면 **Tkinter 설명에 2~3시간을 쓰기보다는 위의 1~18번 정도를 짧게 설명하고 바로 `이미지 1장 + 기존 BBox 표시`를 만들어 보게 하는 방식**이 가장 적절합니다.

나머지 문법은 학생들이 프로젝트 중 막힐 때 찾아보는 참조자료로 사용하면 됩니다.
