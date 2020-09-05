# Spring in Action 1장

#### 학습 목표

1. Intellij에서 스프링 Initilizr로 프로젝트 초기 구조 생성
2. 컨트롤러 클래스 작성
3. 뷰 템플릿 정의
4. 테스트 클래스 작성



### 1. Spring MVC

Model, View, Controller를 분리한 디자인 패턴을 일컫는다.

Django와 비교했을 때 `Model = Django Model `, `View = Django Template`, `Controller = Django View` 역할이라고 보면 이해하기 쉽다.



✨MVC 패턴 흐름

1. **Client -> DispatcherServlet** : URL로 접근하여 정보를 요청한다.
2. **Dispatcher Serverlet -> HandlerMapping** : HandlerMapping에 해당 URL을 매핑한 컨트롤러가 있는지 검색을 요청한다.
3. **DispatcherServerlet -> Controler** : 컨트롤러에 처리를 요청한다.
4. **Controller -> (Model And View) -> DispatcherServlet** : 컨트롤러가 요청을 처리한 후 응답을 받을 View의 이름을 반환한다.
5. **DispatcherServlet -> View Resolver** : 응답 받을 View가 존재하는지 검색한다.
6. **View Resolver -> View** : 검색 결과를 View에 전달한다.
7. **View -> DispatcherServlet** : View가 있다면 처리결과를 DispatcherServlet에 전달한다.



### 2. 기본 annotation

##### 1) @Configuration

하나 이상의 `@Bean` 메소드를 스프링 애플리케이션 컨텍스트에 제공하는 클래스임을 스프링에 알려준다. 

##### 2) @Bean

구성 클래스의 메소드에 지정되어 있으며, 각 메소드에서 반환되는 객체가 애플리케이션 컨텍스트의 빈으로 추가되어야 함을 나타낸다.

```java
@Configuration
public class gyull{
    @Bean
    public Eat eat(){
        return new eat();
    }
    @Bean
    public Run run(){
        return new run();
    }
}
```



### 3. 스프링 Initializr

REST API를 사용하는 브라우저 기반의 웹 애플리케이션이다.

Initializr가 생성한 스프링 프로젝트 구조는 다음과 같다.

![image](https://user-images.githubusercontent.com/42667951/90594284-03953980-e225-11ea-8c2f-e47a1a0cf462.png)



### 4. 웹 요청 처리 

#### 1. Controller

파일 위치 : src/main/java/tacos/HomeController

```java
package tacos;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller                      
public class HomeController {

    @GetMapping("/")     //루트 경로인 /의 웹 요청을 처리        
    public String home() {
        return "home";           
    }

}
```

cf) Django

```python
#views.py
def home(request):
    return render(request,"home.html")
```

```python
#urls.py
urlpatterns=[
    path('/',views.home,name="home"),
]
```



#### 2. View

파일위치 : src/main/resources/templates/home.html

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">

<head>
    <meta charset="EUC-KR">
    <title>Taco Cloud</title>
</head>
<body>
<h1>Welcome to...</h1>
<img th:src="@{/images/TacoCloud.png}"/>
</body>
</html>
```

cf) Django의 Template



#### 3. Test

파일위치 : src/test/java/tacos/HomeControllerTest.java

```java
package tacos;

import static org.hamcrest.Matchers.containsString;
...
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

@WebMvcTest(HomeController.class)    // HomeController의 테스트
public class HomeControllerTest {

    @Autowired
    private MockMvc mockMvc;    // 모의 테스팅을 위함

    @Test
    public void testHomePage() throws Exception {
        mockMvc.perform(get("/"))      

                .andExpect(status().isOk())     // HTTP 200

                .andExpect(view().name("home"))  

                .andExpect(content().string(      
                        containsString("Welcome to...")));
    }
}
```



#### 4. Intellij Services를 이용하여 빌드 및 실행하기

다음 순서로 시행하면 Intellij Services 구성을 볼 수 있다.

1. Edit Configurations

![image](https://user-images.githubusercontent.com/42667951/90595238-4a842e80-e227-11ea-8c2d-60dbe8414296.png)



2. 

![image](https://user-images.githubusercontent.com/42667951/90595374-b6ff2d80-e227-11ea-90d7-c4df64fccf39.png)



3.

![image](https://user-images.githubusercontent.com/42667951/90595451-de55fa80-e227-11ea-95fd-7fe722664a15.png)



4. 

![image](https://user-images.githubusercontent.com/42667951/90595516-ff1e5000-e227-11ea-8892-198213a8b5ec.png)



5. http://locolhost:8080 접속

![image](https://user-images.githubusercontent.com/42667951/90608281-57138180-e23d-11ea-8a94-1db584a107d9.png)

Django를 주로 다뤄본 입장에서는 흐름이 어느정도 비슷해서 더 편안하게 다가오는 것 같다. Unit Test를 작성할 수 있는 것이 독특한 장점처럼 느껴진다.  

### 출처

- Spring in Action 제 5판 , 크레이그 월즈 지음, 심재철 옮김 , JPub
