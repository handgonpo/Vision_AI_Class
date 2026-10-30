## 오늘의 목표

6일차에서는 검사 대상이 제품의 외형에서 제품에 인쇄된 문자 정보로 확장됩니다.

![[Pasted image 20261029174015.png]]

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

![[Pasted image 20261029174842.png]]

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

![[Pasted image 20261029143235.png]]

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

![[Pasted image 20261029144133.png]]

제품 전체를 OCR하지 않고 각 이미지에서:

```
SN: 0637145699424
    └───────────┘
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

Regex는 정규식(정규표현식)을 뜻하며, Regular Expression의 줄임말입니다

정규식이란 문자열이 정해진 규칙에 맞는지 검사하거나, 특정 패턴의 문자를 찾는 방법입니다.

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

## 1.6 PART6 — 촬영 환경 변화에 따른 OCR 성능을 확인합니다

![[Pasted image 20261029145657.png]]

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

![[Pasted image 20261029150903.png]]

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

# 3. 오늘 검사할 제품의 문자 정보를 살펴봅니다

![[Pasted image 20261029154041.png]]

오늘부터는 제품의 모양이나 위치뿐 아니라 **제품 표면에 인쇄된 정보가 올바른지도 검사**합니다.

실제 제조 현장에서는 문자 하나가 잘못 인쇄되어도 제품 식별이나 이력 관리에 문제가 생길 수 있습니다. 따라서 OCR 프로그램을 만들기 전에, 이미지에서 **무엇을 읽고 무엇과 비교할지** 
먼저 정해야 합니다.

## 3.1 제품 라벨에는 어떤 정보가 있나요?

오늘 사용할 제품의 측면에는 다음 세 가지 정보가 인쇄되어 있습니다.

|라벨 표기|의미|6일차에서의 사용|
|---|---|---|
|`SN: 0637145699424`|SN(Serial Number): 제품 식별 번호|**본 실습에서 검사**|
|`Ch.-B.: NU156717-2P`|제품의 배치·로트 관련 코드|**Mini Challenge에서 검사**|
|`Verw.bis: 12.2082`|제품에 표시된 사용기한 관련 정보|이번 실습에서는 검사하지 않음|

한 장의 이미지에 여러 문자가 있더라도 모두 읽을 필요는 없습니다. **이번 본 실습의 검사 대상은 SN 값의 숫자 13자리뿐**입니다.

`SN:` 표시는 어떤 정보인지 알려주는 항목 이름 이므로, OCR에 입력할 ROI는 가능하면 그 뒤의 숫자만 포함하도록 선택합니다. 구체적인 ROI 선택 방법은 뒤의 PART2에서 직접 실습합니다.

## 3.2 SN은 숫자로 보이지만 '문자열'로 다룹니다

예제 SN인 `0637145699424`는 맨 앞에 `0`이 있습니다.

SN은 계산할 숫자가 아니라 **제품을 구분하는 식별 코드**이므로, 앞의 `0`도 정보의 일부입니다.

```
sn = "0637145699424"  # 문자열(str)로 저장
print(len(sn))        # 13
```

이를 정수(`int`)로 바꾸면 앞의 `0`이 사라져 `637145699424`가 됩니다.

이렇게 되면 원래의 13자리 코드를 그대로 비교할 수 없습니다. 따라서 OCR 결과와 정답 SN은 모두 문자열로 보관합니다. CSV를 Excel에서 열 때도 앞자리 `0`이 사라지지 않도록 텍스트 형식으로 가져와야 합니다.

## 3.3 '실제 인쇄값', '기대값', 'OCR 결과'는 서로 다른 정보입니다

처음에는 세 값이 모두 같은 숫자로 보여 헷갈리기 쉽습니다. 그러나 각각의 역할이 다릅니다.

| 이름                            | 뜻                          | 어디에 사용하나요?                  |
| ----------------------------- | -------------------------- | --------------------------- |
| Actual SN (`actual_text`)     | 제품 이미지에 **실제로 인쇄된** 값      | OCR이 제대로 읽었는지 평가할 때 사용하는 정답 |
| Expected SN (`expected_text`) | 해당 제품에 **인쇄되어 있어야 하는** 값   | 제품의 OK / NG 판정 기준           |
| OCR 결과 (`ocr_text`)           | Tesseract가 이미지에서 **읽어낸** 값 | 형식 확인 및 Expected SN 비교에 사용  |

이번 실습에서는 OCR 정확도와 제품 판정 정확도의 차이를 이해하기 위해 일부 제품의 Expected SN을 실제 인쇄값과 다르게 설정했습니다. 이는 실제 불량제품이라는 의미가 아니라, 제조 현장에서 잘못된 SN이 인쇄된 상황을 가정한 교육용 NG 시나리오입니다.

이때 OCR이 `0679256889404`를 읽었다면 문자를 **정확히 인식한 것**이지만, 제품에 기대한 번호와는 다릅니다. 따라서 이 제품의 정답 판정은 **NG**입니다.

여기서 중요한 점은 `actual_text`의 역할입니다.

**실제 검사 프로그램이 제품의 정답 인쇄값을 미리 알고 있는 것은 아닙니다.** 이번 교육에서는 OCR 정확도를 확인하기 위해 사람이 검증한 실제 인쇄값을 정답표(Ground Truth)에 따로 기록합니다.

반면 제품 판정에는 Expected SN을 사용합니다. 두 값의 구체적인 CSV 구성은 뒤의 **10번 실습**에서 준비합니다.

## 3.4 잘못 읽은 것과 잘못 인쇄된 것은 구분해야 합니다

검사 결과가 NG라고 해서 곧바로 제품에 잘못 인쇄되었다고 단정하면 안 됩니다.

예를 들어 촬영 영상이 흐려서 OCR이 일부 숫자를 놓쳤을 수도 있고, **13자리 전체를 읽었더라도 다른 숫자로 잘못 인식**했을 수도 있습니다.

따라서 실패 원인을 확인할 때는 다음을 함께 살펴봐야 합니다.

- **원본 이미지:** 실제로 어떤 SN이 인쇄되어 있는가?
    
- **OCR 결과:** 문자를 빠뜨리거나 잘못 읽지는 않았는가?
    
- **Expected SN:** 검사 기준으로 지정된 번호가 무엇인가?
    

또한 OCR이 반환한 숫자를 프로그램이 임의로 Expected SN에 맞춰 고쳐서는 안 됩니다. 그렇게 하면 읽기 오류가 감춰져 OCR 성능을 제대로 평가할 수 없습니다.

이번 수업에서는 원본 OCR 결과를 보존하고, 필요한 공백이나 줄바꿈만 정리합니다.

**실습을 시작하기 전에 기억하세요.**

오늘 검사기는 SN을 읽는 기능과, 읽은 값이 정상인지 판정하는 기능을 함께 사용합니다.

다음 **4번**에서는 여기에 필요한 OCR·PSM·Whitelist·Regex 기술을 배우고, 이후 실습에서 이 기능들을 하나씩 연결합니다.

---

# 4. 6일차에서 새롭게 사용하는 기술을 먼저 이해합니다

## OCR 전체 흐름

![[Pasted image 20261029155006.png]]
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

PSM(Page Segmentation Mode)은 Tesseract OCR 엔진에 기본으로 포함된 설정 옵션(Parameter)입니다.

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

Whitelist도 PSM과 마찬가지로 Tesseract OCR 엔진에 기본으로 포함된 설정 옵션입니다.

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

![[Pasted image 20261029155423.png]]

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


# --------------------------------------------------
# 1. OCR 환경 확인에 필요한 라이브러리 준비
#
# 무엇을 하나요?
# - Python 표준 라이브러리와 OCR 관련 라이브러리를
#   불러옵니다.
#
# Path:
# 파일과 폴더 경로를 관리합니다.
#
# platform:
# 현재 운영체제 정보를 확인합니다.
#
# shutil:
# Tesseract 실행 파일의 위치를 찾습니다.
#
# subprocess:
# Python에서 터미널 명령어를 실행합니다.
#
# cv2:
# 현재 설치된 OpenCV 버전을 확인합니다.
#
# pytesseract:
# Python에서 Tesseract OCR Engine을 연결하고
# 버전 정보를 확인합니다.
#
# 왜 필요한가요?
# OCR 실습을 시작하기 전에 필요한 프로그램들이
# 설치되어 있는지 확인하기 위해서입니다.
#
# 참고:
# 위 import 문에서 필요한 라이브러리를 불러옵니다.
# --------------------------------------------------


# --------------------------------------------------
# 2. 프로젝트 최상위 폴더와 환경 보고서 경로 설정
#
# 무엇을 하나요?
# - 현재 실행 중인 Python 파일의 위치를 찾습니다.
# - src 폴더의 상위 폴더를 프로젝트 Root로 지정합니다.
# - 환경 확인 결과를 저장할 TXT 파일 경로를 만듭니다.
#
# __file__:
# 현재 실행 중인 Python 파일의 경로입니다.
#
# .resolve():
# 파일의 절대경로를 구합니다.
#
# .parent:
# 현재 경로의 상위 폴더로 이동합니다.
#
# .parent.parent:
# src 폴더에서 프로젝트 최상위 폴더로 이동합니다.
#
# 예:
# subject9_machinevision/
# ├── src/
# │   └── 40_day06_check_ocr_env.py
# └── reports/
#     └── day06_ocr_environment.txt
#
# 왜 필요한가요?
# 프로그램을 어느 위치에서 실행하더라도
# 프로젝트의 reports 폴더에 결과를 저장하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. 현재 운영체제와 OpenCV 버전 확인
#
# 무엇을 하나요?
# - 현재 Python이 실행되는 운영체제 정보를 확인합니다.
# - 설치된 OpenCV 버전을 확인합니다.
# - 확인한 내용을 lines 리스트에 저장합니다.
#
# platform.platform():
# Linux, Windows 등 현재 운영체제의 정보를 반환합니다.
#
# cv2.__version__:
# 현재 설치된 OpenCV 버전을 반환합니다.
#
# lines:
# 환경 확인 결과를 한 줄씩 저장할 리스트입니다.
#
# 왜 필요한가요?
# 학생마다 실행환경이나 라이브러리 버전이 다를 수 있으므로
# 현재 환경 정보를 먼저 기록하기 위해서입니다.
# --------------------------------------------------

lines = [
    f"OS: {platform.platform()}",
    f"OpenCV: {cv2.__version__}",
]


# --------------------------------------------------
# 4. Tesseract OCR Engine 실행 파일 찾기
#
# 무엇을 하나요?
# - 현재 운영체제에서 tesseract 명령어를 찾습니다.
# - 설치된 Tesseract 실행 파일의 경로를 확인합니다.
#
# shutil.which("tesseract"):
# 시스템의 PATH 환경변수에 등록된 폴더에서
# tesseract 실행 파일을 검색합니다.
#
# Tesseract를 찾으면:
# 실행 파일의 경로를 반환합니다.
#
# Tesseract를 찾지 못하면:
# None을 반환합니다.
#
# 예:
# /usr/bin/tesseract
#
# 왜 필요한가요?
# pytesseract는 실제 OCR Engine이 아니라
# Python에서 Tesseract를 호출하는 연결 라이브러리입니다.
#
# 따라서 Tesseract 실행 프로그램이
# 별도로 설치되어 있어야 합니다.
# --------------------------------------------------

tesseract_path = shutil.which(
    "tesseract"
)


# --------------------------------------------------
# 5. Tesseract를 찾지 못한 경우 오류 정보 기록
#
# 무엇을 하나요?
# - Tesseract 실행 파일을 찾았는지 확인합니다.
# - 파일이 없다면 NOT FOUND 메시지를 기록합니다.
#
# tesseract_path is None:
# Tesseract 실행 파일을 찾지 못했다는 의미입니다.
#
# lines.append():
# 환경 확인 결과 리스트에 새로운 내용을 추가합니다.
#
# 왜 필요한가요?
# OCR이 실행되지 않는 원인이
# Tesseract 설치 또는 PATH 설정 문제인지
# 구분하기 위해서입니다.
#
# 주의:
# 이 경우에도 환경 보고서는 저장합니다.
# --------------------------------------------------

if tesseract_path is None:

    lines.append(
        "Tesseract: NOT FOUND"
    )

    lines.append(
        "Tesseract Path: NOT FOUND"
    )


else:

    # --------------------------------------------------
    # 6. Tesseract 설치 경로와 버전 확인
    #
    # 무엇을 하나요?
    # - Tesseract 실행 파일을 찾았다면 경로를 기록합니다.
    # - pytesseract로 실제 Tesseract 버전을 확인합니다.
    #
    # pytesseract.get_tesseract_version():
    # 설치된 Tesseract OCR Engine의 버전을 확인합니다.
    #
    # try:
    # 버전 확인을 시도합니다.
    #
    # except:
    # 버전 확인 중 오류가 발생하면 오류를 기록합니다.
    #
    # 왜 필요한가요?
    # Tesseract 실행 파일이 존재하는지 확인하는 것과
    # 실제로 정상 실행되는지 확인하는 것은 다르기 때문입니다.
    # --------------------------------------------------

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


    # --------------------------------------------------
    # 7. 설치된 OCR 언어 데이터 확인
    #
    # 무엇을 하나요?
    # - Tesseract가 사용할 수 있는 언어 목록을 확인합니다.
    # - tesseract --list-langs 명령어를 실행합니다.
    #
    # subprocess.run():
    # Python 코드에서 외부 명령어를 실행하는 함수입니다.
    #
    # capture_output=True:
    # 명령어의 출력 결과를 Python에서 받아옵니다.
    #
    # text=True:
    # 출력 결과를 문자열로 처리합니다.
    #
    # check=True:
    # 명령어가 실패하면 예외를 발생시킵니다.
    #
    # result.stdout:
    # 명령어의 표준 출력 결과입니다.
    #
    # strip():
    # 출력 결과의 앞뒤 공백과 줄바꿈을 제거합니다.
    #
    # 예:
    # eng → 영어 OCR
    # kor → 한국어 OCR
    # osd → 문자 방향 및 스크립트 감지
    #
    # 왜 필요한가요?
    # Tesseract Engine이 설치되어 있어도
    # 사용할 OCR 언어 데이터가 없다면
    # 문자를 정상적으로 인식하지 못할 수 있기 때문입니다.
    #
    # 이번 6일차 실습:
    # 숫자 13자리 SN을 검사하며 eng를 사용합니다.
    # --------------------------------------------------

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


# --------------------------------------------------
# 8. OCR 환경 확인 결과를 TXT 파일로 저장
#
# 무엇을 하나요?
# - reports 폴더가 있는지 확인합니다.
# - 폴더가 없으면 생성합니다.
# - 지금까지 확인한 환경 정보를 TXT 파일로 저장합니다.
#
# REPORT_PATH.parent.mkdir():
# 결과 파일의 상위 폴더를 생성합니다.
#
# parents=True:
# 필요한 상위 폴더까지 함께 생성합니다.
#
# exist_ok=True:
# 폴더가 이미 있으면 그대로 사용합니다.
#
# "\n".join(lines):
# lines에 저장한 결과를 줄바꿈으로 연결합니다.
#
# write_text():
# 문자열을 TXT 파일에 저장합니다.
#
# encoding="utf-8":
# 한글을 포함한 문자열도 저장할 수 있도록 설정합니다.
#
# 왜 필요한가요?
# 현재 OCR 개발환경을 파일로 기록하여
# 나중에 환경을 재현하거나 오류를 분석할 때
# 참고하기 위해서입니다.
# --------------------------------------------------

REPORT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

REPORT_PATH.write_text(
    "\n".join(lines),
    encoding="utf-8",
)


# --------------------------------------------------
# 9. 확인한 환경 정보와 저장 경로 출력
#
# 무엇을 하나요?
# - lines에 저장된 환경 확인 결과를 출력합니다.
# - TXT 보고서가 저장된 위치를 출력합니다.
#
# 왜 필요한가요?
# 파일을 직접 열지 않아도
# 터미널에서 현재 OCR 환경을 확인하기 위해서입니다.
#
# 출력 내용:
# OS
# OpenCV 버전
# Tesseract 설치 경로
# Tesseract 버전
# 설치된 OCR 언어
# 보고서 저장 경로
# --------------------------------------------------

print(
    "\n".join(lines)
)

print()

print(
    "Saved:",
    REPORT_PATH
)


# --------------------------------------------------
# 10. Tesseract 발견 여부에 따라 최종 메시지 출력
#
# 무엇을 하나요?
# - 4번에서 찾은 Tesseract 실행 파일 경로를 확인합니다.
# - 찾지 못했다면 Failed 메시지를 출력합니다.
# - 찾았다면 OK 메시지를 출력합니다.
#
# OCR Environment Check Failed:
# Tesseract 실행 파일을 찾지 못한 경우
#
# OCR Environment OK:
# Tesseract 실행 파일을 찾은 경우
#
# 주의:
# 현재 코드는 실행 파일의 존재 여부를 기준으로
# 최종 메시지를 결정합니다.
#
# 따라서 OCR Environment OK가 출력되어도
# Tesseract 버전이나 언어 확인에서 오류가 있었다면
# 실제 OCR 실습 전에 추가 확인이 필요합니다.
# --------------------------------------------------

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
[1. OCR 환경 확인에 필요한 라이브러리 준비]

필요한 Python 라이브러리를 불러온다.

Path
→ 프로젝트 파일과 폴더 경로 관리

platform
→ 운영체제 정보 확인

shutil
→ Tesseract 실행 파일 검색

subprocess
→ 터미널 명령어 실행

cv2
→ OpenCV 버전 확인

pytesseract
→ Tesseract 버전 확인

          ↓

[2. 프로젝트 최상위 폴더와 보고서 경로 설정]

현재 실행하는 Python 파일 위치를 확인한다.

src/40_day06_check_ocr_env.py

          ↓

현재 파일의 상위 폴더인 src로 이동한다.

          ↓

한 번 더 상위 폴더로 이동한다.

subject9_machinevision/

          ↓

환경 확인 결과를 저장할 경로를 만든다.

reports/day06_ocr_environment.txt

          ↓

[3. 현재 운영체제와 OpenCV 버전 확인]

환경 정보를 저장할
lines 리스트를 만든다.

          ↓

현재 운영체제 정보를 확인한다.

예:
Linux / WSL2

          ↓

운영체제 정보를 lines에 추가한다.

          ↓

OpenCV 버전을 확인한다.

          ↓

OpenCV 버전을 lines에 추가한다.

          ↓

[4. Tesseract OCR Engine 실행 파일 찾기]

shutil.which()를 이용하여
tesseract 실행 파일을 검색한다.

          ↓

Tesseract가 설치되어 있고
PATH에서 찾을 수 있다면

실행 파일의 경로를 가져온다.

          ↓

[5. Tesseract를 찾지 못한 경우]

Tesseract 실행 파일을 찾았는가?

          ├── NO
          │
          ↓
    Tesseract: NOT FOUND

    Tesseract Path: NOT FOUND

    두 메시지를 lines에 기록한다.

          │
          │
          └── 8번으로 이동

          └── YES
                │
                ↓

[6. Tesseract 설치 경로와 버전 확인]

Tesseract 실행 파일의 경로를 기록한다.

예:
 /usr/bin/tesseract

          ↓

pytesseract.get_tesseract_version()을
호출한다.

          ↓

Tesseract 버전을 정상적으로 확인했다면
버전 정보를 lines에 추가한다.

          ↓

버전을 확인하다가 오류가 발생했다면
오류 내용을 lines에 기록한다.

          ↓

[7. 설치된 OCR 언어 데이터 확인]

Python에서 터미널 명령어를 실행한다.

tesseract --list-langs

          ↓

설치된 OCR 언어 목록을 가져온다.

예:
eng
kor
osd

          ↓

언어 목록을 lines에 추가한다.

          ↓

명령 실행에 실패했다면
오류 내용을 lines에 기록한다.

          ↓

[8. OCR 환경 확인 결과를 TXT 파일로 저장]

reports 폴더가 존재하는지 확인한다.

          ↓

폴더가 없다면 생성한다.

이미 있다면 그대로 사용한다.

          ↓

lines 리스트에 저장된 결과를
줄바꿈 문자로 연결한다.

          ↓

환경 확인 결과를 TXT 파일로 저장한다.

reports/day06_ocr_environment.txt

          ↓

[9. 확인한 환경 정보와 저장 경로 출력]

lines에 저장된 정보를
터미널에 출력한다.

          ↓

운영체제 정보를 출력한다.

OpenCV 버전을 출력한다.

Tesseract 정보를 출력한다.

설치된 OCR 언어 목록을 출력한다.

          ↓

TXT 보고서의 저장 경로를 출력한다.

          ↓

[10. Tesseract 발견 여부에 따라 최종 메시지 출력]

Tesseract 실행 파일을 찾지 못했다면

OCR Environment Check Failed

를 출력한다.

          ↓

Tesseract 실행 파일을 찾았다면

OCR Environment OK

를 출력한다.

          ↓

[최종 결과]

40_day06_check_ocr_env.py
          ↓
운영체제 정보 확인
          ↓
OpenCV 버전 확인
          ↓
Tesseract 실행 파일 검색
          ↓
Tesseract 설치 경로 확인
          ↓
Tesseract 버전 확인
          ↓
OCR 언어 데이터 확인
          ↓
환경 확인 결과 저장
          ↓
day06_ocr_environment.txt
          ↓
OCR 개발환경 준비 상태 확인
          ↓
다음 실습
41_day06_select_text_roi.py
          ↓
제품 이미지 6장의
SN 숫자 영역 ROI 지정
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

## 10.2 `day06_labels.csv` — OCR 검사 정답표를 작성합니다

OCR이 제품의 SN을 정확하게 읽었는지, 제품이 정상인지 확인하려면 **비교할 정답 데이터(Ground Truth)**가 필요합니다.

이번 실습에서는 제품 이미지 6장의 정답표를 직접 작성합니다.

### ① 정답 데이터는 어디에서 가져오나요?

|항목|작성 기준|
|---|---|
|`file`|실제 제품 이미지의 파일명|
|`actual_text`|이미지에 실제 인쇄된 SN|
|`expected_text`|교육용으로 설정한 정상 기준 SN|
|`actual_status`|두 SN이 같으면 OK, 다르면 NG|

**이번 실습에서는 불량 상황을 경험하기 위해 2개 제품의 Expected SN을 일부러 다르게 설정했습니다.** 실제 불량으로 확인된 제품이라는 뜻은 아닙니다.

### ② CSV 파일을 직접 작성합니다

VS Code에서 다음 파일을 생성합니다.

```
configs/day06_labels.csv
```

아래 데이터를 복사하여 저장합니다.

```
file,actual_text,expected_text,actual_status
0637145699424_side.png,0637145699424,0637145699424,OK
0639873538899_side.png,0639873538899,0639873538899,OK
0679256889404_side.png,0679256889404,0679256889400,NG
0702739449714_side.png,0702739449714,0702739449714,OK
0716954573504_side.png,0716954573504,0716954573500,NG
0793387694868_side.png,0793387694868,0793387694868,OK
```

실제 이미지의 확장자가 `.jpg`라면 CSV에서도 `.jpg`로 수정합니다. 파일명과 실제 인쇄값이 일치하는지도 확인합니다.

**정답표 구성:** OK 4개 / 교육용 NG 2개

이 CSV는 40번 코드에서 자동 생성되지 않으며, 이후 Batch 검사와 정확도 평가에 사용됩니다.

### ③ 실제 현장에서 데이터가 수천 장이라면?

대량 데이터는 사람이 모두 직접 작성하지 않습니다.

```
제품 이미지 자동 수집
        +
MES / ERP에서 Expected SN 가져오기
        +
실제 인쇄값 검증
        ↓
Python으로 정답표 CSV 생성
        ↓
OCR 정확도 평가
```

실제 인쇄값은 사람이 확인하거나 신뢰할 수 있는 별도 기록으로 검증해야 합니다. **OCR 결과를 그대로 정답으로 사용하면 OCR 오류를 제대로 평가할 수 없습니다.**

> **핵심:** 이번 수업은 원리를 이해하기 위해 정답표 6개를 직접 작성하지만, 실제 산업현장에서는 생산관리 데이터와 프로그램을 연결하여 자동화합니다.
---

## 10.3 왜 `actual_text`와 `expected_text`를 따로 기록할까요?

**OCR이 문자를 정확히 읽는 것과 제품이 정상인 것은 서로 다르기 때문입니다.**

예를 들어 다음과 같은 제품이 있습니다.

|구분|SN 값|
|---|---|
|실제 인쇄값 (`actual_text`)|`0679256889404`|
|OCR이 읽은 값|`0679256889404`|
|정상 기준값 (`expected_text`)|`0679256889400`|

검사 결과는 다음과 같습니다.

- **OCR 검사: 성공** — 실제 인쇄된 숫자를 정확하게 읽었습니다.
    
- **제품 검사: NG** — 읽은 숫자가 정상 기준값과 다릅니다.
    

따라서 두 가지 정확도를 구분합니다.

- **OCR Accuracy:** 실제 인쇄된 문자를 정확히 읽었는가?
    
- **Inspection Accuracy:** 제품의 OK / NG를 정답대로 판정했는가?
    

이번 실습에서는 이 차이를 이해하기 위해 **일부 Expected SN을 실제 인쇄값과 다르게 설정하여 교육용 NG 상황을 만들었습니다.**

> **핵심:** OCR이 정확하게 읽었더라도 정상 기준값과 다르면 제품은 NG로 판정됩니다.

---

## 10.4 Ground Truth는 언제 사용하나요?

**Ground Truth(그라운드 트루스)**는 OCR이 정확하게 읽었는지 확인하기 위해 미리 준비한 **정답 데이터**입니다.

이번 실습에서는 `day06_labels.csv`에 정답을 기록하고, 뒤의 **Batch 검사(여러 제품 이미지를 한 번에 검사하는 작업)**에서 사용합니다.

### 검사 과정은 다음과 같습니다.

```
제품 이미지 6장
       ↓
Tesseract OCR 실행
       ↓
SN 문자 인식
       ↓
① actual_text와 비교
   → OCR이 정확하게 읽었는가?

② expected_text와 비교
   → 제품이 OK인가, NG인가?
       ↓
③ actual_status와 비교
   → 제품을 정확하게 판정했는가?
```

여기서 `actual_text`는 실제 이미지에 인쇄된 SN을 사람이 확인한 정답이고, `expected_text`는 제품의 정상 기준값입니다. `actual_status`는 제품의 정답 판정(OK / NG)입니다.

> **핵심:** Ground Truth는 OCR 성능과 검사 정확도를 평가하기 위한 데이터입니다. 실제 제조 현장에서는 OCR이 읽은 값과 생산 기준값(Expected SN)을 비교하여 제품을 검사합니다.

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
# --------------------------------------------------
# 1. 필요한 라이브러리 불러오기
#
# 무엇을 하나요?
# - JSON 파일을 저장할 기능을 불러옵니다.
# - 파일과 폴더 경로를 관리할 기능을 불러옵니다.
# - 이미지 읽기, ROI 선택, 이미지 저장 기능을 불러옵니다.
#
# json:
# Python 딕셔너리를 JSON 파일로 저장하는 라이브러리
#
# Path:
# 프로젝트의 파일과 폴더 경로를 관리하는 클래스
#
# cv2:
# OpenCV 라이브러리
# 이미지 읽기, 마우스 ROI 선택, 이미지 저장에 사용
#
# 왜 필요한가요?
# 제품 이미지에서 SN 영역을 선택하고
# 선택한 이미지와 좌표를 저장하기 위해서입니다.
# --------------------------------------------------

import json
from pathlib import Path

import cv2


# --------------------------------------------------
# 2. 프로젝트와 입력·출력 파일 경로 설정
#
# 무엇을 하나요?
# - 현재 Python 파일의 위치를 기준으로
#   프로젝트 최상위 폴더를 찾습니다.
# - 제품 이미지가 있는 폴더를 지정합니다.
# - OCR Rule JSON을 저장할 위치를 지정합니다.
# - ROI 결과 이미지를 저장할 폴더를 지정합니다.
#
# PROJECT_ROOT:
# 프로젝트 최상위 폴더
#
# PRODUCT_DIR:
# 본 실습용 제품 이미지 6장이 저장된 폴더
#
# CONFIG_PATH:
# 이미지별 ROI 좌표와 OCR 설정을 저장할 JSON 파일
#
# OUTPUT_DIR:
# 선택한 ROI와 위치 확인 이미지를 저장할 폴더
#
# 왜 필요한가요?
# 프로그램이 어떤 이미지를 읽고
# 결과를 어디에 저장해야 하는지 지정하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. 본 실습용 제품 이미지 목록 만들기
#
# 무엇을 하나요?
# - PRODUCT_DIR 폴더의 파일을 확인합니다.
# - 확장자가 jpg, jpeg, png인 이미지만 선택합니다.
# - 선택한 이미지를 파일명 순서로 정렬합니다.
#
# PRODUCT_DIR.iterdir():
# 폴더 안에 있는 파일과 폴더를 확인합니다.
#
# path.suffix.lower():
# 파일 확장자를 소문자로 가져옵니다.
#
# sorted():
# 이미지 목록을 정렬합니다.
#
# 왜 필요한가요?
# 제품 이미지 6장을 순서대로 열고
# 각 이미지의 SN 영역을 선택하기 위해서입니다.
#
# 참고:
# 이 코드는 정해진 파일명 6개가 아니라
# 폴더에 존재하는 지원 형식의 이미지를 모두 찾습니다.
# --------------------------------------------------

IMAGE_PATHS = sorted(
    [
        path
        for path in PRODUCT_DIR.iterdir()
        if path.suffix.lower()
        in {".jpg", ".jpeg", ".png"}
    ]
)


# --------------------------------------------------
# 4. 이미지 개수 확인 및 결과 저장 폴더 생성
#
# 무엇을 하나요?
# - 제품 이미지가 있는지 확인합니다.
# - 이미지가 하나도 없다면 오류를 발생시킵니다.
# - 찾은 이미지 개수를 터미널에 출력합니다.
# - ROI 결과를 저장할 폴더를 생성합니다.
#
# if not IMAGE_PATHS:
# 이미지 목록이 비어 있는지 확인합니다.
#
# FileNotFoundError:
# 필요한 이미지가 없을 때 발생시키는 오류입니다.
#
# mkdir():
# 결과를 저장할 폴더를 생성합니다.
#
# parents=True:
# 필요한 상위 폴더까지 함께 생성합니다.
#
# exist_ok=True:
# 폴더가 이미 있어도 오류를 발생시키지 않습니다.
#
# 왜 필요한가요?
# 이미지가 준비되어 있는지 확인하고
# 결과를 저장할 공간을 준비하기 위해서입니다.
#
# 주의:
# 현재 코드는 이미지가 6장인지 검사하지 않습니다.
# 한 장 이상 있으면 발견한 이미지들을 처리합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 5. ROI 좌표 저장용 딕셔너리 생성 및 이미지 읽기
#
# 무엇을 하나요?
# - 이미지별 ROI 좌표를 저장할 빈 딕셔너리를 만듭니다.
# - 제품 이미지를 한 장씩 읽습니다.
# - 이미지 읽기에 실패하면 오류를 발생시킵니다.
#
# roi_by_file:
# 이미지 파일명을 Key로 사용하여
# 해당 이미지의 ROI 좌표를 저장할 딕셔너리
#
# 예:
# {
#   "0637145699424_side.png": [434, 294, 529, 318]
# }
#
# for image_path in IMAGE_PATHS:
# 이미지 목록을 한 장씩 반복하는 코드입니다.
#
# cv2.imread():
# 이미지 파일을 읽는 함수입니다.
#
# 왜 필요한가요?
# 이미지마다 SN의 위치가 다르므로
# 각 이미지의 ROI 좌표를 따로 저장해야 하기 때문입니다.
# --------------------------------------------------

roi_by_file = {}


for image_path in IMAGE_PATHS:

    image = cv2.imread(
        str(image_path)
    )

    if image is None:
        raise FileNotFoundError(
            image_path
        )


    # --------------------------------------------------
    # 6. 제품 이미지를 열고 마우스로 SN ROI 선택
    #
    # 무엇을 하나요?
    # - 현재 검사할 이미지 이름을 출력합니다.
    # - SN 숫자 영역만 선택하라는 안내를 출력합니다.
    # - OpenCV 창을 열어 마우스로 ROI를 선택합니다.
    #
    # cv2.selectROI():
    # 이미지 위에서 마우스로 사각형 영역을 선택하는 함수
    #
    # x:
    # 선택 영역의 왼쪽 위 X좌표
    #
    # y:
    # 선택 영역의 왼쪽 위 Y좌표
    #
    # w:
    # 선택 영역의 너비(Width)
    #
    # h:
    # 선택 영역의 높이(Height)
    #
    # fromCenter=False:
    # 중심점이 아니라 모서리 기준으로 드래그합니다.
    #
    # showCrosshair=True:
    # ROI를 선택할 때 십자선을 표시합니다.
    #
    # 왜 필요한가요?
    # 전체 제품 이미지에서 필요한 13자리 SN 숫자만
    # OCR에 입력하도록 영역을 제한하기 위해서입니다.
    #
    # 선택 방법:
    # ① 마우스로 13자리 숫자 영역을 드래그합니다.
    # ② ENTER 또는 SPACE를 누릅니다.
    #
    # 주의:
    # 가능한 한 "SN:" 문자는 제외하고
    # 13자리 숫자가 모두 포함되도록 선택합니다.
    # --------------------------------------------------

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

    # ROI 선택 창 닫기
    cv2.destroyAllWindows()


    # --------------------------------------------------
    # 7. ROI 선택 여부 확인 및 좌표 변환
    #
    # 무엇을 하나요?
    # - 마우스로 영역을 정상적으로 선택했는지 확인합니다.
    # - 선택 영역의 너비 또는 높이가 0이면 중단합니다.
    # - x, y, w, h를 x1, y1, x2, y2로 변환합니다.
    #
    # 선택할 때 받은 값:
    #
    # x = 시작 X좌표
    # y = 시작 Y좌표
    # w = 너비
    # h = 높이
    #
    # Crop할 때 사용할 값:
    #
    # x1 = 왼쪽 X좌표
    # y1 = 위쪽 Y좌표
    # x2 = 오른쪽 X좌표
    # y2 = 아래쪽 Y좌표
    #
    # 계산 방법:
    #
    # x1 = x
    # y1 = y
    # x2 = x + w
    # y2 = y + h
    #
    # 왜 필요한가요?
    # NumPy 이미지 배열을 자를 때
    # 시작 위치와 끝 위치가 필요하기 때문입니다.
    # --------------------------------------------------

    if w == 0 or h == 0:
        raise RuntimeError(
            f"ROI가 선택되지 않았습니다: "
            f"{image_path.name}"
        )


    x1 = int(x)
    y1 = int(y)
    x2 = int(x + w)
    y2 = int(y + h)


    # --------------------------------------------------
    # 8. 선택한 SN 영역만 ROI Crop
    #
    # 무엇을 하나요?
    # - 원본 이미지에서 선택한 영역만 잘라냅니다.
    # - 13자리 SN 숫자 부분의 이미지를 만듭니다.
    #
    # image[y1:y2, x1:x2]:
    # 이미지 배열에서 지정한 범위만 추출합니다.
    #
    # 주의:
    # NumPy 이미지 배열의 인덱스 순서는
    # [Y좌표, X좌표]입니다.
    #
    # 즉:
    # [세로 범위, 가로 범위]
    #
    # 왜 필요한가요?
    # Tesseract가 전체 제품 이미지가 아니라
    # 실제로 검사해야 하는 숫자 부분에만
    # 집중하도록 하기 위해서입니다.
    #
    # 현재는 아직 OCR을 실행하지 않습니다.
    # --------------------------------------------------

    roi = image[
        y1:y2,
        x1:x2,
    ]


    # --------------------------------------------------
    # 9. 원본 이미지에 ROI 위치 사각형 표시
    #
    # 무엇을 하나요?
    # - 원본 이미지의 복사본을 만듭니다.
    # - 선택한 SN 영역을 초록색 사각형으로 표시합니다.
    #
    # image.copy():
    # 원본 이미지를 변경하지 않도록 복사합니다.
    #
    # cv2.rectangle():
    # 이미지 위에 사각형을 그리는 함수입니다.
    #
    # (x1, y1):
    # 사각형의 왼쪽 위 좌표
    #
    # (x2, y2):
    # 사각형의 오른쪽 아래 좌표
    #
    # (0, 255, 0):
    # OpenCV의 BGR 기준 초록색
    #
    # 3:
    # 사각형 선의 두께
    #
    # 왜 필요한가요?
    # 학생들이 실제 선택한 ROI 위치가
    # 올바른지 결과 이미지를 보고 확인하기 위해서입니다.
    # --------------------------------------------------

    preview = image.copy()

    cv2.rectangle(
        preview,
        (x1, y1),
        (x2, y2),
        (0, 255, 0),
        3,
    )


    # --------------------------------------------------
    # 10. ROI Preview와 실제 SN Crop 이미지 저장
    #
    # 무엇을 하나요?
    # - 두 종류의 결과 이미지 경로를 만듭니다.
    # - 선택 위치가 표시된 Preview 이미지를 저장합니다.
    # - 숫자 영역만 잘라낸 SN ROI 이미지를 저장합니다.
    #
    # image_path.stem:
    # 파일의 확장자를 제외한 이름
    #
    # 예:
    # 0637145699424_side.png
    #
    # stem:
    # 0637145699424_side
    #
    # 저장 결과:
    #
    # *_roi_preview.jpg
    # → 원본 이미지에 ROI 사각형을 표시한 이미지
    #
    # *_sn_roi.jpg
    # → 실제 숫자 영역만 잘라낸 이미지
    #
    # cv2.imwrite():
    # 이미지를 파일로 저장하는 함수입니다.
    #
    # 왜 두 장을 저장하나요?
    #
    # Preview:
    # ROI 위치를 확인하기 위해서입니다.
    #
    # SN ROI:
    # 다음 전처리 및 Tesseract OCR 실습에 사용합니다.
    # --------------------------------------------------

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


    # --------------------------------------------------
    # 11. 이미지별 ROI 좌표를 딕셔너리에 기록
    #
    # 무엇을 하나요?
    # - 현재 이미지 파일명을 Key로 사용합니다.
    # - 선택한 ROI의 네 좌표를 리스트로 저장합니다.
    # - 저장된 좌표를 터미널에 출력합니다.
    #
    # 예:
    #
    # "0637145699424_side.png":
    #     [434, 294, 529, 318]
    #
    # 좌표 순서:
    # [x1, y1, x2, y2]
    #
    # 왜 필요한가요?
    # 이후 Batch OCR에서는 같은 이미지를 다시 열어
    # 마우스로 영역을 선택하지 않고,
    # 저장된 좌표를 읽어 자동 Crop하기 위해서입니다.
    #
    # 이 작업은 모든 제품 이미지에 반복됩니다.
    # --------------------------------------------------

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


# --------------------------------------------------
# 12. Day06 OCR Rule 초안(Draft) 생성
#
# 무엇을 하나요?
# - 앞에서 선택한 이미지별 ROI 좌표를 저장합니다.
# - 앞으로 OCR에서 사용할 기본 설정을 준비합니다.
# - 아직 전처리를 확정하지 않은 Draft 상태로 만듭니다.
#
# baseline_version:
# 현재 OCR Rule의 버전
#
# day06-ocr-draft:
# 아직 최종 확정되지 않은 초안이라는 뜻
#
# reference_file:
# 대표 OCR 실험에 사용할 첫 번째 이미지
#
# roi_by_file:
# 이미지별 SN ROI 좌표
#
# preprocess_mode:
# 아직 전처리 방법을 선택하지 않았으므로
# NOT_SELECTED로 설정
#
# clahe_clip_limit:
# CLAHE에서 대비 강조 정도를 제한하는 값
#
# clahe_grid:
# CLAHE에서 이미지를 나누는 격자 크기
#
# ocr_lang:
# OCR에 사용할 언어 데이터
#
# ocr_oem:
# Tesseract OCR Engine Mode
#
# ocr_psm:
# Tesseract Page Segmentation Mode
# 7은 한 줄짜리 문자로 인식하도록 지정
#
# whitelist:
# 허용할 문자 종류
# 이번 SN 실습에서는 숫자 0~9만 허용
#
# expected_pattern:
# 읽은 문자열이 정확히 숫자 13자리인지 검사하는 Regex
#
# allow_context_correction:
# OCR이 인식한 숫자를 자동으로 다른 숫자로 변경할지 여부
# 이번 실습에서는 False
#
# 왜 필요한가요?
# 다음 전처리 비교와 OCR 실험에서
# 공통으로 사용할 설정을 준비하기 위해서입니다.
#
# 주의:
# 이 단계에서는 아직 최종 OCR Rule이 아닙니다.
# 이후 실험에서 전처리 방법을 결정합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 13. OCR Rule 초안을 JSON 파일로 저장
#
# 무엇을 하나요?
# - configs 폴더가 있는지 확인합니다.
# - 폴더가 없으면 생성합니다.
# - rules 딕셔너리를 JSON 문자열로 변환합니다.
# - JSON 파일로 저장합니다.
#
# json.dumps():
# Python 딕셔너리를 JSON 문자열로 변환합니다.
#
# ensure_ascii=False:
# 한글을 읽을 수 있는 형태로 저장합니다.
#
# indent=2:
# 들여쓰기를 적용하여 내용을 읽기 쉽게 만듭니다.
#
# write_text():
# JSON 문자열을 파일로 저장합니다.
#
# 저장 파일:
# configs/day06_ocr_rules.json
#
# 왜 필요한가요?
# 이후 실습에서 같은 ROI 좌표와 OCR 설정을
# 반복하여 사용할 수 있도록 하기 위해서입니다.
#
# 주의:
# 같은 파일이 이미 있으면 기존 내용을 덮어씁니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 14. ROI 선택 및 OCR Rule 저장 완료 메시지 출력
#
# 무엇을 하나요?
# - JSON 파일 저장 경로를 출력합니다.
# - ROI 준비 작업이 완료되었다는 메시지를 출력합니다.
#
# 왜 필요한가요?
# 다음 실습에서 사용할 입력 데이터와
# OCR 설정 파일이 준비되었음을 확인하기 위해서입니다.
#
# 다음 실습:
# 42_day06_preprocess_compare.py
#
# 대표 이미지의 SN ROI에
# Original / Gray / CLAHE / Otsu / CLAHE+Otsu
# 다섯 가지 전처리를 적용합니다.
# --------------------------------------------------

print()
print(
    "Saved:",
    CONFIG_PATH
)

print(
    "Day06 SN ROI Setup Complete"
)
```
### 의사코드

```
[1. 필요한 라이브러리 불러오기]

json 라이브러리를 불러온다.
→ OCR 설정을 JSON 파일로 저장

Path를 불러온다.
→ 파일과 폴더 경로 관리

cv2를 불러온다.
→ 이미지 읽기, ROI 선택, 결과 이미지 저장

          ↓

[2. 프로젝트와 입력·출력 파일 경로 설정]

현재 실행하는 Python 파일의 위치를 확인한다.

src/41_day06_select_text_roi.py

          ↓

상위 폴더로 두 번 이동하여
프로젝트 최상위 폴더를 찾는다.

          ↓

제품 이미지 폴더를 지정한다.

samples/day06/product/

          ↓

OCR Rule JSON 경로를 지정한다.

configs/day06_ocr_rules.json

          ↓

ROI 결과 이미지 저장 폴더를 지정한다.

outputs/day06/roi/

          ↓

[3. 본 실습용 제품 이미지 목록 만들기]

제품 이미지 폴더 안의 파일을 확인한다.

          ↓

확장자가 다음 중 하나인지 확인한다.

.jpg
.jpeg
.png

          ↓

해당 확장자의 이미지들만 선택한다.

          ↓

파일명 순서대로 정렬한다.

          ↓

IMAGE_PATHS에 이미지 목록을 저장한다.

          ↓

[4. 이미지 개수 확인 및 결과 저장 폴더 생성]

IMAGE_PATHS가 비어 있는지 확인한다.

          ↓

이미지가 하나도 없다면
FileNotFoundError를 발생시킨다.

          ↓

이미지가 있다면
전체 이미지 개수를 출력한다.

          ↓

ROI 결과 저장 폴더가 없으면 생성한다.

          ↓

[5. ROI 좌표 저장용 딕셔너리 생성 및 이미지 읽기]

roi_by_file이라는 빈 딕셔너리를 만든다.

          ↓

IMAGE_PATHS에서 첫 번째 이미지를 가져온다.

          ↓

OpenCV로 이미지를 읽는다.

          ↓

이미지를 읽지 못했다면
FileNotFoundError를 발생시킨다.

          ↓

이미지를 정상적으로 읽었다면
다음 단계로 진행한다.

          ↓

[6. 제품 이미지를 열고 마우스로 SN ROI 선택]

현재 이미지 이름을 출력한다.

          ↓

OpenCV 이미지 창을 연다.

          ↓

학생이 SN의 13자리 숫자 영역을
마우스로 드래그하여 선택한다.

          ↓

ENTER 또는 SPACE를 누른다.

          ↓

선택한 사각형의 정보를 가져온다.

x = 시작 X좌표
y = 시작 Y좌표
w = 선택 영역 너비
h = 선택 영역 높이

          ↓

ROI 선택 창을 닫는다.

          ↓

[7. ROI 선택 여부 확인 및 좌표 변환]

선택 영역의 너비와 높이를 확인한다.

          ↓

너비 또는 높이가 0이면
ROI 선택 오류를 발생시킨다.

          ↓

정상적으로 선택했다면
좌표를 변환한다.

x1 = x
y1 = y

x2 = x + w
y2 = y + h

          ↓

ROI의 왼쪽 위 좌표와
오른쪽 아래 좌표를 준비한다.

          ↓

[8. 선택한 SN 영역만 ROI Crop]

원본 이미지에서
선택한 좌표 범위만 가져온다.

          ↓

image[y1:y2, x1:x2]

          ↓

SN의 13자리 숫자 영역만
잘라낸 ROI 이미지를 만든다.

          ↓

[9. 원본 이미지에 ROI 위치 사각형 표시]

원본 이미지의 복사본을 만든다.

          ↓

선택한 ROI의 네 좌표를 사용하여
초록색 사각형을 그린다.

          ↓

ROI 선택 위치를 확인할
Preview 이미지를 만든다.

          ↓

[10. ROI Preview와 실제 SN Crop 이미지 저장]

첫 번째 결과 파일명을 만든다.

*_roi_preview.jpg

          ↓

원본 이미지에 ROI 위치가 표시된
Preview 이미지를 저장한다.

          ↓

두 번째 결과 파일명을 만든다.

*_sn_roi.jpg

          ↓

숫자 영역만 잘라낸
SN ROI 이미지를 저장한다.

          ↓

[11. 이미지별 ROI 좌표를 딕셔너리에 기록]

현재 이미지 파일명을 가져온다.

          ↓

이미지 파일명을 Key로 사용한다.

          ↓

선택한 ROI 좌표를 기록한다.

[x1, y1, x2, y2]

          ↓

roi_by_file 딕셔너리에 저장한다.

          ↓

저장된 ROI 좌표를 출력한다.

          ↓

아직 처리할 제품 이미지가 남아 있다면
5번으로 돌아간다.

          ↓

다음 제품 이미지에서
5~11번 과정을 반복한다.

          ↓

모든 제품 이미지의 ROI 선택이 끝나면
12번으로 이동한다.

          ↓

[12. Day06 OCR Rule 초안 생성]

rules 딕셔너리를 만든다.

          ↓

Baseline 버전을 설정한다.

day06-ocr-draft

          ↓

대표 이미지 파일명을 기록한다.

reference_file

          ↓

제품 이미지별 ROI 좌표를 기록한다.

roi_by_file

          ↓

아직 전처리 방법을 선택하지 않았으므로

preprocess_mode = NOT_SELECTED

로 설정한다.

          ↓

CLAHE 기본 설정을 준비한다.

clahe_clip_limit = 2.0
clahe_grid = [8, 8]

          ↓

OCR 언어를 설정한다.

ocr_lang = eng

          ↓

Tesseract Engine Mode를 설정한다.

ocr_oem = 3

          ↓

한 줄 문자로 인식하도록 설정한다.

ocr_psm = 7

          ↓

숫자만 허용하도록 설정한다.

whitelist = 0123456789

          ↓

정확히 13자리 숫자인지 검사할
정규식을 준비한다.

expected_pattern = ^[0-9]{13}$

          ↓

OCR 숫자를 자동으로 바꾸지 않도록 설정한다.

allow_context_correction = False

          ↓

[13. OCR Rule 초안을 JSON 파일로 저장]

configs 폴더가 있는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

rules 딕셔너리를
JSON 문자열로 변환한다.

          ↓

한글을 읽을 수 있도록 저장하고
들여쓰기를 적용한다.

          ↓

다음 파일에 저장한다.

configs/day06_ocr_rules.json

          ↓

[14. ROI 선택 및 OCR Rule 저장 완료 메시지 출력]

JSON 저장 경로를 출력한다.

          ↓

다음 완료 메시지를 출력한다.

Day06 SN ROI Setup Complete

          ↓

[최종 결과]

제품 이미지 6장
        ↓
이미지 목록 확인
        ↓
첫 번째 제품 이미지 열기
        ↓
마우스로 13자리 SN 선택
        ↓
ROI 좌표 계산
        ↓
SN 숫자 영역 Crop
        ↓
ROI 위치 Preview 만들기
        ↓
두 이미지 저장
        ↓
이미지별 ROI 좌표 기록
        ↓
다음 이미지 반복
        ↓
6장 모두 ROI 지정
        ↓
OCR Rule Draft 생성
        ↓
day06_ocr_rules.json 저장
        ↓
다음 실습
42_day06_preprocess_compare.py
        ↓
SN ROI에 다섯 가지 전처리 적용
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

`41_day06_select_text_roi.py`를 실행하면 ROI 좌표뿐만 아니라 앞으로 OCR에서 사용할 
기본 설정도 다음 파일에 저장됩니다.

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
# --------------------------------------------------
# 1. OCR 공통 처리에 필요한 라이브러리 불러오기
#
# 무엇을 하나요?
# - JSON 파일을 읽는 기능을 불러옵니다.
# - 프로젝트 파일과 폴더 경로를 관리합니다.
# - 이미지 처리와 문자 인식 기능을 불러옵니다.
#
# json:
# JSON 파일에 저장된 OCR Rule을 읽기 위해 사용합니다.
#
# Path:
# 파일과 폴더 경로를 관리합니다.
#
# cv2:
# OpenCV 라이브러리입니다.
# ROI 전처리, Gray, CLAHE, Otsu 등에 사용합니다.
#
# pytesseract:
# Python에서 Tesseract OCR Engine을 호출합니다.
#
# 왜 필요한가요?
# 이후 여러 OCR 실습에서 공통으로 사용할 기능을
# 하나의 Python 파일에 모아 두기 위해서입니다.
# --------------------------------------------------

import json
from pathlib import Path

import cv2
import pytesseract


# --------------------------------------------------
# 2. 프로젝트 최상위 폴더와 OCR Rule 경로 설정
#
# 무엇을 하나요?
# - 현재 Python 파일의 위치를 확인합니다.
# - 프로젝트 최상위 폴더를 찾습니다.
# - OCR Rule이 저장된 JSON 파일 경로를 지정합니다.
#
# __file__:
# 현재 Python 파일의 경로입니다.
#
# resolve():
# 파일의 절대경로를 구합니다.
#
# parent.parent:
# 현재 파일의 상위 폴더로 두 번 이동합니다.
#
# 예:
#
# subject9_machinevision/
# ├── src/
# │   └── day06_ocr_utils.py
# └── configs/
#     └── day06_ocr_rules.json
#
# DEFAULT_RULES_PATH:
# 41번 실습에서 저장한 OCR 설정 JSON 경로입니다.
#
# 왜 필요한가요?
# 이후 OCR 실습에서 동일한 JSON 설정을
# 반복해서 사용하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. JSON에 저장된 OCR Rule을 읽는 함수
#
# 무엇을 하나요?
# - load_rules() 함수를 정의합니다.
# - JSON 파일에 저장된 OCR 설정을 읽습니다.
# - JSON 데이터를 Python 딕셔너리로 변환합니다.
#
# path=DEFAULT_RULES_PATH:
# 별도의 파일 경로를 전달하지 않으면
# 기본 OCR Rule JSON을 사용합니다.
#
# Path(path).read_text():
# JSON 파일의 내용을 문자열로 읽습니다.
#
# json.loads():
# JSON 문자열을 Python 딕셔너리로 변환합니다.
#
# 반환하는 정보:
#
# baseline_version
# reference_file
# roi_by_file
# preprocess_mode
# clahe_clip_limit
# clahe_grid
# ocr_lang
# ocr_oem
# ocr_psm
# whitelist
# expected_pattern
# allow_context_correction
#
# 왜 필요한가요?
# Python 코드에 OCR 설정을 매번 직접 작성하지 않고
# JSON에 저장된 공통 설정을 가져오기 위해서입니다.
#
# 주의:
# JSON 파일이 없거나 내용이 잘못되면
# 읽는 과정에서 오류가 발생합니다.
# --------------------------------------------------

def load_rules(
    path=DEFAULT_RULES_PATH,
):
    return json.loads(
        Path(path).read_text(
            encoding="utf-8"
        )
    )


# --------------------------------------------------
# 4. 이미지 파일명에 해당하는 ROI 좌표 가져오기
#
# 무엇을 하나요?
# - JSON의 roi_by_file 정보를 가져옵니다.
# - 현재 이미지의 파일명을 확인합니다.
# - 해당 이미지에 저장된 ROI 좌표를 반환합니다.
#
# rules:
# JSON에서 읽은 OCR 설정 딕셔너리입니다.
#
# file_name:
# 현재 OCR할 이미지의 파일명입니다.
#
# roi_by_file:
# 각 이미지의 SN 영역 좌표가 저장된 딕셔너리입니다.
#
# 예:
#
# "0637145699424_side.png":
#     [434, 294, 529, 318]
#
# 파일명을 찾지 못했다면:
# KeyError를 발생시킵니다.
#
# 왜 필요한가요?
# 제품마다 SN의 위치가 서로 다르기 때문에
# 현재 이미지에 맞는 ROI 좌표를 가져와야 합니다.
#
# 이 함수는 이미지 자체를 잘라내지 않습니다.
# ROI 좌표만 가져오는 역할입니다.
# --------------------------------------------------

def get_roi_for_file(
    rules,
    file_name,
):

    # JSON에서 이미지별 ROI 좌표 목록 가져오기
    roi_by_file = rules[
        "roi_by_file"
    ]

    # 현재 이미지의 ROI 좌표가 있는지 확인
    if file_name not in roi_by_file:
        raise KeyError(
            f"ROI 정보가 없습니다: {file_name}"
        )

    # 현재 이미지에 해당하는 ROI 좌표 반환
    return roi_by_file[
        file_name
    ]


# --------------------------------------------------
# 5. ROI 좌표를 이용하여 SN 문자 영역 Crop
#
# 무엇을 하나요?
# - 원본 이미지와 ROI 좌표를 전달받습니다.
# - ROI 좌표를 네 개의 변수로 나눕니다.
# - 이미지에서 해당 부분만 잘라냅니다.
#
# image:
# OpenCV로 읽은 원본 이미지입니다.
#
# roi:
# [x1, y1, x2, y2] 형태의 좌표 리스트입니다.
#
# x1:
# 왼쪽 X좌표
#
# y1:
# 위쪽 Y좌표
#
# x2:
# 오른쪽 X좌표
#
# y2:
# 아래쪽 Y좌표
#
# image[y1:y2, x1:x2]:
# 세로와 가로 범위를 지정하여 이미지를 자릅니다.
#
# 주의:
# NumPy 이미지 배열은 [Y, X] 순서입니다.
#
# 왜 필요한가요?
# 제품 전체를 OCR하지 않고
# 필요한 13자리 SN 숫자 부분만 인식하기 위해서입니다.
#
# 이 함수는 전달받은 좌표의 유효성을
# 별도로 검사하지 않습니다.
# --------------------------------------------------

def crop_roi(
    image,
    roi,
):

    # ROI 좌표를 네 개의 변수로 분리
    x1, y1, x2, y2 = roi

    # 선택한 SN 영역만 잘라서 반환
    return image[
        y1:y2,
        x1:x2,
    ]


# --------------------------------------------------
# 6. Grayscale 이미지에 CLAHE 전처리를 적용하는 함수
#
# 무엇을 하나요?
# - Gray 이미지에 CLAHE를 적용합니다.
# - JSON에 저장된 CLAHE 설정을 사용합니다.
# - 대비가 조정된 이미지를 반환합니다.
#
# CLAHE란?
# Contrast Limited Adaptive Histogram Equalization
#
# 이미지의 작은 영역별로 대비를 조정하여
# 흐린 문자와 배경의 밝기 차이를 개선하는 방법입니다.
#
# gray:
# 흑백으로 변환된 이미지입니다.
#
# rules:
# CLAHE 설정이 저장된 OCR Rule 딕셔너리입니다.
#
# clahe_clip_limit:
# 대비가 지나치게 강조되는 것을 제한하는 값
#
# clahe_grid:
# 이미지의 대비를 계산할 영역 구분 설정
#
# cv2.createCLAHE():
# CLAHE 처리 객체를 생성합니다.
#
# clahe.apply():
# Gray 이미지에 CLAHE를 실제로 적용합니다.
#
# 왜 필요한가요?
# 대비가 낮은 문자에서
# CLAHE가 OCR에 도움이 되는지 비교하기 위해서입니다.
#
# 주의:
# CLAHE가 항상 OCR 성능을 높이는 것은 아닙니다.
# 실제 OCR 결과를 비교하여 판단해야 합니다.
# --------------------------------------------------

def apply_clahe(
    gray,
    rules,
):

    # JSON의 설정값으로 CLAHE 객체 생성
    clahe = cv2.createCLAHE(
        clipLimit=float(
            rules["clahe_clip_limit"]
        ),
        tileGridSize=tuple(
            rules["clahe_grid"]
        ),
    )

    # Gray 이미지에 CLAHE 적용
    return clahe.apply(
        gray
    )


# --------------------------------------------------
# 7. 동일한 SN ROI에서 다섯 가지 전처리 후보 생성
#
# 무엇을 하나요?
# - 하나의 SN ROI를 입력받습니다.
# - 서로 다른 다섯 가지 이미지 상태를 만듭니다.
#
# 생성할 전처리 후보:
#
# ① Original
#    원본 ROI를 그대로 사용
#
# ② Gray
#    색상 정보를 제거한 흑백 이미지
#
# ③ CLAHE
#    Gray 이미지의 국소 대비 개선
#
# ④ Otsu
#    Gray 이미지에 자동 이진화 적용
#
# ⑤ CLAHE + Otsu
#    대비 개선 후 자동 이진화 적용
#
# 왜 필요한가요?
# 같은 SN 이미지라도 전처리 방법에 따라
# Tesseract OCR 결과가 달라질 수 있기 때문입니다.
#
# 이 함수에서는 아직 OCR을 실행하지 않습니다.
# OCR에 입력할 이미지 후보만 생성합니다.
# --------------------------------------------------

def make_preprocess_variants(
    roi,
    rules,
):

    # --------------------------------------------------
    # 7-1. 원본 ROI를 Grayscale로 변환
    #
    # cv2.cvtColor():
    # 이미지 색상 형식을 변환하는 함수
    #
    # cv2.COLOR_BGR2GRAY:
    # BGR 컬러 이미지를 Gray로 변환
    #
    # 왜 필요한가요?
    # CLAHE와 Otsu를 적용할 흑백 이미지를
    # 준비하기 위해서입니다.
    # --------------------------------------------------

    # Gray
    gray = cv2.cvtColor(
        roi,
        cv2.COLOR_BGR2GRAY,
    )


    # --------------------------------------------------
    # 7-2. Gray 이미지에 CLAHE 적용
    #
    # 앞에서 작성한 apply_clahe() 함수를 호출합니다.
    #
    # Gray:
    # 색상 정보가 제거된 이미지
    #
    # CLAHE:
    # Gray 이미지에서 국소 대비를 개선한 이미지
    #
    # 왜 필요한가요?
    # 흐린 숫자를 더 잘 구분할 수 있도록
    # 대비를 개선한 OCR 후보를 만들기 위해서입니다.
    # --------------------------------------------------

    # CLAHE
    clahe = apply_clahe(
        gray,
        rules,
    )


    # --------------------------------------------------
    # 7-3. Gray 이미지에 Otsu 이진화 적용
    #
    # 무엇을 하나요?
    # - Gray 이미지에서 자동으로 Threshold를 계산합니다.
    # - 픽셀을 검정 또는 흰색으로 구분합니다.
    #
    # cv2.threshold():
    # Threshold 기준으로 이미지를 변환하는 함수
    #
    # cv2.THRESH_BINARY:
    # 이진화 적용
    #
    # cv2.THRESH_OTSU:
    # Otsu 방법으로 Threshold 자동 계산
    #
    # _, otsu:
    # 첫 번째 반환값인 Threshold는 사용하지 않고
    # 두 번째 반환값인 이진화 이미지만 저장합니다.
    #
    # 왜 필요한가요?
    # 문자와 배경을 이진화하면
    # OCR 결과가 어떻게 달라지는지 확인하기 위해서입니다.
    # --------------------------------------------------

    # Gray + Otsu
    _, otsu = cv2.threshold(
        gray,
        0,
        255,
        cv2.THRESH_BINARY
        + cv2.THRESH_OTSU,
    )


    # --------------------------------------------------
    # 7-4. CLAHE 이미지에 Otsu 이진화 적용
    #
    # 무엇을 하나요?
    # - 앞에서 CLAHE를 적용한 이미지를 사용합니다.
    # - 그 이미지에 Otsu 이진화를 다시 적용합니다.
    #
    # 처리 순서:
    #
    # Gray
    #   ↓
    # CLAHE
    #   ↓
    # Otsu
    #
    # 왜 필요한가요?
    # 대비를 먼저 개선한 후 이진화하는 방법이
    # OCR에 도움이 되는지 비교하기 위해서입니다.
    # --------------------------------------------------

    # CLAHE + Otsu
    _, clahe_otsu = cv2.threshold(
        clahe,
        0,
        255,
        cv2.THRESH_BINARY
        + cv2.THRESH_OTSU,
    )


    # --------------------------------------------------
    # 7-5. 다섯 가지 전처리 후보를 딕셔너리로 반환
    #
    # 무엇을 하나요?
    # - 전처리 방법 이름을 Key로 사용합니다.
    # - 해당 전처리 결과 이미지를 Value로 저장합니다.
    #
    # original:
    # 원본 SN ROI
    #
    # gray:
    # 흑백 이미지
    #
    # clahe:
    # CLAHE 적용 이미지
    #
    # otsu:
    # Gray + Otsu 이미지
    #
    # clahe_otsu:
    # CLAHE + Otsu 이미지
    #
    # 왜 필요한가요?
    # 이후 다른 코드에서 전처리 이름으로
    # 필요한 이미지를 쉽게 가져오기 위해서입니다.
    # --------------------------------------------------

    return {
        "original": roi,
        "gray": gray,
        "clahe": clahe,
        "otsu": otsu,
        "clahe_otsu": clahe_otsu,
    }


# --------------------------------------------------
# 8. JSON에서 선택한 전처리 한 가지 가져오기
#
# 무엇을 하나요?
# - 앞에서 만든 다섯 가지 전처리 후보를 준비합니다.
# - JSON의 preprocess_mode 값을 읽습니다.
# - 선택된 전처리 결과 하나를 반환합니다.
#
# preprocess_mode:
# OCR에 사용할 전처리 방법 이름
#
# 예:
# preprocess_mode = "original"
#
# 선택 가능한 값:
#
# original
# gray
# clahe
# otsu
# clahe_otsu
#
# preprocess_mode = NOT_SELECTED이면?
# 사용할 전처리가 아직 확정되지 않았으므로
# ValueError를 발생시킵니다.
#
# 왜 필요한가요?
# OCR 비교 실험에서 선택한 방법을
# 이후 Batch 검사에서도 동일하게 사용하기 위해서입니다.
#
# 중요:
# 현재 코드는 다섯 가지 후보를 먼저 모두 계산하고
# 그중 선택된 하나를 반환합니다.
#
# 따라서 original을 선택하더라도
# 나머지 전처리 계산은 내부적으로 수행합니다.
# --------------------------------------------------

def preprocess_for_ocr(
    roi,
    rules,
):

    # 다섯 가지 전처리 후보 생성
    variants = (
        make_preprocess_variants(
            roi,
            rules,
        )
    )

    # JSON에서 선택된 전처리 방법 확인
    mode = rules[
        "preprocess_mode"
    ]

    # 선택한 방법이 후보 목록에 있는지 확인
    if mode not in variants:
        raise ValueError(
            "아직 사용할 전처리가 "
            f"선택되지 않았습니다: {mode}"
        )

    # 선택된 전처리 결과 이미지 반환
    return variants[
        mode
    ]


# --------------------------------------------------
# 9. Tesseract OCR 실행 옵션을 문자열로 만드는 함수
#
# 무엇을 하나요?
# - JSON에 저장된 Tesseract 설정을 가져옵니다.
# - OCR Engine Mode를 설정합니다.
# - Page Segmentation Mode를 설정합니다.
# - 인식할 문자 종류를 제한합니다.
# - 최종 Tesseract 옵션 문자열을 반환합니다.
#
# ocr_oem:
# Tesseract OCR Engine Mode
#
# ocr_psm:
# Tesseract Page Segmentation Mode
#
# 이번 실습:
# PSM = 7
# → 입력 이미지가 한 줄짜리 문자라고 지정
#
# whitelist:
# 인식할 문자 종류를 제한합니다.
#
# 이번 실습:
# 0123456789
# → 숫자만 허용
#
# 생성되는 설정 예:
#
# --oem 3 --psm 7 -c tessedit_char_whitelist=0123456789
#
# 왜 필요한가요?
# 여러 OCR 코드에서 같은 Tesseract 설정을
# 반복해서 작성하지 않기 위해서입니다.
#
# 참고:
# 이 함수는 OCR을 실행하는 함수가 아닙니다.
# Tesseract에 전달할 옵션 문자열만 생성합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 10. Tesseract OCR을 실행하여 문자열 반환
#
# 무엇을 하나요?
# - 전처리가 완료된 이미지를 전달받습니다.
# - 앞에서 작성한 build_ocr_config()를 호출합니다.
# - OCR 실행 옵션을 준비합니다.
# - pytesseract를 이용하여 문자를 인식합니다.
# - 인식한 문자열을 반환합니다.
#
# image:
# Tesseract에 입력할 이미지
#
# rules:
# JSON에서 읽은 OCR 설정
#
# config:
# Tesseract 실행 옵션 문자열
#
# pytesseract.image_to_string():
# 이미지에서 문자를 인식하여 문자열로 반환합니다.
#
# lang=rules["ocr_lang"]:
# 사용할 OCR 언어 데이터를 지정합니다.
#
# config=config:
# 앞에서 준비한 Tesseract 실행 옵션을 적용합니다.
#
# 왜 필요한가요?
# Python에서 실제 Tesseract OCR Engine을 호출하여
# 이미지 속 13자리 SN 숫자를 읽기 위해서입니다.
#
# 반환 예:
#
# '0637145699424\n'
#
# 주의:
# 반환되는 값은 Raw OCR 문자열입니다.
# 마지막에 줄바꿈 문자나 공백이 포함될 수 있습니다.
#
# 문자열 정리, 13자리 Regex 검사,
# Expected SN 비교와 OK/NG 판정은
# 이후 day06_ocr_validation.py에서 수행합니다.
# --------------------------------------------------

def run_ocr(
    image,
    rules,
):

    # Tesseract OCR 실행 옵션 생성
    config = (
        build_ocr_config(
            rules
        )
    )

    # 실제 OCR 실행 후 Raw 문자열 반환
    return pytesseract.image_to_string(
        image,
        lang=rules["ocr_lang"],
        config=config,
    )
```

### 이 공통 파일은 무슨 일을 하나요?

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
[1. OCR 공통 처리에 필요한 라이브러리 불러오기]

json을 불러온다.
→ OCR Rule JSON 읽기

Path를 불러온다.
→ 파일과 폴더 경로 관리

cv2를 불러온다.
→ 이미지 전처리

pytesseract를 불러온다.
→ Tesseract OCR 실행

          ↓

[2. 프로젝트 최상위 폴더와 OCR Rule 경로 설정]

현재 Python 파일의 위치를 확인한다.

src/day06_ocr_utils.py

          ↓

상위 폴더로 두 번 이동하여
프로젝트 최상위 폴더를 찾는다.

          ↓

OCR Rule JSON의 기본 경로를 지정한다.

configs/day06_ocr_rules.json

          ↓

[3. JSON에 저장된 OCR Rule을 읽는 함수]

load_rules() 함수를 만든다.

          ↓

OCR Rule JSON 파일 경로를 전달받는다.

별도 경로를 전달하지 않으면
기본 JSON 파일을 사용한다.

          ↓

JSON 파일을 문자열로 읽는다.

          ↓

json.loads()를 실행한다.

          ↓

JSON 내용을 Python 딕셔너리로 변환한다.

          ↓

OCR Rule 딕셔너리를 반환한다.

          ↓

[4. 이미지 파일명에 해당하는 ROI 좌표 가져오기]

get_roi_for_file() 함수를 만든다.

          ↓

OCR Rule 딕셔너리를 전달받는다.

현재 검사할 이미지 파일명을 전달받는다.

          ↓

rules에서 roi_by_file 정보를 가져온다.

          ↓

현재 이미지 파일명이
roi_by_file에 존재하는지 확인한다.

          ↓

파일명이 없다면
KeyError를 발생시킨다.

          ↓

파일명이 있다면
해당 이미지의 ROI 좌표를 반환한다.

예:
[434, 294, 529, 318]

          ↓

[5. ROI 좌표를 이용하여 SN 문자 영역 Crop]

crop_roi() 함수를 만든다.

          ↓

원본 이미지와 ROI 좌표를 전달받는다.

          ↓

ROI 좌표를 네 개의 변수로 나눈다.

x1 = 왼쪽 X좌표
y1 = 위쪽 Y좌표
x2 = 오른쪽 X좌표
y2 = 아래쪽 Y좌표

          ↓

이미지에서 지정한 영역만 잘라낸다.

image[y1:y2, x1:x2]

          ↓

13자리 SN 숫자 영역의
ROI 이미지를 반환한다.

          ↓

[6. Grayscale 이미지에 CLAHE를 적용하는 함수]

apply_clahe() 함수를 만든다.

          ↓

Gray 이미지와 OCR Rule을 전달받는다.

          ↓

JSON에서 CLAHE 설정을 읽는다.

clahe_clip_limit
clahe_grid

          ↓

설정값을 이용하여
CLAHE 객체를 생성한다.

          ↓

Gray 이미지에 CLAHE를 적용한다.

          ↓

대비가 조정된 이미지를 반환한다.

          ↓

[7. 동일한 SN ROI에서 다섯 가지 전처리 후보 생성]

make_preprocess_variants() 함수를 만든다.

          ↓

SN ROI 이미지와 OCR Rule을 전달받는다.

          ↓

[7-1. Grayscale 변환]

원본 ROI를 Gray로 변환한다.

          ↓

색상 정보가 제거된
Gray 이미지를 만든다.

          ↓

[7-2. CLAHE 적용]

apply_clahe() 함수를 호출한다.

          ↓

Gray 이미지의 국소 대비를 조정한다.

          ↓

CLAHE 이미지를 만든다.

          ↓

[7-3. Gray + Otsu 이진화]

Gray 이미지를 가져온다.

          ↓

Otsu 방법으로 Threshold를 자동 계산한다.

          ↓

픽셀을 검정 또는 흰색으로 구분한다.

          ↓

Otsu 이진화 이미지를 만든다.

          ↓

[7-4. CLAHE + Otsu 이진화]

CLAHE 이미지를 가져온다.

          ↓

CLAHE 결과에 Otsu 이진화를 적용한다.

          ↓

CLAHE + Otsu 이미지를 만든다.

          ↓

[7-5. 다섯 가지 전처리 후보 반환]

지금까지 만든 결과를
하나의 딕셔너리로 정리한다.

original
→ 원본 SN ROI

gray
→ Grayscale 이미지

clahe
→ CLAHE 적용 이미지

otsu
→ Gray + Otsu 이미지

clahe_otsu
→ CLAHE + Otsu 이미지

          ↓

전처리 후보 딕셔너리를 반환한다.

          ↓

[8. JSON에서 선택한 전처리 한 가지 가져오기]

preprocess_for_ocr() 함수를 만든다.

          ↓

SN ROI와 OCR Rule을 전달받는다.

          ↓

make_preprocess_variants()를 호출한다.

          ↓

다섯 가지 전처리 후보를 생성한다.

          ↓

JSON에서 preprocess_mode를 읽는다.

          ↓

선택한 전처리 이름이
후보 목록에 있는지 확인한다.

          ↓

선택한 전처리가 없다면
ValueError를 발생시킨다.

          ↓

선택한 전처리가 있다면
해당 이미지만 반환한다.

예:
preprocess_mode = original

          ↓

원본 SN ROI 이미지를 반환한다.

          ↓

[9. Tesseract OCR 실행 옵션을 만드는 함수]

build_ocr_config() 함수를 만든다.

          ↓

OCR Rule 딕셔너리를 전달받는다.

          ↓

OCR Engine Mode를 확인한다.

ocr_oem = 3

          ↓

Page Segmentation Mode를 확인한다.

ocr_psm = 7

          ↓

허용할 문자 종류를 확인한다.

whitelist = 0123456789

          ↓

세 설정을 하나의 문자열로 연결한다.

--oem 3
--psm 7
-c tessedit_char_whitelist=0123456789

          ↓

완성한 OCR 설정 문자열을 반환한다.

          ↓

[10. Tesseract OCR을 실행하여 문자열 반환]

run_ocr() 함수를 만든다.

          ↓

OCR에 입력할 이미지와
OCR Rule을 전달받는다.

          ↓

build_ocr_config() 함수를 호출한다.

          ↓

Tesseract 실행 옵션을 가져온다.

          ↓

pytesseract.image_to_string()을 호출한다.

          ↓

준비한 이미지를 Tesseract에 전달한다.

          ↓

Tesseract가 이미지 속 숫자를 인식한다.

          ↓

OCR 결과 문자열을 반환한다.

예:
'0637145699424\n'

          ↓

[최종 연결 흐름]

day06_ocr_rules.json
        ↓
load_rules()
        ↓
이미지별 ROI 좌표 확인
        ↓
get_roi_for_file()
        ↓
SN 영역 Crop
        ↓
crop_roi()
        ↓
전처리 후보 생성
        ↓
make_preprocess_variants()
        ↓
Original / Gray / CLAHE / Otsu / CLAHE+Otsu
        ↓
선택된 전처리 적용
        ↓
preprocess_for_ocr()
        ↓
Tesseract 옵션 설정
        ↓
build_ocr_config()
        ↓
OCR 실행
        ↓
run_ocr()
        ↓
Raw OCR 문자열 반환
        ↓
day06_ocr_validation.py
        ↓
문자열 Cleaning
        ↓
13자리 Regex 형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG 판정
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
# --------------------------------------------------
# 1. 필요한 라이브러리와 OCR 공통 함수 불러오기
#
# 무엇을 하나요?
# - 파일 경로와 이미지 처리에 필요한 기능을 불러옵니다.
# - 앞에서 작성한 day06_ocr_utils.py의 공통 함수를 가져옵니다.
#
# Path:
# 파일과 폴더 경로를 관리합니다.
#
# cv2:
# 이미지를 읽고 저장하는 OpenCV 라이브러리입니다.
#
# PROJECT_ROOT:
# 프로젝트 최상위 폴더 경로
#
# load_rules():
# JSON에 저장된 OCR 설정과 ROI 좌표를 읽는 함수
#
# get_roi_for_file():
# 이미지 파일명에 해당하는 ROI 좌표를 가져오는 함수
#
# crop_roi():
# 원본 이미지에서 SN 문자 영역만 잘라내는 함수
#
# make_preprocess_variants():
# 하나의 SN ROI에서 5가지 전처리 이미지를 만드는 함수
#
# 왜 필요한가요?
# 이전에 작성한 공통 함수를 재사용하여
# 같은 기능을 다시 코딩하지 않기 위해서입니다.
# --------------------------------------------------

from pathlib import Path

import cv2

from day06_ocr_utils import (
    PROJECT_ROOT,
    crop_roi,
    get_roi_for_file,
    load_rules,
    make_preprocess_variants,
)


# --------------------------------------------------
# 2. JSON에서 OCR Rule과 ROI 설정 읽기
#
# 무엇을 하나요?
# - load_rules() 공통 함수를 호출합니다.
# - 앞에서 저장한 day06_ocr_rules.json을 읽습니다.
# - JSON 정보를 rules 딕셔너리에 저장합니다.
#
# JSON 파일:
# configs/day06_ocr_rules.json
#
# 주요 정보:
# reference_file:
# 대표 실험에 사용할 이미지 파일명
#
# roi_by_file:
# 각 제품 이미지에 해당하는 SN ROI 좌표
#
# clahe_clip_limit:
# CLAHE 대비 조정 설정
#
# clahe_grid:
# CLAHE 영역 구분 설정
#
# 왜 필요한가요?
# 앞의 41번 실습에서 선택한 ROI 좌표를
# 다시 마우스로 지정하지 않고 사용하기 위해서입니다.
# --------------------------------------------------

rules = load_rules()


# --------------------------------------------------
# 3. 대표 이미지와 전처리 결과 저장 경로 설정
#
# 무엇을 하나요?
# - JSON에서 대표 이미지 파일명을 가져옵니다.
# - 대표 제품 이미지의 전체 경로를 만듭니다.
# - 전처리 결과 이미지를 저장할 폴더를 지정합니다.
#
# file_name:
# JSON에 기록된 대표 이미지 파일명
#
# 예:
# 0637145699424_side.png
#
# IMAGE_PATH:
# 원본 제품 이미지 전체 경로
#
# OUTPUT_DIR:
# 전처리 결과를 저장할 폴더
#
# 왜 필요한가요?
# 모든 제품을 한 번에 처리하기 전에
# 대표 이미지 한 장을 이용하여
# 전처리 결과를 비교하기 위해서입니다.
#
# 이번 실습에서는 대표 이미지 한 장만 사용합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 4. 대표 제품 이미지 읽기 및 오류 확인
#
# 무엇을 하나요?
# - OpenCV로 대표 제품 이미지를 읽습니다.
# - 이미지가 정상적으로 읽혔는지 확인합니다.
#
# cv2.imread():
# 이미지 파일을 읽는 함수입니다.
#
# str(IMAGE_PATH):
# Path 객체를 OpenCV에서 사용할 문자열 경로로 변환합니다.
#
# image:
# 읽어온 원본 제품 이미지가 저장되는 변수입니다.
#
# image is None:
# 이미지 읽기에 실패했다는 의미입니다.
#
# FileNotFoundError:
# 이미지 파일을 읽지 못한 경우 오류를 발생시킵니다.
#
# 왜 필요한가요?
# SN 영역을 Crop하기 위해서는
# 먼저 원본 제품 이미지를 정상적으로 읽어야 하기 때문입니다.
# --------------------------------------------------

image = cv2.imread(
    str(IMAGE_PATH)
)

if image is None:
    raise FileNotFoundError(
        IMAGE_PATH
    )


# --------------------------------------------------
# 5. 대표 이미지에 저장된 SN ROI 좌표 가져오기
#
# 무엇을 하나요?
# - get_roi_for_file() 함수를 호출합니다.
# - 대표 이미지 파일명과 JSON Rule을 전달합니다.
# - 해당 이미지의 ROI 좌표를 가져옵니다.
#
# 입력:
# rules:
# OCR 설정과 이미지별 ROI 정보가 저장된 딕셔너리
#
# file_name:
# 현재 대표 이미지의 파일명
#
# 반환:
# roi_coordinates
#
# 예:
# [434, 294, 529, 318]
#
# 좌표 의미:
# x1 = 434 → 왼쪽 위치
# y1 = 294 → 위쪽 위치
# x2 = 529 → 오른쪽 위치
# y2 = 318 → 아래쪽 위치
#
# 왜 필요한가요?
# 제품 전체 이미지에서 13자리 SN이 있는
# 위치를 알아야 해당 영역만 잘라낼 수 있기 때문입니다.
#
# 주의:
# 이 단계에서는 좌표만 가져오며
# 이미지를 직접 Crop하지는 않습니다.
# --------------------------------------------------

roi_coordinates = (
    get_roi_for_file(
        rules,
        file_name,
    )
)


# --------------------------------------------------
# 6. 저장된 좌표를 이용하여 SN 영역 Crop
#
# 무엇을 하나요?
# - crop_roi() 공통 함수를 호출합니다.
# - 원본 이미지와 ROI 좌표를 전달합니다.
# - 13자리 SN 숫자 영역만 잘라냅니다.
#
# image:
# 원본 제품 이미지
#
# roi_coordinates:
# JSON에서 가져온 ROI 좌표
#
# roi:
# 실제로 잘라낸 SN 문자 이미지
#
# 내부 처리:
# image[y1:y2, x1:x2]
#
# 왜 필요한가요?
# 제품 전체 이미지에는 SN 외에도
# 다른 문자나 배경이 포함되어 있기 때문입니다.
#
# 필요한 숫자 영역만 사용하면
# 이후 OCR에서 다른 문자까지 함께 읽는 문제를
# 줄일 수 있습니다.
#
# 주의:
# 현재는 OCR을 실행하지 않고
# OCR에 사용할 문자 이미지만 준비합니다.
# --------------------------------------------------

roi = crop_roi(
    image,
    roi_coordinates,
)


# --------------------------------------------------
# 7. SN ROI에 다섯 가지 전처리 적용
#
# 무엇을 하나요?
# - make_preprocess_variants() 공통 함수를 호출합니다.
# - 동일한 SN ROI에 서로 다른 5가지 전처리를 적용합니다.
# - 전처리 결과를 variants 딕셔너리에 저장합니다.
#
# ① original
# 원본 SN ROI를 그대로 사용
#
# ② gray
# BGR 컬러 이미지를 Grayscale로 변환
#
# ③ clahe
# Gray 이미지에 CLAHE를 적용하여
# 문자와 배경의 국소 대비를 조정
#
# ④ otsu
# Gray 이미지에 Otsu 자동 이진화 적용
#
# ⑤ clahe_otsu
# CLAHE로 대비를 조정한 뒤 Otsu 이진화 적용
#
# 왜 필요한가요?
# 전처리 방법에 따라 문자 획의 선명도,
# 배경과의 대비, Noise 등이 달라질 수 있기 때문입니다.
#
# 이번 단계에서는 5가지 이미지 후보만 만듭니다.
#
# 아직 어떤 전처리가 OCR에 가장 적합한지
# 결정하지 않습니다.
#
# 실제 OCR 결과는 다음 43번 실습에서 비교합니다.
# --------------------------------------------------

variants = (
    make_preprocess_variants(
        roi,
        rules,
    )
)


# --------------------------------------------------
# 8. 전처리 결과를 저장할 폴더 생성
#
# 무엇을 하나요?
# - 전처리 이미지를 저장할 폴더가 있는지 확인합니다.
# - 폴더가 없으면 생성합니다.
#
# 저장 위치:
# outputs/day06/preprocess/
#
# OUTPUT_DIR.mkdir():
# 지정한 경로에 폴더를 생성하는 함수입니다.
#
# parents=True:
# 필요한 상위 폴더까지 생성합니다.
#
# exist_ok=True:
# 이미 폴더가 존재해도 오류를 발생시키지 않습니다.
#
# 왜 필요한가요?
# 전처리 결과 이미지 5장을
# 동일한 폴더에 저장하기 위해서입니다.
# --------------------------------------------------

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 9. 다섯 가지 전처리 결과를 JPG 이미지로 저장
#
# 무엇을 하나요?
# - variants 딕셔너리의 결과를 하나씩 가져옵니다.
# - 각 이미지에 순번과 전처리 이름을 붙입니다.
# - 결과를 JPG 파일로 저장합니다.
# - 저장한 파일의 경로를 터미널에 출력합니다.
#
# variants.items():
# 전처리 이름과 결과 이미지를 함께 가져옵니다.
#
# enumerate():
# 반복할 때 순번을 함께 생성합니다.
#
# start=1:
# 순번을 1부터 시작합니다.
#
# index:
# 이미지의 저장 순번
#
# name:
# 전처리 방법 이름
#
# result:
# 실제 전처리 결과 이미지
#
# f"{index:02d}":
# 숫자를 두 자리로 표시합니다.
#
# 예:
# 1 → 01
# 2 → 02
# 3 → 03
#
# cv2.imwrite():
# 이미지를 파일로 저장합니다.
#
# 저장 결과:
#
# 01_original.jpg
# 02_gray.jpg
# 03_clahe.jpg
# 04_otsu.jpg
# 05_clahe_otsu.jpg
#
# 왜 필요한가요?
# 같은 SN에 적용한 5가지 전처리 결과를
# 직접 열어서 비교하기 위해서입니다.
#
# 주의:
# 현재 코드에서는 이미지를 저장하지만
# OCR 인식 결과는 아직 생성하지 않습니다.
# --------------------------------------------------

for index, (
    name,
    result,
) in enumerate(
    variants.items(),
    start=1,
):

    # 전처리 순번과 이름으로 파일명 생성
    output_path = (
        OUTPUT_DIR
        / f"{index:02d}_{name}.jpg"
    )

    # 전처리 결과 이미지 저장
    cv2.imwrite(
        str(output_path),
        result,
    )

    # 저장된 파일 경로 출력
    print(
        "Saved:",
        output_path
    )


# --------------------------------------------------
# 10. 대표 이미지와 ROI 정보 및 완료 메시지 출력
#
# 무엇을 하나요?
# - 전처리에 사용한 대표 이미지 이름을 출력합니다.
# - JSON에서 가져온 ROI 좌표를 출력합니다.
# - 전처리 결과 저장이 완료되었다는 메시지를 출력합니다.
#
# Reference:
# 전처리에 사용한 대표 제품 이미지
#
# ROI:
# 대표 이미지에서 잘라낸 SN 영역 좌표
#
# Preprocess Compare Complete:
# 전처리 후보 이미지 생성 완료 메시지
#
# 왜 필요한가요?
# 어떤 이미지와 좌표를 이용하여
# 전처리 결과를 만들었는지 확인하기 위해서입니다.
#
# 다음 실습:
# 43_day06_ocr_compare.py
#
# 이번에 만든 5가지 전처리 후보를
# Tesseract로 실제 OCR하여
# SN 인식 정확도를 비교합니다.
# --------------------------------------------------

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
[1. 필요한 라이브러리와 OCR 공통 함수 불러오기]

Path를 불러온다.
→ 파일과 폴더 경로 관리

cv2를 불러온다.
→ 이미지 읽기와 저장

day06_ocr_utils.py에서
공통 함수와 프로젝트 경로를 불러온다.

PROJECT_ROOT
→ 프로젝트 최상위 경로

load_rules()
→ OCR Rule JSON 읽기

get_roi_for_file()
→ 이미지별 ROI 좌표 가져오기

crop_roi()
→ 문자 영역 Crop

make_preprocess_variants()
→ 다섯 가지 전처리 이미지 생성

          ↓

[2. JSON에서 OCR Rule과 ROI 설정 읽기]

load_rules() 함수를 호출한다.

          ↓

configs/day06_ocr_rules.json
파일을 읽는다.

          ↓

JSON에 저장된 다음 정보를 가져온다.

대표 이미지 이름
이미지별 ROI 좌표
CLAHE 설정
기본 OCR 설정

          ↓

가져온 정보를 rules에 저장한다.

          ↓

[3. 대표 이미지와 전처리 결과 저장 경로 설정]

rules에서 reference_file을 읽는다.

          ↓

대표 이미지 파일명을 가져온다.

예:
0637145699424_side.png

          ↓

대표 이미지 전체 경로를 만든다.

samples/day06/product/
0637145699424_side.png

          ↓

전처리 결과 저장 폴더를 지정한다.

outputs/day06/preprocess/

          ↓

[4. 대표 제품 이미지 읽기 및 오류 확인]

cv2.imread()를 이용하여
대표 제품 이미지를 읽는다.

          ↓

이미지가 정상적으로 읽혔는지 확인한다.

          ↓

이미지 읽기에 실패했다면
FileNotFoundError를 발생시킨다.

          ↓

정상적으로 읽었다면
다음 단계로 이동한다.

          ↓

[5. 대표 이미지의 SN ROI 좌표 가져오기]

get_roi_for_file() 함수를 호출한다.

          ↓

rules와 대표 이미지 파일명을 전달한다.

          ↓

JSON의 roi_by_file에서
현재 이미지에 해당하는 좌표를 찾는다.

          ↓

다음 형태의 좌표를 가져온다.

[x1, y1, x2, y2]

예:
[434, 294, 529, 318]

          ↓

roi_coordinates에 저장한다.

          ↓

[6. 저장된 좌표를 이용하여 SN 영역 Crop]

crop_roi() 함수를 호출한다.

          ↓

원본 이미지와 ROI 좌표를 전달한다.

          ↓

선택한 영역만 잘라낸다.

image[y1:y2, x1:x2]

          ↓

13자리 SN 숫자 부분만 포함된
ROI 이미지를 만든다.

          ↓

roi에 저장한다.

          ↓

[7. SN ROI에 다섯 가지 전처리 적용]

make_preprocess_variants() 함수를 호출한다.

          ↓

같은 SN ROI에
다섯 가지 전처리를 적용한다.

          ↓

① Original

원본 SN ROI를 그대로 사용한다.

          ↓

② Gray

컬러 이미지를 Grayscale로 변환한다.

          ↓

③ CLAHE

Gray 이미지의 국소 대비를 조정한다.

          ↓

④ Otsu

Gray 이미지에 자동 이진화를 적용한다.

          ↓

⑤ CLAHE + Otsu

CLAHE를 적용한 이미지에
Otsu 이진화를 추가로 적용한다.

          ↓

다섯 가지 결과를 딕셔너리로 만든다.

original
gray
clahe
otsu
clahe_otsu

          ↓

variants에 저장한다.

          ↓

[8. 전처리 결과를 저장할 폴더 생성]

결과 저장 폴더가 있는지 확인한다.

outputs/day06/preprocess/

          ↓

폴더가 없다면 생성한다.

이미 존재한다면 그대로 사용한다.

          ↓

[9. 다섯 가지 전처리 결과를 JPG로 저장]

variants에서 첫 번째 전처리를 가져온다.

          ↓

전처리 이름과 결과 이미지를 가져온다.

          ↓

저장 순번을 붙인다.

1 → 01

          ↓

파일명을 만든다.

01_original.jpg

          ↓

cv2.imwrite()로 이미지를 저장한다.

          ↓

저장 경로를 출력한다.

          ↓

다음 전처리 결과를 가져온다.

          ↓

같은 작업을 반복한다.

          ↓

01_original.jpg
02_gray.jpg
03_clahe.jpg
04_otsu.jpg
05_clahe_otsu.jpg

          ↓

다섯 가지 결과 이미지를 모두 저장한다.

          ↓

[10. 대표 이미지와 ROI 정보 및 완료 메시지 출력]

Reference 이미지 이름을 출력한다.

          ↓

현재 사용한 ROI 좌표를 출력한다.

          ↓

전처리 결과 생성 완료 메시지를 출력한다.

Preprocess Compare Complete

          ↓

[최종 결과]

41_day06_select_text_roi.py
          ↓
이미지별 ROI 좌표 저장
          ↓
day06_ocr_rules.json
          ↓
42_day06_preprocess_compare.py
          ↓
대표 이미지 파일명 확인
          ↓
대표 제품 이미지 읽기
          ↓
저장된 SN ROI 좌표 가져오기
          ↓
SN 숫자 영역 Crop
          ↓
다섯 가지 전처리 적용
          ↓
┌────────────────────────┐
│ 01_original.jpg        │
│ 원본 ROI                │
├────────────────────────┤
│ 02_gray.jpg            │
│ Grayscale              │
├────────────────────────┤
│ 03_clahe.jpg           │
│ CLAHE                  │
├────────────────────────┤
│ 04_otsu.jpg            │
│ Otsu 이진화             │
├────────────────────────┤
│ 05_clahe_otsu.jpg      │
│ CLAHE + Otsu           │
└────────────────────────┘
          ↓
전처리 결과 이미지 5장 저장
          ↓
다음 실습
43_day06_ocr_compare.py
          ↓
다섯 가지 전처리 이미지 4배 확대
          ↓
Tesseract OCR 실행
          ↓
실제 SN과 OCR 결과 비교
          ↓
가장 적절한 전처리 후보 선택
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

![[Pasted image 20261029163643.png]]

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

### 이번 코드는 무엇을 하나요?

42번에서는 같은 SN 이미지에 **5가지 전처리**를 적용하여 이미지를 준비했습니다.

이번 43번에서는 같은 전처리를 다시 적용한 뒤, **Tesseract OCR로 실제 숫자를 읽어 어떤 전처리가 가장 정확한지 비교**합니다.

정답 SN은 `0637145699424`이며, 각 OCR 결과가 이 숫자와 완전히 일치하는지 확인합니다.

### 코드의 전체 처리 흐름

```
① OCR 설정 JSON 읽기
   day06_ocr_rules.json
          ↓
② 대표 제품 이미지 불러오기
   0637145699424_side.png
          ↓
③ 저장된 ROI 좌표로 SN 영역 자르기
   0637145699424
          ↓
④ 5가지 전처리 이미지 생성
   ├─ Original
   ├─ Gray
   ├─ CLAHE
   ├─ Otsu
   └─ CLAHE + Otsu
          ↓
⑤ 각 이미지를 동일하게 4배 확대
          ↓
⑥ Tesseract OCR 실행
   PSM 7 + 숫자 Whitelist
          ↓
⑦ OCR 결과의 앞뒤 공백·줄바꿈 제거
          ↓
⑧ 실제 SN 정답과 비교
   0637145699424와 같은가?
          ↓
⑨ 전처리별 성공 / 실패 확인
          ↓
⑩ OCR 비교 결과를 CSV로 저장
          ↓
가장 적합한 전처리 방법 선택
```

### 실행 결과는 어떻게 확인하나요?

이번 실습 데이터에서는 다음과 같은 결과가 나왔습니다.

|전처리 방법|OCR 결과|판정|
|---|---|---|
|Original|`0637145699424`|성공|
|Gray|`0637145699424`|성공|
|CLAHE|`0637145699424`|성공|
|Otsu|`0637245697424`|실패|
|CLAHE + Otsu|`0637245670424`|실패|

비교 결과는 다음 파일에 저장됩니다.

```
reports/day06_preprocess_compare.csv
```

이번 데이터에서는 Original이 정확하게 인식되었으므로, 복잡한 전처리를 추가하지 않고 **Original을 최종 후보로 선택**합니다. 이 설정은 다음 44번 코드에서 확정하여 JSON에 저장합니다.

> **핵심:** 전처리한 이미지가 사람 눈에 선명해 보이더라도 OCR이 정확하게 읽는 것은 아닙니다. 따라서 실제 OCR 결과를 정답과 비교하여 가장 적합한 전처리를 선택합니다.

파일:

```
src/43_day06_ocr_compare.py
```

작성합니다.

```python
# --------------------------------------------------
# 1. 필요한 라이브러리와 OCR 공통 함수 불러오기
#
# 무엇을 하나요?
# - CSV 저장, 파일 경로, 이미지 처리 기능을 준비합니다.
# - 앞에서 작성한 OCR 공통 함수들을 불러옵니다.
#
# csv:
# 전처리별 OCR 비교 결과를 CSV로 저장합니다.
#
# Path:
# 파일 경로와 파일명을 관리합니다.
#
# cv2:
# 이미지 읽기, 확대, 결과 이미지 저장에 사용합니다.
#
# load_rules():
# JSON에 저장된 OCR 설정을 읽습니다.
#
# get_roi_for_file():
# 대표 이미지에 해당하는 ROI 좌표를 가져옵니다.
#
# crop_roi():
# SN 숫자 영역만 잘라냅니다.
#
# make_preprocess_variants():
# 5가지 전처리 후보를 생성합니다.
#
# run_ocr():
# Tesseract OCR을 실행하여 문자열을 반환합니다.
#
# 왜 필요한가요?
# 이전 실습에서 작성한 공통 함수를 재사용하여
# 전처리와 OCR 코드를 다시 작성하지 않기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 2. JSON에서 OCR Rule과 ROI 설정 읽기
#
# 무엇을 하나요?
# - load_rules() 공통 함수를 호출합니다.
# - day06_ocr_rules.json의 설정을 읽습니다.
#
# JSON에는 다음 정보가 저장되어 있습니다.
#
# reference_file:
# 대표 실험 이미지의 파일명
#
# roi_by_file:
# 이미지별 SN 영역 좌표
#
# clahe_clip_limit / clahe_grid:
# CLAHE 전처리 설정
#
# ocr_lang:
# Tesseract가 사용할 언어 데이터
#
# ocr_oem / ocr_psm:
# Tesseract 실행 방식
#
# whitelist:
# 인식하도록 허용한 문자 목록
#
# 왜 필요한가요?
# 모든 전처리 후보에 동일한 OCR 설정을 사용하여
# 공정하게 인식 결과를 비교하기 위해서입니다.
#
# 주의:
# 현재 preprocess_mode가 NOT_SELECTED여도 괜찮습니다.
# 이번 코드는 다섯 가지 후보를 직접 비교하기 때문입니다.
# --------------------------------------------------

rules = load_rules()


# --------------------------------------------------
# 3. 대표 이미지와 결과 저장 경로 설정
#
# 무엇을 하나요?
# - JSON에서 대표 이미지 파일명을 가져옵니다.
# - 해당 이미지의 전체 경로를 만듭니다.
# - OCR 비교 결과 CSV와 이미지 저장 폴더를 지정합니다.
#
# file_name:
# 대표 이미지 파일명
#
# IMAGE_PATH:
# 원본 제품 이미지 경로
#
# REPORT_PATH:
# 전처리별 OCR 결과를 저장할 CSV 경로
#
# OUTPUT_DIR:
# 실제 Tesseract에 입력한 확대 이미지를 저장할 폴더
#
# 왜 필요한가요?
# 대표 이미지 한 장에 동일한 OCR 조건을 적용하고
# 각 결과를 비교·기록하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 4. 대표 이미지 파일명에서 실제 SN 정답 가져오기
#
# 무엇을 하나요?
# - 이미지 파일명에서 확장자를 제거합니다.
# - "_side" 앞부분의 숫자를 가져옵니다.
# - OCR이 정확하게 읽었는지 비교할 정답으로 사용합니다.
#
# 예:
# 0637145699424_side.png
#
# Path(file_name).stem:
# 0637145699424_side
#
# .split("_side")[0]:
# 0637145699424
#
# expected_sn:
# 이번 OCR 비교 실험에서 사용하는 실제 인쇄 SN 정답
#
# 왜 필요한가요?
# Tesseract가 읽은 숫자가 실제 이미지에
# 인쇄된 숫자와 정확히 같은지 확인하기 위해서입니다.
#
# 주의:
# 여기서 expected_sn은 OCR 정확도 비교용 정답입니다.
# 이후 제품 검사에서 사용하는 생산상 Expected SN과는
# 역할이 다를 수 있습니다.
#
# 또한 SN은 앞자리가 0일 수 있으므로
# 숫자형(int)이 아닌 문자열로 유지합니다.
# --------------------------------------------------

expected_sn = (
    Path(file_name)
    .stem
    .split("_side")[0]
)


# --------------------------------------------------
# 5. 대표 제품 이미지 읽기 및 오류 확인
#
# 무엇을 하나요?
# - OpenCV로 대표 제품 이미지를 읽습니다.
# - 이미지 읽기에 성공했는지 확인합니다.
#
# cv2.imread():
# 이미지 파일을 읽는 함수
#
# image:
# 읽어온 원본 제품 이미지
#
# image is None:
# 이미지 파일을 읽지 못했다는 의미
#
# 왜 필요한가요?
# OCR을 수행하려면 먼저 원본 제품 이미지를 읽고
# 그 안에서 SN 숫자 영역을 잘라야 하기 때문입니다.
#
# 이미지가 없거나 읽기에 실패하면
# FileNotFoundError를 발생시킵니다.
# --------------------------------------------------

image = cv2.imread(
    str(IMAGE_PATH)
)

if image is None:
    raise FileNotFoundError(
        IMAGE_PATH
    )


# --------------------------------------------------
# 6. 대표 이미지의 SN ROI 좌표 가져오기
#
# 무엇을 하나요?
# - get_roi_for_file() 함수를 호출합니다.
# - JSON에 저장된 대표 이미지의 ROI 좌표를 가져옵니다.
#
# 전달하는 정보:
# rules:
# OCR 설정 및 이미지별 ROI 좌표
#
# file_name:
# 현재 대표 이미지의 파일명
#
# 반환 예:
# [434, 294, 529, 318]
#
# 좌표 의미:
# [x1, y1, x2, y2]
#
# 왜 필요한가요?
# 41번에서 선택한 정확한 SN 위치를 사용하여
# 같은 문자 영역을 다시 추출하기 위해서입니다.
#
# 현재 단계에서는 좌표만 가져옵니다.
# --------------------------------------------------

roi_coordinates = (
    get_roi_for_file(
        rules,
        file_name,
    )
)


# --------------------------------------------------
# 7. 저장된 좌표로 SN 숫자 영역 Crop
#
# 무엇을 하나요?
# - crop_roi() 공통 함수를 호출합니다.
# - 원본 이미지에서 13자리 SN 부분만 잘라냅니다.
#
# image:
# 원본 제품 이미지
#
# roi_coordinates:
# JSON에 저장된 ROI 좌표
#
# roi:
# 잘라낸 SN 숫자 이미지
#
# crop_roi() 내부 처리:
# image[y1:y2, x1:x2]
#
# 왜 필요한가요?
# 전체 이미지에는 SN 외에도 여러 문자와 배경이 있으므로
# OCR 대상인 13자리 숫자만 추출하기 위해서입니다.
#
# 아직 OCR을 실행하지 않습니다.
# --------------------------------------------------

roi = crop_roi(
    image,
    roi_coordinates,
)


# --------------------------------------------------
# 8. SN ROI에 다섯 가지 전처리 후보 생성
#
# 무엇을 하나요?
# - make_preprocess_variants() 함수를 호출합니다.
# - 동일한 SN ROI에서 5가지 전처리 이미지를 생성합니다.
#
# ① original:
# 원본 SN ROI
#
# ② gray:
# Grayscale 변환
#
# ③ clahe:
# Gray 이미지의 국소 대비 조정
#
# ④ otsu:
# Gray 이미지에 Otsu 자동 이진화
#
# ⑤ clahe_otsu:
# CLAHE 처리 후 Otsu 이진화
#
# 왜 필요한가요?
# 같은 SN이라도 영상의 전처리 상태에 따라
# Tesseract 인식 결과가 달라지는지 확인하기 위해서입니다.
#
# variants:
# 전처리 방법 이름과 결과 이미지를 보관하는 딕셔너리
#
# 예:
# variants["original"] → 원본 ROI
# variants["gray"]     → Gray 이미지
#
# 참고:
# 42번과 동일한 공통 함수를 호출하므로
# 같은 방식으로 전처리 후보를 생성합니다.
# --------------------------------------------------

variants = (
    make_preprocess_variants(
        roi,
        rules,
    )
)


# --------------------------------------------------
# 9. OCR 결과 이미지 폴더 및 결과 리스트 준비
#
# 무엇을 하나요?
# - OCR 입력 이미지를 저장할 폴더를 생성합니다.
# - 전처리별 OCR 결과를 저장할 빈 리스트를 만듭니다.
#
# OUTPUT_DIR.mkdir():
# 결과 이미지 폴더 생성
#
# parents=True:
# 필요한 상위 폴더까지 생성
#
# exist_ok=True:
# 이미 폴더가 있어도 오류 없이 진행
#
# rows:
# 전처리별 OCR 결과를 저장할 리스트
#
# 왜 필요한가요?
# 다섯 가지 OCR 결과를 순서대로 저장하고
# 마지막에 하나의 CSV로 정리하기 위해서입니다.
# --------------------------------------------------

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


rows = []


# --------------------------------------------------
# 10. 전처리 후보를 하나씩 가져와 OCR용으로 4배 확대
#
# 무엇을 하나요?
# - variants에서 전처리 이미지를 하나씩 가져옵니다.
# - 각 이미지를 가로·세로 모두 4배 확대합니다.
#
# enumerate():
# 반복하면서 순번을 생성하는 함수
#
# index:
# 전처리 이미지 순번
#
# name:
# 전처리 방법 이름
#
# processed:
# 현재 전처리 결과 이미지
#
# cv2.resize():
# 이미지 크기를 변경하는 함수
#
# fx=4.0:
# 가로 크기를 4배 확대
#
# fy=4.0:
# 세로 크기를 4배 확대
#
# INTER_CUBIC:
# 주변 픽셀을 이용하여 보간하는 방법
#
# 왜 필요한가요?
# 현재 SN ROI는 작은 이미지이므로
# Tesseract가 문자 획을 좀 더 큰 크기로
# 처리할 수 있도록 준비하기 위해서입니다.
#
# 중요:
# 다섯 가지 이미지에 모두 같은 4배 확대를 적용합니다.
# --------------------------------------------------

for index, (
    name,
    processed,
) in enumerate(
    variants.items(),
    start=1,
):

    # 작은 문자 ROI를 OCR용으로 4배 확대
    enlarged = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )


    # --------------------------------------------------
    # 11. Tesseract OCR 실행 및 Raw 문자열 정리
    #
    # 무엇을 하나요?
    # - 앞에서 만든 run_ocr() 공통 함수를 호출합니다.
    # - 4배 확대된 이미지를 Tesseract에 전달합니다.
    # - Tesseract가 읽은 문자열을 가져옵니다.
    # - 앞뒤 공백과 줄바꿈을 제거합니다.
    #
    # run_ocr():
    # Tesseract OCR을 실행하는 공통 함수
    #
    # 내부 OCR 설정 예:
    #
    # OEM = 3
    # PSM = 7
    # Whitelist = 0123456789
    #
    # raw_text:
    # Tesseract가 반환한 원본 문자열
    #
    # 예:
    # '0637145699424\n'
    #
    # raw_text.strip():
    # 문자열의 앞뒤 공백과 줄바꿈을 제거합니다.
    #
    # ocr_text:
    # 앞뒤 공백·줄바꿈을 정리한 OCR 결과
    #
    # 예:
    # 0637145699424
    #
    # 왜 필요한가요?
    # OCR 결과를 실제 SN 정답과
    # 정확하게 비교하기 위해서입니다.
    #
    # 주의:
    # strip()은 문자열 중간의 숫자를 고치지 않습니다.
    # 이 코드에서는 중간 공백도 제거하지 않습니다.
    # --------------------------------------------------

    # Tesseract OCR
    raw_text = run_ocr(
        enlarged,
        rules,
    )

    # 줄바꿈과 앞뒤 공백만 제거
    ocr_text = raw_text.strip()


    # --------------------------------------------------
    # 12. 실제 SN과 OCR 결과의 Exact Match 확인
    #
    # 무엇을 하나요?
    # - Tesseract가 읽은 문자열과 실제 SN을 비교합니다.
    # - 처음부터 끝까지 완전히 같은지 확인합니다.
    #
    # expected_sn:
    # 이미지 파일명에서 가져온 실제 SN 정답
    #
    # ocr_text:
    # Tesseract가 인식한 SN
    #
    # exact_match:
    # 두 문자열이 완전히 일치하는지 나타내는 값
    #
    # True:
    # OCR이 실제 SN을 정확하게 읽음
    #
    # False:
    # 한 글자라도 다르거나 길이가 다름
    #
    # 예:
    # 실제 SN: 0637145699424
    # OCR SN:  0637145699424
    # 결과: True
    #
    # 실제 SN: 0637145699424
    # OCR SN:  0637245697424
    # 결과: False
    #
    # 왜 필요한가요?
    # 사람 눈에 선명한 전처리가 아니라
    # 실제로 정확하게 읽힌 전처리를 찾기 위해서입니다.
    #
    # 주의:
    # 이것은 OCR 문자열 정확도 비교입니다.
    # 아직 제품의 최종 OK/NG 판정은 수행하지 않습니다.
    # --------------------------------------------------

    exact_match = (
        ocr_text
        == expected_sn
    )


    # --------------------------------------------------
    # 13. 실제 Tesseract 입력 이미지를 JPG로 저장
    #
    # 무엇을 하나요?
    # - 현재 전처리 이름과 순번으로 파일명을 만듭니다.
    # - 4배 확대된 OCR 입력 이미지를 저장합니다.
    #
    # f"{index:02d}":
    # 숫자를 두 자리 형식으로 표시합니다.
    #
    # 예:
    # 1 → 01
    # 2 → 02
    #
    # 저장 파일 예:
    #
    # ocr_01_original.jpg
    # ocr_02_gray.jpg
    # ocr_03_clahe.jpg
    # ocr_04_otsu.jpg
    # ocr_05_clahe_otsu.jpg
    #
    # 왜 필요한가요?
    # Tesseract에 전달한 이미지의 상태를
    # 직접 확인하고 OCR 오류 원인을 분석하기 위해서입니다.
    #
    # 주의:
    # 실제 OCR은 enlarged 배열에 수행합니다.
    # 저장된 JPG는 해당 입력을 확인하기 위한 결과물입니다.
    # --------------------------------------------------

    output_path = (
        OUTPUT_DIR
        / f"ocr_{index:02d}_{name}.jpg"
    )

    cv2.imwrite(
        str(output_path),
        enlarged,
    )


    # --------------------------------------------------
    # 14. 전처리별 OCR 결과 기록 및 터미널 출력
    #
    # 무엇을 하나요?
    # - 현재 전처리 방법의 OCR 결과를 정리합니다.
    # - rows 리스트에 추가합니다.
    # - 현재 결과를 터미널에 출력합니다.
    #
    # pipeline:
    # 전처리 방법 이름
    #
    # expected_sn:
    # 실제 SN 정답
    #
    # raw_text:
    # Tesseract의 원본 문자열
    #
    # ocr_text:
    # 앞뒤 공백과 줄바꿈을 제거한 결과
    #
    # exact_match:
    # OCR 결과가 정답과 같은지 확인한 값
    #
    # repr(raw_text):
    # 줄바꿈 문자가 포함된 문자열을
    # 사람이 확인하기 쉬운 형태로 표현합니다.
    #
    # 예:
    # '0637145699424\n'
    #
    # 왜 필요한가요?
    # 다섯 가지 전처리의 결과를
    # 한눈에 비교할 수 있도록 기록하기 위해서입니다.
    #
    # 반복문이 끝나면 rows에는
    # 다섯 가지 전처리의 OCR 결과가 저장됩니다.
    # --------------------------------------------------

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


# --------------------------------------------------
# 15. 다섯 가지 전처리의 OCR 비교 결과를 CSV로 저장
#
# 무엇을 하나요?
# - 결과 CSV를 저장할 폴더를 준비합니다.
# - CSV 파일을 생성합니다.
# - 컬럼명을 작성합니다.
# - rows에 모아 둔 OCR 결과를 저장합니다.
#
# 저장 파일:
# reports/day06_preprocess_compare.csv
#
# csv.DictWriter():
# 딕셔너리 데이터를 CSV로 저장하는 기능
#
# fieldnames:
# CSV에 사용할 컬럼 이름
#
# writer.writeheader():
# CSV 첫 행에 컬럼명을 작성
#
# writer.writerows(rows):
# 다섯 가지 전처리 결과를 한 번에 저장
#
# encoding="utf-8-sig":
# Excel에서 한글이 깨질 가능성을 줄입니다.
#
# newline="":
# CSV 저장 시 불필요한 빈 줄이 생기는 문제를 방지합니다.
#
# 왜 필요한가요?
# 전처리별 OCR 정확도를 표로 비교하고
# 다음 Rule Freeze 단계에서 참고하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 16. OCR 비교 완료 메시지 및 저장 위치 출력
#
# 무엇을 하나요?
# - CSV 저장 경로를 출력합니다.
# - OCR 비교가 완료되었다는 메시지를 출력합니다.
#
# 왜 필요한가요?
# 다섯 가지 전처리의 OCR 비교가 끝났고
# 결과 파일이 저장되었는지 확인하기 위해서입니다.
#
# 다음 실습:
# 44_day06_freeze_ocr_rules.py
#
# 이번 OCR 비교 결과를 바탕으로
# 사용할 전처리와 OCR 설정을 확정합니다.
#
# 주의:
# 이 코드에서는 JSON Rule을 변경하지 않습니다.
# 전처리 방법의 최종 확정은 다음 실습에서 수행합니다.
# --------------------------------------------------

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
[1. 필요한 라이브러리와 OCR 공통 함수 불러오기]

csv를 불러온다.
→ OCR 결과 CSV 저장

Path를 불러온다.
→ 파일 경로와 이름 처리

cv2를 불러온다.
→ 이미지 읽기, 확대, 저장

day06_ocr_utils에서
공통 함수를 불러온다.

load_rules()
get_roi_for_file()
crop_roi()
make_preprocess_variants()
run_ocr()

          ↓

[2. JSON에서 OCR Rule과 ROI 설정 읽기]

load_rules() 함수를 호출한다.

          ↓

configs/day06_ocr_rules.json을 읽는다.

          ↓

대표 이미지 파일명
이미지별 ROI 좌표
CLAHE 설정
Tesseract OCR 설정

등을 가져온다.

          ↓

rules 딕셔너리에 저장한다.

          ↓

[3. 대표 이미지와 결과 저장 경로 설정]

JSON에서 reference_file을 읽는다.

          ↓

대표 이미지 파일명을 가져온다.

예:
0637145699424_side.png

          ↓

원본 이미지 경로를 만든다.

samples/day06/product/
0637145699424_side.png

          ↓

CSV 저장 경로를 정한다.

reports/day06_preprocess_compare.csv

          ↓

OCR 입력 이미지 저장 폴더를 정한다.

outputs/day06/preprocess/

          ↓

[4. 대표 이미지 파일명에서 실제 SN 정답 가져오기]

대표 이미지의 파일명을 확인한다.

0637145699424_side.png

          ↓

확장자를 제거한다.

0637145699424_side

          ↓

_side 앞의 숫자 부분만 가져온다.

0637145699424

          ↓

expected_sn에 저장한다.

          ↓

이 값을 OCR 결과와 비교할
실제 SN 정답으로 사용한다.

          ↓

[5. 대표 제품 이미지 읽기 및 오류 확인]

cv2.imread()로
대표 제품 이미지를 읽는다.

          ↓

이미지가 정상적으로 읽혔는지 확인한다.

          ↓

이미지를 읽지 못했다면
FileNotFoundError를 발생시킨다.

          ↓

정상적으로 읽었다면
다음 단계로 이동한다.

          ↓

[6. 대표 이미지의 SN ROI 좌표 가져오기]

get_roi_for_file() 함수를 호출한다.

          ↓

rules와 대표 이미지 이름을 전달한다.

          ↓

JSON의 roi_by_file에서
해당 이미지의 ROI 좌표를 찾는다.

          ↓

[x1, y1, x2, y2]

형태의 좌표를 가져온다.

          ↓

roi_coordinates에 저장한다.

          ↓

[7. 저장된 좌표로 SN 숫자 영역 Crop]

crop_roi() 함수를 호출한다.

          ↓

원본 이미지와 ROI 좌표를 전달한다.

          ↓

SN 숫자 영역만 잘라낸다.

          ↓

13자리 SN ROI 이미지를 만든다.

          ↓

[8. SN ROI에 다섯 가지 전처리 후보 생성]

make_preprocess_variants() 함수를 호출한다.

          ↓

동일한 SN ROI에서
다섯 가지 전처리 이미지를 만든다.

          ↓

① Original
→ 원본 ROI 그대로 사용

② Gray
→ 컬러 이미지를 Grayscale로 변환

③ CLAHE
→ Gray 이미지의 국소 대비 조정

④ Otsu
→ Gray 이미지에 자동 이진화 적용

⑤ CLAHE + Otsu
→ 대비 조정 후 자동 이진화 적용

          ↓

다섯 가지 결과를
variants 딕셔너리에 저장한다.

          ↓

[9. OCR 결과 이미지 폴더 및 리스트 준비]

OCR 입력 이미지를 저장할
폴더가 있는지 확인한다.

          ↓

없다면 생성한다.

          ↓

OCR 결과를 저장할
빈 rows 리스트를 만든다.

          ↓

[10. 전처리 후보를 4배 확대]

variants에서
첫 번째 전처리 이미지를 가져온다.

          ↓

현재 전처리 이름과
결과 이미지를 확인한다.

          ↓

cv2.resize()를 실행한다.

          ↓

가로 크기를 4배 확대한다.

세로 크기를 4배 확대한다.

          ↓

INTER_CUBIC 보간법을 사용한다.

          ↓

확대된 OCR 입력 이미지를 만든다.

          ↓

[11. Tesseract OCR 실행 및 Raw 문자열 정리]

run_ocr() 함수를 호출한다.

          ↓

확대된 이미지와 OCR Rule을 전달한다.

          ↓

Tesseract OCR을 실행한다.

PSM = 7
Whitelist = 0123456789

          ↓

Tesseract가 인식한
Raw OCR 문자열을 가져온다.

예:
'0637145699424\n'

          ↓

raw_text.strip()을 실행한다.

          ↓

문자열 앞뒤의 공백과
줄바꿈을 제거한다.

          ↓

정리된 OCR 문자열을 만든다.

0637145699424

          ↓

[12. 실제 SN과 Exact Match 확인]

파일명에서 가져온 실제 SN을 확인한다.

expected_sn = 0637145699424

          ↓

OCR로 읽은 문자열을 확인한다.

ocr_text = 0637145699424

          ↓

두 문자열을 비교한다.

          ↓

처음부터 끝까지 완전히 같은가?

          ├── YES
          │
          ↓
    exact_match = True

          └── NO
                │
                ↓
          exact_match = False

          ↓

[13. 실제 Tesseract 입력 이미지 저장]

현재 전처리 방법의 이름을 확인한다.

          ↓

전처리 순번을 두 자리로 만든다.

          ↓

OCR 입력 이미지의 저장 파일명을 만든다.

예:
ocr_01_original.jpg

          ↓

4배 확대된 이미지를
JPG 파일로 저장한다.

          ↓

[14. 전처리별 OCR 결과 기록 및 출력]

현재 전처리 결과를 정리한다.

          ↓

Pipeline
→ 전처리 이름

Expected SN
→ 실제 SN 정답

Raw OCR
→ Tesseract 원본 문자열

OCR Text
→ 앞뒤 공백·줄바꿈을 정리한 문자열

Exact Match
→ 실제 SN과 일치 여부

          ↓

현재 결과를 rows에 추가한다.

          ↓

터미널에 결과를 출력한다.

          ↓

아직 비교할 전처리 이미지가 남아 있다면
10번으로 돌아간다.

          ↓

다음 전처리 이미지를
같은 방법으로 4배 확대한다.

          ↓

Tesseract OCR을 실행한다.

          ↓

실제 SN과 비교한다.

          ↓

결과 이미지를 저장한다.

          ↓

측정 결과를 rows에 추가한다.

          ↓

다섯 가지 전처리를 모두 반복한다.

          ↓

[15. 전처리별 OCR 비교 결과 CSV 저장]

reports 폴더가 있는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

day06_preprocess_compare.csv 파일을 연다.

          ↓

CSV 컬럼명을 작성한다.

pipeline
expected_sn
raw_text
ocr_text
exact_match

          ↓

rows에 저장된
다섯 가지 OCR 결과를 기록한다.

          ↓

CSV 파일을 저장한다.

          ↓

[16. OCR 비교 완료 메시지 및 저장 위치 출력]

CSV 파일의 저장 경로를 출력한다.

          ↓

OCR 비교 완료 메시지를 출력한다.

OCR Compare Complete

          ↓

[최종 결과]

41_day06_select_text_roi.py
        ↓
대표 이미지 SN ROI 좌표 저장
        ↓
day06_ocr_rules.json
        ↓
43_day06_ocr_compare.py
        ↓
대표 이미지 파일명 확인
        ↓
실제 SN 정답 추출
        ↓
대표 제품 이미지 읽기
        ↓
SN 영역 Crop
        ↓
다섯 가지 전처리 생성
        ↓
┌───────────────────────────────┐
│ Original                      │
│ Gray                          │
│ CLAHE                         │
│ Otsu                          │
│ CLAHE + Otsu                  │
└───────────────────────────────┘
        ↓
모두 동일하게 4배 확대
        ↓
Tesseract OCR 실행
        ↓
Raw OCR 문자열 확인
        ↓
앞뒤 공백과 줄바꿈 정리
        ↓
실제 SN과 비교
        ↓
Exact Match
True / False
        ↓
OCR 입력 이미지 5장 저장
        +
전처리별 OCR 결과 CSV 저장
        ↓
day06_preprocess_compare.csv
        ↓
다음 실습
44_day06_freeze_ocr_rules.py
        ↓
가장 적절한 전처리 방법 선택
        ↓
OCR Rule 확정
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
# --------------------------------------------------
# 1. OCR 문자열 검사에 필요한 정규식 라이브러리 불러오기
#
# 무엇을 하나요?
# - Python의 re 라이브러리를 불러옵니다.
#
# re:
# Regular Expression(정규식)을 처리하는 라이브러리입니다.
#
# 정규식이란?
# 문자열이 우리가 지정한 형식에 맞는지
# 검사하기 위한 규칙 표현식입니다.
#
# 이번 실습에서 사용할 정규식:
#
# ^[0-9]{13}$
#
# 의미:
# 처음부터 끝까지 숫자만 있고
# 정확히 13자리여야 한다는 뜻입니다.
#
# 왜 필요한가요?
# Tesseract가 읽은 문자열이
# 올바른 SN 형식인지 확인하기 위해서입니다.
# --------------------------------------------------

import re


# --------------------------------------------------
# 2. OCR이 읽은 문자열의 공백과 줄바꿈 정리
#
# 무엇을 하나요?
# - clean_ocr_text() 함수를 정의합니다.
# - OCR이 반환한 문자열을 전달받습니다.
# - 앞뒤 공백과 줄바꿈을 제거합니다.
# - 문자열 중간의 공백도 제거합니다.
#
# raw_text:
# Tesseract가 실제로 반환한 원본 문자열입니다.
#
# 예:
# '0637145699424\n'
#
# raw_text.strip():
# 문자열 앞뒤에 있는 공백과 줄바꿈을 제거합니다.
#
# re.sub(r"\s+", "", text):
# 문자열 안에 있는 공백, 탭, 줄바꿈 등을 제거합니다.
#
# \s:
# 공백, 탭, 줄바꿈과 같은 공백 문자
#
# +:
# 공백 문자가 하나 이상 연속되어 있는 경우
#
# "":
# 찾은 공백을 빈 문자열로 바꿉니다.
#
# 반환 결과:
# 0637145699424
#
# 왜 필요한가요?
# OCR 결과에 불필요한 공백이 포함되어 있으면
# 문자열 길이나 Expected SN 비교에 영향을
# 줄 수 있기 때문입니다.
#
# 주의:
# 이 함수는 공백만 제거하며
# OCR이 읽은 숫자를 다른 숫자로 변경하지 않습니다.
# --------------------------------------------------

def clean_ocr_text(
    raw_text,
):

    # 2-1. 문자열 앞뒤의 공백과 줄바꿈 제거
    #
    # 예:
    # '0637145699424\n'
    #          ↓
    # '0637145699424'

    text = raw_text.strip()


    # 2-2. 문자열 중간의 공백 문자 제거
    #
    # 예:
    # '0637 1456 99424'
    #          ↓
    # '0637145699424'
    #
    # 숫자 자체는 변경하지 않습니다.

    text = re.sub(
        r"\s+",
        "",
        text,
    )


    # 정리된 OCR 문자열 반환
    return text


# --------------------------------------------------
# 3. OCR 문자열이 정해진 SN 형식에 맞는지 검사
#
# 무엇을 하나요?
# - validate_pattern() 함수를 정의합니다.
# - OCR 문자열과 정규식을 전달받습니다.
# - 문자열이 정규식과 완전히 일치하는지 검사합니다.
#
# text:
# 공백과 줄바꿈이 정리된 OCR 문자열
#
# pattern:
# SN 형식을 정의한 정규식
#
# 이번 실습의 정규식:
# ^[0-9]{13}$
#
# ^:
# 문자열 시작
#
# [0-9]:
# 숫자 0~9 중 한 개
#
# {13}:
# 숫자가 정확히 13개
#
# $:
# 문자열 끝
#
# re.fullmatch():
# 문자열 전체가 지정한 정규식에
# 일치하는지 확인하는 함수입니다.
#
# bool():
# 검사 결과를 True 또는 False로 변환합니다.
#
# 예:
#
# 0637145699424
# → True
#
# 063714569942
# → False (12자리)
#
# ABC1234567890
# → False (영문 포함)
#
# 왜 필요한가요?
# Tesseract가 문자를 읽었다고 하더라도
# SN이 정확히 숫자 13자리인지는
# 별도로 확인해야 하기 때문입니다.
#
# 주의:
# 13자리 형식이 맞는 것과
# Expected SN이 일치하는 것은 다른 검사입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 4. OCR 문자열을 정리하고 최종 검사 준비
#
# 무엇을 하나요?
# - inspect_text() 함수를 정의합니다.
# - OCR 결과와 Expected SN, 정규식을 전달받습니다.
# - clean_ocr_text()를 호출하여 문자열을 정리합니다.
# - 검사 실패 이유를 저장할 리스트를 준비합니다.
#
# raw_text:
# Tesseract가 읽은 원본 문자열
#
# expected_text:
# 해당 제품에 인쇄되어 있어야 하는 SN
#
# pattern:
# OCR 문자열이 만족해야 하는 형식
#
# 예:
# raw_text = '0637145699424\n'
#
# expected_text = '0637145699424'
#
# pattern = r'^[0-9]{13}$'
#
# cleaned_text:
# 공백을 제거한 OCR 문자열
#
# reasons:
# 검사 기준을 만족하지 못한 이유를
# 저장하는 리스트입니다.
#
# 왜 필요한가요?
# OCR 결과에 대해 다음 세 가지를 검사하기 위해서입니다.
#
# ① OCR 결과가 비어 있는가?
# ② 숫자 13자리 형식인가?
# ③ Expected SN과 같은가?
# --------------------------------------------------

def inspect_text(
    raw_text,
    expected_text,
    pattern,
):

    # OCR 문자열 정리 함수 호출
    cleaned_text = (
        clean_ocr_text(
            raw_text
        )
    )

    # 검사 실패 이유를 저장할 빈 리스트
    reasons = []


    # --------------------------------------------------
    # 5. OCR 결과가 비어 있는지 확인
    #
    # 무엇을 하나요?
    # - 정리된 OCR 문자열이 비어 있는지 확인합니다.
    # - 비어 있다면 NG_OCR_EMPTY를 기록합니다.
    #
    # if not cleaned_text:
    # 문자열이 비어 있는지 확인하는 조건입니다.
    #
    # 예:
    # raw_text = '\n'
    #
    # Cleaning 이후:
    # cleaned_text = ''
    #
    # 검사 결과:
    # NG_OCR_EMPTY
    #
    # 왜 필요한가요?
    # Tesseract가 문자를 전혀 읽지 못한 경우와
    # 일부 숫자를 잘못 읽은 경우를
    # 구분하여 기록하기 위해서입니다.
    #
    # 주의:
    # OCR 결과가 비어 있다는 것은
    # OCR이 문자를 인식하지 못했다는 의미입니다.
    #
    # 제품에 실제로 문자가 없는지까지
    # 이 코드만으로 판단할 수는 없습니다.
    # --------------------------------------------------

    if not cleaned_text:
        reasons.append(
            "NG_OCR_EMPTY"
        )


    # --------------------------------------------------
    # 6. OCR 결과가 정확히 숫자 13자리인지 검사
    #
    # 무엇을 하나요?
    # - validate_pattern() 함수를 호출합니다.
    # - cleaned_text가 정규식을 만족하는지 검사합니다.
    # - 형식이 틀리면 NG_FORMAT을 기록합니다.
    #
    # 검사 정규식:
    # ^[0-9]{13}$
    #
    # 예 1:
    # OCR = 0637145699424
    # → 숫자 13자리
    # → format_ok = True
    #
    # 예 2:
    # OCR = 9728
    # → 4자리
    # → format_ok = False
    #
    # 예 3:
    # OCR = 063714569942A
    # → 영문 포함
    # → format_ok = False
    #
    # 왜 필요한가요?
    # 제조 현장에서 사용할 SN은
    # 정해진 문자열 형식을 만족해야 하기 때문입니다.
    #
    # 주의:
    # 숫자 13자리 형식이 맞더라도
    # 실제 Expected SN과 다를 수 있습니다.
    # --------------------------------------------------

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


    # --------------------------------------------------
    # 7. OCR 결과와 Expected SN을 정확히 비교
    #
    # 무엇을 하나요?
    # - 정리된 OCR 결과와 Expected SN을 비교합니다.
    # - 두 문자열이 완전히 같은지 확인합니다.
    # - 다르면 NG_EXPECTED_MISMATCH를 기록합니다.
    #
    # cleaned_text:
    # OCR이 읽은 SN
    #
    # expected_text:
    # 생산상 인쇄되어 있어야 하는 SN
    #
    # 예 1:
    #
    # OCR:
    # 0637145699424
    #
    # Expected:
    # 0637145699424
    #
    # 결과:
    # expected_match = True
    #
    # 예 2:
    #
    # OCR:
    # 0679256889404
    #
    # Expected:
    # 0679256889400
    #
    # 결과:
    # expected_match = False
    #
    # NG_EXPECTED_MISMATCH:
    # OCR 결과와 Expected SN이 다르다는 의미입니다.
    #
    # 왜 필요한가요?
    # 13자리 숫자 형식이 정확하더라도
    # 제품에 인쇄되어야 하는 번호와 다르면
    # 정상 제품으로 판정할 수 없기 때문입니다.
    #
    # 주의:
    # OCR 결과가 다르다는 사실만으로
    # 실제 인쇄 오류인지 OCR 인식 오류인지
    # 확정할 수는 없습니다.
    # --------------------------------------------------

    expected_match = (
        cleaned_text
        == expected_text
    )

    if not expected_match:
        reasons.append(
            "NG_EXPECTED_MISMATCH"
        )


    # --------------------------------------------------
    # 8. 검사 실패 이유를 확인하여 최종 OK / NG 판정
    #
    # 무엇을 하나요?
    # - reasons 리스트에 실패 이유가 있는지 확인합니다.
    # - 실패 이유가 없다면 OK로 판정합니다.
    # - 하나라도 있다면 NG로 판정합니다.
    #
    # 최종 판정 조건:
    #
    # OCR 결과가 비어 있지 않음
    #          AND
    # 숫자 13자리 형식 만족
    #          AND
    # Expected SN과 일치
    #
    # 모두 만족하면:
    # predicted_status = OK
    #
    # 하나라도 만족하지 못하면:
    # predicted_status = NG
    #
    # 왜 필요한가요?
    # OCR 결과에 대한 검사 조건을 종합하여
    # 최종 제품 판정을 만들기 위해서입니다.
    #
    # 주의:
    # 이 코드는 앞의 세 조건을 모두 검사하므로
    # 하나의 이미지에서 여러 NG Reason이
    # 동시에 기록될 수 있습니다.
    # --------------------------------------------------

    predicted_status = (
        "OK"
        if not reasons
        else "NG"
    )


    # --------------------------------------------------
    # 9. OCR 검사 결과를 딕셔너리로 반환
    #
    # 무엇을 하나요?
    # - 원본 OCR 문자열을 저장합니다.
    # - 정리된 OCR 문자열을 저장합니다.
    # - 형식 검사 결과를 저장합니다.
    # - Expected SN 비교 결과를 저장합니다.
    # - 최종 OK/NG 판정과 실패 이유를 저장합니다.
    #
    # raw_text:
    # Tesseract가 반환한 원본 문자열
    #
    # cleaned_text:
    # 공백과 줄바꿈을 제거한 문자열
    #
    # final_text:
    # 최종 비교에 사용하는 문자열
    #
    # format_ok:
    # SN 형식 검사 결과(True / False)
    #
    # expected_match:
    # Expected SN과 일치 여부
    #
    # predicted_status:
    # 최종 OK / NG 판정
    #
    # reasons:
    # NG 판정 이유를 저장한 리스트
    #
    # 왜 필요한가요?
    # 이후 Batch 검사에서 같은 결과를
    # CSV에 저장하고 성능을 평가하기 위해서입니다.
    #
    # 이번 실습에서는 문자 자동 교정을 하지 않으므로
    # final_text와 cleaned_text는 동일한 값입니다.
    # --------------------------------------------------

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
###  의사코드

```
[1. OCR 문자열 검사에 필요한 정규식 라이브러리 불러오기]

Python의 re 라이브러리를 불러온다.

          ↓

정규식을 이용하여
OCR 문자열의 형식을 검사할 준비를 한다.

          ↓

[2. OCR 문자열의 공백과 줄바꿈 정리]

clean_ocr_text() 함수를 만든다.

          ↓

Tesseract가 읽은 Raw OCR 문자열을 전달받는다.

예:
'0637145699424\n'

          ↓

raw_text.strip()을 실행한다.

          ↓

문자열 앞뒤의 공백과 줄바꿈을 제거한다.

예:
'0637145699424'

          ↓

re.sub()를 실행한다.

          ↓

문자열 중간에 포함된 공백 문자도 제거한다.

예:
'0637 1456 99424'
         ↓
'0637145699424'

          ↓

숫자 자체는 변경하지 않고
공백을 제거한 문자열을 반환한다.

          ↓

[3. OCR 문자열이 SN 형식에 맞는지 검사]

validate_pattern() 함수를 만든다.

          ↓

OCR 문자열과 정규식을 전달받는다.

          ↓

이번 SN 정규식:

^[0-9]{13}$

          ↓

re.fullmatch()로
문자열 전체가 규칙과 일치하는지 확인한다.

          ↓

숫자만 존재하는가?

          AND

정확히 13자리인가?

          ↓

두 조건을 모두 만족하면
True를 반환한다.

          ↓

조건을 만족하지 못하면
False를 반환한다.

          ↓

[4. OCR 문자열을 정리하고 최종 검사 준비]

inspect_text() 함수를 만든다.

          ↓

세 가지 입력값을 전달받는다.

raw_text
→ OCR이 읽은 원본 문자열

expected_text
→ 제품에 인쇄되어 있어야 하는 SN

pattern
→ SN 문자열 형식 정규식

          ↓

clean_ocr_text() 함수를 호출한다.

          ↓

OCR 문자열의 공백과 줄바꿈을 제거한다.

          ↓

정리한 문자열을 cleaned_text에 저장한다.

          ↓

검사 실패 이유를 저장할
빈 reasons 리스트를 만든다.

          ↓

[5. OCR 결과가 비어 있는지 확인]

cleaned_text가 비어 있는지 확인한다.

          ↓

OCR 결과가 비어 있는가?

          ├── YES
          │
          ↓
    NG_OCR_EMPTY를
    reasons에 추가한다.

          └── NO
                │
                ↓
          다음 검사로 이동한다.

          ↓

[6. OCR 결과가 숫자 13자리인지 검사]

validate_pattern() 함수를 호출한다.

          ↓

cleaned_text와 정규식을 전달한다.

          ↓

^[0-9]{13}$

조건을 검사한다.

          ↓

문자열이 정확히
13자리 숫자인가?

          ├── YES
          │
          ↓
    format_ok = True

          └── NO
                │
                ↓
          format_ok = False

          NG_FORMAT을
          reasons에 추가한다.

          ↓

[7. OCR 결과와 Expected SN 비교]

cleaned_text를 가져온다.

          ↓

expected_text를 가져온다.

          ↓

두 문자열이 완전히 같은지 비교한다.

          ↓

OCR 결과와 Expected SN이 같은가?

          ├── YES
          │
          ↓
    expected_match = True

          └── NO
                │
                ↓
          expected_match = False

          NG_EXPECTED_MISMATCH를
          reasons에 추가한다.

          ↓

[8. 최종 OK / NG 판정]

reasons 리스트를 확인한다.

          ↓

검사 실패 이유가 하나도 없는가?

          ├── YES
          │
          ↓
    predicted_status = OK

          └── NO
                │
                ↓
          predicted_status = NG

          ↓

[9. OCR 검사 결과를 딕셔너리로 반환]

다음 검사 결과를 하나의
딕셔너리로 정리한다.

          ↓

raw_text
→ OCR 원본 문자열

cleaned_text
→ 공백을 제거한 문자열

final_text
→ 최종 비교에 사용하는 문자열

format_ok
→ 숫자 13자리 형식 검사 결과

expected_match
→ Expected SN 일치 여부

predicted_status
→ 최종 OK / NG 판정

reasons
→ 검사 실패 이유 목록

          ↓

검사 결과 딕셔너리를 반환한다.

          ↓

[최종 연결 흐름]

Tesseract OCR
          ↓
Raw OCR 문자열

'0637145699424\n'
          ↓
clean_ocr_text()
          ↓
문자열 Cleaning

0637145699424
          ↓
validate_pattern()
          ↓
13자리 숫자 형식 검사

^[0-9]{13}$
          ↓
Expected SN 비교
          ↓
실패 이유 목록 생성
          ↓
최종 OK / NG 판정
          ↓
검사 결과 딕셔너리 반환
          ↓
다음 실습에서 재사용

45_day06_batch_ocr.py
          ↓
제품 이미지 6장 Batch 검사
          ↓
OCR 결과와 제품 판정 CSV 저장
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

# 27. Cleaning — OCR 결과에서 불필요한 문자를 정리합니다

**Cleaning(클리닝)**은 OCR이 읽은 문자열에서 불필요한 공백이나 줄바꿈을 제거하는 작업입니다.

예를 들어 43번 실습에서 Tesseract가 반환한 결과는 다음과 같습니다.

```
Raw OCR (원본 결과)
'0637145699424\n'
        ↓
Cleaning (문자열 정리)
        ↓
0637145699424
```

여기서 `\n`은 줄바꿈을 뜻합니다. Python의 `strip()` 함수를 사용하면 문자열 앞뒤의 공백과 줄바꿈을 제거할 수 있습니다.

### 숫자를 잘못 읽었다면 어떻게 할까요?

예를 들어 OCR이 다음과 같이 읽었다고 가정합니다.

```
OCR 결과: 0637245697424
실제 SN : 0637145699424
```

두 값이 다르더라도 **프로그램이 OCR 결과를 임의로 수정하지 않습니다.**

왜냐하면 OCR이 잘못 읽은 것인지, 제품에 실제로 다른 번호가 인쇄된 것인지 확인해야 하기 때문입니다.

따라서 이번 실습에서는 다음 설정을 유지합니다.

```
allow_context_correction = False
```

이 설정은 OCR이 읽은 숫자를 다른 숫자로 자동 교정하지 않겠다는 의미입니다.

> **핵심:** Cleaning은 공백과 줄바꿈을 정리하는 작업이지, OCR이 읽은 숫자 자체를 바꾸는 작업이 아닙니다. 정리된 문자열은 다음 단계에서 Regex와 Expected SN으로 검사합니다.
---

# 28. Regex와 Expected SN — 형식과 실제 값을 검사합니다

27번에서는 OCR 결과의 불필요한 공백과 줄바꿈을 제거했습니다.

이제 정리된 SN이 올바른지 **두 단계로 검사**합니다.

### ① Regex — 숫자 13자리인지 확인합니다

Regex(정규표현식)는 문자열이 정해진 형식에 맞는지 검사하는 방법입니다.

이번 SN은 숫자 13자리여야 하므로 다음 규칙을 사용합니다.

```
^[0-9]{13}$
```

예를 들어 `0637245697424`는 숫자 13자리이므로 **형식 검사에 통과(Format OK)**합니다.

하지만 형식이 맞는다고 해서 정상 제품이라는 뜻은 아닙니다.

### ② Expected SN — 기준 번호와 일치하는지 확인합니다

Expected SN은 해당 제품에 인쇄되어 있어야 한다고 설정한 **기준 번호**입니다.

```
OCR 결과    : 0637245697424
Expected SN : 0637145699424
                    ↓
              두 번호가 다름
                    ↓
          NG_EXPECTED_MISMATCH
```

`NG_EXPECTED_MISMATCH`는 **읽은 번호가 기준 번호와 일치하지 않는다**는 의미입니다.

### 최종 검사 흐름

```
Cleaning이 끝난 OCR 결과
           ↓
① Regex: 숫자 13자리인가?
   ├─ 아니요 → NG_FORMAT
   └─ 예
        ↓
② Expected SN과 일치하는가?
   ├─ 아니요 → NG_EXPECTED_MISMATCH
   └─ 예 → OK
```

> **핵심:** Regex는 **번호의 형식**을 검사하고, Expected SN은 **번호의 일치 여부**를 검사합니다. 두 검사를 모두 통과해야 최종 OK입니다.

※ OCR 결과가 기준값과 다르다고 해서 실제 인쇄 불량으로 단정할 수는 없습니다. OCR이 숫자를 잘못 읽었을 가능성도 있으므로, NG 원인은 원본 이미지와 함께 확인해야 합니다.

---

# 29. 문자열 검사 함수가 정상적으로 작동하는지 테스트합니다

### 이번 테스트는 무엇을 하나요?

28번에서 작성한 `inspect_text()` 함수가 **OCR 결과를 올바르게 검사하는지 미리 확인하는 테스트**입니다.

아직 실제 제품 이미지를 OCR로 읽는 것이 아닙니다. 대신 **정상값, 자릿수 오류, 번호 불일치, 빈 문자열**을 직접 입력하여 OK / NG가 제대로 나오는지 확인합니다.

### 터미널에서 실행합니다

프로젝트 최상위 폴더에서 다음 코드를 실행합니다.

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

    print("입력:", repr(value))
    print("형식:", result["format_ok"])
    print("기준값 일치:", result["expected_match"])
    print("최종 판정:", result["predicted_status"])
    print("실패 이유:", result["reasons"])
    print("-" * 40)
PY
```

`python - <<'PY'`는 별도 Python 파일을 만들지 않고 **터미널에서 Python 코드를 바로 실행하는 방법**입니다.

### 실행 결과를 확인합니다

|테스트 입력|예상 판정|확인할 내용|
|---|---|---|
|`0637145699424\n`|OK|줄바꿈을 제거하면 기준값과 일치|
|`063714569942`|NG|숫자가 12자리이므로 형식 오류|
|`0637245697424`|NG|13자리지만 기준 번호와 불일치|
|빈 문자열 `""`|NG|읽은 문자가 없어 검사 실패|

특히 빈 문자열은 `NG_OCR_EMPTY` 등 실패 이유를 통해 문자가 인식되지 않은 상황을 구분할 수 있습니다.

> **핵심:** 이번 테스트는 OCR 실행 전에 **문자열 정리 → Regex 형식 검사 → Expected SN 비교 → OK / NG 판정**이 올바르게 작동하는지 확인하는 과정입니다. 테스트가 끝나면 실제 제품 이미지 6장의 Batch 검사로 연결합니다.
---

## 29.1 의사코드로 읽어봅니다

```
[1. OCR 문자열 검사 함수 불러오기]

day06_ocr_validation.py에서
inspect_text() 함수를 불러온다.

이 함수는 다음 작업을 수행한다.

문자열 Cleaning
13자리 숫자 형식 검사
Expected SN 비교
최종 OK / NG 판정
NG Reason 기록

          ↓

[2. 검사할 SN 형식과 Expected SN 설정]

SN 형식을 검사할 정규식을 설정한다.

pattern = ^[0-9]{13}$

→ 숫자로만 구성되어 있어야 한다.
→ 정확히 13자리여야 한다.

          ↓

기대하는 SN을 설정한다.

expected = 0637145699424

          ↓

[3. 테스트할 문자열 4개 준비]

서로 다른 상황을 확인하기 위해
4개의 문자열을 준비한다.

① "0637145699424\n"
   → 정상 SN이지만 줄바꿈이 포함된 경우

② "063714569942"
   → 숫자가 12자리인 경우

③ "0637245697424"
   → 숫자 13자리는 맞지만 Expected SN과 다른 경우

④ ""
   → OCR 결과가 비어 있는 경우

          ↓

[4. 테스트 문자열을 하나씩 검사]

samples에서 첫 번째 문자열을 가져온다.

          ↓

inspect_text() 함수를 호출한다.

다음 세 가지 값을 전달한다.

raw_text
→ 현재 검사할 문자열

expected_text
→ Expected SN

pattern
→ 숫자 13자리 형식 규칙

          ↓

[5. inspect_text() 내부에서 문자열 검사]

먼저 OCR 문자열의 공백과 줄바꿈을 제거한다.

          ↓

정리된 문자열이 비어 있는지 확인한다.

비어 있다면
→ NG_OCR_EMPTY 기록

          ↓

정규식으로 숫자 13자리인지 확인한다.

형식이 다르면
→ NG_FORMAT 기록

          ↓

Expected SN과 완전히 같은지 확인한다.

값이 다르면
→ NG_EXPECTED_MISMATCH 기록

          ↓

실패 이유가 하나도 없다면
→ Status = OK

실패 이유가 하나라도 있다면
→ Status = NG

          ↓

검사 결과를 result에 저장한다.

          ↓

[6. 현재 문자열의 검사 결과 출력]

구분선을 출력한다.

          ↓

Raw
→ 검사에 전달한 원본 문자열 출력

Cleaned
→ 공백과 줄바꿈을 제거한 문자열 출력

Format OK
→ 숫자 13자리 형식 검사 결과 출력

Expected Match
→ Expected SN과 일치 여부 출력

Status
→ 최종 OK / NG 판정 출력

Reasons
→ 검사 실패 이유 목록 출력

          ↓

[7. 나머지 문자열도 같은 방법으로 반복]

아직 검사하지 않은 문자열이 있는가?

          ├── YES
          │
          ↓
    다음 문자열을 가져온다.
          ↓
    inspect_text()를 다시 호출한다.
          ↓
    검사 결과를 출력한다.
          │
          └── 4번부터 반복

          └── NO
                │
                ↓
           반복 종료

          ↓

[8. 최종 결과 확인]

테스트 ①
0637145699424\n
          ↓
줄바꿈 제거
          ↓
숫자 13자리 정상
          ↓
Expected SN 일치
          ↓
OK

----------------------------

테스트 ②
063714569942
          ↓
숫자 12자리
          ↓
NG_FORMAT
          +
NG_EXPECTED_MISMATCH
          ↓
NG

----------------------------

테스트 ③
0637245697424
          ↓
숫자 13자리 정상
          ↓
Expected SN 불일치
          ↓
NG_EXPECTED_MISMATCH
          ↓
NG

----------------------------

테스트 ④
빈 문자열
          ↓
OCR 결과 없음
          ↓
NG_OCR_EMPTY
          +
NG_FORMAT
          +
NG_EXPECTED_MISMATCH
          ↓
NG

          ↓

문자열 검사 함수가
4가지 상황을 의도대로 처리하는지 확인

          ↓

테스트 종료
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
# --------------------------------------------------
# 1. JSON 파일 처리 및 경로 설정에 필요한 라이브러리
#
# 무엇을 하나요?
# - JSON 파일을 읽고 저장할 기능을 불러옵니다.
# - 프로젝트 파일과 폴더 경로를 관리합니다.
#
# json:
# JSON 파일을 읽고 수정된 설정을 저장하는 라이브러리
#
# Path:
# 파일과 폴더 경로를 관리하는 클래스
#
# 왜 필요한가요?
# 앞에서 저장한 OCR Rule JSON을 읽은 뒤
# 실험 결과를 반영하여 다시 저장하기 위해서입니다.
# --------------------------------------------------

import json
from pathlib import Path


# --------------------------------------------------
# 2. 프로젝트 최상위 폴더와 OCR Rule JSON 경로 설정
#
# 무엇을 하나요?
# - 현재 Python 파일의 경로를 확인합니다.
# - 상위 폴더로 두 번 이동하여 프로젝트 Root를 찾습니다.
# - 수정할 OCR Rule JSON 파일의 경로를 지정합니다.
#
# __file__:
# 현재 Python 파일의 경로
#
# .resolve():
# 파일의 절대경로를 구합니다.
#
# .parent.parent:
# src 폴더의 상위인 프로젝트 Root로 이동합니다.
#
# CONFIG_PATH:
# OCR 검사 규칙을 저장할 JSON 파일 경로
#
# 프로젝트 구조:
#
# subject9_machinevision/
# ├── src/
# │   └── 44_day06_freeze_ocr_rules.py
# └── configs/
#     └── day06_ocr_rules.json
#
# 왜 필요한가요?
# 41번 실습에서 만든 기존 JSON 설정을
# 동일한 위치에서 읽고 수정하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. 기존 OCR Rule JSON 파일 읽기
#
# 무엇을 하나요?
# - 41번에서 만든 day06_ocr_rules.json을 읽습니다.
# - JSON 문자열을 Python 딕셔너리로 변환합니다.
#
# CONFIG_PATH.read_text():
# JSON 파일에 저장된 내용을 문자열로 읽습니다.
#
# json.loads():
# JSON 문자열을 Python 딕셔너리로 변환합니다.
#
# rules:
# 기존 OCR 설정을 저장하는 딕셔너리
#
# 기존 JSON에는 다음 정보가 있습니다.
#
# baseline_version:
# 현재 Rule 버전
#
# reference_file:
# 대표 이미지 파일명
#
# roi_by_file:
# 제품 이미지별 ROI 좌표
#
# preprocess_mode:
# 사용할 전처리 방법
#
# ocr_psm / whitelist:
# Tesseract 실행 설정
#
# expected_pattern:
# SN 문자열 형식 검사 규칙
#
# 왜 필요한가요?
# 기존 ROI 좌표와 기본 설정을 유지하면서
# 필요한 항목만 수정하기 위해서입니다.
#
# 주의:
# JSON 파일이 없으면 FileNotFoundError가 발생합니다.
# 먼저 41번 실습을 완료해야 합니다.
# --------------------------------------------------

rules = json.loads(
    CONFIG_PATH.read_text(
        encoding="utf-8"
    )
)


# --------------------------------------------------
# 4. Baseline Rule 버전을 v1.0으로 확정
#
# 무엇을 하나요?
# - 기존 Rule 버전을 변경합니다.
#
# 변경 전:
# day06-ocr-draft
#
# 변경 후:
# day06-ocr-v1.0
#
# baseline_version:
# 현재 OCR 검사 기준의 버전을 기록하는 항목
#
# Draft:
# 아직 최종 결정되지 않은 실험용 설정
#
# v1.0:
# 이번 실습에서 처음 확정한 Baseline Rule
#
# 왜 필요한가요?
# 어떤 버전의 검사 기준을 사용했는지
# 이후 Batch 검사와 평가에서 확인하기 위해서입니다.
#
# 주의:
# 버전명을 변경한다고 파일 자체가
# 수정 불가능한 상태가 되는 것은 아닙니다.
# --------------------------------------------------

rules[
    "baseline_version"
] = "day06-ocr-v1.0"


# --------------------------------------------------
# 5. OCR 전처리 방법을 original로 확정
#
# 무엇을 하나요?
# - 43번 OCR 비교 실험에서 선택한
#   전처리 방법을 JSON에 기록합니다.
#
# 43번 실험에서 비교한 방법:
#
# original
# gray
# clahe
# otsu
# clahe_otsu
#
# 기존 교재의 대표 이미지 실험에서는
# Original, Gray, CLAHE가 정확하게 인식되었습니다.
#
# 따라서 복잡한 처리가 필요하지 않은
# original을 Baseline으로 선택합니다.
#
# original:
# SN ROI에 Gray, CLAHE, Otsu 등의
# 추가 전처리를 적용하지 않는 방법입니다.
#
# 왜 필요한가요?
# Batch 검사에서도 43번에서 선택한
# 동일한 전처리 방법을 사용하기 위해서입니다.
#
# 주의:
# 이 코드는 OCR 결과를 다시 비교하지 않습니다.
# 이미 선택한 전처리 방법을 기록할 뿐입니다.
# --------------------------------------------------

rules[
    "preprocess_mode"
] = "original"


# --------------------------------------------------
# 6. Tesseract PSM과 숫자 Whitelist 확정
#
# 무엇을 하나요?
# - Tesseract의 문자 배치 인식 방식을 설정합니다.
# - OCR에서 허용할 문자를 숫자로 제한합니다.
#
# ocr_psm = 7
#
# PSM(Page Segmentation Mode):
# Tesseract에게 입력 이미지의 문자 배치를
# 어떻게 해석할지 알려주는 설정입니다.
#
# PSM 7:
# 입력 이미지를 한 줄의 문자로 처리합니다.
#
# 이번 SN:
# 0637145699424
#
# 숫자가 한 줄로 인쇄되어 있으므로 PSM 7을 사용합니다.
#
# whitelist = 0123456789
#
# Whitelist:
# Tesseract가 인식할 수 있는 문자 종류를 제한합니다.
#
# 이번 실습에서는 숫자만 필요하므로
# 0부터 9까지만 허용합니다.
#
# 왜 필요한가요?
# SN의 문자 구성에 맞게 OCR 조건을 설정하여
# 이후 모든 제품 이미지에 동일하게 적용하기 위해서입니다.
# --------------------------------------------------

rules[
    "ocr_psm"
] = 7

rules[
    "whitelist"
] = "0123456789"


# --------------------------------------------------
# 7. SN 형식 검사 및 자동 문자 보정 여부 확정
#
# 무엇을 하나요?
# - OCR 문자열의 형식 검사 규칙을 정합니다.
# - OCR이 인식한 숫자를 임의로 변경하지 않도록 설정합니다.
#
# expected_pattern:
# 문자열 형식을 검사하기 위한 정규식(Regex)
#
# ^[0-9]{13}$
#
# ^:
# 문자열의 시작
#
# [0-9]:
# 숫자 0부터 9까지 중 한 개
#
# {13}:
# 숫자가 정확히 13개
#
# $:
# 문자열의 끝
#
# 즉 처음부터 끝까지
# 정확히 숫자 13자리인지 검사합니다.
#
# allow_context_correction = False
#
# Context Correction:
# OCR이 읽은 문자를 주변 정보나 기대값에 맞춰
# 자동으로 고치는 기능을 의미합니다.
#
# False:
# 이번 실습에서는 자동 문자 보정을 사용하지 않습니다.
#
# 예:
# OCR 결과 = 0637245697424
#
# Expected SN = 0637145699424
#
# 프로그램이 OCR 숫자를 임의로
# Expected SN으로 바꾸지 않습니다.
#
# 왜 필요한가요?
# OCR이 잘못 읽은 경우와
# 제품에 실제로 다른 번호가 인쇄된 경우를
# 구분할 수 있도록 하기 위해서입니다.
#
# 주의:
# 현재 코드는 이 설정을 JSON에 기록합니다.
# 실제 문자열 검사에서는 별도로 작성한
# day06_ocr_validation.py의 함수를 사용합니다.
# --------------------------------------------------

rules[
    "expected_pattern"
] = r"^[0-9]{13}$"

rules[
    "allow_context_correction"
] = False


# --------------------------------------------------
# 8. 확정된 OCR Rule을 기존 JSON 파일에 저장
#
# 무엇을 하나요?
# - 수정한 rules 딕셔너리를 JSON 문자열로 변환합니다.
# - 기존 day06_ocr_rules.json 파일에 다시 저장합니다.
#
# json.dumps():
# Python 딕셔너리를 JSON 문자열로 변환합니다.
#
# ensure_ascii=False:
# 한글을 읽을 수 있도록 저장합니다.
#
# indent=2:
# 들여쓰기를 적용하여 내용을 읽기 쉽게 만듭니다.
#
# write_text():
# 문자열을 파일에 저장합니다.
#
# 저장 파일:
# configs/day06_ocr_rules.json
#
# 중요한 점:
# 기존 ROI 좌표와 Reference 이미지 정보는
# 변경하지 않고 그대로 유지합니다.
#
# 변경되는 정보:
#
# baseline_version
# preprocess_mode
# ocr_psm
# whitelist
# expected_pattern
# allow_context_correction
#
# 왜 필요한가요?
# 이후 Batch 검사에서 동일한 OCR Rule을
# 불러와 사용할 수 있도록 하기 위해서입니다.
#
# 주의:
# 기존 JSON 파일에 덮어쓰기 방식으로 저장합니다.
# --------------------------------------------------

CONFIG_PATH.write_text(
    json.dumps(
        rules,
        ensure_ascii=False,
        indent=2,
    ),
    encoding="utf-8",
)


# --------------------------------------------------
# 9. 최종 OCR Rule과 JSON 저장 경로 출력
#
# 무엇을 하나요?
# - Rule Freeze 완료 메시지를 출력합니다.
# - 확정된 설정값을 터미널에 표시합니다.
# - JSON 파일의 저장 경로를 출력합니다.
#
# 출력 항목:
#
# Version:
# 확정된 Baseline Rule 버전
#
# Preprocess:
# 사용할 전처리 방법
#
# PSM:
# Tesseract 문자 배치 설정
#
# Whitelist:
# OCR에서 허용할 문자
#
# Pattern:
# SN 문자열 형식 검사 정규식
#
# Context Correction:
# 자동 문자 보정 사용 여부
#
# Saved:
# JSON 파일 저장 경로
#
# 왜 필요한가요?
# 다음 Batch 실습을 시작하기 전에
# 검사 규칙이 올바르게 저장되었는지 확인하기 위해서입니다.
#
# 다음 실습:
# 45_day06_batch_ocr.py
#
# 제품 이미지 6장에 동일한 OCR Rule을 적용합니다.
# --------------------------------------------------

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
### 의사코드

```
[1. JSON 파일 처리 및 경로 설정 라이브러리 불러오기]

json 라이브러리를 불러온다.
→ JSON 설정 읽기 및 저장

Path를 불러온다.
→ 프로젝트 파일 경로 관리

          ↓

[2. 프로젝트 최상위 폴더와 OCR Rule 경로 설정]

현재 실행 중인 Python 파일의 위치를 찾는다.

src/44_day06_freeze_ocr_rules.py

          ↓

상위 폴더로 두 번 이동한다.

          ↓

프로젝트 최상위 폴더를 찾는다.

subject9_machinevision/

          ↓

OCR Rule JSON 경로를 지정한다.

configs/day06_ocr_rules.json

          ↓

[3. 기존 OCR Rule JSON 읽기]

day06_ocr_rules.json 파일을 읽는다.

          ↓

json.loads()를 이용하여
JSON 문자열을 Python 딕셔너리로 변환한다.

          ↓

기존 OCR 설정을 rules에 저장한다.

          ↓

기존 이미지별 ROI 좌표도 함께 가져온다.

          ↓

[4. Baseline Rule 버전을 v1.0으로 확정]

기존 버전을 확인한다.

day06-ocr-draft

          ↓

Baseline Rule 버전을 변경한다.

day06-ocr-v1.0

          ↓

[5. OCR 전처리 방법을 original로 확정]

43번에서 비교한 OCR 결과를 기준으로
사용할 전처리 방법을 정한다.

          ↓

Original
Gray
CLAHE

세 가지 방법이 정확하게 인식되었다.

          ↓

이 중 가장 단순한
Original을 선택한다.

          ↓

preprocess_mode = original

로 설정한다.

          ↓

[6. Tesseract PSM과 숫자 Whitelist 확정]

이번 OCR 대상은
한 줄짜리 13자리 SN이다.

          ↓

Tesseract 문자 배치 설정을 정한다.

ocr_psm = 7

          ↓

OCR에서 허용할 문자를 정한다.

whitelist = 0123456789

          ↓

숫자만 읽도록 설정한다.

          ↓

[7. SN 형식 검사 및 자동 문자 보정 여부 확정]

SN 형식을 검사할 정규식을 설정한다.

expected_pattern = ^[0-9]{13}$

          ↓

OCR 문자열이 다음 조건을
만족하도록 기준을 정한다.

숫자로만 구성되어 있어야 한다.

          AND

정확히 13자리여야 한다.

          ↓

OCR이 읽은 숫자를
다른 숫자로 자동 수정하지 않도록 설정한다.

allow_context_correction = False

          ↓

[8. 확정된 OCR Rule을 JSON 파일에 저장]

지금까지 수정한 rules 딕셔너리를 가져온다.

          ↓

다음 설정이 확정되었는지 확인한다.

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

          ↓

기존 이미지별 ROI 좌표는
그대로 유지한다.

          ↓

json.dumps()를 이용하여
딕셔너리를 JSON 문자열로 변환한다.

          ↓

한글을 읽을 수 있도록 저장한다.

들여쓰기를 적용한다.

          ↓

기존 JSON 파일을 덮어쓴다.

configs/day06_ocr_rules.json

          ↓

[9. 최종 OCR Rule과 저장 경로 출력]

Rule Freeze 완료 메시지를 출력한다.

          ↓

확정된 Rule 버전을 출력한다.

          ↓

선택된 전처리를 출력한다.

          ↓

PSM을 출력한다.

          ↓

Whitelist를 출력한다.

          ↓

SN 형식 정규식을 출력한다.

          ↓

자동 문자 보정 사용 여부를 출력한다.

          ↓

JSON 파일의 저장 경로를 출력한다.

          ↓

[최종 연결 흐름]

41_day06_select_text_roi.py
        ↓
제품 이미지 6장의 SN ROI 선택
        ↓
day06_ocr_rules.json
(Draft)
        ↓
42_day06_preprocess_compare.py
        ↓
5가지 전처리 이미지 생성
        ↓
43_day06_ocr_compare.py
        ↓
Tesseract OCR 정확도 비교
        ↓
Original / Gray / CLAHE 성공
        ↓
44_day06_freeze_ocr_rules.py
        ↓
Original 전처리 선택
        ↓
PSM 7 설정
        ↓
숫자 Whitelist 설정
        ↓
13자리 Regex 설정
        ↓
자동 문자 보정 사용 안 함
        ↓
Baseline Rule v1.0 확정
        ↓
day06_ocr_rules.json
(Freeze)
        ↓
다음 실습
45_day06_batch_ocr.py
        ↓
제품 이미지 6장에
동일한 OCR Rule 적용
        ↓
13자리 SN 읽기
        ↓
형식 검사
        ↓
Expected SN 비교
        ↓
OK / NG 판정
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
# --------------------------------------------------
# 1. Batch OCR에 필요한 라이브러리와 공통 함수 불러오기
#
# 무엇을 하나요?
# - CSV 처리와 이미지 처리에 필요한 라이브러리를 불러옵니다.
# - 앞에서 만든 OCR 공통 함수와 문자열 검사 함수를 가져옵니다.
#
# csv:
# Ground Truth CSV를 읽고 검사 결과 CSV를 저장합니다.
#
# cv2:
# 제품 이미지 읽기, 확대 및 결과 이미지 저장에 사용합니다.
#
# day06_ocr_utils.py:
# JSON Rule 읽기, ROI Crop, 전처리, Tesseract OCR 담당
#
# day06_ocr_validation.py:
# 문자열 Cleaning, 형식 검사, Expected SN 비교,
# 최종 OK/NG 판정 담당
#
# 왜 필요한가요?
# 앞에서 작성한 기능들을 재사용하여
# 하나의 Batch 검사 프로그램으로 연결하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 2. 제품 이미지, 정답 데이터 및 결과 저장 경로 설정
#
# 무엇을 하나요?
# - 검사할 제품 이미지 폴더를 지정합니다.
# - 실제 SN과 Expected SN이 저장된 CSV를 지정합니다.
# - OCR 입력 이미지를 저장할 폴더를 지정합니다.
# - 최종 검사 결과 CSV 경로를 지정합니다.
#
# PRODUCT_DIR:
# 실제 제품 이미지 6장이 있는 폴더
#
# LABELS_PATH:
# 제품별 정답 정보를 기록한 CSV
#
# 주요 컬럼:
# file          → 이미지 파일명
# actual_text   → 실제로 인쇄된 SN
# expected_text → 생산상 인쇄되어야 하는 SN
# actual_status → 실제 제품 정답 OK/NG
#
# OUTPUT_DIR:
# Tesseract에 입력할 확대 이미지를 저장할 폴더
#
# REPORT_PATH:
# 전체 제품의 OCR 및 검사 결과를 기록할 CSV
#
# 왜 필요한가요?
# 어떤 데이터를 검사하고
# 검사 결과를 어디에 저장할지 지정하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. JSON에 저장된 OCR Rule 읽기
#
# 무엇을 하나요?
# - load_rules() 공통 함수를 호출합니다.
# - day06_ocr_rules.json을 읽습니다.
# - JSON에 저장된 OCR 설정을 가져옵니다.
#
# 주요 설정:
#
# baseline_version:
# OCR Rule 버전
#
# roi_by_file:
# 이미지별 SN ROI 좌표
#
# preprocess_mode:
# 사용할 전처리 방법
#
# ocr_psm:
# Tesseract 문자 배치 설정
#
# whitelist:
# OCR에서 허용할 문자 목록
#
# expected_pattern:
# 13자리 SN 형식 검사 정규식
#
# 왜 필요한가요?
# 앞에서 확정한 동일한 OCR Rule을
# 모든 제품 이미지에 적용하기 위해서입니다.
# --------------------------------------------------

rules = load_rules()


# --------------------------------------------------
# 4. OCR Rule이 v1.0으로 확정되었는지 확인
#
# 무엇을 하나요?
# - JSON에 기록된 baseline_version을 확인합니다.
# - day06-ocr-v1.0이 아니면 프로그램을 중단합니다.
#
# 이번 실습에서 사용할 Rule:
#
# Version = day06-ocr-v1.0
# Preprocess = original
# PSM = 7
# Whitelist = 0123456789
# Regex = ^[0-9]{13}$
#
# 왜 필요한가요?
# 실험용 Draft 설정으로 Batch 검사를 하지 않고
# 앞에서 확정한 Baseline을 사용하기 위해서입니다.
#
# 주의:
# 이 코드는 버전 문자열을 확인합니다.
# JSON 내부 설정이 실제로 변경되지 않았는지까지
# 자동으로 검증하는 것은 아닙니다.
# --------------------------------------------------

if rules[
    "baseline_version"
] != "day06-ocr-v1.0":

    raise RuntimeError(
        "OCR Rule이 Freeze되지 않았습니다."
    )


# --------------------------------------------------
# 5. 제품별 정답 데이터 CSV 존재 여부 확인
#
# 무엇을 하나요?
# - day06_labels.csv 파일이 있는지 확인합니다.
# - 파일이 없다면 오류를 발생시킵니다.
#
# LABELS_PATH.exists():
# 지정한 파일이나 경로가 존재하는지 확인합니다.
#
# FileNotFoundError:
# 필요한 파일을 찾을 수 없을 때 발생시키는 오류입니다.
#
# 왜 필요한가요?
# OCR 결과와 실제 인쇄값을 비교하고
# 제품의 최종 판정이 올바른지 평가하려면
# 제품별 정답 데이터가 필요하기 때문입니다.
#
# 중요:
# actual_text와 actual_status는
# 이번 교육용 실습에서 정확도를 평가하기 위한 정답입니다.
#
# 실제 OCR 검사에서는 이런 정답을
# 미리 알고 있다고 가정하지 않습니다.
# --------------------------------------------------

if not LABELS_PATH.exists():

    raise FileNotFoundError(
        "Batch 검사에 필요한 "
        "day06_labels.csv가 없습니다.\n"
        f"확인 경로: {LABELS_PATH}"
    )


# --------------------------------------------------
# 6. OCR 입력 이미지를 저장할 결과 폴더 생성
#
# 무엇을 하나요?
# - OCR 입력 이미지 저장 폴더가 있는지 확인합니다.
# - 폴더가 없으면 생성합니다.
#
# OUTPUT_DIR.mkdir():
# 결과 폴더를 생성하는 함수
#
# parents=True:
# 필요한 상위 폴더까지 함께 생성합니다.
#
# exist_ok=True:
# 폴더가 이미 있으면 그대로 사용합니다.
#
# 저장 위치:
# outputs/day06/batch/
#
# 왜 필요한가요?
# OCR 실패가 발생했을 때
# 실제 Tesseract에 전달한 이미지를 확인하기 위해서입니다.
# --------------------------------------------------

OUTPUT_DIR.mkdir(
    parents=True,
    exist_ok=True,
)


# --------------------------------------------------
# 7. Ground Truth와 Expected SN 정보를 CSV에서 읽기
#
# 무엇을 하나요?
# - day06_labels.csv를 엽니다.
# - CSV 내용을 딕셔너리 목록으로 읽습니다.
# - 검사할 데이터가 있는지 확인합니다.
# - 검사 결과를 저장할 빈 리스트를 만듭니다.
#
# csv.DictReader():
# CSV의 컬럼명을 Key로 하여 각 행을 읽습니다.
#
# encoding="utf-8-sig":
# UTF-8 BOM이 포함된 CSV도 읽을 수 있도록 합니다.
#
# labels:
# 제품별 정답 정보가 저장된 리스트
#
# rows:
# OCR 검사 결과를 저장할 빈 리스트
#
# 예:
#
# file:
# 0679256889404_side.png
#
# actual_text:
# 0679256889404
#
# expected_text:
# 0679256889400
#
# actual_status:
# NG
#
# 왜 필요한가요?
# 각 제품에 맞는 Expected SN을 가져와
# OCR 결과와 비교하기 위해서입니다.
#
# 주의:
# SN의 맨 앞 0이 사라지지 않도록
# 문자열로 읽어야 합니다.
# --------------------------------------------------

with LABELS_PATH.open(
    "r",
    newline="",
    encoding="utf-8-sig",
) as file:

    labels = list(
        csv.DictReader(file)
    )


# 검사 데이터가 비어 있으면 오류 발생
if not labels:

    raise RuntimeError(
        "day06_labels.csv에 "
        "검사 데이터가 없습니다."
    )


# 이미지별 검사 결과를 저장할 리스트
rows = []


# --------------------------------------------------
# 8. CSV에 기록된 제품 이미지를 하나씩 검사
#
# 무엇을 하나요?
# - labels에서 제품 정보를 한 행씩 가져옵니다.
# - 현재 제품 이미지 파일명을 확인합니다.
# - 실제 이미지가 저장된 전체 경로를 만듭니다.
#
# for item in labels:
# CSV의 제품 정보를 하나씩 반복합니다.
#
# item["file"]:
# 현재 검사할 제품 이미지 파일명
#
# image_path:
# 현재 제품 이미지 전체 경로
#
# 왜 필요한가요?
# 대표 이미지 한 장이 아니라
# 여러 제품에 동일한 OCR 검사 과정을 적용하기 위해서입니다.
#
# 이번 실습의 데이터:
# OK 제품 4장
# NG 제품 2장
# 총 6장
#
# 주의:
# 반복 횟수는 CSV에 기록된 행 개수로 결정됩니다.
# 이 코드에서 6장인지 별도로 확인하지는 않습니다.
# --------------------------------------------------

for item in labels:

    file_name = item[
        "file"
    ]

    image_path = (
        PRODUCT_DIR
        / file_name
    )


    # --------------------------------------------------
    # 9. 현재 제품 이미지 읽기 및 오류 확인
    #
    # 무엇을 하나요?
    # - cv2.imread()로 현재 제품 이미지를 읽습니다.
    # - 이미지가 정상적으로 읽혔는지 확인합니다.
    #
    # image:
    # OpenCV로 읽은 원본 제품 이미지
    #
    # image is None:
    # 이미지 읽기에 실패했다는 의미입니다.
    #
    # 왜 필요한가요?
    # 이미지에서 SN 문자 영역을 추출하려면
    # 먼저 원본 제품 이미지를 읽어야 하기 때문입니다.
    # --------------------------------------------------

    image = cv2.imread(
        str(image_path)
    )

    if image is None:

        raise FileNotFoundError(
            f"제품 이미지를 찾을 수 없습니다: "
            f"{image_path}"
        )


    # --------------------------------------------------
    # 10. 현재 이미지에 저장된 ROI 좌표 가져오기
    #
    # 무엇을 하나요?
    # - get_roi_for_file() 공통 함수를 호출합니다.
    # - 이미지 파일명으로 저장된 ROI 좌표를 찾습니다.
    #
    # JSON의 roi_by_file:
    # 각 이미지 파일명에 대응하는 ROI 좌표 목록
    #
    # 반환 예:
    # [434, 294, 529, 318]
    #
    # 좌표 순서:
    # [x1, y1, x2, y2]
    #
    # 왜 필요한가요?
    # 제품마다 SN 위치가 다르므로
    # 현재 이미지에 맞는 좌표를 사용해야 하기 때문입니다.
    #
    # 주의:
    # 이번 코드는 41번에서 사람이 미리 지정한
    # 좌표를 재사용하는 방식입니다.
    # 문자의 위치를 자동 검출하는 방식은 아닙니다.
    # --------------------------------------------------

    roi_coordinates = (
        get_roi_for_file(
            rules,
            file_name,
        )
    )


    # --------------------------------------------------
    # 11. ROI 좌표를 이용하여 SN 숫자 영역 Crop
    #
    # 무엇을 하나요?
    # - crop_roi() 공통 함수를 호출합니다.
    # - 원본 제품 이미지에서 SN 숫자 부분만 잘라냅니다.
    #
    # 내부 처리:
    # image[y1:y2, x1:x2]
    #
    # roi:
    # 제품에 인쇄된 13자리 숫자 영역
    #
    # 왜 필요한가요?
    # Tesseract가 전체 제품의 여러 문자를 읽지 않고
    # 필요한 SN 숫자만 인식하도록 하기 위해서입니다.
    # --------------------------------------------------

    roi = crop_roi(
        image,
        roi_coordinates,
    )


    # --------------------------------------------------
    # 12. Freeze된 OCR 전처리 방법 적용
    #
    # 무엇을 하나요?
    # - preprocess_for_ocr() 함수를 호출합니다.
    # - JSON에서 선택한 전처리 방법을 적용합니다.
    #
    # 현재 확정된 설정:
    # preprocess_mode = original
    #
    # original:
    # SN ROI에 추가 Gray, CLAHE, Otsu 등을 적용하지 않고
    # 원본 ROI를 사용하는 방법입니다.
    #
    # processed:
    # OCR에 사용할 전처리 결과 이미지
    #
    # 왜 필요한가요?
    # 43번 실험에서 결정한 전처리 방법을
    # 모든 제품 이미지에 동일하게 적용하기 위해서입니다.
    #
    # 참고:
    # 현재 공통 함수는 다섯 가지 전처리를 계산한 뒤
    # 선택된 결과 하나를 반환하는 구조입니다.
    # --------------------------------------------------

    processed = (
        preprocess_for_ocr(
            roi,
            rules,
        )
    )


    # --------------------------------------------------
    # 13. OCR 입력 이미지를 가로·세로 4배 확대
    #
    # 무엇을 하나요?
    # - 전처리한 SN 이미지를 확대합니다.
    # - 가로와 세로 크기를 모두 4배로 만듭니다.
    #
    # cv2.resize():
    # 이미지의 크기를 변경하는 함수
    #
    # fx=4.0:
    # 가로 4배 확대
    #
    # fy=4.0:
    # 세로 4배 확대
    #
    # INTER_CUBIC:
    # 주변 픽셀을 이용하는 보간 방법
    #
    # 왜 필요한가요?
    # 원본 SN ROI가 작기 때문에
    # OCR이 처리할 문자 크기를 키우기 위해서입니다.
    #
    # 중요:
    # 43번 실험에서도 4배 확대했습니다.
    #
    # 같은 확대 조건을 유지해야
    # 실험에서 결정한 조건과 실제 Batch 조건을
    # 일치시킬 수 있습니다.
    #
    # 주의:
    # 확대만으로 원본에 없는 문자 정보가
    # 새로 생기는 것은 아닙니다.
    # --------------------------------------------------

    ocr_input = cv2.resize(
        processed,
        None,
        fx=4.0,
        fy=4.0,
        interpolation=cv2.INTER_CUBIC,
    )


    # --------------------------------------------------
    # 14. Tesseract OCR 실행하여 Raw 문자열 얻기
    #
    # 무엇을 하나요?
    # - run_ocr() 공통 함수를 호출합니다.
    # - 확대된 이미지와 확정된 OCR Rule을 전달합니다.
    # - Tesseract가 읽은 문자열을 가져옵니다.
    #
    # OCR 설정:
    #
    # OEM = 3
    # PSM = 7
    # Whitelist = 0123456789
    #
    # raw_text:
    # Tesseract가 반환한 원본 문자열
    #
    # 예:
    # '0637145699424\n'
    #
    # 왜 필요한가요?
    # 이미지 속 인쇄된 SN을
    # Python에서 비교할 수 있는 문자열로 변환하기 위해서입니다.
    #
    # 주의:
    # 이 단계에서는 문자를 읽기만 합니다.
    # 최종 OK/NG 판정은 다음 단계에서 수행합니다.
    # --------------------------------------------------

    raw_text = run_ocr(
        ocr_input,
        rules,
    )


    # --------------------------------------------------
    # 15. OCR 문자열 검사 및 제품 OK/NG 판정
    #
    # 무엇을 하나요?
    # - inspect_text() 공통 함수를 호출합니다.
    # - Raw OCR 문자열을 정리합니다.
    # - 숫자 13자리 형식을 검사합니다.
    # - Expected SN과 비교합니다.
    # - 최종 제품을 OK 또는 NG로 판정합니다.
    #
    # raw_text:
    # Tesseract가 실제로 읽은 문자열
    #
    # expected_text:
    # 해당 제품에 인쇄되어야 하는 SN
    #
    # pattern:
    # 숫자 13자리 형식을 검사하는 정규식
    #
    # 검사 순서:
    #
    # ① OCR 문자열의 공백과 줄바꿈 제거
    #
    # ② OCR 결과가 비어 있는지 확인
    #
    # ③ 숫자 13자리 형식인지 확인
    #
    # ④ Expected SN과 같은지 확인
    #
    # ⑤ 모든 조건을 만족하면 OK
    #    하나라도 실패하면 NG
    #
    # 예:
    #
    # OCR = 0679256889404
    # Expected = 0679256889400
    #
    # OCR이 실제 인쇄값을 정확히 읽었더라도
    # Expected SN과 다르면 NG입니다.
    #
    # 왜 필요한가요?
    # 단순히 문자를 읽는 OCR 기능을
    # 제조 제품의 OK/NG 검사로 연결하기 위해서입니다.
    # --------------------------------------------------

    result = inspect_text(
        raw_text=raw_text,

        expected_text=item[
            "expected_text"
        ],

        pattern=rules[
            "expected_pattern"
        ],
    )


    # --------------------------------------------------
    # 16. OCR이 실제 인쇄된 SN을 정확하게 읽었는지 평가
    #
    # 무엇을 하나요?
    # - OCR 결과와 실제 인쇄된 SN을 비교합니다.
    # - 완전히 같으면 True, 다르면 False를 기록합니다.
    #
    # result["final_text"]:
    # 문자열 Cleaning을 마친 OCR 결과
    #
    # item["actual_text"]:
    # 제품 이미지에 실제로 인쇄된 SN 정답
    #
    # ocr_exact:
    # OCR이 실제 인쇄 문자를 정확하게 읽었는지 나타냅니다.
    #
    # 예:
    #
    # 실제 인쇄 SN = 0679256889404
    # OCR 결과      = 0679256889404
    #
    # ocr_exact = True
    #
    # 왜 필요한가요?
    # OCR 자체의 문자 인식 정확도를
    # 제품 판정 정확도와 별도로 평가하기 위해서입니다.
    #
    # 중요:
    # actual_text는 정확도 평가용 Ground Truth입니다.
    # OCR을 실행할 때 정답으로 사용하지 않습니다.
    # --------------------------------------------------

    ocr_exact = (
        result["final_text"]
        == item["actual_text"]
    )


    # --------------------------------------------------
    # 17. 프로그램의 최종 제품 판정이 정확한지 평가
    #
    # 무엇을 하나요?
    # - 프로그램이 판정한 OK/NG를 확인합니다.
    # - 실제 제품의 정답 OK/NG와 비교합니다.
    #
    # result["predicted_status"]:
    # OCR 결과와 Expected SN으로 판정한 OK/NG
    #
    # item["actual_status"]:
    # Ground Truth에 기록된 실제 제품 정답
    #
    # inspection_correct:
    # 최종 제품 판정이 실제 정답과 일치하는지 나타냅니다.
    #
    # 예:
    #
    # 실제 상태 = NG
    # 프로그램 판정 = NG
    #
    # inspection_correct = True
    #
    # 왜 필요한가요?
    # OCR이 글자를 정확하게 읽었는지와
    # 제품을 올바르게 OK/NG 판정했는지는
    # 서로 다른 평가이기 때문입니다.
    #
    # 중요:
    # actual_status는 평가용 정답으로만 사용합니다.
    # 제품의 predicted_status 계산에는 사용하지 않습니다.
    # --------------------------------------------------

    inspection_correct = (
        result[
            "predicted_status"
        ]
        == item[
            "actual_status"
        ]
    )


    # --------------------------------------------------
    # 18. 실제 Tesseract에 입력한 확대 이미지 저장
    #
    # 무엇을 하나요?
    # - 현재 이미지 이름을 이용하여 결과 파일명을 만듭니다.
    # - Tesseract에 전달한 OCR 입력 이미지를 저장합니다.
    #
    # image_path.stem:
    # 이미지 파일명에서 확장자를 제외한 부분
    #
    # 예:
    # 0637145699424_side.png
    #
    # 저장 파일:
    # 0637145699424_side_ocr_input.jpg
    #
    # cv2.imwrite():
    # 이미지를 파일로 저장하는 함수
    #
    # 왜 필요한가요?
    # OCR 결과가 잘못되었을 때
    # 실제로 어떤 이미지가 입력되었는지
    # 확인하고 분석하기 위해서입니다.
    # --------------------------------------------------

    output_image_path = (
        OUTPUT_DIR
        / f"{image_path.stem}_ocr_input.jpg"
    )

    cv2.imwrite(
        str(output_image_path),
        ocr_input,
    )


    # --------------------------------------------------
    # 19. 이미지별 OCR 및 제품 검사 결과 기록·출력
    #
    # 무엇을 하나요?
    # - 현재 이미지의 검사 결과를 딕셔너리로 정리합니다.
    # - rows 리스트에 추가합니다.
    # - 터미널에 결과를 출력합니다.
    #
    # 주요 결과:
    #
    # actual_text:
    # 실제 인쇄된 SN
    #
    # expected_text:
    # 생산상 기대하는 SN
    #
    # raw_text:
    # Tesseract가 반환한 원본 문자열
    #
    # cleaned_text:
    # 공백·줄바꿈 등을 정리한 OCR 결과
    #
    # format_ok:
    # 숫자 13자리 형식 검사 결과
    #
    # expected_match:
    # OCR 결과와 Expected SN 일치 여부
    #
    # ocr_exact:
    # OCR 결과와 실제 인쇄 SN 일치 여부
    #
    # predicted_status:
    # 프로그램의 OK/NG 판정
    #
    # inspection_correct:
    # 프로그램 판정과 실제 제품 정답 일치 여부
    #
    # reasons:
    # NG 판정 이유
    #
    # 왜 필요한가요?
    # 제품별 검사 결과를 저장하고
    # 이후 전체 정확도를 계산하기 위해서입니다.
    # --------------------------------------------------

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

    # 현재 검사 결과를 리스트에 추가
    rows.append(
        row
    )


    # ------------------------------------
    # 터미널에 현재 제품 검사 결과 출력
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


# --------------------------------------------------
# 20. 전체 Batch 검사 결과를 CSV로 저장하고 완료 출력
#
# 무엇을 하나요?
# - CSV를 저장할 reports 폴더를 준비합니다.
# - CSV에 기록할 컬럼명을 정의합니다.
# - 모든 제품의 검사 결과를 CSV로 저장합니다.
# - 저장 경로와 완료 메시지를 출력합니다.
#
# REPORT_PATH:
# reports/day06_ocr_results.csv
#
# fieldnames:
# CSV에 사용할 컬럼명 목록
#
# csv.DictWriter():
# 딕셔너리 데이터를 CSV로 저장하는 기능
#
# writer.writeheader():
# CSV 첫 번째 줄에 컬럼명 작성
#
# writer.writerows(rows):
# 모든 이미지의 검사 결과 저장
#
# encoding="utf-8-sig":
# Excel에서 한글이 깨질 가능성을 줄입니다.
#
# 왜 필요한가요?
# 이미지별 결과를 파일로 보존하고
# 46번 전체 성능 평가에서 재사용하기 위해서입니다.
#
# 주의:
# 같은 경로에 CSV가 있으면 기존 결과를 덮어씁니다.
# --------------------------------------------------

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
### 의사코드

```
[1. 필요한 라이브러리와 공통 함수 불러오기]

csv와 cv2를 불러온다.

          ↓

day06_ocr_utils.py에서
OCR 공통 함수를 불러온다.

load_rules()
→ OCR Rule JSON 읽기

get_roi_for_file()
→ 이미지별 ROI 좌표 찾기

crop_roi()
→ SN 영역 잘라내기

preprocess_for_ocr()
→ 확정된 전처리 적용

run_ocr()
→ Tesseract OCR 실행

          ↓

day06_ocr_validation.py에서
inspect_text() 함수를 불러온다.

→ 문자열 정리
→ 13자리 형식 검사
→ Expected SN 비교
→ OK / NG 판정

          ↓

[2. 입력 데이터 및 결과 저장 경로 설정]

제품 이미지 폴더를 지정한다.

samples/day06/product/

          ↓

정답 데이터 CSV 경로를 지정한다.

configs/day06_labels.csv

          ↓

OCR 입력 이미지 저장 폴더를 지정한다.

outputs/day06/batch/

          ↓

최종 검사 결과 CSV 경로를 지정한다.

reports/day06_ocr_results.csv

          ↓

[3. JSON에 저장된 OCR Rule 읽기]

load_rules() 함수를 호출한다.

          ↓

day06_ocr_rules.json을 읽는다.

          ↓

다음 설정을 가져온다.

Rule 버전
이미지별 ROI 좌표
전처리 방법
PSM
Whitelist
Regex

          ↓

rules에 저장한다.

          ↓

[4. Rule Freeze 여부 확인]

현재 Rule 버전을 확인한다.

          ↓

day06-ocr-v1.0인가?

          ├── NO
          │
          ↓
    OCR Rule 오류를 발생시킨다.

    프로그램 중단

          └── YES
                │
                ↓
          다음 단계 진행

          ↓

[5. 제품별 정답 데이터 CSV 확인]

day06_labels.csv가 존재하는지 확인한다.

          ↓

파일이 없다면
FileNotFoundError를 발생시킨다.

          ↓

파일이 있다면
다음 단계로 이동한다.

          ↓

[6. OCR 입력 이미지 저장 폴더 생성]

outputs/day06/batch/
폴더가 있는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

폴더가 있으면 그대로 사용한다.

          ↓

[7. Ground Truth와 Expected SN 읽기]

day06_labels.csv 파일을 연다.

          ↓

각 제품의 정답 정보를 읽는다.

file
→ 제품 이미지 파일명

actual_text
→ 실제 인쇄된 SN

expected_text
→ 생산상 기대하는 SN

actual_status
→ 실제 제품의 OK / NG 정답

          ↓

CSV에 검사 데이터가 없는가?

          ├── YES
          │
          ↓
    오류 발생 및 프로그램 중단

          └── NO
                │
                ↓
          labels에 데이터 저장

          ↓

검사 결과를 저장할
빈 rows 리스트를 만든다.

          ↓

[8. 제품 이미지를 하나씩 검사]

labels에서
첫 번째 제품 정보를 가져온다.

          ↓

검사할 이미지 파일명을 확인한다.

예:
0637145699424_side.png

          ↓

제품 이미지 전체 경로를 만든다.

samples/day06/product/
0637145699424_side.png

          ↓

[9. 제품 이미지 읽기]

cv2.imread()로 제품 이미지를 읽는다.

          ↓

이미지 읽기에 성공했는지 확인한다.

          ↓

실패했다면 오류를 발생시킨다.

          ↓

성공했다면 다음 단계로 이동한다.

          ↓

[10. 현재 이미지의 ROI 좌표 가져오기]

get_roi_for_file() 함수를 호출한다.

          ↓

이미지 파일명을 이용하여
JSON에서 ROI 좌표를 찾는다.

          ↓

[x1, y1, x2, y2]

형태의 좌표를 가져온다.

          ↓

[11. SN 숫자 영역 Crop]

crop_roi() 함수를 호출한다.

          ↓

원본 이미지와 ROI 좌표를 전달한다.

          ↓

SN의 13자리 숫자 부분만 잘라낸다.

          ↓

SN ROI 이미지를 만든다.

          ↓

[12. Freeze된 전처리 적용]

preprocess_for_ocr() 함수를 호출한다.

          ↓

JSON에서 확정된 전처리를 확인한다.

preprocess_mode = original

          ↓

선택된 전처리 결과를 가져온다.

          ↓

processed에 저장한다.

          ↓

[13. SN ROI를 4배 확대]

cv2.resize()를 실행한다.

          ↓

가로 크기를 4배 확대한다.

세로 크기를 4배 확대한다.

          ↓

INTER_CUBIC 보간법을 사용한다.

          ↓

확대된 OCR 입력 이미지를 만든다.

          ↓

[14. Tesseract OCR 실행]

run_ocr() 함수를 호출한다.

          ↓

확대된 이미지와
확정된 OCR Rule을 전달한다.

          ↓

Tesseract가 숫자를 인식한다.

PSM = 7
Whitelist = 0123456789

          ↓

Raw OCR 문자열을 받는다.

예:
'0637145699424\n'

          ↓

[15. OCR 문자열 검사 및 제품 판정]

inspect_text() 함수를 호출한다.

          ↓

다음 정보를 전달한다.

Raw OCR
Expected SN
13자리 숫자 정규식

          ↓

OCR 문자열을 Cleaning한다.

          ↓

문자열이 비어 있는지 확인한다.

          ↓

숫자 13자리 형식인지 확인한다.

          ↓

Expected SN과 같은지 비교한다.

          ↓

검사 조건을 모두 만족하면
Predicted = OK

          ↓

하나라도 실패하면
Predicted = NG

          ↓

검사 결과와 NG Reason을 반환받는다.

          ↓

[16. OCR 자체의 정확도 평가]

OCR 결과인 final_text를 가져온다.

          ↓

실제 인쇄 SN인 actual_text를 가져온다.

          ↓

두 문자열을 비교한다.

          ↓

완전히 일치하는가?

          ├── YES
          │
          ↓
    ocr_exact = True

          └── NO
                │
                ↓
          ocr_exact = False

          ↓

[17. 제품 OK / NG 판정 정확도 평가]

프로그램의 predicted_status를 가져온다.

          ↓

실제 제품 정답인 actual_status를 가져온다.

          ↓

두 값을 비교한다.

          ↓

프로그램 판정과 실제 정답이 같은가?

          ├── YES
          │
          ↓
    inspection_correct = True

          └── NO
                │
                ↓
          inspection_correct = False

          ↓

[18. Tesseract 입력 이미지 저장]

현재 제품 이미지 파일명을 확인한다.

          ↓

OCR 입력 이미지의 파일명을 만든다.

예:
0637145699424_side_ocr_input.jpg

          ↓

4배 확대된 OCR 입력 이미지를 저장한다.

          ↓

outputs/day06/batch/에 기록한다.

          ↓

[19. 이미지별 OCR 및 제품 검사 결과 기록]

현재 제품의 검사 결과를 정리한다.

          ↓

실제 인쇄 SN을 기록한다.

Expected SN을 기록한다.

Raw OCR을 기록한다.

Cleaned Text를 기록한다.

Format OK를 기록한다.

Expected Match를 기록한다.

OCR Exact를 기록한다.

Predicted Status를 기록한다.

Inspection Correct를 기록한다.

NG Reason을 기록한다.

          ↓

검사 결과를 rows 리스트에 추가한다.

          ↓

현재 제품의 결과를 터미널에 출력한다.

          ↓

아직 검사하지 않은 제품이 있는가?

          ├── YES
          │
          ↓
    다음 제품 정보를 가져온다.
          │
          ↓
    8~19번 과정을 반복한다.

          └── NO
                │
                ↓
          모든 제품 검사 완료

          ↓

[20. 전체 Batch 검사 결과 CSV 저장]

reports 폴더가 있는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

CSV 컬럼명을 준비한다.

          ↓

day06_ocr_results.csv 파일을 연다.

          ↓

첫 번째 행에 컬럼명을 작성한다.

          ↓

rows에 저장한
전체 제품 검사 결과를 기록한다.

          ↓

CSV 파일을 저장한다.

          ↓

저장 경로를 출력한다.

          ↓

Day06 Batch OCR Complete

메시지를 출력한다.

          ↓

[최종 전체 흐름]

44_day06_freeze_ocr_rules.py
          ↓
day06-ocr-v1.0 확정
          ↓
45_day06_batch_ocr.py
          ↓
day06_ocr_rules.json 읽기
          +
day06_labels.csv 읽기
          ↓
제품 이미지 6장 순서대로 검사
          ↓
각 이미지의 SN ROI 좌표 확인
          ↓
SN 영역 Crop
          ↓
original 전처리
          ↓
4배 확대
          ↓
Tesseract OCR
          ↓
문자열 Cleaning
          ↓
Regex 형식 검사
          ↓
Expected SN 비교
          ↓
OK / NG 판정
          ↓
┌─────────────────────────────┐
│ OCR 정확도 확인             │
│ OCR 결과 = 실제 인쇄 SN?    │
│ ocr_exact                   │
└─────────────────────────────┘
          +
┌─────────────────────────────┐
│ 제품 검사 정확도 확인       │
│ 예측 OK/NG = 실제 OK/NG?    │
│ inspection_correct          │
└─────────────────────────────┘
          ↓
제품별 결과 기록
          ↓
6장 모두 검사
          ↓
OCR 입력 이미지 저장
          +
day06_ocr_results.csv 저장
          ↓
다음 실습
46_day06_evaluate.py
          ↓
OCR Exact Accuracy 계산
          +
Inspection Accuracy 계산
```

실행합니다.

```
python src/45_day06_batch_ocr.py
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

# 36. `46_day06_evaluate.py` — 전체 OCR 검사 결과를 평가합니다

### 이번 코드는 무엇을 하나요?

45번에서는 제품 이미지 6장을 OCR로 검사하고, 각 제품의 OK / NG 결과를 CSV 파일에 저장했습니다.

이번 46번에서는 **저장된 CSV를 읽어 OCR이 얼마나 정확하게 문자를 인식했는지, 제품의 OK / NG를 얼마나 정확하게 판정했는지 평가**합니다.

### 코드의 전체 처리 흐름

```
45번에서 검사한 제품 이미지 6장
             ↓
day06_ocr_results.csv 읽기
             ↓
① OCR 정확도 평가
   실제 인쇄된 SN을 정확하게 읽었는가?
             ↓
② 제품 검사 정확도 평가
   제품의 OK / NG를 올바르게 판정했는가?
             ↓
③ 실패한 이미지 확인
   어떤 이미지에서 오류가 발생했는가?
             ↓
전체 평가 결과 저장
```

### 두 가지 정확도는 무엇이 다른가요?

|평가 항목|확인하는 내용|
|---|---|
|**OCR Accuracy**|OCR 결과가 실제 인쇄된 SN과 일치하는가?|
|**Inspection Accuracy**|프로그램의 OK / NG 판정이 정답과 일치하는가?|

이번 실습의 예시 결과는 다음과 같습니다.

- **OCR Accuracy: 6/6 (100%)** — 제품 6장의 SN을 모두 정확하게 읽었습니다.
    
- **Inspection Accuracy: 6/6 (100%)** — 교육용 NG 제품 2개를 포함하여 전체 6장의 OK / NG를 모두 올바르게 판정했습니다.
    

즉, 100%는 모든 제품이 정상이라는 뜻이 아니라 **모든 검사 결과가 정답과 일치했다는 뜻**입니다.

### Python 파일을 작성합니다

```
src/46_day06_evaluate.py
```

> **핵심:** 45번은 제품별 검사 결과를 만드는 코드이고, 46번은 그 결과를 분석하여 **OCR과 제품 검사기의 정확도를 확인하는 코드**입니다.

```python
# --------------------------------------------------
# 1. CSV 처리 라이브러리와 프로젝트 경로 불러오기
#
# 무엇을 하나요?
# - CSV 파일을 읽는 기능을 불러옵니다.
# - 앞에서 만든 OCR 공통 모듈에서
#   프로젝트 최상위 폴더 경로를 가져옵니다.
#
# csv:
# 45번 실습에서 저장한 OCR 결과 CSV를 읽습니다.
#
# PROJECT_ROOT:
# 프로젝트 최상위 폴더 경로입니다.
#
# 왜 필요한가요?
# 앞에서 생성한 Batch 검사 결과를 읽어
# 전체 OCR 성능을 평가하기 위해서입니다.
#
# 주의:
# 이번 코드는 실제 OCR을 다시 실행하지 않습니다.
# 이미 저장된 검사 결과 CSV만 분석합니다.
# --------------------------------------------------

import csv

from day06_ocr_utils import (
    PROJECT_ROOT,
)


# --------------------------------------------------
# 2. Batch 결과 CSV와 평가 보고서 저장 경로 설정
#
# 무엇을 하나요?
# - 앞에서 저장한 OCR 검사 결과 CSV 경로를 지정합니다.
# - 전체 평가 결과를 저장할 Markdown 파일 경로를 지정합니다.
#
# RESULT_PATH:
# 45_day06_batch_ocr.py에서 생성한
# 이미지별 OCR 검사 결과 CSV입니다.
#
# METRIC_PATH:
# 이번 코드에서 생성할 전체 성능 평가 보고서입니다.
#
# 입력:
# reports/day06_ocr_results.csv
#
# 출력:
# reports/day06_ocr_metrics.md
#
# 왜 필요한가요?
# 이미지별 검사 결과를 읽고
# 전체 정확도를 하나의 보고서로 정리하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. Batch 검사 결과 CSV가 존재하는지 확인
#
# 무엇을 하나요?
# - RESULT_PATH에 CSV 파일이 존재하는지 확인합니다.
# - 파일이 없다면 오류를 발생시킵니다.
#
# RESULT_PATH.exists():
# 지정한 파일이나 경로가 존재하는지 확인합니다.
#
# FileNotFoundError:
# 필요한 파일을 찾지 못하면 발생시키는 오류입니다.
#
# 왜 필요한가요?
# 45번 Batch 검사를 완료해야
# 실제 OCR 결과와 제품 판정 결과를 평가할 수 있기 때문입니다.
#
# CSV가 없다면:
# 먼저 45_day06_batch_ocr.py를 실행해야 합니다.
# --------------------------------------------------

if not RESULT_PATH.exists():

    raise FileNotFoundError(
        "먼저 45_day06_batch_ocr.py를 "
        "실행해야 합니다.\n"
        f"결과 파일이 없습니다: "
        f"{RESULT_PATH}"
    )


# --------------------------------------------------
# 4. Batch 결과 CSV 읽기 및 데이터 존재 여부 확인
#
# 무엇을 하나요?
# - CSV 파일을 엽니다.
# - CSV의 각 행을 딕셔너리 형태로 읽습니다.
# - 검사 결과가 하나 이상 있는지 확인합니다.
#
# csv.DictReader():
# CSV의 컬럼 이름을 Key로 사용하여
# 각 행을 딕셔너리로 읽는 함수입니다.
#
# rows:
# 전체 이미지의 검사 결과가 저장된 리스트입니다.
#
# 예:
# rows[0]["ocr_exact"] = "True"
#
# rows[0]["inspection_correct"] = "True"
#
# encoding="utf-8-sig":
# UTF-8 BOM이 포함된 CSV도 읽도록 설정합니다.
#
# if not rows:
# 검사 결과가 하나도 없는지 확인합니다.
#
# 왜 필요한가요?
# 저장된 이미지별 검사 결과를 분석하여
# 전체 정확도를 계산하기 위해서입니다.
#
# 중요:
# rows가 비어 있으면 프로그램을 중단합니다.
# 따라서 뒤에서 전체 이미지 수로 나눌 때
# 분모가 0이 되는 문제를 방지할 수 있습니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 5. CSV의 문자열 "True" / "False"를 Boolean으로 변환
#
# 무엇을 하나요?
# - as_bool() 함수를 정의합니다.
# - CSV에서 읽은 문자열을 True 또는 False로 변환합니다.
#
# 왜 필요한가요?
# Python 코드에서는 True와 False를 Boolean으로 사용하지만
# CSV 파일에서 읽으면 문자열이기 때문입니다.
#
# 예:
#
# CSV에 저장된 값:
# "True"
#
# Python에서 읽은 값:
# 문자열 "True"
#
# 우리가 계산에 사용할 값:
# Boolean True
#
# str(value):
# 전달받은 값을 문자열로 변환합니다.
#
# strip():
# 앞뒤 공백을 제거합니다.
#
# lower():
# 영문 대문자를 소문자로 변환합니다.
#
# == "true":
# 최종 문자열이 true와 같은지 확인합니다.
#
# 변환 결과:
#
# "True"  → True
# "true"  → True
# "FALSE" → False
# "False" → False
#
# 주의:
# 현재 함수에서는 "true"와 일치하는 값만 True입니다.
# 다른 값은 모두 False로 처리합니다.
# --------------------------------------------------

def as_bool(
    value,
):
    return (
        str(value)
        .strip()
        .lower()
        == "true"
    )


# --------------------------------------------------
# 6. 전체 검사 이미지 개수 계산
#
# 무엇을 하나요?
# - CSV에 저장된 검사 결과 행의 개수를 계산합니다.
#
# len(rows):
# rows 리스트에 들어 있는 항목 수를 반환합니다.
#
# count:
# 전체 검사 이미지 수
#
# 예:
# 이미지 6장을 검사했다면
# count = 6
#
# 왜 필요한가요?
# OCR 정확도와 제품 검사 정확도를 계산할 때
# 전체 이미지 수가 분모로 사용되기 때문입니다.
# --------------------------------------------------

count = len(
    rows
)


# --------------------------------------------------
# 7. OCR이 실제 인쇄된 SN을 정확히 읽은 개수 계산
#
# 무엇을 하나요?
# - 각 이미지의 ocr_exact 값을 확인합니다.
# - True인 이미지 개수를 계산합니다.
#
# ocr_exact:
# OCR 결과가 실제 인쇄된 SN과 일치하는지 나타냅니다.
#
# True:
# 실제 인쇄된 SN을 정확하게 읽은 경우
#
# False:
# 실제 인쇄된 SN과 OCR 결과가 다른 경우
#
# sum():
# True 값을 1로 계산하여 모두 더합니다.
#
# as_bool():
# CSV에 저장된 "True" 문자열을
# Boolean True로 변환합니다.
#
# 예:
#
# 이미지 6장
#
# True
# True
# True
# True
# True
# True
#
# 결과:
# ocr_correct = 6
#
# 왜 필요한가요?
# OCR 자체의 문자 인식 정확도를
# 계산하기 위해서입니다.
#
# 중요:
# 제품이 OK인지 NG인지는 관계없습니다.
# 실제 인쇄된 문자열을 정확히 읽었는지만 확인합니다.
# --------------------------------------------------

ocr_correct = sum(
    as_bool(
        row["ocr_exact"]
    )
    for row in rows
)


# --------------------------------------------------
# 8. 제품의 최종 OK / NG 판정이 정확한 개수 계산
#
# 무엇을 하나요?
# - 각 이미지의 inspection_correct 값을 확인합니다.
# - True인 이미지 개수를 계산합니다.
#
# inspection_correct:
# 프로그램이 예측한 OK/NG가
# 실제 제품 정답과 일치하는지 나타냅니다.
#
# 예:
#
# 실제 상태 = NG
# 프로그램 판정 = NG
#
# inspection_correct = True
#
# 실제 상태 = OK
# 프로그램 판정 = NG
#
# inspection_correct = False
#
# 왜 필요한가요?
# OCR이 문자를 잘 읽었는지와
# 제품의 정상/불량을 올바르게 판정했는지는
# 서로 다른 평가이기 때문입니다.
#
# 예:
# 6장 모두 정확하게 판정했다면
# inspection_correct_count = 6
# --------------------------------------------------

inspection_correct_count = sum(
    as_bool(
        row["inspection_correct"]
    )
    for row in rows
)


# --------------------------------------------------
# 9. SN 형식 검사와 Expected SN 일치 개수 계산
#
# 무엇을 하나요?
# - 숫자 13자리 형식을 만족한 이미지 수를 계산합니다.
# - OCR 결과가 Expected SN과 일치한 이미지 수를 계산합니다.
#
# format_ok:
# OCR 문자열이 숫자 13자리인지 검사한 결과
#
# 정규식:
# ^[0-9]{13}$
#
# expected_match:
# OCR 결과가 Expected SN과 같은지 확인한 결과
#
# 예:
#
# 제품 6장 중
# 6장이 모두 숫자 13자리이면
#
# format_ok_count = 6
#
# 4장은 Expected SN과 일치하고
# 2장은 다르다면
#
# expected_match_count = 4
#
# 왜 필요한가요?
# OCR 문자열의 형식 검사와
# Expected SN 비교 결과를
# 전체 이미지 기준으로 확인하기 위해서입니다.
#
# 중요:
# Expected Match는 정확도와 다릅니다.
#
# NG 제품에서 실제 인쇄된 SN이 Expected SN과
# 다르다면 Expected Match = False가 정상일 수 있습니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 10. 전체 OCR 정확도 계산
#
# 무엇을 하나요?
# - 실제 SN을 정확하게 읽은 비율을 계산합니다.
#
# 계산식:
#
# OCR Exact Accuracy
# = OCR을 정확하게 읽은 이미지 수 / 전체 이미지 수
#
# 예:
#
# OCR Correct = 6
# Total Images = 6
#
# OCR Accuracy = 6 / 6
#              = 1.0
#              = 100%
#
# 왜 필요한가요?
# Tesseract가 이미지에 실제로 인쇄된 SN을
# 얼마나 정확하게 읽었는지 확인하기 위해서입니다.
#
# 주의:
# 여기서의 정확도는 SN 전체 문자열의
# 완전 일치 비율입니다.
#
# 개별 숫자 하나하나의 문자 정확도나
# Character Error Rate를 계산하는 것은 아닙니다.
# --------------------------------------------------

ocr_accuracy = (
    ocr_correct
    / count
)


# --------------------------------------------------
# 11. 전체 제품 OK / NG 판정 정확도 계산
#
# 무엇을 하나요?
# - 실제 제품 상태를 올바르게 판정한 비율을 계산합니다.
#
# 계산식:
#
# Inspection Accuracy
# = 올바르게 판정한 이미지 수 / 전체 이미지 수
#
# 예:
#
# Inspection Correct = 6
# Total Images = 6
#
# Inspection Accuracy = 6 / 6
#                     = 1.0
#                     = 100%
#
# 왜 필요한가요?
# 제품의 실제 상태가 OK인지 NG인지
# 프로그램이 올바르게 판정했는지 확인하기 위해서입니다.
#
# 중요:
# OCR Accuracy와 Inspection Accuracy는
# 항상 같은 값이 나오는 것은 아닙니다.
# --------------------------------------------------

inspection_accuracy = (
    inspection_correct_count
    / count
)


# --------------------------------------------------
# 12. OCR 실패 및 제품 판정 실패 이미지 목록 찾기
#
# 무엇을 하나요?
# - OCR이 실제 SN을 잘못 읽은 이미지 파일명을 찾습니다.
# - 제품의 OK/NG 판정이 틀린 이미지 파일명을 찾습니다.
#
# ocr_failed_files:
# ocr_exact가 False인 이미지 목록
#
# inspection_failed_files:
# inspection_correct가 False인 이미지 목록
#
# 리스트 컴프리헨션:
# 조건에 맞는 항목만 골라 새로운 리스트를 만드는 문법
#
# if not as_bool(...):
# 검사 결과가 False인 경우에만 파일명을 가져옵니다.
#
# 왜 필요한가요?
# 전체 정확도만 확인하는 것이 아니라
# 어떤 제품에서 오류가 발생했는지
# 파일명으로 확인하기 위해서입니다.
#
# 주의:
# 현재 코드는 실패한 이미지의 파일명만 찾습니다.
# 이미지를 다른 폴더로 복사하거나
# 실패 원인을 자동 분석하지는 않습니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 13. 실패 이미지 목록을 Markdown 문자열로 변환
#
# 무엇을 하나요?
# - OCR 실패 이미지 목록을 Markdown 형태로 만듭니다.
# - 제품 판정 실패 이미지 목록도 같은 방식으로 만듭니다.
#
# "\n".join():
# 여러 문자열을 줄바꿈으로 연결합니다.
#
# "- " + name:
# Markdown 목록 형식으로 파일명을 작성합니다.
#
# 예:
#
# - image_01.png
# - image_02.png
#
# if ocr_failed_files:
# OCR 실패 이미지가 존재하는지 확인합니다.
#
# 실패 이미지가 없다면:
# "- 없음"으로 기록합니다.
#
# 왜 필요한가요?
# Markdown 보고서에서 실패한 이미지 이름을
# 한 줄씩 읽기 쉽게 표시하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 14. 전체 OCR 성능 평가 보고서 작성
#
# 무엇을 하나요?
# - 전체 검사 이미지 수를 기록합니다.
# - OCR Exact Accuracy를 기록합니다.
# - Inspection Accuracy를 기록합니다.
# - 형식 검사 성공 개수를 기록합니다.
# - Expected SN 일치 개수를 기록합니다.
# - OCR 실패 이미지 목록을 기록합니다.
# - 제품 판정 실패 이미지 목록을 기록합니다.
#
# text:
# Markdown 보고서의 전체 내용이 저장되는 문자열
#
# f""" ... """:
# 여러 줄의 문자열 안에 변수 값을 넣는 방법입니다.
#
# {ocr_accuracy:.4f}:
# OCR 정확도를 소수점 아래 4자리까지 표시합니다.
#
# 예:
# 1.0 → 1.0000
#
# {count}:
# 전체 검사 이미지 수를 표시합니다.
#
# 왜 필요한가요?
# 45번에서 저장한 이미지별 결과를
# 하나의 성능 평가 보고서로 정리하기 위해서입니다.
#
# 주의:
# Expected Match는 제품 검사 정확도가 아닙니다.
#
# 실제 NG 제품에서는 Expected SN과 불일치하는 것이
# 의도한 검사 결과일 수 있습니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 15. Markdown 보고서 저장 및 터미널 출력
#
# 무엇을 하나요?
# - 평가 보고서를 저장할 폴더를 확인합니다.
# - 보고서 내용을 Markdown 파일로 저장합니다.
# - 보고서 전체 내용을 터미널에도 출력합니다.
# - 저장된 파일 경로를 출력합니다.
#
# METRIC_PATH.parent.mkdir():
# reports 폴더를 준비합니다.
#
# METRIC_PATH.write_text():
# Markdown 문자열을 파일로 저장합니다.
#
# encoding="utf-8":
# 한글을 포함한 내용을 UTF-8로 저장합니다.
#
# print(text):
# 전체 평가 결과를 터미널에 출력합니다.
#
# 저장 파일:
# reports/day06_ocr_metrics.md
#
# 왜 필요한가요?
# 전체 OCR 성능을 문서로 기록하여
# 이후 실패 분석과 Rule 성능 검토에 사용하기 위해서입니다.
#
# 주의:
# 같은 파일이 이미 존재한다면
# 기존 보고서 내용을 덮어씁니다.
#
# 다음 단계:
# 47_day06_make_challenge_variants.py
#
# 정상 SN 이미지를 Low Contrast, Blur,
# Tilt, Glare 상태로 변형하여
# OCR 검사기의 한계를 확인합니다.
# --------------------------------------------------

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
### 의사코드

```
[1. CSV 라이브러리와 프로젝트 경로 불러오기]

csv 라이브러리를 불러온다.

          ↓

day06_ocr_utils.py에서
PROJECT_ROOT를 불러온다.

          ↓

[2. 입력 CSV와 평가 보고서 경로 설정]

Batch 검사 결과 CSV 경로를 지정한다.

reports/day06_ocr_results.csv

          ↓

평가 보고서 저장 경로를 지정한다.

reports/day06_ocr_metrics.md

          ↓

[3. Batch 검사 결과 파일 확인]

day06_ocr_results.csv가 존재하는지 확인한다.

          ↓

파일이 없는가?

          ├── YES
          │
          ↓
    FileNotFoundError 발생

    먼저 45_day06_batch_ocr.py를
    실행하라는 메시지 출력

    프로그램 중단

          └── NO
                │
                ↓
          다음 단계 진행

          ↓

[4. Batch 결과 CSV 읽기]

CSV 파일을 연다.

          ↓

csv.DictReader()로
이미지별 검사 결과를 읽는다.

          ↓

각 이미지의 결과를
rows 리스트에 저장한다.

          ↓

rows가 비어 있는지 확인한다.

          ↓

검사 결과가 없다면
RuntimeError를 발생시킨다.

          ↓

검사 결과가 있다면
다음 단계로 이동한다.

          ↓

[5. 문자열 True / False 변환 함수]

as_bool() 함수를 만든다.

          ↓

CSV에서 읽은 값을 문자열로 변환한다.

          ↓

앞뒤 공백을 제거한다.

          ↓

모든 영문을 소문자로 변환한다.

          ↓

값이 "true"와 같은가?

          ├── YES
          │
          ↓
    Boolean True 반환

          └── NO
                │
                ↓
          Boolean False 반환

          ↓

[6. 전체 검사 이미지 개수 계산]

rows 리스트의 길이를 계산한다.

          ↓

count에 저장한다.

예:

count = 6

          ↓

[7. OCR을 정확하게 읽은 이미지 개수 계산]

각 이미지의 ocr_exact를 확인한다.

          ↓

문자열 True / False를
Boolean 값으로 변환한다.

          ↓

ocr_exact = True인 이미지 수를 센다.

          ↓

ocr_correct에 저장한다.

예:

ocr_correct = 6

          ↓

[8. 제품 판정이 정확한 이미지 개수 계산]

각 이미지의
inspection_correct를 확인한다.

          ↓

inspection_correct = True인
이미지 수를 센다.

          ↓

inspection_correct_count에 저장한다.

예:

inspection_correct_count = 6

          ↓

[9. SN 형식 및 Expected SN 일치 개수 계산]

각 이미지의 format_ok를 확인한다.

          ↓

숫자 13자리 형식을 만족한
이미지 개수를 계산한다.

          ↓

format_ok_count에 저장한다.

예:

format_ok_count = 6

          ↓

각 이미지의 expected_match를 확인한다.

          ↓

OCR 결과가 Expected SN과 일치한
이미지 개수를 계산한다.

          ↓

expected_match_count에 저장한다.

예:

expected_match_count = 4

          ↓

[10. OCR Exact Accuracy 계산]

OCR을 정확하게 읽은 이미지 수를 가져온다.

ocr_correct

          ↓

전체 이미지 수를 가져온다.

count

          ↓

정확도를 계산한다.

OCR Exact Accuracy
= ocr_correct / count

          ↓

예:

6 / 6 = 1.0000

          ↓

ocr_accuracy에 저장한다.

          ↓

[11. Inspection Accuracy 계산]

최종 OK / NG 판정이
정확한 이미지 수를 가져온다.

inspection_correct_count

          ↓

전체 이미지 수를 가져온다.

count

          ↓

정확도를 계산한다.

Inspection Accuracy
= inspection_correct_count / count

          ↓

예:

6 / 6 = 1.0000

          ↓

inspection_accuracy에 저장한다.

          ↓

[12. OCR 실패 및 제품 판정 실패 이미지 찾기]

rows의 검사 결과를 한 행씩 확인한다.

          ↓

ocr_exact = False인 이미지를 찾는다.

          ↓

해당 이미지 파일명을
ocr_failed_files에 저장한다.

          ↓

inspection_correct = False인
이미지를 찾는다.

          ↓

해당 이미지 파일명을
inspection_failed_files에 저장한다.

          ↓

[13. 실패 이미지 목록을 Markdown 형식으로 변환]

OCR 실패 이미지가 있는지 확인한다.

          ↓

실패 이미지가 있다면

- 이미지 파일명

형태로 목록을 만든다.

          ↓

실패 이미지가 없다면

- 없음

으로 기록한다.

          ↓

제품 판정 실패 이미지도
같은 방법으로 처리한다.

          ↓

ocr_failure_text
inspection_failure_text

두 문자열을 준비한다.

          ↓

[14. 전체 평가 보고서 작성]

Markdown 보고서 내용을 만든다.

          ↓

전체 이미지 개수를 기록한다.

Images

          ↓

OCR Exact Accuracy를 기록한다.

OCR Correct 개수도 함께 기록한다.

          ↓

Inspection Accuracy를 기록한다.

Inspection Correct 개수도 함께 기록한다.

          ↓

숫자 13자리 형식에 성공한
이미지 수를 기록한다.

Format OK

          ↓

Expected SN과 일치한
이미지 수를 기록한다.

Expected Match

          ↓

OCR 실패 이미지 목록을 기록한다.

          ↓

제품 판정 실패 이미지 목록을 기록한다.

          ↓

보고서 내용을 text에 저장한다.

          ↓

[15. Markdown 보고서 저장 및 출력]

reports 폴더가 존재하는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

text에 저장된 보고서 내용을

reports/day06_ocr_metrics.md

파일로 저장한다.

          ↓

보고서 전체 내용을 터미널에 출력한다.

          ↓

저장 경로를 출력한다.

          ↓

[최종 전체 흐름]

45_day06_batch_ocr.py
        ↓
제품 이미지 6장 검사
        ↓
day06_ocr_results.csv
        ↓
46_day06_evaluate.py
        ↓
CSV 결과 읽기
        ↓
전체 이미지 수 확인
        ↓
OCR Correct 집계
        ↓
Inspection Correct 집계
        ↓
Format OK 집계
        ↓
Expected Match 집계
        ↓
OCR Exact Accuracy 계산
        ↓
Inspection Accuracy 계산
        ↓
OCR 실패 이미지 목록 확인
        ↓
제품 판정 실패 이미지 목록 확인
        ↓
Markdown 보고서 작성
        ↓
day06_ocr_metrics.md 저장
        ↓
PART5 Batch 성능 평가 완료
        ↓
다음 실습
47_day06_make_challenge_variants.py
        ↓
Low Contrast / Blur / Tilt / Glare
환경 변화 이미지 생성
```

실행합니다.

```
python src/46_day06_evaluate.py
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
# --------------------------------------------------
# 1. 환경 변화 이미지 생성에 필요한 라이브러리 불러오기
#
# 무엇을 하나요?
# - 파일 경로를 관리할 기능을 불러옵니다.
# - 이미지를 처리할 OpenCV를 불러옵니다.
# - 배열과 이미지 데이터를 생성할 NumPy를 불러옵니다.
# - 앞에서 작성한 OCR 공통 함수를 불러옵니다.
#
# Path:
# 이미지 파일과 폴더 경로를 관리합니다.
#
# cv2:
# 이미지 읽기, 대비 변경, Blur, 회전,
# 빛 반사 효과 생성 및 이미지 저장에 사용합니다.
#
# numpy(np):
# 이미지와 같은 크기의 배열을 생성합니다.
#
# PROJECT_ROOT:
# 프로젝트 최상위 폴더 경로입니다.
#
# load_rules():
# JSON에 저장된 OCR Rule을 읽는 함수입니다.
#
# 왜 필요한가요?
# 정상 SN ROI를 서로 다른 촬영환경처럼 변형하여
# OCR 성능 테스트에 사용할 이미지를 만들기 위해서입니다.
# --------------------------------------------------

from pathlib import Path

import cv2
import numpy as np

from day06_ocr_utils import (
    PROJECT_ROOT,
    load_rules,
)


# --------------------------------------------------
# 2. 대표 SN 이미지와 Challenge 저장 경로 설정
#
# 무엇을 하나요?
# - JSON에 저장된 OCR Rule을 읽습니다.
# - 대표 이미지 파일명을 확인합니다.
# - 대표 이미지의 확장자를 제거합니다.
# - 앞에서 저장한 정상 SN ROI 이미지 경로를 만듭니다.
# - 환경 변화 이미지를 저장할 폴더를 지정합니다.
#
# load_rules():
# configs/day06_ocr_rules.json을 읽습니다.
#
# reference_file:
# 대표 OCR 실험에서 사용한 원본 이미지 파일명
#
# 예:
# 0637145699424_side.png
#
# Path(reference_file).stem:
# 확장자를 제거한 파일명
#
# 결과:
# 0637145699424_side
#
# SOURCE_PATH:
# 41번에서 잘라 저장한 정상 SN ROI 이미지 경로
#
# ROOT:
# 환경 변화 테스트 이미지들을 저장할 최상위 폴더
#
# 왜 필요한가요?
# 41번에서 이미 만든 정상 SN ROI를 다시 사용하여
# 새로운 테스트 이미지를 생성하기 위해서입니다.
#
# 이번 코드에서는 제품 전체 이미지가 아니라
# SN 숫자 영역만 잘라 놓은 이미지를 사용합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. 정상 SN ROI 이미지 읽기 및 크기 확인
#
# 무엇을 하나요?
# - OpenCV로 정상 SN ROI 이미지를 읽습니다.
# - 이미지가 정상적으로 읽혔는지 확인합니다.
# - 이미지의 높이와 너비를 가져옵니다.
#
# cv2.imread():
# 이미지 파일을 읽는 함수입니다.
#
# image:
# 정상 SN ROI 이미지가 저장되는 변수입니다.
#
# image is None:
# 이미지를 읽지 못했다는 의미입니다.
#
# FileNotFoundError:
# 이미지가 없거나 읽기에 실패하면 발생시키는 오류입니다.
#
# image.shape[:2]:
# 이미지의 높이와 너비를 가져옵니다.
#
# height:
# 이미지의 세로 크기
#
# width:
# 이미지의 가로 크기
#
# 왜 필요한가요?
# 이미지 회전이나 빛 반사 영역을 만들 때
# 원본 이미지의 크기를 알아야 하기 때문입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 4. Low Contrast 이미지 생성
#
# 무엇을 하나요?
# - 문자와 배경의 밝기 차이를 줄입니다.
# - 원본 이미지와 일정한 회색 이미지를 혼합합니다.
# - 대비가 약해진 SN 이미지를 만듭니다.
#
# Low Contrast란?
# 문자와 배경의 밝기 차이가 작아
# 숫자를 구분하기 어려운 상태입니다.
#
# np.full_like():
# 원본 이미지와 같은 크기·자료형의 배열을 만듭니다.
#
# neutral:
# 모든 픽셀 값이 128인 중간 회색 이미지입니다.
#
# cv2.addWeighted():
# 두 이미지를 지정한 비율로 혼합합니다.
#
# 계산 방식:
#
# 원본 이미지 × 0.45
#          +
# 회색 이미지 × 0.55
#          +
# 0
#
# 왜 필요한가요?
# 조명이 약하거나 문자와 배경의 대비가 낮아졌을 때
# OCR이 숫자를 얼마나 잘 읽는지 확인하기 위해서입니다.
#
# 주의:
# 이번 코드는 조명 환경을 물리적으로 재현하는 것이 아니라
# 픽셀 값을 변경하여 대비 저하를 모의 실험합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 5. Blur 이미지 생성
#
# 무엇을 하나요?
# - 원본 SN ROI에 Gaussian Blur를 적용합니다.
# - 문자 획과 경계를 흐리게 만듭니다.
#
# Blur란?
# 초점이 맞지 않은 것처럼 문자 경계가 흐려진 상태입니다.
#
# cv2.GaussianBlur():
# 가우시안 필터를 이용하여 이미지를 부드럽게 만드는 함수
#
# image:
# 원본 SN ROI 이미지
#
# (9, 9):
# Blur 계산에 사용할 커널 크기입니다.
#
# 0:
# Gaussian 표준편차를 OpenCV가 커널 크기에 따라
# 자동으로 결정하도록 하는 설정입니다.
#
# 왜 필요한가요?
# 카메라 초점 불량이나 영상 흐림으로
# 숫자의 획을 구분하기 어려워졌을 때
# OCR 결과가 어떻게 달라지는지 확인하기 위해서입니다.
#
# 주의:
# 커널 크기가 커지면 일반적으로 흐림 효과가 강해질 수 있습니다.
# 이번 실습에서는 (9, 9)를 사용합니다.
# --------------------------------------------------

blur = cv2.GaussianBlur(
    image,
    (9, 9),
    0,
)


# --------------------------------------------------
# 6. Tilt 이미지 생성을 위한 회전 중심과 행렬 계산
#
# 무엇을 하나요?
# - SN ROI 이미지의 중심 좌표를 계산합니다.
# - 이미지를 +8도 회전시키기 위한 행렬을 만듭니다.
#
# Tilt란?
# 카메라나 제품이 기울어져
# 문자 줄이 수평이 아닌 상태입니다.
#
# center:
# 이미지의 중심 좌표입니다.
#
# width / 2:
# 이미지 가로 중심
#
# height / 2:
# 이미지 세로 중심
#
# cv2.getRotationMatrix2D():
# 이미지를 회전시키기 위한 변환 행렬을 만드는 함수
#
# center:
# 회전 중심
#
# 8.0:
# 회전 각도 +8도
#
# 1.0:
# 이미지 크기 비율을 유지하는 배율
#
# 왜 필요한가요?
# 이미지의 중앙을 기준으로 일정한 각도만큼
# 회전시켜 문자 기울기 상황을 만들기 위해서입니다.
#
# 주의:
# 이 단계에서는 회전 계산을 위한 행렬만 만듭니다.
# 실제 이미지 회전은 다음 단계에서 수행합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 7. 회전 행렬을 이용하여 Tilt 이미지 생성
#
# 무엇을 하나요?
# - 앞에서 만든 회전 행렬을 이미지에 적용합니다.
# - 정상 SN ROI를 +8도 회전시킵니다.
# - 회전 후 비어 있는 영역은 흰색으로 채웁니다.
#
# cv2.warpAffine():
# 이미지에 회전·이동 등의 변환을 적용하는 함수
#
# image:
# 원본 SN ROI
#
# matrix:
# 6번에서 생성한 회전 변환 행렬
#
# (width, height):
# 회전 결과 이미지의 크기
#
# cv2.INTER_LINEAR:
# 픽셀 위치를 계산할 때 사용하는 선형 보간법
#
# cv2.BORDER_CONSTANT:
# 이미지 경계 바깥 부분을 일정한 색상으로 채웁니다.
#
# borderValue=(255, 255, 255):
# 경계 바깥 부분을 흰색으로 채웁니다.
#
# 왜 필요한가요?
# 카메라 각도나 제품 방향이 달라졌을 때
# OCR이 기울어진 숫자를 정확하게 읽는지
# 확인하기 위해서입니다.
#
# 주의:
# 회전 후에도 출력 이미지 크기는 원본과 같습니다.
# 따라서 회전하면서 문자 끝부분이 잘릴 수 있습니다.
# --------------------------------------------------

tilt = cv2.warpAffine(
    image,
    matrix,
    (width, height),
    flags=cv2.INTER_LINEAR,
    borderMode=cv2.BORDER_CONSTANT,
    borderValue=(255, 255, 255),
)


# --------------------------------------------------
# 8. Glare 이미지 생성을 위한 밝은 반사 영역 만들기
#
# 무엇을 하나요?
# - 원본 SN ROI 이미지를 복사합니다.
# - 복사한 이미지 위에 흰색 타원 영역을 그립니다.
# - 문자 일부가 빛에 가려진 것 같은 효과를 준비합니다.
#
# Glare란?
# 조명이나 강한 반사광 때문에
# 문자 일부가 밝게 보이거나 가려지는 현상입니다.
#
# image.copy():
# 원본을 변경하지 않고 복사합니다.
#
# overlay:
# 빛 반사 효과를 그릴 이미지입니다.
#
# cv2.ellipse():
# 이미지에 타원을 그리는 함수입니다.
#
# 타원의 중심:
# 가로 위치 = width × 0.65
# 세로 위치 = height × 0.50
#
# 타원의 가로·세로 반지름:
# 이미지 크기를 기준으로 계산합니다.
#
# max():
# 타원이 지나치게 작아지지 않도록
# 최소 반지름을 설정합니다.
#
# -20:
# 타원의 회전 각도
#
# 0, 360:
# 타원의 전체 둘레를 그리도록 지정
#
# (255, 255, 255):
# OpenCV BGR 기준 흰색
#
# -1:
# 타원 내부를 색으로 채우는 설정
#
# 왜 필요한가요?
# 포장지 표면의 빛 반사 때문에
# 숫자 일부가 보이지 않는 상황을
# 모의 실험하기 위해서입니다.
#
# 주의:
# 실제 조명의 물리적인 반사 현상을
# 정확히 재현하는 것은 아닙니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 9. 원본 이미지와 반사 영역을 혼합하여 Glare 완성
#
# 무엇을 하나요?
# - 흰색 타원이 그려진 overlay를 가져옵니다.
# - 원본 이미지와 일정한 비율로 혼합합니다.
# - 부분적인 빛 반사 효과가 있는 이미지를 만듭니다.
#
# cv2.addWeighted():
# 두 이미지의 픽셀 값을 비율에 따라 혼합합니다.
#
# 계산 방식:
#
# overlay × 0.60
#        +
# 원본 이미지 × 0.40
#        +
# 0
#
# glare:
# 최종적으로 생성한 빛 반사 테스트 이미지
#
# 왜 필요한가요?
# 문자 일부에 밝은 영역이 생겼을 때
# Tesseract가 숫자를 어떻게 인식하는지
# 확인하기 위해서입니다.
#
# 참고:
# 흰색 타원 영역에서는 원본 픽셀과 흰색 픽셀이 혼합되어
# 주변보다 밝게 표현됩니다.
# --------------------------------------------------

glare = cv2.addWeighted(
    overlay,
    0.60,
    image,
    0.40,
    0,
)


# --------------------------------------------------
# 10. 네 가지 환경 변화 이미지를 딕셔너리로 정리
#
# 무엇을 하나요?
# - 앞에서 생성한 4가지 테스트 이미지를 정리합니다.
# - 환경 변화 조건을 Key로 사용합니다.
# - 해당 결과 이미지를 Value로 저장합니다.
#
# low_contrast:
# 문자와 배경의 대비가 낮아진 이미지
#
# blur:
# 숫자의 획과 경계가 흐려진 이미지
#
# tilt:
# 숫자 영역이 +8도 기울어진 이미지
#
# glare:
# 숫자 일부에 밝은 반사 영역이 생긴 이미지
#
# 왜 필요한가요?
# 각 테스트 조건을 구분하여
# 서로 다른 폴더에 자동으로 저장하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 11. 환경 변화 조건별 결과 저장 폴더 생성
#
# 무엇을 하나요?
# - variants에서 조건 이름과 결과 이미지를 가져옵니다.
# - 각 조건에 해당하는 저장 폴더를 만듭니다.
#
# variants.items():
# 딕셔너리의 Key와 Value를 함께 가져옵니다.
#
# condition:
# 환경 변화 조건 이름
#
# result:
# 해당 조건으로 생성한 이미지
#
# ROOT / condition:
# 조건별 이미지 저장 경로
#
# 예:
# samples/day06/challenge/blur/
#
# mkdir():
# 결과를 저장할 폴더를 생성합니다.
#
# parents=True:
# 필요한 상위 폴더까지 생성합니다.
#
# exist_ok=True:
# 폴더가 이미 있어도 그대로 사용합니다.
#
# 왜 필요한가요?
# 테스트 조건에 따라 이미지를 분리하여
# 이후 OCR 결과를 쉽게 비교하기 위해서입니다.
# --------------------------------------------------

for condition, result in (
    variants.items()
):

    # 현재 환경 변화 조건의 저장 폴더 지정
    output_dir = (
        ROOT
        / condition
    )

    # 폴더가 없으면 생성
    output_dir.mkdir(
        parents=True,
        exist_ok=True,
    )


    # --------------------------------------------------
    # 12. 환경 변화 테스트 이미지 저장 및 경로 출력
    #
    # 무엇을 하나요?
    # - 현재 조건 이름으로 파일명을 만듭니다.
    # - 전처리 결과 이미지를 JPG로 저장합니다.
    # - 저장 경로를 터미널에 출력합니다.
    #
    # 저장 파일:
    #
    # low_contrast_01.jpg
    # blur_01.jpg
    # tilt_01.jpg
    # glare_01.jpg
    #
    # cv2.imwrite():
    # 이미지 배열을 파일로 저장하는 함수
    #
    # 왜 필요한가요?
    # 다음 실습에서 생성한 이미지를 다시 읽어
    # 동일한 OCR Rule로 검사하기 위해서입니다.
    #
    # 주의:
    # 같은 파일명으로 다시 실행하면 기존 파일을 덮어씁니다.
    # --------------------------------------------------

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


# --------------------------------------------------
# 13. 환경 변화 테스트 이미지 생성 완료 메시지 출력
#
# 무엇을 하나요?
# - 4가지 테스트 이미지 생성이 완료되었다는
#   메시지를 터미널에 출력합니다.
#
# 생성된 테스트 조건:
#
# Low Contrast
# Blur
# Tilt
# Glare
#
# 왜 필요한가요?
# 다음 OCR 성능 검증 실습에 사용할
# 테스트 데이터가 준비되었음을 확인하기 위해서입니다.
#
# 다음 실습:
# 48_day06_challenge.py
#
# 같은 day06-ocr-v1.0 Rule로
# 각 환경 변화 이미지를 OCR하여
# 성공과 실패를 비교합니다.
#
# 주의:
# 이번 코드는 테스트 이미지만 만듭니다.
# 아직 Tesseract OCR이나 OK/NG 판정을 수행하지 않습니다.
# --------------------------------------------------

print()
print(
    "Environment Test Images Created"
)
```
### 의사코드

```
[1. 필요한 라이브러리와 공통 함수 불러오기]

Path를 불러온다.
→ 파일과 폴더 경로 관리

cv2를 불러온다.
→ 이미지 읽기, 변형, 저장

NumPy를 불러온다.
→ 이미지와 같은 크기의 배열 생성

day06_ocr_utils.py에서
공통 기능을 불러온다.

PROJECT_ROOT
→ 프로젝트 최상위 폴더

load_rules()
→ OCR Rule JSON 읽기

          ↓

[2. 대표 SN 이미지와 Challenge 저장 경로 설정]

load_rules() 함수를 호출한다.

          ↓

day06_ocr_rules.json을 읽는다.

          ↓

reference_file을 가져온다.

예:
0637145699424_side.png

          ↓

파일명에서 확장자를 제거한다.

0637145699424_side

          ↓

41번에서 저장한
정상 SN ROI 이미지 경로를 만든다.

outputs/day06/roi/
0637145699424_side_sn_roi.jpg

          ↓

환경 변화 이미지 저장 폴더를 지정한다.

samples/day06/challenge/

          ↓

[3. 정상 SN ROI 이미지 읽기 및 크기 확인]

cv2.imread()로
정상 SN ROI 이미지를 읽는다.

          ↓

이미지를 정상적으로 읽었는지 확인한다.

          ↓

이미지가 없다면
FileNotFoundError를 발생시킨다.

          ↓

이미지가 있다면
이미지의 높이와 너비를 확인한다.

height
→ 이미지 세로 크기

width
→ 이미지 가로 크기

          ↓

[4. Low Contrast 이미지 생성]

원본 이미지와 동일한 크기의
회색 이미지를 만든다.

          ↓

모든 픽셀 값을 128로 설정한다.

          ↓

원본 이미지와 회색 이미지를 혼합한다.

원본 이미지 45%
       +
회색 이미지 55%

          ↓

문자와 배경의 밝기 차이가 줄어든다.

          ↓

Low Contrast 이미지를 만든다.

          ↓

[5. Blur 이미지 생성]

원본 SN ROI를 가져온다.

          ↓

Gaussian Blur를 적용한다.

          ↓

커널 크기를 설정한다.

9 × 9

          ↓

문자 획과 경계가 흐려진다.

          ↓

Blur 이미지를 만든다.

          ↓

[6. Tilt 이미지 생성을 위한 회전 행렬 계산]

원본 이미지의 중심 좌표를 계산한다.

          ↓

가로 중심 = width / 2

세로 중심 = height / 2

          ↓

이미지를 회전시킬 각도를 설정한다.

+8도

          ↓

배율을 설정한다.

1.0
→ 원래 배율 유지

          ↓

cv2.getRotationMatrix2D()를 실행한다.

          ↓

회전에 사용할 변환 행렬을 만든다.

          ↓

[7. 회전 행렬을 이용하여 Tilt 이미지 생성]

원본 SN ROI를 가져온다.

          ↓

앞에서 만든 회전 행렬을 적용한다.

          ↓

cv2.warpAffine()을 실행한다.

          ↓

이미지를 +8도 회전시킨다.

          ↓

회전 후 비어 있는 영역을
흰색으로 채운다.

          ↓

Tilt 이미지를 만든다.

          ↓

[8. Glare 이미지의 밝은 반사 영역 생성]

원본 이미지를 복사한다.

          ↓

overlay 이미지를 만든다.

          ↓

이미지의 크기를 기준으로
타원 중심과 크기를 계산한다.

          ↓

cv2.ellipse()를 실행한다.

          ↓

이미지 위에 흰색 타원을 그린다.

          ↓

숫자 일부를 가리는
밝은 영역을 준비한다.

          ↓

[9. 원본 이미지와 반사 영역 혼합]

흰색 타원이 그려진
overlay 이미지를 가져온다.

          ↓

원본 SN ROI를 가져온다.

          ↓

두 이미지를 혼합한다.

overlay 60%
      +
원본 이미지 40%

          ↓

부분적으로 밝은 빛 반사 효과가 생긴다.

          ↓

Glare 이미지를 만든다.

          ↓

[10. 네 가지 환경 변화 이미지 정리]

지금까지 만든 이미지를 확인한다.

          ↓

Low Contrast
→ 대비가 낮아진 이미지

Blur
→ 흐려진 이미지

Tilt
→ +8도 기울어진 이미지

Glare
→ 밝은 반사 영역이 생긴 이미지

          ↓

네 가지 이미지를
variants 딕셔너리에 저장한다.

          ↓

[11. 환경 변화 조건별 저장 폴더 생성]

variants에서
첫 번째 환경 변화 조건을 가져온다.

          ↓

condition
→ 환경 변화 조건 이름

result
→ 생성한 결과 이미지

          ↓

조건 이름을 이용하여
결과 저장 폴더를 만든다.

예:

samples/day06/challenge/low_contrast/

          ↓

폴더가 없다면 생성한다.

          ↓

[12. 환경 변화 테스트 이미지 저장]

현재 조건의 파일명을 만든다.

예:

low_contrast_01.jpg

          ↓

cv2.imwrite()로
이미지를 저장한다.

          ↓

저장 경로를 터미널에 출력한다.

          ↓

아직 저장하지 않은
환경 변화 이미지가 있는가?

          ├── YES
          │
          ↓
    다음 조건과 이미지를 가져온다.
          │
          ↓
    조건별 저장 폴더를 만든다.
          │
          ↓
    결과 이미지를 저장한다.
          │
          └── 11~12번 반복

          └── NO
                │
                ↓
          네 가지 이미지 저장 완료

          ↓

[13. 환경 변화 이미지 생성 완료]

다음 메시지를 출력한다.

Environment Test Images Created

          ↓

[최종 전체 흐름]

41_day06_select_text_roi.py
          ↓
정상 SN ROI 이미지 생성
          ↓
0637145699424_side_sn_roi.jpg
          ↓
47_day06_make_challenge_variants.py
          ↓
정상 SN ROI 읽기
          ↓
이미지 높이·너비 확인
          ↓
네 가지 환경 변화 생성
          ↓
┌────────────────────────────┐
│ Low Contrast               │
│ 문자·배경 대비 낮추기         │
└────────────────────────────┘
          +
┌────────────────────────────┐
│ Blur                       │
│ Gaussian Blur로 흐리게      │
└────────────────────────────┘
          +
┌────────────────────────────┐
│ Tilt                       │
│ +8도 회전                   │
└────────────────────────────┘
          +
┌────────────────────────────┐
│ Glare                      │
│ 밝은 빛 반사 영역 추가        │
└────────────────────────────┘
          ↓
환경 변화 이미지 4장 생성
          ↓
조건별 폴더에 저장
          ↓
samples/day06/challenge/
          ↓
다음 실습
48_day06_challenge.py
          ↓
Freeze된 OCR Rule 적용
          ↓
Tesseract OCR
          ↓
실제 SN과 결과 비교
          ↓
촬영환경별 OCR 성공 / 실패 확인
          ↓
OCR 검사기의 한계 분석
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
    # 1. 원본 이미지의 높이와 너비 확인
    # → 이미지 중심점을 계산하기 위해 사용
    height, width = (
        image.shape[:2]
    )

    # 2. 이미지의 중심 좌표 계산
    # → 이미지를 중앙을 기준으로 회전하기 위해 사용
    center = (
        width / 2,
        height / 2,
    )

    # 3. 회전 변환 행렬 생성
    # → angle: 회전 각도 (양수는 반시계 방향)
    # → 1.0: 원본 이미지 배율 유지
    matrix = (
        cv2.getRotationMatrix2D(
            center,
            angle,
            1.0,
        )
    )

    # 4. 이미지 종류에 따라 빈 공간을 채울 흰색 설정
    # → Grayscale: 255 (흰색)
    # → 컬러(BGR): (255, 255, 255) (흰색)
    if image.ndim == 2:
        border_value = 255
    else:
        border_value = (
            255,
            255,
            255,
        )

    # 5. 이미지 회전 실행 후 결과 반환
    # → warpAffine(): 회전 행렬을 실제 이미지에 적용
    # → INTER_LINEAR: 픽셀을 선형 보간
    # → BORDER_CONSTANT: 회전 후 빈 공간을 지정한 색으로 채움
    # → 출력 이미지 크기는 원본과 동일하게 유지
    return cv2.warpAffine(
        image,
        matrix,
        (width, height),
        flags=cv2.INTER_LINEAR,
        borderMode=cv2.BORDER_CONSTANT,
        borderValue=border_value,
    )
```

의사코드

```
원본 이미지 입력
       ↓
이미지 크기 확인
       ↓
이미지 중심점 계산
       ↓
지정한 각도로 회전 행렬 생성
       ↓
Grayscale / 컬러 이미지 구분
       ↓
회전 후 빈 공간은 흰색 처리
       ↓
이미지 회전 실행
       ↓
회전된 이미지 반환
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
# --------------------------------------------------
# 1. 환경 변화 OCR 검사에 필요한 라이브러리 불러오기
#
# 무엇을 하나요?
# - CSV 결과 저장, 이미지 처리, 파일 경로 기능을 준비합니다.
# - 기존 OCR 공통 함수와 문자열 검사 함수를 불러옵니다.
#
# csv:
# 환경별 OCR 결과를 CSV 파일로 저장
#
# Path:
# 이미지 파일명에서 실제 SN 정답 추출
#
# cv2:
# 이미지 읽기, 4배 확대, 결과 이미지 저장
#
# load_rules():
# 확정된 OCR Rule JSON 읽기
#
# preprocess_for_ocr():
# JSON에서 선택한 전처리 적용
#
# rotate_image():
# 이미지를 지정한 각도로 회전
# Tilt 이미지의 기울기 보정에 사용
#
# run_ocr():
# Tesseract OCR 실행
#
# inspect_text():
# 문자열 Cleaning, 형식 검사, Expected SN 비교,
# 최종 OK/NG 판정
#
# 왜 필요한가요?
# 앞에서 만든 공통 함수를 재사용하여
# 촬영환경 변화에 따른 OCR 결과를 비교하기 위해서입니다.
#
# 주의:
# rotate_image() 함수는 day06_ocr_utils.py에
# 미리 정의되어 있어야 합니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 2. 환경 변화 이미지와 결과 저장 경로 설정
#
# 무엇을 하나요?
# - 47번에서 만든 환경 변화 이미지 폴더를 지정합니다.
# - OCR 입력 결과 이미지를 저장할 폴더를 지정합니다.
# - OCR 비교 결과 CSV 경로를 지정합니다.
#
# ROOT:
# Low Contrast, Blur, Tilt, Glare 이미지가 있는 폴더
#
# OUTPUT_DIR:
# 실제 OCR에 전달한 확대 이미지를 저장할 폴더
#
# REPORT_PATH:
# 환경 변화별 OCR 검사 결과를 저장할 CSV
#
# 왜 필요한가요?
# 네 가지 조건의 테스트 이미지를 읽고
# 검사 결과를 구분하여 저장하기 위해서입니다.
# --------------------------------------------------

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


# --------------------------------------------------
# 3. 확정된 OCR Rule 읽기 및 버전 확인
#
# 무엇을 하나요?
# - load_rules() 함수를 호출합니다.
# - 44번에서 확정한 OCR 설정을 읽습니다.
# - Rule 버전이 day06-ocr-v1.0인지 확인합니다.
#
# 이번 실습의 OCR 조건:
#
# Baseline Version = day06-ocr-v1.0
# Preprocess = original
# PSM = 7
# Whitelist = 0123456789
# Regex = ^[0-9]{13}$
#
# 왜 필요한가요?
# 환경 변화에 따른 OCR 성능을 비교할 때
# OCR Rule 자체가 변경되면 비교가 어려워지기 때문입니다.
#
# 주의:
# 이 코드는 버전 문자열만 확인합니다.
# JSON의 모든 설정이 실제로 동일한지까지
# 검증하는 것은 아닙니다.
# --------------------------------------------------

rules = load_rules()


if rules[
    "baseline_version"
] != "day06-ocr-v1.0":

    raise RuntimeError(
        "OCR Rule이 Freeze되지 않았습니다."
    )


# --------------------------------------------------
# 4. 대표 SN 정답과 결과 저장 폴더 준비
#
# 무엇을 하나요?
# - JSON에서 대표 이미지 파일명을 가져옵니다.
# - 파일명에서 실제 SN 숫자 부분만 추출합니다.
# - 결과 이미지 폴더를 생성합니다.
# - 검사 결과를 저장할 빈 리스트를 준비합니다.
#
# reference_file:
# 대표 제품 이미지 파일명
#
# 예:
# 0637145699424_side.png
#
# Path(reference_file).stem:
# 확장자 제거
#
# .split("_side")[0]:
# "_side" 앞의 SN 숫자 추출
#
# EXPECTED_SN:
# 환경 변화 이미지에서 읽어야 하는 원본 SN
#
# 왜 필요한가요?
# 네 가지 환경 변화 이미지는 모두
# 동일한 대표 SN을 변형해서 만들었기 때문입니다.
#
# 참고:
# 이번 실습에서 EXPECTED_SN은
# OCR이 원본 SN을 정확하게 읽었는지 확인하는
# 정답 문자열로 사용합니다.
# --------------------------------------------------

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


# 환경 변화별 OCR 결과를 저장할 리스트
rows = []


# --------------------------------------------------
# 5. 네 가지 환경 변화 폴더에서 이미지 읽기
#
# 무엇을 하나요?
# - 네 가지 환경 변화 조건을 순서대로 확인합니다.
# - 각 조건의 폴더에서 JPG 이미지를 찾습니다.
# - 이미지를 OpenCV로 읽습니다.
#
# 환경 변화 조건:
#
# low_contrast:
# 문자와 배경의 밝기 차이가 낮아진 상태
#
# blur:
# 숫자의 획이 흐려진 상태
#
# tilt:
# 숫자 줄이 +8도 기울어진 상태
#
# glare:
# 빛 반사로 숫자 일부가 밝아진 상태
#
# input_dir.glob("*.jpg"):
# 해당 폴더에서 JPG 파일을 찾습니다.
#
# sorted():
# 파일명 순서로 정렬합니다.
#
# cv2.imread():
# 이미지 파일을 읽는 함수입니다.
#
# if roi is None:
# 이미지를 읽지 못한 경우 다음 이미지로 이동합니다.
#
# 왜 필요한가요?
# 각각의 촬영환경에서 같은 OCR Rule의
# 인식 결과를 비교하기 위해서입니다.
#
# 주의:
# JPG 파일이 없는 조건은 검사하지 않고 넘어갑니다.
# --------------------------------------------------

for condition in [
    "low_contrast",
    "blur",
    "tilt",
    "glare",
]:

    # 현재 환경 변화 조건의 이미지 폴더
    input_dir = (
        ROOT
        / condition
    )

    # 해당 조건의 JPG 이미지 반복
    for image_path in sorted(
        input_dir.glob("*.jpg")
    ):

        # 환경 변화 SN ROI 읽기
        roi = cv2.imread(
            str(image_path)
        )

        # 이미지 읽기에 실패하면 다음 이미지로 이동
        if roi is None:
            continue


        # --------------------------------------------------
        # 6. Freeze된 전처리 방법 적용
        #
        # 무엇을 하나요?
        # - preprocess_for_ocr() 공통 함수를 호출합니다.
        # - JSON에서 확정한 전처리 방법을 적용합니다.
        #
        # 현재 설정:
        # preprocess_mode = original
        #
        # original:
        # 추가적인 Gray, CLAHE, Otsu 등을 적용하지 않고
        # 입력 ROI를 그대로 사용하는 방법입니다.
        #
        # processed:
        # OCR에 사용할 전처리 결과 이미지
        #
        # 왜 필요한가요?
        # 정상 이미지와 환경 변화 이미지 모두
        # 동일한 전처리 조건에서 비교하기 위해서입니다.
        #
        # 주의:
        # 환경 변화에 따라 OCR 설정을 변경하지 않습니다.
        # --------------------------------------------------

        processed = (
            preprocess_for_ocr(
                roi,
                rules,
            )
        )


        # --------------------------------------------------
        # 7. OCR 입력 이미지를 가로·세로 4배 확대
        #
        # 무엇을 하나요?
        # - 전처리한 SN 이미지를 4배 확대합니다.
        # - 43번과 45번에서 사용한 확대 조건을 유지합니다.
        #
        # cv2.resize():
        # 이미지 크기를 변경하는 함수
        #
        # fx=4.0:
        # 가로 4배 확대
        #
        # fy=4.0:
        # 세로 4배 확대
        #
        # INTER_CUBIC:
        # 주변 픽셀을 이용하는 보간 방법
        #
        # 왜 필요한가요?
        # 이미지 확대율을 동일하게 유지해야
        # 환경 변화에 따른 OCR 결과를 비교할 수 있기 때문입니다.
        # --------------------------------------------------

        ocr_input = cv2.resize(
            processed,
            None,
            fx=4.0,
            fy=4.0,
            interpolation=cv2.INTER_CUBIC,
        )


        # --------------------------------------------------
        # 8. 환경 변화 이미지에 Tesseract OCR 실행
        #
        # 무엇을 하나요?
        # - run_ocr() 공통 함수를 호출합니다.
        # - 확대된 이미지와 OCR Rule을 전달합니다.
        # - Tesseract가 인식한 Raw 문자열을 반환받습니다.
        #
        # OCR 설정:
        #
        # OEM = 3
        # PSM = 7
        # Whitelist = 0123456789
        #
        # raw_text:
        # Tesseract가 읽은 원본 문자열
        #
        # 예:
        # '0637145699424\n'
        #
        # 왜 필요한가요?
        # 흐림, 기울기, 대비 저하, 빛 반사 상태에서
        # 같은 OCR 설정이 얼마나 정확하게 동작하는지
        # 확인하기 위해서입니다.
        # --------------------------------------------------

        raw_text = run_ocr(
            ocr_input,
            rules,
        )


        # --------------------------------------------------
        # 9. OCR 문자열 검사 및 원본 SN과 정확도 비교
        #
        # 무엇을 하나요?
        # - inspect_text() 함수를 호출합니다.
        # - OCR 문자열의 공백과 줄바꿈을 제거합니다.
        # - 숫자 13자리 형식인지 확인합니다.
        # - 대표 SN 정답과 비교하여 OK/NG를 판정합니다.
        # - OCR 문자열이 정답과 정확히 같은지 확인합니다.
        #
        # result["final_text"]:
        # 정리된 OCR 문자열
        #
        # EXPECTED_SN:
        # 원본 대표 이미지의 실제 SN
        #
        # ocr_exact:
        # 두 문자열이 정확히 일치하는지 나타냅니다.
        #
        # True:
        # OCR이 원본 SN을 정확하게 읽은 경우
        #
        # False:
        # 하나라도 잘못 읽거나 인식하지 못한 경우
        #
        # 왜 필요한가요?
        # 촬영환경 변화로 OCR 인식 결과가
        # 달라졌는지 판단하기 위해서입니다.
        #
        # 참고:
        # 이번 테스트에서는 모든 변형 이미지의
        # 정답 SN이 동일합니다.
        # --------------------------------------------------

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


        # --------------------------------------------------
        # 10. 실제 Tesseract OCR 입력 이미지 저장
        #
        # 무엇을 하나요?
        # - 현재 환경 변화 조건으로 파일명을 만듭니다.
        # - 4배 확대된 OCR 입력 이미지를 저장합니다.
        #
        # 저장 파일 예:
        #
        # low_contrast_ocr_input.jpg
        # blur_ocr_input.jpg
        # tilt_ocr_input.jpg
        # glare_ocr_input.jpg
        #
        # 왜 필요한가요?
        # OCR에 실제로 전달한 이미지를 확인하여
        # 실패 원인을 분석하기 위해서입니다.
        #
        # 주의:
        # 같은 조건에 이미지가 여러 장 있으면
        # 동일한 파일명으로 저장되어 덮어써집니다.
        # 이번 47번 실습에서는 조건별 1장을 생성합니다.
        # --------------------------------------------------

        output_path = (
            OUTPUT_DIR
            / f"{condition}_ocr_input.jpg"
        )

        cv2.imwrite(
            str(output_path),
            ocr_input,
        )


        # --------------------------------------------------
        # 11. Deskew를 적용하지 않은 OCR 결과 기록
        #
        # 무엇을 하나요?
        # - 현재 환경 변화 조건의 OCR 결과를 정리합니다.
        # - Deskew 여부를 "no"로 기록합니다.
        # - 결과를 rows 리스트에 추가합니다.
        #
        # condition:
        # 현재 환경 변화 조건 이름
        #
        # deskew:
        # 기울기 보정 사용 여부
        #
        # expected_sn:
        # 실제로 읽어야 하는 대표 SN
        #
        # raw_text:
        # Tesseract 원본 문자열
        #
        # final_text:
        # 공백과 줄바꿈을 제거한 OCR 문자열
        #
        # format_ok:
        # 숫자 13자리 형식 검사 결과
        #
        # expected_match:
        # Expected SN과 일치 여부
        #
        # ocr_exact:
        # OCR 결과가 원본 SN과 정확히 같은지 확인
        #
        # status:
        # 최종 OK / NG 판정
        #
        # reasons:
        # NG 판정 이유
        #
        # 왜 필요한가요?
        # 기울기 보정 이전의 OCR 결과를 기록하여
        # 이후 Deskew 결과와 비교하기 위해서입니다.
        # --------------------------------------------------

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


        # --------------------------------------------------
        # 12. Deskew를 적용하지 않은 OCR 결과 출력
        #
        # 무엇을 하나요?
        # - 현재 환경 변화 조건을 출력합니다.
        # - Deskew를 적용하지 않았음을 표시합니다.
        # - OCR 문자열과 정답 일치 여부를 출력합니다.
        #
        # Condition:
        # 현재 테스트 환경
        #
        # Deskew: no
        # 기울기 보정을 적용하지 않음
        #
        # OCR:
        # Tesseract가 인식한 문자열
        #
        # Exact:
        # 원본 SN과 완전히 일치하는지 확인한 결과
        #
        # 왜 필요한가요?
        # 환경 변화별 OCR 성공과 실패를
        # 터미널에서 바로 확인하기 위해서입니다.
        # --------------------------------------------------

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


        # --------------------------------------------------
        # 13. Tilt 이미지에만 -8도 Deskew 적용
        #
        # 무엇을 하나요?
        # - 현재 환경 변화 조건이 tilt인지 확인합니다.
        # - Tilt 조건일 때만 추가 OCR 실험을 수행합니다.
        # - rotate_image()로 이미지를 -8도 회전시킵니다.
        # - 확정된 전처리를 다시 적용합니다.
        #
        # 47번 실습:
        # 정상 이미지 → +8도 Tilt
        #
        # 이번 48번 실습:
        # +8도 Tilt 이미지 → -8도 Deskew
        #
        # Deskew란?
        # 기울어진 문자 줄을 다시 수평에 가깝게
        # 보정하는 영상처리 방법입니다.
        #
        # 왜 필요한가요?
        # OCR 실패 원인이 문자 기울기라면
        # 기울기 보정 후 결과가 개선되는지 확인하기 위해서입니다.
        #
        # 주의:
        # -8도는 이번 실습에서 이미 알고 있는
        # +8도의 반대 각도를 사용하는 것입니다.
        #
        # 실제 산업현장에서는 기울어진 각도를
        # 별도로 추정해야 할 수 있습니다.
        # --------------------------------------------------

        if condition == "tilt":

            # 기울어진 이미지를 반대 방향으로 8도 회전
            deskewed = rotate_image(
                roi,
                -8.0,
            )

            # Deskew 결과에 확정된 전처리 적용
            deskew_processed = (
                preprocess_for_ocr(
                    deskewed,
                    rules,
                )
            )


            # --------------------------------------------------
            # 14. Deskew 이미지를 4배 확대하고 OCR 실행
            #
            # 무엇을 하나요?
            # - Deskew 결과 이미지를 4배 확대합니다.
            # - Tesseract OCR을 다시 실행합니다.
            # - 인식한 문자열을 검사합니다.
            # - 원본 SN과 일치하는지 확인합니다.
            #
            # deskew_input:
            # 기울기 보정 후 4배 확대된 이미지
            #
            # deskew_raw:
            # Tesseract가 반환한 원본 문자열
            #
            # deskew_result:
            # 문자열 형식 검사 및 OK/NG 판정 결과
            #
            # deskew_exact:
            # Deskew 후 OCR이 정답 SN을 정확하게 읽었는지
            # 확인하는 값
            #
            # 왜 필요한가요?
            # 동일한 Tilt 이미지에서
            # Deskew 적용 전후 OCR 결과를 비교하기 위해서입니다.
            #
            # 중요:
            # Deskew를 적용했다고 해서 OCR이 반드시
            # 성공하는 것은 아닙니다.
            # 회전 과정에서 이미 손실된 문자 정보는
            # 완전히 복원되지 않을 수 있습니다.
            # --------------------------------------------------

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


            # --------------------------------------------------
            # 15. Deskew 후 OCR 입력 이미지 저장
            #
            # 무엇을 하나요?
            # - 기울기 보정 후 실제 OCR에 입력한
            #   확대 이미지를 저장합니다.
            #
            # 저장 파일:
            # tilt_deskew_ocr_input.jpg
            #
            # 왜 필요한가요?
            # 기존 Tilt 이미지와 Deskew 결과를 비교하여
            # 숫자 줄이 얼마나 수평에 가까워졌는지
            # 확인하기 위해서입니다.
            # --------------------------------------------------

            cv2.imwrite(
                str(
                    OUTPUT_DIR
                    / "tilt_deskew_ocr_input.jpg"
                ),
                deskew_input,
            )


            # --------------------------------------------------
            # 16. Deskew 적용 후 OCR 결과 기록 및 출력
            #
            # 무엇을 하나요?
            # - Deskew 적용 결과를 CSV 저장 목록에 추가합니다.
            # - deskew 값을 "yes"로 기록합니다.
            # - OCR 문자열과 Exact Match 결과를 출력합니다.
            #
            # 기존 Tilt 결과:
            # condition = tilt
            # deskew = no
            #
            # Deskew 적용 결과:
            # condition = tilt
            # deskew = yes
            #
            # 왜 필요한가요?
            # 같은 이미지에서 기울기 보정 적용 전후
            # 결과를 서로 구분하여 비교하기 위해서입니다.
            #
            # 이번 실습에서 생성하는 결과 행:
            #
            # Low Contrast → 1행
            # Blur         → 1행
            # Tilt         → 2행 (Deskew 전/후)
            # Glare        → 1행
            #
            # 총 5행
            #
            # 단, 조건별 입력 이미지가 1장씩
            # 정상적으로 읽혔을 때의 결과입니다.
            # --------------------------------------------------

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


# --------------------------------------------------
# 17. 검사 결과 존재 여부 확인 및 CSV 저장
#
# 무엇을 하나요?
# - 테스트 결과가 한 건 이상 있는지 확인합니다.
# - 결과가 없다면 오류를 발생시킵니다.
# - CSV 저장 폴더를 준비합니다.
# - 환경 변화별 OCR 결과를 CSV로 저장합니다.
#
# if not rows:
# 검사 결과가 비어 있는지 확인합니다.
#
# csv.DictWriter():
# 딕셔너리 목록을 CSV로 저장하는 기능
#
# fieldnames=list(rows[0].keys()):
# 첫 번째 결과 딕셔너리의 Key를 CSV 컬럼명으로 사용
#
# writer.writeheader():
# 컬럼명 작성
#
# writer.writerows(rows):
# 전체 검사 결과 저장
#
# encoding="utf-8-sig":
# Excel에서 한글이 깨질 가능성을 줄입니다.
#
# 저장 파일:
# reports/day06_challenge_results.csv
#
# 왜 필요한가요?
# 환경 변화 조건별 OCR 성공과 실패를
# 표로 확인하고 비교하기 위해서입니다.
#
# 주의:
# 이번 코드에서는 결과 CSV가 이미 존재하면
# 기존 파일 내용을 덮어씁니다.
# --------------------------------------------------

if not rows:
    raise RuntimeError(
        "환경 변화 테스트 이미지가 없습니다."
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
        fieldnames=list(
            rows[0].keys()
        ),
    )

    writer.writeheader()

    writer.writerows(
        rows
    )


# --------------------------------------------------
# 18. 환경 변화 OCR 테스트 완료 메시지 출력
#
# 무엇을 하나요?
# - 결과 CSV 저장 경로를 출력합니다.
# - 환경 변화 OCR 검증 완료 메시지를 출력합니다.
#
# 왜 필요한가요?
# 네 가지 환경 변화 및 Tilt Deskew 실험 결과가
# 저장되었음을 확인하기 위해서입니다.
#
# 저장 결과:
# reports/day06_challenge_results.csv
#
# 다음 단계에서는 다음 내용을 확인합니다.
#
# - 어떤 환경 변화에서 OCR이 실패했는가?
# - Tilt 이미지의 Deskew 전후 결과가 달라졌는가?
# - 원본 Baseline Rule의 한계는 무엇인가?
#
# 주의:
# 이번 코드는 OCR 결과를 비교하는 실험입니다.
# 모든 환경에서 정확하게 인식하는
# 새로운 OCR Rule을 자동으로 만들어 주지는 않습니다.
# --------------------------------------------------

print()

print(
    "Saved:",
    REPORT_PATH
)

print(
    "Environment Stress Test Complete"
)
```
### 의사코드

```
[1. 필요한 라이브러리와 공통 함수 불러오기]

csv를 불러온다.
→ 환경 변화 검사 결과 저장

Path를 불러온다.
→ 대표 이미지 파일명에서 SN 추출

cv2를 불러온다.
→ 이미지 읽기, 확대, 저장

day06_ocr_utils.py에서
공통 함수를 불러온다.

load_rules()
→ 확정된 OCR Rule 읽기

preprocess_for_ocr()
→ 선택된 전처리 적용

rotate_image()
→ 이미지 기울기 보정

run_ocr()
→ Tesseract OCR 실행

          ↓

day06_ocr_validation.py에서
inspect_text() 함수를 불러온다.

→ 문자열 정리
→ 숫자 13자리 형식 검사
→ Expected SN 비교
→ OK / NG 판정

          ↓

[2. 입력 이미지와 결과 저장 경로 설정]

환경 변화 테스트 이미지 폴더를 지정한다.

samples/day06/challenge/

          ↓

OCR 입력 이미지 저장 폴더를 지정한다.

outputs/day06/challenge/

          ↓

결과 CSV 경로를 지정한다.

reports/day06_challenge_results.csv

          ↓

[3. 확정된 OCR Rule 읽기 및 확인]

load_rules() 함수를 호출한다.

          ↓

day06_ocr_rules.json을 읽는다.

          ↓

현재 Rule 버전을 확인한다.

          ↓

day06-ocr-v1.0인가?

          ├── NO
          │
          ↓
    RuntimeError 발생
    프로그램 중단

          └── YES
                │
                ↓
          다음 단계 진행

          ↓

[4. 대표 SN 정답과 결과 폴더 준비]

JSON에서 reference_file을 읽는다.

예:
0637145699424_side.png

          ↓

파일 확장자를 제거한다.

0637145699424_side

          ↓

_side 앞의 숫자를 추출한다.

0637145699424

          ↓

EXPECTED_SN에 저장한다.

          ↓

OCR 결과 이미지 폴더를 생성한다.

          ↓

결과를 기록할 빈 리스트를 만든다.

rows = []

          ↓

[5. 네 가지 환경 변화 이미지 읽기]

환경 변화 조건을 순서대로 가져온다.

① low_contrast
② blur
③ tilt
④ glare

          ↓

현재 조건의 폴더를 찾는다.

예:
samples/day06/challenge/low_contrast/

          ↓

폴더 안의 JPG 이미지를 찾는다.

          ↓

첫 번째 이미지를 읽는다.

          ↓

이미지를 읽지 못했다면
다음 이미지로 이동한다.

          ↓

정상적으로 읽었다면
OCR 검사를 시작한다.

          ↓

[6. Freeze된 전처리 적용]

preprocess_for_ocr() 함수를 호출한다.

          ↓

JSON에서 선택한 전처리를 확인한다.

preprocess_mode = original

          ↓

확정된 전처리를 적용한다.

          ↓

processed 이미지를 만든다.

          ↓

[7. OCR 입력 이미지 4배 확대]

cv2.resize()를 실행한다.

          ↓

이미지 가로 크기를 4배 확대한다.

          ↓

이미지 세로 크기를 4배 확대한다.

          ↓

INTER_CUBIC 보간법을 사용한다.

          ↓

ocr_input 이미지를 만든다.

          ↓

[8. Tesseract OCR 실행]

run_ocr() 함수를 호출한다.

          ↓

확대된 이미지와 OCR Rule을 전달한다.

          ↓

Tesseract가 숫자를 인식한다.

          ↓

Raw OCR 문자열을 반환받는다.

예:
'0637145699424\n'

          ↓

[9. 문자열 검사 및 SN 정답 비교]

inspect_text() 함수를 호출한다.

          ↓

Raw OCR 문자열을 Cleaning한다.

          ↓

숫자 13자리인지 검사한다.

          ↓

EXPECTED_SN과 비교한다.

          ↓

최종 OK / NG를 판정한다.

          ↓

정리된 OCR 문자열을 가져온다.

          ↓

실제 SN 정답과 정확히 같은지 확인한다.

          ↓

완전히 일치하는가?

          ├── YES
          │
          ↓
    ocr_exact = True

          └── NO
                │
                ↓
          ocr_exact = False

          ↓

[10. 실제 OCR 입력 이미지 저장]

현재 환경 변화 조건 이름을 확인한다.

          ↓

결과 이미지 파일명을 만든다.

예:
low_contrast_ocr_input.jpg

          ↓

Tesseract에 전달했던
4배 확대 이미지를 저장한다.

          ↓

[11. Deskew 미적용 OCR 결과 기록]

현재 검사 결과를 정리한다.

          ↓

condition
→ 환경 변화 조건

deskew = no
→ 기울기 보정을 적용하지 않음

expected_sn
→ SN 정답

raw_text
→ OCR 원본 문자열

final_text
→ 정리된 OCR 문자열

format_ok
→ 숫자 13자리 형식 검사

expected_match
→ 정답 SN 일치 여부

ocr_exact
→ OCR 정확 인식 여부

status
→ 최종 OK / NG

reasons
→ NG 판정 이유

          ↓

결과를 rows 리스트에 추가한다.

          ↓

[12. Deskew 미적용 결과 출력]

현재 환경 변화 조건을 출력한다.

          ↓

Deskew: no

를 출력한다.

          ↓

OCR이 읽은 문자열을 출력한다.

          ↓

Exact Match 결과를 출력한다.

          ↓

[13. Tilt 이미지에만 Deskew 적용]

현재 환경 변화 조건을 확인한다.

          ↓

condition == tilt 인가?

          ├── NO
          │
          ↓
    Deskew 추가 실험을 건너뛴다.
    다음 이미지 검사로 이동한다.

          └── YES
                │
                ↓
          Deskew 추가 실험 실행

          ↓

47번에서 +8도로 회전시킨
Tilt 이미지를 가져온다.

          ↓

rotate_image()를 호출한다.

          ↓

회전 각도를 -8도로 설정한다.

          ↓

이미지를 반대 방향으로 회전시킨다.

          ↓

Deskew 이미지를 만든다.

          ↓

Freeze된 전처리를 다시 적용한다.

          ↓

[14. Deskew 이미지 확대 및 OCR 재검사]

Deskew 이미지를 4배 확대한다.

          ↓

run_ocr() 함수를 다시 호출한다.

          ↓

Tesseract OCR을 실행한다.

          ↓

Deskew 후 OCR 문자열을 가져온다.

          ↓

inspect_text()로 문자열을 검사한다.

          ↓

숫자 13자리 형식인지 확인한다.

          ↓

EXPECTED_SN과 비교한다.

          ↓

OCR 문자열이 실제 SN과 일치하는가?

          ├── YES
          │
          ↓
    deskew_exact = True

          └── NO
                │
                ↓
          deskew_exact = False

          ↓

[15. Deskew 후 OCR 입력 이미지 저장]

Deskew 후 확대된 이미지를 가져온다.

          ↓

다음 파일명으로 저장한다.

tilt_deskew_ocr_input.jpg

          ↓

[16. Deskew 적용 후 OCR 결과 기록]

condition = tilt

deskew = yes

로 설정한다.

          ↓

Deskew 후 OCR 문자열을 기록한다.

          ↓

형식 검사 결과를 기록한다.

          ↓

Expected SN 일치 여부를 기록한다.

          ↓

deskew_exact를 기록한다.

          ↓

최종 OK/NG와 NG Reason을 기록한다.

          ↓

rows 리스트에 추가한다.

          ↓

Deskew: yes

를 출력한다.

          ↓

Deskew 후 OCR 문자열을 출력한다.

          ↓

Exact Match 결과를 출력한다.

          ↓

아직 처리할 이미지가 남아 있는가?

          ├── YES
          │
          ↓
    다음 이미지를 가져온다.
          │
          ↓
    5~16번에서 해당하는 과정을 반복한다.

          └── NO
                │
                ↓
          전체 이미지 검사 완료

          ↓

[17. 검사 결과 확인 및 CSV 저장]

rows 리스트가 비어 있는지 확인한다.

          ↓

검사 결과가 하나도 없는가?

          ├── YES
          │
          ↓
    RuntimeError 발생
    프로그램 중단

          └── NO
                │
                ↓
          CSV 저장 진행

          ↓

reports 폴더가 있는지 확인한다.

          ↓

폴더가 없다면 생성한다.

          ↓

CSV 컬럼명을 준비한다.

condition
deskew
expected_sn
raw_text
final_text
format_ok
expected_match
ocr_exact
status
reasons

          ↓

day06_challenge_results.csv 파일을 연다.

          ↓

첫 번째 행에 컬럼명을 작성한다.

          ↓

rows에 저장된 전체 검사 결과를 기록한다.

          ↓

CSV 파일을 저장한다.

          ↓

[18. 환경 변화 OCR 테스트 완료]

결과 CSV 저장 경로를 출력한다.

          ↓

완료 메시지를 출력한다.

Environment Stress Test Complete

          ↓

[최종 전체 흐름]

47_day06_make_challenge_variants.py
          ↓
정상 SN ROI에서 4가지 테스트 이미지 생성
          ↓
┌─────────────────────────────┐
│ Low Contrast                │
│ Blur                        │
│ Tilt (+8도)                 │
│ Glare                       │
└─────────────────────────────┘
          ↓
48_day06_challenge.py
          ↓
Freeze된 OCR Rule 읽기
          ↓
4가지 환경 변화 이미지 검사
          ↓
Original 전처리
          ↓
4배 확대
          ↓
Tesseract OCR
          ↓
숫자 13자리 검사
          ↓
EXPECTED_SN 비교
          ↓
Exact Match 기록
          ↓
조건이 Tilt인가?
          │
          ├── NO
          │     ↓
          │   일반 OCR 결과만 기록
          │
          └── YES
                ↓
              기존 Tilt OCR 결과 기록
                ↓
              -8도 Deskew 추가 적용
                ↓
              4배 확대
                ↓
              Tesseract OCR 재실행
                ↓
              Deskew 후 Exact Match 기록
                ↓
              전후 결과 비교
          ↓
모든 조건 검사 완료
          ↓
OCR 입력 이미지 저장
          +
day06_challenge_results.csv 저장
          ↓
촬영환경 변화에 따른
OCR 성능과 한계 확인
```

실행합니다.

```
python src/48_day06_challenge.py
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

## OCR 실패 시 성능을 개선하는 방법

PART6에서는 정상 이미지에서 잘 작동하던 Tesseract OCR이 **Blur(흐림), Tilt(기울어짐), Glare(빛 반사)** 환경에서 실패하는 것을 확인했습니다.

그렇다면 실패한 이미지를 더 수집하여 AI를 학습시켜야 할까요?

**반드시 재학습할 필요는 없습니다.** 이번 실습은 이미 학습된 Tesseract 엔진을 사용하므로, 먼저 촬영 환경과 전처리 방법을 개선하는 것이 좋습니다.

### OCR 성능을 높이는 대표적인 방법

|개선 방법|구체적인 해결 방법|
|---|---|
|**① 촬영 환경 개선**|Blur는 초점·셔터속도 조정, Glare는 조명 각도 변경·확산 조명 등으로 개선|
|**② 이미지 전처리 개선**|대비 조절, 노이즈 제거, 기울기 보정 등을 적용하여 OCR 결과 비교|
|**③ Tesseract 설정 조정**|PSM, 이미지 확대율 등 OCR 옵션을 변경하여 정확도 비교|
|**④ OCR 엔진 변경**|개선이 어렵다면 PaddleOCR 등 다른 사전학습 OCR 엔진과 비교|
|**⑤ 데이터 수집 및 추가 학습**|다양한 흐림·반사·기울기 이미지를 수집하고, 필요하다면 학습 가능한 OCR 모델을 미세조정(Fine-tuning)|

### 실제 현장에서는 어떤 순서로 개선할까요?

```
OCR 실패 이미지 분석
        ↓
촬영 환경 개선
        ↓
전처리 및 Tesseract 설정 변경
        ↓
새로운 이미지로 정확도 재평가
        ↓
성능이 부족하다면?
        ↓
다른 OCR 엔진 또는 추가 학습 검토
        ↓
개선된 OCR Rule 확정
```

주의할 점은 **반사로 문자가 완전히 가려지거나 심하게 흐려져 정보가 사라진 경우에는 AI 재학습만으로 복구하기 어렵다**는 것입니다. 이러한 이미지는 재촬영하거나 별도로 확인해야 합니다.

또한 OCR이 실패했다고 곧바로 제품 인쇄 불량으로 확정하지 않고, **OCR 인식 실패와 실제 제품 불량을 구분**해야 합니다.

> **핵심:** OCR 성능 개선은 무조건 데이터를 더 모아 재학습하는 것이 아닙니다. **촬영 환경 개선 → 전처리·설정 최적화 → 다른 OCR 엔진 검토 → 필요시 추가 학습** 순서로 접근합니다.

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

```markdown
# Day 06 — OCR 검사 결과 및 실패 분석

## 1. 정상 제품 이미지 6장 검사

확정한 OCR 설정(`day06-ocr-v1.0`)을 제품 이미지 6장에 동일하게 적용했습니다.

|설정 항목|적용 값|
|---|---|
|전처리|`original` (원본 ROI 사용)|
|이미지 확대|4배|
|OCR 엔진|Tesseract (`PSM 7`: 한 줄 문자 인식)|
|Whitelist|숫자 `0~9`만 허용|
|Regex|`^[0-9]{13}$` (숫자 13자리)|

**실제 평가 결과**

|평가 항목|결과|의미|
|---|---|---|
|OCR 정확도|**6/6 (100%)**|실제 인쇄된 SN을 모두 정확하게 읽음|
|제품 판정 정확도|**6/6 (100%)**|OK 4장, 교육용 NG 2장을 모두 정답대로 판정|

> **주의:** 제품 판정 정확도 100%는 모든 제품이 OK라는 뜻이 아닙니다. **OK와 NG를 모두 올바르게 구분했다는 뜻**입니다. 이 실습의 NG 2장은 Expected SN을 다르게 설정한 교육용 상황입니다.

## 2. 영상 상태 변화 테스트

이번에는 같은 SN `0637145699424`에 흐림·기울기·빛 반사 등의 조건을 적용하고, **OCR 설정은 바꾸지 않은 채** 결과를 비교했습니다.

|테스트 조건|실제 OCR 결과|인식 성공 여부|확인한 내용|
|---|---|---|---|
|Low Contrast (낮은 대비)|`0637145699424`|**성공**|글자가 옅어져도 정확히 인식함|
|Blur (흐림)|빈 문자열|실패|숫자 획이 흐려져 읽지 못함|
|Tilt (기울어짐)|`9728`|실패|13자리 중 일부 숫자만 인식함|
|Tilt + Deskew (기울기 보정)|빈 문자열|실패|단순 회전 보정만으로 복구되지 않음|
|Glare (빛 반사)|빈 문자열|실패|반사로 문자 정보가 가려져 읽지 못함|

**결과: 5개 조건 중 성공 1개, 실패 4개.**

※ 여기서 ‘실패’는 **OCR 인식 실패**를 의미합니다. 실제 제품에 SN이 잘못 인쇄되었다는 뜻은 아닙니다.

## 3. 결과 해석과 개선 방향

정상 이미지 6장에서는 OCR과 제품 판정이 모두 정확했지만, 영상 품질이 바뀌면 같은 OCR 엔진도 문자를 잘못 읽거나 전혀 읽지 못했습니다.

따라서 OCR이 실패할 때는 다음 순서로 개선합니다.

1. **촬영 환경 개선:** 초점·셔터속도를 조정해 흐림을 줄이고, 조명 각도를 바꿔 반사를 줄입니다.
2. **전처리·설정 재검토:** 기울기 보정, 대비 조절, 확대율 및 Tesseract 설정을 바꿔 다시 비교합니다.
3. **재평가:** 개선한 설정을 정상 이미지와 실패 이미지에 모두 적용해 정확도가 실제로 높아졌는지 확인합니다.
4. **필요한 경우:** 다른 OCR 엔진을 비교하거나, 다양한 조건의 데이터를 수집하여 학습 가능한 OCR 모델의 추가 학습을 검토합니다.

> **핵심:** 이번 테스트는 **어떤 조건에서 OCR이 실패하는지 찾아낸 실험**입니다. 먼저 촬영 조건과 OCR 설정을 개선하고, 효과가 부족할 때 추가 학습을 검토합니다.

---

**확인한 결과 파일**

- 정상 이미지 검사: `reports/day06_ocr_results.csv`
- 정상 이미지 평가: `reports/day06_ocr_metrics.md`
- 환경 변화 테스트: `reports/day06_challenge_results.csv`

이 문서를 프로젝트의 `reports/day06_failure_analysis.md`로 저장합니다.
```

이 문서는 **잘 작성하는 것이 목적이 아닙니다.**

자신이 실행한 결과가:

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