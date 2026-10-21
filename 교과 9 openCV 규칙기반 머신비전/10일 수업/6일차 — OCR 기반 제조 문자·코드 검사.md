## 오늘의 목표

6일차에서는 검사 대상이 제품의 외형에서 제품에 인쇄된 문자 정보로 확장됩니다.

![[Pasted image 20260901194603.png]]

```text
1~5일차

제품 이미지
        ↓
모양 / 크기 / 위치 / 특징 측정
        ↓
숫자 기준과 비교
        ↓
OK / NG


6일차

제품 이미지
        ↓
인쇄된 SN 문자 읽기
        ↓
문자열 형식과 Expected SN 비교
        ↓
OK / NG
```

오늘 본 실습에서는 제품에 인쇄된 다음과 같은 13자리 SN(Serial Number)을 검사합니다.

```
0637145699424
```

오늘의 핵심 질문은 다음과 같습니다.

> 제품에 인쇄된 SN을 OCR로 정확하게 읽고, 읽은 값이 올바른 13자리 형식인지 확인한 뒤 Expected SN과 비교하여 OK / NG를 판정할 수 있을까요?

또한 OCR이 잘못 읽은 경우와 제품에 실제로 잘못된 값이 인쇄된 경우를 구분해 봅니다.

오늘 수업이 끝나면 다음 내용을 이해하고 실습할 수 있으면 됩니다.

```
- 제품 이미지에서 필요한 SN 문자 영역만 ROI로 선택할 수 있다.
- Gray / CLAHE / Otsu 등의 전처리가 OCR 결과에 어떤 영향을 주는지 비교할 수 있다.
- Tesseract와 pytesseract의 역할 차이를 이해한다.
- PSM과 Whitelist를 왜 사용하는지 설명할 수 있다.
- Raw OCR과 정리된 문자열을 구분할 수 있다.
- Regex를 이용하여 숫자 13자리 형식을 검사할 수 있다.
- OCR 결과와 Expected SN을 비교하여 OK / NG를 판정할 수 있다.
- OCR 정확도와 제품 검사 정확도가 서로 다른 의미라는 것을 이해한다.
- 여러 제품 이미지를 Batch로 검사하고 결과를 CSV로 저장할 수 있다.
- Low Contrast / Blur / Tilt / Glare에서 OCR이 어떻게 달라지는지 확인할 수 있다.
- 새로운 제품 이미지의 Ch.-B. 코드에도 같은 OCR 흐름을 적용할 수 있다.
```

6일차에서 가장 중요한 흐름은 다음입니다.

![[Pasted image 20260901195036.png]]

```
제품 이미지
        ↓
문자 ROI
        ↓
OCR
        ↓
문자열 정리
        ↓
형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG
```

> 6일차의 목표는 문자를 한 번 읽어보는 것이 아니라, OCR 결과를 실제 제조 검사 판정까지 연결하는 것입니다.

---

# 1. 6일차의 전체 흐름

6일차는 2일차에서 배운 영상 전처리를 처음부터 다시 배우는 수업이 아닙니다.

이미 배운 ROI와 전처리를 OCR에 다시 사용하고, 여기에 문자 인식과 문자열 검사를 추가합니다.

![[Pasted image 20260901195336.png]]

```
PART1
실습 환경 + OCR 데이터 준비
7~10

        ↓

PART2
SN ROI + 전처리 후보 만들기
11~17

        ↓

PART3
전처리별 Tesseract OCR 비교
18~24

        ↓

PART4
문자열 정리 + 13자리 형식 검사
+ Expected SN 비교 + Rule Freeze
25~31

        ↓

PART5
제품 이미지 6장 Batch 검사
+ OCR / 제품 판정 정확도 평가
32~37

        ↓

PART6
환경 변화 테스트
Low Contrast / Blur / Tilt / Glare
38~45

        ↓

PART7
실패 결과 해석과 기록
46~50

        ↓

PART8
Mini Challenge
새로운 이미지의 Ch.-B. 검사
51~56

        ↓

PART9
산출물 확인 + README + Git + 마무리
57~62
```

---

## 1.1 PART1 — OCR 환경과 데이터를 준비합니다

먼저 OCR을 실행할 수 있는 환경과 실습 데이터를 준비합니다.

```
Tesseract Engine
+
pytesseract
+
본 실습 이미지 6장
+
Mini Challenge 이미지 4장
+
Ground Truth
```

본 실습 이미지 6장에서는 **13자리 SN**을 검사합니다.

---

## 1.2 PART2 — SN ROI와 전처리 후보를 만듭니다

제품 전체를 OCR하지 않고 각 이미지에서:

```
SN: 0637145699424
    └─────────────┘
       이 숫자 부분
```

만 ROI로 선택합니다.

그다음 같은 SN ROI를:

```
Original
Gray
CLAHE
Otsu
CLAHE + Otsu
```

로 처리하여 OCR 입력 후보를 만듭니다.

이 단계에서는 눈으로 가장 예쁜 이미지를 고르는 것이 목적이 아닙니다.

```
전처리 후보 생성
        ↓
실제 OCR 실행
        ↓
어떤 것이 정확하게 읽히는지 판단
```

합니다.

---

## 1.3 PART3 — 실제 OCR 결과를 비교합니다

대표 SN:

```
0637145699424
```

를 다섯 가지 전처리 상태에서 실제 Tesseract로 읽어 봅니다.

실제 실습에서는 다음과 같은 결과가 나왔습니다.

```
Original
→ 0637145699424
→ 성공

Gray
→ 0637145699424
→ 성공

CLAHE
→ 0637145699424
→ 성공

Otsu
→ 0637245697424
→ 실패

CLAHE + Otsu
→ 0637245670424
→ 실패
```

따라서:

> 사람 눈에 선명한 전처리가 반드시 OCR에도 좋은 것은 아니다.

라는 것을 실제 결과로 확인합니다.

이번 데이터에서는 가장 단순한:

```
original
```

을 최종 전처리 후보로 선택했습니다.

---

## 1.4 PART4 — OCR 결과를 제품 검사 Rule로 연결합니다

Tesseract가 문자를 읽었다고 바로 제품을 `OK`라고 하지 않습니다.

다음 순서로 검사합니다.

![[Pasted image 20260901195635.png]]

```
Raw OCR

'0637145699424\n'

        ↓

문자열 정리

0637145699424

        ↓

13자리 숫자 형식인가?

^[0-9]{13}$

        ↓

Expected SN과 같은가?

        ↓

OK / NG
```

이번 SN 실습에서는 OCR 결과의 숫자를 프로그램이 임의로 다른 숫자로 수정하지 않습니다.

```
OCR이 읽은 값
→ 필요한 공백과 줄바꿈만 정리

숫자 자체
→ 자동으로 고치지 않음
```

왜냐하면 OCR이 잘못 읽은 것인지, 실제 제품에 잘못된 SN이 인쇄된 것인지 구분해야 하기 때문입니다.

---

## 1.5 PART5 — 6장의 제품을 Batch로 검사합니다

대표 이미지 한 장에서 결정한 OCR 설정을 6장의 제품 이미지 전체에 적용합니다.

중요한 점은 ROI 좌표는 이미지마다 다르지만 OCR Rule은 동일하다는 것입니다.

```
제품 이미지 6장
        ↓
각 이미지에 저장된 SN ROI
        ↓
같은 OCR Rule 적용

Preprocess = original
PSM = 7
Whitelist = 숫자
Regex = 숫자 13자리

        ↓
4배 확대
        ↓
Tesseract OCR
        ↓
Expected SN 비교
        ↓
OK / NG
```

실제 결과:

```
OCR Correct
6 / 6

OCR Exact Accuracy
100%

Inspection Correct
6 / 6

Inspection Accuracy
100%
```

이 과정에서 중요한 차이도 확인합니다.

```
OCR Accuracy
→ 실제 인쇄된 SN을 정확하게 읽었는가?

Inspection Accuracy
→ 제품의 OK / NG를 정확하게 판정했는가?
```

두 값은 서로 다른 의미입니다.

---

## 1.6 PART6 — 환경 변화에 얼마나 버티는지 확인합니다

정상 이미지에서는 OCR이 모두 성공했습니다.

그러면 실제 촬영 상태가 조금 나빠졌을 때는 어떻게 될까요?

대표 SN ROI 한 장을 프로그램으로 자동 변형합니다.

```
정상 SN
0637145699424

        ↓

Low Contrast
Blur
Tilt
Glare
```

그리고 **같은 `day06-ocr-v1.0` Rule을 그대로 적용**합니다.

실제 결과:

```
Low Contrast
→ 성공

Blur
→ 실패

Tilt
→ 실패

Tilt + Deskew
→ 실패

Glare
→ 실패
```

따라서 이 파트에서는 새로운 OCR을 만드는 것이 아니라:

> 현재 OCR 검사기가 어떤 영상 조건에 약한지 확인합니다.

---

## 1.7 PART7 — 실패한 이유를 간단히 확인합니다

환경 변화 테스트 결과를 보고:

```
Blur
→ 문자가 흐려져 OCR 결과 없음

Tilt
→ 일부 숫자만 인식

Glare
→ 문자 일부가 가려져 OCR 결과 없음
```

처럼 실패 원인을 연결해 봅니다.

그리고 다음 두 상황을 구분합니다.

```
OCR Failure
→ 정상 문자를 OCR이 잘못 읽음

제품 NG
→ OCR은 정확하지만
   실제 인쇄값이 Expected SN과 다름
```

새로운 알고리즘을 배우는 파트가 아니라 실행 결과를 이해하고 기록하는 단계입니다.

---

## 1.8 PART8 — 새로운 `Ch.-B.` 코드에 다시 적용합니다

Mini Challenge에서는 본 실습에서 사용하지 않은 새로운 제품 이미지 4장을 사용합니다.

이번에는 SN이 아니라:

```
Ch.-B.
```

코드를 검사합니다.

본 실습에서는:

```
SN
→ 숫자만
→ Whitelist = 0123456789
→ Regex = 숫자 13자리
```

였습니다.

Mini Challenge에서는 검사 대상에 맞게:

```
Ch.-B.
→ 영문 + 숫자 + -
→ Whitelist 변경
→ Regex 변경
```

합니다.

즉 새로운 OCR 기술을 배우는 것이 아니라:

```
새로운 이미지
        ↓
새로운 문자 ROI
        ↓
OCR 설정 변경
        ↓
OCR
        ↓
형식 확인
        ↓
실제 값과 비교
```

를 스스로 다시 해봅니다.

---

## 1.9 PART9 — 결과를 정리하고 마무리합니다

마지막에는 오늘 만든:

```
코드
Config
OCR 결과 CSV
환경 변화 테스트 결과
Mini Challenge 결과
README
```

를 확인하고 Local Git에 기록합니다.

---

# 2. 1~5일차와 6일차는 어떻게 연결되나요?

6일차에서도 앞에서 배운 기술을 그대로 활용합니다.

![[Pasted image 20260901201440.png]]

```
1일차
ROI
        ↓
제품에서 필요한 부분만 선택

        ↓

2일차
Gray / CLAHE / Otsu
        ↓
OCR 입력 영상 비교

        ↓

3일차
Feature → Rule
        ↓
측정 결과를 기준과 비교하는 사고방식

        ↓

5일차
Calibration / Rule Freeze / Test
        ↓
한 번 결정한 검사 Rule을
새 이미지에 동일하게 적용

        ↓

6일차
OCR
        ↓
이미지에서 문자열 추출
        ↓
문자열 Rule과 비교
        ↓
OK / NG
```

즉 검사 대상만 달라집니다.

```
앞의 수업

이미지
→ 면적 / 개수 / 위치 / Score 같은 값 측정


6일차

이미지
→ SN 문자열 측정
```

하지만 최종 구조는 같습니다.

```
이미지
        ↓
필요한 정보 추출
        ↓
정상 기준과 비교
        ↓
OK / NG
```

4~5일차의 광천김 로고 검사와 6일차 OCR 검사는 별도의 검사 모듈로 관리합니다.

```
Day05
로고 / 외형 검사
→ day05_logo_rules.json


Day06
문자 검사
→ day06_ocr_rules.json
```

> 6일차는 Day05 코드를 계속 확장하는 수업이 아니라, 같은 머신비전 프로젝트 안에 새로운 OCR 검사 모듈을 추가하는 수업입니다.

---

# 3. 오늘 무엇을 검사할까요?

오늘 본 실습의 검사 대상은 제품 측면에 인쇄된 **SN(Serial Number)**입니다.

실제 이미지에는 다음과 같은 정보가 있습니다.

```
SN:
Ch.-B.:
Verw.bis:
```

이 중 본 실습에서는:

```
SN: 0637145699424
```

의 **13자리 숫자만 검사**합니다.

OCR ROI에는 가능하면 `SN:`을 포함하지 않고 숫자만 넣습니다.

```
SN: 0637145699424
    └─────────────┘
       OCR 대상
```

컴퓨터가 확인할 것은 크게 세 가지입니다.

```
1. OCR이 실제 SN을 정확하게 읽었는가?

2. 읽은 값이 정확히 숫자 13자리인가?

3. 그 값이 Expected SN과 같은가?
```

---

## 3.1 정상 제품의 경우

예:

```
실제 인쇄된 SN
0637145699424

Expected SN
0637145699424

OCR
0637145699424
```

OCR 결과가:

```
숫자 13자리
+
Expected SN과 일치
```

하므로:

```
OK
```

입니다.

```
OCR
0637145699424
        ↓
13자리 숫자
        ↓
Expected SN과 일치
        ↓
OK
```

---

## 3.2 OCR 결과의 형식이 잘못된 경우

예를 들어 OCR이:

```
9728
```

만 읽었다고 가정합니다.

이번 SN 규칙은:

```
^[0-9]{13}$
```

이므로 13자리가 아닙니다.

```
OCR
9728
        ↓
13자리 숫자가 아님
        ↓
NG_FORMAT
```

실제로 Tilt 환경 변화 테스트에서:

```
OCR = 9728
```

이라는 결과가 발생했습니다.

---

## 3.3 OCR은 정확하지만 제품이 NG인 경우

실제 실습 데이터 중 하나를 예로 봅니다.

```
실제 인쇄된 SN
0679256889404

Expected SN
0679256889400
```

OCR이:

```
0679256889404
```

라고 읽었습니다.

즉 OCR은 정확합니다.

```
OCR 결과
=
Actual SN
```

하지만:

```
0679256889404
        ≠
0679256889400
```

이므로 제품 판정은:

```
NG
```

입니다.

```
OCR 성공
        ↓
13자리 형식 정상
        ↓
Expected SN과 불일치
        ↓
NG_EXPECTED_MISMATCH
```

> **OCR이 정확하게 읽었다고 해서 제품이 반드시 OK인 것은 아닙니다.**

이 차이를 확인하는 것이 6일차에서 매우 중요합니다.

---

# 4. 6일차에서 새롭게 사용하는 기술을 먼저 이해합니다

## 4.1 OCR

`OCR`은 **Optical Character Recognition**, 즉 광학 문자 인식입니다.

![[Pasted image 20260901202047.png]]

쉽게 말하면 이미지 안에 있는 문자를 컴퓨터가 사용할 수 있는 문자열로 읽는 기술입니다.

```
SN 이미지

0637145699424

        ↓

OCR

        ↓

"0637145699424"
```

오늘은 OCR 결과를 바로 최종 판정으로 사용하지 않습니다.

```
OCR
→ 무엇이 인쇄되어 있는지 읽음

Rule
→ 그 값이 정상인지 판단
```

역할을 나누어 사용합니다.

---

## 4.2 Tesseract와 `pytesseract`

오늘 실제 OCR Engine은 **Tesseract**를 사용합니다.

![[Pasted image 20260901202333.png]]

```
제품 이미지 준비
        ↓
OpenCV
→ 이미지 읽기
→ 필요한 문자 영역 ROI 자르기
→ Gray / CLAHE / Otsu 같은 전처리
→ 필요하면 확대

        ↓
Tesseract
→ 준비된 문자 이미지를 실제 문자열로 인식

        ↓
pytesseract
→ Python 코드에서 Tesseract를 호출하도록 연결

        ↓
Python에서 OCR 결과 사용
→ 문자열 정리
→ Regex 형식 검사
→ Expected SN 비교
→ OK / NG
```

OpenCV는 OCR이 읽기 좋은 이미지를 준비하고, Tesseract는 그 이미지에서 문자를 읽으며, pytesseract는 Python에서 Tesseract를 사용할 수 있게 연결해 줍니다.

따라서:

```bash
pip install pytesseract
```

만 실행했다고 Tesseract 자체가 설치되는 것은 아닙니다.

```
Tesseract
→ 운영체제에 설치하는 OCR 프로그램

pytesseract
→ Python에서 Tesseract를 호출하는 라이브러리
```

라고 이해하면 됩니다.

---

## 4.3 PSM

`PSM(Page Segmentation Mode)`은 Tesseract에게 입력된 문자가 어떤 형태로 배치되어 있는지 알려주는 설정입니다.

![[Pasted image 20260901202940.png]]

오늘 OCR 대상은:

```
0637145699424
```

처럼 한 줄입니다.

따라서:

```
PSM 7
```

을 사용했습니다.

쉽게 말하면 Tesseract에게:

```
이 이미지는
문서 전체가 아니라

한 줄짜리 문자라고 생각하고
읽어라.
```

라고 알려주는 것입니다.

PSM 번호를 모두 외울 필요는 없습니다.

> OCR 대상의 문자 배치에 맞는 PSM을 선택한다는 개념만 이해하면 됩니다.

---

## 4.4 Whitelist

Whitelist는 OCR이 어떤 문자만 읽도록 허용할지 제한하는 설정입니다.

![[Pasted image 20261020085950.png]]

오늘 검사하는 SN은 숫자만 있습니다.

```
0 1 2 3 4 5 6 7 8 9
```

예를 들어 오늘 SN은 숫자만 있으므로 Python 코드에서 이렇게 지정할 수 있습니다.

```python
whitelist = "0123456789"
```

그리고 Tesseract 옵션에 넣습니다.

```python
config = (
    "--psm 7 "
    "-c tessedit_char_whitelist=0123456789"
)
```

그러면 Tesseract는 가능한 한:

```
0 1 2 3 4 5 6 7 8 9
```

만 후보로 사용합니다.

따라서:

```
Whitelist
→ 0123456789
```

로 설정했습니다.

이렇게 하면 Tesseract가:

```
A
B
S
O
-
/
```

같은 필요 없는 문자까지 후보로 고려하지 않고 숫자에 집중할 수 있습니다.

```
본 실습 SN
→ 숫자만 허용


Mini Challenge Ch.-B.
→ 영문 + 숫자 + -
→ Whitelist 변경
```

> 검사하려는 문자 종류가 달라지면 Whitelist도 달라집니다.

---

## 4.5 Raw OCR과 Cleaned Text

Tesseract가 반환한 문자열에는 줄바꿈이나 공백이 들어갈 수 있습니다.

실제 결과:

```
Raw OCR

'0637145699424\n'
```

여기서:

```
\n
```

은 줄바꿈입니다.

코드에서:

```python
raw_text.strip()
```

을 적용하면:

```
0637145699424
```

만 남습니다.

![[Pasted image 20260901204342.png]]

따라서 두 값을 구분합니다.

```
Raw OCR
'0637145699424\n'

→ Tesseract가 실제로 반환한 원본


Cleaned Text
0637145699424

→ 공백과 줄바꿈을 정리한 값
```

이번 실습에서는 OCR이 읽은 숫자를 **다른 숫자로 자동 교정하지 않습니다.**

```
0637245697424

        ↓

프로그램이 임의로

0637145699424

로 고치지 않음
```

OCR 오류와 실제 인쇄 오류를 구분해야 하기 때문입니다.

---

## 4.6 Regex(레젝스)

`Regex`는 정규식(Regular Expression)입니다.

문자열이 우리가 정한 형식과 맞는지 검사할 때 사용합니다.

오늘 SN 형식은:

```
숫자 13자리
```

입니다.

따라서 다음 정규식을 사용합니다.

```
^[0-9]{13}$
```

뜻은:

```
^
→ 문자열 시작

[0-9]
→ 숫자 하나

{13}
→ 정확히 13개

$
→ 문자열 끝
```

즉:

```
처음부터 끝까지
숫자로만 되어 있고
정확히 13자리인가?
```

를 검사합니다.

예:

```
0637145699424
→ True

063714569942
→ False

9728
→ False

ABC1234567890
→ False
```

정규식 문법을 외우는 것이 목적은 아닙니다.

```
OCR
→ 문자를 읽음

Regex
→ 읽은 문자열의 형식을 검사
```

라는 차이를 이해하면 됩니다.

---

## 4.7 Expected SN 검사

13자리 형식이 맞다고 해서 제품이 반드시 정상인 것은 아닙니다.

예:

```
OCR 결과
0679256889404
```

이 값은 숫자 13자리이므로:

```
Format OK = True
```

입니다.

하지만 생산상 기대하는 SN이:

```
0679256889400
```

이라면 서로 다릅니다.

```
0679256889404
        ≠
0679256889400
```

따라서:

```
Expected Match = False
        ↓
제품 NG
```

가 됩니다.

최종 검사는 다음과 같이 진행됩니다.

```
OCR 결과
        ↓
13자리 숫자인가?
        ↓
YES
        ↓
Expected SN과 같은가?
        ↓
YES → OK

NO → NG
```

숫자가 일치한다고 맞는것이 아니라 번호까지 맞아야 OK가 된다는 뜻입니다.

---

## 4.8 Deskew(디스큐)

Deskew는 **기울어진 문자 이미지를 다시 수평에 가깝게 돌리는 처리**입니다.

![[Pasted image 20260901204857.png]]

이번 환경 변화 테스트에서 정상 SN을:

```
+8°
```

기울여 Tilt 이미지를 만들었습니다.

그리고 비교를 위해:

```
+8° Tilt
        ↓
-8° 회전
        ↓
Deskew
```

도 적용해 보았습니다.

이번 실제 결과에서는:

```
Tilt
→ OCR 실패

Tilt + Deskew
→ 여전히 실패
```

했습니다.

즉 Deskew를 적용하면 항상 OCR이 좋아지는 것은 아닙니다.

> 이번 실습에서는 Deskew 알고리즘을 깊게 배우는 것이 아니라, **기울어진 문자를 보정하는 방법 중 하나가 있다는 것을 경험하는 정도**로 이해합니다.

---

# 5. 앞에서 배운 기술과 오늘 새로 사용하는 기술을 구분합니다

| 구분   | 기술             | 6일차에서 사용한 방법                  |
| ---- | -------------- | ----------------------------- |
| 재사용  | ROI            | 제품 전체에서 13자리 SN 영역만 선택        |
| 재사용  | Gray           | 색상을 제거한 OCR 입력 후보             |
| 재사용  | CLAHE          | 문자와 배경의 대비 개선 후보              |
| 재사용  | Otsu           | 문자와 배경을 이진화한 OCR 입력 후보        |
| 재사용  | Rule / Freeze  | 실험 결과를 `day06-ocr-v1.0`으로 고정  |
| 재사용  | Batch          | 같은 OCR Rule을 제품 이미지 6장에 반복 적용 |
| 새 기술 | Tesseract OCR  | SN 이미지를 문자열로 읽음               |
| 새 기술 | `pytesseract`  | Python에서 Tesseract를 호출        |
| 새 기술 | PSM 7          | SN을 한 줄 문자로 인식                |
| 새 기술 | Whitelist      | 본 실습에서는 숫자 `0~9`만 허용          |
| 새 기술 | Text Cleaning  | OCR 결과의 공백·줄바꿈 정리             |
| 새 기술 | Regex          | OCR 결과가 숫자 13자리인지 검사          |
| 새 기술 | Expected SN 비교 | 읽은 SN이 생산상 기대값과 같은지 검사        |
| 새 기술 | Deskew         | Tilt 이미지의 기울기 보정 비교           |

실제 실습에서 전처리 결과도 확인했습니다.

```
Original
→ OCR 성공

Gray
→ OCR 성공

CLAHE
→ OCR 성공

Otsu
→ OCR 실패

CLAHE + Otsu
→ OCR 실패
```

따라서 이번 데이터에서는:

```
가장 복잡한 전처리
        ≠
가장 좋은 OCR 결과
```

였습니다.

가장 단순한:

```
original
```

을 최종 Rule로 선택했습니다.

그리고 6장의 제품 이미지에 적용한 결과:

```
OCR Accuracy
6 / 6
→ 100%

Inspection Accuracy
6 / 6
→ 100%
```

를 확인했습니다.

> 6일차에서는 새로운 영상처리 기법을 많이 배우는 것이 목적이 아닙니다. 
> 앞에서 배운 ROI와 전처리를 OCR에 연결하고, OCR 결과를 문자열 Rule과 Expected SN으로 검사하여 실제 제조 OK / NG 판정까지 연결하는 것이 핵심입니다.

---

# 6. 6일차 프로젝트 구조와 파일 번호를 정리합니다

1~5일차와 같은 프로젝트를 계속 사용합니다.

```text
~/ai_vision/subject9_machinevision
```

새 프로젝트를 만들지 않습니다.

현재 5일차 본문에서 실제 마지막 실행 파일은 다음입니다.

```text
src/39_day05_mini_challenge.py
```

따라서 6일차 실행 파일은 40번부터 시작합니다.

오늘 추가되는 구조는 다음과 같습니다.

```text
40_day06_check_ocr_env.py
41_day06_select_text_roi.py
42_day06_preprocess_compare.py
43_day06_ocr_compare.py
44_day06_freeze_ocr_rules.py
45_day06_batch_ocr.py
46_day06_evaluate.py
47_day06_make_challenge_variants.py
48_day06_challenge.py
49_day06_mini_challenge.py
```

공통 함수는 여러 실행 파일에서 재사용하므로 번호를 붙이지 않습니다.

```text
src/day06_ocr_utils.py
src/day06_ocr_validation.py
```

| 파일명                                   | 역할                    | 설명                                                                                       |
| ------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------- |
| `40_day06_check_ocr_env.py`           | OCR 개발환경 확인           | WSL·OpenCV·Tesseract 버전, 설치 경로, OCR 언어 데이터 등을 확인하고 환경 점검 결과를 저장합니다.                      |
| `41_day06_select_text_roi.py`         | SN 문자 ROI 선택          | 제품 이미지 6장에서 13자리 SN 숫자 영역을 마우스로 선택하고, 잘라낸 이미지와 ROI 좌표를 저장합니다.                            |
| `42_day06_preprocess_compare.py`      | OCR 전처리 이미지 비교        | 대표 SN 이미지에 Original·Gray·CLAHE·Otsu·CLAHE+Otsu를 적용하여 5가지 전처리 결과를 비교합니다.                  |
| `43_day06_ocr_compare.py`             | 전처리별 OCR 정확도 비교       | 5가지 전처리 이미지를 4배 확대하고 Tesseract로 읽어, 어떤 방법이 13자리 SN을 가장 정확하게 인식하는지 확인합니다.                 |
| `44_day06_freeze_ocr_rules.py`        | OCR 검사 Rule 확정        | 실험 결과를 바탕으로 Original·PSM 7·숫자 Whitelist·13자리 Regex를 확정하고 JSON에 저장합니다.                    |
| `45_day06_batch_ocr.py`               | 제품 6장 일괄 검사           | 확정된 OCR Rule로 6장의 SN을 읽고, 13자리 형식 및 Expected SN과 비교하여 OK / NG를 판정하고 CSV에 저장합니다.          |
| `46_day06_evaluate.py`                | OCR 및 검사 정확도 평가       | Batch 검사 결과를 분석하여 OCR Exact Accuracy와 제품 검사 Accuracy를 계산하고 실패 이미지와 평가 결과를 정리합니다.         |
| `47_day06_make_challenge_variants.py` | 촬영환경 변화 이미지 생성        | 정상 SN 이미지에 Low Contrast·Blur·Tilt·Glare를 적용하여 OCR 검사기의 성능을 시험할 이미지 4장을 만듭니다.             |
| `48_day06_challenge.py`               | 촬영환경 변화에 따른 OCR 성능 검증 | 확정된 OCR Rule을 변형 이미지에 적용하고 Tilt·Deskew도 비교하여 어떤 조건에서 OCR이 실패하는지 확인합니다.                   |
| `49_day06_mini_challenge.py`          | 새로운 문자코드 검사 실습        | 새로운 이미지 4장에서 `Ch.-B.` 코드 영역을 직접 선택하고 영문·숫자·하이픈에 맞게 Whitelist와 Regex를 변경하여 OCR 결과를 검증합니다. |
| `day06_ocr_utils.py`                  | OCR 공통 처리 함수          | ROI 추출, 전처리, Tesseract 설정, OCR 실행 등을 여러 파일에서 재사용하도록 정리합니다.                               |
| `day06_ocr_validation.py`             | 문자열 검사 공통 함수          | OCR 문자열 정리, Regex 형식 검사, Expected SN 비교, OK / NG 판정 기능을 제공합니다.                           |


![[Pasted image 20261020090535.png]]

![[Pasted image 20261020092437.png]]

---

## 6.1 Day06 폴더를 만듭니다

입력 데이터 폴더를 만듭니다.

```bash
mkdir -p samples/day06/product
mkdir -p samples/day06/challenge/low_contrast
mkdir -p samples/day06/challenge/blur
mkdir -p samples/day06/challenge/tilt
mkdir -p samples/day06/challenge/glare
mkdir -p samples/day06/mini_challenge/product
mkdir -p samples/day06/mini_challenge/challenge
```

결과 저장 폴더도 준비합니다.

```bash
mkdir -p outputs/day06/roi
mkdir -p outputs/day06/preprocess
mkdir -p outputs/day06/batch
mkdir -p outputs/day06/challenge
mkdir -p outputs/day06/mini_challenge
```

`src`, `configs`, `reports` 폴더는 앞 수업에서 사용하던 폴더를 그대로 사용합니다.

---

## 6.2 본 실습용 이미지 6장을 넣습니다

6일차 본 실습에서는 제품 측면에 인쇄된 **SN(Serial Number)**을 OCR로 검사합니다.

다음 6장을:

```
samples/day06/product/
```

에 넣습니다.

```bash
explorer.exe .
```

```
0637145699424_side
0639873538899_side
0679256889404_side
0702739449714_side
0716954573504_side
0793387694868_side
```

실제 파일의 `.png`, `.jpg` 등 **원본 확장자는 그대로 유지합니다.**

예:

```
samples/day06/product/
├── 0637145699424_side.png
├── 0639873538899_side.png
├── 0679256889404_side.png
├── 0702739449714_side.png
├── 0716954573504_side.png
└── 0793387694868_side.png
```

파일명 앞의 숫자는 이미지에 표시된 SN 정답값으로 사용할 수 있습니다.

```
0637145699424_side.png
        ↓
SN = 0637145699424
```

---

## 6.3 Mini Challenge용 이미지 4장을 따로 넣습니다

본 실습에서 사용하지 않은 다음 4장은 Mini Challenge에서 사용합니다.

```
0185973818825_side
0386316000696_side
0635619134582_side
0924467469402_side
```

저장 위치:

```
samples/day06/mini_challenge/product/
```

예:

```
samples/day06/mini_challenge/product/
├── 0185973818825_side.png
├── 0386316000696_side.png
├── 0635619134582_side.png
└── 0924467469402_side.png
```

본 실습에서는 `SN`을 함께 검사하고, Mini Challenge에서는 처음 보는 4장을 이용해 다른 문자 영역에 OCR 흐름을 스스로 적용합니다.

---

## 6.4 Challenge 이미지는 지금 준비하지 않습니다

다음 폴더는 처음에는 비어 있어도 됩니다.

```
samples/day06/challenge/
├── low_contrast/
├── blur/
├── tilt/
└── glare/
```

뒤에서 `47_day06_make_challenge_variants.py`를 실행하면 정상 문자 이미지를 이용하여 다음 조건을 자동으로 만듭니다.

```
정상 문자 ROI
        ↓
Low Contrast
Blur
Tilt
Glare
```

따라서 Challenge용 원본 이미지를 별도로 준비할 필요는 없습니다.

---

## 6.5 6일차 시작 시 데이터 구조를 확인합니다

```
samples/day06/
│
├── product/
│   ├── 0637145699424_side.png
│   ├── 0639873538899_side.png
│   ├── 0679256889404_side.png
│   ├── 0702739449714_side.png
│   ├── 0716954573504_side.png
│   └── 0793387694868_side.png
│
├── challenge/
│   ├── low_contrast/
│   ├── blur/
│   ├── tilt/
│   └── glare/
│
└── mini_challenge/
    ├── product/
    │   ├── 0185973818825_side.png
    │   ├── 0386316000696_side.png
    │   ├── 0635619134582_side.png
    │   └── 0924467469402_side.png
    │
    └── challenge/
```

즉 6일차 시작 전에 준비할 것은 다음과 같습니다.

```
본 실습 이미지 6장
+
Mini Challenge 이미지 4장
+
Day06 폴더 구조
```

`day06_ocr_rules.json`, `day06_labels.csv`, 결과 CSV와 보고서 파일은 뒤의 실습을 진행하면서 생성합니다.

---

# PART1 — 실습 환경 + OCR 데이터 준비 (7~10)

---

# 7. 기존 프로젝트를 다시 엽니다

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject9_machinevision
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

현재 Python을 확인합니다.

```bash
which python
```

VS Code를 엽니다.

```bash
code -r .
```

1일차부터 사용한 환경을 그대로 사용합니다.

> 6일차를 위해 새로운 `.venv`를 만들지 않습니다.

---

# 8. Tesseract Engine과 Python Package를 준비합니다

먼저 Tesseract 설치 여부를 확인합니다.

```bash
tesseract --version
```

버전 정보가 출력되면 이미 설치되어 있습니다.

Ubuntu / WSL에서 설치되어 있지 않다면:

```bash
sudo apt update
sudo apt install -y tesseract-ocr tesseract-ocr-eng
```

영문·숫자 LOT 검사만 한다면 `eng` 데이터로 진행할 수 있습니다.

한글 OCR까지 추가 확인하려면 선택적으로:

```bash
sudo apt install -y tesseract-ocr-kor
```

를 설치할 수 있습니다.

설치 언어를 확인합니다.

```bash
tesseract --list-langs
```

```
- eng: 영어 (English)
- kor: 한국어 (Korean)
- osd: 방향 및 스크립트 감지 (Orientation and Script Detection)
```

Python Wrapper를 설치합니다.

```bash
pip install pytesseract
```

`pandas`가 없다면:

```bash
pip install pandas
```

| 항목                           | 역할             | 쉽게 설명                                           |
| ---------------------------- | -------------- | ----------------------------------------------- |
| **Tesseract**                | 실제 OCR Engine  | 이미지 속 문자를 읽어서 텍스트로 변환하는 프로그램                    |
| **`tesseract --list-langs`** | 설치된 OCR 언어 확인  | 현재 Tesseract가 어떤 언어를 읽을 수 있는지 확인하는 명령어          |
| **pytesseract**              | Python Wrapper | Python 코드에서 Tesseract OCR을 사용할 수 있게 연결해주는 라이브러리 |
| **pandas**                   | 데이터 처리 라이브러리   | 여러 이미지의 OCR 결과를 표 형태로 정리하고 CSV로 저장할 때 사용        |

현재 Python 환경을 갱신합니다.

```bash
pip freeze > requirements.txt
```

주의합니다.

```text
requirements.txt
→ Python Package 기록

Tesseract Engine
→ 운영체제 Package
```

이므로 Tesseract 자체는 `requirements.txt`만으로 재설치되지 않습니다.

---

# 9. `40_day06_check_ocr_env.py` — OCR 환경을 확인합니다

실제 제품 이미지를 OCR하기 전에, 현재 개발환경에서 OCR 실습에 필요한 프로그램들이 정상적으로 준비되어 있는지 확인하는 코드입니다.

즉 이 파일은 문자를 읽는 실습이 아니라 OCR 환경 점검 실습입니다.

| 확인 항목             | 확인한 내용                               |
| ----------------- | ------------------------------------ |
| OS                | 현재 WSL2 Linux 환경이 정상적으로 실행되는지        |
| OpenCV            | 이미지 처리에 사용할 OpenCV 버전                |
| Tesseract Path    | Tesseract OCR Engine이 실제로 설치되어 있는 위치 |
| Tesseract Version | 현재 설치된 Tesseract 버전                  |
| OCR Language      | 사용할 수 있는 OCR 언어 데이터                  |
| Report            | 확인 결과가 파일로 정상 저장되는지                  |

파일:

```bash
src/40_day06_check_ocr_env.py
```

작성합니다.

```python
from pathlib import Path
import platform
import shutil
import subprocess

import cv2
import pytesseract


# ========================================
# 프로젝트 경로
# ========================================

PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

REPORT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_ocr_environment.txt"
)


# ========================================
# 기본 환경 확인
# ========================================

lines = [
    f"OS: {platform.platform()}",
    f"OpenCV: {cv2.__version__}",
]


# ========================================
# Tesseract 실행 파일 확인
# ========================================

tesseract_path = shutil.which(
    "tesseract"
)

if tesseract_path is None:

    lines.append(
        "Tesseract: NOT FOUND"
    )

    lines.append(
        "Tesseract Path: NOT FOUND"
    )

else:

    lines.append(
        f"Tesseract Path: {tesseract_path}"
    )

    try:

        version = (
            pytesseract
            .get_tesseract_version()
        )

        lines.append(
            f"Tesseract: {version}"
        )

    except Exception as error:

        lines.append(
            f"Tesseract Version Error: {error}"
        )


    # ====================================
    # 설치된 OCR 언어 확인
    # ====================================

    try:

        result = subprocess.run(
            [
                "tesseract",
                "--list-langs",
            ],
            capture_output=True,
            text=True,
            check=True,
        )

        lines.append(
            "Languages:"
        )

        lines.append(
            result.stdout.strip()
        )

    except Exception as error:

        lines.append(
            f"Language Check Error: {error}"
        )


# ========================================
# Report 저장
# ========================================

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

REPORT_PATH.write_text(
    "\n".join(lines),
    encoding="utf-8",
)


# ========================================
# 결과 출력
# ========================================

print(
    "\n".join(lines)
)

print()

print(
    "Saved:",
    REPORT_PATH
)


if tesseract_path is None:

    print(
        "OCR Environment Check Failed"
    )

else:

    print(
        "OCR Environment OK"
    )
```

실행합니다.

```bash
python src/40_day06_check_ocr_env.py
```

### 의사코드

```text
프로그램 시작

        ↓

필요한 기능을 불러온다

Path
→ 파일과 폴더 경로 처리

platform
→ 현재 운영체제 정보 확인

shutil
→ tesseract 실행 파일 위치 확인

subprocess
→ 터미널 명령 실행

cv2
→ OpenCV 버전 확인

pytesseract
→ Python에서 Tesseract 정보 확인

        ↓

현재 실행 중인 Python 파일의 위치를 기준으로
프로젝트 최상위 폴더를 찾는다

src/40_day06_check_ocr_env.py
        ↓
src의 상위 폴더
        ↓
subject9_machinevision

        ↓

환경 확인 결과를 저장할 파일 경로를 만든다

subject9_machinevision/
└── reports/
    └── day06_ocr_environment.txt

        ↓

결과를 모아둘 lines 목록을 만든다

        ↓

현재 운영체제 정보를 확인하여
lines에 추가한다

        ↓

현재 OpenCV 버전을 확인하여
lines에 추가한다

        ↓

컴퓨터에서
tesseract 실행 파일을 찾는다

shutil.which("tesseract")

        ↓

Tesseract 경로가 없는가?

        ├── YES
        │
        │   ↓
        │
        │   "Tesseract: NOT FOUND"
        │   를 기록한다
        │
        │   ↓
        │
        │   "Tesseract Path: NOT FOUND"
        │   를 기록한다
        │
        └── NO
            │
            ↓

            Tesseract 설치 경로를
            lines에 기록한다

            ↓

            pytesseract를 이용하여
            Tesseract 버전을 확인한다

            ↓

            버전을 정상적으로 확인했으면
            lines에 버전을 기록한다

            ↓

            버전을 확인하는 중 오류가 발생하면
            오류 내용을 lines에 기록한다

            ↓

            터미널에서 다음 명령을 실행한다

            tesseract --list-langs

            ↓

            설치된 OCR 언어 목록을 가져온다

            예:
            eng
            kor
            osd

            ↓

            언어 목록을 lines에 기록한다

            ↓

            언어 확인 중 오류가 발생하면
            오류 내용을 lines에 기록한다

        ↓

reports 폴더가 없으면 생성한다

        ↓

지금까지 lines에 모은 내용을
한 줄씩 연결한다

        ↓

reports/day06_ocr_environment.txt
파일에 저장한다

        ↓

같은 내용을 터미널에도 출력한다

        ↓

저장된 Report 파일의 위치를 출력한다

        ↓

Tesseract가 없으면

OCR Environment Check Failed

를 출력한다

        ↓

Tesseract가 있으면

OCR Environment OK

를 출력한다

        ↓

프로그램 종료
```

출력 결과

```
OS: Linux-6.18.33.2-microsoft-standard-WSL2-x86_64-with-glibc2.39
OpenCV: 5.0.0
Tesseract Path: /usr/bin/tesseract
Tesseract: 5.3.4
Languages:
List of available languages in "/usr/share/tesseract-ocr/5/tessdata/" (3):
eng
kor
osd

Saved: /home/youjung/ai_vision/subject9_machinevision/reports/day06_ocr_environment.txt
OCR Environment OK
```

실행 결과 해석

```
OS: Linux ... WSL2
→ 현재 WSL2 Linux 환경에서 실행 중

OpenCV: 5.0.0
→ OpenCV 5.0.0 사용 가능

Tesseract Path: /usr/bin/tesseract
→ Tesseract가 정상 설치되어 있음

Tesseract: 5.3.4
→ Tesseract OCR Engine 버전은 5.3.4

Languages:
eng
kor
osd
→ 영어, 한글, 문자 방향 감지 데이터 사용 가능

Saved:
reports/day06_ocr_environment.txt
→ 환경 확인 결과가 파일로 저장됨

OCR Environment OK
→ 6일차 OCR 실습 환경 준비 완료
```

---

# 10. OCR 평가를 위한 Ground Truth를 준비합니다

앞에서 본 실습용 이미지 6장을 이미 준비했습니다.

이제 뒤의 Batch 검사에서:

```
OCR이 실제 SN을 정확하게 읽었는가?

제품에 인쇄된 SN이
기대값과 일치하는가?
```

를 확인할 수 있도록 **교육용 정답표(Ground Truth)**를 만듭니다.

저장 위치:

```
configs/day06_labels.csv
```

---

## 10.1 Ground Truth에는 세 가지 정답 정보를 기록합니다

각 이미지마다 다음 정보를 기록합니다.

```
actual_text
→ 이미지에 실제로 인쇄된 SN

expected_text
→ 생산 시 인쇄되어야 한다고 가정한 SN

actual_status
→ 실제 제품의 정답 OK / NG
```

이 세 값을 구분해야 뒤에서 **OCR 정확도와 제품 검사 정확도**를 따로 확인할 수 있습니다.

---

## 10.2 `day06_labels.csv`를 작성합니다

파일:

```
configs/day06_labels.csv
```

예:

```
file,actual_text,expected_text,actual_status
0637145699424_side.png,0637145699424,0637145699424,OK
0639873538899_side.png,0639873538899,0639873538899,OK
0679256889404_side.png,0679256889404,0679256889400,NG
0702739449714_side.png,0702739449714,0702739449714,OK
0716954573504_side.png,0716954573504,0716954573500,NG
0793387694868_side.png,0793387694868,0793387694868,OK
```

> 실제 이미지가 `.jpg`라면 CSV에서도 `.jpg`로 작성합니다.

각 열의 의미는 다음과 같습니다.

|열|의미|
|---|---|
|`file`|검사할 이미지 파일명|
|`actual_text`|이미지에 실제로 인쇄된 SN|
|`expected_text`|제품에 인쇄되어야 한다고 가정한 SN|
|`actual_status`|실제 판정 정답 `OK / NG`|

---

## 10.3 왜 `actual_text`와 `expected_text`를 따로 기록할까요?

예를 들어:

```
실제 인쇄된 SN
0679256889404

기대 SN
0679256889400
```

OCR이:

```
0679256889404
```

라고 읽었다면 OCR 자체는 정확합니다.

하지만 기대값과는 다릅니다.

```
OCR 결과 = actual_text
→ OCR 성공

OCR 결과 ≠ expected_text
→ 제품 NG
```

즉:

```
OCR Accuracy
≠
Product Inspection Accuracy
```

입니다.

이번 실습에서 일부 `expected_text`를 다르게 설정하는 것은 잘못된 SN이 인쇄된 제조 상황을 교육용으로 가정하기 위한 것입니다.

---

## 10.4 Ground Truth는 언제 사용하나요?

지금은 CSV만 준비합니다.

뒤의 Batch 검사에서:

```
제품 이미지
        ↓
OCR
        ↓
OCR 결과
        ↓
day06_labels.csv와 비교
        ↓
OCR 정확도 확인
+
제품 OK / NG 확인
```

에 사용합니다.

> `actual_text`는 실제 제조검사에서 프로그램이 미리 알고 있는 값이 아닙니다.  
> 오늘 실습에서 OCR 성능을 평가하기 위한 교육용 정답 데이터입니다.

---

# PART2 — Experiment 1: 문자 ROI + OCR 전처리 비교 (11~17)

---

# 11. 첫 번째 Experiment의 질문

첫 번째 실험에서는 다음을 확인합니다.

> 제품 전체를 OCR하는 것보다 필요한 SN 영역만 잘라서 OCR하는 것이 왜 유리하며, 어떤 전처리가 숫자 인식에 도움이 될까요?

오늘 본 실습의 검사 대상은 제품에 인쇄된 13자리 SN(Serial Number)입니다.

예:

```
SN: 0637145699424
```

OCR에서는 `SN:`이라는 글자까지 읽지 않고 13자리 숫자 영역만 검사합니다.

```
제품 이미지
        ↓
SN 숫자 영역 선택
        ↓
ROI 저장
        ↓
전처리 후보 생성
        ↓
OCR 비교
```

이번 데이터는 이미지마다 제품 위치가 조금씩 다르기 때문에, 먼저 본 실습용 6장의 SN ROI를 각각 지정하여 저장합니다.

> 이 단계에서는 아직 OCR이나 OK / NG 판정을 하지 않습니다.  
> 이후 실험에서 사용할 문자 영역을 준비하는 단계입니다.

---

# 12. 제품 전체가 아니라 SN ROI를 사용하는 이유

제품 이미지에는 OCR 대상이 아닌 정보도 많이 포함되어 있습니다.

```
SN
Ch.-B.
Verw.bis
```

![[{9203CDAC-FBF5-490F-AF3A-89FBDE72BA53}.png]]


제품 전체를 OCR하면 여러 문자가 한꺼번에 인식될 수 있습니다.

```
전체 제품 이미지
        ↓
OCR
        ↓
여러 문자와 숫자가 섞임
        ↓
필요한 SN을 찾기 어려움
```

따라서 검사할 정보만 제한합니다.

```
전체 제품
        ↓
SN 숫자 영역 ROI
        ↓
0637145699424
        ↓
OCR
```

이것은 1일차에서 배운 ROI 개념을 문자 검사에 다시 적용하는 것입니다.

### 이미지 6장을 ROI 처리 하는 이유

현재 이렇게 이미지를 ROI로 일일히 자르는 이유는 현재 6일차는 Tesseract를 이용한 규칙기반 OCR 실습이고, 이미지마다 위치가 달라서 교육용으로 SN 위치를 직접 잘라 본 것입니다.

대량 이미지에서는 보통 이렇게 합니다.

```
[지금 실습]
이미지 6장
→ 사람이 SN ROI 지정
→ OCR
→ 문자열 검사

[실제 대량 처리]
이미지 수천 장
→ 프로그램이 자동으로 문자/문서 위치 탐지
→ 필요한 영역 자동 Crop
→ OCR
→ 문자열 검사
```

대량 운영 시 모든 이미지에 사람이 ROI를 지정하는 것은 아닙니다.  
지금 6장은 OCR 원리를 분리해서 배우기 위해 수동 ROI를 사용한 것이고, 실제 시스템에서는 고정 위치라면 ROI를 한 번 설정하고, 위치가 계속 변하면 Text Detection/객체검출로 ROI를 자동 탐지하는 방식으로 발전시킵니다.

---

# 13. `41_day06_select_text_roi.py` — SN 문자 ROI를 지정합니다

이 파일에서는 본 실습용 이미지 6장을 차례대로 열고, 각 이미지에서 **SN 숫자 영역을 마우스로 선택**합니다.

선택한 결과는 두 가지로 사용합니다.

```
SN ROI 이미지
→ 뒤의 전처리·OCR 실험에서 사용

ROI 좌표
→ day06_ocr_rules.json에 저장
```

파일:

```
src/41_day06_select_text_roi.py
```

작성합니다.

```python
import json
from pathlib import Path

import cv2


# ========================================
# 프로젝트 경로
# ========================================

PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

PRODUCT_DIR = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
)

CONFIG_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_ocr_rules.json"
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "roi"
)


# ========================================
# 제품 이미지 찾기
# ========================================

IMAGE_PATHS = sorted(
    [
        path
        for path in PRODUCT_DIR.iterdir()
        if path.suffix.lower()
        in {".jpg", ".jpeg", ".png"}
    ]
)


if not IMAGE_PATHS:
    raise FileNotFoundError(
        f"이미지가 없습니다: {PRODUCT_DIR}"
    )


print(
    "찾은 제품 이미지:",
    len(IMAGE_PATHS),
)


OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# ========================================
# 이미지별 ROI 저장
# ========================================

roi_by_file = {}


for image_path in IMAGE_PATHS:

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            image_path
        )


    print()
    print(
        "Image:",
        image_path.name
    )

    print(
        "SN의 13자리 숫자 영역만 선택한 후 "
        "ENTER 또는 SPACE를 누르세요."
    )


    x, y, w, h = cv2.selectROI(
        f"Select SN ROI - {image_path.name}",
        image,
        fromCenter=False,
        showCrosshair=True,
    )

    cv2.destroyAllWindows()


    if w == 0 or h == 0:
        raise RuntimeError(
            f"ROI가 선택되지 않았습니다: "
            f"{image_path.name}"
        )


    x1 = int(x)
    y1 = int(y)
    x2 = int(x + w)
    y2 = int(y + h)


    # ====================================
    # ROI Crop
    # ====================================

    roi = image[
        y1:y2,
        x1:x2,
    ]


    # ====================================
    # ROI 위치 Preview
    # ====================================

    preview = image.copy()

    cv2.rectangle(
        preview,
        (x1, y1),
        (x2, y2),
        (0, 255, 0),
        3,
    )


    # ====================================
    # 결과 저장
    # ====================================

    preview_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_roi_preview.jpg"
    )

    roi_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_sn_roi.jpg"
    )


    cv2.imwrite(
        str(preview_path),
        preview,
    )

    cv2.imwrite(
        str(roi_path),
        roi,
    )


    roi_by_file[
        image_path.name
    ] = [
        x1,
        y1,
        x2,
        y2,
    ]


    print(
        "ROI:",
        roi_by_file[
            image_path.name
        ],
    )


# ========================================
# Day06 OCR Rule 초안
# ========================================

rules = {
    "baseline_version":
        "day06-ocr-draft",

    "reference_file":
        IMAGE_PATHS[0].name,

    "roi_by_file":
        roi_by_file,

    "preprocess_mode":
        "NOT_SELECTED",

    "clahe_clip_limit":
        2.0,

    "clahe_grid":
        [8, 8],

    "ocr_lang":
        "eng",

    "ocr_oem":
        3,

    "ocr_psm":
        7,

    "whitelist":
        "0123456789",

    "expected_pattern":
        r"^[0-9]{13}$",

    "allow_context_correction":
        False,
}


# ========================================
# Config 저장
# ========================================

CONFIG_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

CONFIG_PATH.write_text(
    json.dumps(
        rules,
        ensure_ascii=False,
        indent=2,
    ),
    encoding="utf-8",
)


print()
print(
    "Saved:",
    CONFIG_PATH
)

print(
    "Day06 SN ROI Setup Complete"
)
```

실행합니다.

```
python src/41_day06_select_text_roi.py
```

6장의 이미지가 차례대로 열립니다.

각 이미지에서 다음처럼 **SN 값인 13자리 숫자만** 선택합니다.

```
SN: 0637145699424
    └─────────────┘
       이 부분
```

`SN:`까지 포함하지 않는 것이 좋습니다.

![[{AED99F83-2E5B-4BB1-A8B0-E6887E8726A3}.png]]

드래그해서 숫자 영역을 선택한 뒤 **Enter 키 또는 Space 키**를 누룹니다.

이미지가 6장이므로 6번을 반복합니다.

---

## 의사코드로 읽어봅니다

```
프로그램 시작

        ↓

필요한 기능을 불러온다

Path
→ 파일과 폴더 경로 처리

cv2
→ 이미지 읽기, ROI 선택, 결과 저장

        ↓

현재 Python 파일 위치를 기준으로
subject9_machinevision 프로젝트 경로를 찾는다

        ↓

본 실습 이미지 폴더를 지정한다

samples/day06/product/

        ↓

OCR Rule을 저장할 위치를 지정한다

configs/day06_ocr_rules.json

        ↓

ROI 결과를 저장할 위치를 지정한다

outputs/day06/roi/

        ↓

product 폴더에서
jpg / jpeg / png 이미지를 찾는다

        ↓

찾은 이미지를 한 장씩 반복한다

        ↓

이미지를 OpenCV로 읽는다

        ↓

화면에 이미지를 보여준다

        ↓

학생이 마우스로
SN의 13자리 숫자 영역을 선택한다

        ↓

선택한 x, y, width, height를

x1, y1, x2, y2
좌표로 변환한다

        ↓

선택한 부분만 잘라
SN ROI 이미지를 만든다

        ↓

원본 이미지에는
ROI 위치를 사각형으로 표시한다

        ↓

두 이미지를 저장한다

ROI 위치 확인용 Preview
+
실제 SN Crop 이미지

        ↓

현재 이미지의 ROI 좌표를
roi_by_file에 기록한다

        ↓

6장 모두 같은 작업을 반복한다

        ↓

OCR 실험에 사용할 기본 설정을 만든다

PSM = 7
→ 한 줄 문자

Whitelist = 0123456789
→ 숫자만 허용

Regex = ^[0-9]{13}$
→ 정확히 13자리 숫자인지 검사

        ↓

이미지별 ROI 좌표와 OCR 기본 설정을

configs/day06_ocr_rules.json

에 저장한다

        ↓

프로그램 종료
```

---

## 실행 후 만들어지는 결과

다음과 같은 파일이 만들어집니다.

```
outputs/day06/roi/
├── 0637145699424_side_roi_preview.jpg
├── 0637145699424_side_sn_roi.jpg
├── 0639873538899_side_roi_preview.jpg
├── 0639873538899_side_sn_roi.jpg
├── ...
└── 0793387694868_side_sn_roi.jpg
```

![[{0566F51B-CF0D-475B-8608-1C7BCE058728}.png]]

그리고:

```
configs/day06_ocr_rules.json
```

에는 이미지별 ROI 좌표가 저장됩니다.

형태는 다음과 같습니다.

```json
{
  "baseline_version": "day06-ocr-draft",
  "reference_file": "0637145699424_side.png",
  "roi_by_file": {
    "0637145699424_side.png": [
      434,
      294,
      529,
      318
    ],
    "0639873538899_side.png": [
      687,
      271,
      777,
      300
    ],
    "0679256889404_side.png": [
      553,
      253,
      637,
      279
    ],
    "0702739449714_side.png": [
      316,
      267,
      405,
      292
    ],
    "0716954573504_side.png": [
      528,
      266,
      620,
      292
    ],
    "0793387694868_side.png": [
      466,
      270,
      561,
      300
    ]
  },
  "preprocess_mode": "NOT_SELECTED",
  "clahe_clip_limit": 2.0,
  "clahe_grid": [
    8,
    8
  ],
  "ocr_lang": "eng",
  "ocr_oem": 3,
  "ocr_psm": 7,
  "whitelist": "0123456789",
  "expected_pattern": "^[0-9]{13}$",
  "allow_context_correction": false
}
```
---

# 14. 저장된 SN ROI와 Rule을 확인합니다

`41_day06_select_text_roi.py`를 실행하여 본 실습 이미지 6장에서 **13자리 SN 영역을 각각 선택했습니다.**

결과는 다음 폴더에 저장됩니다.

```
outputs/day06/roi/
```

각 이미지마다 두 종류의 파일이 생성됩니다.

```
*_roi_preview.jpg
→ 원본 이미지에서 선택한 ROI 위치 확인

*_sn_roi.jpg
→ 실제 OCR에 입력할 13자리 SN 영역
```

실제 저장된 SN ROI를 확인하면 다음 6개의 숫자 영역이 정상적으로 잘렸습니다.

```
0637145699424
0639873538899
0679256889404
0702739449714
0716954573504
0793387694868
```

확인할 것은 다음과 같습니다.

```
13자리 숫자가 모두 포함되었는가?
        ↓
       YES

숫자 앞이나 뒤가 잘리지 않았는가?
        ↓
       YES

SN:, Ch.-B., Verw.bis 등의
불필요한 문자가 포함되지 않았는가?
        ↓
       YES
```

따라서 현재 ROI는 다음 OCR 전처리 실험에 사용할 수 있는 상태입니다.

---

## 14.1 이미지별 ROI 좌표도 저장되었습니다

선택한 ROI 좌표는 다음 파일에 자동으로 저장되었습니다.

```
configs/day06_ocr_rules.json
```

실제 결과:

```json
"roi_by_file": {
  "0637145699424_side.png": [434, 294, 529, 318],
  "0639873538899_side.png": [687, 271, 777, 300],
  "0679256889404_side.png": [553, 253, 637, 279],
  "0702739449714_side.png": [316, 267, 405, 292],
  "0716954573504_side.png": [528, 266, 620, 292],
  "0793387694868_side.png": [466, 270, 561, 300]
}
```

이미지마다 제품 위치가 다르기 때문에 ROI 좌표가 서로 다른 것이 정상입니다.

```
제품 위치 변화
        ↓
이미지별 ROI 좌표도 달라짐
        ↓
각 이미지에서 SN 영역만 정확하게 Crop
```

---

## 14.2 `day06_ocr_rules.json`에 저장된 기본 OCR 설정을 확인합니다

`41_day06_select_text_roi.py`를 실행하면 ROI 좌표뿐만 아니라 앞으로 OCR에서 사용할 **기본 설정**도 다음 파일에 저장됩니다.

```
configs/day06_ocr_rules.json
```

현재 우리가 생성한 JSON에는 다음과 같은 값이 들어 있습니다.

```
preprocess_mode: NOT_SELECTED
→ 아직 어떤 전처리를 사용할지 결정하지 않음

ocr_psm: 7
→ Tesseract에게 "한 줄짜리 문자"라고 알려줌

whitelist: 0123456789
→ 이번 SN에서는 숫자만 읽도록 제한

expected_pattern: ^[0-9]{13}$
→ OCR 결과가 정확히 13자리 숫자인지 검사

allow_context_correction: false
→ OCR 결과의 숫자를 임의로 고치지 않음
```

```json
"preprocess_mode": "NOT_SELECTED",
"ocr_psm": 7,
"whitelist": "0123456789",
"expected_pattern": "^[0-9]{13}$",
"allow_context_correction": false
```

여기서 가장 중요한 것은:

```
preprocess_mode = NOT_SELECTED
```

입니다.

아직 다음 중 어떤 이미지가 OCR에 가장 잘 맞는지 실험하지 않았기 때문입니다.

```
원본
Gray
CLAHE
Otsu
CLAHE + Otsu
```

다음 실습에서 이 다섯 가지를 비교합니다.

```
SN ROI
        ↓
여러 전처리 적용
        ↓
실제 OCR 결과 비교
        ↓
가장 잘 읽히는 방법 선택
        ↓
preprocess_mode에 저장
```

예를 들어 실험 결과 `clahe`가 가장 잘 읽혔다면 나중에는:

```
"preprocess_mode": "clahe"
```

처럼 바뀝니다.

> 즉 지금 `day06_ocr_rules.json`은 최종 OCR 규칙이 아니라 ROI와 OCR 기본값을 먼저 저장해 둔 초안 상태입니다.

---

## 14.3 41번 실습 결과

```
제품 이미지 6장
        ↓
SN 숫자 영역 각각 선택
        ↓
SN ROI 6개 생성
        ↓
이미지별 ROI 좌표 저장
        ↓
기본 OCR Rule 저장
        ↓
OCR 입력 데이터 준비 완료
```

> `41_day06_select_text_roi.py`에서는 아직 문자를 읽은 것이 아닙니다.  
> 다음 전처리와 OCR 실험에 사용할 13자리 SN 영역을 준비한 것입니다.

---

# 15. 여러 OCR 실습에서 사용할 공통 함수를 만듭니다

앞의 `41_day06_select_text_roi.py`에서 다음 작업을 완료했습니다.

```
제품 이미지 6장
        ↓
각 이미지의 SN ROI 선택
        ↓
이미지별 ROI 좌표 저장
        ↓
day06_ocr_rules.json 생성
```

이제 이후 실습에서는 다음 작업을 여러 번 반복하게 됩니다.

```
JSON Rule 읽기
     ↓
이미지에 맞는 ROI 좌표 찾기
     ↓
SN 영역 Crop
     ↓
전처리
     ↓
Tesseract OCR
```

같은 코드를 실행 파일마다 반복해서 작성하지 않도록 **공통 함수 파일**을 만듭니다.

파일:

```
src/day06_ocr_utils.py
```

작성합니다.

```python
import json
from pathlib import Path

import cv2
import pytesseract


# ========================================
# 프로젝트 경로
# ========================================

PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

DEFAULT_RULES_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_ocr_rules.json"
)


# ========================================
# OCR Rule 읽기
# ========================================

def load_rules(
    path=DEFAULT_RULES_PATH,
):
    return json.loads(
        Path(path).read_text(
            encoding="utf-8"
        )
    )


# ========================================
# 이미지별 ROI 좌표 가져오기
# ========================================

def get_roi_for_file(
    rules,
    file_name,
):
    roi_by_file = rules[
        "roi_by_file"
    ]

    if file_name not in roi_by_file:
        raise KeyError(
            f"ROI 정보가 없습니다: {file_name}"
        )

    return roi_by_file[
        file_name
    ]


# ========================================
# ROI Crop
# ========================================

def crop_roi(
    image,
    roi,
):
    x1, y1, x2, y2 = roi

    return image[
        y1:y2,
        x1:x2,
    ]


# ========================================
# CLAHE
# ========================================

def apply_clahe(
    gray,
    rules,
):
    clahe = cv2.createCLAHE(
        clipLimit=float(
            rules["clahe_clip_limit"]
        ),
        tileGridSize=tuple(
            rules["clahe_grid"]
        ),
    )

    return clahe.apply(
        gray
    )


# ========================================
# 전처리 후보 만들기
# ========================================

def make_preprocess_variants(
    roi,
    rules,
):
    # Gray
    gray = cv2.cvtColor(
        roi,
        cv2.COLOR_BGR2GRAY,
    )

    # CLAHE
    clahe = apply_clahe(
        gray,
        rules,
    )

    # Gray + Otsu
    _, otsu = cv2.threshold(
        gray,
        0,
        255,
        cv2.THRESH_BINARY
        + cv2.THRESH_OTSU,
    )

    # CLAHE + Otsu
    _, clahe_otsu = cv2.threshold(
        clahe,
        0,
        255,
        cv2.THRESH_BINARY
        + cv2.THRESH_OTSU,
    )

    return {
        "original": roi,
        "gray": gray,
        "clahe": clahe,
        "otsu": otsu,
        "clahe_otsu": clahe_otsu,
    }


# ========================================
# 선택된 전처리 적용
# ========================================

def preprocess_for_ocr(
    roi,
    rules,
):
    variants = (
        make_preprocess_variants(
            roi,
            rules,
        )
    )

    mode = rules[
        "preprocess_mode"
    ]

    if mode not in variants:
        raise ValueError(
            "아직 사용할 전처리가 "
            f"선택되지 않았습니다: {mode}"
        )

    return variants[
        mode
    ]


# ========================================
# Tesseract 설정 만들기
# ========================================

def build_ocr_config(
    rules,
):
    return (
        f"--oem {rules['ocr_oem']} "
        f"--psm {rules['ocr_psm']} "
        "-c "
        f"tessedit_char_whitelist="
        f"{rules['whitelist']}"
    )


# ========================================
# OCR 실행
# ========================================

def run_ocr(
    image,
    rules,
):
    config = (
        build_ocr_config(
            rules
        )
    )

    return pytesseract.image_to_string(
        image,
        lang=rules["ocr_lang"],
        config=config,
    )
```

## 이 공통 파일은 무슨 일을 하나요?

```
load_rules()
→ day06_ocr_rules.json 읽기

get_roi_for_file()
→ 해당 이미지의 ROI 좌표 찾기

crop_roi()
→ SN 영역만 잘라내기

make_preprocess_variants()
→ 여러 전처리 결과 만들기

preprocess_for_ocr()
→ 나중에 선택된 전처리 하나만 적용

build_ocr_config()
→ Tesseract 실행 조건 만들기

run_ocr()
→ 실제 OCR 실행
```

현재는 공통 기능을 **준비하는 단계**입니다.

`day06_ocr_utils.py` 자체를 직접 실행하는 것이 아니라 이후 `42`, `43`, `45` 등의 실행 파일에서 불러와 사용합니다.

### 의사코드

```
프로그램에서 공통으로 사용할 기능을 준비한다

        ↓

프로젝트 위치를 찾는다

        ↓

configs/day06_ocr_rules.json
위치를 지정한다

        ↓

load_rules()

→ JSON에 저장된
   ROI와 OCR 설정을 읽는다

        ↓

get_roi_for_file()

→ 현재 이미지 파일명을 이용하여
   그 이미지의 ROI 좌표를 찾는다

        ↓

crop_roi()

→ 찾은 좌표를 이용하여
   SN 숫자 부분만 자른다

        ↓

make_preprocess_variants()

→ 같은 SN ROI로

   Original
   Gray
   CLAHE
   Otsu
   CLAHE + Otsu

   다섯 가지 이미지를 만든다

        ↓

preprocess_for_ocr()

→ 나중에 최종 선택된
   전처리 한 가지를 가져온다

        ↓

build_ocr_config()

→ PSM
   Whitelist
   OCR Engine 설정을 조합한다

        ↓

run_ocr()

→ 준비된 이미지를
   Tesseract에 전달하여
   문자를 읽는다
```

---

# 16. `42_day06_preprocess_compare.py` — SN 전처리 후보를 비교합니다

41번에서는 6장의 SN ROI를 준비했습니다.

42번에서는 먼저 **대표 이미지 1장**을 사용하여 같은 SN 영역에 여러 전처리를 적용합니다.

대표 이미지는 `day06_ocr_rules.json`에 저장된:

```
reference_file
```

을 사용합니다.

현재 값은:

```
0637145699424_side.png
```

입니다.

실험 흐름:

```
0637145699424_side.png
        ↓
저장된 ROI 좌표 확인
        ↓
0637145699424 영역 Crop
        ↓
Original
Gray
CLAHE
Otsu
CLAHE + Otsu
        ↓
각 결과 이미지 저장
```

> 이 단계에서는 아직 어떤 전처리가 가장 좋은지 결정하지 않습니다.  
> 먼저 서로 다른 OCR 입력 이미지를 만들어 비교합니다.

파일:

```
src/42_day06_preprocess_compare.py
```

작성합니다.

```python
from pathlib import Path

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    crop_roi,
    get_roi_for_file,
    load_rules,
    make_preprocess_variants,
)


# ========================================
# Rule 읽기
# ========================================

rules = load_rules()


# ========================================
# 대표 이미지
# ========================================

file_name = rules[
    "reference_file"
]

IMAGE_PATH = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
    / file_name
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "preprocess"
)


# ========================================
# 이미지 읽기
# ========================================

image = cv2.imread(
    str(IMAGE_PATH)
)

if image is None:
    raise FileNotFoundError(
        IMAGE_PATH
    )


# ========================================
# 대표 이미지의 ROI 좌표 가져오기
# ========================================

roi_coordinates = (
    get_roi_for_file(
        rules,
        file_name,
    )
)


# ========================================
# SN 영역 Crop
# ========================================

roi = crop_roi(
    image,
    roi_coordinates,
)


# ========================================
# 전처리 후보 만들기
# ========================================

variants = (
    make_preprocess_variants(
        roi,
        rules,
    )
)


# ========================================
# 결과 폴더
# ========================================

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# ========================================
# 전처리 결과 저장
# ========================================

for index, (
    name,
    result,
) in enumerate(
    variants.items(),
    start=1,
):

    output_path = (
        OUTPUT_DIR
        / f"{index:02d}_{name}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print(
        "Saved:",
        output_path
    )


print()
print(
    "Reference:",
    file_name
)

print(
    "ROI:",
    roi_coordinates
)

print(
    "Preprocess Compare Complete"
)
```

실행합니다.

```
python src/42_day06_preprocess_compare.py
```

### 의사코드

```
프로그램 시작

        ↓

day06_ocr_utils에서
필요한 공통 함수를 불러온다

        ↓

day06_ocr_rules.json을 읽는다

        ↓

reference_file을 확인한다

0637145699424_side.png

        ↓

대표 제품 이미지를 읽는다

        ↓

roi_by_file에서

0637145699424_side.png

에 해당하는 ROI 좌표를 찾는다

        ↓

저장된 좌표를 이용하여
13자리 SN 영역만 Crop한다

        ↓

같은 SN ROI에
다섯 가지 처리를 적용한다

        ↓

1. Original
→ 원본 SN ROI

2. Gray
→ 흑백 영상

3. CLAHE
→ 문자와 배경의 대비 강화

4. Otsu
→ 자동 Threshold로 흑백 분리

5. CLAHE + Otsu
→ 대비 개선 후 흑백 분리

        ↓

각 결과를

outputs/day06/preprocess/

폴더에 저장한다

        ↓

프로그램 종료
```

---

## 16.2 실행 결과를 확인합니다

실행이 끝나면 다음 파일이 생성됩니다.

```
outputs/day06/preprocess/
├── 01_original.jpg
├── 02_gray.jpg
├── 03_clahe.jpg
├── 04_otsu.jpg
└── 05_clahe_otsu.jpg
```

모두 같은:

```
0637145699424
```

를 서로 다른 방식으로 처리한 이미지입니다.

![[{82AFEA64-2C58-4007-91A5-F242638EBC1A}.png]]

즉 검사 대상이 바뀐 것이 아니라 OCR에 넣는 영상의 상태만 바꿔본 것입니다.

---

# 17. 전처리 결과를 비교합니다

저장된 다섯 이미지를 직접 열어봅니다.

```
01_original.jpg
02_gray.jpg
03_clahe.jpg
04_otsu.jpg
05_clahe_otsu.jpg
```

현재 확인할 것은 다음 정도입니다.

```
13자리 숫자가 모두 유지되어 있는가?

숫자와 배경의 차이가 잘 보이는가?

숫자의 선이 끊어지지 않았는가?

0 / 6 / 8 / 9처럼
모양이 비슷한 숫자의 형태가 유지되는가?

Otsu 처리 후 숫자가
지나치게 뭉개지거나 사라지지 않았는가?

CLAHE가 배경 Noise까지
지나치게 강조하지 않았는가?
```

예를 들어:

```
Original
→ 원래 문자 상태 확인

Gray
→ 색상 제거 후 밝기만 비교

CLAHE
→ 흐린 숫자의 Contrast 개선 여부 확인

Otsu
→ 숫자와 배경이 잘 분리되는지 확인

CLAHE + Otsu
→ 대비 개선 후 이진화 효과 확인
```

하지만 여기서 눈으로 보기 좋은 이미지 하나를 바로 선택하지 않습니다.

각 파일의 의미는 다음과 같습니다.

| 파일                  | 처리           | 확인할 내용                |
| ------------------- | ------------ | --------------------- |
| `01_original.jpg`   | 원본 ROI       | 원래 SN 문자 상태           |
| `02_gray.jpg`       | Gray         | 색상을 제거하고 밝기 정보만 사용    |
| `03_clahe.jpg`      | CLAHE        | 흐린 문자와 배경의 대비를 강화     |
| `04_otsu.jpg`       | Otsu         | 문자와 배경을 자동 기준으로 흑백 분리 |
| `05_clahe_otsu.jpg` | CLAHE + Otsu | 대비를 강화한 후 흑백으로 분리     |

현재 결과를 눈으로 보면 5개 모두 `0637145699424`라는 숫자 형태가 유지되고 있습니다.

하지만 여기서 바로:

```
CLAHE가 최고다
Otsu가 최고다
```

라고 결정하지 않습니다.

사람 눈에는 선명해 보여도 Tesseract가 잘못 읽을 수 있기 때문입니다.

```
42번
전처리 후보 생성
        ↓
43번
실제 OCR 실행
        ↓
13자리 숫자 결과 비교
        ↓
가장 적절한 전처리 결정
```

> 즉 42번은 OCR에 넣을 이미지를 만드는 실험이고, 43번은 그 이미지들을 실제로 읽어보는 실험입니다.

---

# PART3 — Experiment 2: Tesseract 실행 + Raw OCR 비교 (18~24)

---

# 18. 두 번째 Experiment의 질문

앞에서는 같은 SN을 다섯 가지 방식으로 전처리했습니다.

이제 실제 Tesseract를 실행하여 다음 질문을 확인합니다.

> 같은 `0637145699424`를 입력했는데 전처리 방법에 따라 OCR 결과가 달라질까요? 어떤 전처리가 실제 13자리 SN을 가장 정확하게 읽을까요?

실험 흐름은 다음과 같습니다.

```
같은 SN ROI

0637145699424

        ↓

Original
Gray
CLAHE
Otsu
CLAHE + Otsu

        ↓

각 이미지를 같은 크기로 확대

        ↓

Tesseract OCR

        ↓

실제 OCR 결과 비교

        ↓

0637145699424와 정확히 같은가?
```

이번 SN ROI는 원본 이미지에서 작은 문자 영역만 잘라냈기 때문에 크기가 작습니다.

따라서 다섯 후보 모두에 **같은 조건으로 4배 확대**한 뒤 OCR을 실행합니다.

```
작은 SN ROI
        ↓
4배 확대
        ↓
Tesseract가 문자의 획을
조금 더 크게 볼 수 있도록 함
```

전처리마다 확대율을 다르게 적용하지 않고 모두 같은 `4배`를 사용하므로 비교 조건은 같습니다.

---

# 19. `43_day06_ocr_compare.py` — 전처리별 OCR 결과를 비교합니다

이 파일은 42번에서 만든 것과 같은 다섯 가지 전처리를 다시 적용한 후 실제 **Tesseract OCR 결과를 비교**합니다.

비교할 정답은 대표 이미지 파일명에 들어 있는:

```
0637145699424
```

입니다.

파일:

```
src/43_day06_ocr_compare.py
```

작성합니다.

```python
import csv
from pathlib import Path

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    crop_roi,
    get_roi_for_file,
    load_rules,
    make_preprocess_variants,
    run_ocr,
)


# ========================================
# Rule 읽기
# ========================================

rules = load_rules()


# ========================================
# 대표 이미지 확인
# ========================================

file_name = rules[
    "reference_file"
]

IMAGE_PATH = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
    / file_name
)

REPORT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_preprocess_compare.csv"
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "preprocess"
)


# ========================================
# 파일명에서 실제 SN 가져오기
# ========================================

expected_sn = (
    Path(file_name)
    .stem
    .split("_side")[0]
)


# ========================================
# 이미지 읽기
# ========================================

image = cv2.imread(
    str(IMAGE_PATH)
)

if image is None:
    raise FileNotFoundError(
        IMAGE_PATH
    )


# ========================================
# 해당 이미지의 ROI 가져오기
# ========================================

roi_coordinates = (
    get_roi_for_file(
        rules,
        file_name,
    )
)


# ========================================
# SN 영역 Crop
# ========================================

roi = crop_roi(
    image,
    roi_coordinates,
)


# ========================================
# 전처리 후보 만들기
# ========================================

variants = (
    make_preprocess_variants(
        roi,
        rules,
    )
)


OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


rows = []


# ========================================
# 전처리별 OCR 비교
# ========================================

for index, (
    name,
    processed,
) in enumerate(
    variants.items(),
    start=1,
):

    # ------------------------------------
    # 작은 문자 ROI를 OCR용으로 4배 확대
    # ------------------------------------

    enlarged = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )


    # ------------------------------------
    # Tesseract OCR
    # ------------------------------------

    raw_text = run_ocr(
        enlarged,
        rules,
    )


    # 줄바꿈과 앞뒤 공백만 제거
    ocr_text = raw_text.strip()


    # ------------------------------------
    # 실제 SN과 정확히 같은지 확인
    # ------------------------------------

    exact_match = (
        ocr_text
        == expected_sn
    )


    # ------------------------------------
    # OCR 입력 이미지도 저장
    # ------------------------------------

    output_path = (
        OUTPUT_DIR
        / f"ocr_{index:02d}_{name}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        enlarged,
    )


    # ------------------------------------
    # 결과 기록
    # ------------------------------------

    rows.append(
        {
            "pipeline":
                name,

            "expected_sn":
                expected_sn,

            "raw_text":
                repr(raw_text),

            "ocr_text":
                ocr_text,

            "exact_match":
                exact_match,
        }
    )


    print(
        "=" * 60
    )

    print(
        "Pipeline:",
        name
    )

    print(
        "Expected:",
        expected_sn
    )

    print(
        "Raw OCR:",
        repr(raw_text)
    )

    print(
        "OCR Text:",
        ocr_text
    )

    print(
        "Exact Match:",
        exact_match
    )


# ========================================
# CSV 저장
# ========================================

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


with REPORT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "pipeline",
            "expected_sn",
            "raw_text",
            "ocr_text",
            "exact_match",
        ],
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


print()

print(
    "Saved:",
    REPORT_PATH
)

print(
    "OCR Compare Complete"
)
```

실행합니다.

```
python src/43_day06_ocr_compare.py
```

### 의사코드

```
프로그램 시작

        ↓

공통 OCR 함수를 불러온다

        ↓

day06_ocr_rules.json을 읽는다

        ↓

대표 이미지 파일명을 확인한다

0637145699424_side.png

        ↓

파일명에서 정답 SN을 가져온다

0637145699424

        ↓

대표 제품 이미지를 읽는다

        ↓

roi_by_file에서
이 이미지의 ROI 좌표를 찾는다

        ↓

저장된 좌표로
SN 숫자 영역만 자른다

        ↓

같은 SN에
5가지 전처리를 적용한다

Original
Gray
CLAHE
Otsu
CLAHE + Otsu

        ↓

각 이미지를 동일하게
4배 확대한다

        ↓

각 이미지를
Tesseract에 넣는다

        ↓

Tesseract가 읽은
Raw OCR 결과를 받는다

        ↓

앞뒤 공백과 줄바꿈을 제거한다

        ↓

OCR 결과가 정답

0637145699424

와 정확히 같은지 비교한다

        ↓

전처리별 결과를 출력한다

Pipeline
Expected SN
Raw OCR
OCR Text
Exact Match

        ↓

모든 결과를

reports/day06_preprocess_compare.csv

에 저장한다

        ↓

프로그램 종료
```

---

## 19.2 실행 결과를 해석합니다

실제 실행 결과는 다음과 같습니다.

```
Pipeline: original
Expected: 0637145699424
OCR Text: 0637145699424
Exact Match: True

Pipeline: gray
Expected: 0637145699424
OCR Text: 0637145699424
Exact Match: True

Pipeline: clahe
Expected: 0637145699424
OCR Text: 0637145699424
Exact Match: True

Pipeline: otsu
Expected: 0637145699424
OCR Text: 0637245697424
Exact Match: False

Pipeline: clahe_otsu
Expected: 0637145699424
OCR Text: 0637245670424
Exact Match: False
```

결과를 간단히 읽으면 다음과 같습니다.

```
original
→ 13자리 SN을 정확히 읽음
→ 성공

gray
→ 13자리 SN을 정확히 읽음
→ 성공

clahe
→ 13자리 SN을 정확히 읽음
→ 성공

otsu
→ 일부 숫자를 잘못 인식함
→ 실패

clahe_otsu
→ 일부 숫자를 잘못 인식함
→ 실패
```

즉 이번 대표 이미지에서는:

```
Original
Gray
CLAHE
        ↓
OCR 성공

Otsu
CLAHE + Otsu
        ↓
OCR 실패
```

라는 결과가 나왔습니다.

특히 Otsu 계열에서는 이진화 과정에서 문자 모양이 달라지면서 일부 숫자를 잘못 읽은 것으로 볼 수 있습니다.

> 이번 결과에서는 강한 이진화가 OCR 성능을 오히려 떨어뜨렸습니다.

---

## 19.3 CSV 결과를 확인합니다

실행 결과는 다음 파일에도 저장되었습니다.

```
reports/day06_preprocess_compare.csv
```

실제 내용:

```
pipeline,expected_sn,raw_text,ocr_text,exact_match
original,0637145699424,'0637145699424\n',0637145699424,True
gray,0637145699424,'0637145699424\n',0637145699424,True
clahe,0637145699424,'0637145699424\n',0637145699424,True
otsu,0637145699424,'0637245697424\n',0637245697424,False
clahe_otsu,0637145699424,'0637245670424\n',0637245670424,False
```

표로 보면 다음과 같습니다.

| Pipeline     | OCR 결과          | Exact Match | 결과  |
| ------------ | --------------- | ----------: | --- |
| `original`   | `0637145699424` |        True | 성공  |
| `gray`       | `0637145699424` |        True | 성공  |
| `clahe`      | `0637145699424` |        True | 성공  |
| `otsu`       | `0637245697424` |       False | 실패  |
| `clahe_otsu` | `0637245670424` |       False | 실패  |
이번 실험에서는 **Original, Gray, CLAHE 세 가지가 모두 정확히 인식**되었습니다.

따라서 다음 단계에서는 이 세 가지 중 하나를 최종 전처리 후보로 선택하면 됩니다.

```
Original
Gray
CLAHE
        ↓
모두 성공
        ↓
더 단순하고 안정적인 방법 우선 검토
```

현재 결과만 본다면 굳이 강한 전처리를 추가할 필요가 없으므로 **`original` 또는 `gray`를 우선 후보**로 보는 것이 자연스럽습니다.

---

# 20. 전처리별 OCR 결과를 정리합니다

대표 SN의 실제 값은 다음입니다.

```
0637145699424
```

5가지 전처리 결과를 비교하면 다음과 같습니다.

|Pipeline|OCR 결과|Exact Match|
|---|---|---|
|`original`|`0637145699424`|True|
|`gray`|`0637145699424`|True|
|`clahe`|`0637145699424`|True|
|`otsu`|`0637245697424`|False|
|`clahe_otsu`|`0637245670424`|False|

결과는 명확합니다.

```
Original
Gray
CLAHE
→ 정확하게 인식

Otsu
CLAHE + Otsu
→ 일부 숫자를 잘못 인식
```

이번 이미지에서는 이진화를 적용한 `Otsu` 계열보다 원본에 가까운 영상이 OCR에 더 유리했습니다.

따라서 현재 우선 후보는 가장 단순한:

```
original
```

로 정합니다.

> 아직 최종 Rule을 고정하는 단계는 아닙니다.  
> 현재는 대표 이미지 실험을 통해 `original`을 우선 후보로 정한 상태입니다.

---

# 21. 이번 OCR에서 `PSM 7`을 사용한 이유를 확인합니다

PSM 7은 Tesseract OCR에게 “이 이미지는 한 줄짜리 문자다”라고 알려주는 설정입니다.

현재 `day06_ocr_rules.json`에는 다음 설정이 들어 있습니다.

```
ocr_psm = 7
```

이번 OCR 입력은 다음과 같은 **한 줄짜리 SN**입니다.

```
0637145699424
```

따라서 Tesseract에게:

```
이 이미지는
한 줄의 문자라고 생각하고 읽어라
```

라고 알려주기 위해 `PSM 7`을 사용했습니다.

> PSM 번호를 외우는 것이 목적이 아니라, OCR 대상의 문자 배치에 맞게 설정한다는 것을 이해하면 됩니다.

---

# 22. 숫자만 읽도록 Whitelist를 제한했습니다

**Whitelist**는 OCR이 읽을 때 허용할 문자 종류를 미리 제한하는 설정입니다.

현재 검사 대상은 13자리 SN이므로 필요한 문자는:

```
0 1 2 3 4 5 6 7 8 9
```

뿐입니다.

따라서 현재 설정은:

```
whitelist = 0123456789
```

입니다.

즉 Tesseract가 영문이나 불필요한 기호까지 후보로 고려하지 않고 **숫자에 집중해서 읽도록 제한**한 것입니다.

```
이번 SN 검사
→ 숫자만 허용

Mini Challenge에서
영문 + 숫자를 검사한다면
→ Whitelist도 다시 변경
```

하게 됩니다.

---

# 23. Raw OCR과 정리된 OCR 결과를 구분합니다

실제 실행 결과를 보면:

```
Raw OCR:
'0637145699424\n'
```

처럼 마지막에 `\n`이 포함되어 있습니다.

이것은 Tesseract가 반환한 원본 문자열(Raw OCR)입니다.

코드에서:

```python
ocr_text = raw_text.strip()
```

을 사용하면 앞뒤 공백과 줄바꿈을 제거하여:

```
0637145699424
```

만 남습니다.

따라서 두 값을 구분합니다.

```
Raw OCR
'0637145699424\n'
→ Tesseract가 실제로 반환한 값

OCR Text
0637145699424
→ 앞뒤 공백과 줄바꿈을 정리한 값
```

Raw OCR은 나중에 OCR이 어떤 결과를 실제로 반환했는지 분석하기 위해 그대로 기록합니다.

---

# 24. Experiment 2를 마무리합니다

`43_day06_ocr_compare.py`를 통해 다음을 확인했습니다.

```
같은 SN ROI
        ↓
5가지 전처리
        ↓
Tesseract OCR
        ↓
실제 SN과 비교
```

이번 결과:

```
Original      → 성공
Gray          → 성공
CLAHE         → 성공
Otsu          → 실패
CLAHE + Otsu  → 실패
```

따라서 현재 정리하면:

```
우선 전처리 후보
→ original

OCR 설정
→ PSM 7

허용 문자
→ 0~9

검사 대상
→ 13자리 SN
```

입니다.

현재 `day06_ocr_rules.json`의:

```
preprocess_mode = NOT_SELECTED
```

는 아직 그대로 두어도 됩니다.

다음 PART에서:

```
OCR 결과
        ↓
문자열 정리
        ↓
13자리 형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG
```

까지 확인한 후 최종 OCR Rule을 고정합니다.

---

# PART4 — Experiment 3: OCR 문자열 검증 + Expected SN 검사 + Rule Freeze (25~31)

---

# 25. 세 번째 Experiment에서는 무엇을 확인할까요?

43번 실험에서 Tesseract가 실제 SN을 읽는 것까지 확인했습니다.

대표 이미지의 실제 SN:

```
0637145699424
```

Tesseract 결과:

```
'0637145699424\n'
```

하지만 OCR이 문자를 읽었다고 바로 제품을 `OK`라고 판정할 수는 없습니다.

다음 두 가지를 추가로 검사해야 합니다.

```
OCR 결과
        ↓
문자열 정리
        ↓
13자리 숫자 형식인가?
        ↓
Expected SN과 같은가?
        ↓
OK / NG
```

즉 이번 Experiment의 질문은 다음입니다.

> OCR이 읽은 문자열이 올바른 13자리 SN 형식인지 확인하고, 실제 생산해야 하는 SN과 비교하여 제품을 OK / NG로 판정할 수 있을까요?

---

# 26. `day06_ocr_validation.py` — OCR 문자열을 검사하는 공통 모듈을 만듭니다

이 파일은 직접 실행하는 프로그램이 아닙니다.

앞으로 Batch 검사와 Challenge에서도 같은 문자열 검사 기능을 사용할 수 있도록 만드는 **공통 함수 파일**입니다.

파일:

```
src/day06_ocr_validation.py
```

이 파일은 다음 역할을 담당합니다.

```
Raw OCR 정리
        ↓
13자리 형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG 판정
        ↓
실패 이유 반환
```

작성합니다.

```python
import re


# ========================================
# OCR 문자열 정리
# ========================================

def clean_ocr_text(
    raw_text,
):
    # 앞뒤 공백과 줄바꿈 제거
    text = raw_text.strip()

    # 중간에 들어간 공백 제거
    text = re.sub(
        r"\s+",
        "",
        text,
    )

    return text


# ========================================
# SN 형식 검사
# ========================================

def validate_pattern(
    text,
    pattern,
):
    return bool(
        re.fullmatch(
            pattern,
            text,
        )
    )


# ========================================
# 최종 제품 판정
# ========================================

def inspect_text(
    raw_text,
    expected_text,
    pattern,
):
    # 1. OCR 문자열 정리
    cleaned_text = (
        clean_ocr_text(
            raw_text
        )
    )

    reasons = []


    # 2. OCR 결과가 비어 있는지 확인
    if not cleaned_text:
        reasons.append(
            "NG_OCR_EMPTY"
        )


    # 3. 13자리 SN 형식인지 확인
    format_ok = (
        validate_pattern(
            cleaned_text,
            pattern,
        )
    )

    if not format_ok:
        reasons.append(
            "NG_FORMAT"
        )


    # 4. Expected SN과 비교
    expected_match = (
        cleaned_text
        == expected_text
    )

    if not expected_match:
        reasons.append(
            "NG_EXPECTED_MISMATCH"
        )


    # 5. 최종 OK / NG
    predicted_status = (
        "OK"
        if not reasons
        else "NG"
    )


    return {
        "raw_text":
            raw_text,

        "cleaned_text":
            cleaned_text,

        "final_text":
            cleaned_text,

        "format_ok":
            format_ok,

        "expected_match":
            expected_match,

        "predicted_status":
            predicted_status,

        "reasons":
            reasons,
    }
```

---

## 26.1 이 파일은 무엇을 하는 코드인가요?

예를 들어 Tesseract가 다음과 같이 반환했습니다.

```
'0637145699424\n'
```

`day06_ocr_validation.py`는 이것을 다음과 같이 처리합니다.

```
Raw OCR
'0637145699424\n'

        ↓
Cleaning

0637145699424

        ↓
Regex

^[0-9]{13}$

        ↓
13자리 숫자
→ Format OK

        ↓
Expected SN

0637145699424

        ↓
일치

        ↓
OK
```

반대로 OCR 결과가:

```
0637245697424
```

라면 13자리 숫자 형식 자체는 정상입니다.

하지만 Expected SN:

```
0637145699424
```

와 다르기 때문에:

```
Format OK
Expected Match = False

        ↓

NG_EXPECTED_MISMATCH
```

가 됩니다.

---

## 26.2 의사코드로 읽어봅니다

```
OCR에서 읽은 문자열을 받는다

        ↓

앞뒤 공백과
줄바꿈을 제거한다

        ↓

문자열이 비어 있는가?

YES
→ NG_OCR_EMPTY 기록

        ↓

문자열이
정확히 13자리 숫자인지 확인한다

^[0-9]{13}$

        ↓

형식이 틀리면

NG_FORMAT 기록

        ↓

OCR 결과와
Expected SN을 비교한다

        ↓

값이 다르면

NG_EXPECTED_MISMATCH 기록

        ↓

오류 이유가 하나도 없으면

OK

        ↓

오류 이유가 하나라도 있으면

NG

        ↓

검사 결과를 반환한다
```

---

# 27. Cleaning은 OCR 결과의 의미를 바꾸는 작업이 아닙니다

실제 43번 결과를 보면:

```
Raw OCR:
'0637145699424\n'
```

처럼 마지막에 줄바꿈 `\n`이 포함되어 있습니다.

Cleaning은 이것을:

```
0637145699424
```

로 정리하는 작업입니다.

```
Raw OCR
'0637145699424\n'

        ↓

Cleaning

0637145699424
```

이번 SN 실습에서는 숫자를 다른 숫자로 자동 수정하지 않습니다.

예를 들어:

```
OCR
0637245697424
```

를 프로그램이 임의로:

```
0637145699424
```

로 바꾸지 않습니다.

왜냐하면 실제로 잘못 인쇄된 SN인지 OCR 오류인지 구분해야 하기 때문입니다.

따라서 현재 설정도:

```
allow_context_correction = false
```

로 유지합니다.

---

# 28. Regex와 Expected SN 검사는 서로 다른 검사입니다

이번 SN 형식은 다음과 같습니다.

```
숫자 13자리
```

Regex:

```
^[0-9]{13}$
```

예를 들어:

```
0637245697424
```

는 숫자 13자리이므로:

```
Format OK
```

입니다.

하지만 현재 기대 SN이:

```
0637145699424
```

라면 값은 다릅니다.

```
0637245697424
        ≠
0637145699424

        ↓

NG_EXPECTED_MISMATCH
```

즉:

```
Regex
→ SN 모양이 올바른가?

Expected SN
→ 실제 값이 올바른가?
```

를 서로 따로 검사합니다.

---

# 29. 문자열 검증 함수를 먼저 테스트합니다

실제 Batch 검사를 하기 전에 방금 만든 문자열 검사 함수가 정상적으로 동작하는지 간단히 확인합니다.

터미널에서 실행합니다.

```bash
python - <<'PY'
from src.day06_ocr_validation import inspect_text

pattern = r"^[0-9]{13}$"
expected = "0637145699424"

samples = [
    "0637145699424\n",
    "063714569942",
    "0637245697424",
    "",
]

for value in samples:
    result = inspect_text(
        raw_text=value,
        expected_text=expected,
        pattern=pattern,
    )

    print("=" * 50)
    print("Raw:", repr(value))
    print("Cleaned:", result["cleaned_text"])
    print("Format OK:", result["format_ok"])
    print(
        "Expected Match:",
        result["expected_match"]
    )
    print(
        "Status:",
        result["predicted_status"]
    )
    print(
        "Reasons:",
        result["reasons"]
    )
PY
```

결과는 다음과 같이 해석할 수 있습니다.

| 입력              | Format | Expected Match | 최종 판정 | 이유                        |
| --------------- | ------ | -------------- | ----- | ------------------------- |
| `0637145699424` | True   | True           | OK    | 형식도 맞고 기대 SN과도 일치         |
| `063714569942`  | False  | False          | NG    | 13자리가 아니고 기대값도 불일치        |
| `0637245697424` | True   | False          | NG    | 13자리 형식은 맞지만 기대 SN과 불일치   |
| 빈 문자열           | False  | False          | NG    | OCR 결과가 없고 형식과 기대값 모두 불일치 |

특히 중요한 것은 다음 사례입니다.

```
0637245697424
```

이 값은:

```
Format OK = True
```

이므로 **13자리 숫자 형식 자체는 정상**입니다.

하지만 기대값:

```
0637145699424
```

와 다르므로:

```
Expected Match = False
        ↓
NG_EXPECTED_MISMATCH
        ↓
최종 판정 NG
```

가 됩니다.

즉 이번 테스트를 통해:

```
형식이 맞는다
≠
기대값과 일치한다
```

는 것을 확인했습니다.

또 빈 문자열의 경우에는:

```
NG_OCR_EMPTY
NG_FORMAT
NG_EXPECTED_MISMATCH
```

가 동시에 기록되어, OCR 자체가 문자를 읽지 못한 경우도 별도로 구분할 수 있음을 확인했습니다.

---

## 29.1 의사코드로 읽어봅니다

```
Expected SN을 정한다

0637145699424

        ↓

검사할 문자열 여러 개를 준비한다

        ↓

문자열을 하나씩
inspect_text()에 전달한다

        ↓

각 문자열마다

Cleaning 결과
형식 검사 결과
Expected SN 비교 결과
최종 OK / NG
실패 이유

를 확인한다

        ↓

문자열 검사 함수가
의도한 대로 동작하는지 확인한다
```

> 29번은 이미지 OCR 실험이 아니라 **문자열 Rule 자체가 정상적으로 동작하는지 확인하는 작은 테스트**입니다.

---

# 30. `44_day06_freeze_ocr_rules.py` — 사용할 OCR Rule을 고정합니다

지금까지의 실험 결과를 정리하면 다음과 같습니다.

43번 OCR 비교 결과:

```
Original
→ 성공

Gray
→ 성공

CLAHE
→ 성공

Otsu
→ 실패

CLAHE + Otsu
→ 실패
```

`Original`, `Gray`, `CLAHE`가 모두 성공했으므로 현재는 가장 단순한 `original`을 선택합니다.

이제 다음 Batch 검사에서 설정이 계속 바뀌지 않도록 현재 Rule을 **v1.0으로 고정(Freeze)**합니다.

파일:

```
src/44_day06_freeze_ocr_rules.py
```

작성합니다.

```python
import json
from pathlib import Path


# ========================================
# 프로젝트 경로
# ========================================

PROJECT_ROOT = (
    Path(__file__)
    .resolve()
    .parent
    .parent
)

CONFIG_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_ocr_rules.json"
)


# ========================================
# 기존 Rule 읽기
# ========================================

rules = json.loads(
    CONFIG_PATH.read_text(
        encoding="utf-8"
    )
)


# ========================================
# 실험 결과를 반영하여 Rule Freeze
# ========================================

rules[
    "baseline_version"
] = "day06-ocr-v1.0"

rules[
    "preprocess_mode"
] = "original"

rules[
    "ocr_psm"
] = 7

rules[
    "whitelist"
] = "0123456789"

rules[
    "expected_pattern"
] = r"^[0-9]{13}$"

rules[
    "allow_context_correction"
] = False


# ========================================
# Config 저장
# ========================================

CONFIG_PATH.write_text(
    json.dumps(
        rules,
        ensure_ascii=False,
        indent=2,
    ),
    encoding="utf-8",
)


# ========================================
# 결과 출력
# ========================================

print(
    "Day06 OCR Rule Freeze"
)

print(
    "Version:",
    rules["baseline_version"]
)

print(
    "Preprocess:",
    rules["preprocess_mode"]
)

print(
    "PSM:",
    rules["ocr_psm"]
)

print(
    "Whitelist:",
    rules["whitelist"]
)

print(
    "Pattern:",
    rules["expected_pattern"]
)

print(
    "Context Correction:",
    rules["allow_context_correction"]
)

print(
    "Saved:",
    CONFIG_PATH
)
```

실행합니다.

```
python src/44_day06_freeze_ocr_rules.py
```

---

## 30.1 44번 파일은 무엇을 하는 코드인가요?

앞의 실험에서는 아직 OCR 설정이 확정되지 않은 상태였습니다.

```
baseline_version
→ day06-ocr-draft

preprocess_mode
→ NOT_SELECTED
```

43번 OCR 비교 결과:

```
Original
→ 성공

Gray
→ 성공

CLAHE
→ 성공

Otsu
→ 실패

CLAHE + Otsu
→ 실패
```

가 나왔습니다.

따라서 가장 단순하면서도 정확하게 인식된 `original`을 선택하고, 현재 OCR 설정을 **v1.0으로 고정**했습니다.

실제 실행 결과:

```bash
Day06 OCR Rule Freeze

Version: day06-ocr-v1.0

Preprocess: original

PSM: 7

Whitelist: 0123456789

Pattern: ^[0-9]{13}$

Context Correction: False
```

즉 최종 Rule은 다음과 같습니다.

```
전처리
→ original

문자 배치
→ PSM 7
→ 한 줄 문자

허용 문자
→ 0~9 숫자만

SN 형식
→ 정확히 13자리 숫자

자동 문자 보정
→ 사용하지 않음
```

이제 다음 Batch 검사에서는 6장의 제품 이미지에 이 설정을 동일하게 적용합니다.

---

## 30.2 의사코드로 읽어봅니다

```
프로그램 시작

        ↓

기존
day06_ocr_rules.json을 읽는다

        ↓

앞의 OCR 실험 결과를 반영한다

        ↓

Version을

day06-ocr-v1.0

으로 변경한다

        ↓

전처리를

original

로 확정한다

        ↓

PSM = 7

→ 한 줄 문자로 OCR

        ↓

Whitelist = 0123456789

→ 숫자만 읽도록 제한

        ↓

Regex = ^[0-9]{13}$

→ 정확히 13자리 숫자인지 검사

        ↓

Context Correction = False

→ OCR 결과를 임의로 수정하지 않음

        ↓

기존에 저장되어 있던
6장의 ROI 좌표는 그대로 유지한다

        ↓

확정된 설정을

configs/day06_ocr_rules.json

에 다시 저장한다

        ↓

최종 Rule을 터미널에 출력한다

        ↓

프로그램 종료
```

---

# 31. 최종 `day06_ocr_rules.json`을 확인합니다

44번 실행 후 다음 파일이 갱신되었습니다.

```
configs/day06_ocr_rules.json
```

실제 결과에서 중요한 부분은 다음과 같습니다.

```json
{
  "baseline_version": "day06-ocr-v1.0",
  "preprocess_mode": "original",
  "ocr_lang": "eng",
  "ocr_oem": 3,
  "ocr_psm": 7,
  "whitelist": "0123456789",
  "expected_pattern": "^[0-9]{13}$",
  "allow_context_correction": false
}
```

41번에서 저장한 이미지별 ROI 좌표도 그대로 유지되었습니다.

```
roi_by_file
        ↓
0637145699424_side.png
0639873538899_side.png
0679256889404_side.png
0702739449714_side.png
0716954573504_side.png
0793387694868_side.png
```

즉 최종 Config에는 다음 세 종류의 정보가 함께 들어 있습니다.

```
1. 어디를 검사할 것인가?
→ roi_by_file
→ 이미지별 SN 위치

2. 어떻게 OCR할 것인가?
→ preprocess_mode = original
→ PSM = 7
→ Whitelist = 숫자만

3. OCR 결과를 어떻게 검사할 것인가?
→ ^[0-9]{13}$
→ 13자리 숫자 형식 확인
→ 자동 문자 보정 사용 안 함
```

현재까지의 흐름을 연결하면 다음과 같습니다.

```
41번
6장 SN ROI 지정
        ↓
42번
5가지 전처리 이미지 생성
        ↓
43번
실제 OCR 결과 비교
        ↓
Original / Gray / CLAHE 성공
        ↓
25~29번
문자열 검증 Rule 확인
        ↓
44번
최종 OCR Rule 확정
        ↓
day06-ocr-v1.0
        ↓
다음 단계
6장 Batch 검사
```

> PART4에서는 OCR 결과를 단순히 읽는 것에서 끝내지 않고, `13자리 형식 검사 → Expected SN 비교 → OK/NG 판정`으로 연결하고, 이후 모든 이미지에 사용할 OCR 설정을 `day06-ocr-v1.0`으로 고정했습니다.

---

# PART5 — Batch Inspection: 여러 이미지 일괄 검사 + 평가 (32~37)

---

# 32. 한 장의 실험에서 여러 장 검사로 넘어갑니다

지금까지는 대표 이미지:

```
0637145699424_side.png
```

를 중심으로 OCR 설정을 비교했습니다.

43번 실험 결과:

```
Original
→ 성공

Gray
→ 성공

CLAHE
→ 성공

Otsu
→ 실패

CLAHE + Otsu
→ 실패
```

를 확인했고, 44번에서 최종 OCR Rule을 다음과 같이 고정했습니다.

```
Version
→ day06-ocr-v1.0

Preprocess
→ original

PSM
→ 7

Whitelist
→ 0123456789

Regex
→ ^[0-9]{13}$

Context Correction
→ False
```

이제 대표 이미지 한 장이 아니라 **6장의 제품 이미지 전체에 같은 OCR Rule을 적용**합니다.

단, 제품 위치는 이미지마다 다르므로 ROI 좌표까지 동일한 것은 아닙니다.

41번에서 이미 다음 구조로 이미지별 ROI를 저장했습니다.

```
roi_by_file

0637145699424_side.png
→ [434, 294, 529, 318]

0639873538899_side.png
→ [687, 271, 777, 300]

0679256889404_side.png
→ [553, 253, 637, 279]

...
```

따라서 Batch 검사는 다음 흐름으로 진행됩니다.

```
제품 이미지 6장
        ↓
파일명 확인
        ↓
각 이미지에 저장된
SN ROI 좌표 찾기
        ↓
SN 영역 Crop
        ↓
Freeze된 original 사용
        ↓
OCR용 4배 확대
        ↓
Tesseract
PSM 7 + 숫자 Whitelist
        ↓
13자리 숫자 형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG 판정
        ↓
전체 결과 CSV 저장
```

즉:

```
ROI 위치
→ 이미지마다 다름

OCR 방법
→ 모두 동일
```

입니다.

> 이제부터는 특정 이미지 한 장이 잘 읽히지 않는다고 중간에 Rule을 바꾸지 않습니다. 먼저 Freeze한 `day06-ocr-v1.0`으로 6장을 모두 검사하고 결과를 확인합니다.

---

# 33. Batch 검사에 필요한 정답 데이터를 준비합니다

45번 Batch 검사를 실행하기 전에 각 제품 이미지의 **정답 정보(Ground Truth)** 를 먼저 준비합니다.

이를 위해 다음 파일을 만듭니다.

```
configs/day06_labels.csv
```

이 파일은 각 이미지에 대해:

```
이미지 파일명
실제로 인쇄된 SN
생산상 기대하는 SN
실제 제품의 OK / NG
```

를 기록한 **Batch 검사 기준표**입니다.

45번 프로그램은 이 파일을 읽어 OCR 결과가 실제 문자와 같은지, 그리고 제품의 최종 OK / NG 판정이 맞는지를 확인합니다.

---

## 33.1 `day06_labels.csv`를 작성합니다

파일:

```
configs/day06_labels.csv
```

내용:

```
file,actual_text,expected_text,actual_status
0637145699424_side.png,0637145699424,0637145699424,OK
0639873538899_side.png,0639873538899,0639873538899,OK
0679256889404_side.png,0679256889404,0679256889400,NG
0702739449714_side.png,0702739449714,0702739449714,OK
0716954573504_side.png,0716954573504,0716954573500,NG
0793387694868_side.png,0793387694868,0793387694868,OK
```

각 항목의 의미는 다음과 같습니다.

|항목|의미|
|---|---|
|`file`|검사할 이미지 파일명|
|`actual_text`|이미지에 실제로 인쇄되어 있는 SN|
|`expected_text`|생산상 해당 제품에 인쇄되어 있어야 하는 SN|
|`actual_status`|제품의 실제 정답 `OK` / `NG`|

---

## 33.2 일부 이미지는 일부러 NG로 설정합니다

이번 실습에는 실제로 `OK / NG` 판정을 해보기 위해 **일부 제품을 의도적으로 NG 상황으로 설정**합니다.

예를 들어 이 이미지의 실제 SN은:

```
0679256889404
```

입니다.

OCR이 이 값을 정확하게 읽으면:

```
OCR 결과
→ 0679256889404
```

가 됩니다.

그런데 이번 실습에서는 이 제품에 원래 인쇄되어 있어야 하는 값을:

```
Expected SN
→ 0679256889400
```

으로 설정합니다.

즉:

```
실제 인쇄된 값
0679256889404

        ≠

원래 인쇄되어야 하는 값
0679256889400

        ↓

NG
```

로 판단하도록 만든 것입니다.

중요한 점은 **이미지의 숫자를 일부러 바꾼 것은 아니라는 것**입니다.

```
이미지
→ 그대로 사용

Expected SN
→ 일부러 다른 값으로 설정

        ↓

잘못된 SN이 인쇄된 제품 상황을 가정
```

하는 방식입니다.

같은 방법으로:

```
0716954573504_side.png
```

도 NG 제품으로 설정합니다.

따라서 이번 6장의 데이터는:

```
4장
→ 실제 SN = Expected SN
→ OK

2장
→ 실제 SN ≠ Expected SN
→ NG
```

로 구성됩니다.

> 이렇게 일부러 OK와 NG를 섞어 두어야 OCR이 문자를 읽는 것뿐 아니라, Expected SN과 비교해서 제품을 제대로 OK / NG 판정하는지도 확인할 수 있습니다.

---

## 33.3 왜 이 파일이 필요한가요?

OCR 결과만 보면 프로그램이 문자를 제대로 읽었는지만 알 수 있습니다.

예:

```
이미지 실제 SN
0679256889404

OCR 결과
0679256889404
```

이면 OCR은 성공입니다.

```
ocr_exact = True
```

하지만 생산상 기대값이:

```
0679256889400
```

이라면 제품은 정상 제품이 아닙니다.

```
OCR 성공

하지만

Actual SN ≠ Expected SN

        ↓

제품 판정 = NG
```

따라서 `day06_labels.csv`가 있어야 다음 두 가지를 따로 평가할 수 있습니다.

```
OCR 정확도
→ 실제 인쇄된 문자를 제대로 읽었는가?

제품 검사 정확도
→ Expected SN과 비교하여
   OK / NG를 제대로 판정했는가?
```

---

## 33.4 45번 실행 전에 확인합니다

`configs/day06_labels.csv`를 저장한 뒤 터미널에서 확인합니다.

```bash
cat configs/day06_labels.csv
```

다음과 같이 6개의 이미지 정보가 표시되면 준비가 완료된 것입니다.

```
4개
→ OK 제품

2개
→ NG 제품
```

이제 준비된 두 설정 파일을 함께 사용합니다.

```
day06_ocr_rules.json
→ 어떻게 OCR하고 검사할 것인가?

day06_labels.csv
→ 각 제품의 정답은 무엇인가?
```

---

# 34. `45_day06_batch_ocr.py` — 6장의 제품 이미지를 일괄 검사합니다

이제 지금까지 따로 만들었던 기능을 하나의 검사 프로그램으로 연결합니다.

```
이미지
→ ROI
→ 전처리
→ 확대
→ OCR
→ 문자열 Cleaning
→ Regex
→ Expected SN
→ OK / NG
→ CSV
```

파일:

```
src/45_day06_batch_ocr.py
```

작성합니다.

```python
import csv

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    crop_roi,
    get_roi_for_file,
    load_rules,
    preprocess_for_ocr,
    run_ocr,
)

from day06_ocr_validation import (
    inspect_text,
)


# ========================================
# 경로 설정
# ========================================

PRODUCT_DIR = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "product"
)

LABELS_PATH = (
    PROJECT_ROOT
    / "configs"
    / "day06_labels.csv"
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "batch"
)

REPORT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_ocr_results.csv"
)


# ========================================
# OCR Rule 읽기
# ========================================

rules = load_rules()


# ========================================
# Freeze된 Rule인지 확인
# ========================================

if rules[
    "baseline_version"
] != "day06-ocr-v1.0":

    raise RuntimeError(
        "OCR Rule이 Freeze되지 않았습니다."
    )


# ========================================
# Label CSV 존재 확인
# ========================================

if not LABELS_PATH.exists():

    raise FileNotFoundError(
        "Batch 검사에 필요한 "
        "day06_labels.csv가 없습니다.\n"
        f"확인 경로: {LABELS_PATH}"
    )


# ========================================
# 출력 폴더 준비
# ========================================

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# ========================================
# Ground Truth / Expected 값 읽기
# ========================================

with LABELS_PATH.open(
    "r",
    newline="",
    encoding="utf-8-sig",
) as file:

    labels = list(
        csv.DictReader(file)
    )


if not labels:

    raise RuntimeError(
        "day06_labels.csv에 "
        "검사 데이터가 없습니다."
    )


rows = []


# ========================================
# 제품 이미지 하나씩 검사
# ========================================

for item in labels:

    file_name = item[
        "file"
    ]

    image_path = (
        PRODUCT_DIR
        / file_name
    )


    # ------------------------------------
    # 이미지 읽기
    # ------------------------------------

    image = cv2.imread(
        str(image_path)
    )

    if image is None:

        raise FileNotFoundError(
            f"제품 이미지를 찾을 수 없습니다: "
            f"{image_path}"
        )


    # ------------------------------------
    # 이미지별 ROI 좌표 가져오기
    # ------------------------------------

    roi_coordinates = (
        get_roi_for_file(
            rules,
            file_name,
        )
    )


    # ------------------------------------
    # SN 영역 Crop
    # ------------------------------------

    roi = crop_roi(
        image,
        roi_coordinates,
    )


    # ------------------------------------
    # Freeze된 전처리 적용
    # 현재: original
    # ------------------------------------

    processed = (
        preprocess_for_ocr(
            roi,
            rules,
        )
    )


    # ------------------------------------
    # 43번과 동일하게 4배 확대
    # ------------------------------------

    ocr_input = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )


    # ------------------------------------
    # Tesseract OCR
    # ------------------------------------

    raw_text = run_ocr(
        ocr_input,
        rules,
    )


    # ------------------------------------
    # 문자열 검사 + 제품 판정
    # ------------------------------------

    result = inspect_text(
        raw_text=raw_text,

        expected_text=item[
            "expected_text"
        ],

        pattern=rules[
            "expected_pattern"
        ],
    )


    # ------------------------------------
    # OCR 자체의 정확도 확인
    # ------------------------------------

    ocr_exact = (
        result["final_text"]
        == item["actual_text"]
    )


    # ------------------------------------
    # 최종 제품 판정 정확도 확인
    # ------------------------------------

    inspection_correct = (
        result[
            "predicted_status"
        ]
        == item[
            "actual_status"
        ]
    )


    # ------------------------------------
    # 실제 OCR 입력 이미지 저장
    # ------------------------------------

    output_image_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_ocr_input.jpg"
    )

    cv2.imwrite(
        str(output_image_path),
        ocr_input,
    )


    # ------------------------------------
    # 결과 저장
    # ------------------------------------

    row = {
        "file":
            file_name,

        "actual_text":
            item["actual_text"],

        "expected_text":
            item["expected_text"],

        "actual_status":
            item["actual_status"],

        "raw_text":
            repr(
                result["raw_text"]
            ),

        "cleaned_text":
            result["cleaned_text"],

        "final_text":
            result["final_text"],

        "format_ok":
            result["format_ok"],

        "expected_match":
            result["expected_match"],

        "ocr_exact":
            ocr_exact,

        "predicted_status":
            result["predicted_status"],

        "inspection_correct":
            inspection_correct,

        "reasons":
            "|".join(
                result["reasons"]
            ),
    }

    rows.append(
        row
    )


    # ------------------------------------
    # 터미널 출력
    # ------------------------------------

    print(
        "=" * 70
    )

    print(
        "File:",
        file_name
    )

    print(
        "Actual:",
        item["actual_text"]
    )

    print(
        "Expected:",
        item["expected_text"]
    )

    print(
        "OCR:",
        result["final_text"]
    )

    print(
        "Format OK:",
        result["format_ok"]
    )

    print(
        "Expected Match:",
        result["expected_match"]
    )

    print(
        "OCR Exact:",
        ocr_exact
    )

    print(
        "Predicted:",
        result["predicted_status"]
    )

    print(
        "Actual Status:",
        item["actual_status"]
    )

    print(
        "Inspection Correct:",
        inspection_correct
    )

    print(
        "Reasons:",
        result["reasons"]
    )


# ========================================
# Batch 결과 CSV 저장
# ========================================

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


fieldnames = [
    "file",
    "actual_text",
    "expected_text",
    "actual_status",
    "raw_text",
    "cleaned_text",
    "final_text",
    "format_ok",
    "expected_match",
    "ocr_exact",
    "predicted_status",
    "inspection_correct",
    "reasons",
]


with REPORT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=fieldnames,
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


print()

print(
    "Saved:",
    REPORT_PATH
)

print(
    "Day06 Batch OCR Complete"
)
```

실행합니다.

```
python src/45_day06_batch_ocr.py
```

### 의사코드

```
프로그램 시작

        ↓

OCR 공통 함수와
문자열 검사 함수를 불러온다

        ↓

day06_ocr_rules.json을 읽는다

        ↓

Version이
day06-ocr-v1.0인지 확인한다

        ↓

day06_labels.csv가
존재하는지 확인한다

        ↓

CSV에서
6개의 검사 정보를 읽는다

        ↓

첫 번째 제품 이미지를 읽는다

        ↓

파일명으로
roi_by_file에서
ROI 좌표를 찾는다

        ↓

SN 영역을 Crop한다

        ↓

Freeze된 전처리
original을 적용한다

        ↓

4배 확대한다

        ↓

Tesseract로 숫자를 읽는다

        ↓

Raw OCR 문자열을 Cleaning한다

        ↓

13자리 숫자인지 검사한다

        ↓

Expected SN과 비교한다

        ↓

OK / NG를 판정한다

        ↓

OCR이 실제 인쇄값과 같은지 확인한다

→ ocr_exact

        ↓

OK / NG 판정이 실제 상태와 같은지 확인한다

→ inspection_correct

        ↓

OCR 입력 이미지와
검사 결과를 저장한다

        ↓

다음 제품으로 이동한다

        ↓

6장 모두 반복한다

        ↓

전체 결과를

reports/day06_ocr_results.csv

에 저장한다

        ↓

프로그램 종료
```

---

## 34.1 `45_day06_batch_ocr.py`는 무엇을 확인하는 코드인가요?

45번은 앞에서 만든 OCR Rule을 6장의 제품 이미지에 반복 적용하여 실제 Batch 검사가 가능한지 확인하는 프로그램입니다.

각 이미지에 대해 다음 과정을 수행했습니다.

```
제품 이미지 읽기
        ↓
파일명으로 해당 ROI 좌표 찾기
        ↓
SN 영역 Crop
        ↓
Freeze된 original 적용
        ↓
4배 확대
        ↓
Tesseract OCR
        ↓
문자열 Cleaning
        ↓
13자리 숫자 형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG 판정
```

그리고 이 과정을 6장 모두에 반복했습니다.

실제 실행 결과 모든 이미지에서:

```
OCR Exact = True
```

가 나왔습니다.

즉 6장의 SN을 모두 실제 인쇄값과 동일하게 읽었습니다.

```
6장 검사
        ↓
6장 모두 OCR 성공
        ↓
OCR Exact
6 / 6
```

또한 최종 제품 판정도 모두:

```
Inspection Correct = True
```

였습니다.

즉:

> 대표 이미지에서 만든 `day06-ocr-v1.0` Rule이 이번 6장의 Batch 데이터에서도 정상적으로 동작한 것을 확인했습니다.

---

## 34.2 왜 Batch에서도 4배 확대를 사용했나요?

43번에서 대표 이미지의 OCR 성능을 비교할 때 작은 SN ROI를 4배 확대해서 Tesseract에 입력했습니다.

```
SN ROI
        ↓
4배 확대
        ↓
Tesseract OCR
```

45번 Batch에서도 같은 조건을 그대로 사용했습니다.

```
43번 대표 이미지 실험
4배 확대

        =

45번 Batch 검사
4배 확대
```

실제 Batch 결과에서는 6장 모두 SN을 정확하게 읽었습니다.

```
OCR Exact
→ 6장 모두 True
```

따라서 현재 데이터에서는 4배 확대를 포함한 OCR 입력 조건이 안정적으로 동작한 것을 확인할 수 있습니다.

> 실험에서 결정한 조건과 Batch의 실제 검사 조건을 동일하게 유지해야 어떤 Rule이 효과가 있었는지 올바르게 판단할 수 있습니다.

---

## 34.4 실제 Batch 결과를 확인합니다

45번 실행 결과 6장의 검사 결과는 다음과 같습니다.

|이미지|OCR 결과|Expected Match|판정|검사 결과|
|---|---|---|---|---|
|`0637145699424_side.png`|`0637145699424`|True|OK|정확|
|`0639873538899_side.png`|`0639873538899`|True|OK|정확|
|`0679256889404_side.png`|`0679256889404`|False|NG|정확|
|`0702739449714_side.png`|`0702739449714`|True|OK|정확|
|`0716954573504_side.png`|`0716954573504`|False|NG|정확|
|`0793387694868_side.png`|`0793387694868`|True|OK|정확|

먼저 OCR 결과를 확인하면:

```
6장 모두

Format OK = True
OCR Exact = True
```

였습니다.

즉 모든 이미지에서:

```
13자리 숫자 형식 정상
        +
실제 인쇄된 SN을 정확하게 OCR
```

했습니다.

---

### OK 제품 4장

다음 4장은 실제 인쇄값과 Expected SN이 같습니다.

```
0637145699424
0639873538899
0702739449714
0793387694868
```

따라서:

```
Actual SN
=
Expected SN

        ↓

Expected Match = True

        ↓

Predicted = OK

        ↓

Actual Status = OK

        ↓

Inspection Correct = True
```

로 정상 판정되었습니다.

---

### NG 제품 2장

다음 두 장은 OCR 자체는 정확했지만 Expected SN과 값이 다릅니다.

첫 번째:

```
Actual SN
0679256889404

Expected SN
0679256889400
```

결과:

```
OCR Exact = True

Expected Match = False

Predicted = NG

Actual Status = NG

Inspection Correct = True

Reasons
→ NG_EXPECTED_MISMATCH
```

두 번째:

```
Actual SN
0716954573504

Expected SN
0716954573500
```

역시:

```
OCR Exact = True

Expected Match = False

Predicted = NG

Actual Status = NG

Inspection Correct = True

Reasons
→ NG_EXPECTED_MISMATCH
```

가 나왔습니다.

즉 두 제품 모두 OCR 실패 때문에 NG가 된 것이 아니라, 실제 인쇄된 SN과 Expected SN이 다르기 때문에 정확하게 NG로 검출된 것입니다.

---

### Batch 결과 파일도 저장되었습니다

전체 검사 결과는 다음 파일에 저장되었습니다.

```
reports/day06_ocr_results.csv
```

또한 Tesseract에 실제 입력한 확대 이미지는:

```
outputs/day06/batch/
```

에 저장되어 있습니다.

45번 Batch 검사에서 실제 Tesseract에 넣기 직전에 4배 확대해서 저장한 OCR 입력 이미지
![[{227841EB-8E67-43AA-8D36-C81FE4A41380}.png]]

코드에서는 이부분에 해당됩니다.

```python
ocr_input = cv2.resize(
    processed,
    None,
    fx=4.0,
    fy=4.0,
    interpolation=cv2.INTER_CUBIC,
)
```

따라서 나중에 OCR 실패가 발생하면:

```
CSV에서 실패 이미지 확인
        ↓
outputs/day06/batch의
ocr_input 이미지 확인
        ↓
ROI / 문자 품질 / OCR 문제 분석
```

순서로 원인을 확인할 수 있습니다.

---

# 35. OCR 정확도와 제품 검사 정확도를 구분합니다

이번 Batch 결과에서 중요한 점은 OCR 성공과 제품 OK가 같은 의미가 아니라는 것입니다.

두 값을 구분해서 봅니다.

```
ocr_exact
→ OCR이 실제 인쇄된 SN을
   정확하게 읽었는가?

inspection_correct
→ 최종 OK / NG 판정이
   실제 정답과 일치했는가?
```

이번 6장의 결과에서는:

```
OCR Exact
→ 6장 모두 True

Inspection Correct
→ 6장 모두 True
```

였습니다.

---

## OK 제품에서는

예를 들어:

```
Actual SN
0637145699424

Expected SN
0637145699424

OCR
0637145699424
```

입니다.

따라서:

```
OCR 정확
        ↓
Expected SN과도 일치
        ↓
Predicted = OK
        ↓
실제 정답도 OK
```

이므로:

```
ocr_exact = True
inspection_correct = True
```

입니다.

---

## NG 제품에서는

다음 이미지의 경우:

```
Actual SN
0679256889404

Expected SN
0679256889400

OCR
0679256889404
```

OCR은 실제 문자를 정확하게 읽었습니다.

```
ocr_exact = True
```

하지만:

```
0679256889404
        ≠
0679256889400
```

이므로 제품은 `NG`입니다.

```
Expected Match = False
        ↓
Predicted = NG
```

실제 정답도 `NG`이므로:

```
inspection_correct = True
```

입니다.

즉 이번 결과는 다음과 같습니다.

```
OCR
→ 실제 인쇄값을 정확히 읽음

        ↓

Expected SN 비교
→ 값이 다름

        ↓

제품 NG 판정

        ↓

실제 정답도 NG

        ↓

검사 성공
```

> OCR은 “무슨 글자가 인쇄되어 있는가?”를 확인하고, 제조검사는 그 OCR 결과가 “원래 인쇄되어야 할 값과 같은가?”까지 확인합니다.

따라서 이번 Batch에서는 4장의 OK 제품과 2장의 NG 제품을 모두 정확하게 구분했습니다.

```
6장 Batch 검사
        ↓
OCR 정확
6 / 6

        +

제품 판정 정확
6 / 6
```

다음 `46_day06_evaluate.py`에서는 이 결과를 다시 읽어 OCR Exact Accuracy와 Inspection Accuracy를 수치로 계산하게 됩니다.

---

# 36. `46_day06_evaluate.py` — Batch 전체 결과를 평가합니다

45번은 이미지 하나하나의 결과를 저장합니다.

46번에서는 그 CSV를 다시 읽어 전체 6장의 성능을 요약합니다.

```
45번
개별 제품 검사 결과

        ↓

day06_ocr_results.csv

        ↓

46번
전체 결과 집계

        ↓

OCR Accuracy
Inspection Accuracy
실패 이미지 확인
```

파일:

```
src/46_day06_evaluate.py
```

작성합니다.

```python
import csv

from day06_ocr_utils import (
    PROJECT_ROOT,
)


# ========================================
# 경로 설정
# ========================================

RESULT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_ocr_results.csv"
)

METRIC_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_ocr_metrics.md"
)


# ========================================
# Batch 결과 파일 확인
# ========================================

if not RESULT_PATH.exists():

    raise FileNotFoundError(
        "먼저 45_day06_batch_ocr.py를 "
        "실행해야 합니다.\n"
        f"결과 파일이 없습니다: "
        f"{RESULT_PATH}"
    )


# ========================================
# CSV 읽기
# ========================================

with RESULT_PATH.open(
    "r",
    newline="",
    encoding="utf-8-sig",
) as file:

    rows = list(
        csv.DictReader(file)
    )


if not rows:

    raise RuntimeError(
        "day06_ocr_results.csv에 "
        "결과가 없습니다."
    )


# ========================================
# 문자열 True / False 변환
# ========================================

def as_bool(
    value,
):
    return (
        str(value)
        .strip()
        .lower()
        == "true"
    )


# ========================================
# 기본 개수 계산
# ========================================

count = len(
    rows
)


ocr_correct = sum(
    as_bool(
        row["ocr_exact"]
    )
    for row in rows
)


inspection_correct_count = sum(
    as_bool(
        row["inspection_correct"]
    )
    for row in rows
)


format_ok_count = sum(
    as_bool(
        row["format_ok"]
    )
    for row in rows
)


expected_match_count = sum(
    as_bool(
        row["expected_match"]
    )
    for row in rows
)


# ========================================
# 정확도 계산
# ========================================

ocr_accuracy = (
    ocr_correct
    / count
)


inspection_accuracy = (
    inspection_correct_count
    / count
)


# ========================================
# 실패 이미지 찾기
# ========================================

ocr_failed_files = [
    row["file"]
    for row in rows
    if not as_bool(
        row["ocr_exact"]
    )
]


inspection_failed_files = [
    row["file"]
    for row in rows
    if not as_bool(
        row["inspection_correct"]
    )
]


# ========================================
# Markdown Report 작성
# ========================================

ocr_failure_text = (
    "\n".join(
        "- " + name
        for name in ocr_failed_files
    )
    if ocr_failed_files
    else "- 없음"
)


inspection_failure_text = (
    "\n".join(
        "- " + name
        for name in inspection_failed_files
    )
    if inspection_failed_files
    else "- 없음"
)


text = f"""# Day 06 OCR Metrics

## Dataset

Images: {count}

## OCR Exact Accuracy

{ocr_accuracy:.4f}

OCR Correct: {ocr_correct} / {count}

## Inspection Accuracy

{inspection_accuracy:.4f}

Inspection Correct: {inspection_correct_count} / {count}

## Format

Format OK: {format_ok_count} / {count}

## Expected Match

Expected Match: {expected_match_count} / {count}

## OCR Failures

{ocr_failure_text}

## Inspection Failures

{inspection_failure_text}
"""


# ========================================
# Report 저장
# ========================================

METRIC_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


METRIC_PATH.write_text(
    text,
    encoding="utf-8",
)


print(
    text
)

print(
    "Saved:",
    METRIC_PATH
)
```

실행합니다.

```
python src/46_day06_evaluate.py
```

### 의사코드

```
프로그램 시작

        ↓

45번에서 만든

day06_ocr_results.csv

가 있는지 확인한다

        ↓

CSV의 6개 검사 결과를 읽는다

        ↓

ocr_exact = True인
이미지 수를 센다

        ↓

OCR Exact Accuracy를 계산한다

        ↓

inspection_correct = True인
이미지 수를 센다

        ↓

Inspection Accuracy를 계산한다

        ↓

13자리 형식에 성공한
이미지 수를 센다

        ↓

Expected SN과 일치한
이미지 수를 센다

        ↓

OCR 실패 이미지 목록을 찾는다

        ↓

제품 판정 실패 이미지 목록을 찾는다

        ↓

전체 결과를

reports/day06_ocr_metrics.md

에 저장한다

        ↓

프로그램 종료
```

---

## 36.1 46번 실행 결과를 확인합니다

`46_day06_evaluate.py`는 45번에서 만든:

```
reports/day06_ocr_results.csv
```

를 읽어 6장의 Batch 검사 결과를 하나의 성능 지표로 정리합니다.

실제 실행 결과는 다음과 같습니다.

```
Images: 6

OCR Exact Accuracy
1.0000

OCR Correct
6 / 6

Inspection Accuracy
1.0000

Inspection Correct
6 / 6

Format OK
6 / 6

Expected Match
4 / 6

OCR Failures
없음

Inspection Failures
없음
```

결과 보고서는 다음 파일에 저장되었습니다.

```
reports/day06_ocr_metrics.md
```

---

## 36.2 결과를 어떻게 해석할까요?

### OCR Exact Accuracy = 1.0000

```
OCR Correct
6 / 6
```

6장의 이미지에서 Tesseract가 실제 인쇄된 13자리 SN을 모두 정확하게 읽었다는 의미입니다.

```
6장 입력
        ↓
6장 모두 실제 SN과 OCR 결과 일치
        ↓
OCR Exact Accuracy
= 100%
```

---

### Inspection Accuracy = 1.0000

```
Inspection Correct
6 / 6
```

4장의 OK 제품과 2장의 NG 제품을 프로그램이 모두 올바르게 판정했습니다.

```
실제 OK 4장
→ 모두 OK 판정

실제 NG 2장
→ 모두 NG 판정

        ↓

Inspection Accuracy
= 100%
```

---

### Format OK = 6 / 6

모든 OCR 결과가:

```
^[0-9]{13}$
```

규칙을 만족했습니다.

---
참고] 
```
정규식이란?
문자열이 내가 정한 형식에 맞는지 검사하는 규칙 표현식입니다.
```

위의 정규식 의미는

```
^
→ 문자열의 시작

[0-9]
→ 숫자 0~9 중 하나

{13}
→ 앞의 숫자가 정확히 13개

$
→ 문자열의 끝
```

즉 뜻은 처음부터 끝까지 숫자로만 되어 있고 정확히 13자리인가? 라는 뜻입니다. 참고만 하세요.

정규식은 종류가 많고 복잡해질 수 있어서, 문법을 암기하는 것보다:

```
이게 정규식이구나
→ 어떤 형식을 검사하려는 거구나
→ 필요하면 검색하거나 생성형 AI로 확인하면 된다
```

정도로 이해하는 게 현실적입니다.

---
즉 6장 모두:

```
숫자만 사용
+
정확히 13자리
```

형식으로 읽혔습니다.

---

## 36.3 `Expected Match = 4 / 6`인데 왜 실패가 아닌가요?

이 부분이 이번 결과에서 가장 중요합니다.

결과에는:

```
Expected Match
4 / 6
```

이라고 나왔습니다.

처음 보면:

```
6장 중 2장이 틀린 것인가?
```

라고 생각할 수 있지만 그렇지 않습니다.

이번 실습에서는 처음부터:

```
4장
→ Actual SN = Expected SN
→ OK

2장
→ Actual SN ≠ Expected SN
→ NG
```

로 정답 데이터를 구성했습니다.

따라서 Expected Match 결과는 정확히:

```
OK 제품 4장
→ Expected Match = True

NG 제품 2장
→ Expected Match = False
```

가 되어야 합니다.

실제 결과도:

```
Expected Match
4 / 6
```

이 나왔으므로 의도한 검사 결과가 정확하게 나온 것입니다.

즉:

```
Expected Match 4 / 6
        ≠
정확도 66.7%

Expected Match 4 / 6
        =
4장은 기대값과 일치하는 OK 제품
2장은 기대값과 다른 NG 제품
```

입니다.

그리고 두 NG 제품도 정확하게 `NG`로 판정했기 때문에:

```
Inspection Accuracy
= 6 / 6
= 1.0000
```

이 됩니다.

> Expected Match는 정확도 지표가 아니라, 실제 인쇄된 SN이 Expected SN과 같은 제품이 몇 장인지 보여주는 값입니다.

---

# 37. PART5 Batch 검사 결과를 최종 정리합니다

이번 Batch 검사에서는 Freeze한:

```
day06-ocr-v1.0
```

Rule을 6장의 제품 이미지에 적용했습니다.

전체 결과는 다음과 같습니다.

|확인 항목|실제 결과|해석|
|---|---|---|
|검사 이미지|6장|Batch 대상|
|OCR Correct|6 / 6|실제 SN을 모두 정확히 읽음|
|OCR Exact Accuracy|1.0000|OCR 정확도 100%|
|Format OK|6 / 6|모두 13자리 숫자 형식 정상|
|Expected Match|4 / 6|OK 4장 일치, NG 2장 불일치|
|Inspection Correct|6 / 6|OK / NG 모두 정확히 판정|
|Inspection Accuracy|1.0000|제품 검사 정확도 100%|
|OCR Failure|없음|OCR 오인식 없음|
|Inspection Failure|없음|최종 판정 오류 없음|

이번 결과를 하나의 흐름으로 정리하면:

```
제품 이미지 6장
        ↓
각 이미지의 SN ROI
        ↓
original
        ↓
4배 확대
        ↓
Tesseract OCR
        ↓
6장 모두 실제 SN 정확히 인식
        ↓
13자리 Regex
6 / 6 정상
        ↓
Expected SN 비교
        ↓
4장 → 일치 → OK
2장 → 불일치 → NG
        ↓
실제 정답과 비교
        ↓
6 / 6 정확
```

따라서 이번 Batch에서는:

```
OCR Exact Accuracy
= 100%

Inspection Accuracy
= 100%
```

를 확인했습니다.

---

## PART5에서 확인한 핵심

45번과 46번을 통해 한 장에서 만든 OCR Rule을 여러 이미지에 동일하게 적용하고, 그 결과를 수치로 평가하는 과정까지 연결했습니다.

```
day06_ocr_rules.json
Freeze된 OCR Rule

        +

day06_labels.csv
정답 데이터

        ↓

45_day06_batch_ocr.py
6장 일괄 검사

        ↓

day06_ocr_results.csv
개별 검사 결과

        ↓

46_day06_evaluate.py
전체 결과 집계

        ↓

OCR Exact Accuracy
1.0000

Inspection Accuracy
1.0000
```

여기서 가장 중요한 것은:

```
OCR이 정확하다
        ≠
제품이 반드시 OK다
```

라는 점입니다.

이번 NG 2장은 OCR도 정확했고 13자리 형식도 정상이었지만:

```
Actual SN
≠
Expected SN
```

이었기 때문에 정확하게 `NG`로 판정되었습니다.

> PART5에서는 `한 장의 OCR 성공 → 여러 장 Batch 검사 → OCR 정확도와 제품 검사 정확도 평가`까지 완료했습니다.

---

# PART6 — 환경 변화 테스트: OCR 검사기의 한계 확인 (38~45)

---

# 38. 이번 파트에서는 무엇을 하나요?

PART5까지 정상 이미지 6장을 검사하여 다음 결과를 확인했습니다.

```
OCR Exact Accuracy
→ 6 / 6
→ 100%

Inspection Accuracy
→ 6 / 6
→ 100%
```

즉 정상적인 이미지에서는 현재 OCR Rule:

```
day06-ocr-v1.0
```

이 잘 동작했습니다.

이제 새로운 OCR을 만드는 것이 아니라 정상 이미지를 일부러 나쁜 상태로 변형하여 현재 검사기가 어디까지 버티는지 확인합니다.

```
정상 SN ROI

0637145699424

        ↓
프로그램으로 자동 변형

Low Contrast
Blur
Tilt
Glare

        ↓
현재 OCR Rule 그대로 적용

        ↓
성공 / 실패 확인
```

> 이번 실험의 목적은 정확도를 다시 100%로 만드는 것이 아니라 현재 OCR 검사기가 어떤 영상 조건에 약한지 확인하는 것입니다.

---

# 39. 새로운 데이터를 준비하지 않습니다

41번에서 이미 만든 대표 SN ROI를 사용합니다.

현재 대표 이미지:

```
0637145699424_side.png
```

대표 SN ROI:

```
outputs/day06/roi/
0637145699424_side_sn_roi.jpg
```

이 정상 ROI 한 장을 프로그램이 자동으로 변형합니다.

```
정상 ROI
0637145699424

        ├── Low Contrast
        │   → 문자와 배경 차이를 약하게 만듦
        │
        ├── Blur
        │   → 문자를 흐리게 만듦
        │
        ├── Tilt
        │   → 문자를 +8° 기울임
        │
        └── Glare
            → 빛 반사처럼 일부를 밝게 만듦
```

---

# 40. `47_day06_make_challenge_variants.py` — 환경 변화 이미지를 자동으로 만듭니다

기존 파일 번호를 유지하기 위해 파일명은 그대로 사용합니다.

파일:

```
src/47_day06_make_challenge_variants.py
```

이 프로그램은 대표 SN ROI 한 장에서 테스트용 이미지 4장을 자동으로 만듭니다.

작성합니다.

```python
from pathlib import Path

import cv2
import numpy as np

from day06_ocr_utils import (
    PROJECT_ROOT,
    load_rules,
)


# ========================================
# 대표 SN ROI 찾기
# ========================================

rules = load_rules()

reference_file = rules[
    "reference_file"
]

reference_stem = (
    Path(reference_file)
    .stem
)

SOURCE_PATH = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "roi"
    / f"{reference_stem}_sn_roi.jpg"
)

ROOT = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "challenge"
)


# ========================================
# 정상 SN ROI 읽기
# ========================================

image = cv2.imread(
    str(SOURCE_PATH)
)

if image is None:
    raise FileNotFoundError(
        SOURCE_PATH
    )


height, width = (
    image.shape[:2]
)


# ========================================
# 1. Low Contrast
# 문자와 배경 차이를 약하게 만듦
# ========================================

neutral = np.full_like(
    image,
    128,
)

low_contrast = cv2.addWeighted(
    image,
    0.45,
    neutral,
    0.55,
    0,
)


# ========================================
# 2. Blur
# 문자를 흐리게 만듦
# ========================================

blur = cv2.GaussianBlur(
    image,
    (9, 9),
    0,
)


# ========================================
# 3. Tilt
# 문자를 +8도 기울임
# ========================================

center = (
    width / 2,
    height / 2,
)

matrix = (
    cv2.getRotationMatrix2D(
        center,
        8.0,
        1.0,
    )
)

tilt = cv2.warpAffine(
    image,
    matrix,
    (width, height),
    flags=cv2.INTER_LINEAR,
    borderMode=cv2.BORDER_CONSTANT,
    borderValue=(255, 255, 255),
)


# ========================================
# 4. Glare
# 빛 반사 영역을 인위적으로 추가
# ========================================

overlay = image.copy()

cv2.ellipse(
    overlay,
    (
        int(width * 0.65),
        int(height * 0.50),
    ),
    (
        max(
            10,
            int(width * 0.24),
        ),
        max(
            8,
            int(height * 0.20),
        ),
    ),
    -20,
    0,
    360,
    (255, 255, 255),
    -1,
)

glare = cv2.addWeighted(
    overlay,
    0.60,
    image,
    0.40,
    0,
)


# ========================================
# 테스트 이미지 저장
# ========================================

variants = {
    "low_contrast":
        low_contrast,

    "blur":
        blur,

    "tilt":
        tilt,

    "glare":
        glare,
}


for condition, result in (
    variants.items()
):

    output_dir = (
        ROOT
        / condition
    )

    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )

    output_path = (
        output_dir
        / f"{condition}_01.jpg"
    )

    cv2.imwrite(
        str(output_path),
        result,
    )

    print(
        "Saved:",
        output_path
    )


print()
print(
    "Environment Test Images Created"
)
```

실행합니다.

```
python src/47_day06_make_challenge_variants.py
```

---

## 40.1 이 코드는 무엇을 하는 코드인가요?

기존 정상 SN:

```
0637145699424
```

한 장을 복사하여 이미지 상태만 다르게 만든 것입니다.

```
정상 SN ROI
        ↓
4개의 복사본 생성
        ↓
각 복사본의 영상 상태 변경
        ↓
테스트 데이터 저장
```

실행 후 다음 파일이 생성됩니다.

```
samples/day06/challenge/
│
├── low_contrast/
│   └── low_contrast_01.jpg
│
├── blur/
│   └── blur_01.jpg
│
├── tilt/
│   └── tilt_01.jpg
│
└── glare/
    └── glare_01.jpg
```

![[{BD618595-7D12-44FA-841A-C80F7258F608}.png]]

---

## 40.2 의사코드

```
프로그램 시작

        ↓

day06_ocr_rules.json에서
대표 이미지 파일명을 확인한다

        ↓

대표 이미지에서 만든
정상 SN ROI를 읽는다

0637145699424

        ↓

같은 이미지를 이용하여

Low Contrast
Blur
Tilt
Glare

4가지 상태를 만든다

        ↓

각 이미지를

samples/day06/challenge/

폴더에 저장한다

        ↓

프로그램 종료
```

---

# 41. 만들어진 환경 변화 이미지를 확인합니다

`47_day06_make_challenge_variants.py`를 실행하여 정상 SN:

```
0637145699424
```

를 기준으로 4가지 환경 변화 이미지가 생성되었습니다.

실제 결과를 확인하면 다음과 같습니다.

|테스트 조건|실제 생성 결과|
|---|---|
|`low_contrast`|문자와 배경의 밝기 차이가 줄어들어 숫자가 전체적으로 옅어짐|
|`blur`|숫자의 획과 경계가 흐려져 문자 구분이 어려워짐|
|`tilt`|SN 전체가 약 +8° 기울어지고 회전에 따른 흰색 여백이 생김|
|`glare`|밝은 반사 영역이 숫자 일부를 직접 가려 문자 정보가 손실됨|

실제 이미지를 보면 변화의 정도도 서로 다릅니다.

```
Low Contrast
→ 숫자는 모두 남아 있지만
   전체적으로 흐릿하고 옅어짐

Blur
→ 숫자는 보이지만
   획의 경계가 많이 퍼짐

Tilt
→ 숫자 정보는 남아 있지만
   한 줄이 기울어진 상태

Glare
→ 일부 숫자가 밝은 영역에 가려져
   실제 문자 정보 일부가 보이지 않음
```

여기서 중요한 점은 네 이미지 모두 같은 문제가 아니라는 것입니다.

```
Low Contrast / Blur / Tilt
→ 문자 정보는 대체로 남아 있음
→ OCR이 버틸 가능성이 있음

Glare
→ 문자 일부가 실제로 가려짐
→ OCR이 복구하기 어려울 가능성이 큼
```

하지만 아직 성공과 실패를 눈으로 결정하지 않습니다.

다음 `48_day06_challenge.py`에서 현재 Freeze된:

```
day06-ocr-v1.0
```

을 그대로 적용하여 실제 OCR 결과를 확인합니다.

```
환경 변화 이미지
        ↓
같은 OCR Rule 적용
        ↓
0637145699424를 정확히 읽는가?
        ↓
성공 / 실패 확인
```

> 이 단계에서는 환경 변화 이미지가 의도대로 만들어졌는지만 확인하고, 실제 OCR 강건성은 다음 실험 결과로 판단합니다.

---

# 42. 48번 테스트 전에 Tilt 보정 함수를 하나 추가합니다

47번에서 다음과 같이 `+8°` 기울어진 SN 이미지를 만들었습니다.

```
정상 SN
        ↓
+8° Tilt
        ↓
기울어진 SN
```

48번에서는 먼저 이 기울어진 이미지를 그대로 OCR해 보고, 추가로 다시 `-8°` 돌려서 OCR 결과가 좋아지는지도 비교합니다.

```
Tilt 이미지
        ↓
그대로 OCR
        ↓
결과 확인

        +

Tilt 이미지
        ↓
-8° 회전
        ↓
수평에 가깝게 복원
        ↓
OCR
        ↓
결과 비교
```

이처럼 기울어진 문자를 다시 수평에 가깝게 펴는 작업을 Deskew라고 합니다.

이를 위해 공통 함수 파일:

```
src/day06_ocr_utils.py
```

맨 아래에 다음 함수를 추가합니다.

```python
def rotate_image(
    image,
    angle,
):
    height, width = (
        image.shape[:2]
    )

    center = (
        width / 2,
        height / 2,
    )

    matrix = (
        cv2.getRotationMatrix2D(
            center,
            angle,
            1.0,
        )
    )

    if image.ndim == 2:
        border_value = 255
    else:
        border_value = (
            255,
            255,
            255,
        )

    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        flags=cv2.INTER_LINEAR,
        borderMode=cv2.BORDER_CONSTANT,
        borderValue=border_value,
    )
```

이번 테스트에서는 우리가 직접 `+8°` 기울여 만들었으므로:

```
+8° Tilt
        ↓
-8° Deskew
```

로 되돌려 봅니다.

> 이 함수는 새로운 OCR 기능을 만드는 것이 아니라, 48번에서 Tilt 이미지의 OCR 결과를 “보정 전 / 보정 후”로 비교하기 위해 추가하는 작은 보조 함수입니다.

---

# 43. `48_day06_challenge.py` — 현재 OCR Rule로 환경 변화 이미지를 검사합니다

이제 4개의 테스트 이미지에 **이미 Freeze한 `day06-ocr-v1.0`을 그대로 적용**합니다.

새로운 Rule을 만들지 않습니다.

파일:

```
src/48_day06_challenge.py
```

작성합니다.

```python
import csv
from pathlib import Path

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    load_rules,
    preprocess_for_ocr,
    rotate_image,
    run_ocr,
)

from day06_ocr_validation import (
    inspect_text,
)


# ========================================
# 경로 설정
# ========================================

ROOT = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "challenge"
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "challenge"
)

REPORT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_challenge_results.csv"
)


# ========================================
# Freeze된 OCR Rule 읽기
# ========================================

rules = load_rules()


if rules[
    "baseline_version"
] != "day06-ocr-v1.0":

    raise RuntimeError(
        "OCR Rule이 Freeze되지 않았습니다."
    )


# ========================================
# 대표 SN 정답 가져오기
# ========================================

reference_file = rules[
    "reference_file"
]

EXPECTED_SN = (
    Path(reference_file)
    .stem
    .split("_side")[0]
)


OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


rows = []


# ========================================
# 4가지 환경 변화 검사
# ========================================

for condition in [
    "low_contrast",
    "blur",
    "tilt",
    "glare",
]:

    input_dir = (
        ROOT
        / condition
    )

    for image_path in sorted(
        input_dir.glob("*.jpg")
    ):

        roi = cv2.imread(
            str(image_path)
        )

        if roi is None:
            continue


        # ==================================
        # Freeze된 전처리 적용
        # 현재: original
        # ==================================

        processed = (
            preprocess_for_ocr(
                roi,
                rules,
            )
        )


        # ==================================
        # Batch와 동일하게 4배 확대
        # ==================================

        ocr_input = cv2.resize(
            processed,
            None,
            fx=4.0,
            fy=4.0,
            interpolation=cv2.INTER_CUBIC,
        )


        # ==================================
        # OCR
        # ==================================

        raw_text = run_ocr(
            ocr_input,
            rules,
        )


        # ==================================
        # 문자열 검사
        # ==================================

        result = inspect_text(
            raw_text=raw_text,
            expected_text=EXPECTED_SN,
            pattern=rules[
                "expected_pattern"
            ],
        )


        ocr_exact = (
            result["final_text"]
            == EXPECTED_SN
        )


        # ==================================
        # 실제 OCR 입력 이미지 저장
        # ==================================

        output_path = (
            OUTPUT_DIR
            / f"{condition}_ocr_input.jpg"
        )

        cv2.imwrite(
            str(output_path),
            ocr_input,
        )


        rows.append(
            {
                "condition":
                    condition,

                "deskew":
                    "no",

                "expected_sn":
                    EXPECTED_SN,

                "raw_text":
                    repr(
                        result["raw_text"]
                    ),

                "final_text":
                    result["final_text"],

                "format_ok":
                    result["format_ok"],

                "expected_match":
                    result["expected_match"],

                "ocr_exact":
                    ocr_exact,

                "status":
                    result[
                        "predicted_status"
                    ],

                "reasons":
                    "|".join(
                        result["reasons"]
                    ),
            }
        )


        print(
            "=" * 60
        )

        print(
            "Condition:",
            condition
        )

        print(
            "Deskew: no"
        )

        print(
            "OCR:",
            result["final_text"]
        )

        print(
            "Exact:",
            ocr_exact
        )


        # ==================================
        # Tilt만 Deskew 후 다시 검사
        # ==================================

        if condition == "tilt":

            deskewed = rotate_image(
                roi,
                -8.0,
            )

            deskew_processed = (
                preprocess_for_ocr(
                    deskewed,
                    rules,
                )
            )

            deskew_input = cv2.resize(
                deskew_processed,
                None,
                fx=4.0,
                fy=4.0,
                interpolation=cv2.INTER_CUBIC,
            )

            deskew_raw = run_ocr(
                deskew_input,
                rules,
            )

            deskew_result = (
                inspect_text(
                    raw_text=deskew_raw,
                    expected_text=EXPECTED_SN,
                    pattern=rules[
                        "expected_pattern"
                    ],
                )
            )

            deskew_exact = (
                deskew_result[
                    "final_text"
                ]
                == EXPECTED_SN
            )


            cv2.imwrite(
                str(
                    OUTPUT_DIR
                    / "tilt_deskew_ocr_input.jpg"
                ),
                deskew_input,
            )


            rows.append(
                {
                    "condition":
                        "tilt",

                    "deskew":
                        "yes",

                    "expected_sn":
                        EXPECTED_SN,

                    "raw_text":
                        repr(
                            deskew_result[
                                "raw_text"
                            ]
                        ),

                    "final_text":
                        deskew_result[
                            "final_text"
                        ],

                    "format_ok":
                        deskew_result[
                            "format_ok"
                        ],

                    "expected_match":
                        deskew_result[
                            "expected_match"
                        ],

                    "ocr_exact":
                        deskew_exact,

                    "status":
                        deskew_result[
                            "predicted_status"
                        ],

                    "reasons":
                        "|".join(
                            deskew_result[
                                "reasons"
                            ]
                        ),
                }
            )


            print(
                "Deskew: yes"
            )

            print(
                "OCR:",
                deskew_result[
                    "final_text"
                ]
            )

            print(
                "Exact:",
                deskew_exact
            )


# ========================================
# 결과 확인
# ========================================

if not rows:
    raise RuntimeError(
        "환경 변화 테스트 이미지가 없습니다."
    )


# ========================================
# CSV 저장
# ========================================

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


with REPORT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=list(
            rows[0].keys()
        ),
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


print()

print(
    "Saved:",
    REPORT_PATH
)

print(
    "Environment Stress Test Complete"
)
```

실행합니다.

```
python src/48_day06_challenge.py
```

### 의사코드

```
프로그램 시작

        ↓

Freeze된
day06-ocr-v1.0을 읽는다

        ↓

대표 SN 정답을 가져온다

0637145699424

        ↓

Low Contrast 이미지를 읽는다

        ↓

original 전처리

        ↓

4배 확대

        ↓

OCR

        ↓

0637145699424와 비교

        ↓

결과 저장


        ↓


Blur도 같은 방법으로 검사


        ↓


Tilt도 같은 방법으로 검사

        ↓

Tilt는 추가로
-8° Deskew

        ↓

다시 OCR

        ↓

Deskew 전후 비교


        ↓


Glare도 같은 방법으로 검사


        ↓

전체 결과를

reports/day06_challenge_results.csv

에 저장

        ↓

프로그램 종료
```

---

## 43.1 `48_day06_challenge.py`에서는 무엇을 확인했나요?

이 프로그램은 정상 이미지에서 잘 동작했던 `day06-ocr-v1.0`을 다음 환경 변화 이미지에 그대로 적용했습니다.

```
Low Contrast
Blur
Tilt
Glare
```

정답 SN은 모두:

```
0637145699424
```

입니다.

결과는 다음 파일에 저장되었습니다.

```
reports/day06_challenge_results.csv
```

확인할 핵심 값은 다음입니다.

```
final_text
→ OCR이 최종적으로 읽은 값

ocr_exact
→ 정답 SN과 정확히 같은가?

status
→ 최종 OK / NG

reasons
→ 왜 NG가 되었는가?
```

---

# 44. 실제 환경 변화 테스트 결과를 확인합니다

실제 결과는 다음과 같습니다.

|조건|Deskew|OCR 결과|Exact|판정|
|---|---|---|---|---|
|`low_contrast`|no|`0637145699424`|True|OK|
|`blur`|no|빈 값|False|NG|
|`tilt`|no|`9728`|False|NG|
|`tilt`|yes|빈 값|False|NG|
|`glare`|no|빈 값|False|NG|

결과를 간단히 해석하면 다음과 같습니다.

```
Low Contrast
→ 13자리 SN을 정확히 읽음
→ 성공

Blur
→ 문자를 읽지 못함
→ 실패

Tilt
→ 9728만 잘못 읽음
→ 실패

Tilt + Deskew
→ 문자를 읽지 못함
→ 보정 후에도 실패

Glare
→ 문자를 읽지 못함
→ 실패
```

이번 테스트에서 현재 OCR Rule은 **저대비에는 버텼지만 Blur, Tilt, Glare에는 약한 것**을 확인했습니다.

---

## 44.1 `reasons`는 무엇을 의미하나요?

예를 들어 Blur 결과는:

```
NG_OCR_EMPTY
NG_FORMAT
NG_EXPECTED_MISMATCH
```

입니다.

의미는:

```
NG_OCR_EMPTY
→ OCR이 아무 문자도 읽지 못함

NG_FORMAT
→ 13자리 숫자 형식을 만족하지 못함

NG_EXPECTED_MISMATCH
→ Expected SN과 일치하지 않음
```

입니다.

Tilt의 경우:

```
OCR = 9728
```

처럼 일부 숫자는 읽었지만 13자리가 아니므로:

```
NG_FORMAT
NG_EXPECTED_MISMATCH
```

가 기록되었습니다.

---

# 45. 환경 변화 테스트 결과를 정리합니다

이번 결과를 한 번에 정리하면:

```
정상 조건
→ 성공

Low Contrast
→ 성공

Blur
→ 실패

Tilt
→ 실패

Tilt + Deskew
→ 실패

Glare
→ 실패
```

즉 현재 `day06-ocr-v1.0`은:

```
Low Contrast
→ 비교적 강함

Blur
→ 약함

Tilt
→ 약함

Glare
→ 약함
```

이라는 한계를 보였습니다.

특히 Tilt에서는 단순히 `-8°`로 되돌리는 Deskew를 적용했지만 OCR이 회복되지 않았습니다.

따라서 이번 테스트에서 확인한 핵심은 다음입니다.

> 정상 이미지에서는 100%였던 OCR도 영상 품질이 달라지면 쉽게 실패할 수 있으며, 단순한 보정만으로 항상 복구되는 것은 아닙니다.

이제 다음 PART에서는 이 실패가:

```
Blur 때문인지
Tilt 때문인지
Glare 때문인지
전처리 때문인지
OCR 자체의 한계인지
```

를 구분해서 분석하면 됩니다.

---

# PART7 — Failure Analysis: OCR 실패 원인 확인 (46~50)

---

# 46. 실패 결과를 해석합니다

PART6의 실제 결과는 다음과 같았습니다.

| 조건              | OCR 결과          | 결과  | 원인                 |
| --------------- | --------------- | --- | ------------------ |
| `low_contrast`  | `0637145699424` | 성공  | 저대비에서도 문자 인식 유지    |
| `blur`          | 빈 값             | 실패  | 문자 경계가 흐려짐         |
| `tilt`          | `9728`          | 실패  | 기울기로 일부 숫자만 인식     |
| `tilt + deskew` | 빈 값             | 실패  | 단순 회전 보정으로 복구되지 않음 |
| `glare`         | 빈 값             | 실패  | 반사로 문자 일부가 가려짐     |

여기서 기억할 것은 하나입니다.

```
프로그램의 NG 결과
        ↓
왜 NG가 되었는지
실제 이미지를 보고 원인을 찾는다
```

예를 들어:

```
Blur
        ↓
OCR 결과 없음
        ↓
13자리 형식 실패
        ↓
NG
```

입니다.

---

# 47. OCR 실패와 제품 NG를 구분합니다

두 가지는 서로 다릅니다.

```
제품 NG

OCR은 정확하게 읽음
        ↓
실제 SN ≠ Expected SN
        ↓
제품 자체가 NG
```

반대로:

```
OCR Failure

제품 SN은 정상
        ↓
Blur / Tilt / Glare
        ↓
OCR이 잘못 읽음
        ↓
OCR 검사 실패
```

즉:

> **제품이 잘못된 것과 OCR이 잘못 읽은 것은 다릅니다.**

---

# 48. 실패 분석 결과를 기록할 준비를 합니다

OCR이 실패했을 때는 복잡하게 분석하지 않고 다음 정도만 확인합니다.

```
입력 이미지 상태는 어떤가?

        ↓

OCR은 무엇을 읽었는가?

        ↓

13자리 형식이 맞는가?

        ↓

Expected SN과 같은가?

        ↓

실패 원인은 무엇인가?
```

이번 테스트에서는 이미 결과가 나왔으므로 다음 단계에서 이 내용을 한 번 정리하여 기록하고 마무리합니다.

---

# 49. 오늘 나온 OCR 결과를 간단히 기록합니다

지금까지 실제로 실행한 결과를 하나의 문서로 정리합니다.

새로운 분석을 하는 것이 아닙니다.

```
PART5
→ 정상 이미지 검사 결과

PART6
→ 영상 상태를 바꾼 테스트 결과

        ↓

한 문서에 간단히 정리
```

파일을 만듭니다.

```
reports/day06_failure_analysis.md
```

다음과 같이 작성합니다.

```
# Day 06 OCR 결과 정리

## 1. 정상 이미지 검사

사용한 OCR Rule:

- Version: day06-ocr-v1.0
- Preprocess: original
- PSM: 7
- 읽는 문자: 숫자만
- SN 형식: 숫자 13자리

결과:

- OCR 정확도: 6 / 6 성공
- 제품 판정 정확도: 6 / 6 성공

정상 이미지에서는 OCR과 제품 판정이 모두 잘 동작했습니다.


## 2. 영상 상태를 바꾼 테스트

정답 SN:

0637145699424


### Low Contrast

OCR 결과:

0637145699424

결과:

성공

확인한 점:

문자가 조금 옅어져도 이번 이미지에서는 정확하게 읽었습니다.


### Blur

OCR 결과:

읽지 못함

결과:

실패

확인한 점:

문자가 많이 흐려지면 숫자의 경계를 구분하기 어려워졌습니다.


### Tilt

OCR 결과:

9728

결과:

실패

확인한 점:

문자가 기울어지면서 13자리 전체를 읽지 못하고 일부 숫자만 읽었습니다.


### Tilt + Deskew

OCR 결과:

읽지 못함

결과:

실패

확인한 점:

기울기를 다시 돌려도 이번 테스트에서는 OCR이 회복되지 않았습니다.


### Glare

OCR 결과:

읽지 못함

결과:

실패

확인한 점:

빛 반사로 숫자 일부가 가려지면 OCR이 읽기 어려웠습니다.


## 3. 오늘 확인한 내용

정상 이미지에서는 OCR이 잘 동작했습니다.

하지만 이미지가 흐려지거나,
문자가 기울어지거나,
빛 반사로 문자가 가려지면
OCR이 실패할 수 있다는 것을 확인했습니다.
```

이 문서는 **잘 작성하는 것이 목적이 아닙니다.**

학생은 자신이 실행한 결과가:

```
어떤 조건에서 성공했는지

어떤 조건에서 실패했는지

왜 실패했다고 생각하는지
```

를 짧게 기록할 수 있으면 됩니다.

---

# 50. 6일차 OCR 실습을 마무리합니다

오늘은 제품 이미지의 SN을 읽고 검사하는 과정을 실습했습니다.

전체 흐름을 다시 보면 다음과 같습니다.

```
제품 이미지
        ↓
SN 영역 선택
        ↓
OCR
        ↓
0637145699424
        ↓
숫자 13자리인지 확인
        ↓
Expected SN과 비교
        ↓
OK / NG
```

그리고 여러 장을 검사하여 정상 조건에서는:

```
OCR
6 / 6 성공

제품 판정
6 / 6 성공
```

하는 것을 확인했습니다.

마지막으로 같은 SN 이미지를 일부러 변형하여 OCR이 영상 상태 변화에 얼마나 버티는지도 확인했습니다.

```
Low Contrast
→ 성공

Blur
→ 실패

Tilt
→ 실패

Tilt + Deskew
→ 실패

Glare
→ 실패
```

여기서 중요한 것은 실패한 것 자체가 아닙니다.

실제 제조현장에서는 OCR 프로그램이 같아도:

```
초점이 흐려지거나

제품이 기울어지거나

조명이 달라지거나

빛이 반사되면
```

결과가 달라질 수 있다는 것을 확인한 것입니다.

따라서 오늘 기억해야 할 핵심은 다음과 같습니다.

> OCR은 문자를 읽는 프로그램만 잘 만드는 것으로 끝나지 않습니다. 카메라가 문자를 얼마나 선명하고 안정적으로 촬영하는지도 매우 중요합니다.

이제 기본 OCR 검사 과정을 모두 경험했습니다.

다음 Mini Challenge에서는 새로운 제품 이미지에서 다른 문자 영역을 직접 선택하고, 오늘 사용한 방법을 다시 적용해 봅니다.

---

# PART8 — Mini Challenge: 새로운 제품 이미지에서 다른 문자 검사하기 (51~56)

---

# 51. Mini Challenge에서는 무엇을 하나요?

지금까지는 본 실습 이미지 6장에서 **SN 13자리 숫자**를 OCR로 검사했습니다.

이번에는 처음 사용하는 다음 4장의 제품 이미지를 사용합니다.

```
samples/day06/mini_challenge/product/

0185973818825_side.png
0386316000696_side.png
0635619134582_side.png
0924467469402_side.png
```

이 이미지들에도 다음과 같은 문자 정보가 있습니다.

```
SN:
Ch.-B.:
Verw.bis:
```

본 실습에서는 `SN`을 검사했으므로 Mini Challenge에서는 다른 영역인:

```
Ch.-B.
```

값을 검사합니다.

즉 새로운 OCR 기술을 배우는 것이 아니라 지금까지 배운 흐름을 **다른 문자에 직접 다시 적용**해 보는 것입니다.

```
새로운 제품 이미지 4장
        ↓
Ch.-B. 영역 선택
        ↓
OCR
        ↓
문자열 정리
        ↓
문자 형식 확인
        ↓
실제 값과 비교
        ↓
성공 / 실패 확인
```

> 앞에서 했던 코드를 처음부터 모두 다시 만드는 것이 아니라, **검사 대상이 바뀌면 무엇을 바꾸어야 하는지 확인하는 복습 실습**입니다.

---

# 52. SN 검사와 무엇이 달라질까요?

앞에서는 검사 대상이:

```
0637145699424
```

와 같은 **숫자 13자리 SN**이었습니다.

그래서:

```
Whitelist
→ 0123456789

Regex
→ ^[0-9]{13}$
```

를 사용했습니다.

하지만 이번 `Ch.-B.`에는 숫자뿐 아니라 **영문과 `-` 기호도 포함될 수 있습니다.**

따라서 그대로 사용할 수 없습니다.

이번 Mini Challenge에서는 다음처럼 시작합니다.

```
Whitelist
→ ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-

Regex
→ ^[A-Z0-9-]+$
```

Regex의 의미는 간단합니다.

```
영문 대문자
숫자
-

이 문자들로 이루어진 코드인가?
```

를 확인하는 것입니다.

> 정규식을 외울 필요는 없습니다. 이번에는 `Ch.-B.`에서 사용하는 문자 종류가 SN과 다르기 때문에 검사 규칙도 바뀐다는 것만 이해하면 됩니다.

---

# 53. 이번 Mini Challenge에서 직접 할 일

복잡하게 다시 실험하지 않습니다.

다음 5가지만 직접 합니다.

```
1. 새로운 이미지 4장을 연다

2. 각 이미지에서
   Ch.-B. 값만 ROI로 선택한다

3. OCR로 읽는다

4. 실제 이미지에 적힌 값과 비교한다

5. 성공 / 실패를 기록한다
```

전처리는 앞의 실험 결과를 이용하여 먼저:

```
original
```

로 시작합니다.

만약 특정 이미지에서 OCR이 실패하면 그때만:

```
gray

또는

clahe
```

를 다시 시험합니다.

> Mini Challenge에서 Original / Gray / CLAHE / Otsu / CLAHE+Otsu를 무조건 처음부터 모두 다시 비교할 필요는 없습니다. **필요할 때 앞의 42~43번 실습으로 돌아가 참고하면 됩니다.**

---

# 54. `49_day06_mini_challenge.py` — 새로운 4장의 `Ch.-B.`를 검사합니다

파일:

```
src/49_day06_mini_challenge.py
```

이번 코드에서는 이미지가 하나씩 열립니다.

학생은 각 이미지에서:

```
Ch.-B.:
```

오른쪽에 인쇄된 **실제 코드 값만 선택**합니다.

그 후 터미널에서 이미지에 보이는 값을 직접 입력하여 OCR 결과와 비교합니다.

작성합니다.

```python
import csv

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    load_rules,
    preprocess_for_ocr,
    run_ocr,
)

from day06_ocr_validation import (
    clean_ocr_text,
    validate_pattern,
)


# ========================================
# 경로 설정
# ========================================

PRODUCT_DIR = (
    PROJECT_ROOT
    / "samples"
    / "day06"
    / "mini_challenge"
    / "product"
)

OUTPUT_DIR = (
    PROJECT_ROOT
    / "outputs"
    / "day06"
    / "mini_challenge"
)

REPORT_PATH = (
    PROJECT_ROOT
    / "reports"
    / "day06_mini_challenge_results.csv"
)


# ========================================
# 기존 OCR 설정 가져오기
# ========================================

rules = load_rules()


# ========================================
# Ch.-B. 검사에 맞게 변경
# ========================================

rules[
    "preprocess_mode"
] = "original"

rules[
    "ocr_psm"
] = 7

rules[
    "whitelist"
] = (
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    "0123456789-"
)

rules[
    "expected_pattern"
] = r"^[A-Z0-9-]+$"


# ========================================
# Mini Challenge 이미지 찾기
# ========================================

image_paths = sorted(
    [
        path
        for path in PRODUCT_DIR.iterdir()
        if path.suffix.lower()
        in {
            ".jpg",
            ".jpeg",
            ".png",
        }
    ]
)


if not image_paths:
    raise FileNotFoundError(
        f"Mini Challenge 이미지가 없습니다: "
        f"{PRODUCT_DIR}"
    )


OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


rows = []


# ========================================
# 4장 하나씩 검사
# ========================================

for image_path in image_paths:

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        continue


    print()
    print(
        "=" * 60
    )

    print(
        "Image:",
        image_path.name
    )

    print(
        "Ch.-B.의 코드 값만 "
        "마우스로 선택하세요."
    )


    # ====================================
    # Ch.-B. ROI 직접 선택
    # ====================================

    x, y, w, h = cv2.selectROI(
        f"Select Ch.-B. ROI - {image_path.name}",
        image,
        fromCenter=False,
        showCrosshair=True,
    )

    cv2.destroyAllWindows()


    if w == 0 or h == 0:
        print(
            "ROI 선택이 취소되었습니다."
        )
        continue


    roi = image[
        int(y):int(y + h),
        int(x):int(x + w),
    ]


    # ====================================
    # 앞에서 선택한 original 사용
    # ====================================

    processed = (
        preprocess_for_ocr(
            roi,
            rules,
        )
    )


    # ====================================
    # 이전 실습과 동일하게 4배 확대
    # ====================================

    ocr_input = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )


    # ====================================
    # OCR
    # ====================================

    raw_text = run_ocr(
        ocr_input,
        rules,
    )

    ocr_text = (
        clean_ocr_text(
            raw_text
        )
        .upper()
    )


    # ====================================
    # 이미지에서 직접 확인한 정답 입력
    # ====================================

    print(
        "OCR 결과:",
        ocr_text
    )

    expected_text = input(
        "이미지에 실제로 보이는 "
        "Ch.-B. 값을 입력하세요: "
    ).strip().upper()


    # ====================================
    # 형식과 OCR 정확도 확인
    # ====================================

    format_ok = (
        validate_pattern(
            ocr_text,
            rules[
                "expected_pattern"
            ],
        )
    )

    exact_match = (
        ocr_text
        == expected_text
    )


    # ====================================
    # OCR 입력 이미지 저장
    # ====================================

    output_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_chb_ocr_input.jpg"
    )

    cv2.imwrite(
        str(output_path),
        ocr_input,
    )


    # ====================================
    # 결과 기록
    # ====================================

    rows.append(
        {
            "file":
                image_path.name,

            "expected_text":
                expected_text,

            "ocr_text":
                ocr_text,

            "format_ok":
                format_ok,

            "exact_match":
                exact_match,
        }
    )


    print(
        "Format OK:",
        format_ok
    )

    print(
        "Exact Match:",
        exact_match
    )


# ========================================
# CSV 저장
# ========================================

if not rows:
    raise RuntimeError(
        "검사 결과가 없습니다."
    )


REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)


with REPORT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "file",
            "expected_text",
            "ocr_text",
            "format_ok",
            "exact_match",
        ],
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


print()
print(
    "Saved:",
    REPORT_PATH
)

print(
    "Day06 Mini Challenge Complete"
)
```

실행합니다.

```
python src/49_day06_mini_challenge.py
```

---

## 54.1 실행할 때 무엇을 하면 되나요?

이미지가 한 장씩 열립니다.

다음 문자 전체를 잡는 것이 아니라:

```
Ch.-B.:
```

오른쪽에 있는 **코드 값만** 선택합니다.

```
Ch.-B.:  XXXXXXXX
         └───────┘
          이 부분
```

ROI를 선택하면 OCR 결과가 먼저 출력됩니다.

예:

```
OCR 결과: ...
```

그다음 프로그램이 묻습니다.

```
이미지에 실제로 보이는 Ch.-B. 값을 입력하세요:
```

학생은 화면에서 직접 읽은 값을 입력합니다.

그러면 프로그램이:

```
OCR 결과

        vs

학생이 확인한 실제 값
```

을 비교합니다.

---

## 54.2 무엇을 확인하면 되나요?

결과는 두 가지만 보면 됩니다.

```
Format OK
→ 영문 / 숫자 / - 형식으로 읽혔는가?

Exact Match
→ OCR 결과가 실제 코드와 정확히 같은가?
```

예:

```
Format OK: True
Exact Match: True
```

라면 성공입니다.

반대로:

```
Format OK: True
Exact Match: False
```

라면 코드 모양은 맞지만 **OCR이 일부 문자를 잘못 읽은 것**입니다.

결과는 다음 파일에 자동으로 저장됩니다.

```
reports/day06_mini_challenge_results.csv
```

OCR에 실제로 사용한 확대 이미지는:

```
outputs/day06/mini_challenge/
```

에서 확인할 수 있습니다.

---

# 55. 실패한 이미지가 있다면 한 번만 개선해 봅니다

4장 모두 성공했다면 여기서 끝내도 됩니다.

실패한 이미지가 있다면 그 이미지에 대해서만 앞에서 배운 전처리를 다시 생각합니다.

```
original 실패
        ↓

이미지가 너무 흐린가?
        ↓
gray 또는 clahe를 한 번 시도

        ↓

OCR 다시 확인
```

예를 들어 코드의:

```
rules[
    "preprocess_mode"
] = "original"
```

을:

```
rules[
    "preprocess_mode"
] = "gray"
```

또는:

```
rules[
    "preprocess_mode"
] = "clahe"
```

로 바꾸어 다시 실행할 수 있습니다.

중요한 것은 전처리를 계속 바꾸어 무조건 정답을 만드는 것이 아닙니다.

```
왜 실패했는가?

다른 전처리가 도움이 되었는가?
```

를 한 번 확인하면 충분합니다.

> Otsu까지 포함한 모든 전처리를 다시 전부 시험하고 싶다면 앞의 **42~43번 전처리 비교 실습**을 참고합니다.

---

# 56. Mini Challenge 결과를 짧게 정리합니다

마지막으로 결과를 길게 보고서로 작성할 필요는 없습니다.

다음 파일:

```
reports/day06_mini_challenge.md
```

에 자신이 실행한 결과만 간단히 작성합니다.

```
# Day 06 Mini Challenge

## 검사 대상

Ch.-B. 제품 코드

## 사용한 데이터

새로운 제품 이미지 4장

## 바꾼 OCR 설정

SN 검사:
숫자만

Mini Challenge:
영문 대문자 + 숫자 + -

## 사용한 전처리

original

## 결과

4장 중 OCR 성공:
작성

4장 중 OCR 실패:
작성

## 실패한 이미지가 있었다면

파일:
작성

OCR 결과:
작성

실제 값:
작성

다시 사용해 본 전처리:
작성

결과가 좋아졌는가?:
작성

## 확인한 점

SN에서 사용했던
ROI → OCR → 문자열 정리 → 형식 검사 → 실제 값 비교
과정을 다른 문자 코드에도 다시 적용할 수 있었다.
```

---

## Mini Challenge에서 확인할 핵심

이번에는 코드를 많이 작성하는 것이 목표가 아닙니다.

학생이 다음 흐름을 스스로 다시 할 수 있으면 됩니다.

```
새로운 이미지
        ↓
무엇을 검사할지 결정

Ch.-B.

        ↓
그 문자만 ROI

        ↓
검사 문자 종류 확인

숫자만?
영문도 포함?
- 도 포함?

        ↓
Whitelist 변경

        ↓
OCR

        ↓
형식 확인

        ↓
실제 문자와 비교

        ↓
성공 / 실패 확인
```

본 실습과 비교하면 바뀐 것은 사실 많지 않습니다.

```
본 실습

SN
→ 숫자 13자리
→ 숫자 Whitelist
→ 13자리 Regex


Mini Challenge

Ch.-B.
→ 영문 + 숫자 + -
→ Whitelist 변경
→ 문자 형식 Regex 변경
```

> 검사 대상이 달라져도 `ROI → OCR → 문자열 검사 → 정답 비교`라는 전체 구조는 그대로라는 것을 스스로 확인하는 것이 Mini Challenge의 목적입니다.

---

# PART9 — 6일차 마무리: 산출물 확인 + README + Git + 자가체크 (57~62)

---

# 57. 오늘 만든 결과를 확인합니다

6일차 실습을 마치면 다음 파일들이 만들어져 있어야 합니다.

## 실행 코드

```bash
src/40_day06_check_ocr_env.py
src/41_day06_select_text_roi.py
src/42_day06_preprocess_compare.py
src/43_day06_ocr_compare.py
src/44_day06_freeze_ocr_rules.py
src/45_day06_batch_ocr.py
src/46_day06_evaluate.py
src/47_day06_make_challenge_variants.py
src/48_day06_challenge.py
src/49_day06_mini_challenge.py
```

여러 파일에서 함께 사용하는 공통 함수:

```
src/day06_ocr_utils.py
src/day06_ocr_validation.py
```

## OCR 설정과 정답 데이터

```
configs/day06_ocr_rules.json
configs/day06_labels.csv
```

## 이미지 결과

```
outputs/day06/roi/
outputs/day06/preprocess/
outputs/day06/batch/
outputs/day06/challenge/
outputs/day06/mini_challenge/
```

## 검사 결과와 기록

```
reports/day06_ocr_environment.txt
reports/day06_preprocess_compare.csv
reports/day06_ocr_results.csv
reports/day06_ocr_metrics.md
reports/day06_challenge_results.csv
reports/day06_failure_analysis.md
reports/day06_mini_challenge_results.csv
reports/day06_mini_challenge.md
```

모든 파일명을 외울 필요는 없습니다.

중요한 것은 다음 구조입니다.

```
src
→ 실행 코드

configs
→ 검사 Rule과 정답 기준

outputs
→ 실제 처리된 이미지

reports
→ OCR 결과와 분석 기록
```

---

# 58. README에 6일차 결과를 정리합니다

`README.md`에는 오늘 진행한 내용을 길게 다시 쓰지 않습니다.

무엇을 검사했고, 어떤 설정을 사용했고, 어떤 결과가 나왔는지만 남깁니다.

다음 내용을 추가합니다.

```markdown
## Day 06 — OCR 기반 제조 문자 검사

### 검사 대상

제품에 인쇄된 13자리 SN

Mini Challenge:
Ch.-B. 제품 코드


### 사용 기술

- OpenCV ROI
- Gray / CLAHE / Otsu 비교
- Tesseract OCR
- pytesseract
- PSM
- Whitelist
- Text Cleaning
- Regex
- Expected SN 비교
- Deskew


### 최종 OCR Rule

Version: day06-ocr-v1.0

Preprocess: original

PSM: 7

Whitelist: 0123456789

Pattern: ^[0-9]{13}$


### Batch 검사 결과

검사 이미지: 6장

OCR Exact Accuracy: 100%

Inspection Accuracy: 100%

OCR Failure: 없음

Inspection Failure: 없음


### 환경 변화 테스트

Low Contrast: 성공

Blur: 실패

Tilt: 실패

Tilt + Deskew: 실패

Glare: 실패


### Mini Challenge

검사 대상: Ch.-B.

OCR 성공:
실습 결과 작성

OCR 실패:
실습 결과 작성


### 주요 결과 파일

- configs/day06_ocr_rules.json
- reports/day06_preprocess_compare.csv
- reports/day06_ocr_results.csv
- reports/day06_ocr_metrics.md
- reports/day06_challenge_results.csv
- reports/day06_failure_analysis.md
- reports/day06_mini_challenge_results.csv
- reports/day06_mini_challenge.md


### 오늘 확인한 핵심

제품 이미지에서 필요한 문자 영역을 ROI로 선택하고,
OCR로 읽은 문자열을 형식과 Expected Value로 검사하여
OK / NG 판정으로 연결했습니다.

또한 OCR이 정확하게 문자를 읽는 것과
제품 자체가 정상인 것은 서로 다른 문제라는 것을 확인했습니다.
```

Mini Challenge 결과 부분만 실제 실행 결과에 맞게 작성하면 됩니다.

---

# 59. Local Git에 6일차 작업을 기록합니다

먼저 현재 상태를 확인합니다.

```bash
git status
```

6일차 코드와 설정 파일을 추가합니다.

```
git add \
src/day06_ocr_utils.py \
src/day06_ocr_validation.py \
src/40_day06_check_ocr_env.py \
src/41_day06_select_text_roi.py \
src/42_day06_preprocess_compare.py \
src/43_day06_ocr_compare.py \
src/44_day06_freeze_ocr_rules.py \
src/45_day06_batch_ocr.py \
src/46_day06_evaluate.py \
src/47_day06_make_challenge_variants.py \
src/48_day06_challenge.py \
src/49_day06_mini_challenge.py \
configs/day06_ocr_rules.json \
configs/day06_labels.csv \
requirements.txt
```

Commit 합니다.

```bash
git commit -m "feat: add day06 OCR inspection"
```

---

# 60. README와 결과 기록을 Git에 남깁니다

오늘 작성한 README와 Report를 추가합니다.

```
git add \
README.md \
reports/day06_ocr_environment.txt \
reports/day06_preprocess_compare.csv \
reports/day06_ocr_results.csv \
reports/day06_ocr_metrics.md \
reports/day06_challenge_results.csv \
reports/day06_failure_analysis.md \
reports/day06_mini_challenge_results.csv \
reports/day06_mini_challenge.md
```

Commit 합니다.

```bash
git commit -m "docs: record day06 OCR results"
```

마지막으로 확인합니다.

```bash
git status
```

정리할 파일이 없다면 6일차 작업 기록이 완료된 것입니다.

> 수업에서는 기관에서 허용한 Local Git 또는 내부 저장소를 사용하고, 외부 반출이 제한된 이미지나 데이터는 외부 저장소에 올리지 않습니다.

---

# 61. 6일차 완료 체크

수업을 마치기 전에 실제로 하나씩 확인합니다.

### OCR 환경과 데이터

- [ ]  Tesseract가 정상적으로 실행되었다.
- [ ]  `pytesseract`를 이용해 Python에서 OCR을 실행했다.
- [ ]  본 실습용 제품 이미지 6장을 준비했다.
- [ ]  `day06_labels.csv`를 준비했다.

### SN OCR 실습

- [ ]  제품 이미지에서 13자리 SN ROI를 선택했다.
- [ ]  Original / Gray / CLAHE / Otsu / CLAHE+Otsu를 비교했다.
- [ ]  전처리별 실제 OCR 결과를 비교했다.
- [ ]  Original을 최종 전처리로 선택했다.
- [ ]  PSM 7이 한 줄 문자용 설정이라는 것을 이해했다.
- [ ]  Whitelist로 숫자만 읽도록 제한했다.
- [ ]  Raw OCR과 정리된 문자열의 차이를 확인했다.
- [ ]  `^[0-9]{13}$`가 13자리 숫자를 검사하는 정규식이라는 것을 이해했다.
- [ ]  Expected SN과 OCR 결과를 비교했다.
- [ ]  `day06-ocr-v1.0`으로 Rule을 고정했다.

### Batch 검사

- [ ]  제품 이미지 6장을 Batch로 검사했다.
- [ ]  `day06_ocr_results.csv`를 확인했다.
- [ ]  OCR Exact Accuracy가 `6 / 6 = 100%`임을 확인했다.
- [ ]  Inspection Accuracy가 `6 / 6 = 100%`임을 확인했다.
- [ ]  OCR 성공과 제품 OK가 같은 의미가 아니라는 것을 이해했다.

### 환경 변화 테스트

- [ ]  정상 SN ROI에서 Low Contrast 이미지를 만들었다.
- [ ]  Blur 이미지를 만들었다.
- [ ]  Tilt 이미지를 만들었다.
- [ ]  Glare 이미지를 만들었다.
- [ ]  같은 OCR Rule로 다시 검사했다.
- [ ]  Low Contrast에서는 OCR에 성공했다.
- [ ]  Blur에서는 OCR에 실패했다.
- [ ]  Tilt에서는 OCR에 실패했다.
- [ ]  Tilt에 Deskew를 적용해 보았지만 이번 실험에서는 복구되지 않았다.
- [ ]  Glare에서는 OCR에 실패했다.
- [ ]  영상 품질과 촬영 환경이 OCR 성능에 영향을 준다는 것을 확인했다.

### Mini Challenge

- [ ]  처음 보는 제품 이미지 4장을 사용했다.
- [ ]  SN이 아닌 `Ch.-B.` 영역을 직접 선택했다.
- [ ]  문자 종류에 맞게 Whitelist를 변경했다.
- [ ]  문자 형식에 맞게 Regex를 변경했다.
- [ ]  OCR 결과와 실제 `Ch.-B.` 값을 비교했다.
- [ ]  실패한 이미지가 있다면 Gray 또는 CLAHE를 한 번 다시 적용해 보았다.
- [ ]  Mini Challenge 결과를 기록했다.

### 마무리

- [ ]  `reports/day06_failure_analysis.md`를 작성했다.
- [ ]  `reports/day06_mini_challenge.md`를 작성했다.
- [ ]  README에 6일차 결과를 추가했다.
- [ ]  Local Git에 6일차 작업을 Commit 했다.

---

# 62. 6일차에서 무엇을 배웠는지 연결합니다

1~5일차에서는 주로 제품의 모양, 크기, 위치, 특징을 측정했습니다.

6일차에서는 검사 대상이 문자 정보로 확장되었습니다.

![[Pasted image 20260901210044.png]]

```
1~5일차

제품 이미지
        ↓
모양 / 크기 / 위치 / 특징 측정
        ↓
숫자 기준
        ↓
OK / NG


6일차

제품 이미지
        ↓
문자 ROI
        ↓
OCR
        ↓
문자열
        ↓
형식 검사
        ↓
Expected Value 비교
        ↓
OK / NG
```

검사 대상은 달라졌지만 기본 생각은 같습니다.

```
이미지
        ↓
필요한 정보 추출
        ↓
기준과 비교
        ↓
OK / NG
        ↓
결과 기록
```

6일차에서 반드시 기억할 흐름은 다음입니다.

```
문자 ROI
        ↓
OCR
        ↓
문자열 정리
        ↓
형식 검사
        ↓
Expected Value 비교
        ↓
OK / NG
```

그리고 한 가지를 더 확인했습니다.

```
OCR이 정확하다
≠
제품이 반드시 OK다
```

OCR은 무엇이 인쇄되어 있는지 읽는 역할이고, 제품 검사는 그 값이 원래 인쇄되어야 할 값과 맞는지 판단하는 역할입니다.

또한:

```
초점
기울기
조명
반사
```

와 같은 영상 조건이 달라지면 동일한 OCR 프로그램도 실패할 수 있음을 실제 실험으로 확인했습니다.