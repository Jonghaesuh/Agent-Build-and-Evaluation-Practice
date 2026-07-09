# 고객 이탈 예측 모델링 결과

## 데이터 요약
- 원본 행 수: 500
- 중복 제거 후 행 수: 500
- 타깃 비율(이탈): 0.050
- 주요 처리:
  - customer_id 중복 제거(마지막 레코드 유지)
  - plan_type 미지 카테고리를 `unknown` 처리
  - support_tickets_90d 결측은 0으로 해석
  - last_login_days_ago 결측은 중위값으로 대체
  - monthly_payment 이상치는 IQR 기준으로 clipping

## 모델 성능
- logistic_regression: AUC 0.4552, AP 0.0685
- random_forest: AUC 0.7059, AP 0.3070
- hist_gbdt: AUC 0.5966, AP 0.0794

## 선택 모델
- random_forest (AUC 0.7059)

## 해석 가능한 이탈 요인
- 최근 로그인 지연이 길수록 위험 증가 가능
- 지원 티켓 수가 많을수록 이탈 위험 증가 가능
- 신규 가입자와 기존 고객의 이용 패턴 차이 반영
