# 4. 반응형 프로그래밍 & SwiftUI

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

버딩크는 RxSwift 기반이다. 면접에서는 "Rx를 써봤다"보다 Rx를 왜 썼고, Combine·async/await와 비교하면 어떤지, 마이그레이션은 어떻게 할지를 묻는 경우가 많다. SwiftUI는 최신 트렌드 대응력 확인용.

## 4-1. RxSwift 심화

- **Cold Observable**: 구독할 때마다 새로 실행(네트워크 요청 Single 등). **Hot**: 구독과 무관하게 이벤트 발생(Subject, UI 이벤트).
- **PublishSubject**: 구독 이후 이벤트만. **BehaviorSubject**: 최신값 1개를 새 구독자에게 즉시 전달.
- **Relay(RxRelay)**: error/completed가 없는 Subject → UI 상태 보관용으로 안전.
- **Driver**: 메인 스레드 보장 + 에러 없음 + 공유(replay 1). UI 바인딩 전용.
- **share(replay:)**: 여러 구독자가 같은 실행 결과를 공유 → 중복 네트워크 호출 방지.
- **subscribe(on:)**: 구독(생성) 작업이 실행될 스케줄러. **observe(on:)**: 이후 연산자·구독 결과를 받을 스케줄러.
- **flatMapLatest**: 새 이벤트가 오면 이전 내부 스트림을 취소 → 검색·탭 전환에 사용.

## 4-2. RxSwift ↔ Combine 대응표

| RxSwift | Combine |
|---|---|
| Observable | AnyPublisher |
| PublishSubject / BehaviorSubject | PassthroughSubject / CurrentValueSubject |
| DisposeBag | Set\<AnyCancellable\> |
| observe(on:) | receive(on:) |
| flatMapLatest | map + switchToLatest |
| Driver | receive(on: main) + 에러 처리(replaceError 등) |

## 4-3. 마이그레이션 전략 (면접 단골)

- 한 번에 갈아엎지 않는다. Data 레이어(Repository/API)부터 async/await로 전환하고, Presentation은 Rx 바인딩을 유지하며 경계에서 브리징.
- 신규 화면부터 새 방식 적용 → 테스트로 동작 보장 → 점진 확대.

## 4-4. SwiftUI 핵심

- View는 값(struct)이며 상태가 바뀌면 `body`를 다시 계산. 실제 화면 객체의 생명주기와 View 값의 생명주기는 다르다.
- **Identity**: 구조적 identity(코드 위치)와 명시적 identity(`id`). identity가 바뀌면 상태가 초기화된다.
- `@State`: 뷰가 소유하는 상태. `@Binding`: 상위 상태에 대한 참조.
- `@Observable`(iOS 17, Observation 프레임워크): body에서 실제로 읽은 프로퍼티만 추적해 필요한 뷰만 갱신. 기존 `ObservableObject` + `@Published`는 어떤 프로퍼티든 바뀌면 관찰 뷰 전체 갱신.
- 소유: `@Observable` 객체는 `@State`로 소유, 바인딩이 필요하면 `@Bindable`. (구 방식: `@StateObject` 소유 / `@ObservedObject` 참조)
- **UIKit 연동**: `UIHostingController`로 SwiftUI 화면을 UIKit에 삽입, `UIViewRepresentable`로 UIKit 뷰를 SwiftUI에 삽입 → 기존 UIKit 앱에 점진 도입.

## 버딩크 포트폴리오 연결

- MVVM Input/Output + Driver/Relay 사용 방식을 "UI 바인딩에서 에러로 스트림이 끊기는 문제를 막기 위해"처럼 이유와 함께 설명.
- 버딩크 화면 하나(예: 운동 완료 화면)를 SwiftUI로 옮긴다면 어떻게 할지 설계 답변 준비.

## 예상 질문 & 답변 포인트

- **Q. Subject와 Relay의 차이?**
  → Relay는 error/completed가 없어 UI 상태 스트림이 종료되지 않음.
- **Q. subscribeOn과 observeOn 차이?**
  → 생성·구독 작업 위치 vs 이후 다운스트림 실행 위치.
- **Q. Rx 코드를 async/await로 옮긴다면?**
  → Data 레이어부터 점진 전환, 경계 브리징, 테스트로 회귀 방지.
- **Q. @Observable이 ObservableObject보다 나은 점?**
  → 읽은 프로퍼티 단위 추적으로 불필요한 뷰 갱신 감소, @Published 불필요.

## 검색 키워드

RxSwift hot cold observable · RxRelay Driver · share replay · RxSwift to Combine cheat sheet · SwiftUI view identity · WWDC21 "Demystify SwiftUI" · WWDC23 "Discover Observation in SwiftUI" · UIHostingController UIKit migration
