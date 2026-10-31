# 3. UIKit 내부 원리 & 리스트 성능

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

웹툰 앱은 세로로 긴 이미지 스크롤이 핵심 경험이다. 스크롤 끊김(hitch), 메모리 급증, 셀 재사용 버그를 원리로 설명하고 해결할 수 있어야 한다. 공고의 "콘텐츠 뷰어, 메모리 관리 및 앱 속도 최적화"와 직결된다.

## 3-1. 생명주기

- **앱/씬**: Unattached → Foreground Inactive → Foreground Active → Background → Suspended. iOS 13부터 SceneDelegate가 UI 생명주기를 담당.
- **ViewController**: `loadView` → `viewDidLoad`(1회) → `viewWillAppear` → `viewIsAppearing`(iOS 17 도입, iOS 13까지 백디플로이) → `viewWillLayoutSubviews` / `viewDidLayoutSubviews` → `viewDidAppear` → `viewWillDisappear` → `viewDidDisappear`.
- 프레임 기반 계산은 `viewDidLoad`가 아니라 레이아웃 이후(`viewIsAppearing` / `viewDidLayoutSubviews`)에서.

## 3-2. RunLoop

- 메인 스레드는 RunLoop로 이벤트(터치, 타이머, 소스)를 처리하며 대기한다.
- 스크롤 중에는 RunLoop가 tracking 모드로 바뀌어 default 모드에 등록된 Timer가 멈춘다 → `.common` 모드로 등록하면 해결.

## 3-3. 화면 업데이트 사이클 (Render Loop)

- 이벤트 처리 후 RunLoop 끝에서 Layout(`layoutSubviews`) → Display(`draw`) → Commit(Core Animation 트랜잭션을 Render Server로 전송) → GPU 렌더.
- `setNeedsLayout`: 다음 사이클에 레이아웃 예약. `layoutIfNeeded`: 예약된 레이아웃을 즉시 수행(애니메이션에 사용). `setNeedsDisplay`: 다시 그리기 예약.
- **프레임 예산**: 60Hz는 약 16.67ms, 120Hz(ProMotion)는 약 8.33ms 안에 한 프레임을 끝내야 한다. 넘으면 hitch(끊김).
- **오프스크린 렌더링**: 별도 버퍼에 먼저 그린 뒤 합성하는 비싼 작업. `shadowPath` 없는 그림자, mask, 일부 `cornerRadius` + `clipsToBounds` 조합 등에서 발생. 해결: `shadowPath` 지정, 미리 가공한 이미지 사용.

## 3-4. Auto Layout 비용

- 제약 조건을 연립 방정식처럼 풀어 프레임을 계산. 제약 추가/삭제(constraint churn)를 반복하면 비용이 커진다.
- 셀 configure마다 `remakeConstraints`(SnapKit)를 호출하는 패턴은 지양. 제약은 init에서 한 번 만들고 `constant`·`isActive`만 변경.

## 3-5. Diffable Data Source & Compositional Layout

- **Snapshot**: 섹션·아이템 식별자(Hashable)의 상태. apply 시 이전 스냅샷과 비교해 삽입/삭제/이동을 자동 계산·애니메이션.
- **identity 함정**: 모델 전체를 Hashable 아이템으로 쓰면 내용이 바뀔 때 "다른 아이템"으로 인식돼 삭제+삽입 애니메이션이 일어남. ID만 아이템으로 쓰고 내용 변경은 `reconfigureItems`(iOS 15+, 셀 재생성 없이 갱신) 또는 `reloadItems`로.
- apply는 항상 같은 큐(보통 메인)에서 일관되게 호출.
- **Compositional Layout**: Item → Group → Section 조합. 섹션별 다른 레이아웃, `orthogonalScrollingBehavior`로 가로 스크롤 섹션.

## 3-6. 셀 재사용 · Prefetching · 이미지 메모리

- **재사용 버그**: 비동기 이미지가 늦게 도착해 다른 셀에 표시됨 → `prepareForReuse`에서 요청 취소 + 결과 반영 전 식별자 비교.
- **Prefetching**: `UICollectionViewDataSourcePrefetching`로 곧 보일 셀의 데이터를 미리 요청, `cancelPrefetching`으로 취소.
- **디코딩된 이미지 메모리 ≈ 가로px × 세로px × 4바이트**. 파일 크기와 무관. 예: 1080×10000 이미지 ≈ 41MB.
- **다운샘플링**: ImageIO의 `CGImageSourceCreateThumbnailAtIndex`로 화면 크기(포인트 × scale)에 맞춰 디코딩하면 메모리가 크게 준다.
- **NSCache**: 메모리 압박 시 자동으로 비워지는 캐시, 스레드 안전. 메모리 캐시 + 디스크 캐시 2단 구성이 일반적.
- 디코딩은 메인 스레드 밖에서 (iOS 15+ `UIImage.prepareForDisplay` / `byPreparingForDisplay()`).

```swift
func downsample(url: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage? {
    let opts = [kCGImageSourceShouldCache: false] as CFDictionary
    guard let src = CGImageSourceCreateWithURL(url as CFURL, opts) else { return nil }
    let maxPixel = max(pointSize.width, pointSize.height) * scale
    let thumbOpts = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxPixel
    ] as CFDictionary
    guard let cg = CGImageSourceCreateThumbnailAtIndex(src, 0, thumbOpts) else { return nil }
    return UIImage(cgImage: cg)
}
```

## 버딩크 포트폴리오 연결

- TrainerDashboard·WorkoutLog의 Diffable 구현: 아이템 식별자를 무엇으로 잡았는지, 갱신을 어떻게 했는지 설명.
- ShortsFeed 스켈레톤 UI: 로딩 체감 개선 + 셀 재사용 시 상태 초기화 방법.
- 피드백 이미지 마크업·줌: 큰 이미지의 메모리 처리 방식 점검(다운샘플링 적용 여부).

## 예상 질문 & 답변 포인트

- **Q. setNeedsLayout과 layoutIfNeeded 차이?**
  → 전자는 다음 사이클 예약, 후자는 즉시 수행. 애니메이션 블록 안에서 layoutIfNeeded.
- **Q. 스크롤 중 타이머가 멈추는 이유?**
  → RunLoop가 tracking 모드로 전환. common 모드로 등록.
- **Q. 긴 웹툰 이미지를 그대로 띄우면 생기는 문제?**
  → 디코딩 메모리가 픽셀 수에 비례해 폭증. 다운샘플링·분할·캐시 정책 필요.
- **Q. Diffable에서 셀 내용만 바뀌었는데 깜빡이는 이유?**
  → 모델 전체 해시로 identity가 바뀜. ID 기반 아이템 + reconfigureItems.
- **Q. 오프스크린 렌더링이란?**
  → 합성 전 별도 버퍼 렌더. 그림자는 shadowPath로 회피.

## 검색 키워드

UIViewController lifecycle viewIsAppearing · RunLoop common mode · iOS render loop hitch · offscreen rendering shadowPath · Diffable data source reconfigureItems · WWDC18 "Image and Graphics Best Practices" · WWDC21 "Make blazing fast lists and collection views" · WWDC20 "Eliminate animation hitches with XCTest"
