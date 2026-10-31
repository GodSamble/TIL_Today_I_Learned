# 네이버웹툰 iOS 개발자 군복무 학습 가이드

> 버딩크 포트폴리오(UIKit · RxSwift · Clean Architecture · AVPlayer)를 면접에서 "왜?"까지 설명할 수 있는 이론으로 연결하기

## 1. 타깃 포지션 요약

| 항목 | 내용 (최근 공고 기준) |
|---|---|
| 포지션 | 한국 웹툰 iOS 개발 (경력 1년 이상). 매년 1~2월 공고 반복 (2025.01, 2026.02) |
| 주요 업무 | 웹툰 iOS 앱 주요 기능 설계·개발 / 콘텐츠 뷰어, 결제, SDK / 확장성 있는 아키텍처와 모듈화 / UI·UX 개선 / 성능 이슈 진단, 메모리·속도 최적화 |
| 전형 | 지원서 리뷰 → 프리 인터뷰 → 실무 인터뷰(1:1 다회, Swift·iOS 지식 + 라이브 코딩) → Culture-Fit |
| 근무 | 분당 정자동 그린팩토리 (원격 병행 정책은 지원 시 재확인) |
| 내 목표 | 지원 시점 2028년 1~2월 (전역 2028.03) |

### 1-B. 공고 B: iOS 앱 개발 (라인웹툰·네이버웹툰·시리즈 통합)

- 라인웹툰(글로벌), 네이버웹툰, 웹소설, 시리즈, 시리즈on iOS 개발
- 이미지뷰어, ePub뷰어, 동영상
- Swift/Objective-C 경험, Autolayout, Storyboard, 디자인 패턴 지식, 상용 앱 1개 이상 개발
- API 연동·데이터 처리, 이미지 처리 이해, 성능 최적화·구조 개선, Functional Programming
- 주요 업무: 뷰어, 다국어 / 앱 공통모듈화와 구조 개선
- 우대사항: 다른 언어 학습 능력

버딩크로 커버되지 않는 부분은 [9장 공고 B 보강](09-posting-b.md)에서 다룹니다.

## 2. 공고 키워드 ↔ 챕터 ↔ 버딩크 매핑

| 공고 키워드 | 챕터 | 버딩크에서 꺼낼 사례 |
|---|---|---|
| 메모리 관리 · 안정성 | 1 Swift 심화 | RxSwift 구독의 `[weak self]`, DisposeBag |
| 성능 · 앱 속도 최적화 | 2 동시성 · 3 UIKit · 7 성능 | Swift Concurrency context 이슈 해결, Diffable 대시보드, ShortsFeed 스켈레톤 |
| 콘텐츠 뷰어 | 3 UIKit · 6 미디어·뷰어 | AVPlayerLayer presentationSize KVO, 피드백 이미지 마크업·줌 |
| 결제 | 6 미디어·뷰어 (StoreKit) | 코칭 상품 Apple IAP 정책 대응, 가격 표시·멤버십 만료 로직 |
| 아키텍처 · 모듈화 · SDK | 4 반응형 · 5 아키텍처 | Clean Architecture + DIContainer, Moya/Realm 레이어 분리 |
| 라이브 코딩 | 8 CS 기본기 | Swift로 알고리즘 풀이 |
| ePub뷰어 · 다국어 | 9 공고 B 보강 | 강의 콘텐츠 뷰어 경험을 확장 (신규 학습) |
| Objective-C · Storyboard | 9 공고 B 보강 | 코드 기반(SnapKit) UI와의 비교로 설명 (신규 학습) |
| 디자인 패턴 · FP | 9 공고 B 보강 · 4 반응형 | Delegate/Coordinator, RxSwift 연산자 체인 |

## 3. 이 가이드 사용법

- 각 챕터 = 왜 중요한가 → 핵심 개념 → 버딩크 연결 → 예상 질문 → 검색 키워드
- Xcode가 없는 환경이므로 개념마다 "정의 1줄 → 내 코드 사례 → 트레이드오프" 3단 노트를 만든다.
- 면접 답변은 1분 버전으로 말해보기. 라이브 코딩은 반드시 Swift로.
- Swift/Xcode 버전별 동작은 바뀔 수 있으니 Apple 공식 문서(developer.apple.com/documentation)와 WWDC 세션으로 최종 확인한다.

## 챕터 목차

1. [Swift 언어 심화](01-swift-deep-dive.md)
2. [동시성 (GCD → Swift Concurrency)](02-concurrency.md)
3. [UIKit 내부 원리 & 리스트 성능](03-uikit-performance.md)
4. [반응형 프로그래밍 & SwiftUI](04-reactive-swiftui.md)
5. [아키텍처 · 모듈화 · 테스트](05-architecture-testing.md)
6. [웹툰 직결: 콘텐츠 뷰어 · 미디어 · 결제](06-viewer-media-payment.md)
7. [성능 측정 · 보안 · 배포](07-performance-security-release.md)
8. [CS 기본기 & Swift 라이브 코딩](08-cs-live-coding.md)
9. [공고 B 보강: ePub · 다국어 · Objective-C · Storyboard · 패턴 · FP](09-posting-b.md)
- [부록: 면접 스토리 & 15개월 로드맵](appendix.md)

> 공고·전형 정보는 2025~2026년 공개 공고와 후기 기준이며 변경될 수 있습니다.
