# 1. Swift 언어 심화

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

실무 인터뷰는 Swift·iOS 질문 비중이 매우 높고, 답을 외웠는지가 아니라 실제 코드에 어떻게 적용하는지, 문제의 근본 원인을 아는지를 본다. 공고의 "메모리 관리, 안정성"이 이 챕터에 해당한다.

## 1-1. ARC와 메모리 관리

- **ARC**: 클래스 인스턴스를 참조하는 강한 참조 수(reference count)를 컴파일러가 자동으로 증감시키고, 0이 되면 해제(`deinit`)한다. 값 타입(struct, enum)은 대상이 아니다.
- **strong**: 기본 참조. 카운트를 올린다.
- **weak**: 카운트를 올리지 않는다. 대상이 해제되면 자동으로 nil이 되므로 항상 `var` + 옵셔널.
- **unowned**: 카운트를 올리지 않고 nil도 되지 않는다. 대상이 먼저 해제된 뒤 접근하면 크래시. "참조 대상이 나보다 반드시 오래 산다"가 보장될 때만 사용.
- **순환 참조(retain cycle)**: A↔B가 서로 강하게 잡아 둘 다 해제되지 않는 상태. 가장 흔한 형태는 self가 클로저를 보유하고, 클로저가 self를 캡처하는 경우.
- 캡처 리스트 `[weak self]`는 클로저가 생성되는 시점에 평가된다.

```swift
// 순환 참조 예: viewModel을 VC가 소유, 구독 클로저가 self(VC)를 강하게 캡처
viewModel.title
    .bind { [weak self] title in   // weak로 끊기
        self?.titleLabel.text = title
    }
    .disposed(by: disposeBag)      // VC 해제 → disposeBag 해제 → 구독 dispose
```

## 1-2. 값 타입 vs 참조 타입

- **struct/enum**: 대입·전달 시 복사(값 의미론). 공유 상태가 없어 스레드 안전성 추론이 쉽다.
- **class**: 참조 공유, 상속 가능, identity(`===`) 존재, `deinit` 존재.
- **선택 기준(Apple 권장)**: 기본은 struct. 동일성(identity)이 필요하거나, 공유 가변 상태가 의도이거나, Objective-C 연동이 필요할 때 class.
- **Copy-on-Write**: Array, Dictionary, String 등 표준 라이브러리 컬렉션은 복사 시 버퍼를 공유하다가 쓰기 시점에만 실제 복사한다. 직접 만든 struct에는 자동 적용되지 않으며, 구현하려면 내부 class 버퍼 + `isKnownUniquelyReferenced`를 사용한다.

## 1-3. 프로토콜 · 제네릭 · some / any

- **associatedtype**: 프로토콜 안의 제네릭 자리. (예: Repository 프로토콜의 Entity 타입)
- **some (opaque type)**: "구체 타입 하나가 정해져 있지만 숨김". 컴파일 타임에 타입이 고정되어 정적 디스패치·최적화 가능. SwiftUI의 `some View`.
- **any (existential)**: "이 프로토콜을 따르는 아무 타입". 런타임에 박스(existential container)로 감싸 동적 디스패치. 유연하지만 비용이 있다. Swift 5.6부터 `any` 표기가 도입됨.
- **판단 기준**: 타입이 하나로 정해지면 some/제네릭, 서로 다른 타입을 한 배열에 담아야 하면 any.

## 1-4. 메서드 디스패치

| 방식 | 대상 | 특징 |
|---|---|---|
| Static (Direct) | struct/enum 메서드, `final`, `private`, extension의 메서드 | 가장 빠름, 인라이닝 가능 |
| Table (V-Table) | class의 오버라이드 가능한 메서드 | 클래스별 함수 포인터 테이블 조회 |
| Witness Table | 프로토콜 요구사항 호출 | 타입별 프로토콜 구현 테이블 조회 |
| Message | `@objc dynamic` | Obj-C 런타임, KVO·swizzling 가능, 가장 느림 |

- 상속할 계획이 없는 클래스(대부분의 ViewController, ViewModel)에 `final`을 붙이면 정적 디스패치가 가능해진다.
- KVO가 `@objc dynamic`을 요구하는 이유도 여기서 연결된다.

## 1-5. Codable · Property Wrapper · KeyPath

- **Codable**: 프로퍼티가 모두 Codable이면 컴파일러가 인코딩/디코딩 구현을 합성. 서버 키가 다르면 `CodingKeys`, 누락·타입 불일치 대응은 `init(from:)` 커스텀 + `decodeIfPresent`.
- **Property Wrapper**: `wrappedValue`로 저장 로직을 재사용(`@UserDefault` 같은 래퍼). `projectedValue`는 `$`로 접근.
- **KeyPath**: 프로퍼티 경로를 값으로 다룸. `observe(\.status)` 같은 Swift KVO API에 쓰인다.

## 버딩크 포트폴리오 연결

- RxSwift 바인딩 전반의 `[weak self]` + `disposed(by:)` 패턴을 "왜 필요한가(ARC·순환 참조)"로 설명.
- Domain Entity를 struct로, Repository·Service를 class/protocol로 둔 이유를 값/참조 선택 기준으로 설명.
- 서버 응답 불일치 대응 Codable 커스텀 경험이 있다면 사례로 정리.

## 예상 질문 & 답변 포인트

- **Q. weak와 unowned의 차이와 각각 언제 쓰나요?**
  → 카운트 미증가는 같음. weak는 해제 시 nil(옵셔널), unowned는 해제 후 접근 시 크래시. 생명주기 보장 여부가 기준.
- **Q. struct와 class 중 무엇을 기본으로 쓰나요?**
  → struct 기본. identity·공유 상태·상속·Obj-C 연동 필요 시 class. 값 의미론이 동시성 안전성에도 유리.
- **Q. Array를 복사하면 매번 메모리가 복사되나요?**
  → 아니요, CoW. 쓰기 시점에 참조가 유일하지 않을 때만 복사.
- **Q. some과 any의 차이는?**
  → some은 컴파일 타임 고정 단일 타입(정적), any는 런타임 박싱(동적, 비용 있음).
- **Q. final을 붙이면 무엇이 좋아지나요?**
  → 오버라이드 불가로 정적 디스패치 최적화. 설계 의도도 명확해짐.

## 검색 키워드

Swift ARC retain cycle closure capture list · weak vs unowned · Copy-on-Write isKnownUniquelyReferenced · Swift opaque type vs existential · Swift method dispatch witness table · WWDC "Understanding Swift Performance" · WWDC22 "Embrace Swift generics"
