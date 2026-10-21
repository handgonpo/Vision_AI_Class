## 오늘의 목표

수업이 끝나면 다음을 할 수 있어야 합니다.

- 어떤 데이터·설정·코드로 검사 결과가 나왔는지 기록할 수 있습니다.
- One-shot과 Few-shot이 **같은 이미지 20장**을 검사했는지 확인하고 성능지표를 다시 계산할 수 있습니다.
- 기존 Rule은 보존하고 한 가지 설정만 바꾼 Candidate와 비교할 수 있습니다.
- 카메라 밝기 변화가 이상검출 결과에 미치는 영향을 측정할 수 있습니다.
- One-shot과 Few-shot 결과에 **OR / AND 결합 규칙**을 적용하여 단독 검사와 비교할 수 있습니다.
- Accuracy만이 아니라 Precision·Recall·FP·FN을 함께 보고 결론을 내릴 수 있습니다.
- 결과와 실행 기준 버전을 Git·README에 정리할 수 있습니다.

**주의:** 여기서 Hybrid는 서로 다른 제품의 OK/NG를 억지로 합치는 실습이 아닙니다. **같은 MVTec bottle 이미지**에 대한 두 검사 결과를 결합합니다.

---

# 1. 1~8일차와 9일차는 왜 연결되나요?

앞 수업에서 네 가지 검사 유형을 경험했습니다.

| 일차 | 데이터 | 만든 검사·결과 |
|---|---|---|
| 1~3일차 | 병뚜껑 | ROI → 전처리 → 면적·Contour 측정 → Rule 검사 |
| 4~5일차 | 광천김 로고 | Matching 특징 측정 → Calibration → Rule Freeze → Test |
| 6일차 | 제조 제품의 SN | OCR → 13자리 형식 검사 → 기준번호와 비교 |
| 7일차 | MVTec bottle | 사전학습 ResNet18 특징 → 정상 기준 → 이상검출 |
| 8일차 | 동일한 MVTec bottle 원본 | One-shot / Few-shot → 공통 Query에서 성능 비교 |

9일차에서는 새 검사를 처음부터 만들기보다 **앞서 얻은 결과를 검증·관리·결합**합니다.

![[Pasted image 20260913202841.png]]

> **그림 참고:** 원본 자료의 그림은 그대로 유지했습니다. Hybrid 부분은 그림의 기존 표현보다 **아래 PART 6의 동일 데이터 결과 결합 실습**을 기준으로 이해하세요.

---

# 2. 이전 수업에서 얻은 주요 결과

| 실습 | 교재에 기록된 결과 | 9일차에서 활용하는 방식 |
|---|---|---|
| 3일차 병뚜껑 | Area Ratio 기준 `0.3608`, Batch Accuracy `1.0` | 규칙도 버전 관리 대상임을 이해 |
| 5일차 광천김 | Calibration `91.67%`, Test `50%` | Baseline/Candidate 실험의 예 |
| 6일차 OCR | 6장 OCR·제품 판정 정확도 `100%` | OCR Result와 Config의 이력 확인 |
| 7일차 이상검출 | 83장 Test, Accuracy `92.77%` | 기존 Reference/Threshold를 고정한 Drift 실험 |
| 8일차 One-shot | 공통 Query 20장, Accuracy `85%` | 재현성 확인 및 Hybrid 입력 |
| 8일차 Few-shot | 동일 Query 20장, Accuracy `90%` | 재현성 확인 및 Hybrid 입력 |

**중요:** 위 수치는 서로 검사 대상과 Test 구성도 다릅니다. 단순히 정확도 숫자만 나란히 놓고 어떤 기술이 우수하다고 판단하지 않습니다. **Hybrid 비교에는 같은 Day08 Query 20장의 결과만 사용**합니다.

![[Pasted image 20261019121329.png]]

---

# 3. 9일차에서 최종적으로 만드는 것

| 산출물 | 왜 필요한가요? |
|---|---|
| `version_manifest.json` | 데이터·설정·산출물·결과·Git 버전 추적 |
| `day08_reproducibility_audit.csv` / `.json` | 두 검사 결과가 같은 Query인지 확인하고 Metric 재계산 |
| `experiment_log.csv` / `experiment_comparison.csv` | Day05 Rule의 한 변수 변경 비교 |
| `drift_results.csv` / `drift_summary.csv` | Day07 검사기를 유지한 채 밝기 변화 영향 평가 |
| `hybrid_inputs.csv` | Day08 동일 샘플별 One-shot·Few-shot 예측 결합 |
| `hybrid_policy.json` | Test 결과를 보기 전에 정한 OR / AND 규칙 기록 |
| `hybrid_results.csv` / `hybrid_metrics.csv` | Hybrid의 샘플별 결과 및 성능지표 |
| `hybrid_comparison.md` | 단독 검사와 결합 검사의 차이를 해석 |
| `day09_analysis.md`, README, Git | 실제 결과와 실패 원인 정리 |

![[Pasted image 20260913203754.png]]

---

# 4. 오늘은 어떤 순서로 진행하나요?

```text
1~8일차 산출물 확인
       ↓
Version Manifest 생성
       ↓
Day08 One-shot/Few-shot 결과 재검증
       ↓
Day05 Baseline vs Candidate 비교
       ↓
Day07 밝기 변화 Data Drift 측정
       ↓
Day08 공통 Query 결과로 Hybrid OR/AND 비교
       ↓
검사기술 선택 Challenge
       ↓
Blur Drift Mini Challenge
       ↓
실패 분석 → README → Git → 10일차 연결
```

![[Pasted image 20260913205236.png]]

> **그림 참고:** 기존 그림에 표시된 `Rule / OCR / Anomaly 결과 연결`은 이전 실습 흐름입니다. 이번 개정에서는 그 자리를 **One-shot / Few-shot Hybrid 성능 비교**로 대체합니다.

---

# 5. 핵심 용어를 쉽게 이해합니다

| 용어 | 쉽게 말하면 |
|---|---|
| Dataset | 검사에 사용한 이미지 |
| Config | 검사 설정과 판정 규칙 |
| Artifact | 다시 불러올 수 있는 Reference·Prototype 등 |
| Result | 샘플별 예측과 측정 결과 |
| Version / Git Commit | 어떤 상태의 파일과 코드였는지 구분 |
| Manifest / Lineage | 결과가 어디에서 왔는지 적은 연결 목록 |
| Hash / Fingerprint | 파일 또는 데이터 구성의 변경 여부를 확인하는 지문 |
| Baseline / Candidate | 원래 기준 / 한 가지만 변경한 후보 |
| Reproducibility | 저장된 결과를 다시 계산해 확인할 수 있는가 |
| Drift | 규칙은 그대로인데 입력 환경의 특성이 변하는 현상 |
| Hybrid | **같은 검사 대상**에 대해 두 방법의 판정을 결합하는 방식 |
| FP / FN | 정상을 불량으로 오판 / 불량을 정상으로 놓침 |
| Rollback / Git Tag | 이전 기준으로 돌아가기 / 상태에 붙인 이름 |

![[Pasted image 20260913205900.png]]

![[Pasted image 20260913210630.png]]

---

# 6. 오늘의 실습 구성과 데이터

| PART | 핵심 활동 | 재사용 데이터 |
|---|---|---|
| PART 1 (7~13) | 환경·필수 파일 확인 | Day05~08 산출물 |
| PART 2 (14~21) | Version / Manifest | 기존 Config·Artifact·Result |
| PART 3 (22~28) | One-shot / Few-shot 재현성 Audit | Day08 공통 Query 결과 |
| PART 4 (29~38) | 한 변수 Experiment | Day05 Calibration |
| PART 5 (39~47) | 밝기 Drift 측정 | Day07 정상 Test·Frozen Rule |
| **PART 6 (48~56)** | **One-shot / Few-shot OR·AND Hybrid 성능 비교** | **Day08 동일 Query 20장 예측 CSV** |
| PART 7 (57~62) | 상황별 기술 선택 Challenge | 앞 수업 개념 |
| PART 8 (63~67) | Blur Drift Mini Challenge | Day07 정상 Test |
| PART 9 (68~81) | 결과 정리·Git·10일차 연결 | Day09 결과 |

### 반드시 사용할 8일차 자료

```text
samples/day08/common_query/
configs/day08_one_shot_rules.json
configs/day08_fewshot_config.json
artifacts/day08/one_shot_reference.npy
artifacts/day08/fewshot_prototypes.npz
reports/day08_one_shot_results.csv
reports/day08_fewshot_results.csv
reports/day08_method_comparison.csv
```

**오늘의 원칙:** 이전 결과를 덮어쓰지 않습니다. 같은 데이터에서 비교하며, Test 결과를 보고 Threshold나 OR/AND 정책을 새로 맞추지 않습니다.

![[Pasted image 20260913212317.png]]

---

# MLOps · 실험관리 · Hybrid 성능 비교: 파일별 역할 및 설명

|파일명|역할|설명|
|---|---|---|
|`70_day09_check_env.py`|이전 실습 환경·산출물 확인|1~8일차의 주요 결과 파일과 폴더가 존재하는지 확인하고, CSV 컬럼·JSON 설정값 등이 정상인지 검사합니다.|
|`71_day09_build_version_manifest.py`|Version Manifest 생성|기존 Dataset·Config·Artifact·Result의 경로와 Hash, Git Commit을 기록하여 검사 결과의 출처를 추적할 수 있도록 합니다.|
|`72_day09_audit_day08_results.py`|8일차 결과 재현성 검증|One-shot과 Few-shot이 동일한 Query를 사용했는지 확인하고, 저장된 CSV에서 Accuracy·Precision·Recall·FP·FN을 다시 계산하여 기존 결과와 비교합니다.|
|`73_day09_make_experiment_configs.py`|Baseline·Candidate 설정 생성|5일차 광천김의 기존 Rule을 보존하면서 Calibration 데이터를 기준으로 `inlier_ratio_min`만 변경한 새로운 실험 설정을 만듭니다.|
|`74_day09_run_experiments.py`|설정별 검사 실험|동일한 5일차 Calibration 데이터에 Baseline과 Candidate Rule을 각각 적용하여 검사 성능을 계산하고 결과를 기록합니다.|
|`75_day09_compare_experiments.py`|실험 성능 비교|두 실험의 Accuracy·Precision·Recall·FP·FN을 하나의 비교표로 정리하여 설정 변경에 따른 성능 차이를 확인합니다.|
|`76_day09_make_drift_dataset.py`|밝기 변화 데이터 생성|7일차 정상 병 이미지 20장을 준비하고, 같은 이미지의 밝기를 낮춘 변형본을 만들어 Data Drift 비교용 이미지 쌍을 생성합니다.|
|`77_day09_measure_drift.py`|Data Drift 영향 측정|기존 ResNet18 정상 기준과 Threshold를 유지한 채 원본·밝기 변경 이미지의 Anomaly Score와 FP Rate 변화를 측정합니다.|
|`78_day09_make_hybrid_demo.py`|Hybrid 비교 데이터 준비|8일차 One-shot·Few-shot의 실제 검사 결과 CSV를 동일한 `sample_id`로 연결하고, OR·AND 판정 정책을 준비합니다.|
|`day09_hybrid_core.py`|Hybrid 판정 공통 함수|One-shot과 Few-shot의 예측을 받아 OR·AND 규칙으로 최종 Normal / Anomaly를 결정하고 판정 이유를 반환하는 공통 함수를 제공합니다.|
|`79_day09_run_hybrid_pipeline.py`|Hybrid 성능 검증|동일한 Query 20장에서 One-shot·Few-shot 단독 검사와 OR·AND Hybrid의 Accuracy·Precision·Recall·FP·FN을 비교합니다.|
|`day09_mini_drift.py`|Blur Drift Mini Challenge|학생이 정상 이미지에 Gaussian Blur를 적용하고, 기존 검사 기준을 유지하면서 Score와 FP Rate 변화를 직접 분석하는 자율 실습 파일입니다.|

![[Pasted image 20261020094942.png]]

![[Pasted image 20261020100010.png]]


---
### 프로젝트에 추가하는 파일

```text
subject9_machinevision/
├── configs/day09/
│   ├── exp-001.json
│   ├── exp-002.json
│   └── hybrid_policy.json
├── reports/day09/
│   ├── version_manifest.json
│   ├── day08_reproducibility_audit.csv
│   ├── day08_reproducibility_audit.json
│   ├── experiment_log.csv
│   ├── experiment_comparison.csv
│   ├── drift_results.csv
│   ├── drift_summary.csv
│   ├── hybrid_inputs.csv
│   ├── hybrid_results.csv
│   ├── hybrid_metrics.csv
│   ├── hybrid_comparison.md
│   ├── day09_tech_matrix.md
│   ├── day09_mini_challenge.md
│   └── day09_analysis.md
└── src/
    ├── 70_day09_check_env.py
    ├── 71_day09_build_version_manifest.py
    ├── 72_day09_audit_day08_results.py
    ├── 73_day09_make_experiment_configs.py
    ├── 74_day09_run_experiments.py
    ├── 75_day09_compare_experiments.py
    ├── 76_day09_make_drift_dataset.py
    ├── 77_day09_measure_drift.py
    ├── 78_day09_make_hybrid_demo.py
    ├── day09_hybrid_core.py
    ├── 79_day09_run_hybrid_pipeline.py
    └── day09_mini_drift.py
```

---

# PART1 — 9일차 시작 + 1~8일차 실제 결과 Audit + 환경 점검 (7~13)

# 7. 기존 프로젝트를 그대로 엽니다

프로젝트 폴더로 이동합니다.

```bash
cd ~/ai_vision/subject9_machinevision
```

가상환경을 활성화합니다.

```bash
source .venv/bin/activate
```

Python을 확인합니다.

```bash
which python
```

VS Code를 엽니다.

```bash
code -r .
```

9일차에서도 새 프로젝트나 새 `.venv`를 만들지 않습니다.

1일차부터 같은 프로젝트와 같은 환경을 이어서 사용하는 것이 정상입니다.

---

# 8. 9일차 폴더를 준비합니다

9일차에서 새로 만드는 파일만 별도 폴더에 저장합니다.

```bash
mkdir -p configs/day09

mkdir -p samples/day09/drift/baseline
mkdir -p samples/day09/drift/lighting

mkdir -p samples/day09/mini_drift/baseline
mkdir -p samples/day09/mini_drift/blur

mkdir -p outputs/day09/drift
mkdir -p outputs/day09/mini_drift

mkdir -p reports/day09
```

기존 Day05~Day08 폴더의 파일을 덮어쓰지 않습니다.

```text
Day05 ~ Day08
→ 읽기 전용으로 재사용

Day09
→ 새 Config / Result만 생성
```

---

# 9. 코드를 작성하기 전에 1~8일차 결과가 왜 필요한지 확인합니다

9일차에서는 새로운 검사 방법을 처음부터 다시 만들지 않습니다. 

앞에서 만든 결과를 가져와 비교하고, 확인하고, 관리하는 실습을 합니다.

```text
Day05
Calibration + Frozen Rule
        ↓
One-variable Experiment

Day06
OCR Rule + OCR Result
        ↓
Version·Result 이력 점검

Day07
Normal Reference + Frozen Threshold
        ↓
Drift

Day08
One-shot / Few-shot
Config + Artifact + Result
        ↓
Reproducibility Audit
```

그렇다고 9일차가 Day05부터 갑자기 시작되는 것은 아닙니다.

Day01~Day04에서 먼저 검사에 필요한 기본 과정을 만들었습니다.

```text
Day01
검사할 영역 ROI 결정
        ↓
Day02
영상 품질과 전처리 확인
        ↓
Day03
Feature를 숫자로 측정하고 Rule 생성
        ↓
Day04
로고와 형태를 Matching으로 비교
        ↓
Day05 이후
실제 검사 기준과 결과 생성
```

즉 전체 흐름은 이렇게 이해하면 됩니다.

```
Day01~04
검사 방법의 기반 만들기
        ↓
Day05~08
실제 검사 기준과 결과 만들기
        ↓
Day09
그 결과를 다시 확인하고
비교하고 관리하기
```

---

# 10. `70_day09_check_env.py` — 이전 결과가 제대로 준비되어 있는지 먼저 확인합니다

9일차에서는 Day05 ~ Day08에서 만든 파일을 다시 사용합니다.

따라서 코드를 바로 실행하기 전에 필요한 파일이 실제로 존재하고, 안의 구조도 예상한 형태인지 먼저 확인해야 합니다.

```text
필요한 파일이 없음 
CSV는 있는데 필요한 컬럼이 없음 
JSON은 있는데 필요한 Key가 없음 
Day07 Reference 파일이 없음 
Day08 One-shot / Few-shot Result가 없음 
Day08 Artifact가 없음
```

이런 오류를 실습 중간에 발견하면  
어느 단계에서 문제가 생겼는지 찾느라 시간이 많이 걸릴 수 있습니다.

그래서 9일차의 첫 번째 코드에서는 먼저 다음을 확인합니다.

```
이전 결과 파일이 있는가?
        ↓
필요한 CSV 컬럼이 있는가?
        ↓
필요한 JSON Key가 있는가?
        ↓
다음 실습에서 실제로 사용할 수 있는 상태인가?
```

즉 이 코드는 새로운 검사를 수행하는 코드가 아니라:

> 9일차 실습을 시작하기 전에 필요한 재료가 모두 준비되어 있는지 자동으로 점검하는 시작 전 검사 코드입니다.

파일:

```text
src/70_day09_check_env.py
```

작성합니다.

```python
from pathlib import Path
import csv
import json
import subprocess


# --------------------------------------------------
# 1. 전체 학습 이력 확인용
#    없어도 오늘 계산을 즉시 중단하지는 않지만
#    이전 결과가 어디까지 남아 있는지 확인합니다.
# --------------------------------------------------
HISTORY_FILES = [
    Path("reports/day01_problem_definition.md"),
    Path("reports/day02_parameter_table.csv"),
    Path("reports/day03_inspection_results.csv"),
    Path("reports/day04_homography_results.csv"),
]


# --------------------------------------------------
# 2. Day09 핵심 실습에 필요한 필수 파일
# --------------------------------------------------
REQUIRED_FILES = [
    # Day05
    Path("configs/day05_logo_rules.json"),
    Path("reports/day05_calibration_measurements.csv"),

    # Day06
    Path("configs/day06_ocr_rules.json"),
    Path("configs/day06_labels.csv"),
    Path("reports/day06_ocr_results.csv"),

    # Day07
    Path("configs/day07_anomaly_rules.json"),
    Path("artifacts/day07/normal_reference.npz"),
    Path("reports/day07_anomaly_scores.csv"),

    # Day08 CORE
    Path("configs/day08_one_shot_rules.json"),
    Path("configs/day08_fewshot_config.json"),
    Path("artifacts/day08/one_shot_reference.npy"),
    Path("artifacts/day08/fewshot_prototypes.npz"),
    Path("reports/day08_one_shot_results.csv"),
    Path("reports/day08_fewshot_results.csv"),
    Path("reports/day08_method_comparison.csv"),
]


REQUIRED_DIRS = [
    Path("samples/day07/test/normal"),
    Path("samples/day08/common_query/normal"),
    Path("samples/day08/common_query/anomaly/broken_small"),
]


DAY05_REQUIRED_COLUMNS = {
    "file",
    "actual",
    "template_score",
    "sift_good_matches",
    "homography_found",
    "homography_inliers",
    "inlier_ratio",
}


DAY05_REQUIRED_RULE_KEYS = {
    "homography_found",
    "sift_good_matches_min",
    "inlier_ratio_min",
}


DAY06_REQUIRED_COLUMNS = {
    "file",
    "actual_text",
    "expected_text",
    "actual_status",
    "final_text",
    "format_ok",
    "expected_match",
    "predicted_status",
    "reasons",
}


DAY08_RESULT_COLUMNS = {
    "sample_id",
    "file",
    "actual",
    "predicted",
    "correct",
}


DAY08_COMPARISON_COLUMNS = {
    "method",
    "samples",
    "accuracy",
    "anomaly_precision",
    "anomaly_recall",
    "tp",
    "tn",
    "fp",
    "fn",
}


def read_csv_columns(path):
    with path.open(
        "r",
        newline="",
        encoding="utf-8-sig",
    ) as file:
        reader = csv.DictReader(file)
        return set(reader.fieldnames or [])


print("=" * 70)
print("DAY09 ENVIRONMENT CHECK")
print("=" * 70)


# --------------------------------------------------
# 3. Day01~04 학습 이력 확인
# --------------------------------------------------
print("\n[Day01~04 History]")

for path in HISTORY_FILES:
    status = "OK" if path.exists() else "HISTORY MISSING"
    print(f"[{status}] {path}")


# --------------------------------------------------
# 4. Day05~08 필수 파일 확인
# --------------------------------------------------
missing_required = []

print("\n[Required Files]")

for path in REQUIRED_FILES:
    if path.exists():
        print("[OK]     ", path)
    else:
        print("[MISSING]", path)
        missing_required.append(path)


print("\n[Required Directories]")

for path in REQUIRED_DIRS:
    file_count = (
        len(
            [
                p
                for p in path.rglob("*")
                if p.is_file()
            ]
        )
        if path.exists()
        else 0
    )

    if path.exists() and file_count > 0:
        print(
            f"[OK]      {path}  files={file_count}"
        )
    else:
        print(f"[MISSING] {path}")
        missing_required.append(path)


if missing_required:
    print("\n필수 파일 또는 폴더가 부족합니다.")

    for path in missing_required:
        print("-", path)

    raise RuntimeError(
        "Day05~Day08 CORE 산출물을 먼저 확인하세요."
    )


# --------------------------------------------------
# 5. Day05 Calibration CSV Schema
# --------------------------------------------------
day05_columns = read_csv_columns(
    Path(
        "reports/day05_calibration_measurements.csv"
    )
)

missing = DAY05_REQUIRED_COLUMNS - day05_columns

if missing:
    raise RuntimeError(
        "Day05 Calibration CSV 컬럼이 부족합니다: "
        f"{sorted(missing)}"
    )

print("\n[OK] Day05 Calibration CSV Schema")


# --------------------------------------------------
# 6. Day05 Frozen Rule Schema
# --------------------------------------------------
day05_rule = json.loads(
    Path(
        "configs/day05_logo_rules.json"
    ).read_text(
        encoding="utf-8"
    )
)

required_rules = day05_rule.get(
    "required_rules",
    {},
)

missing = (
    DAY05_REQUIRED_RULE_KEYS
    - set(required_rules.keys())
)

if missing:
    raise RuntimeError(
        "Day05 Rule Key가 부족합니다: "
        f"{sorted(missing)}"
    )

print("[OK] Day05 Frozen Rule Schema")


# --------------------------------------------------
# 7. Day06 OCR Result Schema
# --------------------------------------------------
day06_columns = read_csv_columns(
    Path("reports/day06_ocr_results.csv")
)

missing = DAY06_REQUIRED_COLUMNS - day06_columns

if missing:
    raise RuntimeError(
        "Day06 OCR Result 컬럼이 부족합니다: "
        f"{sorted(missing)}"
    )

print("[OK] Day06 OCR Result Schema")


# --------------------------------------------------
# 8. Day07 Frozen Rule + Reference
# --------------------------------------------------
day07_rule = json.loads(
    Path(
        "configs/day07_anomaly_rules.json"
    ).read_text(
        encoding="utf-8"
    )
)

if "anomaly_threshold" not in day07_rule:
    raise KeyError(
        "Day07 Rule에 anomaly_threshold가 없습니다."
    )

reference_path = Path(
    day07_rule.get(
        "reference_path",
        "artifacts/day07/normal_reference.npz",
    )
)

if not reference_path.exists():
    raise FileNotFoundError(
        reference_path
    )

print("[OK] Day07 Frozen Rule / Reference")


# --------------------------------------------------
# 9. Day08 One-shot / Few-shot Result Schema
# --------------------------------------------------
for result_path in [
    Path("reports/day08_one_shot_results.csv"),
    Path("reports/day08_fewshot_results.csv"),
]:
    columns = read_csv_columns(
        result_path
    )

    missing = DAY08_RESULT_COLUMNS - columns

    if missing:
        raise RuntimeError(
            f"{result_path} 컬럼이 부족합니다: "
            f"{sorted(missing)}"
        )

print("[OK] Day08 One/Few-shot Result Schema")


comparison_columns = read_csv_columns(
    Path(
        "reports/day08_method_comparison.csv"
    )
)

missing = (
    DAY08_COMPARISON_COLUMNS
    - comparison_columns
)

if missing:
    raise RuntimeError(
        "Day08 Method Comparison 컬럼이 부족합니다: "
        f"{sorted(missing)}"
    )

print("[OK] Day08 Method Comparison Schema")


# --------------------------------------------------
# 10. Day08 Common Query 기본 수량 확인
# --------------------------------------------------
normal_count = len(
    list(
        Path(
            "samples/day08/common_query/normal"
        ).glob("*.png")
    )
)

anomaly_count = len(
    list(
        Path(
            "samples/day08/common_query/"
            "anomaly/broken_small"
        ).glob("*.png")
    )
)

print()
print("Day08 Common Query")
print("Normal :", normal_count)
print("Anomaly:", anomaly_count)

if normal_count != 10 or anomaly_count != 10:
    raise RuntimeError(
        "Day08 Common Query는 현재 수업 기준 "
        "Normal 10장 + Anomaly 10장을 예상합니다."
    )


# --------------------------------------------------
# 11. Git Commit 확인
# --------------------------------------------------
try:
    commit = subprocess.check_output(
        ["git", "rev-parse", "--short", "HEAD"],
        text=True,
        stderr=subprocess.DEVNULL,
    ).strip()

except Exception:
    commit = "NO_GIT"


print()
print("Git Commit:", commit)
print("\nDay09 Environment OK")
```

실행합니다.

```bash
python src/70_day09_check_env.py
```

### 실행 결과 해석

```
Day01~04 History
→ Day02 대표 Report 1개만 없음

Day05~08 Required Files
→ 모두 OK

Required Directories
→ 모두 OK

Day05~08 Schema
→ 모두 OK

Day08 Common Query
→ Normal 10장
→ Anomaly 10장

Git Commit
→ NO_GIT

최종
→ Day09 Environment OK
```

### 위의 결과를 쉽게 해석하면

9일차 핵심 실습에 필요한 Day05 ~ Day08의 파일과 데이터 구조는 모두 정상입니다.

즉:

```
Experiment에 필요한 Day05 결과
→ 준비 완료

이력 관리에 필요한 Day06 OCR 결과
→ 준비 완료

Drift에 필요한 Day07 결과
→ 준비 완료

Reproducibility에 필요한 Day08 결과
→ 준비 완료
```

Day02의 `HISTORY MISSING`은 과거 이력 확인용 파일이므로  
현재 9일차 핵심 실습을 막는 문제는 아닙니다.

다만:

```
Git Commit: NO_GIT
```

으로 표시되었으므로,  
Code Version을 기록하기 전에 Git 상태는 따로 확인해야 합니다.

## 내 결과는 직접 확인합니다

각자의 결과는 강사 결과와 다를 수 있습니다.

이전 실습을 일부 완료하지 않았거나,  
파일을 다른 이름으로 저장했거나,  
중간 오류를 해결하지 않고 넘어간 경우에는 `MISSING`이나 Schema 오류가 나올 수 있습니다.

따라서 강사 결과를 그대로 맞추려고 하지 말고 다음을 확인합니다.

```
MISSING이 있는가?
        ↓
어느 Day의 어떤 파일인가?

Schema 오류가 있는가?
        ↓
CSV 컬럼 또는 JSON Key가 다른가?

이미지 수가 다른가?
        ↓
이전 Dataset 구성이 제대로 되었는가?

Git Commit이 NO_GIT인가?
        ↓
Git 저장소와 Commit 상태를 확인
```

부족한 항목이 있다면 그 파일을 억지로 새로 만들지 말고, 해당 이전 일차로 돌아가 왜 결과가 만들어지지 않았는지 먼저 확인한 뒤 다시 실행합니다.

### 의사코드

```text
1. Day01~04의 대표 결과가 남아 있는지 이력으로 확인한다.

2. Day05~08에서
   오늘 직접 사용할 필수 파일이 있는지 확인한다.

3. Day05 Calibration CSV의 실제 컬럼을 확인한다.

4. Day05 Frozen Rule의 필수 Key를 확인한다.

5. Day06 OCR Result의 실제 컬럼을 확인한다.

6. Day07 Threshold와 Reference를 확인한다.

7. Day08 One-shot / Few-shot Result의
   sample_id / actual / predicted 등의 컬럼을 확인한다.

8. Day08 Method Comparison의
   Accuracy / Precision / Recall / FP / FN 컬럼을 확인한다.

9. Day08 Common Query가
   Normal 10장 + Anomaly 10장인지 확인한다.

10. 현재 Git Commit을 확인한다.

11. 필수 조건이 하나라도 틀리면
    뒤 실습을 시작하기 전에 중단한다.

12. 모두 맞으면
    Day09 Environment OK를 출력한다.
```

---

# 11. 왜 파일이 있는지만 확인하면 안 되나요?

파일이 존재한다고 해서 바로 사용할 수 있는 것은 아닙니다. 

파일 안에 필요한 항목이 제대로 들어 있는지도 확인해야 합니다. 이때 파일 안의 구조와 항목을 `Schema`라고 합니다. 

쉽게 말하면:

```text
Schema는 “이 파일 안에는 어떤 항목이 들어 있어야 하는가?”를 정한 약속입니다.
```

예를 들어:

```text
reports/day08_one_shot_results.csv
```

파일이 있어도 `sample_id` 컬럼이 없다면  
One-shot과 Few-shot이 같은 이미지를 검사했는지 비교할 수 없습니다.

또:

```text
configs/day05_logo_rules.json
```

파일이 있어도 `inlier_ratio_min` 값이 없다면  
뒤의 Experiment에서 필요한 기준을 읽을 수 없습니다.

즉 다음처럼 확인합니다.

```text
파일이 있는가?
        ↓
파일 안에 필요한 컬럼 / Key가 있는가?
        ↓
뒤 실습에서 실제로 사용할 수 있는가?
```

비유하면:

```
서류가 책상 위에 있다
        ↓
끝 X

서류가 있다
        ↓
필수 항목이 모두 작성되어 있다
        ↓
실제로 사용할 수 있다
O
```

그래서 9일차 시작 단계에서는  
파일 존재 여부와 함께 Schema까지 확인합니다.

---

# 12. 왜 앞에서 만든 검사 기준을 그대로 사용하나요?

![[Pasted image 20260913221241.png]]

7일차에서는 정상 이미지를 이용하여 이상검출에 사용할 검사 기준을 이미 만들었습니다.

```
normal_reference.npz
→ 정상 제품의 기준

anomaly_rules.json
→ 정상 / 이상을 나누는 기준값
```

9일차에서는 이 기준을 다시 만들지 않고 그대로 사용합니다.

이번에 확인하려는 것은 검사 기준 자체가 아니라  
조명이나 밝기처럼 입력 환경이 달라졌을 때 검사 결과가 어떻게 변하는가이기 때문입니다.

```text
검사 기준
→ 그대로 유지 🔒

입력 환경
→ 밝기 변경

        ↓

검사 결과가 달라지는지 확인
        ↓
Data Drift 분석
```

만약 검사 기준과 밝기를 동시에 바꾸면 결과가 왜 달라졌는지 알기 어렵습니다.

따라서:

> 비교하려는 조건 하나만 바꾸고, 나머지 기준은 그대로 유지합니다.

8일차의 One-shot Reference와 Few-shot Prototype도 같은 원칙으로  
새로 만들지 않고 기존에 저장된 파일을 그대로 사용합니다.

> 9일차는 검사기를 다시 만드는 수업이 아니라, 이미 만든 검사 기준과 결과를 보존한 상태에서 비교하고 관리하는 수업입니다.

---

# 13. Git 상태를 시작 전에 확인합니다

```bash
git status
```

현재 Commit을 확인합니다.

```bash
git log -1 --oneline
```

9일차에서 Code Version은 이 Git Commit과 연결됩니다.

아직 Commit하지 않은 변경이 많다면 어떤 상태에서 실험을 실행했는지 나중에 설명하기 어렵습니다.

---

# PART2 — Version / Lineage + Version Manifest (14~21)

# 14. Version은 결과의 출처를 설명하기 위한 정보입니다

예를 들어 다음 결과가 있다고 생각해 봅니다.

```text
Accuracy = 90%
```

숫자만 남아 있으면 다음 질문에 답하기 어렵습니다.

```text
어떤 Dataset이었지?
어떤 Query였지?
어떤 Rule이었지?
어떤 Reference / Prototype이었지?
어떤 코드였지?
```

그래서 결과와 조건을 함께 기록합니다.

---

# 15. 오늘 관리할 다섯 가지 Version

| 구분 | 질문 | 현재 교과9 예 |
|---|---|---|
| Dataset Version | 어떤 이미지였는가? | `samples/day08/common_query` |
| Config Version | 어떤 Rule/Parameter였는가? | `day08_one_shot_rules.json` |
| Artifact Version | 어떤 기준 파일이었는가? | `one_shot_reference.npy` |
| Result Version | 어떤 실행 결과인가? | `day08_one_shot_results.csv` |
| Code Version | 어느 코드 상태인가? | Git Commit |

Rule 기반 검사에도 Version 관리가 필요합니다.

Model Weight가 없어도 Threshold, ROI, Regex, Reference, Prototype이 바뀌면 결과가 달라질 수 있기 때문입니다.

---

# 16. Hash는 파일 이름이 아니라 내용의 변화를 확인합니다

파일명이 같아도 내용은 바뀔 수 있습니다.

```text
day07_anomaly_rules.json
파일명 동일

하지만

threshold 변경
        ↓
내용 변경
```

오늘은 파일 내용을 SHA256으로 계산하여 기록합니다.

쉽게 말하면 Hash는:

> **이 파일의 내용이 그때와 같은가?**

를 확인하는 지문입니다.

---

# 17. Dataset은 교육용 Fingerprint로 기록합니다

Dataset은 파일이 여러 장이므로 하나의 파일처럼 Hash하기 어렵습니다.

오늘은 각 파일의:

```text
상대 경로
+
파일 크기
```

를 정렬하여 이어 붙인 뒤 SHA256을 계산합니다.

```text
Dataset 폴더
        ↓
파일 목록 정렬
        ↓
relative_path | size
        ↓
SHA256
        ↓
Dataset Fingerprint
```

이것은 빠르게 Dataset 구성이 바뀌었는지 확인하는 교육용 방식입니다.

> 각 이미지 파일 내용 전체를 Hash하는 강한 무결성 검증과는 다릅니다. 오늘은 **구성 변화 추적**을 위한 간단한 Fingerprint로 사용합니다.

---

# 18. Lineage는 결과가 어디에서 왔는지 연결하는 관계입니다

9일차에서 `Lineage`는 어렵게 생각하지 않습니다.

예를 들어 Day08 One-shot 결과는 다음 관계를 가지고 있습니다.

```text
samples/day08/common_query
        +
configs/day08_one_shot_rules.json
        +
artifacts/day08/one_shot_reference.npy
        +
src/65_day08_test_one_shot.py
        ↓
reports/day08_one_shot_results.csv
```

Few-shot도 같은 방식입니다.

```text
samples/day08/common_query
        +
configs/day08_fewshot_config.json
        +
artifacts/day08/fewshot_prototypes.npz
        +
src/67_day08_test_fewshot.py
        ↓
reports/day08_fewshot_results.csv
```

이 연결을 알 수 있어야:

```text
왜 이 결과가 나왔는가?
```

를 다시 설명할 수 있습니다.

---

# 19. `71_day09_build_version_manifest.py` — 1~8일차의 대표 근거를 한 파일에 기록합니다

Manifest는 결과를 다시 계산하는 프로그램이 아닙니다.

```text
어디에 무엇이 있었는가?
파일 내용 지문은 무엇인가?
Dataset 구성은 무엇인가?
어느 Git Commit인가?
```

를 기록하는 지도입니다.

파일:

```text
src/71_day09_build_version_manifest.py
```

작성합니다.

```python
from datetime import datetime
from pathlib import Path
import hashlib
import json
import subprocess


OUTPUT_PATH = Path(
    "reports/day09/version_manifest.json"
)


# --------------------------------------------------
# 1. Dataset / Image Group
# --------------------------------------------------
DATASETS = {
    "day02_preprocess_input":
        Path("samples/day02"),

    "day03_cap_inspection":
        Path("samples/day03"),

    "day04_logo_matching":
        Path("samples/day04"),

    "day05_logo_calibration_test":
        Path("samples/day05/resized"),

    "day06_ocr_product":
        Path("samples/day06/product"),

    "day07_normal_test":
        Path("samples/day07/test/normal"),

    "day08_common_query":
        Path("samples/day08/common_query"),
}


# --------------------------------------------------
# 2. Config
# --------------------------------------------------
CONFIG_FILES = [
    Path("configs/day05_logo_rules.json"),
    Path("configs/day06_ocr_rules.json"),
    Path("configs/day06_labels.csv"),
    Path("configs/day07_anomaly_rules.json"),
    Path("configs/day08_one_shot_rules.json"),
    Path("configs/day08_fewshot_config.json"),
]


# --------------------------------------------------
# 3. Artifact
# --------------------------------------------------
ARTIFACT_FILES = [
    Path("artifacts/day07/normal_reference.npz"),
    Path("artifacts/day08/one_shot_reference.npy"),
    Path("artifacts/day08/fewshot_prototypes.npz"),
]


# --------------------------------------------------
# 4. Result Source
# --------------------------------------------------
RESULT_SOURCE_FILES = [
    # Day01~04
    Path("reports/day01_problem_definition.md"),
    Path("reports/day02_parameter_table.csv"),
    Path("reports/day02_preprocess_reason.md"),
    Path("reports/day03_inspection_results.csv"),
    Path("reports/day04_homography_results.csv"),

    # Day05
    Path("reports/day05_calibration_measurements.csv"),
    Path("reports/day05_test_results.csv"),

    # Day06
    Path("reports/day06_ocr_results.csv"),
    Path("reports/day06_ocr_metrics.md"),

    # Day07
    Path("reports/day07_anomaly_scores.csv"),
    Path("reports/day07_test_metrics.md"),

    # Day08
    Path("reports/day08_one_shot_results.csv"),
    Path("reports/day08_fewshot_results.csv"),
    Path("reports/day08_method_comparison.csv"),
]


def file_sha256(path):
    sha = hashlib.sha256()

    with path.open("rb") as file:
        while True:
            chunk = file.read(
                1024 * 1024
            )

            if not chunk:
                break

            sha.update(chunk)

    return sha.hexdigest()


def directory_summary(directory):
    if not directory.exists():
        return {
            "exists": False,
            "file_count": 0,
            "fingerprint": None,
        }

    files = sorted(
        path
        for path in directory.rglob("*")
        if path.is_file()
    )

    records = []

    for path in files:
        relative = path.relative_to(
            directory
        )

        records.append(
            f"{relative.as_posix()}|"
            f"{path.stat().st_size}"
        )

    joined = "\n".join(
        records
    )

    fingerprint = hashlib.sha256(
        joined.encode("utf-8")
    ).hexdigest()

    return {
        "exists": True,
        "file_count": len(files),
        "fingerprint": fingerprint,
    }


def file_summary(path):
    return {
        "exists": path.exists(),
        "size":
            path.stat().st_size
            if path.exists()
            else None,
        "sha256":
            file_sha256(path)
            if path.exists()
            else None,
    }


try:
    git_commit = subprocess.check_output(
        [
            "git",
            "rev-parse",
            "HEAD",
        ],
        text=True,
        stderr=subprocess.DEVNULL,
    ).strip()

except Exception:
    git_commit = "NO_GIT"


manifest = {
    "created_at":
        datetime.now().isoformat(
            timespec="seconds"
        ),

    "git_commit":
        git_commit,

    "datasets": {},
    "configs": {},
    "artifacts": {},
    "result_sources": {},
}


for name, directory in DATASETS.items():
    manifest["datasets"][name] = (
        directory_summary(
            directory
        )
    )


for path in CONFIG_FILES:
    manifest["configs"][
        str(path)
    ] = file_summary(
        path
    )


for path in ARTIFACT_FILES:
    manifest["artifacts"][
        str(path)
    ] = file_summary(
        path
    )


for path in RESULT_SOURCE_FILES:
    manifest["result_sources"][
        str(path)
    ] = file_summary(
        path
    )


OUTPUT_PATH.parent.mkdir(
    parents=True,
    exist_ok=True,
)

OUTPUT_PATH.write_text(
    json.dumps(
        manifest,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)


print("=" * 70)
print("DAY09 VERSION MANIFEST")
print("=" * 70)

print(
    "Git Commit:",
    git_commit,
)

for name, info in (
    manifest["datasets"].items()
):
    print(
        "Dataset:",
        name,
        "exists=",
        info["exists"],
        "files=",
        info["file_count"],
    )

print(
    "Saved:",
    OUTPUT_PATH,
)
```

실행합니다.

```bash
python src/71_day09_build_version_manifest.py
```

---

# 20. Version Manifest를 확인합니다

```bash
cat reports/day09/version_manifest.json
```

다음 구조가 들어 있습니다.

```text
created_at
git_commit

datasets
→ exists
→ file_count
→ fingerprint

configs
→ exists
→ size
→ sha256

artifacts
→ exists
→ size
→ sha256

result_sources
→ exists
→ size
→ sha256
```

Day01~04의 오래된 대표 Report가 일부 없다면 `exists=false`로 기록될 수 있습니다.

하지만 PART1에서 필수로 확인한 Day05~Day08 CORE가 없다면 그 상태로 뒤 실습을 계속하지 않습니다.

이 차이를 구분합니다.

```text
과거 전체 이력
→ Manifest에 현재 상태 기록

오늘 직접 계산에 필요한 CORE
→ Environment Check에서 필수
```

---

# 21. Manifest는 결과 자체가 아니라 결과를 찾기 위한 지도입니다

Manifest만 보고 Day08 Accuracy가 85%인지 90%인지 알 수 있는 것은 아닙니다.

Manifest가 알려주는 것은:

```text
어떤 Dataset
어떤 Config
어떤 Artifact
어떤 Result
어떤 Code Version
```

이 연결입니다.

실제 Metric은 Result CSV에서 확인합니다.

다음 PART에서는 바로 이 점을 확인합니다.

```text
Manifest
→ "무엇으로 만들었는가?"

Day08 Reproducibility Audit
→ "저장된 결과에서 같은 Metric을 다시 계산할 수 있는가?"
```

---

# PART3 — Day08 Result Reproducibility Audit (22~28)

# 22. 왜 8일차 결과를 9일차에서 다시 확인하나요?

8일차에서는 One-shot과 Few-shot 방법 자체를 실습했습니다.

9일차에서는 같은 알고리즘을 다시 학습하거나 같은 Feature를 다시 추출하는 것이 목적이 아닙니다.

오늘 확인할 것은 다음입니다.

```text
One-shot 결과
85%

Few-shot 결과
90%
```

라는 숫자가 남아 있을 때:

```text
두 방법이 같은 Query를 사용했는가?

저장된 CSV만으로
Accuracy / Precision / Recall / FP / FN을
다시 계산할 수 있는가?

저장된 Method Comparison과
다시 계산한 값이 같은가?

어떤 Config와 Artifact가
이 결과의 기준이었는가?
```

를 확인하는 것입니다.

8일차와 9일차를 한 문장으로 구분하면:

> **8일차는 방법을 실험하는 날이고, 9일차는 그 실험 결과의 근거를 다시 추적하는 날입니다.**

---

# 23. Day08에서 연결할 파일을 확인합니다

One-shot:

```text
Dataset
samples/day08/common_query

        +

Config
configs/day08_one_shot_rules.json

        +

Artifact
artifacts/day08/one_shot_reference.npy

        +

Result
reports/day08_one_shot_results.csv
```

Few-shot:

```text
Dataset
samples/day08/common_query

        +

Config
configs/day08_fewshot_config.json

        +

Artifact
artifacts/day08/fewshot_prototypes.npz

        +

Result
reports/day08_fewshot_results.csv
```

두 방법의 결과를 요약한 파일:

```text
reports/day08_method_comparison.csv
```

이 있습니다.

8일차에서 실제로 사용한 Common Query는:

```text
Normal 10장
+
Anomaly / broken_small 10장

총 20장
```

이었습니다.

두 결과 CSV의 `sample_id`가 같아야 공정한 비교입니다.

---

# 24. Audit에서는 무엇을 다시 계산하나요?

One-shot과 Few-shot 각각에서:

```text
actual
+
predicted
        ↓
TP / TN / FP / FN
        ↓
Accuracy
Anomaly Precision
Anomaly Recall
```

을 다시 계산합니다.

Positive는 8일차와 동일하게 `Anomaly`로 둡니다.

```text
TP
실제 Anomaly
예측 Anomaly

TN
실제 Normal
예측 Normal

FP
실제 Normal
예측 Anomaly

FN
실제 Anomaly
예측 Normal
```

그리고 다음 두 값을 비교합니다.

```text
다시 계산한 Metric

        VS

reports/day08_method_comparison.csv
저장 Metric
```

둘이 같아야 합니다.

단, Method Comparison CSV는 8일차에서 소수점 네 자리로 저장했으므로 비교할 때도 같은 자리수로 맞춥니다.

---

# 25. `72_day09_audit_day08_results.py` — 저장된 Day08 결과를 재검증합니다

파일:

```text
src/72_day09_audit_day08_results.py
```

작성합니다.

```python
from pathlib import Path
import csv
import hashlib
import json

import pandas as pd


ONE_RESULT = Path(
    "reports/day08_one_shot_results.csv"
)

FEW_RESULT = Path(
    "reports/day08_fewshot_results.csv"
)

COMPARISON = Path(
    "reports/day08_method_comparison.csv"
)


ONE_CONFIG = Path(
    "configs/day08_one_shot_rules.json"
)

FEW_CONFIG = Path(
    "configs/day08_fewshot_config.json"
)

ONE_ARTIFACT = Path(
    "artifacts/day08/one_shot_reference.npy"
)

FEW_ARTIFACT = Path(
    "artifacts/day08/fewshot_prototypes.npz"
)


AUDIT_CSV = Path(
    "reports/day09/"
    "day08_reproducibility_audit.csv"
)

AUDIT_JSON = Path(
    "reports/day09/"
    "day08_reproducibility_audit.json"
)


REQUIRED_RESULT_COLUMNS = {
    "sample_id",
    "actual",
    "predicted",
}


REQUIRED_COMPARISON_COLUMNS = {
    "method",
    "samples",
    "accuracy",
    "anomaly_precision",
    "anomaly_recall",
    "tp",
    "tn",
    "fp",
    "fn",
}


def sha256(path):
    digest = hashlib.sha256()

    with path.open("rb") as file:
        while True:
            chunk = file.read(
                1024 * 1024
            )

            if not chunk:
                break

            digest.update(chunk)

    return digest.hexdigest()


def validate_result_schema(
    df,
    name,
):
    missing = (
        REQUIRED_RESULT_COLUMNS
        - set(df.columns)
    )

    if missing:
        raise RuntimeError(
            f"{name} Result 컬럼이 부족합니다: "
            f"{sorted(missing)}"
        )

    if df["sample_id"].duplicated().any():
        duplicated = (
            df.loc[
                df["sample_id"].duplicated(),
                "sample_id",
            ]
            .astype(str)
            .tolist()
        )

        raise RuntimeError(
            f"{name} sample_id 중복: "
            f"{duplicated}"
        )


def metrics(df):
    actual = (
        df["actual"]
        .astype(str)
        .str.strip()
    )

    predicted = (
        df["predicted"]
        .astype(str)
        .str.strip()
    )

    tp = int(
        (
            (actual == "Anomaly")
            & (predicted == "Anomaly")
        ).sum()
    )

    tn = int(
        (
            (actual == "Normal")
            & (predicted == "Normal")
        ).sum()
    )

    fp = int(
        (
            (actual == "Normal")
            & (predicted == "Anomaly")
        ).sum()
    )

    fn = int(
        (
            (actual == "Anomaly")
            & (predicted == "Normal")
        ).sum()
    )

    total = (
        tp + tn + fp + fn
    )

    if total != len(df):
        raise RuntimeError(
            "actual / predicted 값에 "
            "Normal 또는 Anomaly가 아닌 값이 있습니다."
        )

    accuracy = (
        (tp + tn) / total
        if total
        else 0.0
    )

    precision = (
        tp / (tp + fp)
        if (tp + fp)
        else 0.0
    )

    recall = (
        tp / (tp + fn)
        if (tp + fn)
        else 0.0
    )

    return {
        "samples": total,
        "accuracy": accuracy,
        "anomaly_precision": precision,
        "anomaly_recall": recall,
        "tp": tp,
        "tn": tn,
        "fp": fp,
        "fn": fn,
    }


# --------------------------------------------------
# 1. Result 읽기
# --------------------------------------------------
one = pd.read_csv(
    ONE_RESULT
)

few = pd.read_csv(
    FEW_RESULT
)

comparison = pd.read_csv(
    COMPARISON
)


validate_result_schema(
    one,
    "One-shot",
)

validate_result_schema(
    few,
    "Few-shot",
)


missing = (
    REQUIRED_COMPARISON_COLUMNS
    - set(comparison.columns)
)

if missing:
    raise RuntimeError(
        "Method Comparison 컬럼이 부족합니다: "
        f"{sorted(missing)}"
    )


# --------------------------------------------------
# 2. 같은 Common Query인지 확인
# --------------------------------------------------
one_ids = set(
    one["sample_id"].astype(str)
)

few_ids = set(
    few["sample_id"].astype(str)
)

same_query = (
    one_ids == few_ids
)

if not same_query:
    only_one = sorted(
        one_ids - few_ids
    )

    only_few = sorted(
        few_ids - one_ids
    )

    raise RuntimeError(
        "One-shot과 Few-shot Query가 다릅니다.\n"
        f"Only One-shot: {only_one}\n"
        f"Only Few-shot: {only_few}"
    )


# --------------------------------------------------
# 3. Metric 재계산
# --------------------------------------------------
recomputed = {
    "one_shot":
        metrics(one),

    "few_shot":
        metrics(few),
}


# --------------------------------------------------
# 4. 저장된 Comparison과 비교
# --------------------------------------------------
comparison = comparison.set_index(
    "method"
)

rows = []
all_metric_match = True

for method in [
    "one_shot",
    "few_shot",
]:
    if method not in comparison.index:
        raise RuntimeError(
            f"Method Comparison에 {method} 행이 없습니다."
        )

    current = recomputed[
        method
    ]

    stored = comparison.loc[
        method
    ]

    checks = {
        "samples_match":
            int(stored["samples"])
            == current["samples"],

        "accuracy_match":
            round(
                float(stored["accuracy"]),
                4,
            )
            == round(
                current["accuracy"],
                4,
            ),

        "precision_match":
            round(
                float(
                    stored[
                        "anomaly_precision"
                    ]
                ),
                4,
            )
            == round(
                current[
                    "anomaly_precision"
                ],
                4,
            ),

        "recall_match":
            round(
                float(
                    stored[
                        "anomaly_recall"
                    ]
                ),
                4,
            )
            == round(
                current[
                    "anomaly_recall"
                ],
                4,
            ),

        "tp_match":
            int(stored["tp"])
            == current["tp"],

        "tn_match":
            int(stored["tn"])
            == current["tn"],

        "fp_match":
            int(stored["fp"])
            == current["fp"],

        "fn_match":
            int(stored["fn"])
            == current["fn"],
    }

    metric_match = all(
        checks.values()
    )

    all_metric_match = (
        all_metric_match
        and metric_match
    )

    rows.append(
        {
            "method":
                method,

            "samples":
                current["samples"],

            "accuracy":
                round(
                    current["accuracy"],
                    4,
                ),

            "anomaly_precision":
                round(
                    current[
                        "anomaly_precision"
                    ],
                    4,
                ),

            "anomaly_recall":
                round(
                    current[
                        "anomaly_recall"
                    ],
                    4,
                ),

            "tp":
                current["tp"],

            "tn":
                current["tn"],

            "fp":
                current["fp"],

            "fn":
                current["fn"],

            "same_common_query":
                same_query,

            "stored_metrics_match":
                metric_match,
        }
    )


# --------------------------------------------------
# 5. Audit CSV 저장
# --------------------------------------------------
AUDIT_CSV.parent.mkdir(
    parents=True,
    exist_ok=True,
)

with AUDIT_CSV.open(
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
# 6. Config / Artifact / Result Hash 기록
# --------------------------------------------------
files_for_hash = [
    ONE_CONFIG,
    FEW_CONFIG,
    ONE_ARTIFACT,
    FEW_ARTIFACT,
    ONE_RESULT,
    FEW_RESULT,
    COMPARISON,
]

hashes = {}

for path in files_for_hash:
    if not path.exists():
        raise FileNotFoundError(
            path
        )

    hashes[str(path)] = sha256(
        path
    )


audit = {
    "same_common_query":
        same_query,

    "query_count":
        len(one_ids),

    "all_metric_match":
        all_metric_match,

    "metrics":
        recomputed,

    "sha256":
        hashes,
}


AUDIT_JSON.write_text(
    json.dumps(
        audit,
        indent=2,
        ensure_ascii=False,
    ),
    encoding="utf-8",
)


print("=" * 70)
print("DAY08 REPRODUCIBILITY AUDIT")
print("=" * 70)

print(
    "Same Common Query:",
    same_query,
)

print(
    "Query Count:",
    len(one_ids),
)

for row in rows:
    print()
    print(
        "Method   :",
        row["method"],
    )

    print(
        "Accuracy :",
        row["accuracy"],
    )

    print(
        "TP/TN/FP/FN:",
        row["tp"],
        row["tn"],
        row["fp"],
        row["fn"],
    )

    print(
        "Stored Metric Match:",
        row[
            "stored_metrics_match"
        ],
    )

print()
print(
    "All Metric Match:",
    all_metric_match,
)

print(
    "Audit CSV :",
    AUDIT_CSV,
)

print(
    "Audit JSON:",
    AUDIT_JSON,
)

if not all_metric_match:
    raise RuntimeError(
        "Day08 저장 Metric과 재계산 Metric이 다릅니다."
    )
```

---

# 26. Audit 코드는 어떤 순서로 동작하나요?

### 의사코드

```text
1. One-shot Result CSV를 읽는다.

2. Few-shot Result CSV를 읽는다.

3. 두 CSV에
   sample_id / actual / predicted가 있는지 확인한다.

4. sample_id 중복이 없는지 확인한다.

5. One-shot sample_id 집합과
   Few-shot sample_id 집합을 비교한다.

6. 서로 다르면 실행을 중단한다.

7. actual / predicted로
   TP / TN / FP / FN을 다시 센다.

8. Accuracy를 다시 계산한다.

9. Anomaly Precision을 다시 계산한다.

10. Anomaly Recall을 다시 계산한다.

11. day08_method_comparison.csv를 읽는다.

12. 저장된 Metric과
    다시 계산한 Metric을 비교한다.

13. Config / Artifact / Result 파일의
    SHA256을 계산한다.

14. 결과를
    day08_reproducibility_audit.csv
    에 저장한다.

15. Query 일치 여부와 Hash를
    day08_reproducibility_audit.json
    에 저장한다.

16. 모든 Metric이 일치하면
    Audit를 완료한다.
```

---

# 27. Audit를 실행하고 실제 결과를 확인합니다

실행합니다.

```bash
python src/72_day09_audit_day08_results.py
```

결과 파일을 확인합니다.

```bash
cat reports/day09/day08_reproducibility_audit.csv
```

```bash
cat reports/day09/day08_reproducibility_audit.json
```

8일차에서 실제로 확인했던 결과는 다음이었습니다.

```text
One-shot
Accuracy 85%
FP 1
FN 2

Few-shot
Accuracy 90%
FP 0
FN 2
```

정상적인 상태라면 현재 Audit에서도 같은 결과가 다시 계산되어야 합니다.

하지만 숫자가 다르다고 해서 문서에 맞추기 위해 CSV를 수정하면 안 됩니다.

먼저 다음을 확인합니다.

```text
Common Query가 바뀌었는가?

Result CSV가 다시 생성되었는가?

Config가 바뀌었는가?

Artifact가 바뀌었는가?

Git Commit이 달라졌는가?
```

이 질문을 하기 위해 Version Manifest와 Hash가 필요합니다.

---

# 28. PART3에서 가장 중요한 것은 결과 숫자가 아니라 연결입니다

One-shot 결과를 다음처럼 설명할 수 있어야 합니다.

```text
Common Query
        +
One-shot Config
        +
One-shot Reference Artifact
        +
Code
        ↓
One-shot Result
        ↓
Metrics
```

Few-shot도 같습니다.

```text
Common Query
        +
Few-shot Config
        +
Few-shot Prototype Artifact
        +
Code
        ↓
Few-shot Result
        ↓
Metrics
```

그리고 두 Result의 `sample_id`가 같아야 다음 비교가 의미가 있습니다.

```text
같은 Query
        ↓
방법만 다름
        ↓
결과 비교 가능
```

> **재현성은 결과가 좋다는 뜻이 아니라, 그 결과가 어떤 조건에서 만들어졌는지 다시 확인할 수 있다는 뜻입니다.**

---

# PART4 — Experiment 관리: Day05 Baseline vs Candidate (29~38)

---

# 29. 오늘 Experiment는 5일차 광천김 Calibration을 재사용합니다

5일차에서는:

```text
Calibration
        ↓
Rule 결정
        ↓
Freeze
        ↓
Test
```

를 수행했습니다.

9일차에서는 이미 만들어진 Baseline을 덮어쓰지 않고 **Candidate Version을 별도로 만들어 비교**합니다.

---

# 30. 왜 공식 Test가 아니라 Calibration을 사용하나요?

오늘 목적은 새로운 최종 Rule을 확정하는 것이 아닙니다.

Experiment 관리 방법을 연습하는 것입니다.

따라서:

```text
Calibration
→ Parameter 비교

Official Test
→ 반복 튜닝에 사용하지 않음
```

원칙을 유지합니다.

---

# 31. 오늘 바꿀 변수는 `inlier_ratio_min` 하나입니다

Day05 Rule에는 여러 값이 있습니다.

```text
homography_found
sift_good_matches_min
inlier_ratio_min
```

5일차 Test에서 정상 3장의 공통 NG Reason이 `LOW_INLIER_RATIO`였기 때문에,
9일차에서는 **Inlier Ratio 기준을 검토할 가치가 있는 변수**로 선택합니다.

하지만 Test에서 본 세 이미지의 Inlier Ratio를 보고 Candidate 값을 정하지 않습니다.

```text
Day05 Test
→ 실패 유형 확인
→ LOW_INLIER_RATIO가 반복된다는 사실만 관찰

        ↓

Day05 Calibration
→ Candidate 값의 근거를 계산
```

오늘 Candidate에서는 `inlier_ratio_min` 하나만 바꿉니다.
`homography_found`와 `sift_good_matches_min`은 Baseline과 동일하게 유지합니다.

Candidate 값은 임의로 `-0.03`처럼 정하지 않습니다.
Calibration의 실제 `OK` 행에서 관측된 `inlier_ratio`의 최솟값을 읽어
**"Calibration에서 관측한 정상 범위의 하단까지 허용한다면 어떻게 되는가?"**라는
한 변수 실험용 후보로 사용합니다.

```text
Baseline
→ Day05 Frozen inlier_ratio_min

Candidate
→ Calibration OK에서 관측한
   최소 inlier_ratio

다른 Rule
→ 그대로 유지
```

이 Candidate는 더 좋은 Rule이라는 뜻이 아닙니다.
오히려 FP가 줄지 않거나 FN이 늘 수도 있습니다.

> **핵심은 Test를 이용해 Threshold를 맞추는 것이 아니라, Test에서는 실패 유형만 확인하고 Candidate 값은 Calibration 데이터에서 계산한다는 점입니다.**


# 32. `73_day09_make_experiment_configs.py` — Baseline과 Candidate Config를 만듭니다

현재 검증된 Day05 Frozen Rule은 실제 판정에 다음 세 가지를 사용합니다.

```text
homography_found
sift_good_matches_min
inlier_ratio_min
```

9일차 Experiment에서도 **Day05에 없는 다른 Rule을 새로 추가하지 않습니다.**

Candidate의 `inlier_ratio_min`은 Test에서 가져오지 않고
`reports/day05_calibration_measurements.csv`의 `actual == OK` 행에서 계산합니다.

파일:

```text
src/73_day09_make_experiment_configs.py
```

작성합니다.

```python
from copy import deepcopy
from pathlib import Path
import json

import pandas as pd


BASE_RULE_PATH = Path("configs/day05_logo_rules.json")
CALIBRATION_PATH = Path(
    "reports/day05_calibration_measurements.csv"
)
OUTPUT_DIR = Path("configs/day09")


if not BASE_RULE_PATH.exists():
    raise FileNotFoundError(BASE_RULE_PATH)

if not CALIBRATION_PATH.exists():
    raise FileNotFoundError(CALIBRATION_PATH)


base_rule = json.loads(
    BASE_RULE_PATH.read_text(encoding="utf-8")
)

required = base_rule.get("required_rules", {})

required_keys = {
    "homography_found",
    "sift_good_matches_min",
    "inlier_ratio_min",
}

missing_keys = required_keys - set(required.keys())

if missing_keys:
    raise KeyError(
        f"Day05 Rule Key가 부족합니다: {sorted(missing_keys)}"
    )


calibration = pd.read_csv(CALIBRATION_PATH)

required_columns = {
    "actual",
    "inlier_ratio",
}

missing_columns = required_columns - set(calibration.columns)

if missing_columns:
    raise RuntimeError(
        "Day05 Calibration CSV 컬럼이 부족합니다: "
        f"{sorted(missing_columns)}"
    )


actual = (
    calibration["actual"]
    .astype(str)
    .str.strip()
    .str.upper()
)

ok_rows = calibration.loc[actual == "OK"].copy()

if ok_rows.empty:
    raise RuntimeError(
        "Calibration에서 actual=OK 행을 찾을 수 없습니다."
    )


baseline_ratio = float(
    required["inlier_ratio_min"]
)

candidate_ratio = round(
    float(
        pd.to_numeric(
            ok_rows["inlier_ratio"],
            errors="raise",
        ).min()
    ),
    6,
)

if candidate_ratio >= baseline_ratio:
    raise RuntimeError(
        "Calibration OK 데이터에서 Baseline보다 낮은 "
        "Inlier Ratio 후보를 얻지 못했습니다. "
        "Test 값을 이용해 Candidate를 임의로 만들지 마세요."
    )


exp001 = {
    "experiment_id": "EXP-001",
    "description": "Day05 frozen baseline",
    "dataset_version": "day05_calibration_v1",
    "source_rule_version": base_rule.get(
        "baseline_version",
        "UNKNOWN",
    ),
    "changed_variable": "none",
    "required_rules": deepcopy(required),
}

exp002 = deepcopy(exp001)
exp002["experiment_id"] = "EXP-002"
exp002["description"] = (
    "Calibration-derived inlier ratio candidate"
)
exp002["changed_variable"] = "inlier_ratio_min"
exp002["old_value"] = baseline_ratio
exp002["new_value"] = candidate_ratio
exp002["candidate_source"] = (
    "minimum inlier_ratio among Day05 Calibration actual=OK"
)
exp002["required_rules"]["inlier_ratio_min"] = (
    candidate_ratio
)


OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

for experiment in [exp001, exp002]:
    output_path = OUTPUT_DIR / (
        experiment["experiment_id"].lower() + ".json"
    )

    output_path.write_text(
        json.dumps(
            experiment,
            indent=2,
            ensure_ascii=False,
        ),
        encoding="utf-8",
    )

    print("Saved:", output_path)

print()
print("Baseline Inlier Ratio :", baseline_ratio)
print("Candidate Inlier Ratio:", candidate_ratio)
print(
    "Candidate Source      :",
    "Day05 Calibration actual=OK minimum",
)
print("Changed Variable       : inlier_ratio_min only")
```

실행합니다.

```bash
python src/73_day09_make_experiment_configs.py
```

이 프로그램은 Day05 Frozen Rule을 수정하지 않습니다.

```text
configs/day05_logo_rules.json
→ 그대로 유지

reports/day05_calibration_measurements.csv
→ Candidate 값 계산에만 사용

configs/day09/exp-001.json
→ Baseline 복사본

configs/day09/exp-002.json
→ Calibration 근거로 계산한
   inlier_ratio_min만 변경한 Candidate
```

실행 후 반드시 다음을 확인합니다.

```text
Baseline Inlier Ratio
Candidate Inlier Ratio
Candidate Source
Changed Variable = inlier_ratio_min only
```

Candidate가 Baseline보다 좋아질 것이라고 미리 가정하지 않습니다.
실제 비교 결과는 다음 `74_day09_run_experiments.py`에서 같은 Calibration 데이터로 확인합니다.


# 33. 두 Experiment Config를 직접 비교합니다

```bash
cat configs/day09/exp-001.json
cat configs/day09/exp-002.json
```

확인할 것은 하나입니다.

```text
EXP-001
changed_variable = none

EXP-002
changed_variable = inlier_ratio_min
```

그리고 나머지 Rule은 같아야 합니다.

---

# 34. Experiment에서 사용할 판정 로직을 확인합니다

9일차 Experiment는 **현재 검증된 5일차 판정 로직을 그대로 재사용**합니다.

```text
Homography 성공
        +
SIFT Good Match가 기준 이상
        +
Inlier Ratio가 기준 이상
        ↓
OK

하나라도 실패
        ↓
NG
```

즉 Day05 Calibration CSV에 실제로 저장된:

```text
homography_found
sift_good_matches
inlier_ratio
```

만 사용합니다.

9일차에서 Day05에 없던 `position_error`, `scale_ratio` 같은 새 측정값을 임의로 추가하지 않습니다.

# 35. `74_day09_run_experiments.py` — 같은 Calibration에서 두 Config를 실행합니다

이 프로그램은 **Day05에서 실제 생성한 Calibration CSV 구조에 맞추어** 실행합니다.

Day05 CSV의 핵심 컬럼은 다음입니다.

```text
file
actual
template_score
sift_good_matches
homography_found
homography_inliers
inlier_ratio
```

오늘 판정에는 그중 `homography_found`, `sift_good_matches`, `inlier_ratio`를 사용합니다.

파일:

```text
src/74_day09_run_experiments.py
```

작성합니다.

```python
from datetime import datetime
from pathlib import Path
import csv
import json
import subprocess

import pandas as pd


DATA_PATH = Path("reports/day05_calibration_measurements.csv")
CONFIG_DIR = Path("configs/day09")
OUTPUT_PATH = Path("reports/day09/experiment_log.csv")


if not DATA_PATH.exists():
    raise FileNotFoundError(DATA_PATH)


def parse_bool(value):
    text = str(value).strip().lower()

    if text in {"1", "true", "yes"}:
        return True

    if text in {"0", "false", "no"}:
        return False

    raise ValueError(
        f"Boolean으로 해석할 수 없는 값입니다: {value}"
    )


def row_passes_rule(row, config):
    required = config["required_rules"]

    homography_ok = True

    if required.get("homography_found", True):
        homography_ok = parse_bool(
            row["homography_found"]
        )

    sift_ok = (
        float(row["sift_good_matches"])
        >= float(required["sift_good_matches_min"])
    )

    inlier_ok = (
        float(row["inlier_ratio"])
        >= float(required["inlier_ratio_min"])
    )

    return all(
        [
            homography_ok,
            sift_ok,
            inlier_ok,
        ]
    )


try:
    git_commit = subprocess.check_output(
        ["git", "rev-parse", "--short", "HEAD"],
        text=True,
        stderr=subprocess.DEVNULL,
    ).strip()
except Exception:
    git_commit = "NO_GIT"


df = pd.read_csv(DATA_PATH)

required_columns = {
    "file",
    "actual",
    "template_score",
    "sift_good_matches",
    "homography_found",
    "homography_inliers",
    "inlier_ratio",
}

missing_columns = required_columns - set(df.columns)

if missing_columns:
    raise RuntimeError(
        "Day05 Calibration CSV 컬럼이 부족합니다: "
        f"{sorted(missing_columns)}"
    )

rows = []

for config_path in sorted(CONFIG_DIR.glob("exp-*.json")):
    config = json.loads(
        config_path.read_text(encoding="utf-8")
    )

    tp = tn = fp = fn = 0

    for _, item in df.iterrows():
        actual = str(item["actual"]).strip().upper()
        predicted = "OK" if row_passes_rule(item, config) else "NG"

        if actual == "NG" and predicted == "NG":
            tp += 1
        elif actual == "OK" and predicted == "OK":
            tn += 1
        elif actual == "OK" and predicted == "NG":
            fp += 1
        elif actual == "NG" and predicted == "OK":
            fn += 1
        else:
            raise RuntimeError(
                f"알 수 없는 actual 값입니다: {actual}"
            )

    total = tp + tn + fp + fn
    accuracy = (tp + tn) / total if total else 0.0
    precision = tp / (tp + fp) if (tp + fp) else 0.0
    recall = tp / (tp + fn) if (tp + fn) else 0.0

    rows.append(
        {
            "timestamp": datetime.now().isoformat(timespec="seconds"),
            "experiment_id": config["experiment_id"],
            "dataset_version": config["dataset_version"],
            "code_commit": git_commit,
            "changed_variable": config.get("changed_variable", ""),
            "old_value": config.get("old_value", ""),
            "new_value": config.get("new_value", ""),
            "homography_required": config[
                "required_rules"
            ]["homography_found"],
            "sift_good_matches_min": config[
                "required_rules"
            ]["sift_good_matches_min"],
            "inlier_ratio_min": config[
                "required_rules"
            ]["inlier_ratio_min"],
            "samples": total,
            "accuracy": round(accuracy, 6),
            "ng_precision": round(precision, 6),
            "ng_recall": round(recall, 6),
            "tp": tp,
            "tn": tn,
            "fp": fp,
            "fn": fn,
        }
    )

if not rows:
    raise RuntimeError("Experiment Config가 없습니다.")

OUTPUT_PATH.parent.mkdir(parents=True, exist_ok=True)

with OUTPUT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=list(rows[0].keys()),
    )
    writer.writeheader()
    writer.writerows(rows)

print("=" * 70)
print("DAY09 EXPERIMENT RUN")
print("=" * 70)

for row in rows:
    print()
    print("Experiment:", row["experiment_id"])
    print("Changed   :", row["changed_variable"])
    print("Accuracy  :", row["accuracy"])
    print("Precision :", row["ng_precision"])
    print("Recall    :", row["ng_recall"])
    print(
        "TP/TN/FP/FN:",
        row["tp"],
        row["tn"],
        row["fp"],
        row["fn"],
    )

print()
print("Saved:", OUTPUT_PATH)
```

실행합니다.

```bash
python src/74_day09_run_experiments.py
```

실제 Accuracy와 FP / FN은 현재 Day05 Calibration 데이터에서 계산됩니다.

문서에 임의의 결과값을 미리 적어두지 않습니다.

### 의사코드

```text
1. Day05 Calibration CSV를 읽는다.
2. 현재 CSV가 Day05에서 실제 생성한 컬럼을 가지고 있는지 확인한다.
3. EXP-001과 EXP-002 Config를 읽는다.
4. 각 Calibration 이미지에 같은 Day05 판정 구조를 적용한다.
5. EXP-001은 Frozen Baseline 값을 사용한다.
6. EXP-002는 inlier_ratio_min만 변경한다.
7. 실제 NG를 Positive로 두고 TP / TN / FP / FN을 계산한다.
8. Accuracy / NG Precision / NG Recall을 계산한다.
9. 어떤 Git Commit에서 실행했는지도 함께 기록한다.
10. 결과를 experiment_log.csv로 저장한다.
```

# 36. Experiment 결과에서 무엇을 봐야 하나요?

```bash
cat reports/day09/experiment_log.csv
```

다음을 함께 봅니다.

```text
Accuracy
NG Precision
NG Recall
FP
FN
```

예를 들어 Threshold를 완화하면 정상 FP가 줄 수 있지만 불량 FN이 늘 수도 있습니다.

반대로 결과가 전혀 바뀌지 않을 수도 있습니다.

둘 다 의미 있는 결과입니다.

> Candidate가 Baseline보다 반드시 좋아져야 하는 실습이 아닙니다.

---

# 37. `75_day09_compare_experiments.py` — 비교표를 만듭니다

파일:

```text
src/75_day09_compare_experiments.py
```

작성합니다.

```python
from pathlib import Path

import pandas as pd


INPUT_PATH = Path("reports/day09/experiment_log.csv")
OUTPUT_PATH = Path("reports/day09/experiment_comparison.csv")


df = pd.read_csv(INPUT_PATH)

columns = [
    "experiment_id",
    "changed_variable",
    "old_value",
    "new_value",
    "homography_required",
    "sift_good_matches_min",
    "inlier_ratio_min",
    "samples",
    "accuracy",
    "ng_precision",
    "ng_recall",
    "fp",
    "fn",
]

missing = [column for column in columns if column not in df.columns]

if missing:
    raise RuntimeError(
        f"Experiment Log 컬럼이 부족합니다: {missing}"
    )

comparison = df[columns].copy()

OUTPUT_PATH.parent.mkdir(parents=True, exist_ok=True)
comparison.to_csv(
    OUTPUT_PATH,
    index=False,
    encoding="utf-8-sig",
)

print("=" * 70)
print("DAY09 EXPERIMENT COMPARISON")
print("=" * 70)
print(comparison.to_string(index=False))
print()
print("Saved:", OUTPUT_PATH)
```

실행합니다.

```bash
python src/75_day09_compare_experiments.py
```

---

# 38. Experiment에서 내릴 결론은 숫자 순위가 아닙니다

다음 질문에 답합니다.

```text
무엇을 바꾸었는가?

무엇은 그대로 두었는가?

FP가 증가했는가 감소했는가?

FN이 증가했는가 감소했는가?

이 변경을 왜 검토해야 하는가?
```

오늘은 Candidate를 Day05 공식 Baseline으로 덮어쓰지 않습니다.

---

# PART5 — Data Drift: Day07 Frozen Baseline에서 밝기만 변경 (39~47)

---

## PART5를 시작하기 전에 — 7일차 Stress Test와 9일차 Drift를 구분합니다


7일차에서도 정상 이미지에 `Dark / Bright / Blur / Shift / Rotation` 변형을 주어 같은 Frozen Rule이 어떤 조건에서 흔들리는지 확인했습니다.

```text
7일차 Stress / Challenge
"어떤 조건에서 검사기가 실패하는가?"
        ↓
개별 취약 조건 발견

실제 결과
Dark / Bright / Shift
→ FP

Blur / Rotation
→ PASS
```

9일차에서는 같은 변형을 다시 체험하는 것이 목적이 아닙니다.

이번에는 **운영 중 입력 환경이 달라졌을 때 기존 검사 Version의 전체 Score 분포와 FP Rate가 어떻게 변하는지**를 확인합니다.

```text
9일차 Drift
같은 Normal 이미지 Pair
        +
같은 Reference
        +
같은 Threshold
        +
같은 Code
        ↓
Brightness만 변경
        ↓
Mean Brightness
Anomaly Score Mean / Std
FP Rate
        ↓
운영환경 변화 계량화
```

즉 7일차가 **취약 조건을 발견하는 실험**이었다면, 9일차는 **같은 Frozen 검사기를 운영한다고 가정하고 입력 변화의 영향을 측정하는 실험**입니다.

---

# 39. Drift는 프로그램 오류와 다른 문제입니다

운영 중 다음 상황이 생길 수 있습니다.

```text
어제
카메라와 조명 상태 정상
        ↓
검사 안정적

한 달 후
조명 밝기 감소
        ↓
입력 이미지가 달라짐
        ↓
Score 분포 변화
```

Code가 바뀌지 않아도 입력 데이터가 달라질 수 있습니다.

---

# 40. Data Drift를 오늘 수준에서 이해합니다

오늘은 다음처럼 이해합니다.

```text
기존 입력 데이터 분포
        ↓
운영환경 변화
        ↓
입력 데이터 분포가 달라짐
        ↓
Data Drift
```

예:

```text
밝기
Camera Exposure
배경
제품 위치
제품 재질
```

오늘은 그중 `Brightness` 하나만 사용합니다.

---

# 41. Drift 실험의 고정 조건과 변경 조건

고정:

```text
제품
이미지 Pair
Day07 Normal Reference
Day07 Threshold
ResNet18 Feature 추출 방식
판정 Rule
```

변경:

```text
Brightness
```

입니다.

---

# 42. `76_day09_make_drift_dataset.py` — 동일 정상 이미지 20쌍을 만듭니다

파일:

```text
src/76_day09_make_drift_dataset.py
```

작성합니다.

```python
from pathlib import Path
import shutil

import cv2


SOURCE_DIR = Path("samples/day07/test/normal")
BASELINE_DIR = Path("samples/day09/drift/baseline")
LIGHTING_DIR = Path("samples/day09/drift/lighting")
PREVIEW_PATH = Path("outputs/day09/drift/preview.jpg")

LIGHTING_ALPHA = 0.65
MAX_IMAGES = 20


image_files = sorted(SOURCE_DIR.glob("*.png"))[:MAX_IMAGES]

if not image_files:
    raise RuntimeError(
        f"Day07 Normal Test 이미지가 없습니다: {SOURCE_DIR}"
    )

for directory in [BASELINE_DIR, LIGHTING_DIR]:
    if directory.exists():
        shutil.rmtree(directory)
    directory.mkdir(parents=True, exist_ok=True)


for image_path in image_files:
    image = cv2.imread(str(image_path))

    if image is None:
        raise FileNotFoundError(image_path)

    baseline_path = BASELINE_DIR / image_path.name
    lighting_path = LIGHTING_DIR / image_path.name

    shutil.copy2(image_path, baseline_path)

    drifted = cv2.convertScaleAbs(
        image,
        alpha=LIGHTING_ALPHA,
        beta=0,
    )

    if not cv2.imwrite(str(lighting_path), drifted):
        raise RuntimeError(
            f"Lighting 이미지를 저장하지 못했습니다: {lighting_path}"
        )


first_name = image_files[0].name
baseline_preview = cv2.imread(str(BASELINE_DIR / first_name))
lighting_preview = cv2.imread(str(LIGHTING_DIR / first_name))

if baseline_preview is None or lighting_preview is None:
    raise RuntimeError("Preview 이미지를 만들 수 없습니다.")

preview = cv2.hconcat(
    [baseline_preview, lighting_preview]
)

PREVIEW_PATH.parent.mkdir(parents=True, exist_ok=True)

if not cv2.imwrite(str(PREVIEW_PATH), preview):
    raise RuntimeError("Preview 저장에 실패했습니다.")

print("=" * 70)
print("DAY09 DRIFT DATASET")
print("=" * 70)
print("Source Images :", len(image_files))
print("Baseline      :", len(list(BASELINE_DIR.glob("*.png"))))
print("Lighting      :", len(list(LIGHTING_DIR.glob("*.png"))))
print("Alpha         :", LIGHTING_ALPHA)
print("Preview       :", PREVIEW_PATH)
```

실행합니다.

```bash
python src/76_day09_make_drift_dataset.py
```

---

# 43. Baseline과 Lighting 이미지를 눈으로 확인합니다

다음 파일을 엽니다.

```text
outputs/day09/drift/preview.jpg
```

왼쪽은 원본, 오른쪽은 밝기를 낮춘 이미지입니다.

제품 자체에는 결함을 만들지 않았습니다.

따라서 두 이미지의 실제 정답은 모두:

```text
Normal
```

입니다.

---

# 44. 기존 Day07 Baseline을 그대로 사용합니다

다음 두 파일을 재사용합니다.

```text
artifacts/day07/normal_reference.npz
configs/day07_anomaly_rules.json
```

중요합니다.

```text
Reference 다시 생성 X
Threshold 다시 결정 X
Rule 다시 Freeze X
```

9일차에서는 **운영환경이 달라졌을 때 기존 Baseline이 어떻게 반응하는지** 봅니다.

---

# 45. `77_day09_measure_drift.py` — 밝기와 Anomaly Score를 비교합니다

파일:

```text
src/77_day09_measure_drift.py
```

작성합니다.

```python
from pathlib import Path
import csv
import json

import cv2
import numpy as np
import pandas as pd

from day07_anomaly_core import (
    anomaly_score,
    create_backbone,
    extract_features,
)


DRIFT_ROOT = Path("samples/day09/drift")
RULE_PATH = Path("configs/day07_anomaly_rules.json")
OUTPUT_PATH = Path("reports/day09/drift_results.csv")
SUMMARY_PATH = Path("reports/day09/drift_summary.csv")


if not RULE_PATH.exists():
    raise FileNotFoundError(RULE_PATH)

rules = json.loads(
    RULE_PATH.read_text(encoding="utf-8")
)

threshold = float(rules["anomaly_threshold"])
reference_path = Path(
    rules.get(
        "reference_path",
        "artifacts/day07/normal_reference.npz",
    )
)

if not reference_path.exists():
    raise FileNotFoundError(reference_path)

reference_data = np.load(reference_path)
reference_vector = reference_data["reference_vector"]

model = create_backbone()
rows = []

for condition in ["baseline", "lighting"]:
    folder = DRIFT_ROOT / condition
    image_files = sorted(folder.glob("*.png"))

    if not image_files:
        raise RuntimeError(
            f"Drift 이미지가 없습니다: {folder}"
        )

    for image_path in image_files:
        gray = cv2.imread(
            str(image_path),
            cv2.IMREAD_GRAYSCALE,
        )

        if gray is None:
            raise FileNotFoundError(image_path)

        mean_brightness = float(np.mean(gray))

        feature, _ = extract_features(
            model,
            image_path,
        )

        score = anomaly_score(
            feature,
            reference_vector,
        )

        predicted = (
            "Anomaly"
            if score > threshold
            else "Normal"
        )

        rows.append(
            {
                "file": image_path.name,
                "condition": condition,
                "actual": "Normal",
                "mean_brightness": round(mean_brightness, 3),
                "anomaly_score": round(score, 6),
                "threshold": round(threshold, 6),
                "predicted": predicted,
                "false_positive": predicted == "Anomaly",
            }
        )

OUTPUT_PATH.parent.mkdir(parents=True, exist_ok=True)

with OUTPUT_PATH.open(
    "w",
    newline="",
    encoding="utf-8-sig",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=list(rows[0].keys()),
    )
    writer.writeheader()
    writer.writerows(rows)


df = pd.DataFrame(rows)

summary = (
    df.groupby("condition")
    .agg(
        image_count=("file", "count"),
        brightness_mean=("mean_brightness", "mean"),
        anomaly_score_mean=("anomaly_score", "mean"),
        anomaly_score_std=("anomaly_score", "std"),
        false_positive_rate=("false_positive", "mean"),
    )
    .reset_index()
)

summary.to_csv(
    SUMMARY_PATH,
    index=False,
    encoding="utf-8-sig",
)

print("=" * 70)
print("DAY09 DRIFT RESULT")
print("=" * 70)
print(summary.to_string(index=False))
print()
print("Threshold:", threshold)
print("Detail   :", OUTPUT_PATH)
print("Summary  :", SUMMARY_PATH)
```

실행합니다.

```bash
python src/77_day09_measure_drift.py
```

---

# 46. Drift 결과는 실제 실행값으로 읽습니다

```bash
cat reports/day09/drift_summary.csv
```

확인할 값은 다음입니다.

```text
brightness_mean
anomaly_score_mean
anomaly_score_std
false_positive_rate
```

문서에는 결과를 미리 단정하지 않습니다.

실행 후 다음 질문에 답합니다.

```text
Lighting에서 평균 밝기가 실제로 낮아졌는가?

Anomaly Score 평균도 변했는가?

정상인데 Anomaly로 판단한 FP가 늘었는가?

Score 변화가 거의 없다면 왜 그럴까?
```

---

# 47. Drift가 발생했다고 바로 모델을 다시 만들지는 않습니다

먼저 원인을 확인합니다.

```text
조명 장치 문제인가?
Camera Exposure 문제인가?
렌즈 오염인가?
제품 자체가 달라졌는가?
배경이 달라졌는가?
```

장비 문제인데 Model을 다시 학습하면 문제를 잘못 해결할 수 있습니다.

---

# PART6 — 같은 데이터로 Hybrid 성능 비교 (48~56)

# 48. Hybrid는 무엇을 확인하는 실습인가요?

**Hybrid 검사는 여러 코드를 한 파일로 합치는 것이 아닙니다.** 같은 검사 대상에 적용한 두 방법의 결과를 결합하고, 단독 검사의 성능과 비교하는 과정입니다.

이번에는 8일차에서 이미 검사한 **MVTec bottle Common Query 20장**을 사용합니다.

```text
                같은 병 이미지 20장
                       ↓
            ┌──────────┴──────────┐
            ↓                     ↓
         One-shot               Few-shot
         예측 결과              예측 결과
            └──────────┬──────────┘
                       ↓
           결합 규칙 OR / AND
                       ↓
              최종 Normal / Anomaly
                       ↓
            실제 정답과 비교
                       ↓
         Accuracy / Precision / Recall
                      FP / FN
                       ↓
          단독 검사보다 나아졌는가?
```

두 방법은 모두 사전학습 ResNet18 특징을 활용하지만 기준을 만드는 방식이 다릅니다. **새 모델을 학습하지 않고 기존의 샘플별 예측 CSV를 재사용**합니다.

# 49. 왜 광천김·OCR·MVTec 결과를 합치지 않나요?

광천김 로고, SN 상자, MVTec 병은 서로 다른 제품입니다. 이들의 OK/NG를 단순히 합쳐도 **어느 제품이 불량인지 정의할 수 없으므로 실제 검사 성능을 평가할 수 없습니다.**

반면 Day08 One-shot과 Few-shot은 같은 20장을 검사했습니다. 각 이미지의 `sample_id`와 실제 정답 `actual`이 같으므로, 결합 결과도 같은 정답과 비교할 수 있습니다.

**오늘 반드시 구분합니다.**

- *검사 모듈 통합*: 여러 프로그램의 출력 형식을 연결하는 소프트웨어 작업
- *Hybrid 성능 평가*: 동일한 대상에 적용한 여러 검사 결과를 결합한 뒤 FP·FN과 정확도 등을 검증하는 작업

오늘은 두 번째를 실습하면서 결과를 공통 CSV 형태로 저장하는 첫 번째 기능도 함께 배웁니다.

# 50. 오늘 사용할 데이터와 안전 규칙

입력 파일은 8일차에 생성한 실제 결과입니다.

```text
reports/day08_one_shot_results.csv
reports/day08_fewshot_results.csv
```

두 CSV에서 필요한 항목은 다음과 같습니다.

| 컬럼 | 의미 |
|---|---|
| `sample_id` | 동일 이미지인지 확인하는 고유 ID |
| `file` | 실제 파일 경로(추적용) |
| `actual` | 실제 정답: `Normal` 또는 `Anomaly` |
| `predicted` | 검사기가 예측한 값 |

**확인할 사항:** 두 파일 각각 ID 중복이 없어야 하고, 양쪽 ID 집합과 각 ID의 정답이 같아야 합니다. 이후 `sample_id`를 기준으로 연결합니다. **행 순서만 보고 결합하지 않습니다.**

8일차에서 보고된 85%와 90%는 **기존 결과**이며, Hybrid 정확도는 **직접 실행하여 얻은 값**을 기록합니다.

이번에는 Test를 이용해 정책을 최적화하지 않습니다. OR와 AND를 **비교할 두 가지 고정된 교육용 정책**으로 먼저 정한 뒤 그 성능을 확인합니다. 결과가 더 좋아 보이는 쪽을 곧바로 실서비스의 검증된 정책으로 선언해서도 안 됩니다. 실제 적용에는 별도의 검증용 데이터와 새로운 최종 Test가 필요합니다.

# 51. OR와 AND를 먼저 이해합니다

여기서는 `Anomaly`를 **불량(Positive)** 으로 정합니다.

| One-shot | Few-shot | OR 판정 | AND 판정 |
|---|---|---|---|
| Normal | Normal | Normal | Normal |
| Anomaly | Normal | **Anomaly** | Normal |
| Normal | Anomaly | **Anomaly** | Normal |
| Anomaly | Anomaly | **Anomaly** | **Anomaly** |

- **OR**: 둘 중 하나라도 불량이면 불량으로 판단합니다. 놓치는 불량(FN)이 줄 수 있지만 정상 오탐(FP)이 늘 수 있습니다.
- **AND**: 둘 다 불량일 때만 불량으로 판단합니다. 정상 오탐(FP)은 줄 수 있지만 놓치는 불량(FN)이 늘 수 있습니다.

둘 중 어느 방법이 좋은지는 **같은 20장의 실제 정답과 비교하여** 확인합니다. 결합한다고 무조건 좋아지는 것은 아닙니다.

# 52. `78_day09_make_hybrid_demo.py` — 실제 Day08 결과를 연결합니다

기존 파일명을 유지하지만, **이제 가짜 검사 Case를 만들지 않습니다.** 두 Result CSV를 `sample_id`로 결합하여 `hybrid_inputs.csv`를 생성합니다.

이 코드는 다음 순서로 작동합니다.

```text
Day08 재현성 Audit 통과 확인
       ↓
One-shot / Few-shot 결과 CSV 읽기
       ↓
ID·정답·예측 형식 검사
       ↓
같은 sample_id인지 확인
       ↓
같은 이미지별 예측값 한 행으로 저장
       ↓
hybrid_inputs.csv + hybrid_policy.json
```

파일: `src/78_day09_make_hybrid_demo.py`

```python
from pathlib import Path
import json
import pandas as pd

ONE_PATH = Path("reports/day08_one_shot_results.csv")
FEW_PATH = Path("reports/day08_fewshot_results.csv")
AUDIT_PATH = Path("reports/day09/day08_reproducibility_audit.json")
OUTPUT_DIR = Path("reports/day09")
POLICY_PATH = Path("configs/day09/hybrid_policy.json")
INPUT_PATH = OUTPUT_DIR / "hybrid_inputs.csv"

# PART3에서 같은 Query와 결과를 이미 검증했는지 확인합니다.
if not AUDIT_PATH.exists():
    raise FileNotFoundError("PART3의 Day08 Audit를 먼저 실행하세요.")

audit = json.loads(AUDIT_PATH.read_text(encoding="utf-8"))
if not audit.get("same_common_query") or not audit.get("all_metric_match"):
    raise RuntimeError("Day08 Audit가 통과하지 않았습니다.")


def load_result(path, prefix):
    if not path.exists():
        raise FileNotFoundError(path)
    df = pd.read_csv(path, dtype={"sample_id": str})
    required = {"sample_id", "file", "actual", "predicted"}
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"{path}: 누락 컬럼 {sorted(missing)}")

    df = df[["sample_id", "file", "actual", "predicted"]].copy()
    for col in ["sample_id", "file", "actual", "predicted"]:
        if df[col].isna().any():
            raise ValueError(f"{path}: {col} 빈 값")
        df[col] = df[col].astype(str).str.strip()
        if df[col].eq("").any():
            raise ValueError(f"{path}: {col} 빈 문자열")

    if df["sample_id"].duplicated().any():
        raise ValueError(f"{path}: sample_id 중복")
    valid = {"Normal", "Anomaly"}
    if not set(df["actual"]).issubset(valid):
        raise ValueError(f"{path}: actual에는 Normal/Anomaly만 사용합니다.")
    if not set(df["predicted"]).issubset(valid):
        raise ValueError(f"{path}: predicted에는 Normal/Anomaly만 사용합니다.")

    return df.rename(columns={
        "file": f"{prefix}_file",
        "actual": f"{prefix}_actual",
        "predicted": f"{prefix}_predicted",
    })


one = load_result(ONE_PATH, "one_shot")
few = load_result(FEW_PATH, "few_shot")

if set(one["sample_id"]) != set(few["sample_id"]):
    raise RuntimeError("One-shot/Few-shot의 검사 이미지 ID가 다릅니다.")

merged = one.merge(few, on="sample_id", how="inner", validate="one_to_one")
if not merged["one_shot_actual"].eq(merged["few_shot_actual"]).all():
    raise RuntimeError("같은 이미지의 실제 정답이 서로 다릅니다.")
if not merged["one_shot_file"].eq(merged["few_shot_file"]).all():
    raise RuntimeError("같은 ID인데 이미지 경로가 다릅니다. Day08 결과를 확인하세요.")

if len(merged) != 20:
    raise RuntimeError(f"이번 수업은 공통 이미지 20장을 사용합니다. 현재 {len(merged)}장")

merged = merged.sort_values("sample_id").reset_index(drop=True)
inputs = pd.DataFrame({
    "sample_id": merged["sample_id"],
    "file": merged["one_shot_file"],
    "actual": merged["one_shot_actual"],
    "one_shot_predicted": merged["one_shot_predicted"],
    "few_shot_predicted": merged["few_shot_predicted"],
})

policy = {
    "policy_version": "day09-hybrid-v2.0",
    "dataset": "Day08 MVTec bottle common_query 20",
    "positive_label": "Anomaly",
    "policies": ["OR", "AND"],  # 결과 확인 전에 고정한 비교 정책
    "sources": {
        "one_shot": str(ONE_PATH),
        "few_shot": str(FEW_PATH),
        "day08_audit": str(AUDIT_PATH),
    },
    "version_manifest": "reports/day09/version_manifest.json",
    "note": "실제 Day08 저장 예측의 결합; 새로운 CNN 학습은 수행하지 않음",
}

OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
POLICY_PATH.parent.mkdir(parents=True, exist_ok=True)
inputs.to_csv(INPUT_PATH, index=False, encoding="utf-8-sig")
POLICY_PATH.write_text(
    json.dumps(policy, ensure_ascii=False, indent=2), encoding="utf-8"
)
print("Common Query:", len(inputs))
print("Saved:", INPUT_PATH)
print("Policy:", POLICY_PATH)
```

실행:

```bash
python src/78_day09_make_hybrid_demo.py
```

확인:

```bash
head -n 5 reports/day09/hybrid_inputs.csv
cat configs/day09/hybrid_policy.json
```

`hybrid_inputs.csv`의 **한 행은 동일한 병 이미지 한 장에 대한 두 검사기의 예측**입니다. `hybrid_policy.json`은 결합 규칙과 결과 출처를 기록합니다.

# 53. 코드를 이해하는 의사코드

```text
1. Day08 Audit가 통과했는지 확인한다.
2. One-shot과 Few-shot 결과를 읽는다.
3. sample_id가 중복되거나 서로 다르면 중단한다.
4. 같은 이미지의 실제 정답이 같은지 확인한다.
5. 같은 이미지별 두 예측을 한 행으로 모은다.
6. 결과 CSV와 OR/AND 정책 JSON을 저장한다.
```

**실패했을 때:** 데이터 수나 `sample_id`, `file`, `actual`이 다르면 임의로 고쳐 합치지 마세요. Day08에서 생성한 원본 결과를 다시 확인해야 합니다.

# 54. `day09_hybrid_core.py` — OR·AND 판정 함수를 만듭니다

이 파일은 이미지나 모델을 다시 실행하지 않습니다. 입력받은 **두 예측값**을 정책에 따라 결합합니다.

파일: `src/day09_hybrid_core.py`

```python
VALID_LABELS = {"Normal", "Anomaly"}


def hybrid_judge(one_predicted, few_predicted, policy):
    if one_predicted not in VALID_LABELS or few_predicted not in VALID_LABELS:
        raise ValueError("예측값은 Normal 또는 Anomaly여야 합니다.")
    if policy not in {"OR", "AND"}:
        raise ValueError(f"지원하지 않는 정책: {policy}")

    one_ng = one_predicted == "Anomaly"
    few_ng = few_predicted == "Anomaly"

    is_anomaly = (
        (one_ng or few_ng) if policy == "OR"
        else (one_ng and few_ng)
    )
    votes = [
        method
        for method, flagged in [
            ("ONE_SHOT", one_ng),
            ("FEW_SHOT", few_ng),
        ]
        if flagged
    ]
    return {
        "predicted": "Anomaly" if is_anomaly else "Normal",
        "reason": "|".join(votes) if is_anomaly else "NORMAL",
    }


def calculate_metrics(actual_values, predicted_values):
    actual = list(actual_values)
    predicted = list(predicted_values)
    if len(actual) != len(predicted) or not actual:
        raise ValueError("정답과 예측 수가 같고 1개 이상이어야 합니다.")
    if not set(actual + predicted).issubset(VALID_LABELS):
        raise ValueError("정답과 예측에는 Normal/Anomaly만 사용합니다.")

    tp = sum(a == "Anomaly" and p == "Anomaly" for a, p in zip(actual, predicted))
    tn = sum(a == "Normal" and p == "Normal" for a, p in zip(actual, predicted))
    fp = sum(a == "Normal" and p == "Anomaly" for a, p in zip(actual, predicted))
    fn = sum(a == "Anomaly" and p == "Normal" for a, p in zip(actual, predicted))
    total = len(actual)
    return {
        "samples": total,
        "accuracy": round((tp + tn) / total, 4),
        "anomaly_precision": round(tp / (tp + fp), 4) if (tp + fp) else 0.0,
        "anomaly_recall": round(tp / (tp + fn), 4) if (tp + fn) else 0.0,
        "tp": tp,
        "tn": tn,
        "fp": fp,
        "fn": fn,
    }


if __name__ == "__main__":
    assert hybrid_judge("Anomaly", "Normal", "OR")["predicted"] == "Anomaly"
    assert hybrid_judge("Anomaly", "Normal", "AND")["predicted"] == "Normal"
    assert hybrid_judge("Normal", "Normal", "OR")["predicted"] == "Normal"
    print("Hybrid Core Self Check: PASS")
```

함수가 정상 동작하는지 먼저 실행합니다.

```bash
python src/day09_hybrid_core.py
```

정상 출력: `Hybrid Core Self Check: PASS`

# 55. `79_day09_run_hybrid_pipeline.py` — 실제 결과로 성능을 비교합니다

이 코드는 단독 One-shot/Few-shot, OR Hybrid, AND Hybrid의 성능을 **같은 20장**에서 계산합니다.

파일: `src/79_day09_run_hybrid_pipeline.py`

```python
from pathlib import Path
import json
import pandas as pd

from day09_hybrid_core import hybrid_judge, calculate_metrics

INPUT_PATH = Path("reports/day09/hybrid_inputs.csv")
POLICY_PATH = Path("configs/day09/hybrid_policy.json")
RESULT_PATH = Path("reports/day09/hybrid_results.csv")
METRIC_PATH = Path("reports/day09/hybrid_metrics.csv")
REPORT_PATH = Path("reports/day09/hybrid_comparison.md")

if not INPUT_PATH.exists() or not POLICY_PATH.exists():
    raise FileNotFoundError("먼저 78_day09_make_hybrid_demo.py를 실행하세요.")

inputs = pd.read_csv(INPUT_PATH, dtype={"sample_id": str})
policy = json.loads(POLICY_PATH.read_text(encoding="utf-8"))
required = {"sample_id", "file", "actual", "one_shot_predicted", "few_shot_predicted"}
if required - set(inputs.columns):
    raise RuntimeError(f"Hybrid 입력 CSV 컬럼이 부족합니다: {sorted(required - set(inputs.columns))}")
if inputs["sample_id"].duplicated().any() or len(inputs) != 20:
    raise RuntimeError("공통 20장이어야 하며 sample_id 중복이 없어야 합니다.")

results = []
for _, row in inputs.iterrows():
    actual = row["actual"]
    one = row["one_shot_predicted"]
    few = row["few_shot_predicted"]
    for mode in policy["policies"]:
        result = hybrid_judge(one, few, mode)
        results.append({
            "sample_id": row["sample_id"],
            "file": row["file"],
            "actual": actual,
            "policy": mode,
            "one_shot_predicted": one,
            "few_shot_predicted": few,
            "hybrid_predicted": result["predicted"],
            "correct": actual == result["predicted"],
            "reason": result["reason"],
            "policy_version": policy["policy_version"],
        })

RESULT_PATH.parent.mkdir(parents=True, exist_ok=True)
pd.DataFrame(results).to_csv(RESULT_PATH, index=False, encoding="utf-8-sig")

methods = {
    "one_shot": inputs["one_shot_predicted"].tolist(),
    "few_shot": inputs["few_shot_predicted"].tolist(),
}
for mode in policy["policies"]:
    methods[f"hybrid_{mode.lower()}"] = [
        hybrid_judge(o, f, mode)["predicted"]
        for o, f in zip(inputs["one_shot_predicted"], inputs["few_shot_predicted"])
    ]

metric_rows = []
for method, predictions in methods.items():
    metric_rows.append({
        "method": method,
        **calculate_metrics(inputs["actual"].tolist(), predictions),
    })
metrics = pd.DataFrame(metric_rows)
metrics.to_csv(METRIC_PATH, index=False, encoding="utf-8-sig")

# Day08 원래 Accuracy와 단독검사 결과가 일치하는지 재확인합니다.
original = pd.read_csv("reports/day08_method_comparison.csv")
if not {"method", "accuracy"}.issubset(original.columns):
    raise RuntimeError("Day08 Method Comparison 형식이 다릅니다.")
for method in ["one_shot", "few_shot"]:
    stored = original.loc[original["method"] == method, "accuracy"]
    current = metrics.loc[metrics["method"] == method, "accuracy"].iloc[0]
    if len(stored) != 1 or round(float(stored.iloc[0]), 4) != round(float(current), 4):
        raise RuntimeError(f"{method} 저장 Accuracy와 다시 계산한 값이 다릅니다.")

lines = [
    "# Day09 Hybrid 성능 비교", "",
    "- 데이터: Day08 MVTec bottle Common Query 20장",
    "- 정책: OR / AND (Test 결과를 보기 전에 고정한 비교 정책)",
    "- 두 검사 모두 기존 Day08 예측 CSV 사용; 재학습·재추론 없음", "",
    "| 검사 방식 | Accuracy | Precision | Recall | FP | FN |",
    "|---|---:|---:|---:|---:|---:|",
]
for m in metric_rows:
    lines.append(
        f"| {m['method']} | {m['accuracy']:.1%} | "
        f"{m['anomaly_precision']:.1%} | {m['anomaly_recall']:.1%} | "
        f"{m['fp']} | {m['fn']} |"
    )
lines += [
    "", "## 관찰", "",
    "- OR에서 FN이 줄었나요? FP는 늘었나요?", 
    "- AND에서 FP가 줄었나요? FN은 늘었나요?",
    "- 어떤 상황에 어떤 정책을 선택할지 근거를 적으세요.",
    "- 이 20장은 비교용 Test이므로 여기서 정책을 선택했다면 **별도 최종 Test**가 필요합니다.",
]
REPORT_PATH.write_text("\n".join(lines) + "\n", encoding="utf-8")

print("=" * 65)
print("DAY09 HYBRID COMPARISON")
print("=" * 65)
print(metrics.to_string(index=False))
print("Saved:", RESULT_PATH)
print("Saved:", METRIC_PATH)
print("Saved:", REPORT_PATH)
```

실행:

```bash
python src/79_day09_run_hybrid_pipeline.py
```

확인:

```bash
cat reports/day09/hybrid_metrics.csv
cat reports/day09/hybrid_comparison.md
```

실습 결과에 `Accuracy = 100%`가 나와야 성공하는 것이 아닙니다. **동일한 입력에서 각 방법의 성능이 어떻게 달라졌는지 정확히 비교하면 성공**입니다.

# 56. Hybrid 결과를 해석합니다

`hybrid_metrics.csv`에서 다음 항목을 함께 확인합니다.

| 항목 | 의미 |
|---|---|
| Accuracy | 전체 이미지 중 정답 비율 |
| Precision | 이상이라고 판정한 것 중 실제 이상 비율 |
| Recall | 실제 이상 중 찾아낸 비율 |
| FP | 정상 이미지를 이상이라고 잘못 판단한 수 |
| FN | 이상 이미지를 정상이라고 놓친 수 |

**생각해 볼 질문**

1. OR 방식에서 불량 미검출(FN)이 줄었습니까?
2. AND 방식에서 정상 오탐(FP)이 줄었습니까?
3. 성능이 좋아진 항목과 나빠진 항목이 동시에 있습니까?
4. 검사 현장에서 *불량을 놓치면 안 되는 문제*라면 무엇을 우선해야 할까요?

**정확한 결론:** Hybrid는 성능 향상을 *시험하는 방법*이지 성능 향상을 *보장하는 기술*이 아닙니다. One-shot과 Few-shot은 공통 ResNet18 특징을 사용하므로 오류가 서로 겹치면 개선이 없을 수도 있습니다.

---

# PART7 — Challenge: 검사 목적에 맞는 기술 선택 (57~62)

# 57. 여러 기술을 무조건 연결하지 않습니다

Hybrid가 필요하려면 **같은 대상에서 서로 보완할 수 있는 검사가 존재해야 합니다.** 검사 목적이 단순하면 한 가지 방법이 더 효율적입니다.

# 58. 검사 기술을 선택하는 순서

```text
무엇을 검사하나요?
  ├─ 개수·크기·위치처럼 숫자로 설명 가능 → OpenCV Rule
  ├─ 로고·라벨 정합                           → Template / SIFT
  ├─ SN·LOT 등 문자                           → OCR
  ├─ 예상 못 한 표면 이상                       → Anomaly Detection
  ├─ 정상 기준·불량 예제가 매우 적음             → One-shot / Few-shot
  └─ 객체 위치·종류를 검출해야 함                → AI Detection
                           ↓
         하나의 방법으로 부족하고 목적이 같음?
                           ↓
             그때 Hybrid를 검토합니다.
```

# 59. Situation A — 부품이 정확히 4개 있어야 합니다

배경·카메라가 고정되어 있고 개수만 검사한다면 `Threshold → Morphology → Contour → Count Rule`을 먼저 고려합니다.

# 60. Situation B — 제품의 SN을 검사해야 합니다

`SN ROI → OCR → 형식 검사(Regex) → 기준번호 비교`가 기본 구조입니다.

# 61. Situation C / D / E / F — 직접 선택합니다

- **C:** 정상 제품은 많지만 표면 결함의 형태를 미리 전부 정의하기 어렵습니다.
- **D:** 제품 위치가 달라지고 객체 종류도 다양하며 라벨 데이터가 충분합니다.
- **E:** 하나의 제품에서 부품 상태·인쇄 문자·표면 이상을 모두 검사해야 합니다. 어떤 검사를 실제로 결합할지 이유를 설명합니다.
- **F:** 정상·불량 예제가 각각 1~5장뿐입니다. One-shot/Few-shot 후보를 비교합니다.

# 62. `day09_tech_matrix.md`를 작성합니다

파일: `reports/day09/day09_tech_matrix.md`

```markdown
# Day09 검사기술 선택 Matrix

| 검사 문제 | 선택 기술 | 선택 이유 | 제외한 기술과 이유 |
|---|---|---|---|
| 부품 개수 | | | |
| 로고 정합 | | | |
| SN / LOT | | | |
| 미지의 표면 이상 | | | |
| 위치가 계속 변하는 객체 | | | |
| Low-data 검사 | | | |
| 복합 검사 | | | |

## Hybrid가 필요한 경우

## 한 기술만 사용하는 것이 좋은 경우
```

기술 이름만 나열하지 말고 **해당 제품에서 어떤 문제를 해결하는지** 적습니다.

---

# PART8 — Mini Challenge: 밝기 대신 Blur로 Drift 확인 (63~67)

# 63. 오늘의 Mini Challenge

PART 5에서는 정상 병 이미지의 밝기만 변경했습니다. 이번에는 **같은 정상 이미지를 흐리게(Blur) 만든 후 기존 Day07 검사기로 다시 검사**합니다.

# 64. 무엇을 바꾸고 무엇을 유지할까요?

| 변경할 것 | 그대로 둘 것 |
|---|---|
| Gaussian Blur 커널 하나 (예: `(7, 7)`) | 동일한 정상 이미지 목록, Day07 Normal Reference, Threshold, ResNet18 Feature, 검사 Rule |

# 65. 기존 코드를 참고해 직접 작성합니다

- `src/76_day09_make_drift_dataset.py`: 같은 이미지의 원본·변형 Pair 생성
- `src/77_day09_measure_drift.py`: Frozen Rule로 Score·FP Rate 비교

작성할 파일: `src/day09_mini_drift.py`

```python
from pathlib import Path
import cv2

SOURCE_DIR = Path("samples/day07/test/normal")
BASELINE_DIR = Path("samples/day09/mini_drift/baseline")
BLUR_DIR = Path("samples/day09/mini_drift/blur")

# TODO 1: 원본 정상 이미지 20장을 불러옵니다.
# TODO 2: baseline에는 원본을 복사합니다.
# TODO 3: blur에는 같은 이미지에 Gaussian Blur만 적용합니다.
# TODO 4: Day07 Reference와 Threshold를 그대로 읽습니다.
# TODO 5: 두 조건의 Anomaly Score와 FP Rate를 계산합니다.
# TODO 6: 결과를 CSV 또는 Markdown으로 정리합니다.

print("Day09 Mini Drift를 완성하세요.")
```

# 66. Mini Challenge 진행 순서

```text
같은 정상 병 20장 선택
       ↓
원본 Baseline 저장
       ↓
Gaussian Blur 적용 이미지 저장
       ↓
기존 Day07 Reference와 Threshold 재사용
       ↓
두 조건의 Anomaly Score 계산
       ↓
Score 평균·표준편차·FP Rate 비교
       ↓
관찰과 실패 원인 기록
```

결과 파일: `reports/day09/day09_mini_challenge.md`

```markdown
# Day09 Mini Challenge — Blur Drift

- 사용 이미지:
- Blur Kernel:
- 변경하지 않은 Reference:
- 변경하지 않은 Threshold:

| Condition | Mean Score | Score Std | FP Rate |
|---|---:|---:|---:|
| Baseline | | | |
| Blur | | | |

## 예상과 실제 차이

## 실패 원인과 개선 아이디어
```

# 67. 완료 기준

- [ ] 원본과 Blur의 파일 구성이 같습니다.
- [ ] Blur 하나만 변경했습니다.
- [ ] 기존 Reference와 Threshold를 다시 만들거나 수정하지 않았습니다.
- [ ] 실제 Score와 FP Rate를 비교했습니다.
- [ ] 결과가 예상과 달라도 그대로 기록했습니다.

---

# PART9 — 실패 분석·결과 정리·Git·10일차 연결 (68~81)

# 68. 성능이 낮게 나와도 무엇을 확인할까요?

- **재현성 실패:** 기존 CSV와 재계산한 Metric이 다릅니다. `sample_id`, Config, Artifact를 확인합니다.
- **Candidate 악화:** Threshold 한 가지 변경이 FP/FN에 어떤 영향을 주었는지 확인합니다.
- **Drift 민감도:** 검사기는 그대로인데 조명·Blur 때문에 정상 오탐이 늘었는지 봅니다.
- **Hybrid 악화:** OR/AND 정책이 FP 또는 FN을 늘렸는지 확인합니다. 두 예측이 동일 샘플인지도 다시 확인합니다.

좋은 결과만 남기지 말고 **실패한 조건과 원인**을 기록합니다.

# 69. `day09_analysis.md`에 실제 수치를 기록합니다

파일: `reports/day09/day09_analysis.md`

```markdown
# Day09 실습 결과 분석

## 1. Version / Lineage
- Git Commit:
- Day08 Common Query Fingerprint:
- 주요 Config / Artifact Hash:

## 2. Day08 재현성 점검
- 동일 Common Query 확인:
- One-shot Accuracy / FP / FN:
- Few-shot Accuracy / FP / FN:
- 저장된 Metric 일치 여부:

## 3. Day05 Experiment
- Baseline 설정값 / Accuracy / FP / FN:
- Candidate 설정값 / Accuracy / FP / FN:
- 실제로 바뀐 변수 하나:
- 결과 해석:

## 4. Day07 Drift
- 고정 조건:
- 변경한 Brightness:
- Baseline과 Lighting의 Mean Score / FP Rate:
- 변화 원인:

## 5. Hybrid 성능 비교
- 동일 Query 20장 확인:
- Policy Version:
- One-shot Accuracy / FP / FN:
- Few-shot Accuracy / FP / FN:
- Hybrid OR Accuracy / FP / FN:
- Hybrid AND Accuracy / FP / FN:
- 결합이 개선한 지표:
- 결합이 악화한 지표:
- 실제 현장 적용 전 추가로 검증할 내용:

## 6. Mini Challenge
- Blur Kernel:
- Mean Score / FP Rate 변화:
- 예상과 다른 점:

## 7. 오늘의 결론
- 재현 가능한 검사 실험에 필요한 기록:
- Hybrid를 사용하면 좋은 상황 / 사용하지 않아야 할 상황:
```

**실행 결과에 맞게 빈칸을 채웁니다. Hybrid 정확도는 미리 정답 숫자를 적어두지 않습니다.**

# 70. 예상 결과에 맞추어 데이터를 수정하지 않습니다

결과가 마음에 들지 않아도 Test 예측을 수정하거나, 정확도를 높이기 위해 Test를 보고 OR/AND 외 새 정책을 즉석에서 튜닝하지 않습니다. 개선 정책을 추가로 검토한다면 별도의 검증 데이터와 최종 Test를 준비해야 합니다.

# 71. 9일차의 핵심 결과 다섯 가지

| 실습 | 확인할 핵심 |
|---|---|
| Version Manifest | 어떤 파일·설정·코드에서 실행했는가? |
| Reproducibility Audit | 동일 Query·정답·Metric을 재확인할 수 있는가? |
| Experiment | 한 변수 변경이 어떤 영향을 주었는가? |
| Drift | 환경만 바뀌어도 FP Rate가 달라지는가? |
| Hybrid | 같은 20장에 두 방법을 결합하면 성능이 어떻게 변하는가? |

# 72. 최종 산출물 확인

```text
[실행 코드]
src/70_day09_check_env.py
src/71_day09_build_version_manifest.py
src/72_day09_audit_day08_results.py
src/73_day09_make_experiment_configs.py
src/74_day09_run_experiments.py
src/75_day09_compare_experiments.py
src/76_day09_make_drift_dataset.py
src/77_day09_measure_drift.py
src/78_day09_make_hybrid_demo.py
src/day09_hybrid_core.py
src/79_day09_run_hybrid_pipeline.py
src/day09_mini_drift.py

[Config]
configs/day09/exp-001.json
configs/day09/exp-002.json
configs/day09/hybrid_policy.json

[주요 Report]
reports/day09/version_manifest.json
reports/day09/day08_reproducibility_audit.csv
reports/day09/day08_reproducibility_audit.json
reports/day09/experiment_log.csv
reports/day09/experiment_comparison.csv
reports/day09/drift_results.csv
reports/day09/drift_summary.csv
reports/day09/hybrid_inputs.csv
reports/day09/hybrid_results.csv
reports/day09/hybrid_metrics.csv
reports/day09/hybrid_comparison.md
reports/day09/day09_tech_matrix.md
reports/day09/day09_mini_challenge.md
reports/day09/day09_analysis.md
```

# 73. README에는 결론만 정리합니다

예를 들어 다음 정도만 기록하세요.

```markdown
## Day09 — MLOps / Hybrid

- Version Manifest: reports/day09/version_manifest.json
- Day08 Audit 결과:
- Day05 한 변수 변경 결과:
- Day07 Drift FP Rate 변화:
- Day08 One-shot vs Few-shot vs Hybrid OR/AND 결과:
- 가장 중요한 실패 사례와 이유:
- Hybrid 결합의 장점과 한계:
- 실행 Git Commit:
```

# 74. Git에 올릴 파일을 구분합니다

`src/`, `configs/day09/`, `reports/day09/` 중 기관 정책상 허용되는 코드·설정·보고서를 기록합니다. 제품 이미지·데이터셋·모델 파일은 기관의 보안 정책을 우선합니다.

`.gitignore` 예시:

```gitignore
samples/day07/
samples/day08/
samples/day09/
artifacts/day07/
artifacts/day08/
outputs/day09/
__pycache__/
*.pyc
```

# 75. 먼저 코드와 Config를 Commit합니다

```bash
git status
git add src configs/day09 .gitignore
git commit -m "feat: add day09 version experiment drift hybrid comparison"
git log -1 --oneline
```

이미 Commit이 되어 있다면 현재 상태에 맞추어 진행합니다.

# 76. Manifest를 실행 기준 Commit에 맞춥니다

코드와 Config를 Commit한 후:

```bash
python src/71_day09_build_version_manifest.py
```

실험을 수행한 Code/Config Commit을 Manifest에 기록합니다. 이후 결과와 보고서를 Commit합니다.

```bash
git add reports/day09 README.md
git commit -m "docs: record day09 evaluation results"
```

# 77. Git Tag로 기준점을 표시합니다

```bash
git tag -l day09-baseline-v1.0
```

동일 Tag가 없고 Tag를 만들려는 상태가 확정되었으면:

```bash
git tag day09-baseline-v1.0
git tag
```

Tag는 특정 코드·문서 상태를 찾기 위한 이름입니다.

# 78. Rollback은 무엇인가요?

새 설정에 문제가 생기면 이전 안정 버전으로 돌아가는 것입니다. 오늘은 실제 파일을 덮어쓰지 않고 내용만 확인합니다.

```bash
git show day09-baseline-v1.0:configs/day05_logo_rules.json
```

# 79. Baseline을 덮어쓰지 않는 이유

Baseline을 보존해야 **원래 검사기와 새 후보의 차이**, **어느 설정에서 실패했는지**, **되돌릴 기준점**을 확인할 수 있습니다.

# 80. 자가 체크리스트

- [ ] Day05~08 핵심 산출물의 존재와 Schema를 확인했습니다.
- [ ] Manifest를 생성하고 Config / Artifact / Result와 Git 버전을 확인했습니다.
- [ ] Day08 One-shot/Few-shot의 공통 Query·정답·Metric이 일치합니다.
- [ ] Day05 Frozen Baseline은 보존한 채 한 변수만 바꿨습니다.
- [ ] Day07 Reference/Threshold를 고정하고 밝기만 변경했습니다.
- [ ] Day08 CSV의 같은 `sample_id` 20장으로 Hybrid 입력을 만들었습니다.
- [ ] **OR/AND 정책을 Test 결과를 보고 새로 튜닝하지 않았습니다.**
- [ ] 단독 검사와 Hybrid의 Accuracy·Precision·Recall·FP·FN을 비교했습니다.
- [ ] 결합이 성능을 반드시 높이지는 않는다는 사실을 설명할 수 있습니다.
- [ ] Drift Mini Challenge에서 한 변수만 바꿨습니다.
- [ ] 결과를 `day09_analysis.md`·README·Git에 정리했습니다.

# 81. 오늘 배운 내용을 정리합니다

```text
1~8일차의 검사 결과 준비
          ↓
Version / Manifest
          ↓
Day08 결과 재현성 확인
          ↓
Day05 한 변수 Experiment
          ↓
Day07 Data Drift
          ↓
Day08 같은 병 이미지의 두 결과 결합
          ↓
Hybrid OR / AND 성능 비교
          ↓
실패 분석 + Version 기록
```

> **9일차 핵심:** 좋은 검사 결과는 정확도 숫자만으로 설명되지 않습니다. 어떤 데이터와 기준으로 만들었는지 추적하고, 같은 입력에서 비교하며, 결합의 효과와 한계를 검증할 수 있어야 합니다.

## 다음 10일차와 연결합니다

**10일차는 9일차 Hybrid 결과를 서빙하는 수업이 아닙니다.** 6일차에서 만든 OCR 검사와 Frozen Rule을 대표 모듈로 재사용하여 다음 과정을 구현합니다.

```text
6일차 OCR 검사 모듈
        ↓
SN ROI → OCR → Regex → Expected SN
        ↓
Final OK/NG + Reason
        ↓
FastAPI → JSON → Web Dashboard
        ↓
PC · Tablet · Mobile
```

따라서 **이번 9일차 변경은 10일차의 FastAPI 코드·실습 진행에 영향을 주지 않습니다.**
