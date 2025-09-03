# Spring-Portfolio
Spring 백엔드 코드를 보여드리기 위한 프로젝트 입니다.


# 전체 구조(레이어 관점)

- **Experience/BFF**: API Gateway + BFF/GraphQL(스키마 연합), Web/App  
- **Platform / Shared**: 모든 도메인이 공통으로 쓰는 기반  
- **Commerce 도메인**: SKU 판매 흐름  
- **Reservation 도메인**: 시간/자원 기반 예약 흐름  
- **Community/CS**: 리뷰·게시판·고객지원

---

# Platform / Shared (공통, 규제/인프라 성격)

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

# Commerce 도메인 (SKU/수량 재고 중심)

- **Catalog(SKU)**: 상품/옵션/카테고리  
- **Pricing**: 기본가/규칙, 프로모션 엔진 연동  
- **Inventory(Qty)**: 수량 재고, 예약 도메인과 분리  
- **Cart**: 장바구니/사전검증  
- **Order**: 주문 생성/상태, 이벤트 발행(`OrderPlaced`)  
- **Fulfillment/Shipping**: 출고/택배/송장  
- **Returns**: 반품/RMA/환불 연동

---

# Reservation 도메인 (시간×자원 재고 중심)

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

# Community / CS

- **UGC**: 리뷰/평점/게시판/댓글/신고  
- **Moderation**: 금칙어/스팸/휴먼 심사  
- **Customer Support**: 티켓/상담/라우팅
