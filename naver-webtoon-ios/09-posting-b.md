# 9. 공고 B 보강: ePub · 다국어 · Objective-C · Storyboard · 패턴 · FP

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

통합 공고는 라인웹툰(글로벌)·시리즈(웹소설)까지 담당 범위가 넓다. ePub뷰어, 다국어, Objective-C, Storyboard, Functional Programming은 버딩크에서 직접 다루지 않은 키워드라 이론과 미니 실습으로 메워야 한다. 서류·면접에서 "이 항목 경험 있나요?"에 "원리는 이렇게 이해하고 있고, 이렇게 적용하겠다"로 답할 수 있는 수준이 목표.

## 9-1. ePub 뷰어 (웹소설 · 시리즈)

- **파일 구조**: 사실상 ZIP 컨테이너. `META-INF/container.xml`이 패키지 문서(OPF) 위치를 가리키고, OPF 안에 metadata(제목·저자), manifest(모든 리소스 목록), spine(읽는 순서)이 있다. 본문은 XHTML + CSS, 목차는 nav 문서(ePub 3) 또는 NCX(ePub 2).
- **Reflowable vs Fixed Layout**: 소설은 글자 크기에 따라 재배치되는 reflowable, 만화·그림책은 고정 레이아웃.
- **iOS 렌더링 방식**: 보통 WKWebView에 XHTML을 로드하고 CSS(폰트·줄간격·여백·다크 모드)를 주입. 페이지 넘김 모드는 CSS multi-column으로 화면 너비 단위 페이지를 만들고 스크롤 오프셋으로 이동.
- **읽은 위치 저장**: 단순 퍼센트는 글자 크기 변경 시 틀어짐 → 챕터(spine index) + 문서 내 위치로 저장. 표준은 EPUB CFI(Canonical Fragment Identifier).
- **Native ↔ Web 통신**: `WKScriptMessageHandler`(JS → 네이티브), `evaluateJavaScript`(네이티브 → JS)로 페이지 정보·하이라이트 처리.
- **오픈소스 참고**: Readium Swift Toolkit (ePub 파싱·렌더링 구조 학습용).
- **성능 포인트**: 챕터 단위 지연 로딩, 다음 챕터 프리로드, WebView 재사용, 폰트 사전 로딩.

## 9-2. 다국어 (라인웹툰 글로벌)

- **String Catalog**(`.xcstrings`, Xcode 15+): 문자열·복수형·기기별 변형을 한 파일에서 관리. 코드에서는 `String(localized:)` 또는 기존 `NSLocalizedString`.
- **복수형**: 언어마다 규칙이 다름(영어 one/other, 일부 언어는 few/many 등) → 문자열을 코드에서 직접 조합하지 말고 카탈로그의 plural 변형 사용.
- **Formatter**: 날짜·숫자·통화는 Locale 기반 포맷(`Date.FormatStyle`, `NumberFormatter`)으로. 하드코딩 금지.
- **레이아웃**: 번역문은 길이가 크게 달라짐 → Auto Layout에서 고정 너비 지양, leading/trailing 사용(RTL 언어 자동 반전), 줄 수 0·폰트 축소 정책.
- **폰트**: 태국어·인도네시아어 등 문자 체계별 행간·폰트 fallback 확인.
- **서버 연동**: `Accept-Language` 헤더, 지역별 콘텐츠·가격(StoreKit `Product`의 `displayPrice`는 현지 통화로 자동 표기).
- **테스트**: Scheme의 App Language / Region 변경, Pseudolanguage(Double-Length, Right-to-Left)로 레이아웃 깨짐 확인.

## 9-3. Objective-C (레거시 코드 대응)

- **왜 필요한가**: 오래된 대형 앱에는 Obj-C 코드가 남아 있고, Swift와 혼용된다. "읽고 수정하고 Swift로 옮길 수 있는가"가 포인트.
- **기본 문법**: `.h`(인터페이스) / `.m`(구현), 메시지 전송 `[obj method:arg]`, `@property (nonatomic, strong/weak/copy)`, nil에 메시지를 보내면 크래시 없이 nil/0 반환.
- **메모리**: Obj-C도 ARC 사용. 블록에서 순환 참조 방지는 `__weak typeof(self) weakSelf = self;`.
- **Swift 혼용**: Swift → Obj-C 사용은 Bridging Header, Obj-C → Swift 사용은 자동 생성되는 `프로젝트명-Swift.h` 임포트 + Swift 쪽 `@objc` 노출.
- **Nullability**: `nonnull` / `nullable` 주석이 Swift에서 옵셔널 여부를 결정. 없으면 암시적 언래핑 옵셔널(`!`)로 들어와 크래시 위험. `NS_ASSUME_NONNULL_BEGIN/END`로 일괄 지정.
- **Category vs Extension**: Obj-C Category는 Swift extension과 유사. Method Swizzling·KVO는 Obj-C 런타임(메시지 디스패치) 기반 → [1장 디스패치](01-swift-deep-dive.md)와 연결.

```objc
// Obj-C 블록 순환 참조 방지
__weak typeof(self) weakSelf = self;
[self.loader loadWithCompletion:^(UIImage *image) {
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (!strongSelf) { return; }
    strongSelf.imageView.image = image;
}];
```

## 9-4. Storyboard · XIB

- **구성**: Storyboard(여러 화면+전환), XIB(단일 뷰·셀). `@IBOutlet`은 보통 weak(뷰 계층이 소유), `@IBAction`으로 이벤트 연결.
- **Segue**: 화면 전환 정의, 데이터 전달은 `prepare(for:sender:)`.
- **DI 문제와 해결**: Storyboard VC는 init을 직접 못 불러 생성자 주입이 어렵다 → iOS 13+ `instantiateViewController(identifier:creator:)`로 의존성 주입 가능.
- **장단점 비교 (면접 단골)**: 시각적 확인·빠른 프로토타입 vs XML 병합 충돌, 런타임에야 발견되는 연결 오류, 재사용성 한계. 버딩크는 SnapKit 코드 기반 → "팀 협업·충돌 관리를 위해 코드 기반을 택했고, Storyboard 프로젝트도 읽고 유지보수할 수 있다"로 답변.
- **IBDesignable / IBInspectable**: 커스텀 뷰를 Interface Builder에서 미리보기·속성 편집.

## 9-5. 디자인 패턴

| 패턴 | iOS 예 | 버딩크 연결 |
|---|---|---|
| Delegate | UITableViewDelegate, 1:1 이벤트 위임 (weak delegate) | 셀 → VC 이벤트 전달 |
| Observer | NotificationCenter, KVO, Rx/Combine | AVPlayerLayer KVO, Rx 바인딩 |
| Singleton | URLSession.shared, UserDefaults.standard | 남용 시 테스트 어려움 → DI로 대체 |
| Factory / DI Container | 객체 생성 책임 분리 | DIContainer |
| Coordinator | 화면 전환 책임 분리 | (도입 검토 사례로 설명) |
| MVC / MVVM | Apple 기본 MVC의 Massive VC 문제 → MVVM | MVVM Input/Output |

## 9-6. Functional Programming

- **핵심 개념**: 순수 함수(같은 입력 → 같은 출력, 부수 효과 없음), 불변성(`let`, 값 타입), 고차 함수(함수를 인자/반환값으로), 함수 합성.
- **Swift 고차 함수**: `map`, `filter`, `reduce`, `compactMap`(nil 제거), `flatMap`(평탄화). 지연 평가가 필요하면 `.lazy`.
- **Optional/Result의 map·flatMap**: 값이 있을 때만 변환을 이어감 → if-let 중첩 감소.
- **FRP 연결**: RxSwift는 시간에 따라 흐르는 값에 FP 연산자를 적용하는 함수형 반응형 프로그래밍. ViewModel의 transform을 순수 함수에 가깝게 만들면 테스트가 쉬워진다.
- **주의**: 지나친 체이닝은 가독성·디버깅을 해침 → 중간 결과에 이름을 붙여 분리.

```swift
// 명령형 → 함수형
let completedMinutes = logs
    .filter { $0.isCompleted }
    .compactMap { $0.durationMinutes }
    .reduce(0, +)
```

## 버딩크 포트폴리오 연결

- **ePub**: 강의 콘텐츠 뷰어(영상·이미지) 경험을 "뷰어 공통 구조(로딩·위치 저장·캐시)"로 일반화해서 설명.
- **다국어**: 버딩크에 String Catalog로 영어 1개만 추가해보는 미니 실습을 휴가 때 진행하면 "다국어 경험"으로 말할 수 있다.
- **Obj-C·Storyboard**: 작은 데모(Obj-C 클래스 1개 + Bridging Header, Storyboard 화면 1개)를 별도 레포로 만들어 두면 자격요건 대응 근거가 된다.
- **FP**: RxSwift 연산자 체인과 고차 함수 사용 코드를 예시로 지정.

## 예상 질문 & 답변 포인트

- **Q. ePub 뷰어를 만든다면 구조를 어떻게 잡겠어요?**
  → OPF 파싱(manifest·spine) → 챕터별 WKWebView 렌더 + CSS 주입 → CFI 기반 위치 저장 → 다음 챕터 프리로드.
- **Q. 다국어 대응 시 주의점?**
  → 문자열 조합 금지(복수형), Locale 포맷터, leading/trailing·가변 너비, Pseudolanguage 테스트.
- **Q. Obj-C 코드를 Swift에서 안전하게 쓰려면?**
  → nullability 주석으로 옵셔널 명확화, Bridging Header, 점진적 Swift 전환.
- **Q. Storyboard와 코드 UI 중 무엇을 선호하나요?**
  → 협업 규모에선 코드 기반 선호(충돌·재사용), 다만 Storyboard 프로젝트 유지보수 가능. creator 기반 DI 언급.
- **Q. Functional Programming을 어떻게 적용했나요?**
  → 고차 함수로 데이터 가공, Rx 연산자 체인, ViewModel transform을 순수 함수에 가깝게.

## 검색 키워드

EPUB OPF manifest spine · EPUB CFI · WKWebView CSS columns pagination · Readium Swift Toolkit · Xcode String Catalog xcstrings plural · iOS localization pseudolanguage · Objective-C bridging header nullability · `__weak typeof(self)` · instantiateViewController creator · iOS design patterns delegate coordinator · Swift functional programming map flatMap compactMap · WWDC23 Discover String Catalogs
