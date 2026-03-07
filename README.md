# 🏦 AI Finance Guard - 지능형 구독 및 자산 관리 에이전트

여러 곳에 흩어진 구독 서비스와 지출 내역을 **AI로 자동 분석**하여 불필요한 지출을 차단하고 미래 자산을 예측합니다.

## ✨ 핵심 기능

### 📸 **OCR 영수증 스캔**
- 영수증 사진 → Claude Vision API → 자동 정보 추출
- 상점명, 금액, 구매 날짜, 결제 수단 자동 인식
- 품목 단위 추출 지원

### 💸 **지출 관리 & 자동 분류**
- 모든 지출을 SQLite에 저장
- 2단계 카테고리 분류:
  1. 규칙 기반 (스타벅스→카페) - 빠르고 API 비용 없음
  2. Claude 폴백 - 규칙 미매핑 항목만 처리
- 월별/카테고리별 리포트 생성

### 🎫 **구독 자동 감지**
- 지출 패턴 분석
- 조건: 3회 이상 + 금액 변동 <10% + 일정 간격
- 월간/연간 구독 자동 분류
- 신뢰도 지수(0.0~1.0) 제공

### 📊 **예산 예측**
- 가중 이동평균 (최근 3개월: 20%, 30%, 50%)
- 신뢰 구간(95%) 계산
- 카테고리별 예측 지원

### 💡 **저렴한 대안 추천**
- "Netflix" → "Tving, 왓챠, 쿠팡플레이" 등
- 월 절약액 자동 계산
- 각 대안의 장단점 분석

### 💰 **저축 목표 추적**
- 월 저축액 계산
- 단리/복리 예측 (연 3%)
- 목표 금액 달성 기간 계산

## 🚀 설치 및 사용

### 필수 요구사항
- FreeLang v2.2.0+ (Node.js 기반)
- ANTHROPIC_API_KEY 환경변수

### 설치
```bash
git clone https://gogs.dclub.kr/kim/ai-finance-guard.git
cd ai-finance-guard

# .env 파일 생성 (API 키 설정)
cp .env.example .env
# ANTHROPIC_API_KEY=sk-... 추가
```

### 기본 사용법

#### 📸 영수증 스캔
```bash
freelang main.fl scan --image receipt.jpg
# 자동 추출 + DB 저장 + 카테고리 분류
```

#### 💸 지출 수동 추가
```bash
freelang main.fl add --amount 15000 --store "스타벅스" --date 2026-03-07
freelang main.fl add --amount 35000 --store "배달의민족" --category 외식
```

#### 📋 지출 목록 조회
```bash
freelang main.fl list                    # 전체 목록
freelang main.fl list --month 2026-02    # 2월 지출
freelang main.fl report                  # 월별 리포트 (카테고리 분석)
```

#### 📊 예산 예측
```bash
freelang main.fl predict              # 다음 달 예산 예측 (기본값)
freelang main.fl predict --income 3000000  # 월 소득 3백만원 기준 저축 예측
```

#### 🎫 구독 자동 감지
```bash
freelang main.fl sub detect   # 지출 패턴에서 구독 후보 찾기
freelang main.fl sub list     # 등록된 구독 서비스 목록
freelang main.fl sub upcoming # 다가올 갱신 목록
```

#### 💡 대안 서비스 추천
```bash
freelang main.fl recommend --name "Netflix"
# Netflix 대신 더 저렴한 대안 서비스 추천
```

#### 💰 저축 목표
```bash
freelang main.fl savings                  # 기본 저축 분석
freelang main.fl savings --target 10000000  # 천만원 목표까지의 기간 계산
```

## 📁 프로젝트 구조

```
ai-finance-guard/
├── .env                          # 환경 설정 (ANTHROPIC_API_KEY)
├── .gitignore                    # Git 무시 규칙
├── finance.db                    # SQLite 데이터베이스 (자동 생성)
│
├── claude_client.fl              # Claude API REST 래퍼
│   ├── call_claude(prompt, model)
│   ├── call_claude_vision(prompt, image_b64, media_type)
│   └── extract_json(response_text)
│
├── expense_tracker.fl            # SQLite 관리
│   ├── init_db()
│   ├── add_expense(...)
│   └── get_monthly_expenses(year, month)
│
├── ocr_processor.fl              # 영수증 OCR
│   ├── image_to_base64(image_path)
│   └── extract_receipt(image_path)
│
├── category_classifier.fl        # 카테고리 분류 (규칙 + Claude)
│   ├── classify_by_rule(store_name)
│   ├── classify_by_claude(store_name, amount)
│   └── classify(store_name, amount)
│
├── budget_predictor.fl           # 예산 예측
│   ├── calc_std(values)
│   ├── weighted_moving_average(monthly_totals)
│   └── predict_savings(income, expense, months)
│
├── subscription_manager.fl       # 구독 관리
│   ├── add_subscription(name, amount, cycle, ...)
│   ├── detect_recurring(expenses)
│   └── get_total_monthly_cost()
│
├── recommender.fl                # 대안 추천
│   ├── recommend_alternatives(service_name, price)
│   └── analyze_subscriptions(subscriptions)
│
├── formatters.fl                 # CLI 출력
│   ├── print_table(headers, rows)
│   ├── print_monthly_report(...)
│   └── print_savings_forecast(...)
│
├── finance_agent.fl              # 메인 오케스트레이터
│   ├── init()
│   ├── scan_receipt(image_path)
│   ├── add_expense(...)
│   ├── detect_subscriptions()
│   └── recommend_for_service(...)
│
└── main.fl                       # CLI 진입점
    ├── main()
    ├── get_flag(flag_name)
    └── print_help()
```

## 🗄️ 데이터베이스 스키마

### expenses (지출)
```
id, date, store_name, amount, category, payment_method,
note, source, receipt_image_path, created_at
```

### subscriptions (구독)
```
id, name, amount, cycle, monthly_cost, next_renewal_date,
is_active, auto_detected, notes, created_at
```

### subscriptions_history (구독 이력)
```
id, subscription_id, renewal_date, amount, status
```

### recurring_candidates (자동 감지 후보)
```
id, store_name, avg_amount, interval_days, occurrences,
confidence, suggested_cycle, is_confirmed
```

### budgets, income_records 등 6개 테이블

## 🤖 AI 기술 스택

- **Claude API 3.5 Sonnet** - 텍스트 분류, 자동 추천
- **Claude Vision API** - 영수증 OCR (base64 이미지)
- **가중 이동평균** - 시계열 예측
- **StandardScaler** - 금액 정규화
- **FreeLang stdlib** - 순수 로컬 처리 (API 의존성 최소)

## 💾 로컬 저장 & 보안

✅ **로컬 SQLite 데이터베이스** - 클라우드 업로드 없음
✅ **환경변수 API 키** - .env 파일로 보안 관리
✅ **.gitignore** - .env, *.db 자동 제외

## 📈 사용 예시

### 시나리오 1: 월별 지출 분석
```bash
# 1. 영수증 스캔
freelang main.fl scan --image 스타벅스.jpg
freelang main.fl scan --image 삼겹살.jpg

# 2. 월별 리포트
freelang main.fl report --month 2026-02

# 출력:
# 📊 2026-02 지출 리포트
# ════════════════════════════
#   카페: 45000원 (15%)
#   외식: 180000원 (60%)
#   식료품: 75000원 (25%)
# 총 지출: 300000원
```

### 시나리오 2: 구독 최적화
```bash
# 1. 구독 자동 감지
freelang main.fl sub detect

# 출력:
# 📌 구독 서비스 후보 (3개):
#   - Netflix: 17000원/월
#   - Spotify: 10900원/월
#   - Coupang: 9900원/월

# 2. 대안 추천
freelang main.fl recommend --name "Netflix"

# 출력:
# 💡 Netflix의 대안 서비스:
#   - Tving: 13900원/월
#   - 왓챠: 12900원/월
# 🏆 추천: Tving (월 3100원 절약)
```

### 시나리오 3: 저축 계획
```bash
freelang main.fl predict --income 3000000

# 출력:
# 💡 다음 달 예산 예측
# ────────────────────────
# 예상 지출: 1,200,000원
# 신뢰 구간: 1,050,000 ~ 1,350,000원

# 💰 저축 예측 (6개월)
# 월 저축액: 1,800,000원
# 단리 합계: 10,800,000원
# 복리 합계: 10,853,400원 (연 3%)
```

## 🔧 개발 참조

### FreeLang 패턴
- `import { functionName } from "./stdlib/module"` - 함수 임포트
- `import moduleName from "./stdlib/path"` - 모듈 임포트
- `let var = value` - 변수 선언
- `fn funcName(params) { ... }` - 함수 정의

### CLI 인자 파싱
```fl
fn get_flag(flag_name) {
  let i = 0
  while i < args.length {
    if (args[i] == flag_name && i + 1 < args.length) {
      return args[i + 1]
    }
    i = i + 1
  }
  return null
}
```

### Claude API 호출
```fl
let response = call_claude("프롬프트", "claude-3-5-sonnet-20241022")
let json_result = extract_json(response)
```

## 📝 라이선스
MIT License

## 👤 저자
Claude (Anthropic) - FreeLang 구현
Kim (사용자) - AI Finance Guard 설계

## 🎯 향후 계획

- [ ] pattern_analyzer.fl - 고급 지출 패턴 분석
- [ ] PDF 영수증 지원 (OCR)
- [ ] 가족 공유 모드 (다중 사용자)
- [ ] 웹 대시보드 (Next.js)
- [ ] 모바일 앱 (React Native)
- [ ] 구독 자동 취소 기능
- [ ] 신용카드 자동 연동

## 📞 지원

문제 발생 시: https://gogs.dclub.kr/kim/ai-finance-guard/issues
