# Spring-Portfolio
Spring 백엔드 코드를 보여드리기 위한 프로젝트 입니다.

# 초기 구성

## 아키텍처 개요 — 7팩 모듈러 모놀리스 (→ 점진적 MSA)

> 초기엔 7개 **모듈**로 경계를 세우고 하나의 배포 단위로 운영.  
> 트래픽/규제/릴리즈 독립성 필요 시 모듈을 **서비스로 추출**한다.

### Platform / Shared (4)
1. **Identity & Users**
   - Auth(OIDC/OAuth), RBAC, 사용자 프로필/주소록, 알림 설정
2. **Payments & Wallet**
   - 결제 Authorize/Capture/Void/Refund, Vault, 3DS, 멱등키
   - Wallet/Loyalty(포인트/크레딧) 원장
3. **Promotion & Risk**
   - 쿠폰/캠페인 룰(스코프: `SKU`/`BOOKABLE`)
   - Fraud/Risk 룰, Tax/VAT 훅(계산은 도메인별 확장 포인트)
4. **Experience Infra**
   - Notification(Email/SMS/Push) + 템플릿
   - Media(업로드/리사이즈/CDN)
   - Search & Reco(인덱서/Query API/개인화—읽기 전용)

### 도메인 (3)
5. **Commerce** — SKU/수량 재고 중심
   - Catalog, Pricing(기본가), Inventory(Qty), Cart, Order, Fulfillment, Returns
6. **Reservation** — 시간×자원 재고 중심
   - Listing/Property, Rates & Plans, Availability, Allocation/Hold, Booking Engine(사가), Policies, Channel Manager, Payout/Settlement
7. **Community / CS**
   - UGC(리뷰/게시판/댓글/신고), Moderation, Customer Support(티켓)

---

## 경계 원칙 (핵심)
- **데이터 소유권**: 모듈별 DB 스키마(초기엔 한 클러스터 OK). 교차 FK 금지, 앱 레벨 무결성.
- **돈 흐름 분리**: 고객 결제는 `Payments & Wallet`, 호스트/가맹점 정산은 `Reservation.Payout`.
- **규칙 계산**: 환불/위약금/요금은 **도메인**(예: `Reservation.Policies`)이 계산 → `Payments.refund(amount)` 호출.
- **검색/추천/미디어/알림**은 공통화하되 **읽기 API 얇게**, 쓰기 모델은 도메인에 둔다(CQRS).


---

# 최종 구성

## 전체 구조(레이어 관점)

- **Experience/BFF**: API Gateway + BFF/GraphQL(스키마 연합), Web/App  
- **Platform / Shared**: 모든 도메인이 공통으로 쓰는 기반  
- **Commerce 도메인**: SKU 판매 흐름  
- **Reservation 도메인**: 시간/자원 기반 예약 흐름  
- **Community/CS**: 리뷰·게시판·고객지원

---

## Platform / Shared (공통, 규제/인프라 성격)

- **Identity/Auth**: 로그인, OAuth, RBAC  
- **User Profile**: 주소록/결제수단 토큰/알림설정  
- **Payments**: Authorize/Capture/Void/Refund, Vault, 3DS, 멱등키  
- **Promotion Engine**: 쿠폰/캠페인(스코프: SKU, BOOKING)  
- **Loyalty/Wallet**: 포인트/크레딧/바우처  
- **Notification**: 이메일/SMS/푸시 + 템플릿  
- **Media**: 업로드/리사이즈/CDN  
- **Search Indexer & Query API**: 인덱싱 파이프라인 + 검색 API(컬렉션 분리)  
- **Recommendation**: 개인화/랭킹(읽기 전용)  
- **Fraud & Risk**: 이상거래/차단 룰/ML  
- **Tax & Compliance**: VAT/전자세금계산서/감사로그  

> **핵심 포인트**: 결제/보안/검색 같은 고복잡·고재사용 요소는 **무조건 플랫폼으로 분리**.

---

## Commerce 도메인 (SKU/수량 재고 중심)

- **Catalog(SKU)**: 상품/옵션/카테고리  
- **Pricing**: 기본가/규칙, 프로모션 엔진 연동  
- **Inventory(Qty)**: 수량 재고, 예약 도메인과 분리  
- **Cart**: 장바구니/사전검증  
- **Order**: 주문 생성/상태, 이벤트 발행(`OrderPlaced`)  
- **Fulfillment/Shipping**: 출고/택배/송장  
- **Returns**: 반품/RMA/환불 연동

---

## Reservation 도메인 (시간×자원 재고 중심)

- **Listing/Property**: 숙소/객실타입/호스트 메타  
- **Rates & Plans**: 요금 캘린더/성수기/프로모션 연결  
- **Availability**: 날짜·타임슬롯 재고 모델  
- **Allocation/Hold**: 재고 홀드/만료(짧은 TTL), 멱등  
- **Booking Engine**: 예약 생성·확정·취소(사가 오케스트레이터)  
- **Policies**: 취소/노쇼/환불 규칙 계산  
- **Channel Manager**: 외부 OTA 재고/요금 동기화  
- **Payout/Settlement**: 호스트 정산/수수료/세금  
- **Host Portal API**: 호스트 달력/콘텐츠 관리  

> **돈 흐름 분리**: 고객 결제는 **Payments(플랫폼)**, 호스트 송금/정산은 **Payout/Settlement(예약 쪽)**.

---

## Community / CS

- **UGC**: 리뷰/평점/게시판/댓글/신고  
- **Moderation**: 금칙어/스팸/휴먼 심사  
- **Customer Support**: 티켓/상담/라우팅
