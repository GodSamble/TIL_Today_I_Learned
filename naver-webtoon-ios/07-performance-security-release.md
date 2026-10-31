# 7. 성능 측정 · 보안 · 배포

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

공고의 "성능 이슈를 진단"은 도구를 써봤는지를 묻는 것이다. 감이 아니라 Instruments 수치로 말할 수 있어야 한다. 보안은 버딩크의 API 키 관리 방식과 직접 연결된다.

## 7-1. 측정 도구

| 도구 | 용도 |
|---|---|
| Time Profiler | CPU 시간을 많이 쓰는 함수 찾기(메인 스레드 블로킹) |
| Allocations | 메모리 할당 추적, Generations로 화면 진입/이탈 반복 시 누적 확인 |
| Leaks / Memory Graph Debugger | 순환 참조·누수 객체 시각화 (Xcode 디버그 바에서 실행) |
| Animation Hitches | 스크롤 끊김 원인 프레임 분석 |
| MetricKit · Xcode Organizer | 실사용자 기기의 런치 시간, 행(hang), 크래시, 배터리 지표 수집 |

## 7-2. 앱 런치 최적화

- 런치 = 프로세스 생성·dyld 로딩(pre-main) + `didFinishLaunching`~첫 화면 렌더.
- 개선: 시작 시 하던 SDK 초기화·DB 마이그레이션을 지연/백그라운드로, 첫 화면에 필요한 것만 로드, 동적 프레임워크 수 관리.

## 7-3. 보안

- **토큰 저장**: UserDefaults(평문 plist)가 아니라 Keychain. 접근성 옵션(`kSecAttrAccessibleAfterFirstUnlock` 등)으로 보호 수준 지정.
- **API 키**: xcconfig → Info.plist 방식은 Git 노출은 막지만 앱 바이너리에는 포함되어 추출 가능. 중요한 키(예: OpenAI)는 서버 프록시로 호출하는 것이 정석.
- **ATS**: 기본적으로 HTTPS 강제. **SSL Pinning**: 신뢰할 인증서/공개키를 앱에 고정해 중간자 공격 방지(갱신 운영 비용 존재).

## 7-4. 크래시 분석

- dSYM으로 크래시 주소를 함수·라인으로 심볼리케이션. Crashlytics 업로드 설정 확인.
- 흔한 원인: 강제 언래핑, 인덱스 범위 초과, unowned 해제 후 접근, 백그라운드 스레드 UI 접근, 메인 스레드 워치독 종료(장시간 블로킹).

## 7-5. 배포 · CI/CD

- **코드 서명**: 인증서(누가 서명) + App ID + 프로비저닝 프로파일(어떤 기기·권한) + Entitlements(푸시, IAP 등 기능).
- **Fastlane**: match(인증서 공유), gym(빌드), pilot(TestFlight 업로드)로 배포 자동화. **Xcode Cloud**: Apple 제공 CI.

## 버딩크 포트폴리오 연결

- AI 피드백의 OpenAI 키를 xcconfig로 관리한 점 → "Git 노출은 막았지만 바이너리 추출 위험을 인지, 개선안은 서버 프록시"로 답하면 오히려 플러스.
- FCM 이슈(`FirebaseAppDelegateProxyEnabled` 설정) 디버깅 경험 → "설정·SDK 내부 동작까지 추적한 사례"로 정리.
- SPM 한글 경로·Firebase/gRPC C++20 빌드 에러 해결 → 빌드 환경 트러블슈팅 역량.

## 예상 질문 & 답변 포인트

- **Q. 메모리 누수를 어떻게 찾나요?**
  → Memory Graph로 남은 객체 확인, Allocations Generations로 반복 진입 시 누적 확인, 원인은 대부분 클로저 캡처.
- **Q. API 키를 앱에 안전하게 둘 수 있나요?**
  → 클라이언트에 완전히 안전한 보관은 불가. 서버 프록시가 정답.
- **Q. 토큰은 어디에 저장하나요?**
  → Keychain. UserDefaults는 평문.
- **Q. 앱 런치가 느리면?**
  → Instruments App Launch로 측정 후 초기화 지연·경량화.

## 검색 키워드

Instruments Time Profiler Allocations generations · Xcode Memory Graph Debugger · MetricKit · iOS app launch time optimization · iOS Keychain kSecAttrAccessible · API key security mobile proxy · dSYM symbolication · Fastlane match · WWDC22 "Optimize app launch" 관련 세션
