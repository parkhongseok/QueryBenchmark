# QueryBenchmark

쿼리 성능을 정량적으로 비교하기 위한 실험 및 최적화 프로젝트입니다.  
SNS 서비스에서 자주 사용되는 `피드 목록 조회` 쿼리를 중심으로 다양한 인덱스 구조와 실행 계획을 분석하고, 각 쿼리별 응답 속도 차이를 시각화합니다.

## 실험 목적

- 피드 기반 SNS에서 **정렬, 조건절, 서브쿼리**가 포함된 복잡한 쿼리를 개선
- **EXPLAIN**, **실행 시간 측정**, **인덱스 전략 수립** 등 실전 튜닝 절차 경험
- 정량적 시각화로 쿼리 성능의 변화를 비교

## 실험 환경

- **Database**: MariaDB 10.6
- **데이터 규모**:
  - Users: 12,000명
  - Feeds: 1,000,000개
  - Comments & Likes: 각 100,000개 이상
- **도구**: Python + Matplotlib + Pandas

## 실험 대상 쿼리

```sql
SELECT
    f.id AS feedId,
    u.id AS userId,
    f.content,
    (SELECT COUNT(*) FROM feed_likes l WHERE l.feed_id = f.id),
    (SELECT COUNT(*) FROM comment c WHERE c.feed_id = f.id AND c.deleted_at IS NULL),
    f.created_at
FROM feeds f
JOIN users u ON f.user_id = u.id
WHERE f.id < ? AND f.deleted_at IS NULL
ORDER BY f.created_at DESC
LIMIT 10;
```

## 성능 분석 항목

- 쿼리 실행 시간 (ms)

- 인덱스 사용 여부 및 EXPLAIN 결과

- 인덱스 변경 전후의 응답 시간 비교

- 쿼리 조건 변화에 따른 실행 시간 비교

## 결과

    1000배 성능 개선 : 약 2초 -> 0.002초

## 회고

인덱스가 단순히 성능을 무조건 향상시키는 것은 아니며,
정렬 조건과 WHERE 절을 모두 커버해야 진정한 효과를 본다는 점을 실험을 통해 확인하였습니다.
