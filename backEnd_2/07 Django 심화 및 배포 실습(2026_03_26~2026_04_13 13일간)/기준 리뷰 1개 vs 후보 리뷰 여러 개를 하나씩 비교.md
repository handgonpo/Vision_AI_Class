# ✅ 2. 실제로 비교되는 것

코드에서 이 부분이 핵심입니다 👇
```python
similarity_result = FastAPIClient.get_similarity(  
    source_review.content,  
    candidate.content  
)
```
여기서 비교되는 건 딱 2개입니다
- `source_review.content` 기준 리뷰 (1개)
- `candidate.content` 다른 리뷰 (여러 개 중 하나)

지금 이 코드는 파이썬에서는 딕셔너리
```python
results.append({
    "review_id": candidate.id,
    "username": candidate.user.username,
    "content": candidate.content,
    "score": round(similarity_result["similarity"], 4),
    "created_at": candidate.created_at.strftime("%Y-%m-%d %H:%M"),
})
```

```딕셔너리데이터
[
    {
        "review_id": 12,
        "username": "user1",
        "content": "향이 좋아서 만족",
        "score": 0.8721,
        "created_at": "2026-04-01 15:30"
    },
    {
        "review_id": 15,
        "username": "user2",
        "content": "별로였어요",
        "score": 0.4123,
        "created_at": "2026-04-01 14:10"
    }
]
```

전체흐름
```
브라우저 → Django(APIView) → for문 실행
                           ↓
                  FastAPI 여러 번 호출
                           ↓
                  결과 모아서 JSON 반환
                           ↓
                      브라우저
```

# ✅ 3. 단계별로 쪼개서 설명

## ① 브라우저 요청
```
GET /ai/reviews/10/analyze/
```

👉 이 요청이 Django로 들어옴

---

## ② Django에서 for문 실행
```
for candidate in candidate_reviews:
```
👉 여기서 핵심

**이 for문은 Django에서 돌고 있음**

---

## ③ for문 안에서 FastAPI 호출
```
similarity_result = FastAPIClient.get_similarity(  
    source_review.content,  
    candidate.content  
)
```

👉 이게 무슨 의미냐면
```
Django → FastAPI → 결과 받음
```
이걸 **한 번이 아니라 여러 번 반복**

---

## 📌 실제 흐름
```
1번 후보 → FastAPI 호출 → 결과 받음  
2번 후보 → FastAPI 호출 → 결과 받음  
3번 후보 → FastAPI 호출 → 결과 받음  
...
```

👉 즉:

❌ FastAPI에서 for문 도는게 아님  
⭕ Django에서 FastAPI를 여러 번 호출하는 구조

---

# ✅ 4. FastAPI는 뭐만 하냐?

👉 FastAPI는 이것만 함
```
텍스트 2개 받음 → 유사도 계산 → 결과 반환
```
예:
```
{  
  "similarity": 0.8721  
}
```

👉 끝. 반복 안 함

---

# ✅ 5. Django가 하는 역할

👉 Django가 전체 컨트롤러 역할
1. DB에서 리뷰 가져옴  
2. for문 돌림  
3. FastAPI 계속 호출  
4. 결과 모음  
5. 정렬  
6. JSON 만들어서 응답

---

# ✅ 6. “바로 화면으로 가냐?” 질문 답

👉 YES, 하지만 **중간 과정이 있음**
```
FastAPI → Django → 브라우저
```
👉 FastAPI 결과가 바로 화면으로 가는게 아니라

❌ FastAPI → 브라우저 (X)  
⭕ FastAPI → Django → 브라우저 (O)

---

# ✅ 7. 그림으로 이해 (완벽 정리)
```
[브라우저]  
     ↓ 요청  
[Django APIView]  
     ↓  
for문 시작  
     ↓  
[FastAPI] (1번)  
     ↓  
[FastAPI] (2번)  
     ↓  
[FastAPI] (3번)  
     ↓  
결과 리스트 완성  
     ↓  
JSON 응답  
     ↓  
[브라우저 화면]
```
---

# ✅ 8. 한 줄 핵심 정리

👉  
**for문은 Django에서 돌고,  
FastAPI는 그 안에서 “한 번씩 호출되는 계산기 역할”이다**

---

# 🚨 9. 왜 헷갈리냐 (중요 포인트)

헷갈리는 이유 👇

👉 “AI 처리니까 FastAPI가 다 하는거 아닌가?”

❌ 아님  
👉 AI는 계산만 함  
👉 **흐름 제어는 Django가 한다**

---

# 🔥 10. 실무 기준 한 단계 더

지금 구조:
```
for문 → FastAPI 여러 번 호출 (느림)
```
실무 구조:

FastAPI 한 번 호출 → 벡터 여러 개 반환  
→ DB에서 한 번에 비교

👉 여기서 성능이 확 바뀜


---

전체 흐름
① 기준 리뷰 1개 가져옴
```python
source_review = Review.objects.get(id=review_id)
```

예:
이 제품 진짜 좋아요 향이 좋아요

---
② 같은 상품 리뷰 20개 가져옴
```
candidate_reviews = ...
```
예:
1. 향이 좋아서 만족함  
2. 별로였어요 냄새 이상함  
3. 향은 좋은데 지속력 약함  
...

---
③ for문으로 하나씩 비교
```python
for candidate in candidate_reviews:
```

여기서 발생하는 일:
기준 리뷰 vs 리뷰1  
기준 리뷰 vs 리뷰2  
기준 리뷰 vs 리뷰3  
...

---
✅ 4. 실제 비교 방식 (AI 내부)
FastAPI 안에서는 이런 흐름입니다:
```
텍스트 → 임베딩 벡터 변환 → 코사인 유사도 계산
```

예시:
```
"이 제품 진짜 좋아요"  
        ↓  
[0.123, 0.442, 0.992, ...]  
  
"향이 좋아서 만족함"  
        ↓  
[0.120, 0.430, 0.980, ...]  
  
→ cosine similarity 계산  
→ 0.87 (높음)
```

---
✅ 5. for문을 쓰는 이유 (중요)
AI 모델은 기본적으로
❌ 한 번에 여러 문장 비교 안 함 (지금 구조에서는)  
⭕ 두 문장씩 비교하는 구조

그래서:
```
1 vs 1  
1 vs 2  
1 vs 3  
```
...

반복문으로 돌리는 것

---
✅ 6. 이 방식 이름
이 방식은 보통 이렇게 부릅니다:

Brute Force Similarity (완전 탐색)  
또는 Pairwise Comparison (쌍 비교)

---
✅ 7. 한 줄 정리

기준 리뷰 1개를 기준으로, 다른 리뷰들을 하나씩 AI에 보내서 유사도를 계산하는 구조

---
✅ 8. 지금 구조의 특징

장점
- 구현 쉬움
- 이해 쉬움
- 정확도 안정적

단점
- 느림 (리뷰 많아지면 문제)
- FastAPI 호출 여러 번 발생