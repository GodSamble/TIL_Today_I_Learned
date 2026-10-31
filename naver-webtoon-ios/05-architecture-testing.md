# 5. 아키텍처 · 모듈화 · 테스트

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

공고에 "확장성 있는 아키텍처 설계 및 모듈화, SDK"가 명시돼 있다. 버딩크의 Clean Architecture + DIContainer는 강점이지만, 테스트 코드 경험은 이력서의 가장 큰 빈칸이라 반드시 채워야 한다.

## 5-1. Clean Architecture 레이어

| 레이어 | 구성 | 버딩크 예 |
|---|---|---|
| Presentation | ViewController, ViewModel | WorkoutLogViewController + ViewModel |
| Domain | Entity, UseCase, Repository 프로토콜 | 운동 기록 Entity, 기록 저장 UseCase |
| Data | Repository 구현, API(Moya), 로컬 DB(Realm) | Moya TargetType, Realm 저장소 |

- **의존성 규칙**: 바깥(Presentation, Data) → 안(Domain)으로만 의존. Domain은 UIKit, Moya, Realm을 모른다.
- **장점**: 테스트 용이(프로토콜로 Mock 주입), 라이브러리 교체 영향 최소화. **단점**: 보일러플레이트, 작은 기능엔 과설계. → "왜 이 규모에서 택했는가"를 답할 수 있어야 한다.

## 5-2. MVVM과 DI

- **Input/Output 패턴**: ViewModel이 `transform(input:) -> Output`으로 UI 이벤트를 받아 상태를 내보냄 → 데이터 흐름이 단방향으로 명확.
- 생성자 주입을 기본으로, 의존성은 프로토콜 타입으로 받는다.
- **DIContainer / Composition Root**: 객체 생성과 조립을 한곳(앱 시작점)에 모아 각 객체는 자기 의존성을 "받기만" 한다.
- **Coordinator**: 화면 전환 책임을 VC에서 분리 → VC 재사용성, 딥링크 처리 용이.

## 5-3. 모듈화

- SPM 로컬 패키지로 Feature(화면 묶음), Domain, Network, DesignSystem 등을 분리.
- 효과: 빌드 시간 단축(변경 모듈만 재빌드), 팀별 소유권, 접근 제어(`public` / `internal`)로 의존 강제.
- **Interface 모듈 패턴**: Feature끼리 직접 의존하지 않고 인터페이스 모듈에만 의존 → 순환 의존 방지.
- **Tuist**: Swift 매니페스트로 Xcode 프로젝트를 생성·관리. 대규모 모듈 프로젝트의 pbxproj 충돌 감소.
- **SDK 개발 관점**: 공개 API 최소화, 하위 호환, 문서화, 버전 정책(semantic versioning).

## 5-4. 테스트

- **XCTest**: `XCTestCase`, `XCTAssert…`, 비동기는 async 테스트 메서드나 expectation.
- **Swift Testing**(Xcode 16+): `@Test`, `#expect`, `#require`, `@Suite`, 파라미터화 테스트. 신규 테스트는 이쪽으로 작성하는 추세.
- **ViewModel 테스트**: Repository 프로토콜의 Mock을 주입 → Input 이벤트 → Output 검증. Rx는 RxTest의 `TestScheduler` / RxBlocking 사용.
- **테스트 피라미드**: 단위 테스트 다수 → 통합 → UI 테스트 소수. 스냅샷 테스트(swift-snapshot-testing)로 UI 회귀 방지.

```swift
import Testing

struct WorkoutLogViewModelTests {
    @Test func 완료_시_통계가_갱신된다() async throws {
        let repo = MockWorkoutRepository(stub: [.sample])
        let vm = WorkoutLogViewModel(repository: repo)

        await vm.complete(id: 1)

        #expect(vm.completedCount == 1)
        #expect(repo.savedIDs == [1])
    }
}
```

## 버딩크 포트폴리오 연결

- DIContainer 구조도를 깃허브 문서와 일치시켜 면접에서 그림으로 설명할 수 있게.
- 복무 후·휴가 중 가능하면 핵심 ViewModel 2~3개에 단위 테스트를 추가해 레포에 반영 → "테스트 경험" 빈칸을 직접 채움.
- 멀티 직군 팀(iOS 3명 포함) 협업 경험을 모듈·컨벤션 관점에서 정리.

## 예상 질문 & 답변 포인트

- **Q. Clean Architecture를 이 규모에 적용한 이유와 단점?**
  → 테스트·교체 용이성 vs 보일러플레이트. 팀 협업과 기능 확장을 고려한 선택.
- **Q. DI를 왜 쓰나요?**
  → 결합도 감소, Mock 주입 테스트, 객체 생성 책임 분리.
- **Q. 모듈화의 장점과 비용?**
  → 빌드 시간·소유권·접근 제어 vs 초기 설정 비용, 모듈 경계 설계 난이도.
- **Q. ViewModel을 어떻게 테스트하나요?**
  → 프로토콜 Mock 주입, 입력 → 출력 검증, 비동기는 async 테스트.

## 검색 키워드

iOS Clean Architecture MVVM · MVVM input output transform · DIContainer composition root · Coordinator pattern iOS · SPM modularization interface module · Tuist · Swift Testing @Test #expect · WWDC24 "Meet Swift Testing" · RxTest TestScheduler
