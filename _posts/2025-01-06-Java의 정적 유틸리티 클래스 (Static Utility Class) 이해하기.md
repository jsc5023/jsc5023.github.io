---
layout: post
title: "Java의 정적 유틸리티 클래스 (Static Utility Class) 이해하기"
date: 2025-01-06
categories: [Java, Programming]
tags: [Java, Static, Utility, File IO]
---

## 들어가며
오늘 File 입출력 관련 강의를 듣다가 흥미로운 점을 발견했습니다. 강사님이 파일 처리를 위한 유틸리티 클래스를 만들 때 static 키워드를 사용하셨는데요. 처음에는 '왜 굳이 static으로 만들까?'라는 의문이 들었습니다. 이 글에서는 제가 이해한 정적 유틸리티 클래스의 개념과 사용 이유에 대해 설명하고자 합니다.

---

## 정적 유틸리티 클래스란?
정적 유틸리티 클래스는 인스턴스 변수 없이 정적 메서드만을 포함하는 클래스입니다. 주로 특정 작업을 수행하는 관련 메서드들을 묶어서 제공할 때 사용됩니다.

---
### 특징
- 모든 메서드가 static으로 선언됨
- 인스턴스 변수가 없음
- 생성자가 private으로 선언됨 (인스턴스화 방지)
- 상태를 유지할 필요가 없는 순수 기능성 메서드들의 모음

---

## 예제 코드로 이해하기

💡 **핵심 포인트**
일반 클래스와 정적 유틸리티 클래스의 가장 큰 차이점은 static 키워드의 사용 여부입니다.

```java
// 일반적인 클래스로 구현한 경우
public class FileHandler {
    public void writeFile(String content, String path) {
        // 파일 쓰기 로직
    }
    
    public String readFile(String path) {
        // 파일 읽기 로직
        return content;
    }
}

// 사용 시
FileHandler handler = new FileHandler();
handler.writeFile("내용", "경로");
```

```java
// 정적 유틸리티 클래스로 구현한 경우
public class FileUtils {

    // 인스턴스화 방지를 위한 private 생성자
    // static 메서드만 사용할 것이므로 객체 생성을 막습니다.
    private FileUtils() {
        throw new AssertionError("이 클래스는 인스턴스화할 수 없습니다!");
    }
    
    // static 과 public을 중점적으로 보면 됩니다.
    public static void writeFile(String content, String path) {
        // 파일 쓰기 로직
    }
    
    public static String readFile(String path) {
        // 파일 읽기 로직
        return content;
    }
}

// 사용 시
FileUtils.writeFile("내용", "경로");
```

## 왜 정적 유틸리티 클래스를 사용할까?

1. **메모리 효율성**
   - 객체를 생성할 필요가 없어 메모리를 절약할 수 있습니다.

2. **편의성**
   - 객체 생성 없이 바로 메서드를 호출할 수 있어 사용이 간편합니다.
   - 코드가 더 깔끔해지고 가독성이 향상됩니다.(위의 예시를 참고하면 빠르게 이해 가능합니다.)

## 자주 사용되는 정적 유틸리티 클래스 예시

``` java
// 파일 관련
File.separator
Files.copy()
Paths.get()

// 테스트 관련
assertThat()
assertEquals()

// 컬렉션 관련
Collections.sort()
Arrays.asList()

// 문자열 관련
String.format()
String.valueOf()

// 숫자 관련
Integer.parseInt()
Double.parseDouble()
```

저는 이 글을 쓰면서, 생각보다 많은곳에 사용한다는 것을 알았습니다.
특히 Collections.sort랑, assertThat, Integer.parseInt 이런게 전부 정적 유틸리티 클래스 였습니다.
저는 이때까지 그냥 사용하다가 정적 클래스를 만드는 도중에 왜 사용할까 만 생각했는데, 이런 자세한 것이 있었구나 라고 생각하게 되었습니다.

---

## ⚠️ 주의사항
1. **과도한 사용 금지**
   - 상태 관리가 필요하거나 객체지향적 설계가 더 적합한 경우에는 일반 클래스를 사용해야 합니다.

예를 들어보면, 

``` java
// 잘못된 예시 - static으로 만들면 안 되는 케이스
public class ShoppingCart {
    private static List<Item> items;  // static으로 장바구니 상태 관리
    
    public static void addItem(Item item) {
        items.add(item);
    }
    
    public static int getTotalPrice() {
        return items.stream()
                   .mapToInt(Item::getPrice)
                   .sum();
    }
}
```
---

이러면 static이 모든 사용자가 동일한 List를 사용하니까, 에러가 나타날 수 있습니다.
이런식으로 실시간으로 변경되는 상태를 저장해야 될 경우에는 일반 클래스를 사용해야 합니다.

## 마치며
처음에는 단순히 "왜 static을 쓸까?"라는 의문에서 시작했지만, 정적 유틸리티 클래스의 개념을 이해하고 나니 그 사용 이유와 장점이 명확해졌습니다. 상태 관리가 필요 없는 순수 기능성 메서드들을 효율적으로 구현하고 사용할 수 있는 좋은 방법이라는 것을 알게 되었습니다.