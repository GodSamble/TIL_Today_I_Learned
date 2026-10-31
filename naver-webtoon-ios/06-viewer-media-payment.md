# 6. 웹툰 직결: 콘텐츠 뷰어 · 미디어 · 결제

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

공고 업무 첫 줄이 "콘텐츠 뷰어, 결제, SDK"다. 다른 지원자와 차별화되는 챕터이자, 버딩크의 AVPlayer·IAP 경험이 그대로 이어지는 영역이다.

## 6-1. 웹툰 뷰어 설계 포인트

- **구조**: 한 회차 = 세로로 긴 이미지 여러 장(컷). UICollectionView/UITableView로 컷 단위 셀 구성 → 재사용으로 메모리 일정 유지.
- **메모리**: 각 컷을 화면 너비 × scale에 맞춰 다운샘플링. 보이지 않는 컷은 메모리 캐시에서 내려도 되게 설계.
- **프리패치**: 현재 위치 기준 다음 N컷 미리 다운로드·디코딩, 빠른 스크롤 시 취소.
- **캐시 계층**: 메모리(NSCache) → 디스크 → 네트워크. Kingfisher/Nuke가 쓰는 구조를 개념적으로 이해.
- **셀 높이**: 이미지 비율을 미리 알면(서버 메타데이터) 레이아웃 점프 없이 스크롤 위치 유지 가능.
- **확대/축소**: UIScrollView의 zoom(`viewForZooming`) 활용. 단일 초대형 이미지는 `CATiledLayer`로 보이는 타일만 렌더하는 방식도 존재.
- 텍스트 콘텐츠(웹소설)는 이미지 뷰어와 달리 ePub 뷰어 구조가 필요 → [9장](09-posting-b.md) 참고.
- **유료 콘텐츠 보호·오프라인 보관**: 저장 파일 암호화, Data Protection(파일 보호 등급) 같은 고려사항을 "이런 걸 고민하겠다" 수준으로 언급.

## 6-2. AVFoundation (영상 콘텐츠)

- **구성**: `AVAsset`(미디어) → `AVPlayerItem`(재생 상태) → `AVPlayer`(재생 제어) → `AVPlayerLayer`(화면 출력).
- **상태 관찰**: `AVPlayerItem.status`(readyToPlay/failed), `AVPlayer.timeControlStatus`(playing/paused/`waitingToPlayAtSpecifiedRate` = 버퍼링), `addPeriodicTimeObserver`로 진행 시간.
- **presentationSize**: 실제 영상 표시 크기. 세로/가로 영상 판별과 레이아웃 조정에 사용(버딩크 KVO 해결 사례).
- **비동기 로딩**: iOS 15+ `try await asset.load(.duration, .tracks)` 형태의 async API.
- **HLS**: m3u8 플레이리스트 + 짧은 세그먼트. 네트워크 대역폭에 따라 화질을 바꾸는 ABR. `preferredForwardBufferDuration`, `preferredPeakBitRate`로 버퍼·화질 제어.
- **피드형 재생**: 화면에 보이는 셀만 재생, 플레이어 재사용(풀링), 화면 밖에서 pause·해제로 메모리 관리.

## 6-3. StoreKit 2 (쿠키·결제)

- **상품 유형**: 소모성(consumable, 쿠키 충전형), 비소모성, 자동 갱신 구독.
- **흐름**: `Product.products(for:)` 조회 → `product.purchase()` → 결과의 `VerificationResult` 검증 → 서버에 지급 반영 → `transaction.finish()`.
- **Transaction.updates**: 앱 실행 시 리스너를 띄워 앱 밖에서 완료된 결제·환불 등 미처리 트랜잭션을 처리. 빠뜨리면 결제했는데 지급 안 되는 사고.
- **서버 검증**: App Store Server API / Server Notifications로 서버가 최종 판단. 클라이언트만 믿지 않는다.
- **finish 시점**: 서버 지급이 완료된 후 finish. 그 전 실패 시 재시도 가능하게.

```swift
func buy(_ product: Product) async throws {
    let result = try await product.purchase()
    switch result {
    case .success(let verification):
        guard case .verified(let tx) = verification else { throw PurchaseError.unverified }
        try await server.grant(transactionID: tx.id)   // 서버 지급 먼저
        await tx.finish()                              // 그다음 finish
    case .userCancelled, .pending: break
    @unknown default: break
    }
}
```

## 버딩크 포트폴리오 연결

- **AVPlayerLayer presentationSize KVO**: "세로 영상이 잘못 렌더링 → presentationSize가 준비 시점 이후에 확정됨을 파악 → KVO로 관찰해 레이아웃 갱신" 스토리를 대표 사례로.
- MediaConvert 하드코딩 해상도 제거로 세로 영상 지원한 서버 측 경험까지 연결하면 "클라이언트-서버 양쪽을 아는 개발자" 포지셔닝.
- 코칭 상품의 Apple IAP 정책 대응, 멤버십 만료 로직 → 결제 도메인 이해도로 어필.

## 예상 질문 & 답변 포인트

- **Q. 긴 웹툰 회차를 메모리 문제 없이 보여주려면?**
  → 컷 단위 셀 재사용, 다운샘플링, 프리패치·취소, 메모리/디스크 캐시 계층.
- **Q. 영상이 버퍼링 중인지 어떻게 알죠?**
  → timeControlStatus의 waitingToPlayAtSpecifiedRate 관찰.
- **Q. 결제 후 앱이 종료되면?**
  → Transaction.updates 리스너로 다음 실행 시 미처리 트랜잭션 처리, 서버 지급 후 finish.
- **Q. HLS의 장점?**
  → 세그먼트 스트리밍, ABR로 네트워크 상황 대응, 표준 지원.

## 검색 키워드

iOS image viewer large image memory · CATiledLayer · Kingfisher Nuke cache architecture · AVPlayer timeControlStatus KVO · HLS adaptive bitrate · StoreKit 2 Transaction.updates finish · App Store Server API · WWDC21 "Meet StoreKit 2" · WWDC22 "Create a great video playback experience"
