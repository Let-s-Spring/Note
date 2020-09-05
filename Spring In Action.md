# Spring In Action

## Chap 4(Security)



스프링보안을 위해서는 스프링 시큐리티를 먼저 활성화 해야한다. 따라서 `pom.xml` 파일에서 <dependency>에 아래의 코드를 추가하자

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
```

처음 추가한 파일은 스프링 보트 보안 스타터 의존성이고 두번째는 보안테스트 의존성이다.



> 궁금한점 
>
> > 스프링의 classpath는 어디를 지칭하는 걸까?
> >
> > * JVM이 프로그램을 실행할때 Class파일을 찾는데 기준이 되는 경로 
> > * Classpath를 지정하지 않으면,  JVM이 위치한 디렉토리에서만 클래스들을 찾게됨
> > * Does `classpath:` search for resource relative to the document in which it is specified(in case of web applications)?
> >   * `classpath:` is always relative to the classpath root. If you put a `/` at the start of the path, it is silently removed.
> > * Is it more fast to search if i give direct location of resource? instead e.g. `classpath:/WEB-INF/classes/myfolder/myfile.txt`
> >   * No, that won't work at all. The classpath root contains `/WEB-INF/classes`, so the path should be relative to that.





하고나면

![image-20200905123911111](/Users/nayeong-yun/Library/Application Support/typora-user-images/image-20200905123911111.png)

위와 같은 로그인 창이 나오고 

__![Screen Shot 2020-09-05 at 12.40.56 PM](/Users/nayeong-yun/Library/Application Support/typora-user-images/Screen Shot 2020-09-05 at 12.40.56 PM.png)__

이 나왔다. 이렇게 하면 그냥 보안 기능을 끝낸거 같다. 우선 기능만 알아보자면 

* 모든 HTTP 요청 경로는 인증되어야 한다.
* 로그인을 하였다고 해도 특정 역할이나 권한이 부여되지 않는다.
* 로그인 페이지가 따로 없다.
* 스프링 시큐리티의 HTTP 기본인증(위 그림)을 사용하여 인증한다.
* 사용자는 하나만 있고 이름은 user이다. 비밀번호는 암호화 해 준다.



이런방법은 실제 우리가 앱을 만들때 딱히 도움은 되지 않는다. 따라서 스프링 자동-구성이 하는 것을 대체 하기위한 작업을 해야 한다. 

한명이상의 사용자를 가질 수 있도록 타코 클라우드에 적합한 사용자 스토어를 구성해보자.



> 기본 구성 클래스 설정
>
> 
>
> ```java
> @Configuration
> @EnableWebSecurity
> public class SecurityConfig extends WebSecurityConfigurerAdapter {
>     @Override
>     protected void configure(HttpSecurity http) throws Exception//http 보안을 구성하는 메소드
>     {
>         http.authorizeRequests().antMatchers("/design","orders")
>                 .access("hasRole('ROLE_USER')")
>                 .antMatchers("/","/**").access("permitAll")
>                 .and()
>                 .httpBasic();
>     }
> 
>     @Override
>     public void configure(AuthenticationManagerBuilder auth) throws Exception//사용자 인증을 구성하는 메소드
>     {
>         auth.inMemoryAuthentication()
>                 .withUser("user1")
>                 .password("{noop}password1")
>                 .authorities("ROLE_USER")
>                 .and()
>                 .withUser("user2")
>                 .password("{noop}password2")
>                 .authorities("ROLE_USER");
>     }
> }
> ```

위 클래스로 인해 우리는 __사용자의 HTTP 요청 경로에 대해 접근 제한과 같은 보안 관련 처리를 우리가 원하는 대로 할 수 있게 해준다__.



또한 우리는 __사용자스토어__ 라는 것을 구성해야한다. 사용자스토어란 __한 명 이상의 사용자를 처리할 수 있도록 사용자 정보를 유지,관리__하는 것이다.



### 스프링에서 제공하는 사용자 스토어

* 인메모리 사용자 스토어
* JDBC 기반 사용자 스토어
* LDAP 기반 사용자 스토어
* 커스텀 사용자 명세 서비스



어떤 사용자 스토어를 구성하든 관계없이 __configure(AuthenticationManagerBuilder auth)__ 이 메소드를 구성해야 한다.



차례대로 살펴보도록 하자



## 인메모리 사용자 스토어

> 만일 변경이 필요 없는 사용자만 미리 정해 놓고 앱을 사용한다면 아예 구성 코드 내부에 정의할 수 있다. 





```java
@Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception//사용자 인증을 구성하는 메소드
    {
        auth.inMemoryAuthentication()
                .withUser("user1")
                .password("{noop}password1")
                .authorities("ROLE_USER")
                .and()
                .withUser("user2")
                .password("{noop}password2")
                .authorities("ROLE_USER");
    }
```

* AuthenticationManagerBuilder는 인증 명세(로그인 창)를 구성하기 위해 빌더 형태의 API를 사용한다.
* inMemoryAuthentication는 보안 구성 자체에 __사용자 정보를 직접 구성__ 할 수 있게 해준다.
* withUser를 호출하면 해당 사용자의 구성이 시작되고 매개인자로 사용자 이름을 전달한다.
* 비밀번호는 password, 부여권한은 authorities로 설정된다. ("ROLE_USER" 말고 "USER"라고 설정해도 된다.)





스프링5 부터는 반드시 비밀번호를 암호화해야한다. 만약 password()메소드를 호출하여 암호화하지 않은면 접근거부(HTTP 403, HTTP 500)가 발생한다. 그러나 인메모리 사용자 스토어의 간단한 테스트를 위해 {noop}을 지정하여 비밀번호를 암호화하지 않았다.



##### 사용되는 용도

* 테스트 목적
* 간단한 앱

##### 단점

* 사용자 정보의 추가나 변경 어려움(변경을 원하면 보안 구성코드(위의 코드)를 변경한 후 앱에서 다시 빌드하고 배포,설치해야함)



타코앱의 목적

=> 고객 스스로 사용자 등록하고 자신의 정보를 변경할 수 있어야 함



### JDBC 기반의 사용자 스토어

```java
 @Autowired
    DataSource dataSource;//자동 주입됨
    
    
    

public void configure(AuthenticationManagerBuilder auth) throws Exception
    {
  			auth.jdbcAuthentication().dataSource(dataSource);
  			//데이터베이스를 액세스 하는 방법을 알 수 있도록 dataSource()메소드를 호출하여 DataSource도 설정해야 한다. 
    }
    
```

> 



스프링 시큐리티의 사용자 정보 데이터베이스 스키마를 사용할 때는 방금전에 작성한 configure() 메소드의 코드면 충분 

=> 사용자 정보를 저장하는 테이블과 열이 정해져 있고 쿼리가 미리 생성되어 있어서





스프링 시큐리티에 사전 지정된 데이터베이스 테이블과 SQL 쿼리를 사용하려면 관련 테이블을 생성하고 사용자 데이터를 추가해야한다. 

즉, scheme.sql 파일과 데이터를 추가하는 data.sql 파일을 작성한 후 프로젝트의 아래(src/main/resources)에 두면 된다.



데이터와 스키마를 추가하고 다시 앱을 빌드해 들어가보면 

![Screen Shot 2020-09-05 at 2.30.03 PM](/Users/nayeong-yun/Library/Application Support/typora-user-images/Screen Shot 2020-09-05 at 2.30.03 PM.png)

이런 오류가 생성된다. 

=> 암호화를 하지 않았기 때문에(아까 Spring Security 5 버전부터는 의무적으로 PasswordEncoder를 사용해서 비밀번호를 암호화해야한다고 했다.) 



하지만 암호화 코드를 추가하더라도 인증은 안된다. 왜냐하면 로그인 시에 입력된 비밀번호를 암호화한 값과 암호화되지않은 users테이블의 password 값과 비교시 일치하지 않기 때문에!



따라서 코드를 제대로 테스트하기 위해서는 __비밀번호를 암호화하지 않는 PasswordEncoder를 임시로 작성__ 하고 테스트 해야한다. (즉 , 암호화 한척만 해주는것)



또한 스프링 시큐리티의 것과 다른 데이터베이스(예를 들면, 테이블이나 열의 이름이 다를 때)를 사용한다면 스프링 SQL 쿼리를 우리의 SQL 쿼리로 대체할 수 있다. 

```java
auth.jdbcAuthentication().dataSource(dataSource)
        .usersByUsernameQuery(
                "select username, password, enabled from users "+"where username=?")
        .authoritiesByUsernameQuery(
                "select username, authority from authorities "+ "where username=?"
        );
```

 

##### 스프링 시큐리티의 기본 SQL 쿼리를 우리의 것으로 대체할 때 지켜야 할 사항

* 매개변수(where 절에 사용됨)는 하나이며 username이어야 한다.
* 사용자 권한 쿼리에서는 해당 사용자 이름 부여된 권한을 포함하는 0 또는 다수의 행을 반환할 수 있다.
* 그리고 그룹 권한 쿼리에서는 각각 그룹 id, 그룹 이름, 권한 열을 갖는 0 또는 다수의 행을 반환 할 수 있다.



##### 암호화된 비밀번호 사용

* 비밀번호를 데이터베이스 저장할 때는 사용자가 입력한 비밀번호와 데이터베이스에 있는 비밀번호는 동일한 암호화 알고리즘을 사용해서 암호화 해야한다.



```java
auth.jdbcAuthentication().dataSource(dataSource)
        .usersByUsernameQuery(
                "select username, password, enabled from users "+"where username=?")
        .authoritiesByUsernameQuery(
                "select username, authority from authorities "+ "where username=?"
        )
        .passwordEncoder(new BCryptPasswordEncoder());// 여기만 추가
```



passwordEncoder를 구현한 종류

* BCryptPasswordEncoder: bcrypt를 해싱 암호화
* NoOpPasswrodEncoder : 암호화 x
* Pbkdf2PasswordEncoder: PBKDF2를 암호화한다.
* ScyptPasswordEncoder: scrypt를 해싱 암호화 한다.
* StandardPasswordEncoder: SHA-256을 해싱 암호화한다.
* 우리가 구현한 인코더



그렇다면 PasswordEncoder는 어떻게 되어있을까?

```java
public interface PasswordEncoder{
  String encode(CharSequence rawPassword); //암호화
  boolean matches(CharSequence rawPassword, String encodedPassword);//웹페이지 암호화 동일한 암호화 방식사용하여 비교
}
```





JDBC 기반으로 사용자를 인증하는 법을 알았다. 여기서는 인증이 제대로 되는지 확인하기 위해 비밀번호를 암호화 하지 않았고 이 방법은 테스트할 때만 임시로 사용되는 것이다. 따라서 JDBC기반으로 인증하는 jdbcAuthentication() 이외에 다른 인증 방법을 사용할 것이다. 

그전에 또 다른 사용자 LDAP(Lightweight Directory Access Protocol)를 알아보자





### LDAP 기반 사용자 스토어

* jdbcAuthentication()와 비슷한 ldapAuthentication() 메소드를 사용할 수 있다 .

```java
auth.ldapAuthentication()
                .userSearchBase("ou=people")//사용자를 검색하기 위한 기준점 쿼리 제공(//따라서 루트부터 검색안함)
                .userSearchFilter("(uid={0})")//사용자를 검색하기 위해서 
                .groupSearchBase("ou=groups")//그룹을 찾기 위한 기준점 쿼기 제공(따라서 루트부터 검색안함)
                .groupSearchFilter("member={0}")//그룹을 검색하기 위해서 
  							.passwordCompare();// 비밀번호를 비교하는 방법으로 LDAP 인증을 하고자 할 때 
   							.passwordEncoder(new BCryptPasswordEncoder())//암호화, LDAP 서버에 비밀번호 전달될때 해커가 가로챌수 있음
                  																					 //따라서 이를 방지하기 위해 암호화 시킴
                .passwordAttribute("userPasscode");
```

* 사용자와 그룹 모두의 LDAP 기본 쿼리는 비어 있어서 쿼리에 의한 검색이 LDAP 계층의 루트 부터 수행 된다는 것을 나타낸다. 
* 하지만 기준점 쿼리를 제공할 수 있다. 
* ou는 구성단위 즉 어디서 부터 시작할지를 알려준다. 



#### 비밀번호 비교 구성하기

* LDAP의 기본인증전략은 사용자가 직접 LDAP 서버에서 인증받도록 한다. 
* 비밀번호를 비교하는 방법도 있다.( 입력된 비밀번호를 LDAP 디렉터리에 전송 후 사용자의 비밀번호 속성 값과 비교하도록 DLAP 서버에 요청한다.)-- 서버에서 작업 수행 되어 실제 비밀번호는 노출 되지 않는다
* passwordCompare를 사용하면 로그인 폼에 입력된 비밀번호가 사용자의 LDAP 서버에 있는 userPassword 속성값과 비교 된다. 
* 만약 비밀번호가 다른 속성에 있다면 passwordAttribute()를 사용하여 비밀번호 속성의 이름을 지정할 수 있다. (ex. passwordAttribute("userPasscode"))





##### 원격서버 참조하기

* LDAP 서버는 어디있는 것일까? => localhost:33389 포트에 있다.
* 그러나 만약 LDAP 서버가 다른 컴퓨터에서 실행중이라면 contextSource()메서도를 사용해서 해당 서버의 위치를 구성할 수 있다. (.contextSource().url("ldap://tacocloud.com:389/dc=tacocloud,dc=com");)



##### 내장된 LDAP 서버 구성하기

* 인증을 기다리는 LDAP 서버가 없는 경우 스프링 시큐리티에서 제공하는 내장 LDAP서버 이용할 수 있음
* 내장된 LDAP 사용 할 떄는 원격 LDAP 서버의 URL을 설정하는 대신 root() 메서드를 사용해서 내장 LDAP 서버의 루트 경로를 지정할 수 있다.
* LDAP 서버가 시작될때는 classpath 에서 찾을 수 있는 LDIF 파일로 부터 데이터를 로드한다. 
* LDIF 파일이란 일반 텍스트 파일에 LDAP 데이터를 나타내는 표준화된 방법, 각 레코드는 하나이상의 줄로 구성되며 각줄은 한쌍으로 된 name: value를 포함한다. 각 레코드는 빈줄로 구분된다.
* 코드

```java
auth.ldapAuthentication()
                .userSearchBase("ou=people")//사용자를 검색하기 위한 기준점 쿼리 제공(//따라서 루트부터 검색안함)
                .userSearchFilter("(uid={0})")//사용자를 검색하기 위해서
                .groupSearchBase("ou=groups")//그룹을 찾기 위한 기준점 쿼기 제공(따라서 루트부터 검색안함)
                .groupSearchFilter("member={0}")//그룹을 검색하기 위해서
                .contextSource()
                .root("dc=tacocloud,dc=com")
                .ldif("classpath:users.ldif")//classpath의 루트에서 users.ldif 파일 찾아서 LDAP 서버로 데이터를 로드하라고 요청
                .and()
                .passwordCompare()
                .passwordEncoder(new BCryptPasswordEncoder())
                .passwordAttribute("userPasscode");
```





> 지금까지 알아보았던 스프링 시큐리티에 내장된 사용자 스토어는 편리하고 일반적인 용도로 사용하기 좋다.
>
> 하지만 만약 스프링에 내장된 사용자 스토어가 우리 요구를 충족하지 못할 떄는 우리가 커스텀 사용자 명세 서비스를 생성하고 구성해야 한다.

__TIP__ 

* 로그인 같은 테스트를 진행할 시 Incognito 모드를 이용하는 것이 좋다. 	
  * 쿠키, 임시 인터넷 파일등을 저장하지 못하기 때문이다. 따라서 보안 관련 변경이 확실하게 적용되었는지 알 수 있다.
* 보안에서 username은 ID에 해당한다.





참고자료

![image-20200905140300480](/Users/nayeong-yun/Library/Application Support/typora-user-images/image-20200905140300480.png)

SpEL

* The Spring Expression Language (SpEL for short) is a powerful expression language that supports querying and manipulating an object graph at runtime.



* varchar와 varchar2의 차이

  * varchar은 ms-sql에서 사용하는 형식이고 varchar2는 oracle에서 사용하는 형식이다.
  * 결국 문법상으로 같은 형식이고 사용하는 DBMS에 따라서 다르다.

* varchar과 char의 차이

  * varchar은 흔히 주소값과 같이 변동이 쉬울 때 적용하는 형식
  * char은 주민등록번호과 같이 형식이 정해져 있을때 사용하면 효율적 
  * varchar은 가변형 길이를 말한다. varchar(20)라고 하면 크기가 20바이트가 아니고, 실제로 입력하는 바이트의 길이가 된다.
* Default Method(PasswordEncoder 다 implement 안해서)
   * 인터페이스는 기능에 대한 선언만 가능하기 때문에, 실제 코드를 구현한 로직은 포함될 수 없습니다. 하지만 자바8에서 이러한 룰을 깨트리는 기능이 나오게 되었는 데, 그것이 Default Method(디펄트 메소드)입니다. 메소드 선언시에 default를 명시하게 되면 인터페이스 내부에서도 코드가 포함된 메소드를 선언 할 수 있습니다. (접근제어자에서 사용하는 default와 같은 키워드 이지만, 접근제어자는 아무것도 명시 하지 않은 접근 제어자를 default라고 하며 인터페이스의 default method는 'default'라는 키워드를 명시해야 합니다.)













