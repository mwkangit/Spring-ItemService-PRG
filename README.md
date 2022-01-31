# Spring-ItemService-PRG


## Description

본 프로젝트는 Spring MVC를 이용하여 상품 등록, 수정, 상세, 목록 로직을 수행하는 프로그램을 구현한 것이다. 모든 뷰는 Thymeleaf 템플릿 엔진을 사용하여 동적 뷰를 만들었으며 새로 고침 오류를 방지하기 위해 PRG를 이용하여 구현하였다. 비즈니스 로직을 간결하게 refactoring 단계를 나태내었으며 오류를 방지하기 위한 로직을 메소드마다 추가하였다.



------



## Environment

![framework](https://img.shields.io/badge/Framework-SpringBoot-green)![framework](https://img.shields.io/badge/Language-java-b07219) 

Framework: `Spring Boot` 2.6.2

Project: `Gradle`

Packaging: `Jar` 

IDE: `Intellij`

Template Engine: `Thymeleaf`

Dependencies: `Spring Web`, `Lombok`, `Thymeleaf`



------



## Installation

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black) 

```
./gradlew build
cd build/lib
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```



![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white) 

```
gradlew build
cd build/lib
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```



------



## Core Feature

```java
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes){
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);

    return "redirect:/basic/items/{itemId}";
}
```

상품 저장하는 로직으로 PRG를 이용하여 redirect 했다. Redirect하지 않고 새로고침 시 마지막 요청인 POST가 요청되어 원하지 않는 데이터가 중복되어 늘어날 수 있다. 이러한 상황을 보완하기 위해 프로젝트의 핵심인 PRG를 이용하여 GET으로 redirect하는 방법을 선택했다.



------



## Demonstration Video

![Spring-ItemService-PRG](https://user-images.githubusercontent.com/79822924/151759045-13bf025b-92af-47f0-a836-b5a78e341970.gif)



------



## More Explanation

[Spring-ItemService-PRG-Note.md](https://github.com/mwkangit/Spring-ItemService-PRG/blob/master/Spring%20-ItemService-PRG-Note.md)
