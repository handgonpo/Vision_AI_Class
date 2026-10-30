
## 오늘의 핵심 질문

1일차에서는 같은 `Apple Red 1` Train을 가지고 두 가지 방법을 경험했습니다.

```text
실험 A — Augmentation

원본 Apple
        ↓
사람이 변형 규칙 지정
Brightness / Rotation / Blur / Noise
        ↓
기존 이미지를 변형한 학습데이터 생성
```

```text
실험 B — VAE

Apple Train
        ↓
Encoder
        ↓
Latent Space
        ↓
Decoder
        ↓
Reconstruction / Random Sample
```

1일차 VAE에서는 Apple의 **붉은색·둥근 형태 같은 큰 특징은 어느 정도 나타났지만,
Random Sample의 윤곽과 세부 무늬가 흐릿하여 바로 좋은 학습데이터라고 보기 어려웠습니다.**

여기서 중요한 점이 있습니다.

> **2일차는 1일차 VAE 모델을 계속 튜닝하는 수업이 아닙니다.**

VAE의 Epoch나 Latent Dimension을 계속 늘려 같은 모델을 고치는 대신,
**같은 Apple Train을 다른 생성 방식에 넣어 결과가 어떻게 달라지는지 비교**합니다.

```text
1일차 VAE 결과
→ 생성 원리는 확인
→ Random Sample 품질에는 한계가 있었음

        ↓

2일차 질문

"다른 생성 방식은
Apple 이미지를 어떻게 만들까?"

        ↓

DCGAN
WGAN-GP
Pretrained Diffusion
```

세 방법의 역할도 서로 다릅니다.

```text
DCGAN
→ Apple Train을 직접 학습
→ Generator와 Discriminator가 경쟁하며 이미지 생성

WGAN-GP
→ 같은 Apple Train을 직접 학습
→ Critic + Gradient Penalty로
  GAN 학습을 더 안정적으로 만들어 보는 방법

Pretrained Diffusion
→ Apple Train으로 오늘 새로 학습하지 않음
→ 이미 대규모 데이터로 학습된 모델에
  Prompt + Seed를 주어 Apple 이미지 생성
```

따라서 오늘의 목적은 단순히

```text
VAE보다 무조건 더 좋은 모델 찾기
```

가 아닙니다.

오늘은 다음 질문에 직접 답해 봅니다.

```text
같은 Apple Train으로
DCGAN과 WGAN-GP를 학습하면
VAE와 어떤 차이가 나타나는가?

        +

Pretrained Diffusion은
어떤 품질의 Apple을 만드는가?

        +

보기 좋은 Synthetic Image가
실제 학습데이터로도 적합한가?
```

생성 결과가 좋아 보이더라도 바로 학습데이터로 사용하지 않습니다.

```text
생성
        ↓
결과 비교
        ↓
Domain Gap 확인
        ↓
Automatic QA
        ↓
Human QA
        ↓
학습데이터 후보인지 판단
```

> 2일차의 핵심은 “생성 품질을 무조건 올리는 것”이 아니라,
> VAE 다음 단계의 여러 생성 방식을 직접 비교하고
> 어떤 결과가 실제 학습데이터 후보가 될 수 있는지 판단하는 것입니다.

---

# 오늘의 수업 목표

수업이 끝나면 다음 내용을 설명하고 직접 실행할 수 있어야 합니다.

- 1일차 VAE Random Sample의 한계를 2일차 생성모델 비교와 연결하여 설명할 수 있습니다.
- 1일차 프로젝트와 교과 11 공통 `.venv`를 그대로 이어서 사용할 수 있습니다.
- 1일차 `Apple Red 1`의 Train·Validation·Test 역할을 다시 구분할 수 있습니다.
- 2일차 GAN 학습에는 **1일차 Apple Train만 사용**해야 하는 이유를 설명할 수 있습니다.
- Augmentation·VAE·GAN·WGAN-GP·Diffusion의 차이를 설명할 수 있습니다.
- GAN의 Generator와 Discriminator 역할을 설명할 수 있습니다.
- Random Noise에서 DCGAN이 새로운 이미지를 만드는 흐름을 설명할 수 있습니다.
- DCGAN의 Epoch별 생성 결과와 Loss를 확인할 수 있습니다.
- Mode Collapse가 어떤 현상인지 생성 결과를 보고 설명할 수 있습니다.
- WGAN-GP의 Critic과 Gradient Penalty가 어떤 역할을 하는지 큰 흐름으로 설명할 수 있습니다.
- 같은 Apple Train으로 DCGAN과 WGAN-GP를 학습하고 결과를 비교할 수 있습니다.
- 사전학습 Diffusion 모델에 Prompt·Seed를 입력하여 Apple 이미지를 생성할 수 있습니다.
- Prompt를 고정하고 Seed만 바꾸는 실험을 할 수 있습니다.
- Seed를 고정하고 Prompt만 바꾸는 실험을 할 수 있습니다.
- Real Apple과 Synthetic Apple 사이의 Domain Gap을 확인할 수 있습니다.
- 생성 이미지를 Automatic QA와 Human QA로 확인할 수 있습니다.
- 생성 조건을 Metadata로 기록하는 이유를 설명할 수 있습니다.
- Mini Challenge에서 `Banana 1`을 사용하여 2일차의 핵심 생성·비교·QA 흐름을 스스로 다시 수행할 수 있습니다.
- 오늘 만든 코드와 결과를 README와 Git에 정리할 수 있습니다.

---

# 오늘 수업의 전체 흐름

2일차는 1일차 VAE 뒤에서 **새로운 생성 방법을 비교하는 수업**입니다.

1일차에서 이미 준비한 `Apple Red 1`의 Train을 그대로 사용하므로
새로운 Apple 데이터를 다시 수집하지 않습니다.

```text
1일차에서 확보한 기준

Apple Train
+
VAE 결과
→ 큰 특징은 학습했지만 Random Sample은 흐릿했음

        ↓

2일차
"다른 생성 방식에서는 결과가 어떻게 달라질까?"
```

오늘은 세 가지 생성 방식을 확인합니다.

![[Pasted image 20261008104044.png]]

여기서 세 가지를 꼭 구분합니다.

```text
VAE / DCGAN / WGAN-GP
→ 실제 Apple Train에서 직접 학습

Diffusion
→ 오늘 Apple Train으로 새로 학습하지 않음
→ Pretrained Model을 사용

Augmentation
→ 모델이 새 이미지를 학습해서 그리는 방식이 아니라
  기존 이미지를 사람이 정한 규칙으로 변형
```

따라서 결과가 더 선명하다고 해서 곧바로
“이 모델이 Apple Train을 더 잘 학습했다”고 해석하지 않습니다.

특히 Diffusion은 사전학습 지식을 이미 가지고 있기 때문에
DCGAN·WGAN-GP와 **완전히 같은 조건의 학습 성능 비교**가 아닙니다.

오늘은 다음 기준으로 결과를 봅니다.

```text
형태가 Apple처럼 보이는가?
색이 자연스러운가?
다양성이 있는가?
같은 패턴만 반복되지 않는가?
Artifact가 있는가?
Real Apple과 Domain Gap은 어느 정도인가?
학습데이터 후보로 사용할 수 있는가?
```

> **오늘의 데이터 운영 원칙**
>
> `Apple Red 1`의 1일차 **Train만** DCGAN과 WGAN-GP의 학습에 사용합니다.
>
> Validation과 Test는 생성모델 학습에 섞지 않습니다.
>
> Diffusion은 Apple Train으로 새로 학습하는 것이 아니라 **Pretrained Diffusion Model**을 사용합니다.
>
> 생성된 이미지는 바로 학습데이터로 승인하지 않고 Real과 비교한 뒤 QA를 수행합니다.

---

# 오늘 사용할 실제 데이터

## 1. 본 실습 — 1일차 `Apple Red 1` Train

1일차에서 이미 다음 구조를 만들었습니다.

```text
day01_image_aug_vae/
└─ data/
   └─ apple/
      ├─ source_train/
      ├─ source_test/
      │
      └─ split/
         ├─ train/
         ├─ val/
         └─ test/
```

2일차의 DCGAN과 WGAN-GP가 실제로 사용하는 폴더는 다음입니다.

```text
../day01_image_aug_vae/data/apple/split/train/
```

예를 들어 1일차에서 `Apple Red 1` Training 492장을 사용했다면 약 15%를 Validation으로 나누었으므로 Train은 약 419장이 됩니다.

실제 수량은 오늘 코드로 다시 확인합니다.

```text
Apple Train
→ DCGAN 학습
→ WGAN-GP 학습

Apple Validation
→ 오늘 GAN 학습에 사용하지 않음

Apple Test
→ 오늘 GAN 학습에 사용하지 않음
```

### 왜 같은 Apple Train을 계속 사용하나요?

DCGAN과 WGAN-GP는 **같은 Apple Train을 사용해야 학습 방식의 차이를 비교하기 쉽습니다.**

1일차 VAE도 같은 Apple Train을 사용했으므로, VAE·DCGAN·WGAN-GP는 동일한 실제 데이터에서 서로 다른 생성모델의 특징을 비교할 수 있습니다.

Diffusion은 조금 다릅니다. 오늘 사용하는 Diffusion은 Apple Train으로 새로 학습하지 않고 
**사전학습 모델에 Prompt와 Seed를 입력하여 이미지를 생성**합니다. 

따라서 Apple Train은 Diffusion의 학습 입력이 아니라 **생성 결과와 비교할 Real Reference** 역할을 합니다.

```text
Apple Train
    ├─ VAE 학습
    ├─ DCGAN 학습
    └─ WGAN-GP 학습

Pretrained Diffusion
    └─ Prompt + Seed로 생성

        ↓

모든 결과를
같은 Real Apple 기준으로 비교
```

이 구분을 해 두면 “같은 데이터로 학습한 모델 비교”와 “사전학습 모델의 생성 결과 비교”를 혼동하지 않습니다.

---

## 2. Diffusion은 Apple 이미지를 어떻게 만드나요?

1일차에서는 VAE가 **Apple Train 이미지를 직접 보면서**
사과의 색과 형태 같은 특징을 학습했습니다.

```text
1일차 VAE

Apple Train
        ↓
VAE가 Apple 특징 학습
        ↓
새로운 Apple Sample 생성
```

2일차에서는 다른 생성 방법들도 비교합니다.

그중 **Diffusion은 VAE와 출발 방식이 다릅니다.**

오늘 사용하는 Diffusion은  
Apple Train 419장을 다시 학습시키는 것이 아니라,  
이미 많은 이미지로 학습되어 있는  
**Pretrained Diffusion Model**을 사용합니다.

그래서 우리가 생성형 AI의 이미지 생성 기능에

> **“흰 배경에 빨간 사과 한 개를 만들어 줘.”**

라고 요청하듯,  
원하는 이미지의 모습을 문장으로 입력합니다.

이 문장을 `Prompt`라고 합니다.

```
Pretrained Diffusion Model
        +
Prompt
        ↓
Synthetic Apple 생성
```

예를 들어 오늘은 다음과 같이 요청합니다.

```
a single red apple on a clean white background,
centered object,
realistic product photography,
neutral lighting
```

쉽게 말하면 다음 뜻입니다.

```
흰 배경
   +
빨간 사과 한 개
   +
가운데 배치
   +
실제 제품 사진 같은 모습
   +
자연스러운 조명
```

여기서 1일차의 Apple Train은  
Diffusion을 다시 학습시키는 데 사용하는 것이 아닙니다.

Diffusion이 만든 Apple이  
실제 Apple과 얼마나 비슷한지 비교할 때  
**Real Reference**로 사용합니다.

```
1일차 Apple Train
→ 실제 Apple 기준

Diffusion
→ Prompt로 Synthetic Apple 생성

        ↓

Real Apple
VS
Synthetic Apple

        ↓

나중에 품질 비교
```

> **핵심**
> 
> VAE는 우리 Apple Train을 직접 학습했고,  
> 이번 Diffusion은 이미 학습된 모델에게  
> Prompt로 원하는 Apple의 모습을 설명하여 이미지를 생성합니다.

---

# 오늘 배울 기술 스택

| 기술 | 오늘 하는 일 | 쉽게 말하면 |
|---|---|---|
| Python | 생성모델 학습·생성·검수 코드 작성 | 오늘 실험 전체를 실행 |
| pathlib | 1일차와 2일차 경로 연결 | 같은 데이터를 안전하게 재사용 |
| PyTorch | DCGAN·WGAN-GP 구현 | 생성모델 학습 |
| TorchVision | 이미지 전처리·Grid 저장 | 64×64 학습 이미지와 결과 저장 |
| Pillow | 이미지 읽기·비교 이미지 생성 | 결과 확인용 이미지 처리 |
| NumPy | Seed·배열 계산 | 재현 가능한 실험 보조 |
| Pandas | Loss·Metadata·QA CSV 저장 | 실험 결과 기록 |
| DCGAN | Noise에서 새로운 이미지 생성 | Generator와 Discriminator 경쟁 |
| WGAN-GP | GAN 학습 안정성 비교 | Critic + Gradient Penalty |
| Diffusers | Pretrained Diffusion 실행 | Prompt·Seed로 이미지 생성 |
| Prompt | 생성 조건 지정 | 어떤 이미지를 만들지 설명 |
| Seed | Random 결과 재현 | 같은 조건을 다시 만들기 |
| Domain Gap | Real과 Synthetic 차이 확인 | 실제 환경과 얼마나 다른지 판단 |
| Automatic / Human QA | 생성 이미지 검수 | 파일 검사 + 사람이 의미 확인 |
| Metadata | 생성 조건 기록 | 어떤 모델·Seed·Prompt로 만들었는지 저장 |
| Git | 2일차 코드와 기록 관리 | 실험을 다시 확인할 수 있게 저장 |

---

# 오늘 코드는 이 순서로 이해합니다

2일차에는 DCGAN·WGAN-GP·Diffusion처럼 코드가 길어집니다.

전체 코드를 한 줄씩 해석하려고 하면 핵심을 놓치기 쉽습니다.

따라서 주요 코드에서는 다음 순서로 읽습니다.

```text
① 이 코드는 왜 필요한가?
        ↓
② 입력(Input)은 무엇인가?
        ↓
③ 어떤 처리를 하는가?
        ↓
④ 무엇이 출력(Output)되는가?
        ↓
⑤ 의사코드로 전체 흐름 확인
        ↓
⑥ 실제 코드 작성
        ↓
⑦ 핵심 코드 2~3곳만 확인
        ↓
⑧ 실행 결과를 보고 의미 해석
```

> **코드를 외우는 것이 목표가 아닙니다.**
> 어떤 데이터가 들어가서 어떤 과정을 거쳐 어떤 결과가 나오는지 설명할 수 있으면 됩니다.

---

# PART 1. 1일차 프로젝트에서 2일차를 이어서 시작합니다

---

# 1. 1일차와 2일차가 어떻게 연결되는지 확인합니다

1일차에서 VAE를 학습했을 때 Loss는 줄어들었고 Apple의 큰 색·형태는 나타났습니다.

하지만 Random Sample을 실제로 보면 윤곽과 세부 무늬가 많이 뭉개져 있었습니다.

```text
1일차 VAE

Apple Train
        ↓
VAE 학습
        ↓
Loss 감소
        ↓
Apple의 큰 특징은 일부 학습
        ↓
Random Sample은 흐릿함
        ↓
"생성 원리는 이해했지만
 바로 좋은 학습데이터라고 하기는 어렵다"
```

그래서 2일차에는 **VAE를 계속 고치는 대신 다른 생성 방식을 비교**합니다.

```text
같은 Apple Train
        ├─ VAE       ← 1일차
        ├─ DCGAN     ← 2일차
        └─ WGAN-GP   ← 2일차

Pretrained Diffusion
        └─ Prompt + Seed로 생성  ← 2일차
```

여기서 주의합니다.

```text
VAE 결과를
DCGAN에 넣는다
        X

VAE 결과를
WGAN-GP에 넣는다
        X
```

==DCGAN과 WGAN-GP는 **VAE 결과를 이어받지 않습니다.**==

둘 다 1일차에서 준비한 **원본 Apple Train**에서 다시 시작합니다.

Diffusion도 다릅니다.

```text
DCGAN / WGAN-GP
→ 우리 Apple Train으로 직접 학습

Diffusion
→ 이미 학습된 Pretrained Model 사용
→ Apple Prompt와 Seed로 이미지 생성
```

따라서 오늘은 모델 간 “승자”를 정하는 수업이라기보다
**생성 방식에 따라 품질·다양성·실패 형태가 어떻게 달라지는지 직접 확인하는 수업**입니다.

오늘도 1일차에서 배운 데이터 관리 원칙은 그대로 유지합니다.

```text
Train만 학습에 사용
        ↓
생성 조건 기록
        ↓
결과를 눈으로 비교
        ↓
Domain Gap
        ↓
Automatic QA
        ↓
Human QA
        ↓
사용 후보 판단
```

---

# 2. 교과 11 프로젝트 Root로 이동합니다

```bash
cd ~/ai_vision/subject11_synthetic_data
```

1일차 폴더를 확인합니다.

```bash
ls
```

다음 폴더가 보여야 합니다.

```text
day01_image_aug_vae
```

---

# 3. 2일차 폴더를 만듭니다

```bash
mkdir -p day02_generative_models
cd day02_generative_models
```

오늘 필요한 폴더를 만듭니다.

```bash
mkdir -p \
src \
scripts \
results/apple/dcgan/epochs \
results/apple/dcgan/final \
results/apple/wgan_gp/epochs \
results/apple/wgan_gp/final \
results/apple/diffusion \
results/apple/compare \
results/banana/dcgan/epochs \
results/banana/dcgan/final \
results/banana/wgan_gp/epochs \
results/banana/wgan_gp/final \
results/banana/diffusion \
results/banana/compare \
models/apple \
models/banana \
reports/apple \
reports/banana
```

파일을 준비합니다.

```bash
touch src/day02_common.py
touch scripts/00_check_day02.py
touch scripts/01_train_dcgan.py
touch scripts/02_train_wgan_gp.py
touch scripts/03_compare_gan_results.py
touch scripts/04_diffusion_generate.py
touch scripts/05_domain_gap_review.py
touch scripts/06_synthetic_qa.py
touch requirements_day02.txt
touch day02_notes.md
touch README.md
touch .gitignore
```

### 지금 무엇을 한 것인가요?

```text
1일차
→ Real Data와 VAE 결과 보관

2일차
→ 생성모델 코드와 결과를 별도 관리

apple
→ 본 실습

banana
→ Mini Challenge
```

---

# 4. 1일차 가상환경을 그대로 사용합니다

1일차에서 만든 `.venv`는 `day01_image_aug_vae` 안이 아니라 **교과 11 Root에 있는 공통 가상환경**입니다.

현재 위치가 `day02_generative_models`이므로 한 단계 위의 `.venv`를 활성화합니다.

```text
subject11_synthetic_data/
├─ .venv/                  ← 교과 11 공통 환경
├─ day01_image_aug_vae/
└─ day02_generative_models/  ← 현재 위치
```

```bash
source ../.venv/bin/activate
```

Python 위치를 확인합니다.

```bash
which python
```

PyTorch와 CUDA도 확인합니다.

```bash
python -c "import torch; print(torch.__version__); print('CUDA:', torch.cuda.is_available())"
```

---

# 5. Diffusion 실행환경과 Local Model을 먼저 확인합니다

DCGAN과 WGAN-GP는 1일차의 PyTorch 환경을 그대로 사용합니다.

Diffusion은 모델 파일과 추가 라이브러리가 준비되어 있어야 하므로, 
본 실습보다 먼저 **환경과 Model 위치를 확인**합니다.

```text
교과 11 공통 .venv
        ↓
Diffusers · Transformers · Accelerate 확인
        ↓
DIFFUSION_MODEL_ID 확인
        ↓
Local Model이면 model_index.json 확인
        ↓
Environment Check
        ↓
Diffusion 1장 Smoke Test
        ↓
Prompt / Seed 본 실험
```

Diffusion 실습에 필요한 패키지는 별도 파일에 기록합니다.

**파일: `requirements_day02.txt`** 생성합니다.

```bash
touch ../requirements_day02.txt
```

생성후 텍스트 안에 다음과 같이 입력합니다.

```text
diffusers
transformers
accelerate
safetensors
```

필요한 패키지 준비된 설치 경로에서 설치합니다.

```bash
pip install -r ../requirements_day02.txt
```

설치 여부를 확인합니다.

```bash
python -c "import diffusers, transformers, accelerate; print('Diffusion libraries OK')"
```

2일차 설치 상태도 기록합니다.

```bash
pip freeze > reports/environment_freeze_day02.txt
```

---
## Diffusion에서 사용할 사전학습 모델을 지정합니다

오늘 Diffusion 실습에서는
**SD-Turbo라는 사전학습된 이미지 생성 모델**을 사용합니다.

이 모델은 우리가 Apple 이미지로 처음부터 학습시키는 모델이 아닙니다.

이미 학습이 끝난 모델을 불러온 뒤,
Prompt를 입력하여 새로운 Apple 이미지를 생성합니다.

```text
Pretrained SD-Turbo
        +
Prompt
        ↓
Synthetic Apple
```

오늘 사용할 모델을 다음과 같이 지정합니다.

```bash
export DIFFUSION_MODEL_ID="stabilityai/sd-turbo"
```

이 명령은

> **“오늘 Diffusion 코드에서 `stabilityai/sd-turbo` 모델을 사용하겠다.”**

라는 설정입니다.

현재 설정된 모델을 확인합니다.

```bash
echo "$DIFFUSION_MODEL_ID"
```

다음과 같이 출력되면 정상입니다.

```
stabilityai/sd-turbo
```

---

# 6. 2일차에서 사용할 공통 경로 코드를 작성합니다

1일차에서는 `day01_common.py`를 만들어
Apple과 Banana의 데이터·결과 경로를 공통으로 관리했습니다.

2일차도 같은 방식을 사용합니다.

다만 이번에는 2일차 코드가

- 1일차에서 만들어 둔 `split/train`을 찾아가고
- 2일차에서 생성되는 결과·모델·리포트의 저장 위치를 사용해야 합니다.

그래서 2일차용 공통 파일을 하나 만듭니다.

```text
1일차 split/train
        ↓
src/day02_common.py
        ↓
2일차 여러 실습 코드
        ↓
results / models / reports
```

`day02_common.py`는 모델을 학습하는 코드가 아닙니다.

> **1일차의 Train 위치와 2일차의 저장 위치를  
> 여러 Python 코드에서 공통으로 사용할 수 있게 연결해 주는 파일입니다.**

이제 코드를 작성합니다.

**파일: `src/day02_common.py`**

```python
from __future__ import annotations

from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]
SUBJECT_ROOT = ROOT.parent
DAY01_ROOT = SUBJECT_ROOT / "day01_image_aug_vae"

VALID_EXTENSIONS = {
    ".jpg",
    ".jpeg",
    ".png",
    ".bmp",
}


def list_images(directory: Path) -> list[Path]:
    if not directory.exists():
        return []

    return sorted(
        path
        for path in directory.iterdir()
        if path.is_file()
        and path.suffix.lower() in VALID_EXTENSIONS
    )


def day01_train_dir(dataset: str) -> Path:
    if dataset not in {"apple", "banana"}:
        raise ValueError(
            f"지원하지 않는 dataset: {dataset}"
        )

    return (
        DAY01_ROOT
        / "data"
        / dataset
        / "split"
        / "train"
    )


def day02_paths(dataset: str) -> dict[str, Path]:
    result_root = ROOT / "results" / dataset
    model_root = ROOT / "models" / dataset
    report_root = ROOT / "reports" / dataset

    paths = {
        "train": day01_train_dir(dataset),
        "dcgan_epochs": result_root / "dcgan" / "epochs",
        "dcgan_final": result_root / "dcgan" / "final",
        "wgan_epochs": result_root / "wgan_gp" / "epochs",
        "wgan_final": result_root / "wgan_gp" / "final",
        "diffusion": result_root / "diffusion",
        "compare": result_root / "compare",
        "model": model_root,
        "report": report_root,
    }

    for key, path in paths.items():
        if key == "train":
            continue

        path.mkdir(parents=True, exist_ok=True)

    return paths
```

공통 경로 파일이 정상적으로 연결되는지 짧게 확인합니다.

```bash
python -c "from src.day02_common import day02_paths; print(day02_paths('apple')['train'])"
```

다음과 같이 1일차 Apple Train 경로가 출력되면 정상입니다.

```text
.../day01_image_aug_vae/data/apple/split/train
```

## 작성한 코드의 흐름을 의사코드로 확인합니다

이 코드는 **1일차 Train 데이터를 찾아오고, 2일차 결과를 저장할 경로를 준비하는 공통 파일**입니다.

```text
2일차 프로젝트 위치 확인
        ↓
교과 11 전체 Root 확인
        ↓
1일차 프로젝트 위치 연결
        ↓
apple 또는 banana 선택
        ↓
1일차 split/train 경로 가져오기
        ↓
2일차에서 사용할
results / models / reports 경로 준비
        ↓
필요한 경로를 다른 실습 코드에 전달
```

쉽게 설명하면 다음 흐름입니다ㅏ

```
1일차 Train 찾기
        +
2일차 저장 위치 준비
        ↓
DCGAN / WGAN-GP / Diffusion 등의
실습 코드에서 공통으로 사용
```

## 코드에서 꼭 확인할 핵심

전체 코드를 외울 필요는 없습니다.  
다음 세 부분만 이해하면 됩니다.

### ① 1일차와 2일차의 위치를 연결합니다

```python
ROOT = Path(__file__).resolve().parents[1]
SUBJECT_ROOT = ROOT.parent
DAY01_ROOT = SUBJECT_ROOT / "day01_image_aug_vae"
```

현재 2일차 프로젝트에서 출발하여  
**교과 11 Root를 찾고 → 다시 1일차 프로젝트 위치를 찾아가는 부분**입니다.

```
subject11_synthetic_data/
│
├─ day01_image_aug_vae/     ← 가져올 Train 데이터
│
└─ day02_generative_models/ ← 현재 실습
```

---

### ② `day01_train_dir()` — 1일차 Train을 찾아옵니다

```python
def day01_train_dir(dataset: str) -> Path:
```

`apple` 또는 `banana`를 받아서 1일차에서 만들어 둔

```
data/apple/split/train/
```

또는

```
data/banana/split/train/
```

경로를 찾아줍니다.

즉,

```
2일차에서 데이터를 새로 만드는 것이 아니라
        ↓
1일차에서 준비한 Train을 다시 사용
```

하는 부분입니다.

---

### ③ `day02_paths()` — 2일차 결과 저장 위치를 준비합니다

```python
def day02_paths(dataset: str) -> dict[str, Path]:
```

2일차에서 앞으로 생성되는 결과를 어디에 저장할지 한 번에 정리합니다.

```
DCGAN 결과
WGAN-GP 결과
Diffusion 결과
비교 결과
Model
Report
```

특히 다음 부분을 확인합니다.

```python
if key == "train":
    continue
```

`train`은 **1일차에 이미 만들어진 데이터를 가져오는 경로**이므로  
2일차 코드가 새로 만들지 않습니다.

반면 나머지 결과 폴더는

```python
path.mkdir(parents=True, exist_ok=True)
```

로 필요한 경우 자동으로 준비합니다.

> **핵심:**  
> `day02_common.py`는 1일차 Train은 그대로 가져오고,  
> 2일차에서 새로 만들어질 결과의 저장 위치를 공통으로 관리합니다.

---

# 7. 2일차 실습을 시작하기 전에 준비 상태를 확인합니다

1일차에서는 실습을 시작하기 전에

- Python과 주요 라이브러리가 정상적으로 설치되어 있는지
- CUDA와 GPU를 사용할 수 있는지
- 사용할 이미지가 정상적으로 준비되어 있는지

확인했습니다.

2일차에서도 긴 모델 학습을 시작하기 전에
같은 방식으로 필요한 준비 상태를 먼저 확인합니다.

이번에는 1일차와 달리
**Diffusion 모델을 추가로 사용하므로 모델 설정도 함께 확인합니다.**

즉 `00_check_day02.py`는 새로운 모델을 학습하는 코드가 아니라,

> **“2일차 실습을 시작해도 되는 상태인가?”**

를 한 번에 확인하는 사전 점검 코드입니다.

주로 다음 내용을 확인합니다.

```text
Python과 필요한 라이브러리가 정상인가?

CUDA / GPU를 사용할 수 있는가?

오늘 사용할 Diffusion Model이 지정되어 있는가?

1일차에서 만든 Apple 또는 Banana Train을
2일차 코드가 정상적으로 찾을 수 있는가?

이미지를 실제로 읽을 수 있는가?
```

**파일: `scripts/00_check_day02.py`**

```python
from __future__ import annotations

import argparse
import os
import sys
from pathlib import Path

import accelerate
import cv2
import diffusers
import safetensors
import torch
import torchvision
import transformers


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
    list_images,
)


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def main() -> None:
    args = parse_args()
    paths = day02_paths(args.dataset)

    print("=" * 60)
    print("Subject 11 - Day 02 Check")
    print("=" * 60)

    print("PyTorch      :", torch.__version__)
    print("TorchVision  :", torchvision.__version__)
    print("Diffusers    :", diffusers.__version__)
    print("Transformers :", transformers.__version__)
    print("Accelerate   :", accelerate.__version__)
    print("Safetensors  :", safetensors.__version__)

    cuda_available = torch.cuda.is_available()

    print("CUDA Available :", cuda_available)

    if cuda_available:
        print(
            "GPU            :",
            torch.cuda.get_device_name(0),
        )
    else:
        print("GPU            : CPU mode")

    print("-" * 60)

    model_id = os.environ.get(
        "DIFFUSION_MODEL_ID",
        "",
    )

    if not model_id:
        raise RuntimeError(
            "DIFFUSION_MODEL_ID가 설정되지 않았습니다. "
            "제공된 Local Model 경로 또는 사용할 Model ID를 "
            "먼저 지정하세요."
        )

    print(
        "Diffusion Model:",
        model_id,
    )

    local_model = Path(
        model_id
    ).expanduser()

    looks_like_local_path = (
        model_id.startswith("/")
        or model_id.startswith(".")
        or model_id.startswith("~")
    )

    if local_model.exists():
        if not local_model.is_dir():
            raise RuntimeError(
                "DIFFUSION_MODEL_ID가 Local 경로이지만 "
                f"폴더가 아닙니다: {local_model}"
            )

        model_index = (
            local_model
            / "model_index.json"
        )

        if not model_index.exists():
            raise RuntimeError(
                "Local Diffusion Model에 "
                f"model_index.json이 없습니다: {model_index}"
            )

        print(
            "Diffusion Source:",
            "LOCAL MODEL",
        )

        print(
            "model_index.json:",
            model_index,
        )

    elif looks_like_local_path:
        raise RuntimeError(
            "지정한 Local Diffusion Model 경로가 없습니다: "
            f"{local_model}"
        )

    else:
        print(
            "Diffusion Source:",
            "MODEL ID / CACHE",
        )

        print(
            "[INFO] 인터넷 제한 환경에서는 "
            "이 Model이 Local Cache 또는 기관 내부 환경에 "
            "미리 준비되어 있어야 합니다."
        )

    print("-" * 60)

    train_dir = paths["train"]

    if not train_dir.exists():
        raise RuntimeError(
            f"1일차 Train 폴더가 없습니다: {train_dir}"
        )

    image_files = list_images(train_dir)

    if not image_files:
        raise RuntimeError(
            f"1일차 Train 이미지가 없습니다: {train_dir}"
        )

    unreadable = []

    for path in image_files:
        image = cv2.imread(str(path))

        if image is None:
            unreadable.append(path.name)

    if unreadable:
        raise RuntimeError(
            "읽을 수 없는 이미지가 있습니다: "
            + ", ".join(unreadable[:5])
        )

    first = cv2.imread(
        str(image_files[0])
    )

    height, width = first.shape[:2]

    print("Dataset      :", args.dataset)
    print("Train Dir    :", train_dir)
    print("Train Images :", len(image_files))
    print("First Size   :", f"{width} x {height}")

    if len(image_files) < 100:
        print(
            "[WARNING] 이미지 수가 적어 "
            "GAN 생성 품질이 불안정할 수 있습니다."
        )

    print("=" * 60)
    print("Day 02 preflight: READY")


if __name__ == "__main__":
    main()
```

### 작성한 코드의 흐름을 빠르게 이해합니다

```text
2일차 실행환경 확인
        ↓
PyTorch · Diffusion 관련 라이브러리 확인
        ↓
CUDA / GPU 사용 가능 여부 확인
        ↓
오늘 사용할 Diffusion Model 설정 확인
        ↓
1일차 Apple 또는 Banana Train 경로 가져오기
        ↓
Train 이미지가 실제로 존재하는지 확인
        ↓
이미지를 정상적으로 읽을 수 있는지 확인
        ↓
문제가 없으면

Day 02 preflight: READY
```

쉽게 말하면 다음 순서입니다.

```
컴퓨터 환경 정상?
        ↓
Diffusion 모델 준비됨?
        ↓
1일차 Train 연결됨?
        ↓
이미지 정상?
        ↓
본 실습 시작
```

## 코드에서 꼭 확인할 핵심

전체 코드를 외울 필요는 없습니다.  
다음 세 부분만 이해하면 됩니다.

### ① GPU를 사용할 수 있는지 확인합니다

```python
cuda_available = torch.cuda.is_available()
```

PyTorch가 NVIDIA GPU를 사용할 수 있는지 확인합니다.

```
True
→ GPU 사용 가능

False
→ CPU로 실행
```

---

### ② 사용할 Diffusion Model이 지정되어 있는지 확인합니다

```python
model_id = os.environ.get(
    "DIFFUSION_MODEL_ID",
    "",
)
```

앞에서 설정한

```bash
export DIFFUSION_MODEL_ID="stabilityai/sd-turbo"
```

값을 Python 코드에서 가져오는 부분입니다.

즉,

```
DIFFUSION_MODEL_ID
        ↓
오늘 사용할 Diffusion Model 확인
```

입니다.

---

### ③ 1일차 Train 데이터가 정상적으로 연결되는지 확인합니다

```python
train_dir = paths["train"]
image_files = list_images(train_dir)
```

2일차에서 데이터를 새로 만드는 것이 아니라  
**1일차에서 만들어 둔 `split/train`을 찾아오는 부분**입니다.

그다음 이미지를 실제로 읽어 보면서 문제가 없는지도 확인합니다.

```python
image = cv2.imread(str(path))
```

이미지를 하나라도 읽지 못하면 본 학습으로 넘어가지 않습니다.

---

## 핵심만 기억합니다

```
00_check_day02.py

환경 확인
+
Diffusion Model 확인
+
1일차 Train 연결 확인
        ↓
모두 정상이면
Day 02 preflight: READY
```

> `Day 02 preflight: READY`가 출력되면  
> **2일차의 모델 학습과 이미지 생성 실습을 시작할 준비가 된 것입니다.**

---

본 실습에서는 Apple을 확인합니다.

```bash
python scripts/00_check_day02.py --dataset apple
```

## 실행 결과를 확인합니다

실행 결과에서 다음 항목이 모두 정상적으로 확인되었습니다.

```text
PyTorch / TorchVision / Diffusers 등
→ 필요한 라이브러리 정상

CUDA Available : True
GPU : NVIDIA GeForce RTX 4060
→ GPU 사용 가능

Diffusion Model : stabilityai/sd-turbo
→ 오늘 사용할 Diffusion Model 정상 설정

Train Images : 419
First Size : 100 x 100
→ 1일차 Apple Train 데이터 정상 연결

Day 02 preflight: READY
→ 2일차 실습 준비 완료
```

---

# PART 2. DCGAN으로 Apple의 새로운 이미지를 생성합니다

---

# 8. GAN의 Generator와 Discriminator를 이해합니다

1일차에서는 VAE를 이용하여 Apple 이미지를 생성했습니다.

이번에는 같은 `Apple Train`을 이용하지만,
이미지를 만드는 방식이 전혀 다른 **DCGAN**을 학습합니다.

세 그림을 순서대로 보면서
**DCGAN이 어떤 구조로 학습하고, 어떻게 새로운 Apple 이미지를 만들어 내는지**
이해해 봅니다.

![[Pasted image 20261008123010.png]]

## 첫 번째 그림 — VAE와 DCGAN은 이미지를 만드는 방식이 다릅니다

1일차의 VAE부터 다시 떠올려 봅니다.

VAE는 실제 Apple 이미지를 입력받아

```text
Real Apple
    ↓
Encoder
    ↓
Latent Space
    ↓
Decoder
    ↓
Reconstruction
```

처럼 이미지의 특징을 압축하고 다시 복원하는 방법을 학습했습니다.

DCGAN은 방식이 다릅니다.

DCGAN에는 VAE처럼 이미지를 압축하는 `Encoder`가 없습니다.

대신 두 개의 모델이 함께 학습합니다.

```
Generator
→ 새로운 이미지를 만드는 모델

Discriminator
→ Real과 Fake를 구분하는 모델
```

Generator는 `Random Noise`, 즉 랜덤 숫자에서 출발하여  
Fake Apple 이미지를 만듭니다.

Discriminator는

```
실제 Apple Train 이미지
        +
Generator가 만든 Fake Apple
```

을 보면서 어느 쪽이 Real이고 어느 쪽이 Fake인지 구분합니다.

> **VAE는 복원하는 방법을 학습하고,  
> DCGAN은 Generator와 Discriminator의 경쟁을 통해  
> 새로운 이미지를 만드는 방법을 학습합니다.**

---

## 두 번째 그림 — DCGAN은 Random Numbers에서 Apple 이미지를 만들기 시작합니다


![[Pasted image 20261008124246.png]]

여기서 가장 먼저 주의할 점이 있습니다.

```
Random Noise
=
원본 Apple에 Noise를 넣은 이미지
```

가 아닙니다.

Generator에 들어가는 것은 다음과 같은 **그냥 랜덤한 숫자들**입니다.

```
[0.31, -0.72, 0.15, 0.88, ...]
```

이 숫자 안에 처음부터

```
빨간색
둥근 모양
꼭지
```

같은 Apple의 특징이 들어 있는 것도 아닙니다.

학습을 시작할 때 Generator는 사과를 만드는 방법을 모릅니다.

그래서 처음에는

```
Random Numbers
        ↓
Generator
        ↓
어색하거나 의미 없는 Fake Image
```

가 만들어질 수 있습니다.

그리고 Discriminator에는 두 종류의 이미지가 들어갑니다.

```
1일차에서 준비한 실제 Apple Train
        ↓
Real

Generator가 만든 이미지
        ↓
Fake
```

Discriminator는 이 두 종류를 보면서  
**Real Apple과 Fake Apple을 구분하는 방법**을 학습합니다.

---
## 세 번째 그림 — 419장의 Real Apple을 이용한 경쟁 학습이 반복됩니다

오늘 DCGAN은 처음부터 Apple을 알고 있는 모델이 아닙니다.

![[Pasted image 20261008145019.png]]

1일차에서 준비한 **Apple Train 419장**을 이용하여  
Generator와 Discriminator를 처음부터 함께 학습합니다.

중요한 점은 Generator가 419장의 Apple 이미지를 직접 받아  
그 이미지를 복사하는 것이 아니라는 것입니다.

학습은 다음처럼 진행됩니다.

```
Apple Train의 Real Apple
        ↓
Discriminator
        ↑
Generator가 만든 Fake Apple
        ↑
Generator
        ↑
Random Numbers
```

처음에는 Generator가 만든 이미지가 어색하기 때문에  
Discriminator가 쉽게 Fake라고 판단할 수 있습니다.

```
처음

Random Numbers
        ↓
Generator
        ↓
어색한 이미지
        ↓
Discriminator
        ↓
"Fake"
```

이 판단 결과를 이용하여  
Generator의 내부 가중치가 조금씩 수정됩니다.

다시 새로운 Random Numbers를 입력하고,  
Generator가 새로운 Fake Apple을 만들고,  
Discriminator가 다시 Real / Fake를 구분합니다.

이 과정을 계속 반복합니다.

```
처음
→ 사과처럼 보이지 않음

        ↓ 학습 반복

중간
→ 빨간색과 둥근 형태가 조금 나타남

        ↓ 학습 반복

나중
→ Real Apple과 더 비슷한 형태의 이미지 생성
```

여기서 Discriminator는

```
Real Apple A
VS
Fake Apple A
```

처럼 특정 이미지 두 장이 얼마나 똑같은지  
1:1로 비교하는 모델이 아닙니다.

여러 Real Apple을 보면서

```
색
형태
윤곽
질감
배경
```

같은 **실제 Apple 데이터의 전체적인 패턴을 구분하는 방법**을 학습합니다.

Generator는 이 Discriminator가 Fake라고 판단하기 어렵도록  
자신의 이미지 생성 방법을 계속 개선합니다.

---

## 세 그림을 한 번에 연결하면

![[Pasted image 20261008151102.png]]

```
Apple Train 419장
→ 실제 Apple의 기준을 제공

Random Numbers
→ Generator의 시작값

        ↓

Generator
→ Fake Apple 생성

        ↓

Discriminator
→ Real Apple / Fake Apple 구분

        ↓

판단 결과를 이용하여
Generator와 Discriminator의 가중치 수정

        ↓

다시 생성하고 다시 판별

        ↓
        ↓
      반복

        ↓

Generator가 점점
Apple과 비슷한 이미지를 만드는 방법을 학습
```

> **핵심**
> 
> Random Numbers 자체에 Apple 특징이 들어 있는 것이 아닙니다.
> 
> DCGAN은 419장의 Real Apple을 이용한 경쟁 학습을 반복하면서  
> **Generator가 랜덤 숫자를 Apple과 비슷한 이미지로 바꾸는 방법을 배우는 모델**입니다.

학습이 끝난 뒤 새로운 이미지를 만들 때는  
학습된 Generator만 사용할 수 있습니다.

```
학습 완료

새로운 Random Numbers
        ↓
학습된 Generator
        ↓
새로운 Synthetic Apple
```

이때는 Generator가 이미 Apple 이미지를 만드는 방법을 학습했기 때문에  
새로운 Random Numbers를 넣으면 새로운 형태의 Apple 이미지를 생성할 수 있습니다.


---

# 9. DCGAN의 이미지 크기 흐름을 확인합니다

![[Pasted image 20261008151526.png]]

```text
Generator

Noise
100 × 1 × 1
        ↓
4 × 4
        ↓
8 × 8
        ↓
16 × 16
        ↓
32 × 32
        ↓
64 × 64 RGB
```

원본은 100×100이지만 오늘 GAN은 64×64로 학습합니다.

---

# 10. DCGAN 학습 코드를 작성합니다

이제 앞에서 배운 DCGAN의 원리를
실제 코드로 학습시켜 봅니다.

이번 코드는 **1일차에서 준비한 Apple Train을 이용하여
DCGAN의 Generator와 Discriminator를 실제로 학습하는 코드**입니다.

학습을 시작하면 두 모델이 함께 동작합니다.

```text
Apple Train의 Real Apple
        ↓
Discriminator가 Real인지 확인

Random Noise
        ↓
Generator가 Fake Apple 생성
        ↓
Discriminator가 Fake인지 확인
```

이 과정을 여러 번 반복하면서

- Discriminator는 Real Apple과 Fake Apple을 더 잘 구분하도록 학습하고
- Generator는 Discriminator가 Fake라고 판단하기 어려운 Apple 이미지를 만들도록 학습합니다.

즉 이번 코드에서는

> **Random Noise에서 시작한 Generator가  
> Apple Train을 기준으로 점점 더 Apple과 비슷한 이미지를 만드는 과정을 실제로 학습시킵니다.**

학습이 진행되는 동안  
Epoch별로 생성 이미지가 어떻게 달라지는지도 저장합니다.

학습이 끝나면 다음 결과가 남습니다.

```
학습된 Generator
학습된 Discriminator
Epoch별 생성 이미지
최종 생성 이미지
Loss 기록
학습 조건 기록
```

이제 실제 DCGAN 학습 코드를 작성합니다.

**파일: `scripts/01_train_dcgan.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn

from PIL import Image
from torch.utils.data import DataLoader, Dataset
from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths, list_images


IMAGE_SIZE = 64
CHANNELS = 3
NOISE_DIM = 100
BATCH_SIZE = 32
SEED = 42
VISUAL_SEED = 2026


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--epochs",
        type=int,
        default=15,
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(self, root_dir: Path):
        self.files = list_images(root_dir)

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (IMAGE_SIZE, IMAGE_SIZE)
                ),
                transforms.ToTensor(),
                transforms.Normalize(
                    (0.5, 0.5, 0.5),
                    (0.5, 0.5, 0.5),
                ),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        image = Image.open(
            self.files[index]
        ).convert("RGB")

        return self.transform(image)


class Generator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.ConvTranspose2d(
                NOISE_DIM, 512, 4, 1, 0, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                512, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                256, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                128, 64, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(64),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                64, CHANNELS, 4, 2, 1, bias=False
            ),
            nn.Tanh(),
        )

    def forward(self, noise):
        return self.model(noise)


class Discriminator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.Conv2d(
                CHANNELS, 64, 4, 2, 1, bias=False
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                64, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                128, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                256, 512, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                512, 1, 4, 1, 0, bias=False
            ),
            nn.Sigmoid(),
        )

    def forward(self, image):
        return self.model(image).view(-1)


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def make_noise(
    count: int,
    device,
    seed: int,
):
    noise_generator = torch.Generator(
        device=device
    ).manual_seed(seed)

    return torch.randn(
        count,
        NOISE_DIM,
        1,
        1,
        generator=noise_generator,
        device=device,
    )


def save_final_samples(
    generator,
    output_dir,
    device,
) -> None:
    generator.eval()

    final_noise = make_noise(
        16,
        device,
        VISUAL_SEED + 1,
    )

    with torch.no_grad():
        samples = generator(final_noise)
        samples = (samples + 1) / 2

    for index, image in enumerate(
        samples,
        start=1,
    ):
        save_image(
            image,
            output_dir / f"sample_{index:02d}.png",
        )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    set_seed(SEED)
    paths = day02_paths(args.dataset)

    reset_directory(
        paths["dcgan_epochs"]
    )

    reset_directory(
        paths["dcgan_final"]
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Dataset:", args.dataset)

    dataset = FlatImageDataset(
        paths["train"]
    )

    loader_generator = torch.Generator()
    loader_generator.manual_seed(SEED)

    loader = DataLoader(
        dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=loader_generator,
        drop_last=False,
    )

    generator = Generator().to(device)
    discriminator = Discriminator().to(
        device
    )

    criterion = nn.BCELoss()

    optimizer_g = torch.optim.Adam(
        generator.parameters(),
        lr=0.0002,
        betas=(0.5, 0.999),
    )

    optimizer_d = torch.optim.Adam(
        discriminator.parameters(),
        lr=0.0002,
        betas=(0.5, 0.999),
    )

    fixed_noise = make_noise(
        64,
        device,
        VISUAL_SEED,
    )

    history = []

    for epoch in range(
        1,
        args.epochs + 1,
    ):
        generator.train()
        discriminator.train()

        epoch_d = 0.0
        epoch_g = 0.0
        image_count = 0

        for real_images in loader:
            real_images = real_images.to(
                device
            )

            batch_size = real_images.size(
                0
            )

            real_labels = torch.ones(
                batch_size,
                device=device,
            )

            fake_labels = torch.zeros(
                batch_size,
                device=device,
            )

            optimizer_d.zero_grad()

            real_output = discriminator(
                real_images
            )

            loss_real = criterion(
                real_output,
                real_labels,
            )

            noise = torch.randn(
                batch_size,
                NOISE_DIM,
                1,
                1,
                device=device,
            )

            fake_images = generator(
                noise
            )

            fake_output = discriminator(
                fake_images.detach()
            )

            loss_fake = criterion(
                fake_output,
                fake_labels,
            )

            loss_d = (
                loss_real
                + loss_fake
            )

            loss_d.backward()
            optimizer_d.step()

            optimizer_g.zero_grad()

            fake_output = discriminator(
                fake_images
            )

            loss_g = criterion(
                fake_output,
                real_labels,
            )

            loss_g.backward()
            optimizer_g.step()

            epoch_d += (
                loss_d.item()
                * batch_size
            )

            epoch_g += (
                loss_g.item()
                * batch_size
            )

            image_count += batch_size

        avg_d = epoch_d / image_count
        avg_g = epoch_g / image_count

        history.append(
            {
                "epoch": epoch,
                "d_loss": avg_d,
                "g_loss": avg_g,
            }
        )

        print(
            f"Epoch {epoch:02d}/"
            f"{args.epochs} "
            f"D={avg_d:.4f} "
            f"G={avg_g:.4f}"
        )

        generator.eval()

        with torch.no_grad():
            samples = generator(
                fixed_noise
            )

            samples = (
                samples + 1
            ) / 2

            save_image(
                samples,
                paths["dcgan_epochs"]
                / f"epoch_{epoch:03d}.png",
                nrow=8,
            )

    model_g = (
        paths["model"]
        / "dcgan_generator.pt"
    )

    model_d = (
        paths["model"]
        / "dcgan_discriminator.pt"
    )

    torch.save(
        generator.state_dict(),
        model_g,
    )

    torch.save(
        discriminator.state_dict(),
        model_d,
    )

    loss_path = (
        paths["report"]
        / "dcgan_loss.csv"
    )

    pd.DataFrame(
        history
    ).to_csv(
        loss_path,
        index=False,
        encoding="utf-8-sig",
    )

    save_final_samples(
        generator,
        paths["dcgan_final"],
        device,
    )

    metadata_path = (
        paths["report"]
        / "dcgan_metadata.csv"
    )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "method",
                "dataset",
                "epochs",
                "batch_size",
                "noise_dim",
                "seed",
                "epoch_visual_seed",
                "final_sample_seed",
                "learning_rate",
                "model_generator",
                "model_discriminator",
            ],
        )

        writer.writeheader()

        writer.writerow(
            {
                "method": "dcgan",
                "dataset": args.dataset,
                "epochs": args.epochs,
                "batch_size": BATCH_SIZE,
                "noise_dim": NOISE_DIM,
                "seed": SEED,
                "epoch_visual_seed": VISUAL_SEED,
                "final_sample_seed": VISUAL_SEED + 1,
                "learning_rate": 0.0002,
                "model_generator": str(
                    model_g
                ),
                "model_discriminator": str(
                    model_d
                ),
            }
        )

    print("DCGAN Training Completed")
    print("Loss    :", loss_path)
    print(
        "Samples :",
        paths["dcgan_final"],
    )


if __name__ == "__main__":
    main()
```

이 코드는 **DCGAN을 실제로 학습시키는 전체 코드**이므로, 코드 뒤에서는 모든 줄을 해석하기보다 **① 데이터 준비 → ② Generator → ③ Discriminator → ④ 두 모델의 반복 학습 → ⑤ 결과 저장** 흐름만 이해하면 됩니다.

## 의사코드

```text
1일차에서 준비한 Apple Train을 불러온다
        ↓
Apple 이미지를 64×64 RGB 크기로 맞춘다
        ↓
두 모델을 준비한다

Generator
→ 가짜 Apple을 만드는 모델

Discriminator
→ 진짜 Apple과 가짜 Apple을 구분하는 모델

        ↓

[DCGAN 학습 시작]

실제 Apple을 Discriminator에게 보여준다
→ "이것은 Real이다"라고 학습

        ↓

랜덤 숫자를 Generator에 넣는다
        ↓
Generator가 Fake Apple을 만든다
        ↓
Fake Apple을 Discriminator에게 보여준다
→ "이것은 Fake이다"라고 학습

        ↓

이번에는 Generator를 학습한다
→ 자신이 만든 Fake Apple을
  Discriminator가 Real이라고 판단하도록
  Generator의 가중치를 수정

        ↓

이 과정을 Apple Train 전체에 반복한다
        ↓
한 Epoch 완료

        ↓

설정한 Epoch 수만큼 다시 반복한다
        ↓

매 Epoch마다 같은 Random Noise를 넣어
Generator가 만든 Apple의 변화를 저장한다

        ↓

학습 완료
        ↓

학습된 Generator / Discriminator 저장
최종 생성 Apple 저장
Loss 기록 저장
학습 조건 Metadata 저장
```

핵심만 빠르게 이해하면

```
Real Apple로 Discriminator 학습
        ↓
Noise → Generator → Fake Apple
        ↓
Fake Apple로 Discriminator 학습
        ↓
Generator가 더 Real처럼 만들도록 학습
        ↓
반복
```

## 코드에서 꼭 이해할 핵심

### ① 먼저 학습 조건을 정합니다

```python
IMAGE_SIZE = 64
CHANNELS = 3
NOISE_DIM = 100
BATCH_SIZE = 32
```

오늘 DCGAN은 Apple 이미지를 `64×64 RGB`로 학습하고,  
Generator는 `100차원의 Random Noise`에서 이미지 생성을 시작합니다. 붙여넣은 마크다운

이미지는 학습 전에 `64×64`로 바꾸고 정규화합니다.

```python
transforms.Resize((IMAGE_SIZE, IMAGE_SIZE))
transforms.ToTensor()
transforms.Normalize(
    (0.5, 0.5, 0.5),
    (0.5, 0.5, 0.5),
)
```

Generator의 마지막이 `Tanh()`이므로 Real Image와 Fake Image가 비슷한 값 범위를 사용하도록 맞춰 주는 부분입니다. 붙여넣은 마크다운

---

### ② Generator — Random Noise를 Apple 이미지로 바꿉니다

```python
class Generator(nn.Module):
```

Generator는 `100 × 1 × 1` Noise에서 시작하여  
`ConvTranspose2d`를 반복하면서 점점 이미지 크기를 키웁니다.

```
100 × 1 × 1
    ↓
4 × 4
    ↓
8 × 8
    ↓
16 × 16
    ↓
32 × 32
    ↓
64 × 64 RGB
```

즉 **랜덤 숫자를 실제 이미지 형태로 바꾸는 부분**입니다. 붙여넣은 마크다운

---

### ③ Discriminator — Real과 Fake를 구분합니다

```python
class Discriminator(nn.Module):
```

Generator와 반대로 이미지를 점점 작은 특징으로 압축하면서  
마지막에는

```
Real인가?
Fake인가?
```

를 판단합니다.

마지막 `Sigmoid()`는 판단 결과를 `0~1` 범위로 만드는 역할을 합니다. 붙여넣은 마크다운

---

### ④ 가장 중요한 부분은 학습 Loop입니다

먼저 Discriminator를 학습합니다.

```python
real_output = discriminator(real_images)

fake_images = generator(noise)

fake_output = discriminator(
    fake_images.detach()
)
```

```
Real Apple
→ Real이라고 판단하도록 학습

Fake Apple
→ Fake라고 판단하도록 학습
```

`fake_images.detach()`가 중요한 이유는  
**지금은 Discriminator만 학습하고 Generator는 건드리지 않기 위해서**입니다. 붙여넣은 마크다운

그다음 Generator를 학습합니다.

```python
fake_output = discriminator(
    fake_images
)

loss_g = criterion(
    fake_output,
    real_labels,
)
```

이번에는 Fake Apple을 Discriminator에게 다시 보여 주면서

```
Generator의 목표

"내가 만든 Fake Apple을
Discriminator가 Real이라고 판단하게 만들자."
```

라는 방향으로 Generator의 가중치를 수정합니다. 붙여넣은 마크다운

이 두 과정이 DCGAN 학습의 핵심입니다.

```
Discriminator 개선
        ↕
Generator 개선
        ↕
계속 반복
```

---

## `fixed_noise`는 왜 사용하나요?

```python
fixed_noise = make_noise(
    64,
    device,
    VISUAL_SEED,
)
```

Epoch마다 새로운 Noise를 사용하면  
**입력이 달라져서 Generator가 실제로 얼마나 발전했는지 비교하기 어렵습니다.**

그래서 같은 `fixed_noise`를 계속 사용하여

```
Epoch 1
        ↓
Epoch 5
        ↓
Epoch 10
        ↓
Epoch 15
```

에서 **같은 시작값으로 만든 Apple이 어떻게 변하는지** 확인합니다. 붙여넣은 마크다운 붙여넣은 마크다운

---

## 문제가 생기면 어디를 확인하나요?

|문제가 보일 때|먼저 확인할 코드|
|---|---|
|Apple 이미지를 못 찾음|`FlatImageDataset`, `paths["train"]`|
|이미지 크기가 맞지 않음|`IMAGE_SIZE`, `transforms.Resize()`|
|Generator 출력 크기가 이상함|`Generator`의 `ConvTranspose2d` 구조|
|Real / Fake 판단이 이상함|`Discriminator`, `BCELoss()`|
|학습이 거의 진행되지 않음|`epochs`, `learning_rate`, `BATCH_SIZE`|
|생성 이미지가 계속 이상함|Generator/Discriminator Loss와 Epoch별 이미지 함께 확인|
|비슷한 Apple만 계속 생성됨|Mode Collapse 가능성 확인|
|Epoch별 변화가 비교되지 않음|`fixed_noise`, `VISUAL_SEED` 확인|
|결과 파일이 안 생김|`save_image()`, `torch.save()`, 결과 경로 확인|

> 핵심은 코드
> 
> 문제가 생겼을 때  
> **데이터 문제인지 → Generator 문제인지 → Discriminator 문제인지 → 학습 Loop 문제인지 → 저장 문제인지**  
> 구분해서 해당 부분을 찾아갈 수 있으면 됩니다.

---

# 11. DCGAN을 실행합니다

먼저 Smoke Test를 합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset apple \
  --epochs 1
```

실행 결과:

```
Device : cuda
Dataset: apple
Epoch 01/1 D=0.2297 G=5.1763
DCGAN Training Completed
```

`DCGAN Training Completed`가 출력되었으므로  
GPU에서 DCGAN 학습 코드가 정상적으로 동작하는 것을 확인했습니다.

이제 본 학습을 `15 Epoch`로 실행합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset apple \
  --epochs 15
```

> Smoke Test의 1 Epoch 결과에서 이어서 학습하는 것이 아닙니다.
> 본 실습을 실행하면 **새로운 DCGAN 학습이 처음부터 다시 시작됩니다.**

---

# 12. DCGAN 결과를 관찰합니다

15 Epoch 학습이 끝난 뒤 다음 결과를 확인합니다.

```text
results/apple/dcgan/epochs/
└─ epoch_015.png

results/apple/dcgan/final/
├─ sample_01.png
├─ ...
└─ sample_16.png
```

`epoch_015.png`를 보면 64개의 생성 이미지 대부분에서  
비슷한 특징이 나타납니다.

![[Pasted image 20261008155025.png]]

```text
빨간색
        ↓
Apple의 색 특징은 나타남

둥근 형태
        ↓
Apple의 전체적인 모양은 나타남

하지만

꼭지
표면 무늬
선명한 윤곽
세부 질감
        ↓
아직 부족함
```

즉 Generator가 완전히 의미 없는 이미지를 만드는 상태에서 벗어나  
**Apple의 붉은색과 둥근 형태 같은 큰 특징은 학습한 것**으로 볼 수 있습니다.

하지만 실제 Apple과 비교하면  
아직 이미지가 거칠고 세부 표현이 부족합니다.

또한 `sample_01.png`와 `sample_16.png`를 비교하면  
서로 다른 Random Noise에서 생성된 이미지인데도  
색과 전체 형태가 매우 비슷합니다.

```
서로 다른 Random Noise
        ↓
Generator
        ↓
생성된 Apple

하지만 결과가 서로 매우 비슷함
        ↓
이미지 다양성이 부족함
```

따라서 이번 `15 Epoch` 결과는 다음과 같이 해석할 수 있습니다.

```
Apple의 빨간색 특징
→ 학습됨

Apple의 둥근 형태
→ 어느 정도 학습됨

세부적인 형태와 질감
→ 아직 부족함

생성 이미지의 다양성
→ 낮아 보임

좋은 합성 학습데이터인가?
→ 아직 바로 사용하기 어려움
```

**이번 실험에서는 DCGAN이 Random Noise에서  
Apple과 비슷한 이미지를 만들어 가는 원리는 확인할 수 있었습니다.**

하지만 현재 결과만으로는  
다양성과 세부 품질이 충분하다고 보기 어렵습니다.

---

## DCGAN Loss도 함께 확인합니다

```bash
tail reports/apple/dcgan_loss.csv
```

`d_loss`와 `g_loss`는 서로 경쟁하는 두 모델의 학습 상태를 보여줍니다.

```
d_loss
→ Discriminator가 Real / Fake를 구분하는 과정의 Loss

g_loss
→ Generator가 만든 Fake를
  Discriminator가 Real로 판단하게 만드는 과정의 Loss
```

이번 실험에서는 다음과 같은 변화가 나타났습니다.

```
Epoch 01
D = 0.2291
G = 5.1650

        ↓

Epoch 02 ~ 07
D = 약 0.006 ~ 0.018
G = 약 6.89 → 8.26

        ↓

Epoch 08
D = 2.7659
G = 9.5973

        ↓

Epoch 09 ~ 15
Loss가 다시 크게 변화

        ↓

Epoch 15
D = 0.4031
G = 3.8614
```

### 초반에는 Discriminator가 매우 강했습니다

Epoch 2~7에서는 `d_loss`가 거의 0에 가까워지고  
`g_loss`는 계속 증가했습니다.

쉽게 해석하면 다음과 같습니다.

```
Discriminator
→ Real과 Fake를 매우 쉽게 구분

Generator
→ 만든 이미지가 Fake라고 쉽게 들킴
→ 학습이 어려운 상태
```

즉 초반에는 **Discriminator가 Generator보다 훨씬 유리한 상태**였습니다.

---

### Epoch 8에서는 학습 상태가 크게 흔들렸습니다

```
Epoch 08
D = 2.7659
G = 9.5973
```

앞 Epoch와 비교하면 두 Loss가 갑자기 크게 변했습니다.

GAN은 두 모델이 서로 경쟁하면서 학습하기 때문에  
이처럼 Loss가 일정하게 감소하지 않고 크게 흔들릴 수 있습니다.

```
Generator 변화
        ↕
Discriminator 변화
        ↕
두 모델의 경쟁 상태 변화
        ↓
Loss가 갑자기 커지거나 작아질 수 있음
```

따라서 Epoch 8의 값만 보고  
"학습이 실패했다"고 판단하지 않습니다.

---

### 이후에는 두 모델의 경쟁 상태가 다시 달라졌습니다

Epoch 9 이후에는 `g_loss`가 점차 감소했고,  
`d_loss`는 초반처럼 거의 0에 머물지 않고 더 큰 값으로 변했습니다.

```
Epoch 09
D = 0.4438
G = 5.8146

        ↓

Epoch 15
D = 0.4031
G = 3.8614
```

즉 학습 초반과 비교하면  
**Discriminator가 Fake를 너무 쉽게 구분하던 상태에서  
두 모델의 경쟁 관계가 다시 변한 것**을 확인할 수 있습니다.

하지만 이것만으로

```
Epoch 15가 가장 좋은 모델이다
```

라고 판단할 수는 없습니다.

---

## GAN Loss는 이렇게 봅니다

일반적인 분류 모델처럼

```
Loss가 계속 감소한다
        ↓
모델이 계속 좋아진다
```

라고 해석하지 않습니다.

GAN에서는 Generator와 Discriminator가 서로 경쟁하기 때문에  
한쪽의 학습 상태가 바뀌면 다른 쪽 Loss도 함께 흔들릴 수 있습니다.

따라서 이번 결과는 다음을 함께 확인해야 합니다.

```
Loss 변화
        +
Epoch별 생성 이미지
        +
Final Sample
        +
이미지 다양성
        +
Mode Collapse 여부
```

> **이번 실험에서는 초반에 Discriminator가 매우 강했고,  
> Epoch 8에서 학습이 크게 흔들린 뒤,  
> 이후 Generator와 Discriminator의 경쟁 상태가 다시 변하는 모습을 확인했습니다.**
> 
> 최종 품질은 Loss 숫자만으로 판단하지 않고  
> 실제 생성된 Apple 이미지와 함께 확인합니다.

---

# 13. 최종 Sample의 다양성과 Mode Collapse 가능성을 확인합니다

DCGAN 학습이 끝나면 두 종류의 생성 결과를 확인할 수 있습니다.

```text
epoch_015.png
→ Epoch 15에서 Generator가 얼마나 발전했는지 확인하는 64장 Grid

final/sample_01.png
...
final/sample_16.png
→ 학습이 모두 끝난 Generator에
   새로운 Random Noise 16개를 넣어 만든 최종 이미지
```


`[epoch_015.png]`

```
→ Epoch 15에서 Generator가 얼마나 발전했는지 확인하는 64장 Grid
```

![[Pasted image 20261008160438.png]]


`[final/sample_01.png ~ final/sample_15.png]`

```
→ 학습이 모두 끝난 Generator에
   새로운 Random Noise 16개를 넣어 만든 최종 이미지
```

![[Pasted image 20261008160253.png]]![[Pasted image 20261008160258.png]]![[Pasted image 20261008160302.png]]![[Pasted image 20261008160307.png]]![[Pasted image 20261008160312.png]]![[Pasted image 20261008160317.png]]![[Pasted image 20261008160321.png]]![[Pasted image 20261008160326.png]]![[Pasted image 20261008160330.png]]![[Pasted image 20261008160336.png]]![[Pasted image 20261008160344.png]]![[Pasted image 20261008160340.png]]![[Pasted image 20261008160351.png]]![[Pasted image 20261008160356.png]]![[Pasted image 20261008160400.png]]![[Pasted image 20261008160406.png]]

둘의 역할은 조금 다릅니다.

`epoch_015.png`는 Epoch별 변화를 비교하기 위한 이미지입니다.

```
Epoch 1
Epoch 2
...
Epoch 15
```

에서 **같은 Random Noise를 계속 사용**하므로,  
Generator가 학습하면서 같은 입력을 어떻게 다르게 만들어 가는지 확인할 수 있습니다.

반면 `final/sample_01.png ~ sample_16.png`는  
학습이 끝난 Generator에 **새로운 Random Noise 16개**를 입력하여 만든 결과입니다.

```
학습 완료된 Generator

Random Noise 1  → sample_01.png
Random Noise 2  → sample_02.png
Random Noise 3  → sample_03.png
...
Random Noise 16 → sample_16.png
```

따라서 Final Sample에서는

> **“서로 다른 Noise를 넣었을 때 Generator가 다양한 Apple을 만들 수 있는가?”**

를 확인합니다.

이번 결과를 보면 `sample_01.png`와 `sample_16.png`의  
색과 전체적인 형태가 매우 비슷합니다.

또 `epoch_015.png`에서도 많은 이미지가 비슷한

```
빨간색
+
둥근 형태
+
비슷한 크기
```

를 가지고 있습니다.

이것은 Generator가 Apple의 큰 특징은 학습했지만  
**다양한 Apple을 만드는 능력은 아직 부족할 가능성**을 보여줍니다.

```
서로 다른 Random Noise
        ↓
Generator
        ↓
서로 다른 Apple이 만들어져야 함

하지만

비슷한 Apple이 계속 생성됨
        ↓
다양성 부족
        ↓
Mode Collapse 가능성 확인
```

`Mode Collapse`는 Generator가 서로 다른 Noise를 받아도  
몇 가지 비슷한 형태의 이미지만 반복해서 만드는 현상입니다.

> 이번 결과에서는 비슷한 이미지가 많이 나타나므로  
> **Mode Collapse의 징후를 의심할 수 있습니다.**
> 
> 다만 `sample_01`과 `sample_16` 두 장만 보고 확정하지 않고,  
> `sample_01 ~ sample_16` 전체와 Epoch Grid를 함께 확인합니다.

---

## 생각해보기 문제 — DCGAN이 더 좋은 Apple을 만들려면 무엇을 바꿔야 할까요?

현재 DCGAN 결과에서는 다음 문제가 보였습니다.

```text
Apple의 빨간색과 둥근 형태
→ 어느 정도 학습됨

하지만

세부적인 윤곽과 질감
→ 부족함

서로 다른 Noise의 생성 결과
→ 서로 비슷함

Loss
→ 학습 중 크게 흔들림
```

### 문제

현재 Generator가 만든 Apple을  
**더 사과답고, 더 다양하게 만들기 위해서는 무엇을 바꾸어 볼 수 있을까요?**

다음 질문을 생각해 봅니다.

```
① Epoch만 계속 늘리면 해결될까?

② Generator와 Discriminator의
   학습 균형을 바꿀 수 있을까?

③ 학습 데이터의 양과 다양성이
   더 많아지면 도움이 될까?
```

---

### 정답

`Epoch`를 늘려 더 오래 학습해 보는 것은 한 가지 방법입니다.

하지만

```
Epoch 증가
=
무조건 품질 향상
```

은 아닙니다.

이번 실험처럼 Generator와 Discriminator의 균형이 흔들리면  
오래 학습해도 결과가 좋아지지 않을 수 있습니다.

더 좋은 결과를 만들기 위해서는 다음과 같은 방법을 생각할 수 있습니다.

```
1. Epoch 조정
→ 조금 더 학습하면서
  Epoch별 생성 결과를 계속 확인

2. Generator / Discriminator 학습 균형 조정
→ Learning Rate
→ 두 모델의 학습 횟수
→ 학습 방법 등을 조정

3. Real Train의 다양성 확보
→ 더 다양한 실제 Apple
→ 필요한 경우 현실적인 Augmentation 활용

4. 더 높은 해상도나 더 큰 모델 사용
→ 세부 표현을 개선할 수 있지만
  연산량과 학습 난이도도 증가
```

특히 이번 결과에서는

```
초반 Discriminator가 너무 강했음
      +
Loss가 크게 흔들렸음
      +
생성 이미지의 다양성이 부족했음
```

을 확인했습니다.

따라서 다음 질문으로 이어집니다.

> **Generator와 Discriminator의 경쟁 방식을 조금 다르게 바꾸면  
> 학습이 더 안정적으로 진행될 수 있을까?**

다음 실습에서는 같은 Apple Train을 그대로 사용하고  
**WGAN-GP**라는 다른 GAN 학습 방법으로 이 질문을 확인합니다.

```
같은 Apple Train

DCGAN
→ Discriminator
→ 학습이 흔들리는 모습 확인

        ↓

다음 실험

WGAN-GP
→ Critic
→ Gradient Penalty
→ 학습 안정성이 어떻게 달라지는지 확인
```

> WGAN-GP가 반드시 더 좋은 Apple을 만든다는 뜻은 아닙니다.  
> **학습 방법을 바꾸었을 때 안정성과 생성 결과가 실제로 어떻게 달라지는지 
> 직접 비교하는 것이 다음 실험의 목적입니다.**

---

# PART 3. WGAN-GP로 학습 안정성을 비교합니다

---

# 14. WGAN-GP가 무엇을 바꾸는지 이해합니다

![[Pasted image 20261008161851.png]]

이 그림은 **DCGAN과 WGAN-GP가 둘 다 사과 이미지를 생성하는 모델이지만,  
판단하는 방식과 학습을 안정시키는 방식이 다르다**는 것을 보여줍니다.

**WGAN-GP도 DCGAN처럼 Apple Train을 사용해서 Random Noise에서 새로운 Apple 이미지를 만듭니다.**  
하지만 **진짜/가짜를 판단하는 방식**과 **학습을 안정시키는 방식**이 다릅니다.

```
공통점
→ 둘 다 사과 데이터를 보고 학습함
→ 둘 다 랜덤 노이즈에서 가짜 사과 이미지를 생성함

차이점
→ 가짜 이미지를 평가하는 방식이 다름
→ 학습이 흔들리지 않게 만드는 방식이 다름
```

### 위쪽 DCGAN 영역

##### DCGAN은 어떻게 학습하나요?

그림의 위쪽 파란 영역이 **DCGAN**입니다.

왼쪽에는 **Real Apple**, 아래에는 **Random Noise**, 가운데에는 **Generator**, 그리고 그 오른쪽에는 **Discriminator**가 있습니다.

흐름은 다음과 같습니다.

```
Random Noise
        ↓
Generator
        ↓
Fake Apple 생성
        ↓
Discriminator
        ↓
Real인가 Fake인가 구분
```

그리고 Discriminator는 **진짜 사과(Real Apple)**도 같이 봅니다.

즉 DCGAN에서는 Discriminator가 이렇게 판단합니다.

```
이 이미지는 진짜에 가까운가?
이 이미지는 가짜에 가까운가?
```

쉽게 말하면:

- **Generator**  
    → “진짜 사과처럼 보이는 이미지를 만들자”
- **Discriminator**  
    → “이게 진짜인지 가짜인지 구분하자”

입니다.

---

##### DCGAN의 출력 예시는 무엇을 뜻하나요?

오른쪽 위 예시 이미지를 보면  
사과처럼 보이는 이미지가 여러 장 생성되어 있습니다.

이 부분은 다음 뜻입니다.

```
DCGAN도 사과의 큰 특징은 배울 수 있음
→ 붉은색
→ 둥근 형태
```

하지만 같이 적혀 있는 설명처럼 아직 문제가 있습니다.

```
세부 질감이 부족함
이미지가 거칠어 보임
비슷한 이미지가 반복될 수 있음
학습 중 Loss가 흔들릴 수 있음
```

즉 **DCGAN은 사과 비슷한 이미지를 만들 수는 있지만,  
학습이 불안정하거나 다양성이 부족해질 수 있다**는 뜻입니다.

---

### 아래쪽 WGAN-GP 영역

이제 아래 크림색 영역이 **WGAN-GP**입니다.

겉보기에는 비슷합니다.

```
Random Noise
        ↓
Generator
        ↓
Fake Apple 생성
```

여기까지는 DCGAN과 같습니다.

---

##### 그런데 무엇이 다를까요?

가장 큰 차이는 **Discriminator 대신 Critic을 사용한다**는 점입니다.

DCGAN에서는

```
Real / Fake를 구분
```

했다면,

WGAN-GP에서는

```
Real / Fake를 딱 잘라 분류하기보다
이미지에 점수를 줌
```

입니다.

그림에도 이렇게 나와 있습니다.

```
Real Image Score
→ 높은 값

Fake Image Score
→ 낮은 값
```

즉 Critic은 이렇게 생각한다고 보면 됩니다.

> “이 이미지는 진짜 데이터 분포에 얼마나 가까운가?”

그래서 WGAN-GP는  
단순히 “진짜/가짜”를 딱 끊어서 보는 것보다  
**얼마나 진짜 데이터에 가까운지 더 부드럽게 평가**하려는 방식이라고 이해하면 됩니다.

---

##### Gradient Penalty는 무엇인가요?

그림 아래쪽 보라색 박스가 **Gradient Penalty**입니다.

이 부분이 WGAN-GP의 핵심 차이입니다.

쉽게 말하면:

> **Critic이 너무 극단적으로 판단해서 학습이 흔들리지 않도록 잡아주는 장치**

입니다.

즉,

```
Critic이 너무 세게 몰아붙이면
→ 학습이 불안정해질 수 있음

Gradient Penalty
→ 판단이 너무 급해지지 않도록 제약
→ 학습을 좀 더 안정적으로 만듦
```

이라고 이해하시면 됩니다.

수학적으로 깊게 들어가지 않아도,  
수업에서는 그냥 이렇게 이해해도 충분합니다.

---

### 그래서 DCGAN과 WGAN-GP의 차이는?

그림을 기준으로 아주 간단히 말하면:

```
DCGAN
→ Discriminator가 Real/Fake를 분류

WGAN-GP
→ Critic이 Real/Fake에 점수를 주고
   Gradient Penalty로 학습을 안정화
```

즉 핵심 차이는 두 가지입니다.

```
1. 판별 방식의 차이
   DCGAN : 분류
   WGAN-GP : 점수 평가

2. 안정화 방식의 차이
   DCGAN : 기본 경쟁 학습
   WGAN-GP : Gradient Penalty 추가
```

---

### 왜 WGAN-GP를 배우나요?

앞에서 DCGAN 결과를 봤을 때 이런 문제가 있었죠.

```
사과처럼 보이기는 함
하지만
세부 표현이 약함
비슷한 이미지가 반복됨
Loss가 흔들림
```

그래서 다음 질문이 생깁니다.

> **“학습 방식을 조금 바꾸면 더 안정적으로 사과를 배울 수 있을까?”**

이 질문에 답해보는 것이 WGAN-GP 실습입니다.

즉 WGAN-GP는

```
DCGAN이 안 좋았으니 완전히 다른 데이터를 쓰는 것
```

이 아니라,

```
같은 Apple Train 데이터를 사용하되
학습 방법을 바꿔서
결과가 더 안정적이고 더 자연스러워지는지 확인하는 실험
```

입니다.

---

### 그림의 오른쪽 아래 결과 예시는 어떻게 보면 되나요?

오른쪽 아래에는 WGAN-GP 결과 예시가 들어 있습니다.

이 부분은 “무조건 더 좋다”는 뜻이 아닙니다.

그림에서도 적혀 있듯이:

```
더 자연스러운 Apple 형태와 질감이 기대됨
이미지 다양성이 DCGAN보다 좋을 수 있음
학습이 더 안정적으로 진행되는 경향이 있음
하지만 반드시 더 좋은 결과가 나오는 것은 아님
실제 실행 결과를 비교해야 함
```

즉 이 부분의 의미는:

> WGAN-GP는 DCGAN의 문제를 줄이기 위해 고안된 방식이지만,  
> 실제로 더 좋은지는 직접 돌려보고 비교해야 한다.

입니다.

---

### 핵심 설명

```
DCGAN
→ 진짜/가짜를 구분하면서 학습
→ 단순하고 기본적인 GAN 구조
→ 하지만 학습이 흔들리거나 다양성이 부족할 수 있음

WGAN-GP
→ 진짜/가짜를 점수로 평가
→ Gradient Penalty로 학습을 안정화
→ 더 자연스럽고 다양한 결과를 기대할 수 있음
→ 그러나 실제로는 결과를 직접 비교해야 함
```

> DCGAN은 “진짜냐 가짜냐”를 구분하며 학습하고,  
> WGAN-GP는 “진짜에 얼마나 가까운가”를 점수로 평가하면서  
> Gradient Penalty로 학습을 더 안정적으로 만들려는 모델입니다.

---

# 15. WGAN-GP 학습 코드를 작성합니다

이번 실습의 목적은

> 학습 방식을 DCGAN에서 WGAN-GP로 바꾸었을 때  
> Loss의 변화와 생성된 Apple 이미지가 어떻게 달라지는지 확인하는 것

입니다.

WGAN-GP가 반드시 더 좋은 이미지를 만든다고 가정하지 않고,  
실제 실행 결과를 보고 비교합니다.

**파일: `scripts/02_train_wgan_gp.py`**

```python
from __future__ import annotations

import argparse
import csv
import random
import shutil
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import torch
import torch.nn as nn

from PIL import Image
from torch.utils.data import DataLoader, Dataset
from torchvision import transforms
from torchvision.utils import save_image


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths, list_images


IMAGE_SIZE = 64
CHANNELS = 3
NOISE_DIM = 100
BATCH_SIZE = 32
LAMBDA_GP = 10.0
N_CRITIC = 5
SEED = 42
VISUAL_SEED = 2026


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--epochs",
        type=int,
        default=15,
    )

    return parser.parse_args()


def set_seed(seed: int) -> None:
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)


class FlatImageDataset(Dataset):
    def __init__(self, root_dir: Path):
        self.files = list_images(root_dir)

        if not self.files:
            raise RuntimeError(
                f"No images found: {root_dir}"
            )

        self.transform = transforms.Compose(
            [
                transforms.Resize(
                    (IMAGE_SIZE, IMAGE_SIZE)
                ),
                transforms.ToTensor(),
                transforms.Normalize(
                    (0.5, 0.5, 0.5),
                    (0.5, 0.5, 0.5),
                ),
            ]
        )

    def __len__(self):
        return len(self.files)

    def __getitem__(self, index):
        image = Image.open(
            self.files[index]
        ).convert("RGB")

        return self.transform(image)


class Generator(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.ConvTranspose2d(
                NOISE_DIM, 512, 4, 1, 0, bias=False
            ),
            nn.BatchNorm2d(512),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                512, 256, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(256),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                256, 128, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(128),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                128, 64, 4, 2, 1, bias=False
            ),
            nn.BatchNorm2d(64),
            nn.ReLU(True),

            nn.ConvTranspose2d(
                64, CHANNELS, 4, 2, 1, bias=False
            ),
            nn.Tanh(),
        )

    def forward(self, noise):
        return self.model(noise)


class Critic(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.Conv2d(
                CHANNELS, 64, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                64, 128, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                128, 256, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                256, 512, 4, 2, 1
            ),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(
                512, 1, 4, 1, 0
            ),
        )

    def forward(self, image):
        return self.model(image).view(-1)


def gradient_penalty(
    critic,
    real,
    fake,
    device,
):
    batch_size = real.size(0)

    alpha = torch.rand(
        batch_size,
        1,
        1,
        1,
        device=device,
    )

    interpolated = (
        alpha * real
        + (1 - alpha) * fake
    )

    interpolated.requires_grad_(True)

    scores = critic(interpolated)

    gradients = torch.autograd.grad(
        outputs=scores,
        inputs=interpolated,
        grad_outputs=torch.ones_like(scores),
        create_graph=True,
        retain_graph=True,
        only_inputs=True,
    )[0]

    gradients = gradients.view(
        batch_size,
        -1,
    )

    penalty = (
        (
            gradients.norm(
                2,
                dim=1,
            )
            - 1
        )
        ** 2
    ).mean()

    return penalty


def reset_directory(directory: Path) -> None:
    if directory.exists():
        shutil.rmtree(directory)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )


def make_noise(
    count: int,
    device,
    seed: int,
):
    noise_generator = torch.Generator(
        device=device
    ).manual_seed(seed)

    return torch.randn(
        count,
        NOISE_DIM,
        1,
        1,
        generator=noise_generator,
        device=device,
    )


def save_final_samples(
    generator,
    output_dir,
    device,
) -> None:
    generator.eval()

    final_noise = make_noise(
        16,
        device,
        VISUAL_SEED + 1,
    )

    with torch.no_grad():
        samples = generator(final_noise)
        samples = (samples + 1) / 2

    for index, image in enumerate(
        samples,
        start=1,
    ):
        save_image(
            image,
            output_dir / f"sample_{index:02d}.png",
        )


def main() -> None:
    args = parse_args()

    if args.epochs < 1:
        raise ValueError(
            "--epochs는 1 이상이어야 합니다."
        )

    set_seed(SEED)
    paths = day02_paths(args.dataset)

    reset_directory(
        paths["wgan_epochs"]
    )

    reset_directory(
        paths["wgan_final"]
    )

    device = torch.device(
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Dataset:", args.dataset)

    dataset = FlatImageDataset(
        paths["train"]
    )

    loader_generator = torch.Generator()
    loader_generator.manual_seed(SEED)

    loader = DataLoader(
        dataset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        generator=loader_generator,
        drop_last=False,
    )

    generator = Generator().to(device)
    critic = Critic().to(device)

    optimizer_g = torch.optim.Adam(
        generator.parameters(),
        lr=1e-4,
        betas=(0.0, 0.9),
    )

    optimizer_c = torch.optim.Adam(
        critic.parameters(),
        lr=1e-4,
        betas=(0.0, 0.9),
    )

    fixed_noise = make_noise(
        64,
        device,
        VISUAL_SEED,
    )

    history = []

    for epoch in range(
        1,
        args.epochs + 1,
    ):
        generator.train()
        critic.train()

        epoch_c = 0.0
        epoch_g = 0.0
        critic_update_count = 0
        batch_counter = 0

        for real_images in loader:
            real_images = real_images.to(
                device
            )

            batch_size = real_images.size(
                0
            )

            for _ in range(N_CRITIC):
                noise = torch.randn(
                    batch_size,
                    NOISE_DIM,
                    1,
                    1,
                    device=device,
                )

                fake_images = generator(
                    noise
                )

                critic_real = critic(
                    real_images
                ).mean()

                critic_fake = critic(
                    fake_images.detach()
                ).mean()

                gp = gradient_penalty(
                    critic,
                    real_images,
                    fake_images.detach(),
                    device,
                )

                loss_c = (
                    critic_fake
                    - critic_real
                    + LAMBDA_GP * gp
                )

                optimizer_c.zero_grad()
                loss_c.backward()
                optimizer_c.step()

                epoch_c += loss_c.item()
                critic_update_count += 1

            noise = torch.randn(
                batch_size,
                NOISE_DIM,
                1,
                1,
                device=device,
            )

            fake_images = generator(
                noise
            )

            loss_g = -critic(
                fake_images
            ).mean()

            optimizer_g.zero_grad()
            loss_g.backward()
            optimizer_g.step()

            epoch_g += loss_g.item()
            batch_counter += 1

        avg_c = (
            epoch_c
            / critic_update_count
        )

        avg_g = (
            epoch_g
            / batch_counter
        )

        history.append(
            {
                "epoch": epoch,
                "critic_loss": avg_c,
                "generator_loss": avg_g,
            }
        )

        print(
            f"Epoch {epoch:02d}/"
            f"{args.epochs} "
            f"C={avg_c:.4f} "
            f"G={avg_g:.4f}"
        )

        generator.eval()

        with torch.no_grad():
            samples = generator(
                fixed_noise
            )

            samples = (
                samples + 1
            ) / 2

            save_image(
                samples,
                paths["wgan_epochs"]
                / f"epoch_{epoch:03d}.png",
                nrow=8,
            )

    model_g = (
        paths["model"]
        / "wgan_gp_generator.pt"
    )

    model_c = (
        paths["model"]
        / "wgan_gp_critic.pt"
    )

    torch.save(
        generator.state_dict(),
        model_g,
    )

    torch.save(
        critic.state_dict(),
        model_c,
    )

    loss_path = (
        paths["report"]
        / "wgan_gp_loss.csv"
    )

    pd.DataFrame(
        history
    ).to_csv(
        loss_path,
        index=False,
        encoding="utf-8-sig",
    )

    save_final_samples(
        generator,
        paths["wgan_final"],
        device,
    )

    metadata_path = (
        paths["report"]
        / "wgan_gp_metadata.csv"
    )

    with metadata_path.open(
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        writer = csv.DictWriter(
            file,
            fieldnames=[
                "method",
                "dataset",
                "epochs",
                "batch_size",
                "noise_dim",
                "seed",
                "epoch_visual_seed",
                "final_sample_seed",
                "learning_rate",
                "n_critic",
                "lambda_gp",
                "model_generator",
                "model_critic",
            ],
        )

        writer.writeheader()

        writer.writerow(
            {
                "method": "wgan_gp",
                "dataset": args.dataset,
                "epochs": args.epochs,
                "batch_size": BATCH_SIZE,
                "noise_dim": NOISE_DIM,
                "seed": SEED,
                "epoch_visual_seed": VISUAL_SEED,
                "final_sample_seed": VISUAL_SEED + 1,
                "learning_rate": 1e-4,
                "n_critic": N_CRITIC,
                "lambda_gp": LAMBDA_GP,
                "model_generator": str(
                    model_g
                ),
                "model_critic": str(
                    model_c
                ),
            }
        )

    print(
        "WGAN-GP Training Completed"
    )

    print("Loss    :", loss_path)
    print(
        "Samples :",
        paths["wgan_final"],
    )


if __name__ == "__main__":
    main()
```

### 의사코드

```text
1일차 Apple Train을 불러온다
        ↓
Apple 이미지를 64×64로 준비한다
        ↓
두 모델을 만든다

Generator
→ Random Noise에서 Fake Apple 생성

Critic
→ Real Apple과 Fake Apple에 점수를 줌

        ↓

[WGAN-GP 학습 시작]

Real Apple을 가져온다
        ↓
Random Noise로 Fake Apple을 만든다
        ↓

Critic이

Real Apple에 점수를 주고
Fake Apple에 점수를 준다
        ↓

Real과 Fake 사이에서
Gradient Penalty를 계산한다
        ↓

Critic의 Loss를 계산하고
Critic을 학습한다
        ↓

이 Critic 학습을 5번 반복한다
        ↓

이번에는 새로운 Random Noise로
Fake Apple을 다시 만든다
        ↓

Critic이 Fake Apple에 점수를 준다
        ↓

Generator가
더 높은 Critic 점수를 받도록 학습한다
        ↓

Apple Train 전체에 반복
        ↓
한 Epoch 완료
        ↓

설정한 Epoch 수만큼 반복
        ↓

매 Epoch마다 같은 fixed_noise로
생성 결과를 저장한다
        ↓

학습 완료
        ↓

Generator / Critic 모델 저장
Final Sample 저장
Loss 저장
Metadata 저장
```

코드에서는 다음 세 부분을 찾아보세요.

```text
① Critic
→ DCGAN의 Discriminator와 무엇이 다른가?

② gradient_penalty()
→ 학습을 안정시키기 위한 Penalty가 어디서 계산되는가?

③ N_CRITIC
→ Generator 1번에 Critic을 몇 번 학습하는가?
```

정답

### ① Critic

```python

# DCGAN의 Discriminator와 달리 마지막에 Sigmoid()가 없습니다.

class Critic(nn.Module):
    def __init__(self):
        super().__init__()

        self.model = nn.Sequential(
            nn.Conv2d(CHANNELS, 64, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(64, 128, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(128, 256, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(256, 512, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),

            nn.Conv2d(512, 1, 4, 1, 0),
        )

    def forward(self, image):
        return self.model(image).view(-1)
```

```
핵심
→ DCGAN의 Discriminator는 마지막에 Sigmoid()를 사용하여
  Real / Fake 확률처럼 판단했습니다.

→ WGAN-GP의 Critic은 Sigmoid()를 사용하지 않고
  이미지에 제한되지 않은 점수를 출력합니다.
```

### ② gradient_penalty()

```python

# Real과 Fake 사이의 중간 이미지를 만들고
# Gradient 크기를 이용해 Penalty를 계산합니다.

def gradient_penalty(
    critic,
    real,
    fake,
    device,
):
    batch_size = real.size(0)

    alpha = torch.rand(
        batch_size,
        1,
        1,
        1,
        device=device,
    )

    interpolated = (
        alpha * real
        + (1 - alpha) * fake
    )

    interpolated.requires_grad_(True)

    scores = critic(interpolated)

    gradients = torch.autograd.grad(
        outputs=scores,
        inputs=interpolated,
        grad_outputs=torch.ones_like(scores),
        create_graph=True,
        retain_graph=True,
        only_inputs=True,
    )[0]

    gradients = gradients.view(
        batch_size,
        -1,
    )

    penalty = (
        (
            gradients.norm(2, dim=1)
            - 1
        )
        ** 2
    ).mean()

    return penalty
```

### ③ N_CRITIC

```python
# Generator 1번 학습 전에 Critic을 5번 학습합니다.

N_CRITIC = 5
```

실제 학습 Loop에서는 다음 부분입니다.

```python
for _ in range(N_CRITIC):
    # Critic 학습
    ...
    optimizer_c.zero_grad()
    loss_c.backward()
    optimizer_c.step()

# Critic 5번 학습 후

# Generator 1번 학습
optimizer_g.zero_grad()
loss_g.backward()
optimizer_g.step()
```

```
핵심

Critic 5번 학습
        ↓
Generator 1번 학습
        ↓
반복
```

```text
Critic
→ Real / Fake를 점수로 평가

Gradient Penalty
→ Critic 학습을 안정화

N_CRITIC = 5
→ Generator 1번 전에 Critic을 5번 학습
```

---
## 성능이 좋지 않을 때 어디를 확인해야 하나요?

WGAN-GP 코드를 전부 외울 필요는 없습니다.

실제 생성 결과가 좋지 않을 때는
다음 핵심 부분을 먼저 확인합니다.

| 문제가 보일 때 | 먼저 확인할 코드 | 의미 |
|---|---|---|
| Apple 형태가 잘 만들어지지 않음 | `Generator` | 이미지를 만드는 모델 구조 |
| Critic이 너무 강하거나 약해 보임 | `N_CRITIC` | Generator 1번당 Critic 학습 횟수 |
| 학습이 심하게 흔들림 | `LAMBDA_GP` | Gradient Penalty 강도 |
| 전체 학습 속도가 너무 빠르거나 불안정함 | `learning_rate` | 가중치를 얼마나 크게 수정하는지 |
| 결과가 아직 덜 학습된 것 같음 | `epochs` | 전체 학습 반복 횟수 |
| 비슷한 이미지만 반복됨 | Final Sample + Generator 학습 상태 | 다양성 부족 여부 |
| GPU 메모리가 부족함 | `BATCH_SIZE` | 한 번에 학습할 이미지 수 |

---

### ① 가장 먼저 볼 값 — `N_CRITIC`

```python
N_CRITIC = 5
```

현재 코드는

```
Critic 5번 학습
        ↓
Generator 1번 학습
```

으로 동작합니다.

즉 Critic이 먼저 Real과 Fake의 차이를 충분히 평가한 뒤  
Generator를 한 번 개선하는 구조입니다. 붙여넣은 마크다운

실제 학습 Loop에서도 다음 부분이 이것을 결정합니다.

```python
for _ in range(N_CRITIC):
```

붙여넣은 마크다운

> Critic이 지나치게 강하거나 Generator 학습이 잘 따라오지 못한다고 판단되면  
> `N_CRITIC`은 확인해야 할 중요한 실험 값입니다.

다만 처음부터 임의로 계속 바꾸지 않고  
기준값 `5`로 먼저 결과를 확인합니다.

---

### ② WGAN-GP의 핵심 — `LAMBDA_GP`

```python
LAMBDA_GP = 10.0
```

이 값은 **Gradient Penalty를 Loss에 얼마나 강하게 반영할지** 결정합니다. 붙여넣은 마크다운

실제 Critic Loss는 다음처럼 계산됩니다.

```python
loss_c = (
    critic_fake
    - critic_real
    + LAMBDA_GP * gp
)
```

붙여넣은 마크다운

쉽게 보면

```
Real과 Fake의 점수 차이
        +
Gradient Penalty
        ↓
Critic 학습
```

입니다.

`LAMBDA_GP`는 WGAN-GP의 **학습 안정성과 직접 연결된 중요한 값**이므로,  
Loss가 지나치게 흔들리는 실험에서는 확인해야 합니다.

하지만 `10.0`은 먼저 사용할 기준값이고,  
결과가 나쁘다고 바로 값을 바꾸기보다는  
생성 이미지와 Loss를 함께 확인합니다.

---

### ③ 학습 속도를 결정하는 `learning_rate`

현재 Generator와 Critic은 모두 다음 학습률을 사용합니다.

```python
lr=1e-4
```

그리고 Adam 설정은

```python
betas=(0.0, 0.9)
```

입니다. 붙여넣은 마크다운

쉽게 말하면 Learning Rate는

```
한 번 학습할 때
모델의 가중치를 얼마나 크게 움직일 것인가?
```

를 결정합니다.

```
너무 큼
→ 학습이 크게 흔들릴 수 있음

너무 작음
→ 학습이 매우 느릴 수 있음
```

따라서 **학습이 지나치게 불안정하거나 거의 발전하지 않을 때**  
확인하는 값입니다.

---

### ④ 실제 Apple을 만드는 부분은 `Generator`입니다

```python
class Generator(nn.Module):
```

Generator가 최종적으로 Synthetic Apple을 만드는 모델입니다.

```
Random Noise
        ↓
Generator
        ↓
64 × 64 Fake Apple
```

따라서 Epoch를 충분히 학습했는데도

```
Apple 형태가 계속 무너짐
윤곽이 지나치게 흐림
세부 특징이 거의 없음
```

같은 문제가 지속된다면  
단순한 학습 횟수뿐 아니라 **Generator의 모델 구조 자체**도 개선 대상이 될 수 있습니다. 붙여넣은 마크다운

다만 이번 수업에서는 모델 층을 직접 변경하는 것보다  
기본 WGAN-GP 구조를 이해하는 것이 먼저입니다.

---

### ⑤ `epochs`는 늘릴 수 있지만 무조건 정답은 아닙니다

현재 기본값은

```python
--epochs 15
```

입니다. 붙여넣은 마크다운

15 Epoch 결과가 아직 충분하지 않다면

```
15 Epoch
→ 30 Epoch
→ 결과 비교
```

처럼 더 학습해 볼 수 있습니다.

하지만

```
Epoch 증가
=
무조건 더 좋은 이미지
```

는 아닙니다.

따라서 Epoch를 늘릴 때도 반드시

```
Epoch별 이미지
+
Critic Loss
+
Generator Loss
+
Final Sample 다양성
```

을 함께 확인해야 합니다.

---

## 결국 성능을 개선할 때는 이 순서로 봅니다

```
생성 이미지가 이상함
        ↓

① 충분히 학습했는가?
→ Epoch

        ↓

② Critic과 Generator의 균형은 적절한가?
→ N_CRITIC

        ↓

③ Gradient Penalty가 정상적으로 작동하는가?
→ LAMBDA_GP

        ↓

④ 학습이 너무 빠르거나 느리지 않은가?
→ Learning Rate

        ↓

⑤ 그래도 형태가 부족한가?
→ Generator 구조
→ 데이터 양과 다양성
```

> **핵심**
> 
> WGAN-GP의 성능이 좋지 않을 때는  
> 무조건 Epoch만 늘리는 것이 아니라  
> `N_CRITIC`, `LAMBDA_GP`, `learning_rate`, Generator 구조와  
> 실제 생성 결과를 함께 확인해야 합니다.

````
N_CRITIC
→ Critic과 Generator의 학습 균형

LAMBDA_GP
→ Gradient Penalty 강도

learning_rate
→ 두 모델이 학습하는 속도
````

---

# 16. WGAN-GP를 실행하고 실제 결과를 확인합니다

먼저 코드가 정상적으로 실행되는지 `1 Epoch`로 Smoke Test를 합니다.

```bash
python scripts/02_train_wgan_gp.py \
  --dataset apple \
  --epochs 1
```

실행 결과:

```
Device : cuda
Dataset: apple
Epoch 01/1 C=-80.8056 G=75.5820
WGAN-GP Training Completed
```

`WGAN-GP Training Completed`가 출력되었으므로  
GPU에서 학습 코드가 정상적으로 동작하는 것을 확인했습니다.

이제 본 학습을 `15 Epoch`로 실행합니다.

```bash
python scripts/02_train_wgan_gp.py \
  --dataset apple \
  --epochs 15
```

> Smoke Test 결과에서 이어서 학습하는 것이 아닙니다.  
> 본 학습은 다시 처음부터 시작합니다.

## 실제 생성 결과를 확인합니다

15 Epoch가 끝난 뒤 `epoch_015.png`를 확인합니다.

```
results/apple/wgan_gp/epochs/
└─ epoch_015.png
```

![[Pasted image 20261008165551.png]]

이번 결과에서는 대부분의 이미지에서

```
붉은색
  +
Apple의 둥근 형태
  +
밝은 반사 영역
```

이 비교적 뚜렷하게 나타났습니다.

즉 WGAN-GP의 Generator가  
Apple Train에서 **사과의 큰 색과 형태를 학습했다**고 볼 수 있습니다.

실제 결과를 조금 더 자세히 보면 다음과 같습니다.

```
Apple의 붉은색
→ 잘 나타남

Apple의 전체적인 형태
→ 비교적 잘 나타남

사과마다 형태 차이
→ 어느 정도 나타남

밝은 반사 위치
→ 이미지마다 조금씩 달라짐

세부 질감과 윤곽
→ 아직 거칠고 흐린 부분이 있음
```

특히 모든 이미지가 완전히 같은 동그란 모양으로 반복되기보다는  
조금 더 길거나 넓은 Apple,  
반사 위치가 다른 Apple 등이 나타납니다.

따라서 현재 결과에서는  
**DCGAN 실험에서 보였던 심한 반복 패턴이 다소 줄어든 모습**을 확인할 수 있습니다.

하지만 아직

```
선명한 Apple 윤곽
표면의 자연스러운 질감
꼭지와 세부 구조
실제 사진 수준의 표현
```

까지 충분히 만들어진 것은 아닙니다.

즉,

> **WGAN-GP가 Apple의 큰 형태와 다양성을 어느 정도 학습했지만,  
> 아직 Real Apple과 같은 품질의 합성데이터라고 보기는 어렵습니다.**

---

## Final Sample도 확인합니다

학습이 끝난 Generator에
**서로 다른 Random Noise 16개**를 입력하여 최종 Apple 이미지를 생성했습니다.

![[Pasted image 20261008165655.png]]![[Pasted image 20261008165659.png]]![[Pasted image 20261008165704.png]]![[Pasted image 20261008165708.png]]![[Pasted image 20261008165712.png]]![[Pasted image 20261008165718.png]]![[Pasted image 20261008165722.png]]![[Pasted image 20261008165726.png]]![[Pasted image 20261008165730.png]]![[Pasted image 20261008165736.png]]![[Pasted image 20261008165741.png]]![[Pasted image 20261008165745.png]]![[Pasted image 20261008165750.png]]![[Pasted image 20261008165754.png]]![[Pasted image 20261008165759.png]]![[Pasted image 20261008165804.png]]

```
results/apple/wgan_gp/final/

sample_01.png
...
sample_16.png
```

이번에는 한두 장만 보지 않고  
**16장의 전체 결과를 함께 비교합니다.**

실제 결과를 보면 모든 이미지에서 공통적으로

```
붉은색
+
Apple과 비슷한 전체 형태
+
흰 배경
```

이 유지되고 있습니다.

즉 Generator가  
**“붉은 Apple 한 개”라는 큰 특징은 비교적 안정적으로 학습했다**고 볼 수 있습니다.

---

### 서로 다른 형태도 나타납니다

16장을 자세히 보면 완전히 같은 Apple만 반복되지는 않습니다.

```
둥근 Apple
조금 길쭉한 Apple
가로로 넓은 Apple
약간 각진 Apple
울퉁불퉁한 Apple
```

처럼 전체적인 형태에 차이가 있습니다.

밝게 반사되는 부분도

```
왼쪽에 있는 이미지
가운데에 있는 이미지
거의 보이지 않는 이미지
```

등으로 조금씩 달라집니다.

즉,

```
서로 다른 Random Noise
        ↓
Generator
        ↓
조금씩 다른 Apple 생성
```

이 실제 결과에서 확인됩니다.

---

### DCGAN 결과와 비교하면 어떨까요?

앞에서 실행한 DCGAN에서는 많은 이미지가

```
비슷한 둥근 형태
  +
비슷한 크기
  +
비슷한 색
```

로 반복되는 모습이 강했습니다.

이번 WGAN-GP 결과에서는

```
DCGAN

비슷한 형태가 많이 반복
        ↓

WGAN-GP

형태와 밝은 영역에
조금 더 차이가 나타남
```

을 관찰할 수 있습니다.

따라서 **이번 15 Epoch 실험 결과만 놓고 보면**  
WGAN-GP에서 DCGAN보다 생성 이미지의 다양성이 조금 더 나타났다고 볼 수 있습니다.

---

### 하지만 아직 부족한 부분도 있습니다

다양성이 조금 좋아졌다고 해서  
실제 Apple 사진 수준의 품질이 된 것은 아닙니다.

현재 결과에서는

```
Apple의 큰 형태
→ 잘 나타남

붉은색
→ 잘 유지됨

이미지 간 다양성
→ 어느 정도 나타남

하지만

꼭지
→ 거의 표현되지 않음

표면 질감
→ 흐리고 뭉개짐

윤곽
→ 일부 이미지에서 울퉁불퉁하거나 부자연스러움

세부 표현
→ 아직 부족함
```

을 확인할 수 있습니다.

특히 일부 Apple은 둥근 사과라기보다  
약간 네모나거나 찌그러진 형태로 보입니다.

따라서 이번 결과는

> **“WGAN-GP가 Apple의 색과 큰 형태를 학습했고,  
> 서로 다른 Noise에서 어느 정도 다양한 Apple도 만들었다.”**

라고 해석할 수 있습니다.

하지만

> **“실제 Apple과 거의 같은 고품질 합성데이터를 만들었다.”**

라고 판단하기에는 아직 부족합니다.

---

## Mode Collapse도 함께 판단합니다

이번 Final Sample에서는 서로 다른 형태가 어느 정도 나타나므로

```
서로 다른 Noise
        ↓
거의 완전히 같은 이미지 반복
```

과 같은 **심한 Mode Collapse는 현재 Sample에서는 뚜렷하게 보이지 않습니다.**

다만

```
모두 비슷한 붉은색
비슷한 크기
비슷한 배경
```

처럼 공통적인 패턴은 여전히 강합니다.

따라서

> **DCGAN보다 다양성은 개선되어 보이지만,  
> 16장의 결과만으로 Mode Collapse가 완전히 해결되었다고 단정하지 않습니다.**

라고 해석합니다.

---

## 이번 Final Sample의 결론

```
Apple 색상
→ 잘 학습됨

Apple 큰 형태
→ 잘 학습됨

이미지 간 형태 차이
→ 어느 정도 있음

DCGAN 대비 다양성
→ 이번 실험에서는 조금 개선되어 보임

세부 질감과 윤곽
→ 아직 부족함

심한 Mode Collapse
→ 현재 결과에서는 뚜렷하지 않음

바로 학습데이터로 사용 가능한가?
→ 아직 추가 검수가 필요함
```

> **이번 WGAN-GP 결과에서 가장 중요한 변화는  
> “사과처럼 보인다”뿐 아니라  
> 서로 다른 Noise에서 조금씩 다른 Apple이 만들어졌다는 점입니다.**
> 
> 하지만 다음 단계에서는 DCGAN과 WGAN-GP 결과를 같은 화면에 놓고  
> **형태·색·다양성·Artifact를 직접 비교한 뒤 판단합니다.**

---

## WGAN-GP Loss를 실제 결과로 확인합니다

```bash
tail reports/apple/wgan_gp_loss.csv
```

실제 결과는 다음과 같습니다.

```
Epoch 06   Critic = -42.9838   Generator =   0.0064
Epoch 07   Critic = -34.7535   Generator =   1.9262
Epoch 08   Critic = -29.4769   Generator =  -7.7966
Epoch 09   Critic = -25.6273   Generator =  -3.0862
Epoch 10   Critic = -21.8970   Generator = -16.2603
Epoch 11   Critic = -19.2320   Generator =  -8.4732
Epoch 12   Critic = -17.7947   Generator = -14.9508
Epoch 13   Critic = -16.5619   Generator = -14.6394
Epoch 14   Critic = -15.3619   Generator = -19.2123
Epoch 15   Critic = -14.8091   Generator = -18.4949
```

WGAN-GP의 Loss는 DCGAN의 `BCELoss`와 계산 방식이 다르기 때문에  
**음수가 나와도 오류가 아닙니다.**

이번 결과에서는 Critic Loss가

```
-42.98
   ↓
-14.81
```

처럼 변화했습니다.

Generator Loss도

```
0 근처
   ↓
중간에 오르내림
   ↓
Epoch 15에서 -18.49
```

처럼 계속 변했습니다.

즉 두 모델이 학습하면서  
**Critic이 Real과 Fake에 주는 점수 관계도 계속 달라지고 있다는 것**을 확인할 수 있습니다.

특히 Generator Loss는 일정하게 감소하지 않고 중간에 다시 올라갔다 내려갑니다.

```
Epoch 10  → -16.26
Epoch 11  →  -8.47
Epoch 12  → -14.95
Epoch 14  → -19.21
Epoch 15  → -18.49
```

이처럼 WGAN-GP도 Loss가 한 방향으로만 움직이지 않습니다.

중요한 것은 다음입니다.

```
Loss가 더 작다
        ≠
이미지가 무조건 더 좋다
```

따라서 이번 결과는

```
Critic Loss
+
Generator Loss
+
Epoch별 생성 이미지
+
Final Sample의 형태와 다양성
```

을 함께 보고 판단합니다.

> **이번 실험에서는 Loss 값이 계속 변화하면서도  
> Generator가 붉은 Apple 형태를 만들고,  
> Final Sample에서도 서로 조금씩 다른 형태가 나타났습니다.**
> 
> 따라서 Loss 숫자 하나보다  
> 실제 생성 이미지가 어떻게 변했는지를 함께 보는 것이 중요합니다.

---

# 17. DCGAN과 WGAN-GP 결과를 한 화면에서 비교하는 코드를 작성합니다

앞에서는 DCGAN과 WGAN-GP를 각각 학습하고
최종 Apple 이미지를 확인했습니다.

하지만 두 결과를 따로 보면

```text
어느 쪽이 더 사과답게 보이는지
어느 쪽이 더 다양한지
어느 쪽에서 비슷한 이미지가 반복되는지
```

를 비교하기가 어렵습니다.

그래서 이번에는 두 모델의 최종 결과를  
**같은 화면에 나란히 배치하여 직접 비교하는 코드**를 작성합니다.

이 코드는 새로운 모델을 학습하지 않습니다.

```
DCGAN Final Sample
        +
WGAN-GP Final Sample
        ↓
같은 크기의 Grid로 정리
        ↓
한 화면에 나란히 배치
        ↓
형태 · 색 · 다양성 · 반복 패턴 비교
```

즉 이번 코드의 목적은

> **DCGAN과 WGAN-GP 중 어떤 결과가 현재 실험에서 더 자연스럽고 다양한 Apple을 만들었는지 눈으로 비교하기 위한 것**

입니다.

비교할 때는 Loss 숫자만 보지 않고  
실제 생성 이미지를 중심으로 확인합니다.

**파일: `scripts/03_compare_gan_results.py`**

```python
from __future__ import annotations

import argparse
import sys
from pathlib import Path

from PIL import Image, ImageDraw


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
    list_images,
)


TILE_SIZE = 128
GRID_COLUMNS = 4


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def make_grid(
    image_paths,
    title,
):
    selected = image_paths[:16]

    if not selected:
        raise RuntimeError(
            f"{title} 이미지가 없습니다."
        )

    rows = (
        len(selected)
        + GRID_COLUMNS - 1
    ) // GRID_COLUMNS

    header = 40

    canvas = Image.new(
        "RGB",
        (
            TILE_SIZE * GRID_COLUMNS,
            header + TILE_SIZE * rows,
        ),
        "white",
    )

    draw = ImageDraw.Draw(canvas)

    draw.text(
        (10, 12),
        title,
        fill="black",
    )

    for index, path in enumerate(
        selected
    ):
        image = Image.open(
            path
        ).convert("RGB")

        image = image.resize(
            (TILE_SIZE, TILE_SIZE)
        )

        column = index % GRID_COLUMNS
        row = index // GRID_COLUMNS

        x = column * TILE_SIZE
        y = header + row * TILE_SIZE

        canvas.paste(
            image,
            (x, y),
        )

    return canvas


def main() -> None:
    args = parse_args()
    paths = day02_paths(args.dataset)

    dcgan_files = list_images(
        paths["dcgan_final"]
    )

    wgan_files = list_images(
        paths["wgan_final"]
    )

    dcgan_grid = make_grid(
        dcgan_files,
        "DCGAN Final Samples",
    )

    wgan_grid = make_grid(
        wgan_files,
        "WGAN-GP Final Samples",
    )

    width = (
        dcgan_grid.width
        + wgan_grid.width
    )

    height = max(
        dcgan_grid.height,
        wgan_grid.height,
    )

    canvas = Image.new(
        "RGB",
        (width, height),
        "white",
    )

    canvas.paste(
        dcgan_grid,
        (0, 0),
    )

    canvas.paste(
        wgan_grid,
        (dcgan_grid.width, 0),
    )

    output_path = (
        paths["compare"]
        / "dcgan_vs_wgan_gp.jpg"
    )

    canvas.save(output_path)

    print(
        "Saved:",
        output_path,
    )


if __name__ == "__main__":
    main()
```

### 코드 작성 전 의사코드

```text
실행할 데이터셋을 선택한다
→ apple 또는 banana

        ↓

DCGAN이 만든 Final Sample 이미지를 불러온다
        ↓
최대 16장을 선택한다
        ↓
각 이미지를 128×128 크기로 맞춘다
        ↓
4열 Grid로 배치한다
        ↓
"DCGAN Final Samples" 제목을 붙인다

        ↓

WGAN-GP가 만든 Final Sample 이미지를 불러온다
        ↓
최대 16장을 선택한다
        ↓
각 이미지를 128×128 크기로 맞춘다
        ↓
같은 4열 Grid로 배치한다
        ↓
"WGAN-GP Final Samples" 제목을 붙인다

        ↓

DCGAN Grid와 WGAN-GP Grid를
왼쪽과 오른쪽에 나란히 배치한다

        ↓

두 모델의 결과를 한 화면에서 비교할 수 있는
이미지 한 장을 만든다

        ↓

dcgan_vs_wgan_gp.jpg
파일로 저장한다

        ↓

저장된 파일 경로를 화면에 출력한다
```

핵심만 더 짧게 보면 다음 흐름입니다.

```
DCGAN Final 16장 → Grid 생성
                     +
WGAN-GP Final 16장 → Grid 생성
                     ↓
              좌우로 나란히 배치
                     ↓
          dcgan_vs_wgan_gp.jpg 저장
```

실행합니다.

```bash
python scripts/03_compare_gan_results.py \
  --dataset apple
```

---

# 18. DCGAN과 WGAN-GP의 실제 결과를 비교합니다

두 모델의 Final Sample 16장을 한 화면에서 비교했습니다.

![[Pasted image 20261008170819.png]]

```text
왼쪽
→ DCGAN Final Samples

오른쪽
→ WGAN-GP Final Samples
```

실제 결과를 보면 두 모델 모두  
**붉은색 Apple이라는 큰 특징은 학습했습니다.**

하지만 생성 결과의 성격은 서로 다릅니다.

| 비교 항목       | DCGAN                 | WGAN-GP                  |
| ----------- | --------------------- | ------------------------ |
| 빨간색 특징      | 잘 나타남                 | 잘 나타남                    |
| Apple 형태    | 대부분 비슷한 둥근 형태         | 둥글거나 길쭉한 등 형태 차이가 더 큼    |
| 이미지 다양성     | 낮음                    | DCGAN보다 높음               |
| 비슷한 결과 반복   | 많이 보임                 | 상대적으로 적음                 |
| 세부 질감       | 거칠고 흐림                | 여전히 흐리지만 밝기·형태 차이가 더 나타남 |
| 형태 왜곡       | 적지만 매우 비슷함            | 일부 이미지가 찌그러지거나 각져 보임     |
| 심한 Artifact | 뚜렷한 붕괴는 없지만 Noise가 많음 | 일부 형태 왜곡이 보이지만 심한 붕괴는 없음 |

## DCGAN 결과부터 봅니다

DCGAN 결과에서는 대부분의 Apple이

```
비슷한 붉은색
   +
비슷한 둥근 형태
   +
비슷한 크기
```

로 만들어졌습니다.

즉 Apple처럼 보이는 큰 특징은 학습했지만,  
16장의 결과가 서로 매우 비슷합니다.

```
서로 다른 Random Noise
        ↓
DCGAN Generator
        ↓
비슷한 형태의 Apple 반복
```

따라서 이번 결과에서는  
**생성 다양성이 부족한 것이 DCGAN의 가장 큰 문제**로 보입니다.

또한 표면이 거칠고 Noise가 많이 보이며,  
꼭지나 자연스러운 세부 질감은 거의 표현되지 않았습니다.

---

## WGAN-GP 결과를 봅니다

WGAN-GP도 붉은 Apple의 특징은 잘 유지하고 있습니다.

하지만 DCGAN과 비교하면

```
둥근 형태
조금 길쭉한 형태
넓은 형태
밝은 반사 위치가 다른 형태
```

처럼 이미지 사이의 차이가 더 많이 나타납니다.

즉 이번 실험에서는

```
DCGAN
→ 비슷한 Apple 반복이 많음

WGAN-GP
→ 서로 다른 형태가 조금 더 나타남
```

이라는 차이를 확인할 수 있습니다.

다만 WGAN-GP가 모든 면에서 더 좋은 것은 아닙니다.

일부 이미지는

```
형태가 찌그러져 보임
약간 각진 형태가 나타남
Apple 윤곽이 자연스럽지 않음
```

과 같은 문제도 보입니다.

따라서 WGAN-GP는 **다양성은 증가했지만,  
형태 안정성까지 완전히 해결된 것은 아닙니다.**

---

## 이번 실제 결과의 핵심 차이

```
DCGAN

장점
→ Apple의 붉은색과 둥근 형태가 일정함

문제
→ 결과가 서로 너무 비슷함
→ 다양성이 부족함
→ 질감이 거침


WGAN-GP

장점
→ Apple 형태와 밝기 변화가 더 다양함
→ DCGAN보다 반복 패턴이 줄어듦

문제
→ 일부 Apple의 형태가 찌그러짐
→ 세부 질감은 여전히 부족함
```

따라서 이번 `15 Epoch` 실험에서는

> **WGAN-GP가 DCGAN보다 다양한 Apple을 생성하는 모습은 확인할 수 있었습니다.**

하지만

> **WGAN-GP가 모든 이미지에서 더 자연스럽고 더 좋은 Apple을 만들었다고 말할 수는 없습니다.**

일부 WGAN-GP 결과에는 형태 왜곡이 있기 때문입니다.

---

## 1일차 VAE와도 연결해서 생각합니다

1일차 VAE에서는 Apple의 큰 특징은 나타났지만  
Random Sample의 윤곽과 세부 표현이 흐릿했습니다.

이번 결과까지 연결하면 다음처럼 이해할 수 있습니다.

```
VAE
→ Apple의 큰 특징 학습
→ Random Sample이 흐릿함

DCGAN
→ Apple 형태가 더 분명하게 나타남
→ 하지만 비슷한 이미지가 많이 반복됨

WGAN-GP
→ 이미지 간 다양성이 더 나타남
→ 하지만 일부 형태가 불안정함
```

즉 생성 방법이 바뀌면  
**문제가 사라지는 것이 아니라 실패 형태도 달라질 수 있습니다.**

---

## 이번 비교에서 얻은 결론

```
더 선명하다
        ≠
더 좋은 합성 학습데이터

더 다양하다
        ≠
모든 이미지가 정상이다
```

좋은 합성데이터라면

```
Apple다운 형태
  +
자연스러운 색과 질감
  +
충분한 다양성
  +
심한 Artifact 없음
```

을 함께 만족해야 합니다.

> **이번 실제 결과에서는 WGAN-GP가 DCGAN보다 다양성 측면에서는 좋아 보이지만,  
> 두 모델 모두 아직 Real Apple을 대체할 정도의 고품질 합성데이터라고 판단하기는 어렵습니다.**

---

# PART 4. Pretrained Diffusion으로 Apple 이미지를 생성합니다

---

# 19. Diffusion은 오늘 처음부터 학습하지 않습니다

오늘은 대규모 Diffusion Model을 새로 학습하지 않고 **준비된 Pretrained Model**을 사용합니다.

![[Pasted image 20261008171551.png]]

```text
Noise
        ↓
Prompt 조건
        ↓
Denoising
        ↓
새로운 이미지
```

직접 비교할 조건은 `Prompt`, `Seed`, `Inference Steps`, `Guidance`입니다. 이 값들의 복잡한 수학적 의미보다 **어떤 조건을 바꾸었을 때 결과가 어떻게 달라지는지**에 집중합니다.

---

# 20. Prompt와 Seed의 역할을 구분합니다

- **Prompt**  
    → 모델에게 **어떤 이미지를 만들지 글로 설명하는 조건**입니다.  
    → 예: `"a red apple on a white background"`  
    → 쉽게 말해 **“무엇을 그릴지 알려주는 문장”**입니다.
    
- **Seed**  
    → 이미지 생성을 시작할 때 사용하는 **Random Noise의 시작값을 고정하는 숫자**입니다.  
    → 같은 Prompt와 같은 Seed를 사용하면 **같은 Random 시작점에서 다시 생성**할 수 있습니다.  
    → 쉽게 말해 **“랜덤의 시작 위치를 고정하는 번호”**입니다.

```text
Prompt
→ 어떤 이미지를 만들지 설명

Seed
→ Random Noise의 시작값
→ 같은 주요 조건을 다시 재현할 때 사용
```

오늘은 두 실험을 분리합니다.

```text
실험 A
Prompt 고정
Seed 변경

실험 B
Seed 고정
Prompt 변경
```

---

# 21. Diffusion 생성 코드를 작성합니다

Diffusion은 앞의 DCGAN·WGAN-GP와 출발점이 다릅니다.

이 코드는 Apple Train으로 Diffusion을 새로 학습하지 않습니다.
**이미 학습된 Pretrained Diffusion Model**을 불러와 Prompt와 Seed를 입력하여 이미지를 생성합니다.

**파일: `scripts/04_diffusion_generate.py`**

```python
from __future__ import annotations

import argparse
import csv
import os
import sys
from pathlib import Path

import torch

from diffusers import (
    AutoPipelineForText2Image,
)


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import day02_paths


IMAGE_WIDTH = 512
IMAGE_HEIGHT = 512


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--prompt",
        required=True,
    )

    parser.add_argument(
        "--seed",
        type=int,
        default=42,
    )

    parser.add_argument(
        "--steps",
        type=int,
        default=4,
    )

    parser.add_argument(
        "--guidance",
        type=float,
        default=0.0,
    )

    parser.add_argument(
        "--count",
        type=int,
        default=1,
    )

    parser.add_argument(
        "--tag",
        default="sample",
    )

    return parser.parse_args()


def load_pipeline(
    model_id,
    device,
):
    dtype = (
        torch.float16
        if device == "cuda"
        else torch.float32
    )

    pipeline = (
        AutoPipelineForText2Image
        .from_pretrained(
            model_id,
            torch_dtype=dtype,
        )
    )

    return pipeline.to(device)


def main() -> None:
    args = parse_args()

    if args.steps < 1:
        raise ValueError(
            "--steps는 1 이상이어야 합니다."
        )

    if args.count < 1:
        raise ValueError(
            "--count는 1 이상이어야 합니다."
        )

    model_id = os.environ.get(
        "DIFFUSION_MODEL_ID"
    )

    if not model_id:
        raise RuntimeError(
            "DIFFUSION_MODEL_ID "
            "환경변수를 먼저 설정하세요."
        )

    paths = day02_paths(args.dataset)

    device = (
        "cuda"
        if torch.cuda.is_available()
        else "cpu"
    )

    print("Device :", device)
    print("Model  :", model_id)

    if device == "cpu":
        print(
            "[WARNING] CPU에서는 "
            "Diffusion 생성이 오래 걸릴 수 있습니다."
        )

    pipeline = load_pipeline(
        model_id,
        device,
    )

    metadata_path = (
        paths["report"]
        / "diffusion_metadata.csv"
    )

    rows = []

    for index in range(args.count):
        current_seed = (
            args.seed + index
        )

        generator = (
            torch.Generator(
                device=device
            )
            .manual_seed(
                current_seed
            )
        )

        result = pipeline(
            prompt=args.prompt,
            num_inference_steps=(
                args.steps
            ),
            guidance_scale=(
                args.guidance
            ),
            width=IMAGE_WIDTH,
            height=IMAGE_HEIGHT,
            generator=generator,
        )

        image = result.images[0]

        file_name = (
            f"{args.tag}"
            f"_seed{current_seed}"
            f"_step{args.steps}.png"
        )

        output_path = (
            paths["diffusion"]
            / file_name
        )

        image.save(output_path)

        rows.append(
            {
                "file": file_name,
                "method": "diffusion",
                "dataset": args.dataset,
                "model": model_id,
                "prompt": args.prompt,
                "seed": current_seed,
                "steps": args.steps,
                "guidance": args.guidance,
                "width": IMAGE_WIDTH,
                "height": IMAGE_HEIGHT,
                "approved": "pending",
            }
        )

        print(
            "Saved:",
            output_path,
        )

    fieldnames = [
        "file",
        "method",
        "dataset",
        "model",
        "prompt",
        "seed",
        "steps",
        "guidance",
        "width",
        "height",
        "approved",
    ]

    existing_rows = []

    if (
        metadata_path.exists()
        and metadata_path.stat().st_size > 0
    ):
        with metadata_path.open(
            "r",
            newline="",
            encoding="utf-8-sig",
        ) as file:
            existing_rows = list(
                csv.DictReader(file)
            )

    new_files = {
        row["file"]
        for row in rows
    }

    existing_rows = [
        row
        for row in existing_rows
        if row.get("file") not in new_files
    ]

    with metadata_path.open(
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
            existing_rows + rows
        )

    print(
        "Metadata:",
        metadata_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드

```text
실행할 조건을 입력받는다

Prompt
Seed
Steps
Guidance
생성할 이미지 수

        ↓

사용할 Pretrained Diffusion Model을 확인한다
        ↓
GPU가 있으면 GPU를 사용한다
        ↓
Pretrained Model을 불러온다

        ↓

[이미지 생성 반복]

Seed로 Random Noise의 시작점을 만든다
        ↓
Prompt를 모델에 전달한다
        ↓
설정한 Steps만큼 Denoising한다
        ↓
Synthetic Image를 만든다
        ↓
이미지를 파일로 저장한다

        ↓

사용한

Model
Prompt
Seed
Steps
Guidance

조건을 Metadata에 기록한다

        ↓

diffusion_metadata.csv 저장
```

## 코드에서 꼭 이해할 핵심

### ① 어떤 모델을 사용할 것인가?

```python
model_id = os.environ.get(
    "DIFFUSION_MODEL_ID"
)

pipeline = (
    AutoPipelineForText2Image
    .from_pretrained(
        model_id,
        torch_dtype=dtype,
    )
)
```

여기서는 **이미 학습이 끝난 Diffusion Model을 불러옵니다.**

```
Apple Train으로 새로 학습
X

Pretrained Model 사용
O
```

생성 품질의 기본 능력 자체를 바꾸려면  
결국 **어떤 Pretrained Model을 사용하는가**가 중요한 요소입니다.

---

### ② 생성 결과를 가장 직접적으로 바꾸는 부분

```python
result = pipeline(
    prompt=args.prompt,
    num_inference_steps=args.steps,
    guidance_scale=args.guidance,
    width=IMAGE_WIDTH,
    height=IMAGE_HEIGHT,
    generator=generator,
)
```

이 부분이 실제 이미지 생성의 핵심입니다.

```
prompt
→ 무엇을 만들 것인가?

seed
→ 어떤 Random 시작점에서 시작할 것인가?

steps
→ 몇 단계에 걸쳐 이미지를 생성할 것인가?

guidance
→ Prompt 조건을 얼마나 반영할 것인가?
```

---

## 결과가 좋지 않을 때 어디를 바꿔야 하나요?

### 1. Apple의 모습이 원하는 것과 다르다 → `Prompt`

```bash
--prompt "a single red apple on a clean white background ..."
```

가장 먼저 확인할 부분입니다.

```
사과 모양
배경
조명
위치
사진 스타일
```

등 원하는 조건을 Prompt에 더 정확하게 작성합니다.

---

### 2. 비슷한 조건에서 다른 Apple을 만들고 싶다 → `Seed`

```bash
--seed 42
```

Seed는 **품질을 높이는 값이라기보다 Random 시작점을 바꾸는 값**입니다.

```
같은 Prompt + 다른 Seed
        ↓
다른 형태의 Apple 생성
```

따라서 다양성을 확인할 때 바꿉니다.

---

### 3. 생성 과정의 설정을 바꾸고 싶다 → `Steps`, `Guidance`

```bash
--steps 4
--guidance 0.0
```

현재 사용하는 `SD-Turbo`에서는 `4 Steps`, `Guidance 0.0`을 기준으로 실험합니다.

```
Steps
→ Denoising 반복 조건

Guidance
→ Prompt 반영 조건
```

값을 크게 한다고 무조건 이미지가 좋아지는 것은 아닙니다.

특히 **SD-Turbo는 적은 Steps로 빠르게 생성하도록 만들어진 모델**이므로  
처음에는 현재 기준값을 유지하고 결과를 비교합니다.

---

## 꼭 기억할 수정 포인트

```
이미지 내용이 마음에 들지 않는다
→ Prompt 확인

다른 형태를 생성하고 싶다
→ Seed 변경

생성 과정의 조건을 실험하고 싶다
→ Steps / Guidance 변경

모델 자체의 생성 능력을 바꾸고 싶다
→ Pretrained Model 변경
```

반면 다음 값은 혼동하지 않습니다.

```
--count
→ 생성하는 이미지 수
→ 품질을 높이는 값이 아님

--dataset apple
→ Apple로 모델을 학습하는 값이 아님
→ 결과 저장 위치와 비교 대상을 구분하는 값
```

> **이 코드에서 가장 먼저 볼 것은 `Prompt → Seed → Steps / Guidance`입니다.**
> 
> 이미지 품질 자체의 한계가 모델에서 오는 경우에는 그다음으로 **Pretrained Model**을 검토합니다.

---

# 22. Diffusion Smoke Test — 기준 Apple Prompt로 한 장 생성합니다

앞에서 사용할 Pretrained Diffusion Model을 다음과 같이 지정했습니다.

```bash
export DIFFUSION_MODEL_ID="stabilityai/sd-turbo"
```

현재 설정을 확인합니다.

```bash
echo "$DIFFUSION_MODEL_ID"
```

다음처럼 출력되면 정상입니다.

```
stabilityai/sd-turbo
```

이 값은

> **Hugging Face에서 `stabilityai/sd-turbo`라는  
> 사전학습 Diffusion Model을 불러오겠다**

는 의미입니다.

이번 실습에서는 Apple Train으로 Diffusion을 새로 학습하지 않습니다.

```
Pretrained SD-Turbo
        +
Apple Prompt
        +
Seed
        ↓
Synthetic Apple 생성
```

### 먼저 한 장만 생성합니다

본 실험 전에 한 장만 생성하여 다음 과정이 정상적으로 연결되는지 확인합니다.

```
Pretrained Model Load
        ↓
Prompt + Seed 입력
        ↓
이미지 생성
        ↓
파일 저장
        ↓
Metadata 저장
```

실행합니다. 

여기에서 seed는 42이고 
prompt는 "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" 문장 입니다.

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 42 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag base
```

이미지와 `reports/apple/diffusion_metadata.csv`가 정상적으로 생성되면 Diffusion 실행환경이 준비된 것입니다.

![[Pasted image 20261008173111.png]]

---

# 23. Prompt를 고정하고 Seed만 100으로 바꿉니다

이번 실험에서는 Prompt는 그대로 유지하고,  
Seed만 100 → 101 → 102 → 103으로 바꾸어 결과가 어떻게 달라지는지 확인합니다.

실행 명령은 다음과 같습니다.

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 100 \
  --steps 4 \
  --guidance 0.0 \
  --count 4 \
  --tag same_prompt
```

이 명령은 `count=4` 이므로  
다음 네 장의 이미지를 생성합니다.

```
same_prompt_seed100_step4.png
same_prompt_seed101_step4.png
same_prompt_seed102_step4.png
same_prompt_seed103_step4.png
```

## 실제 결과를 기준으로 해석합니다

이번 결과에서는 **Prompt가 같기 때문에 공통적으로 다음 특징이 유지**되었습니다.

```
빨간 사과
  +
밝은 배경
  +
제품 사진 같은 느낌
```

하지만 Seed가 달라지면서 세부 결과는 분명히 달라졌습니다.

![[Pasted image 20261012103548.png]]

---
### Seed 100

Apple 한 개가 생성되었지만 꼭지 주변과 사과 중심부의 구조가 부자연스럽습니다.

```
여러 개의 꼭지처럼 보이는 형태
    +
비정상적인 중심 무늬
```

가 나타납니다.

→ Artifact가 있는 결과

---
### Seed 101

Apple이 한 개가 아니라 두 개처럼 생성되었습니다.

```
Prompt
"a single red apple"

하지만 결과
→ Apple이 2개 나타남
```

따라서 Prompt의 객체 개수 조건을 제대로 지키지 못했습니다.

→ Reject 후보

---
### Seed 102

Apple 한 개가 생성되었고 네 결과 중 비교적 자연스러운 형태를 보입니다.

```
객체 수
→ 정상

Apple 형태
→ 비교적 자연스러움

큰 구조적 Artifact
→ 상대적으로 적음
```

→ 현재 결과 중 사용 후보에 가장 가까움

---
### Seed 103

Apple 한 개가 생성되었고  
전체 형태도 비교적 자연스럽습니다.

하지만 표면의 흰 점과 질감이 매우 강하기 때문에  
Real Apple 데이터와 비교하여 추가 확인해야 합니다.

→ Needs Review 후보

---
이번 실험의 핵심 결론은 다음과 같습니다.

같은 Prompt를 사용하더라도 Seed가 달라지면 서로 다른 Apple이 생성됩니다. 
그러나 일부 Seed에서는 사과 개수 오류나 여러 개의 꼭지처럼 보이는 구조적 Artifact가 발생했습니다. 따라서 Diffusion으로 생성된 이미지는 선명해 보여도 반드시 Human QA가 필요합니다.

---

# 24. Seed를 102로 고정하고 Prompt의 조명만 바꿉니다

23번에서는 같은 Prompt에서 Seed를 `100~103`으로 바꾸어
생성 결과가 어떻게 달라지는지 확인했습니다.

그 결과 이번 실험에서는 `Seed 102`의 Apple이
비교적 한 개의 정상적인 형태로 생성되었습니다.

따라서 이번에는 **Seed 102를 비교 기준으로 고정**하고,
Prompt의 조명 표현만 바꾸어 결과가 어떻게 달라지는지 확인합니다.

> Seed 102가 항상 좋은 Seed라는 뜻은 아닙니다.
> 이번 실행 결과에서 조명 변화를 비교하기 위한
> **기준 Seed로 선택한 것**입니다.

```text
23번

같은 Prompt
+
Seed 100 / 101 / 102 / 103
        ↓
생성 결과 비교
        ↓
Seed 102를 비교 기준으로 선택


24번

Seed 102 고정
        ↓
Prompt의 조명 표현만 변경
        ↓
밝기 · 반사 · 그림자가
어떻게 달라지는지 비교
```

한글 Prompt도 입력 자체는 가능하지만 지금 사용하는 `stabilityai/sd-turbo`에서는 수업 실험의 **재현성과 Prompt 반영의 안정성**을 생각하면 영어 Prompt를 사용합니다.

## 기준 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 102 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag neutral_light
```

```
a single red apple
→ 빨간 사과 한 개

on a clean white background
→ 깨끗한 흰색 배경 위에

centered object
→ 사과를 화면 중앙에 배치하고

realistic product photography
→ 실제 제품 사진처럼 사실적으로 표현하며

neutral lighting
→ 특별히 밝거나 어둡지 않은 중립적인 조명을 사용한다
```

깨끗한 흰색 배경 중앙에 빨간 사과 한 개를 놓고, 중립적인 조명으로 실제 제품 사진처럼 사실적으로 표현해 주세요.

---
## 어두운 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, dim lighting" \
  --seed 102 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag dim_light
```

```
a single red apple
→ 빨간 사과 한 개

on a clean white background
→ 깨끗한 흰색 배경 위에

centered object
→ 화면 중앙에 배치

realistic product photography
→ 실제 제품 사진처럼 사실적으로 표현

dim lighting
→ 어두운 조명을 사용
```

깨끗한 흰색 배경 중앙에 빨간 사과 한 개를 놓고, 어두운 조명에서 실제 제품 사진처럼 사실적으로 표현해 주세요.

---
## 강한 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset apple \
  --prompt "a single red apple on a clean white background, centered object, realistic product photography, strong overhead lighting and reflections" \
  --seed 102 \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag strong_light
```

```
a single red apple
→ 빨간 사과 한 개

on a clean white background
→ 깨끗한 흰색 배경 위에

centered object
→ 화면 중앙에 배치

realistic product photography
→ 실제 제품 사진처럼 사실적으로 표현

strong overhead lighting
→ 위쪽에서 강한 조명을 비추고

and reflections
→ 표면에 강한 반사광이 나타나게 한다
```

깨끗한 흰색 배경 중앙에 빨간 사과 한 개를 놓고, 위쪽에서 강한 조명을 비추어 표면 반사가 나타나는 실제 제품 사진처럼 표현해 주세요.

---
세 Prompt의 차이는 딱 이것입니다

```
공통 조건

빨간 사과 한 개
+
흰 배경
+
중앙 배치
+
사실적인 제품 사진

        ↓

바꾸는 조건만 다름

neutral lighting
→ 중립적인 조명

dim lighting
→ 어두운 조명

strong overhead lighting and reflections
→ 위에서 강한 조명 + 강한 반사
```

---
이제 세 결과를 비교합니다.

```
Seed = 102
→ 모두 동일

Apple을 설명하는 기본 Prompt
→ 모두 동일

바뀐 부분
→ 조명 표현만 다름


neutral lighting
        VS
dim lighting
        VS
strong overhead lighting
and reflections
```

![[Pasted image 20261012110034.png]]

## 같은 Seed인데 왜 꼭지 모양이 달라졌을까요?

세 이미지 모두 `Seed = 102`를 사용했습니다.

하지만 Seed가 같다고 해서
사과의 모양과 꼭지까지 완전히 똑같아지는 것은 아닙니다.

```text
Seed = 102
→ Random Noise의 시작점만 동일

Prompt
→ 생성 과정의 방향을 결정
```

이번 실험에서는 Seed는 같지만 Prompt의 조명 표현을 바꾸었습니다.

```
neutral lighting
dim lighting
strong overhead lighting and reflections
```

Diffusion Model은 같은 Random Noise에서 시작하더라도  
Prompt가 달라지면 **Denoising 과정의 방향도 달라집니다.**

```
같은 Seed
        ↓
같은 Random 시작점

하지만

다른 Prompt
        ↓
다른 Denoising 과정
        ↓
최종 이미지의 세부 구조도 달라질 수 있음
```

즉 Prompt에서 조명만 바꾸었다고 해도  
모델이 실제로는 조명만 독립적으로 변경하는 것이 아닙니다.

사과의

```
꼭지
형태
표면 무늬
반사 위치
그림자
윤곽
```

도 함께 달라질 수 있습니다.

---

## 실제 결과를 보면

### Neutral Lighting

사과의 전체 형태는 비교적 자연스럽고  
꼭지도 비교적 단순하게 표현되었습니다.

### Dim Lighting

전체 형태는 비슷하지만  
꼭지 주변 구조가 조금 달라졌습니다.

### Strong Lighting

조명 변화뿐 아니라  
**꼭지가 두 개처럼 보이는 비정상적인 구조**가 나타났습니다.

이 부분은 정상적인 Apple의 특징이 아니라  
생성 과정에서 발생한 **Artifact**로 보는 것이 적절합니다.

```
Prompt 변경
        ↓
조명 변화 시도

하지만 동시에

꼭지 구조도 변화
        ↓
비정상적인 꼭지 발생
        ↓
Human QA 필요
```

---

## 이번 실험에서 꼭 이해해야 할 점

```
같은 Seed
   ≠
같은 사과 모양

같은 Seed
→ 같은 Random 시작점

다른 Prompt
→ Denoising 경로가 달라짐
→ 최종 형태도 달라질 수 있음
```

따라서 이번 결과는 오히려 중요한 실험 결과입니다.

> Seed를 고정하고 Prompt의 조명 표현만 바꾸었지만,  
> 모델은 조명뿐 아니라 사과의 세부 형태까지 함께 바꾸었습니다.

특히 Strong Lighting 결과에서는  
꼭지가 여러 개처럼 생성되었으므로  
Prompt 조건을 바꿀 때 예상하지 않은 구조적 Artifact가 생길 수 있다는 것을 확인할 수 있습니다.

---

## 이번 결과의 결론

```
Seed 102 고정
→ Random 시작점 동일

Prompt 변경
→ 조명 조건 변경

실제 결과
→ 밝기와 반사뿐 아니라
  꼭지와 세부 형태도 함께 변함

강한 조명 결과
→ 꼭지 구조 Artifact 발생
```

> **Diffusion은 조명·형태·질감을 완전히 분리해서 제어하는 모델이 아닙니다.**
> 
> 그래서 Prompt에서 조명 한 단어만 바꾸어도  
> 예상하지 않은 형태 변화가 생길 수 있으며,  
> 생성 이미지는 반드시 Human QA로 확인해야 합니다.

**==이번 실험에서는 구조적 Artifact가 발생했으므로, 생성 결과를 자동 승인하지 않고 
Human QA를 거쳐 정상 이미지만 후보로 사용해야 합니다.`

---

# 25. Diffusion Metadata를 실제 결과로 확인합니다

지금까지 생성한 Diffusion 이미지의 조건이
`diffusion_metadata.csv`에 기록되어 있는지 확인합니다.

```bash
cat reports/apple/diffusion_metadata.csv
```

이번 결과에는 다음 실험 기록이 남아 있습니다.

```
Smoke Test
→ Seed 42
→ base_seed42_step4.png

Prompt 고정 + Seed 변경
→ Seed 100
→ Seed 101
→ Seed 102
→ Seed 103

Seed 고정 + Prompt 변경
→ 처음에는 Seed 200으로 실험
→ 이후 Seed 102를 비교 기준으로 다시 실험
```

즉 Metadata를 보면  
**어떤 이미지가 어떤 조건에서 만들어졌는지 다시 추적할 수 있습니다.**

| 열                  | 실제 의미                                            |
| ------------------ | ------------------------------------------------ |
| `file`             | 생성된 이미지 파일명                                      |
| `method`           | 생성 방법. 현재는 `diffusion`                           |
| `dataset`          | 결과를 구분할 데이터 이름. 현재는 `apple`                      |
| `model`            | 사용한 Pretrained Model. 현재는 `stabilityai/sd-turbo` |
| `prompt`           | 이미지 생성에 사용한 문장                                   |
| `seed`             | Random Noise의 시작값                                |
| `steps`            | Denoising 반복 횟수                                  |
| `guidance`         | Prompt 반영 관련 설정                                  |
| `width` / `height` | 생성 이미지 크기. 현재 `512 × 512`                        |
| `approved`         | Human QA 전 상태. 현재 모두 `pending`                   |
## Diffusion Metadata를 확인합니다

`diffusion_metadata.csv`에는
각 이미지가 **어떤 조건으로 생성되었는지** 기록되어 있습니다.

예:

```text
file     → neutral_light_seed102_step4.png
model    → stabilityai/sd-turbo
seed     → 102
steps    → 4
guidance → 0.0
```

즉 나중에 결과가 이상할 때

```
어떤 모델을 썼는가?
어떤 Prompt를 썼는가?
어떤 Seed를 썼는가?
어떤 생성 조건이었는가?
```

를 다시 확인할 수 있습니다.

## `approved = pending`의 의미

```
pending
→ 이미지 생성은 완료
→ 아직 사용 여부는 결정하지 않음
```

이번 실험에서는

```
사과가 2개 생성됨
꼭지가 여러 개 생성됨
형태가 비정상적임
```

같은 결과도 있었기 때문에  
생성 직후 바로 `approved`로 판단하지 않습니다.

```
생성
→ QA
→ 사용 여부 결정
```

> Metadata는 생성 이력을 남기고,  
> QA 후 어떤 이미지를 사용할지 판단하기 위한 기록입니다.

---

# PART 5. Real Apple과 Synthetic Apple의 차이를 검수합니다

앞에서는 DCGAN, WGAN-GP, Diffusion으로 새로운 Apple 이미지를 만들었습니다.

하지만 이미지를 만드는 데 성공했다고 해서
바로 학습데이터로 사용할 수 있는 것은 아닙니다.

이제부터는 생성된 이미지가 **실제 Apple과 얼마나 비슷하고,
어떤 부분에서 다른지 확인하는 단계**로 넘어갑니다.

---

# 26. Domain Gap을 이해합니다

`Domain Gap`은 쉽게 말해

> **실제 데이터와 합성 데이터 사이의 차이**

입니다.

예를 들어 Synthetic Apple이 선명하게 보여도

```text
사과 형태가 이상하거나
색이 실제 데이터와 다르거나
배경·조명이 다르거나
꼭지가 여러 개 생기는 Artifact가 있다면
```

Real Apple과 같은 학습데이터라고 볼 수 없습니다.

이번 실습에서는 복잡한 수치로 계산하기보다  
**형태·색·배경·조명·Artifact·다양성**을 실제 이미지로 비교합니다.

---

# 27. Real과 Synthetic 결과를 검수할 CSV를 만듭니다

지금까지 다음 이미지가 준비되었습니다.

```
Real Apple
DCGAN Apple
WGAN-GP Apple
Diffusion Apple
```

이제 이 이미지들을 **같은 기준으로 비교하고 기록할 수 있는 검수표**가 필요합니다.

`scripts/05_domain_gap_review.py`는  
Domain Gap을 자동으로 판정하는 AI 모델이 아닙니다.

이 코드는

- Real Apple 대표 이미지
- DCGAN Final Image
- WGAN-GP Final Image
- Diffusion Image

를 모아 `domain_gap_review.csv`에 정리하고,

```
형태는 자연스러운가?
색은 실제 Apple과 비슷한가?
배경과 조명은 적절한가?
Artifact는 있는가?
다양성은 충분한가?
최종적으로 사용할 수 있는가?
```

를 사람이 기록할 수 있도록 **빈 검수 항목을 만들어 주는 코드**입니다.

즉,

> **05번 코드는 판정하는 코드가 아니라,  
> 사람이 Real과 Synthetic 이미지를 같은 기준으로 검수할 수 있도록  
> 검수표를 준비하는 코드입니다.**

자동으로 파일 이상을 검사하는 과정은  
다음 `06_synthetic_qa.py`에서 따로 진행합니다.

**파일: `scripts/05_domain_gap_review.py`**

```python
from __future__ import annotations

import argparse
import random
import sys
from pathlib import Path

import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
    list_images,
)


SEED = 42


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    parser.add_argument(
        "--overwrite",
        action="store_true",
        help=(
            "기존 domain_gap_review.csv를 "
            "의도적으로 다시 만들 때 사용합니다."
        ),
    )

    return parser.parse_args()


def add_rows(
    rows,
    files,
    method,
):
    for path in files:
        rows.append(
            {
                "file": str(path),
                "method": method,
                "realistic_shape": "",
                "realistic_color": "",
                "realistic_background": "",
                "realistic_light": "",
                "artifact": "",
                "diversity": "",
                "domain_gap_level": "",
                "decision": "",
                "note": "",
            }
        )


def main() -> None:
    args = parse_args()
    paths = day02_paths(
        args.dataset
    )

    rows = []

    train_files = list_images(
        paths["train"]
    )

    if not train_files:
        raise RuntimeError(
            "Real Train 이미지가 없습니다."
        )

    rng = random.Random(SEED)

    real_count = min(
        10,
        len(train_files),
    )

    real_files = rng.sample(
        train_files,
        real_count,
    )

    dcgan_files = list_images(
        paths["dcgan_final"]
    )

    wgan_files = list_images(
        paths["wgan_final"]
    )

    diffusion_files = list_images(
        paths["diffusion"]
    )

    if not dcgan_files:
        raise RuntimeError(
            "DCGAN Final 이미지가 없습니다."
        )

    if not wgan_files:
        raise RuntimeError(
            "WGAN-GP Final 이미지가 없습니다."
        )

    if not diffusion_files:
        raise RuntimeError(
            "Diffusion 이미지가 없습니다."
        )

    add_rows(
        rows,
        real_files,
        "real_reference",
    )

    add_rows(
        rows,
        dcgan_files,
        "dcgan",
    )

    add_rows(
        rows,
        wgan_files,
        "wgan_gp",
    )

    add_rows(
        rows,
        diffusion_files,
        "diffusion",
    )

    output_path = (
        paths["report"]
        / "domain_gap_review.csv"
    )

    if (
        output_path.exists()
        and not args.overwrite
    ):
        raise RuntimeError(
            "domain_gap_review.csv가 이미 있습니다. "
            "작성한 검수 내용을 보호하기 위해 덮어쓰지 않습니다. "
            "정말 새로 만들려면 --overwrite를 사용하세요."
        )

    pd.DataFrame(
        rows
    ).to_csv(
        output_path,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        "Saved:",
        output_path,
    )

    print(
        "Real Reference:",
        len(real_files),
    )

    print(
        "DCGAN:",
        len(dcgan_files),
    )

    print(
        "WGAN-GP:",
        len(wgan_files),
    )

    print(
        "Diffusion:",
        len(diffusion_files),
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
dataset이 apple인지 banana인지 입력받는다
        ↓
해당 데이터의 경로를 찾는다
        ↓
Real Train 이미지 목록을 가져온다
        ↓
Seed 42를 사용하여
Real 이미지 중 최대 10장을 대표 이미지로 선택한다
        ↓
DCGAN의 final 이미지를 가져온다
        ↓
WGAN-GP의 final 이미지를 가져온다
        ↓
Diffusion 생성 이미지를 가져온다
        ↓
필요한 이미지가 하나라도 없으면
오류를 발생시키고 실행을 중단한다
        ↓
각 이미지에

file
method
형태
색
배경
조명
Artifact
다양성
Domain Gap
최종 판단
메모

항목을 만든다
        ↓
사람이 작성할 검수 항목은
빈 값으로 준비한다
        ↓
domain_gap_review.csv가
이미 존재하는지 확인한다
        ↓
기존 파일이 있으면
사람이 작성한 내용을 보호하기 위해
기본적으로 덮어쓰지 않는다
        ↓
--overwrite를 사용한 경우에만
새 검수표를 다시 만든다
        ↓
domain_gap_review.csv로 저장한다
        ↓
Real / DCGAN / WGAN-GP / Diffusion
이미지가 각각 몇 장 들어갔는지 출력한다
```

### 핵심만 정리하면

이 코드는 이미지를 평가하는 코드가 아닙니다.

```
Real 대표 이미지
   +
DCGAN Final
   +
WGAN-GP Final
   +
Diffusion 결과

   ↓

한 곳에 모음

   ↓

사람이 검수할 CSV 틀 생성
```

특히 DCGAN과 WGAN-GP에서는 `epochs` 폴더가 아니라 
최종 생성 결과가 들어 있는 `final` 폴더를 사용합니다.

> 즉, `05_domain_gap_review.py`는 Real과 Synthetic 이미지를 같은 기준으로 비교할 수 있도록 검수표를 준비하는 코드입니다.

---
## Domain Gap 검수표를 생성합니다

DCGAN·WGAN-GP·Diffusion 생성이 모두 끝났으므로
검수표를 만듭니다.

```bash
python scripts/05_domain_gap_review.py \
  --dataset apple
```

실제 실행 결과:

```text
Saved: reports/apple/domain_gap_review.csv

Real Reference: 10
DCGAN: 16
WGAN-GP: 16
Diffusion: 7
```

즉 이번 검수표에는 총 49장의 이미지가 들어갔습니다.

```
Real Apple        10장
DCGAN             16장
WGAN-GP           16장
Diffusion           7장
----------------------
총                  49장
```

CSV를 열어 보면 이미지 경로와 생성 방법은 이미 기록되어 있고,  
검수 항목은 비어 있습니다.

예:

```
file
→ 어떤 이미지인지

method
→ real_reference / dcgan / wgan_gp / diffusion

realistic_shape
realistic_color
realistic_background
realistic_light
artifact
diversity
domain_gap_level
decision
note
→ 사람이 실제 이미지를 보고 작성
```

### 왜 값이 비어 있나요?

예를 들어 다음과 같이 보입니다.

```
sample_01.png,dcgan,,,,,,,,,
```

오류가 아닙니다.

`05_domain_gap_review.py`는 이미지를 자동 평가하는 코드가 아니라  
**사람이 검수할 수 있는 표를 미리 만드는 코드**이기 때문입니다.

따라서 다음 단계에서 실제 이미지를 보면서 값을 작성합니다.

결과:

```text
reports/apple/domain_gap_review.csv
```

---

# 28. Domain Gap을 대표 이미지로 검수합니다

49장을 모두 사람이 작성하지 않습니다.

이번 실습에서는 각 방법의 **대표 이미지 몇 장만 직접 확인**하고,
전체 이미지는 다음 Automatic QA에서 프로그램으로 검사합니다.

```text
Real Apple
→ 대표 이미지 3~4장 확인

DCGAN
→ 대표 이미지 4장 확인

WGAN-GP
→ 대표 이미지 4장 확인

Diffusion
→ 생성 수가 적으므로 주요 결과 확인
```

사람이 확인할 핵심 항목은 다음 정도면 충분합니다.

```
형태
→ 실제 Apple처럼 자연스러운가?

색
→ Real Apple과 비슷한가?

Artifact
→ 찌그러짐, 여러 개의 꼭지,
  객체 수 오류 등이 있는가?

다양성
→ 비슷한 결과만 반복되는가?

최종 판단
→ approved / needs_review / rejected
```

## 이번 실제 결과를 정리합니다

현재까지 생성된 결과를 Real Apple과 비교하면 다음과 같습니다.

|방법|실제 관찰 결과|판단|
|---|---|---|
|Real Apple|실제 촬영 이미지로 형태·색·배경이 안정적|비교 기준|
|DCGAN|붉은 Apple 특징은 나타났지만 이미지가 서로 매우 비슷하고 표면이 거칠고 흐림|`needs_review`|
|WGAN-GP|DCGAN보다 형태가 다양하지만 일부 Apple이 찌그러지거나 각진 형태로 생성됨|`needs_review`|
|Diffusion|매우 선명하지만 일부 이미지에서 사과가 2개 생성되거나 꼭지가 여러 개 생기는 Artifact 발생|이미지별 판단 필요|

Diffusion에서는 실제로 다음과 같은 실패 사례가 확인되었습니다.

```
same_prompt_seed101_step4.png
→ "single apple" Prompt인데 Apple이 2개 생성됨
→ rejected

strong_light_seed102_step4.png
→ 꼭지가 여러 개처럼 생성됨
→ rejected

same_prompt_seed102_step4.png
→ 한 개의 Apple이 비교적 자연스럽게 생성됨
→ 사용 후보 / 추가 확인
```

이번 결과의 핵심은 다음과 같습니다.

```
DCGAN
→ 형태는 비슷하지만 다양성 부족

WGAN-GP
→ 다양성은 증가했지만 일부 형태 왜곡

Diffusion
→ 가장 선명하지만 구조적 Artifact 발생 가능
```

따라서

> **선명한 이미지가 가장 좋은 합성데이터라는 뜻은 아닙니다.**  
> 형태가 정상인지, Artifact가 없는지, Real 데이터와 충분히 비슷한지를 함께 확인해야 합니다.

---

## 전체 이미지는 다음 Automatic QA에서 확인합니다

다음 `06_synthetic_qa.py`에서는 모든 생성 이미지에 대해

```
파일을 정상적으로 읽을 수 있는가?
이미지 크기가 정상인가?
거의 균일한 이미지인가?
완전히 같은 이미지가 중복되어 있는가?
```

를 자동으로 확인합니다.

```
Human Review
→ 형태·객체 의미·Artifact 확인

Automatic QA
→ 파일·크기·밝기·중복 확인
```

> 사람과 프로그램이 서로 다른 항목을 나누어 검사합니다.
---

# 29. 생성 이미지의 Automatic QA를 수행하는 코드를 작성합니다

28번에서는 Real Apple과 Synthetic Apple을 직접 보면서
형태나 Artifact 같은 **이미지의 의미와 품질**을 확인했습니다.

하지만 생성 이미지가 많아지면 모든 파일을 사람이 하나씩 확인하기 어렵습니다.

그래서 이번에는 프로그램이 먼저 확인할 수 있는
**기본적인 기술적 이상을 자동으로 검사**합니다.

---

## Automatic QA는 무엇을 검사하나요?

이번 코드에서는 DCGAN, WGAN-GP, Diffusion으로 만든
전체 생성 이미지를 대상으로 다음을 확인합니다.

```text
이미지 파일을 정상적으로 읽을 수 있는가?

Width / Height가 정상적으로 존재하는가?

이미지 밝기가 지나치게 한쪽으로 몰려 있지는 않은가?

거의 한 가지 밝기만 있는
비정상적으로 균일한 이미지인가?

내용이 완전히 동일한 이미지가
중복되어 있지는 않은가?
```

## 28번과 29번의 차이

```
28번 Human Review

Apple처럼 보이는가?
형태가 자연스러운가?
꼭지가 이상하지 않은가?
Artifact가 있는가?


29번 Automatic QA

파일이 정상인가?
크기가 정상인가?
거의 단색인가?
완전히 중복된 파일인가?
```

즉,

> 사람은 이미지의 의미와 형태를 확인하고,  
> 프로그램은 반복적으로 검사할 수 있는 기술적인 문제를 자동으로 확인합니다.


**파일: `scripts/06_synthetic_qa.py`**

```python
from __future__ import annotations

import argparse
import hashlib
import random
import sys
from pathlib import Path

import cv2
import numpy as np
import pandas as pd


ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(ROOT))

from src.day02_common import (
    day02_paths,
    list_images,
)


SEED = 42


def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--dataset",
        default="apple",
        choices=["apple", "banana"],
    )

    return parser.parse_args()


def sha256_file(path: Path) -> str:
    digest = hashlib.sha256()

    with path.open("rb") as file:
        for chunk in iter(
            lambda: file.read(
                1024 * 1024
            ),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()


def collect_method_files(paths):
    return {
        "dcgan": list_images(
            paths["dcgan_final"]
        ),
        "wgan_gp": list_images(
            paths["wgan_final"]
        ),
        "diffusion": list_images(
            paths["diffusion"]
        ),
    }


def make_human_grid(
    method_files,
    output_path,
):
    selected = []
    rng = random.Random(SEED)

    for method in [
        "dcgan",
        "wgan_gp",
        "diffusion",
    ]:
        files = method_files[
            method
        ]

        sample_count = min(
            4,
            len(files),
        )

        sampled = (
            rng.sample(
                files,
                sample_count,
            )
            if sample_count > 0
            else []
        )

        for path in sampled:
            selected.append(
                (method, path)
            )

    if not selected:
        return

    tile_size = 160
    columns = 4
    header = 24

    tiles = []

    for method, path in selected:
        image = cv2.imread(
            str(path)
        )

        if image is None:
            continue

        image = cv2.resize(
            image,
            (
                tile_size,
                tile_size,
            ),
        )

        canvas = np.full(
            (
                tile_size + header,
                tile_size,
                3,
            ),
            255,
            dtype=np.uint8,
        )

        canvas[
            header:,
            :,
        ] = image

        cv2.putText(
            canvas,
            method,
            (5, 17),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.45,
            (0, 0, 0),
            1,
            cv2.LINE_AA,
        )

        tiles.append(canvas)

    if not tiles:
        return

    while len(tiles) % columns != 0:
        tiles.append(
            np.zeros_like(
                tiles[0]
            )
        )

    rows = []

    for start in range(
        0,
        len(tiles),
        columns,
    ):
        rows.append(
            np.hstack(
                tiles[
                    start:
                    start + columns
                ]
            )
        )

    grid = np.vstack(rows)

    saved = cv2.imwrite(
        str(output_path),
        grid,
    )

    if not saved:
        raise RuntimeError(
            f"QA Grid 저장 실패: {output_path}"
        )


def main() -> None:
    args = parse_args()
    paths = day02_paths(
        args.dataset
    )

    method_files = (
        collect_method_files(
            paths
        )
    )

    all_items = []

    for method, files in (
        method_files.items()
    ):
        for path in files:
            all_items.append(
                (method, path)
            )

    if not all_items:
        raise RuntimeError(
            "QA할 생성 이미지가 없습니다."
        )

    hash_groups = {}

    for method, path in all_items:
        file_hash = sha256_file(
            path
        )

        hash_groups.setdefault(
            file_hash,
            [],
        ).append(
            str(path)
        )

    duplicate_hashes = {
        file_hash
        for file_hash, names
        in hash_groups.items()
        if len(names) > 1
    }

    rows = []

    for method, path in all_items:
        image = cv2.imread(
            str(path)
        )

        readable = (
            image is not None
        )

        width = 0
        height = 0
        mean_brightness = 0.0
        std_brightness = 0.0

        if readable:
            height, width = (
                image.shape[:2]
            )

            gray = cv2.cvtColor(
                image,
                cv2.COLOR_BGR2GRAY,
            )

            mean_brightness = float(
                gray.mean()
            )

            std_brightness = float(
                gray.std()
            )

        file_hash = sha256_file(
            path
        )

        exact_duplicate = (
            file_hash
            in duplicate_hashes
        )

        near_uniform = (
            std_brightness < 2.0
            if readable
            else True
        )

        auto_pass = (
            readable
            and width > 0
            and height > 0
            and not exact_duplicate
            and not near_uniform
        )

        rows.append(
            {
                "file": str(path),
                "method": method,
                "readable": readable,
                "width": width,
                "height": height,
                "mean_brightness": round(
                    mean_brightness,
                    2,
                ),
                "std_brightness": round(
                    std_brightness,
                    2,
                ),
                "near_uniform": near_uniform,
                "exact_duplicate": (
                    exact_duplicate
                ),
                "auto_pass": auto_pass,
            }
        )

    qa_path = (
        paths["report"]
        / "synthetic_qa.csv"
    )

    pd.DataFrame(
        rows
    ).to_csv(
        qa_path,
        index=False,
        encoding="utf-8-sig",
    )

    grid_path = (
        paths["compare"]
        / "human_qa_grid.jpg"
    )

    make_human_grid(
        method_files,
        grid_path,
    )

    passed = sum(
        row["auto_pass"]
        for row in rows
    )

    print(
        "QA total:",
        len(rows),
    )

    print(
        "QA pass :",
        passed,
    )

    print(
        "QA fail :",
        len(rows) - passed,
    )

    print(
        "QA CSV  :",
        qa_path,
    )

    print(
        "QA Grid :",
        grid_path,
    )


if __name__ == "__main__":
    main()
```

### 의사코드로 읽어보기

```text
### 의사코드로 읽어보기

```text
dataset이 apple인지 banana인지 입력받는다
        ↓
DCGAN Final Image 목록을 가져온다
        ↓
WGAN-GP Final Image 목록을 가져온다
        ↓
Diffusion Image 목록을 가져온다
        ↓
검사할 이미지가 하나도 없으면
오류를 발생시키고 종료한다
        ↓
각 이미지 파일의 SHA-256 Hash를 계산한다
        ↓
Hash가 같은 파일이 여러 개 있는지 확인한다
        ↓
같으면 Exact Duplicate로 표시한다
        ↓
각 이미지를 OpenCV로 읽는다
        ↓
정상적으로 읽을 수 있는지 확인한다
        ↓
이미지의 Width와 Height를 확인한다
        ↓
Grayscale로 변환한다
        ↓
평균 밝기를 계산한다
        ↓
밝기 표준편차를 계산한다
        ↓
밝기 변화가 거의 없으면
near_uniform = True로 표시한다
        ↓
다음 조건을 모두 만족하는지 확인한다

이미지를 정상적으로 읽을 수 있음
+
Width / Height가 0보다 큼
+
Exact Duplicate가 아님
+
Near Uniform Image가 아님
        ↓
모두 만족하면
auto_pass = True
        ↓
각 이미지의 검사 결과를
synthetic_qa.csv에 저장한다
        ↓
DCGAN / WGAN-GP / Diffusion에서
각각 최대 4장을 Seed 42로 선택한다
        ↓
선택한 이미지를 한 화면에 모아
human_qa_grid.jpg를 만든다
        ↓
전체 이미지 수
QA 통과 수
QA 실패 수를 출력한다
```

### 핵심만 정리하면

이 코드는 다음 두 가지 일을 합니다.

```
1. Automatic QA

전체 생성 이미지
        ↓
파일 읽기
크기 확인
밝기 확인
완전 중복 확인
        ↓
synthetic_qa.csv
```

```
2. Human QA 준비

DCGAN / WGAN-GP / Diffusion
        ↓
방법별 최대 4장 선택
        ↓
human_qa_grid.jpg
```

여기서 꼭 기억할 점은 다음입니다.

> `auto_pass = True`는 Apple이 잘 생성되었다는 뜻이 아니라,  
> 파일·크기·밝기·중복 같은 기본적인 기술 검사에 문제가 없었다는 뜻입니다.

사과가 두 개이거나 꼭지가 여러 개 생기는 것처럼 이미지의 의미와 형태가 이상한 문제는 사람이 `human_qa_grid.jpg`를 보고 다시 확인해야 합니다.

실행합니다.

```bash
python scripts/06_synthetic_qa.py \
  --dataset apple
```

---

# 30. Automatic QA와 Human QA 결과를 확인합니다

Automatic QA를 실행한 결과는 다음과 같습니다.

```text
QA total : 39
QA pass  : 37
QA fail  : 2
```

39장의 Synthetic Image 중 37장은 기본적인 파일 검사를 통과했고,  
2장은 `Exact Duplicate`로 확인되었습니다.

실제 CSV를 보면 다음 두 파일입니다.

두 이미지는

```
같은 Prompt
    +
같은 Seed 102
    +
같은 Steps 4
    +
같은 Guidance 0.0
```

조건으로 생성되었기 때문에 **완전히 같은 이미지가 만들어졌습니다.**

따라서 Automatic QA가 두 파일을 중복으로 정확하게 찾아낸 것입니다.

> `QA pass = 37`은 이미지 품질이 좋다는 뜻이 아니라  
> 파일·크기·밝기·중복 등의 기본적인 기술 검사에 문제가 없었다는 뜻입니다.

## Human QA Grid를 확인합니다

```
results/apple/compare/human_qa_grid.jpg
```

![[Pasted image 20261012121157.png]]

실제 Grid에서는 다음 차이가 보입니다.

|방법|실제 관찰|
|---|---|
|DCGAN|Apple 색과 둥근 형태는 보이지만 결과가 거의 비슷하고 흐림|
|WGAN-GP|DCGAN보다 형태가 다양하지만 일부 Apple이 찌그러져 있음|
|Diffusion|가장 선명하지만 일부 이미지에서 꼭지가 여러 개 생기는 Artifact가 보임|

따라서 사람은 다음을 확인합니다.

```
Apple 형태가 정상적인가?
Artifact가 있는가?
같은 결과만 반복되는가?
실제 학습데이터 후보로 사용할 수 있는가?
```

Human QA 판단은 다음 세 가지로 기록합니다.

```
approved
→ 사용 후보

needs_review
→ 추가 확인 필요

rejected
→ 사용하기 어려움
```

> Automatic QA는 기술적인 이상을 찾고,  
> Human QA는 실제 Apple처럼 보이는지를 판단합니다.

---

# 31. 1일차와 2일차의 생성 결과를 정리합니다

이번 실습에서는 같은 Apple을 대상으로 여러 생성 방법을 비교했습니다.

| 방법 | 실제 결과 |
|---|---|
| Augmentation | 기존 Real Image의 밝기·회전·Noise 등을 직접 변형 |
| VAE | Apple의 큰 특징은 보였지만 Random Sample이 흐릿함 |
| DCGAN | 붉은 Apple은 생성했지만 결과가 서로 매우 비슷함 |
| WGAN-GP | DCGAN보다 형태가 다양했지만 일부 이미지가 찌그러짐 |
| Diffusion | 가장 선명했지만 사과 수 오류, 여러 꼭지, 중복 이미지가 발생함 |

이번 결과에서 중요한 점은 다음입니다.

```text
선명하다
≠
좋은 학습데이터다
```

생성 이미지가 보기 좋아도

```
형태가 이상하거나
Artifact가 있거나
같은 이미지가 반복되거나
Real 데이터와 차이가 크다면
```

그대로 학습데이터로 사용하기 어렵습니다.

---

## 결과는 이렇게 간단히 기록합니다

모든 이미지를 길게 설명하지 않고,  
각 방법에서 가장 눈에 띈 특징과 문제점만 기록합니다.

예:

```
VAE
- 결과: Apple 특징은 보였지만 Random Sample이 흐릿함
- 판단: 선명한 학습데이터로 사용하기 어려움

DCGAN
- 결과: 붉은 Apple 생성
- 문제: 여러 이미지가 매우 비슷함
- 판단: 다양성 부족

WGAN-GP
- 결과: DCGAN보다 다양한 형태 생성
- 문제: 일부 Apple이 찌그러짐
- 판단: 다양성은 개선되었지만 형태 안정성 확인 필요

Diffusion
- 결과: 가장 선명하고 사실적인 이미지 생성
- 문제: 사과 2개, 여러 꼭지, Exact Duplicate 발생
- 판단: 생성 후 Human QA 필요
```

실제 실습 기록에는 다음 정도만 남기면 충분합니다.

```
## 생성모델 결과 요약

- VAE: Random Sample이 흐릿했다.
- DCGAN: Apple 형태는 생성했지만 다양성이 낮았다.
- WGAN-GP: 다양성은 증가했지만 일부 형태 왜곡이 있었다.
- Diffusion: 가장 선명했지만 구조적 Artifact와 중복 이미지가 발생했다.

## 최종 결론

생성 이미지는 선명도만으로 평가하지 않는다.
Real 데이터와 비교하여 형태, 다양성, Artifact, 중복 여부를 확인한 뒤 사용 여부를 결정한다.
```

> 이번 실험의 핵심은 어떤 모델이 무조건 최고인지를 정하는 것이 아니라, 각 생성 방식의 장점과 실패 패턴을 실제 결과로 확인하는 것입니다.

---

# PART 6. 2일차 Apple 본 실습을 마무리합니다

---

# 32. Apple 본 실습의 주요 결과 파일을 확인합니다

Apple 본 실습이 끝나면 다음 결과가 남아 있어야 합니다.

```text
results/apple/
├─ dcgan/
│  ├─ epochs/
│  └─ final/
├─ wgan_gp/
│  ├─ epochs/
│  └─ final/
├─ diffusion/
└─ compare/
   ├─ dcgan_vs_wgan_gp.jpg
   └─ human_qa_grid.jpg

models/apple/
├─ dcgan_generator.pt
├─ dcgan_discriminator.pt
├─ wgan_gp_generator.pt
└─ wgan_gp_critic.pt

reports/apple/
├─ dcgan_loss.csv
├─ dcgan_metadata.csv
├─ wgan_gp_loss.csv
├─ wgan_gp_metadata.csv
├─ diffusion_metadata.csv
├─ domain_gap_review.csv
└─ synthetic_qa.csv
```

모든 파일의 내용을 다시 확인할 필요는 없습니다.

다음 네 종류의 결과가 정상적으로 남아 있는지만 확인합니다.

```
생성 이미지
→ DCGAN / WGAN-GP / Diffusion 결과

학습 기록
→ Loss

실험 조건
→ Metadata

검수 결과
→ Domain Gap / Automatic QA / Human QA
```

이 네 종류가 준비되어 있으면 Apple 본 실습은 완료된 것입니다.

---
# 33. Apple 본 실습의 최종 결과를 짧게 기록합니다

앞에서 이미 각 모델의 결과를 자세히 확인했으므로  
여기서는 다시 분석하지 않습니다.

자신의 실제 실행 결과를 다음 세 문장으로만 정리합니다.

```
DCGAN:
→ 붉은 Apple은 생성했지만 결과가 서로 매우 비슷해 다양성이 낮았다.

WGAN-GP:
→ DCGAN보다 다양한 형태가 생성되었지만 일부 형태 왜곡이 있었다.

Diffusion:
→ 가장 선명했지만 사과 수 오류, 여러 꼭지, 중복 이미지 같은 Artifact가 발생했다.
```

마지막으로 한 문장으로 정리합니다.

> 생성 이미지는 보기 좋은지만 확인하는 것이 아니라  
> 다양성, Artifact, Domain Gap, 중복 여부를 함께 확인한 뒤 사용 여부를 결정해야 합니다.

---

현업에서는 하나의 생성모델을 무조건 사용하는 것이 아니라, 여러 후보 모델을 1일차 2일차와 
같은 기준으로 실험·기록·검수하여 실제 업무 데이터에 가장 적합한 모델을 선택합니다. 
이번 실습은 그 실무 판단 과정을 경험하는 단계입니다.

---

# PART 7. Mini Challenge — Banana 1로 생성모델 비교·검수 과정을 스스로 수행합니다

---

# 34. Mini Challenge의 목표를 확인합니다

1일차 Mini Challenge에서 이미 `Banana 1` 데이터를 준비하고 다음 과정을 수행했습니다.

```
Banana 원본
        ↓
Train / Validation / Test
        ↓
Augmentation
        ↓
VAE
        ↓
QA
        ↓
Leakage 확인
```

2일차에는 **1일차에서 만든 Banana Train을 그대로 이어서 사용**합니다.

```
../day01_image_aug_vae/data/banana/split/train/
```

새로운 Banana 데이터를 다시 다운로드하거나 수집하지 않습니다.

이번 Challenge에서는 Apple 본 실습에서 배운 생성모델 비교 방법을 Banana에 스스로 적용합니다.

```
Banana Train
        ↓
DCGAN
        ↓
WGAN-GP
        ↓
두 GAN 결과 비교
        ↓
Pretrained Diffusion
        ↓
Seed 변화 실험
        ↓
비교 기준 Seed 선택
        ↓
Prompt 변화 실험
        ↓
Real Banana와 비교
        ↓
Automatic QA
+
Human QA
        ↓
학습데이터 사용 가능성 판단
```

> 이번 Challenge의 목표는 명령어를 모두 실행하는 것이 아니라  
> **새로운 데이터에서도 생성 결과를 비교하고, 실패 원인을 찾아 사용 가능 여부를 판단할 수 있는지 확인하는 것**입니다.

---

# 35. Challenge 1 — Banana Train이 준비되어 있는지 확인합니다

먼저 1일차의 Banana Train과 현재 실행환경이 정상적으로 연결되는지 확인합니다.

```bash
python scripts/00_check_day02.py \
  --dataset banana
```

실행 결과에서 다음만 확인합니다.

```
Dataset
→ banana

Train Dir
→ 1일차 Banana split/train 경로

Train Images
→ Banana Train 이미지 수

CUDA Available
→ GPU 사용 가능 여부

Day 02 preflight
→ READY
```

간단히 기록합니다.

```
Banana Train 수:

대표 이미지 크기:

Preflight 결과:
```

`READY`가 확인되면 다음 단계로 진행합니다.

---

# 36. Challenge 2 — Banana DCGAN을 짧게 학습합니다

Apple에서는 15 Epoch로 본 실습을 진행했습니다.

Mini Challenge에서는 구조를 다시 경험하는 것이 목적이므로 **5 Epoch**로 짧게 실행합니다.

```bash
python scripts/01_train_dcgan.py \
  --dataset banana \
  --epochs 5
```

Final 결과를 확인합니다.

```
results/banana/dcgan/final/
```

다음 세 가지를 중심으로 봅니다.

```
노란색 Banana 특징이 나타나는가?

길고 휘어진 형태가 나타나는가?

서로 다른 Noise인데
비슷한 결과만 반복되는가?
```

결과를 짧게 기록합니다.

```
Banana 형태:

다양성:

가장 눈에 띈 실패:
```

> 5 Epoch 결과가 좋지 않아도 실패한 실습이 아닙니다.  
> 현재 조건에서 DCGAN이 어떤 결과를 만드는지 관찰하는 것이 목적입니다.

---

# 37. Challenge 3 — 같은 Banana Train으로 WGAN-GP를 실행합니다

이번에는 같은 데이터와 같은 Epoch 조건에서 WGAN-GP를 실행합니다.

```bash
python scripts/02_train_wgan_gp.py \
  --dataset banana \
  --epochs 5
```

Final 결과를 확인합니다.

```
results/banana/wgan_gp/final/
```

다음을 살펴봅니다.

```
Banana 형태가 어느 정도 나타나는가?

DCGAN보다 형태가 다양해졌는가?

찌그러짐이나 비정상적인 형태는 없는가?
```

간단히 기록합니다.

```
Banana 형태:

다양성:

가장 눈에 띈 실패:
```

---

# 38. Challenge 4 — DCGAN과 WGAN-GP를 같은 화면에서 비교합니다

비교 이미지를 만듭니다.

```bash
python scripts/03_compare_gan_results.py \
  --dataset banana
```

확인할 파일:

```
results/banana/compare/
└─ dcgan_vs_wgan_gp.jpg
```

같은 기준으로 두 결과를 비교합니다.

|비교 항목|DCGAN|WGAN-GP|
|---|---|---|
|노란색 특징|||
|Banana 형태|||
|다양성|||
|비슷한 결과 반복|||
|형태 왜곡 / Artifact|||

마지막으로 한 문장만 작성합니다.

```
현재 5 Epoch 실험에서는
___________________________
```

> 현재 5 Epoch 결과만으로  
> “DCGAN이 항상 좋다” 또는 “WGAN-GP가 항상 좋다”라고 일반화하지 않습니다.

---

# 39. Challenge 5 — Diffusion의 Seed를 바꾸어 Banana를 생성합니다

Apple 실습에서 Diffusion Model의 실행 방법은 이미 확인했습니다.

이번에는 같은 Pretrained Model을 사용하고 Banana Prompt로 결과를 생성합니다.

기준 Prompt:

```
a single yellow banana on a clean white background,
centered object,
realistic product photography,
neutral lighting
```

전체 의미:

> 깨끗한 흰색 배경 중앙에 노란 Banana 한 개를 놓고,  
> 중립적인 조명에서 실제 제품 사진처럼 표현합니다.

먼저 Prompt는 그대로 두고 Seed만 바꿉니다.

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed 300 \
  --steps 4 \
  --guidance 0.0 \
  --count 4 \
  --tag same_prompt
```

다음 네 Seed 결과가 생성됩니다.

```
Seed 300
Seed 301
Seed 302
Seed 303
```

네 결과를 보고 다음을 확인합니다.

```
Banana가 한 개인가?

길고 휘어진 형태가 자연스러운가?

꼭지나 끝부분에 이상한 구조가 없는가?

객체가 여러 개로 생성되지 않았는가?

심한 Artifact가 없는가?
```

그중 **Prompt 조건을 비교적 잘 만족하는 Seed 하나를 다음 실험의 기준으로 선택**합니다.

```
선택한 Seed:

선택 이유:
```

> 좋은 Seed가 정해져 있는 것은 아닙니다.  
> 이번 생성 결과 중 **Prompt 비교에 사용할 정상적인 기준 Sample을 선택하는 것**입니다.

---

# 40. Challenge 6 — 선택한 Seed를 고정하고 Prompt의 조명만 바꿉니다

39번에서 선택한 Seed를 사용합니다.

예를 들어 `Seed 302`를 선택했다면 다음처럼 지정합니다.

```
SELECTED_SEED=302
```

`302`는 예시입니다.  
실제로 자신이 선택한 Seed로 바꿉니다.

### 기준 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, neutral lighting" \
  --seed "$SELECTED_SEED" \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag neutral_light
```

### 어두운 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, dim lighting" \
  --seed "$SELECTED_SEED" \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag dim_light
```

### 강한 조명

```bash
python scripts/04_diffusion_generate.py \
  --dataset banana \
  --prompt "a single yellow banana on a clean white background, centered object, realistic product photography, strong overhead lighting and reflections" \
  --seed "$SELECTED_SEED" \
  --steps 4 \
  --guidance 0.0 \
  --count 1 \
  --tag strong_light
```

이제 다음을 비교합니다.

```
Seed
→ 동일

기본 Banana Prompt
→ 동일

변경한 조건
→ 조명 표현만 변경
```

결과에서 확인합니다.

```
밝기와 반사가 달라졌는가?

Banana의 기본 형태는 유지되었는가?

조명을 바꾸었는데
예상하지 않은 형태 변화도 생겼는가?

Artifact가 발생한 결과가 있는가?
```

간단히 기록합니다.

```
Prompt 변화가 가장 잘 반영된 결과:

형태가 가장 안정적인 결과:

Reject가 필요한 결과와 이유:
```

Metadata도 확인합니다.

```bash
cat reports/banana/diffusion_metadata.csv
```

다음 내용이 다시 확인되면 됩니다.

```
Model
Prompt
Seed
Steps
Guidance
```

---

# 41. Challenge 7 — Real Banana와 Synthetic Banana의 차이를 확인합니다

Domain Gap 검수표를 만듭니다.

```bash
python scripts/05_domain_gap_review.py \
  --dataset banana
```

결과:

```csv
reports/banana/domain_gap_review.csv
```

이 CSV의 모든 행을 일일이 작성할 필요는 없습니다.

다음 결과를 대표적으로 확인합니다.

```
Real Banana
→ 비교 기준

DCGAN Final
→ 대표 결과와 실패 결과

WGAN-GP Final
→ 대표 결과와 실패 결과

Diffusion
→ 정상 후보와 Artifact 결과
```

다음 세 문장만 정리합니다.

```
DCGAN과 Real Banana의 가장 큰 차이:

WGAN-GP와 Real Banana의 가장 큰 차이:

Diffusion과 Real Banana의 가장 큰 차이:
```

필요하다면 대표 이미지 몇 장의 `decision`과 `note`만 작성합니다.

예:

```
approved
→ 현재 사용 후보

needs_review
→ 추가 확인 필요

rejected
→ 형태나 Artifact 때문에 사용하기 어려움
```

> Domain Gap 검수의 목적은 모든 CSV 칸을 채우는 것이 아니라  
> **Real 데이터와 Synthetic 데이터가 어디에서 달라지는지 발견하는 것**입니다.

---

# 42. Challenge 8 — Automatic QA와 Human QA를 수행합니다

전체 Synthetic Image의 기술적인 이상을 자동 검사합니다.

```bash
python scripts/06_synthetic_qa.py \
  --dataset banana
```

확인할 결과:

```
reports/banana/synthetic_qa.csv

results/banana/compare/
└─ human_qa_grid.jpg
```

터미널에서 다음 값을 기록합니다.

```
QA total:

QA pass:

QA fail:
```

`synthetic_qa.csv`에서는 다음을 확인합니다.

```
readable
→ 파일을 정상적으로 읽을 수 있는가?

near_uniform
→ 거의 한 가지 밝기로 된 비정상 이미지인가?

exact_duplicate
→ 완전히 동일한 이미지가 중복되었는가?

auto_pass
→ 기본적인 기술 검사에 통과했는가?
```

중요합니다.

```
auto_pass = True
≠
좋은 Banana 학습데이터
```

예를 들어 Banana가 두 개 생성되거나 형태가 이상해도 파일 자체가 정상이라면 Automatic QA를 통과할 수 있습니다.

따라서 `human_qa_grid.jpg`를 보고 사람이 다시 확인합니다.

```
Banana 한 개가 자연스럽게 보이는가?

길고 휘어진 형태가 유지되는가?

꼭지와 끝부분이 정상적인가?

여러 객체로 분리되지는 않았는가?

심한 Artifact가 없는가?

결과들이 충분히 다양한가?
```

최종적으로 대표 Sample을

```
approved
needs_review
rejected
```

중 하나로 판단합니다.

---

# 43. Mini Challenge 결과를 기록합니다

**파일: `reports/banana/challenge_notes.md`**

모든 실행 내용을 다시 적지 않습니다.

**실제로 관찰한 결과와 판단만 짧게 기록합니다.**

```markdown
# Day02 Mini Challenge - Banana 1

## 1. 데이터

- Banana Train 수:
- 사용한 데이터: 1일차 Banana Train 재사용

## 2. DCGAN

- Epoch: 5
- 생성 특징:
- 다양성:
- 대표 실패:

## 3. WGAN-GP

- Epoch: 5
- 생성 특징:
- DCGAN과 달랐던 점:
- 대표 실패:

## 4. GAN 비교

- 현재 5 Epoch에서 더 안정적으로 보인 방법:
- 판단 이유:

## 5. Diffusion

- Model: stabilityai/sd-turbo
- 비교한 Seed: 300 / 301 / 302 / 303
- 선택한 기준 Seed:
- 선택 이유:
- Prompt 변경 결과:
- 대표 Artifact:

## 6. Domain Gap / QA

- DCGAN과 Real의 가장 큰 차이:
- WGAN-GP와 Real의 가장 큰 차이:
- Diffusion과 Real의 가장 큰 차이:
- Automatic QA Total / Pass / Fail:
- Human QA에서 발견한 문제:

## 7. 학습데이터 사용 판단

- DCGAN:
  - approved / needs_review / rejected
  - 이유:

- WGAN-GP:
  - approved / needs_review / rejected
  - 이유:

- Diffusion:
  - 전체 사용 / 일부 사용 / 사용 어려움
  - 이유:

## 8. 최종 결론

- 이번 Banana 실험에서 가장 적합해 보인 생성 방법:
- 그대로 사용하기 어려운 방법과 이유:
- 품질이 부족하다면 다음에 시도할 개선 또는 다른 생성모델:
```

여기서 마지막 판단이 중요합니다.

```
이미지를 생성했다
        ↓
끝
X


이미지를 생성했다
        ↓
Real과 비교했다
        ↓
QA를 수행했다
        ↓
사용 가능성을 판단했다
        ↓
부족하면
조건 개선 또는 다른 모델 검토
O
```

현업에서도 하나의 생성모델을 무조건 사용하지 않습니다.

**같은 기준으로 후보 모델을 생성·기록·검수하고, 실제 업무 데이터에 가장 적합한 방법을 선택하는 과정 자체가 중요한 작업입니다.**

---

# 44. Mini Challenge 완료 조건을 확인합니다

아래 항목을 확인하면 Challenge가 완료된 것입니다.

- [ ] 1일차 `Banana 1` Train을 그대로 사용했다.
- [ ] Banana Train과 실행환경이 정상인지 확인했다.
- [ ] DCGAN을 5 Epoch 실행하고 결과를 관찰했다.
- [ ] WGAN-GP를 5 Epoch 실행하고 결과를 관찰했다.
- [ ] 같은 조건에서 두 GAN의 Final 결과를 비교했다.
- [ ] Diffusion에서 같은 Prompt로 여러 Seed를 비교했다.
- [ ] 결과가 비교적 정상적인 Seed 하나를 스스로 선택했다.
- [ ] 선택한 Seed를 고정하고 조명 Prompt만 변경했다.
- [ ] Diffusion Metadata를 확인했다.
- [ ] Real Banana와 Synthetic Banana의 Domain Gap을 비교했다.
- [ ] Automatic QA의 `Pass`가 이미지 품질 승인을 뜻하지 않는다는 것을 이해했다.
- [ ] Human QA로 형태와 Artifact를 확인했다.
- [ ] 각 생성 방법이 실제 합성데이터 후보로 사용할 수 있는지 판단했다.
- [ ] 결과가 부족할 경우 조건을 개선하거나 다른 생성모델을 검토할 수 있음을 이해했다.
- [ ] `challenge_notes.md`에 결과와 판단 근거를 기록했다.

---

# PART 8. 2일차 결과를 기록하고 Git으로 마무리합니다

---

# 45. `day02_notes.md`에 실제 실험 결과를 정리합니다

`day02_notes.md`는 오늘 실험에서 나온 **실제 결과와 판단 근거**를 기록하는 파일입니다.

코드 실행 과정을 다시 설명하지 않고,
실제로 관찰한 결과만 짧게 남깁니다.

**파일: `day02_notes.md`**

```markdown
# Subject 11 - Day 02
## GAN · WGAN-GP · Diffusion

## 1. Apple 데이터

- Dataset: Apple Red 1
- 사용 데이터: 1일차 Train 재사용
- GAN 학습: Train만 사용
- Validation / Test: GAN 학습에 사용하지 않음

---

## 2. DCGAN 결과

- Epoch: 15
- 빨간 Apple 특징: 나타남
- 형태: 대부분 비슷한 둥근 형태
- 다양성: 낮음
- 반복 패턴: 많이 보임
- 표면: 거칠고 흐림

### 판단

Apple의 큰 특징은 학습했지만
생성 결과가 서로 매우 비슷하여
현재 결과를 그대로 합성 학습데이터로 사용하기에는
다양성이 부족했다.

---

## 3. WGAN-GP 결과

- Epoch: 15
- 빨간 Apple 특징: 나타남
- 다양성: DCGAN보다 높음
- 형태: 더 다양한 형태가 생성됨
- 실패 사례: 일부 이미지가 길쭉하거나 찌그러지고 각진 형태로 생성됨

### 판단

DCGAN보다 다양성은 증가했지만
형태 안정성이 충분하지 않았다.

현재 결과 전체를 그대로 학습데이터로 사용하기보다는
추가 개선과 검수가 필요하다.

---

## 4. Diffusion 결과

- Model: stabilityai/sd-turbo
- Steps: 4
- Guidance: 0.0
- Prompt 고정 Seed 비교: 100 / 101 / 102 / 103
- 비교 기준으로 선택한 Seed: 102

### 실제 관찰

- Seed 100: Apple 구조가 다소 부자연스러움
- Seed 101: 한 개의 Apple Prompt인데 여러 객체가 생성됨
- Seed 102: 비교적 자연스러운 한 개의 Apple 생성
- Seed 103: Apple은 한 개지만 표면 질감 확인 필요

조명 Prompt를 변경했을 때도
밝기뿐 아니라 Apple 구조가 함께 달라지는 경우가 있었다.

특히 일부 결과에서는
여러 꼭지와 같은 구조적 Artifact가 확인되었다.

### 판단

Diffusion은 가장 선명하고 사실적인 결과를 만들었지만
모든 생성 이미지를 그대로 사용할 수는 없었다.

Human QA를 통과한 이미지 일부만
학습데이터 후보로 사용할 수 있다고 판단했다.

---

## 5. Automatic QA 결과

- QA Total: 39
- QA Pass: 37
- QA Fail: 2

Exact Duplicate:

- neutral_light_seed102_step4.png
- same_prompt_seed102_step4.png

두 이미지는 같은 Prompt와 Seed 조건으로 생성되어
파일 내용이 완전히 동일했다.

### 중요한 해석

Automatic QA 통과는
좋은 학습데이터라는 의미가 아니다.

Automatic QA
→ 파일·크기·밝기·중복 검사

Human QA
→ 객체 수·형태·Artifact·현실성 검사

---

## 6. 생성방법 비교

| 방법 | 이번 실험에서 확인한 결과 |
|---|---|
| Augmentation | Real Image를 현실적인 범위에서 직접 변형 |
| VAE | Apple의 특징은 보였지만 Random Sample이 흐림 |
| DCGAN | Apple은 생성했지만 다양성이 낮음 |
| WGAN-GP | 다양성은 증가했지만 일부 형태 왜곡 발생 |
| Diffusion | 가장 선명하지만 구조적 Artifact와 중복 가능 |

---

## 7. Banana Mini Challenge

- Banana Train 수:
- DCGAN Epoch:
- DCGAN 결과:
- WGAN-GP Epoch:
- WGAN-GP 결과:
- 현재 더 안정적으로 보인 방법:
- Diffusion 기준 Seed:
- Diffusion 대표 Artifact:
- Automatic QA Total / Pass / Fail:
- Human QA 판단:
- 현재 가장 사용 가능성이 높은 방법:

---

## 8. 최종 결론

이번 실험에서는
VAE·DCGAN·WGAN-GP 결과를 그대로 합성 학습데이터로 사용하기에는
품질 또는 다양성 측면에서 부족한 부분이 있었다.

Diffusion은 더 현실적인 결과를 생성했지만
Artifact와 중복이 발생했기 때문에
Human QA를 통과한 일부 결과만 사용 후보가 될 수 있었다.

따라서 합성데이터는 생성 후 바로 사용하는 것이 아니라

생성
→ 기록
→ Real과 비교
→ Automatic QA
→ Human QA
→ 사용 여부 판단

과정을 거쳐야 한다.

현재 모델의 품질이 부족하다면
다른 생성모델이나 생성 조건을 같은 기준으로 다시 실험하고
업무 목적에 맞는 방법을 선택해야 한다.
```

---

# 46. README에 2일차 프로젝트를 정리합니다

README는 상세한 실험일지가 아닙니다.

GitHub에서 프로젝트를 처음 보는 사람이

```
무엇을 실험했는가?
어떤 데이터를 사용했는가?
어떤 순서로 실행하는가?
무슨 결과가 나왔는가?
```

를 빠르게 이해할 수 있도록 작성합니다.

**파일: `README.md`**

````markdown
# Subject 11 - Day 02
## GAN · WGAN-GP · Diffusion Synthetic Data Lab

## 1. 실습 목적

같은 Real Image를 기준으로 여러 생성방법을 비교하고,
생성된 이미지가 실제 AI 학습데이터로 사용할 수 있는지
Domain Gap과 QA를 통해 판단합니다.

---

## 2. Dataset

### Main Lab

Day 01의 `Apple Red 1` Train을 그대로 사용합니다.

```text
../day01_image_aug_vae/data/apple/split/train/
```

### Mini Challenge

Day 01의 `Banana 1` Train을 그대로 사용합니다.

```text
../day01_image_aug_vae/data/banana/split/train/
```

새로운 Real Image를 다시 수집하지 않습니다.

---

## 3. Pipeline

```text
Real Train
        ↓
 ┌──────┴──────┐
 ↓             ↓

DCGAN       WGAN-GP
 ↓             ↓
Generated Image
        ↓
GAN 결과 비교


Pretrained Diffusion
        ↓
Prompt + Seed
        ↓
Generated Image


Real
VS
Synthetic
        ↓
Domain Gap
        ↓
Automatic QA
        ↓
Human QA
        ↓
사용 후보 판단
```

---

## 4. Main Scripts

```text
scripts/
├─ 00_check_day02.py
├─ 01_train_dcgan.py
├─ 02_train_wgan_gp.py
├─ 03_compare_gan_results.py
├─ 04_diffusion_generate.py
├─ 05_domain_gap_review.py
└─ 06_synthetic_qa.py
```

---

## 5. Apple 실제 결과

### DCGAN

```text
Apple 특징
→ 생성됨

문제
→ 결과가 서로 매우 비슷함
→ 다양성 부족
→ 흐리고 거친 표면
```

### WGAN-GP

```text
DCGAN보다
→ 다양성 증가

문제
→ 일부 형태 왜곡
→ 찌그러지거나 각진 Apple 발생
```

### Diffusion

```text
장점
→ 가장 선명하고 사실적인 이미지

문제
→ 객체 수 오류
→ 여러 꼭지
→ 구조적 Artifact
→ Exact Duplicate 발생
```

Automatic QA 결과:

```text
Total : 39
Pass  : 37
Fail  : 2
```

따라서 이번 Apple 실험에서는

```text
DCGAN / WGAN-GP
→ 현재 결과 전체 사용은 어려움

Diffusion
→ Human QA를 통과한 일부 결과만 사용 후보
```

로 판단했습니다.

---

## 6. 핵심 결론

```text
보기 좋은 이미지
≠
좋은 학습데이터
```

Synthetic Data는 다음 기준을 함께 확인해야 합니다.

```text
형태
다양성
Artifact
Domain Gap
Duplicate
Metadata
QA
```

Automatic QA를 통과했다고
학습데이터 사용이 자동 승인되는 것은 아닙니다.

실제 객체의 형태와 의미는
Human QA로 다시 확인해야 합니다.

---

## 7. Mini Challenge

`Banana 1`에서는 같은 스크립트를 재사용하여

```text
DCGAN
→ WGAN-GP
→ Diffusion
→ Domain Gap
→ Automatic QA
→ Human QA
```

를 스스로 수행합니다.

목표는 단순 실행이 아니라
**어떤 생성방법이 Banana 합성데이터에 더 적합한지
실제 결과를 근거로 판단하는 것**입니다.

---

## 8. Main Outputs

```text
results/
→ 생성 이미지와 비교 Grid

models/
→ 학습된 GAN Model

reports/
→ Loss
→ Metadata
→ Domain Gap Review
→ Automatic QA 결과
```

상세한 실험 결과는 다음 파일에 기록합니다.

```text
day02_notes.md
reports/banana/challenge_notes.md
```

---

## 9. Next

Day 03에서는 단순히 이미지를 생성하는 단계를 넘어

```text
Background
+
Object PNG
        ↓
Cut-Paste
        ↓
Synthetic Image
+
Bounding Box
        ↓
YOLO Label
```

형태로 Detection 학습데이터를 자동 생성합니다.
````

---

# 47. Git에 올리지 않을 파일을 확인합니다

모델 파일과 대량의 생성 이미지는 Git에 올리지 않습니다.

**파일: `.gitignore`**

```
# Python
__pycache__/
*.pyc

# Virtual Environment
.venv/

# Model weights
models/
*.pt
*.pth
*.bin
*.safetensors

# Generated images
results/

# Model cache
.cache/
huggingface/

# Editor
.vscode/

# OS
.DS_Store
```

다음 파일은 Git에 남깁니다.

```
Python Code
README.md
day02_notes.md
reports/*.csv
reports/banana/challenge_notes.md
```

특히 `reports/`의 CSV는 크기가 작고

```
어떤 조건으로 학습했는가?
Loss가 어떻게 변했는가?
어떤 이미지가 QA에서 실패했는가?
```

를 다시 확인할 수 있으므로 실험 기록으로 보관합니다.

---

# 48. 2일차 작업을 Git에 저장합니다

현재 실제 프로젝트 Root로 이동합니다.

```bash
cd ~/project11
```

현재 상태를 확인합니다.

```bash
git status
```

2일차 파일을 Stage에 올립니다.

```bash
git add day02_generative_models
```

다시 확인합니다.

```bash
git status
```

다음 파일들이 포함되어 있는지 확인합니다.

```
Python 코드
README.md
day02_notes.md
reports/
```

반대로 다음 대용량 파일은 포함되지 않아야 합니다.

```
models/
results/
```

Commit합니다.

```bash
git commit -m "feat: add subject11 day02 generative model lab"
```

최근 Commit을 확인합니다.

```bash
git log --oneline --decorate -5
```

원격 저장소가 연결되어 있는지 확인합니다.

```bash
git remote -v
```

이미 Upstream이 설정되어 있다면 Push합니다.

```bash
git push
```

> 외부 Git 사용이 제한된 환경에서는 Local Git 또는 기관 내부 Git 저장소에 같은 방식으로 기록합니다.

---

# 49. 2일차 최종 자가 체크리스트를 확인합니다

중복되는 세부 실행 항목은 제외하고 핵심만 확인합니다.

- [ ] 1일차 Apple Train만 GAN 학습에 사용했다.
- [ ] DCGAN과 WGAN-GP의 생성 결과 차이를 설명할 수 있다.
- [ ] GAN Loss만으로 생성 품질을 판단하지 않는 이유를 이해했다.
- [ ] Diffusion에서 Prompt와 Seed의 역할을 설명할 수 있다.
- [ ] Metadata에 생성 조건을 기록했다.
- [ ] Real과 Synthetic의 Domain Gap을 확인했다.
- [ ] Automatic QA와 Human QA의 역할 차이를 설명할 수 있다.
- [ ] `auto_pass=True`가 학습데이터 최종 승인을 의미하지 않는다는 것을 이해했다.
- [ ] Banana Mini Challenge에서 같은 Pipeline을 스스로 적용했다.
- [ ] `day02_notes.md`와 README를 정리했다.
- [ ] Git Commit을 완료했다.

---

# 50. 2일차에서 반드시 기억할 세 가지를 정리합니다

## ① 생성모델마다 결과의 성격이 다릅니다

```
VAE
→ 흐림

DCGAN
→ 다양성 부족

WGAN-GP
→ 다양성 증가
→ 일부 형태 왜곡

Diffusion
→ 선명함
→ Artifact와 중복 가능
```

어떤 방법이 항상 가장 좋다고 단정하지 않습니다.

실제 데이터를 생성한 뒤 결과를 비교해야 합니다.

---

## ② 생성 성공과 학습데이터 사용 승인은 다릅니다

```
이미지 생성
        ↓
Real과 비교
        ↓
Domain Gap
        ↓
Automatic QA
        ↓
Human QA
        ↓
사용 후보 판단
```

이번 실험에서도 Diffusion은 선명했지만  
모든 이미지가 학습데이터로 적합한 것은 아니었습니다.

---

## ③ 현업에서는 적합한 생성방법을 찾아 검증합니다

현재 결과가 부족하다면

```
다른 생성모델
또는
다른 생성 조건
        ↓
같은 기준으로 실험
        ↓
Metadata
        ↓
Domain Gap
        ↓
QA
        ↓
비교
        ↓
업무 목적에 맞는 방법 선택
```

과정을 다시 수행할 수 있습니다.

> 이번 실습은 생성모델을 실행하는 것에서 끝나는 것이 아니라, 생성 결과를 검증하고 실제 업무에 사용할 수 있는 합성데이터인지 판단하는 기본적인 실무 과정을 경험한 것입니다.

