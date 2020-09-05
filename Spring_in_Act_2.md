





# Spring in Action 2장

#### 학습 목표

1. 컨트롤러 클래스, 뷰 템플릿 발전
2. 폼 제출 처리
3. 유효성 검사
4. 뷰 컨트롤러



### 1. 시작하기 앞서... 기본 설정

IntelliJ 와 Gradle을 사용하면서 몇가지 기본 설정해야 할 사항이 있는 것을 확인할 수 있었다. 이에 대한 내용을 먼저 짚은 후 본 내용으로 들어가겠다.

#### 1) IntelliJ lombok 설정

1. **lombok dependency 설정**

2. **lombok plugin 설정**

   - Windows : `Ctrl + Alt + S`  | MacOS : `Cmd + ,`

   - 왼쪽 화면에서 Plugins 선택 후 오른쪽 화면에서 lombok 검색.

   - Lombok plugin Install

   - 설치 후 IntelliJ 재시작

3. **Enable annotion 설정** : Windows : `Ctrl + Alt + S`  | MacOS : `Cmd + ,`

   - 왼쪽화면에서 Build,Execution,Deployment > Compiler > Annotation Processing 선택

   - 오른쪽 화면에서 Enable annotation processing 체크

   

#### 2) cannot resolve method 에러

lombok 사용 시 import가 되지 않아 컴파일 에러가 발생하는 경우가 생긴다. 이럴 경우 뜨는 에러가  *cannot resolve method* 에러이다.

나의 경우 해결 방식은 다음과 같았다.

1. **Enable annotation 설정** ( 위 1번 참조)
2. **Project 경로에 있는 .idea 폴더 제거** (프로젝트 별 설정 파일 포함되어 있는 폴더)



#### 3) import javax.validation 에러 

spring 2.3 이상 버전에서 import javax.validation 을 사용하려하면 에러가 생긴다. 

spring 2.3 미만 버전에서는 아래와 같이 Web-starter가 Validation-starter을 가져왔으나 

```
spring boot -> web -> validation
```

spring 2.3이상 버전에서는 더이상 가져오지 않는다. 그렇기에 초기 프로젝트를 생성할 때 validation 의존성을 설정하거나 pom.xml에 의존성을 추가해주어야 한다.

```xml
<!-- pom.xml -->
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-validation</artifactId></dependency>

```

의존성 추가 후 intelliJ를 재시작하면 의존성이 잘 추가 되었음을 확인할 수 있다.



#### 4) 기본 터미널 gitbash 설정

Windows운영체제에서 IntelliJ는 기본 터미널을 cmd로 가져간다. 사용이 너무 불편하여 기본 터미널을 gitbash로 설정하였다.

1. Windows : `Ctrl + Alt + S`  | MacOS : `Cmd + ,`
2. 왼쪽 검색 필드에 Terminal 검색
3. Shell path를 `cmd.exe`에서 Git의 sh.exe가 설치된 경로인 `"C:\Program Files\Git\bin\sh.exe" -login -i`로 변경

IntelliJ를 재시작하면 터미널이 git bash로 잘 변경되었음을 확인할 수 있다.



### 2. 전체 흐름

![image](https://user-images.githubusercontent.com/42667951/91008397-47b77e00-e619-11ea-9145-7a794951d99a.png)



### 3. 도메인 객체

```java
//src/main/java/tacos/Taco
package tacos;
import java.util.List;

//유효성 검사 규칙 선언 : name의 null과 최소한 하나 이상의 식자재를 선택했는지 확인
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

import lombok.Data;

@Data
public class Taco {

    @NotNull
    @Size(min=5, message="Name must be at least 5 characters long")
    private String name;

    @Size(min=1, message="You must choose at least 1 ingredient")
    private List<String> ingredients;
}

```

`@Data` 는 Lombok anotation으로 런타임 시 Lombok에 final 속성의 생성자를 생성하도록 명령하며 getter와 setter의 생성도 명령한다.

이를 위해서 1-1의 IntelliJ lombok설정이 필요하다.



### 4. 컨트롤러 : RequestMapping, GetMapping

```java
package tacos.web;

...
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
//폼 제출
import org.springframework.web.bind.annotation.PostMapping;
//유효성 검사
...
import tacos.Taco;
import tacos.Ingredient;
import tacos.Ingredient.Type;

@Slf4j//로그를 위함
@Controller //해당 클래스가 컨트롤러로 식별되게 하며, 컴포턴트 검색을 해야 함.
//1. Spring이 DesignTacoController 찾은 후
//2. 스프링 애플리케이션 컨텍스트의 bean으로 이 클래스의 인스턴스 자동 생성
@RequestMapping("/design") // /design으로 시작하는 경로의 요청 처리

public class DesignTacoController {

    @GetMapping// /design의 HTTP Get 요청이 수신될 때 showDesignForm()메서드가 호출됌
    public String showDesignForm(Model model) {
        List<Ingredient> ingredients = Arrays.asList(
                ...
                new Ingredient("SLSA", "Salsa", Type.SAUCE),
                new Ingredient("SRCR", "Sour Cream", Type.SAUCE)
        );
	           ...
        model.addAttribute("taco", new Taco());

        return "design"; 
                ...
    }
    ...
}
```

여기서 봐야할 것은 컨트롤러와 뷰와 url의 연결순서이다.

`@RequestMapping("/design")`은 /design으로 시작하는 경로의 요청을 받으며 `@GetMapping`은 /design의 경로에서 HTTP Get 요청이 수신될 때 하위의 메소드를 호출하는 역할을 한다.

다음 코드를 보면 `@GetMapping`이 showDesignForm 메소드를 통해 `Ingredient`의 요소들을 생성하고 이를 모델 객체의 속성으로 추가한다.

이 후 "design" 이름의 뷰, 즉 `design.html`을 반환하게 된다. 



### 5. 폼 제출 처리 : PostMapping

design.html의 Form태그는 다음과 같이 action이 지정되어 있지 않다.

```html
 <form method="POST" th:object="${taco}">
```

cf) Thymeleaf 사용법에 대해서는 다음 블로그에 잘 정리되어 있다. [참조](https://cyberx.tistory.com/132)

이 경우 서버가 form의 데이터를 모아  Get요청과 같은 경로로 서버에 HTTP POST요청을 전송한다. 

```java
 @PostMapping
    public String processDesign(@Valid Taco design, Errors errors){
        if (errors.hasErrors()){
            return "design";
        }
        log.info("Processing design: " + design);
        return "redirect:/orders/current";
   }

```

위 4번 컨트롤러에 추가하는 코드이다.HTTP POST요청을 받으면 `PostMapping`을 이용하여 POST 요청을 처리한다.  

처리를 한 후에는 /orders/current주소를 반환하게 된다.



### 6. 유효성 검사

스프링은 Bean Validation API를 지원한다. 해당 API를 이용해서 유효성 검사를 쉽게 코드에 추가할 수 있다.

```java
//taco 도메인 클래스
...
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

@Data
public class Taco{
    @NotNull
    @Size(min=3, message="Name must be at least 3 ~")
    private String name;
    ...
}
```

```java
//Order 클래스
...
import javax.validation.constraints.Digits;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.NotBlank;
import org.hibernate.validator. constraints.CreditCardNumber;

@Data
public class Order {

    @NotBlank(message="Name is required")
    private String deliveryName;
    
    @CreditCardNumber(message="Not a valid credit card number")
    private String ccNumber;

    @Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$", message="Must be formatted MM/YY")
    private String ccExpiration;

    @Digits(integer=3, fraction=0, message="Invalid CVV")
    private String ccCVV;
}
```



### 7. 뷰 컨트롤러

뷰에 요청을 전달하며 모델 데이터나 사용자 입력을 처리하지 않는 컨트롤러는 뷰 컨트롤러로 선언한다.

```java
//main/java/tacos/web/WebConfig
package tacos.web;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer{

    @Override
    public void addViewControllers(ViewControllerRegistry registry){
        registry.addViewController("/").setViewName("home");
    }
}
```

뷰 컨트롤러가 '/' 경로로 addViewController을 호출하면 이는 ViewControllerRegistration 객체의 setViewName()을 호출한다.  해당 경로의 요청을 전달하는 뷰로는 "home" , 즉 `home.html`을 지정한다.



### 8. 출력 결과

- **http://127.0.0.8080/design**

![image](https://user-images.githubusercontent.com/42667951/91011041-8d2a7a00-e61e-11ea-8561-eb78f064a01c.png)



- **design log**

![image](https://user-images.githubusercontent.com/42667951/91011931-2efe9680-e620-11ea-9e96-2b6d8ffe23af.png)



- **http://127.0.0.8080/orders/current**

![image](https://user-images.githubusercontent.com/42667951/91011151-bea34580-e61e-11ea-900c-49b354a8ed78.png)



- **order validation**

![image](https://user-images.githubusercontent.com/42667951/91011852-0a0a2380-e620-11ea-8c42-76db33c2c840.png)



- **order log**

![image](https://user-images.githubusercontent.com/42667951/91011889-1d1cf380-e620-11ea-8847-f406c83c90a2.png)



- **home**

![image](https://user-images.githubusercontent.com/42667951/91011976-3de54900-e620-11ea-9fb9-46fe67beaf00.png)



### 출처

- Spring in Action 제 5판 , 크레이그 월즈 지음, 심재철 옮김 , JPub
- [Thymeleaf 사용법](https://cyberx.tistory.com/132)

