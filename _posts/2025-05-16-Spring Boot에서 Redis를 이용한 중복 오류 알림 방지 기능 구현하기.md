---
layout: post
title: "Spring Boot에서 Redis를 이용한 중복 오류 알림 방지 기능 구현하기"
date: 2025-05-16
categories: [문제 해결]
tags: [Java, Spring Boot, Redis, Slack, 모니터링]
---

## 들어가며

대학교 통합 시스템 개발 프로젝트 진행 중, 오류 모니터링 시스템을 구축하면서 겪은 문제와 해결 과정을 공유합니다. 특히 동일한 오류 메시지가 반복적으로 Slack에 전송되는 문제를 Redis를 활용하여 해결한 경험을 담았습니다.

## 1. 문제 상황

Spring Boot 애플리케이션의 예외 상황을 모니터링하기 위해 Slack webhook을 연동했습니다. 그러나 초기 구현에서는 오류 메시지에 대한 알림 전송 제한이 없어 다음과 같은 문제가 발생했습니다:

- **동일한 오류의 반복 알림**: 같은 `NullPointerException`이 짧은 시간 내에 여러 번 발생하면 Slack에 수십 개의 동일한 알림이 전송
- **중요 알림 식별 어려움**: 반복되는 알림으로 인해 정작 중요한 새로운 오류를 식별하기 어려워짐
- **Slack 채널의 노이즈 증가**: 팀 커뮤니케이션 채널에서 불필요한 알림이 대화를 방해

![중복 알림 폭주 현상](assets/img/unihub_alarm_monitoring_1.png)
*같은 오류가 같은 시간에 반복해서 전송되는 모습*

## 2. 1차 해결 방안: ConcurrentHashMap을 활용한 중복 알림 방지

처음에는 애플리케이션 메모리 내에서 중복 알림을 방지하기 위해 `ConcurrentHashMap`을 사용했습니다:

```java
@Component
@RequiredArgsConstructor
public class AlertNotifier {

    private final RestTemplate restTemplate = new RestTemplate();
    private final ConcurrentHashMap<String, Long> recentExceptions = new ConcurrentHashMap<>();

    public void notifyError(String title, Exception e) {

        String exceptionType = e.getClass().getSimpleName();
        long now = System.currentTimeMillis();

        Long lastSentTime = recentExceptions.get(exceptionType);
        if (lastSentTime != null && now - lastSentTime < duplicateIntervalMs) {
            return;
        }

         // 알림 발송 코드 생략
    }
}
```

### 개선 효과

이 방식으로 중복 알림을 방지한 결과, 오류 알림 메시지가 크게 줄어들었습니다:

**개선 이전 (5월 13-14일)**:
- STG(개발 테스트 서버): 11건
- PROD(운영 테스트 서버): 16건
- **총 27건**

**개선 이후 (5월 15-16일)**:
- STG: 3건
- PROD: 2건
- **총 5건**

**감소율**: (27-5)/27 × 100 ≈ **81.5%**

## 3. 1차 해결책의 한계

`ConcurrentHashMap` 방식은 동작했지만, 다음과 같은 문제가 있었습니다:

1. **메모리 관리 이슈**: 오류 메시지가 계속 쌓이면 `Map`에서 자동으로 제거되지 않아 메모리 누수 가능성
2. **서버 재시작 시 데이터 손실**: 애플리케이션이 재시작되면 중복 알림 방지 상태가 초기화됨

## 4. 최종 해결책: Redis TTL 기능을 활용한 중복 알림 방지

위 문제들을 해결하기 위해 Redis의 키-값 저장소와 TTL(Time To Live) 기능을 활용했습니다:

```java
@Component
@RequiredArgsConstructor
public class AlertNotifier {

    // 일부 코드만 작성

    private final StringRedisTemplate redisTemplate;

    public void notifyError(
            String title, Exception e, String httpMethod, String path, String params
    ) {
        String exceptionType = e.getClass().getSimpleName();
        // 중복 알림 제어: Redis TTL 사용
        String key = "alert:exception:" + exceptionType;
        Boolean absent = redisTemplate.opsForValue()
                .setIfAbsent(key, "1", duplicateIntervalMs, TimeUnit.MILLISECONDS);
        if (!Boolean.TRUE.equals(absent)) {
            return;
        }

        // ...
    }
```

## 5. 알람 탬플릿 개선

초기에는 단순히 오류 메시지만 전송했지만, 효과적인 문제 분석을 위해 Slack의 Block Kit을 활용한 구조화된 알림 템플릿을 구현했습니다:

환경 정보: DEV/STG/PROD 등 어떤 환경에서 발생한 오류인지 표시
스택트레이스 필터링: 프로젝트 패키지로 시작하는 클래스만 표시하여 핵심 오류 원인 집중
요청 정보 포함: HTTP 메서드, 경로, 파라미터 등 오류 발생 컨텍스트 제공
Block Kit 형식: 시각적으로 구분되는 헤더, 섹션, 구분선 등 활용

이러한 템플릿 개선으로 알림의 가독성이 높아져 빠른 원인 파악이 가능해졌습니다.


```java
@Component
@RequiredArgsConstructor
public class AlertNotifier {
    /**
     * 에러 발생 시 Slack에 Block Kit 형식으로 알림 전송
     */
    public void notifyError(
            String title, Exception e, String httpMethod, String path, String params
    ) {
        // 이전 코드 생략

        // 스택 스니펫 5줄
        String stackSnippet = Arrays.stream(e.getStackTrace())
                // 우리의 패키지만
                .filter(f -> f.getClassName().startsWith("com.WEB4_5_GPT_BE.unihub"))
                // 최대 5줄
                .limit(5)
                .map(StackTraceElement::toString)
                .collect(Collectors.joining("\n"));

        // Slack Block Kit JSON
        String payload = """
        {
          "blocks": [
            {
              "type":"header",
              "text":{"type":"plain_text","text":"🚨 [%s] %s (%s)","emoji":true}
            },
            {
              "type":"section",
              "fields":[
                {"type":"mrkdwn","text":"*메시지:*\n%3$s"},
                {"type":"mrkdwn","text":"*엔드포인트:*\n`%4$s %5$s`"},
                {"type":"mrkdwn","text":"*파라미터:*\n%6$s"}
              ]
            },
            {"type":"divider"},
            {
              "type":"section",
              "text":{"type":"mrkdwn","text":"*스택트레이스 (핵심):*\n```%7$s```"}
            }
          ]
        }
        """.formatted(
                        activeProfile.toUpperCase(),   // %1$s
                        title,                        // %2$s
                        escape(e.getMessage()),       // %3$s
                        httpMethod,                   // %4$s
                        path,                         // %5$s
                        escape(params),               // %6$s
                        escape(stackSnippet)         // %7$s
                );

        // 이후 코드 생략
    }

    private String escape(String input) {
        if (input == null) return "";
        return input
                .replace("\\", "\\\\")
                .replace("\"", "\\\"")
                .replace("\n", "\\n")
                .replace("\r", "");
    }
}
```

## 최종 알람 결과 템플릿(DEV -> 로컬에서 보낸 Slack)
![alt text](assets/img/final_unihub_alarm_monitoring.png)

## 5. 최종 개선 효과 및 이점

### 기술적 이점

1. **자동 만료 처리**: Redis의 TTL 기능으로 중복 알림 방지 레코드가 일정 시간 후 자동 삭제됨
2. **메모리 효율성**: 메모리 내 컬렉션 대신 외부 저장소를 사용하여 애플리케이션 메모리 사용 최적화
3. **재시작 내구성**: 애플리케이션이 재시작되어도 중복 알림 방지 상태가 유지됨

### 비즈니스/운영 측면의 이점

1. **중요 알림 가시성 향상**: 중복 알림이 제거되어 실제 중요한 새로운 오류에 집중 가능
2. **팀 커뮤니케이션 개선**: Slack 채널의 불필요한 노이즈 감소
3. **운영 효율성**: 오류 대응 시간 단축 및 팀 피로도 감소

## 결론

이번 경험을 통해 간단한 문제처럼 보이던 중복 오류 알림 이슈를 해결하면서 분산 시스템에서의 상태 관리와 메모리 효율성에 대해 깊이 생각해볼 수 있었습니다. Redis의 TTL 기능은 이런 유형의 문제를 해결하는 데 매우 적합한 도구였으며, 이를 통해 더 효율적이고 견고한 모니터링 시스템을 구축할 수 있었습니다.

실제 개발 환경에서는 작은 최적화가 큰 차이를 만들어낼 수 있다는 것을 다시 한번 깨닫게 된 경험이었습니다.

## 참고 자료

- [Spring Data Redis 공식 문서](https://spring.io/projects/spring-data-redis)