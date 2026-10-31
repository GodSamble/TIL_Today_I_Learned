# 8. CS 기본기 & Swift 라이브 코딩

[← 목차](00-overview.md)

## 왜 네이버웹툰 면접에서 중요한가

네이버웹툰은 별도 온라인 코테보다 면접 중 라이브 코딩 비중이 크다. 난이도 자체보다 Swift 문법이 손에 붙어 있는지, 풀이 과정을 말로 설명하는지가 관건이다.

## 8-1. Swift 자료구조와 시간 복잡도

| 구조 | 주요 연산 | 메모 |
|---|---|---|
| Array | 인덱스 접근 O(1), append 평균 O(1), 앞 삽입·삭제 O(n) | 큐로 쓸 때 `removeFirst`는 O(n) → 인덱스 포인터 방식 |
| Dictionary / Set | 조회·삽입·삭제 평균 O(1) | 키는 Hashable |
| String | 인덱스 접근이 O(1) 아님(유니코드 문자 단위) | `Array(s)`로 바꿔서 처리 |
| Heap / Deque | 표준 라이브러리 미포함 | 힙은 직접 구현 준비(swift-collections에 존재) |

```swift
// 라이브 코딩용 큐 (removeFirst O(n) 회피)
var queue = [start]; var head = 0
while head < queue.count {
    let cur = queue[head]; head += 1
    for next in graph[cur] where !visited[next] {
        visited[next] = true; queue.append(next)
    }
}
```

## 8-2. 우선 연습 유형

- 구현·시뮬레이션, 해시, 스택/큐, 정렬, BFS/DFS, 이분 탐색, 그리디, 기초 DP
- 프로그래머스 Lv.1~2를 Swift로만 풀고, 풀이를 소리 내어 설명하는 연습
- **라이브 코딩 순서**: 문제 재확인 → 예시로 검증 → 단순 풀이 제시 → 개선 풀이 → 복잡도 언급 → 엣지 케이스

## 8-3. 운영체제

- **프로세스 vs 스레드**: 프로세스는 독립 메모리 공간, 스레드는 프로세스 내 메모리를 공유하는 실행 흐름.
- **컨텍스트 스위칭**: 실행 상태 저장·복원 비용. 스레드 과다 생성이 성능을 떨어뜨리는 이유.
- **뮤텍스 vs 세마포어**: 뮤텍스는 1개 접근 잠금(소유 개념), 세마포어는 N개까지 접근 허용 카운터.
- **데드락 4조건**: 상호 배제, 점유 대기, 비선점, 순환 대기.

## 8-4. 네트워크

- **TCP vs UDP**: 신뢰성·순서 보장 연결 지향 vs 빠르지만 비보장.
- **HTTP 버전**: HTTP/2 멀티플렉싱(한 연결에 여러 요청), HTTP/3는 QUIC(UDP 기반).
- **HTTPS/TLS**: 인증서로 서버 신원 확인 후 대칭키 합의, 이후 암호화 통신.
- **캐싱**: `Cache-Control`, `ETag` / `If-None-Match`(304 Not Modified). iOS의 URLCache가 이를 따른다.
- **REST · 상태 코드**: 2xx 성공, 4xx 클라이언트 오류(401 인증, 403 권한, 404 없음), 5xx 서버 오류.

## 버딩크 포트폴리오 연결

- 재설치 트랩 해결("already registered" HTTP 400을 성공으로 처리) → 상태 코드와 멱등성(idempotency) 관점으로 설명하면 CS와 실무가 연결된다.
- TrainerQR upsert 로직 수정 → 중복 요청에 안전한 설계라는 같은 맥락.

## 예상 질문 & 답변 포인트

- **Q. Swift String 인덱싱이 느린 이유?**
  → 유니코드 확장 문자소 단위라 임의 접근이 O(n). Array로 변환.
- **Q. 프로세스와 스레드 차이?**
  → 메모리 공간 독립 vs 공유. 스레드 간 공유 상태가 동시성 문제의 원인.
- **Q. ETag는 무엇인가요?**
  → 리소스 버전 식별자. 변경 없으면 304로 본문 전송 생략.

## 검색 키워드

Swift coding test tips readLine String Array · Swift heap implementation · programmers Swift · deadlock conditions · HTTP/2 multiplexing HTTP/3 QUIC · TLS handshake · ETag Cache-Control URLCache · idempotency HTTP methods
