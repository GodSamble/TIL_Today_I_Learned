# 2. 동시성 (GCD → Swift Concurrency)

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

웹툰 뷰어는 이미지 다운로드·디코딩·캐싱이 동시에 일어나고, 결제는 상태 경합이 치명적이다. 공고의 "성능 최적화, 안정성"을 동시성 지식으로 증명해야 한다. 버딩크에서 직접 해결한 Concurrency 이슈는 가장 강한 카드다.

## 2-1. GCD 기본

- **Serial queue**: 한 번에 하나씩 순서대로. **Concurrent queue**: 여러 작업 동시 실행.
- **sync**: 작업이 끝날 때까지 호출한 스레드를 막음. **async**: 바로 반환.
- **main queue**: 메인 스레드에서 도는 serial queue. UIKit 업데이트는 반드시 여기서.
- **데드락 대표 예**: 메인 스레드에서 `DispatchQueue.main.sync { }` 호출 → 자기 자신을 기다려 멈춤.
- **QoS**: userInteractive > userInitiated > utility > background. 우선순위 역전 주의.

## 2-2. Race Condition과 스레드 안전성

- 여러 스레드가 같은 가변 상태를 동시에 읽고/쓰면 결과가 실행 순서에 따라 달라짐 → 크래시·데이터 손상.
- 해결 수단: serial queue로 접근 직렬화, NSLock 등 락, 그리고 Swift Concurrency의 actor.

## 2-3. Swift Concurrency 핵심

- **async/await**: `await`는 중단 지점(suspension point). 스레드를 막지 않고 작업을 멈췄다가 재개. 재개되는 스레드는 같은 스레드라는 보장이 없다.
- **Task { }**: 현재 actor 컨텍스트와 우선순위를 상속. **Task.detached**: 상속하지 않음(특별한 이유가 없으면 지양).
- **구조적 동시성**: `async let`, `TaskGroup`은 부모 작업 안에서 자식 작업 생명주기가 관리됨. 부모 취소 시 자식도 취소.
- **취소는 협력적**: 자동으로 멈추지 않음. `Task.isCancelled` / `try Task.checkCancellation()`으로 직접 확인. (셀 재사용 시 이미지 로딩 취소에 활용)
- **actor**: 내부 가변 상태를 한 번에 하나의 작업만 접근하도록 격리. 외부에서는 `await`로 접근.
- **actor 재진입(reentrancy)**: actor 메서드 안에서 `await`하는 동안 다른 호출이 끼어들 수 있음 → await 전후로 상태가 바뀌었을 수 있다고 가정하고 작성.
- **@MainActor**: 메인 스레드에서 실행되는 전역 actor. ViewController, ViewModel의 UI 상태에 적용.
- **Sendable**: 동시성 경계를 안전하게 넘을 수 있는 타입 표시. 불변 struct는 대체로 자동 충족.

## 2-4. Swift 6 / 6.2 변화 (Xcode 26)

- Swift 6 언어 모드: 데이터 레이스 가능성을 컴파일 타임 에러로 검출(strict concurrency).
- Swift 6.2 Approachable Concurrency: 빌드 설정의 Approachable Concurrency와 Default Actor Isolation(MainActor/nonisolated)으로 구성. Xcode 26의 신규 프로젝트는 MainActor 기본 격리 + Approachable Concurrency가 켜진 상태로 생성된다.
- 이 모드에서는 nonisolated async 함수가 호출한 쪽의 actor에서 실행(`nonisolated(nonsending)` 기본화). 기존처럼 백그라운드로 보내려면 `@concurrent`를 명시.
- 즉 "async = 자동 백그라운드"라는 가정이 깨진다. 무거운 작업(이미지 디코딩 등)은 의도적으로 오프로딩해야 한다.

## 2-5. 브리징과 실전 패턴

```swift
// 콜백 API → async 브리징 (resume은 정확히 한 번)
func fetchProfile() async throws -> Profile {
    try await withCheckedThrowingContinuation { cont in
        provider.request(.profile) { result in
            switch result {
            case .success(let res): cont.resume(with: Result { try res.map(Profile.self) })
            case .failure(let err): cont.resume(throwing: err)
            }
        }
    }
}
```

```swift
// 토큰 갱신 중복 방지: 진행 중인 갱신 Task를 공유
actor TokenRefresher {
    private var task: Task<String, Error>?
    func validToken() async throws -> String {
        if let task { return try await task.value }
        let t = Task { try await api.refresh() }
        task = t
        defer { task = nil }
        return try await t.value
    }
}
```

- RxSwift 6.5 이상은 `Observable.values`(AsyncSequence)와 `Single.value`(async) 등 Concurrency 연동 API를 제공한다. 버딩크 RxSwift 버전을 확인해둘 것.

## 버딩크 포트폴리오 연결

- "Swift Concurrency context 이슈"와 "Xcode 26 beta에서 Concurrency context 버그 우회" 경험을 원인(어떤 격리 규칙 위반이었는지) → 해결 → 지금이라면 어떻게 할지(`@MainActor`, `@concurrent`) 구조로 재정리.
- AI 피드백(URLSession 직접 호출)을 async/await로 짠 부분이 있다면 취소·에러 처리 방식을 설명할 수 있게.

## 예상 질문 & 답변 포인트

- **Q. GCD와 Swift Concurrency의 차이는?**
  → GCD는 스레드/큐 중심, 스레드 폭증·콜백 지옥 위험. Concurrency는 작업 중심, 구조적 취소, 컴파일러가 데이터 레이스 검사.
- **Q. actor를 쓰면 데이터 레이스가 완전히 사라지나요?**
  → 동시 접근 레이스는 막지만, await 지점 재진입 때문에 논리적 경합은 남는다.
- **Q. Task와 Task.detached 차이?**
  → Task는 actor 컨텍스트·우선순위 상속, detached는 상속 없음.
- **Q. 메인 스레드에서 main.sync를 호출하면?**
  → 데드락. 자기 큐가 끝나길 기다리므로.
- **Q. 셀이 재사용될 때 이전 이미지 요청은?**
  → prepareForReuse에서 Task cancel, 결과 반영 전 셀 식별자 확인.

## 검색 키워드

Swift Concurrency async await suspension point · actor reentrancy · Sendable · Swift 6 strict concurrency migration · Swift 6.2 approachable concurrency @concurrent · withCheckedThrowingContinuation · WWDC21 "Meet async/await", "Protect mutable state with Swift actors" · WWDC25 "Embracing Swift concurrency"
