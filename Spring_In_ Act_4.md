Spring_In_ Act4

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



----



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

그전에 또 다른 사용자 LDAP(Lightweight Directory Access Protocol)를 알아보자.



-----



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



#### 사용자 인증의 커스터마이징

```java
@Entity
@Data
@NoArgsConstructor(access= AccessLevel.PRIVATE, force = true)
/*
	접근 권한 필드들이 final로 생성되어 있는 경우에는 필드를 초기화 할 수 없기 때문에 
  생성자를 만들 수 없고 에러가 발생하게 됩니다.
  이 때는 @NoArgsConstructor(force = true) 옵션을 이용해서 final 필드를 0, false, null 등으로 
   초기화를 강제로 시켜서 생성자를 만들 수 있습니다.
   */
@RequiredArgsConstructor
public class User implements UserDetails {
    private static final long serialVersionUID=1L;

    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;

    private final String username;
    private final String password;
    private final String fullname;
    private final String street;
    private final String city;
    private final String state;
    private final String zip;
    private final String phoneNumber;
  
  //해당 메소드는 해당 사용자에게 부여된 권한을 저장한 컬렉션을 반환한다.

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities()
    {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public boolean isAccountNonExpired()
    {
        return true;
    }

    @Override
    public boolean isAccountNonLocked()
    {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired()
    {
        return true;
    }
    @Override
    public boolean isEnabled()
    {
        return true;
    }

}
```

> User클래스는 UserDetails를 구현하였는데  이로인하여 해당 사용자에게 부여된 권한과 해당사용자 계정을 사용할 수 있는지 여부등을 알려줌.





`UserRepository 인터페이스`

```java
public interface UserRepository extends CrudRepository<User, Long> {
    User findByUsername(String username); //이 메소드는 사용자 이름 즉 ,id로 user를 찾기위한 메소드이다.
}

```

3장에서 배웠듯이 JPA는 인터페이스의 구현체를 런타임시 자동으로 생성한다. 





이제 UserRepository도 있으니 UserService를 생성해보자

```java
@Service//서비스의 역할은 Repo를 사용하여 처리하는 역할
public class UserRepositoryUserDetailsService implements UserDetailsService {

    private UserRepository userRepo;

    @Autowired
    public UserRepositoryUserDetailsService(UserRepository userRepo)//의존성주입
    {
        this.userRepo=userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) //절대로 null을 반환하지 않는 규칙이 있음.
            throws UsernameNotFoundException {
        User user=userRepo.findByUsername(username);//따라서 findByUsername메소드가 null을 반환하면 Exception 발생
        if(user!=null)
        {
            return user;
        }
        throw new UsernameNotFoundException(
                "User "+username+" not found"
        );

    }
}
```



UserDetailService의 형태

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

Username 즉 id가 있으면 UserDetails 객체를 반환하거나 없으면 UsernameNotFoundException 예외를 발생시킨다.



이때 다시 커스텀 명세 서비스를 스프링 시큐리티에 구성하는것을 해야한다. 

따라서 

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Qualifier("userRepositoryUserDetailsService")
    @Autowired//의존성 주입
    private UserDetailsService userDetailsService;//어떻게 UserDetailsService인터페이스를 의존성 주입을 해줄까?
  																								//우리가 해당 메소드 구현한 UserRepositoryUserDetailsService가 들어옴.
  
  	@Bean//리턴값 Spring Container가 관리해줌 
  	public PasswordEncoder encoder()
    {
      return new BCryptPasswordEncoder();
    }
  
  	@Override
  	protected void configure(AuthenticationManagerBuilder auth) throws Exception
    {
      auth
        .userDetailService(userDetailsService)
        .passwordEncoder(encoder());
    }
```







이제는 DB에 사용자 정보를 저장하는 방법이 필요하다. 따라서 사용자 등록 페이지를 생성하자

RegistrationController//등록 폼을 보여주고 처리한다. 

```java
@Controller//컴포넌트 자동검생이 되어야한다는것을 알림
@RequestMapping("/register")
public class RegistrationController {
    private UserRepository userRepo;
    private PasswordEncoder passwordEncoder;

    public RegistrationController(UserRepository userRepo, PasswordEncoder passwordEncoder)
    {
        this.userRepo=userRepo;
        //방금 추가 했던 SecurityConfig 클래스에 추가했던 passwordEncoder과 같은 빈
        this.passwordEncoder=passwordEncoder;
    }

    @GetMapping
    public String registerForm()
    {
        return "registration";
    }

    @PostMapping
    public String processRegistration(RegistrationForm form)//registration의 폼이 제출되면 이 메소드 작동
    {
        userRepo.save(form.toUser(passwordEncoder));
        return "redirect:/login";
    }
}
```



`registration.html`

```java
<form method="POST" th:action="@{/register}" id="registerForm">
```

> 이를 보면 /register로 들어와서 `registration.html` 을 봐서 그대로 입력을 진행한다. 제출을 하면 다시 `/register` 로 가고 이번에는 POST메소드로 간다. 따라서 processRegistration이 장동하고 여기서 form 의 User를 저장한다.



`RegistrationForm`

```java
@Data
public class RegistrationForm {
    private String username;
    private String password;
    private String fullname;
    private String street;
    private String city;
    private String state;
    private String zip;
    private String phone;

    public User toUser(PasswordEncoder passwordEncoder)
    {
        return new User(
                username
                ,passwordEncoder.encode(password)
                ,fullname
                ,street
                ,city
                ,state
                ,zip
                ,phone
        );
    }
    
```



이제 타코 클라우드 애플리케이션의 사용자 등록과 인증 지원이 완성 되었다. 그러나 지금은 애플리케이션을 시작해도 등록 페이지를 볼 수 없다. 

기본적으로 모든 웹 요청은 인증이 필요하기 때문이다. 이 문제를 해결하기 위해 지금부터는 웹 요청의 보안을 처리를 살펴본다.





### 웹 요청 보안 처리하기

> 타코를 디자인 하거나 주문하기 전에 사용자를 인증해야 한다. 그러나 홈페이지, 로그인 페이지, 등록페이지는 인증되지 않은 모든 사용자가 사용할 수 있어야 한다. 
>
> => 이러한 보안 규칙을 구성하기 위해서는 SecurityConfig 클래스에 다음의 configure(HttpSecurity) 메소드를 오버라이딩 해야한다. 

```java
@Override
protected void configure(HttpSecurity http) throws Exception
{
  
}
```

이 configure() 메서도는 HttpSecurity 객체를 인자로 받는다.  이 객체를 활용 하여 구성할수 있는것은 다음과 같다.

* HTTP 요청 처리를 허용하기 전에 충족되어야 할 특정 보안 조건을 구성한다.
* 커스텀 로그인 페이지를 구성한다.
* 사용자가 애플리케이션의 로그아웃을 할 수 있도록 한다.
* CSRF 공격으로부터 보호하도록 구성한다. 



처리하기

=> /design과 /order의 요청은 인증된 사용자에게만 해야하고 나머지는 모든 사용자가 허용되어야 한다. 

```java
@Override
    protected void configure(HttpSecurity http) throws Exception
    {
        http.authorizeRequests()
                .antMatchers("/design","orders")// /design과 /order의 요청은 
                .hasRole("ROLE_USER") //ROLE_USER의 권한을 갖는 사용자에게만 허용된다.
                .antMatchers("/","/**")
          			.permitAll()//이외의 모든 요청은 모든 사용자에게 허용된다.
    }
```

> 순서가 중요하다. `antMatchers()` 에서 지정된 경로의 패턴 일치를 검사하므로 먼저 지정된 보안 규칙이 우선 적으로 처리됨



`SecurityConfig 보안 구성 클래스의 최종 코드`

```java
package tacos.security;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

import javax.sql.DataSource;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Qualifier("userRepositoryUserDetailsService")
    @Autowired
    private UserDetailsService userDetailsService;

    @Bean//왜 자기 여기에 갑자기 Bean을 해준 것일까?
         //여기에 Bean을 함으로서 우리가 얻는 이득은 BCryptPasswordEncoder()인스턴스가 애플리케이션 컨텍스트로 부터 주입되어 반환된다.
         //이렇게 함으로써 우리가 원하는 종류의 PasswordEncoder 빈 객체를 스프링 관리하에 사용할 수 있다.
    public PasswordEncoder encoder()
    {
        return new BCryptPasswordEncoder();
    }


    //로그인 페이지는 뷰만 있어서 매운 간단하 WebConfig에 뷰컨트롤러로 선언해도 충
    //기본 로그인 페이지를 교체하기 위해 커스텀 로그인 페이지가 있는 경로를 스프링 시큐리티에 알려줘야 함
    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        http.authorizeRequests()
                .antMatchers("/design","orders")
                .hasRole("ROLE_USER")
                .antMatchers("/","/**")
                .permitAll()
                .and()//인증 구성 코드와 연결, 인증 구성이 끝나서 추가적인 HTTP 구성을 적용할 준비가 되었다는것을 나타낸다.
                .formLogin()//커스텀 로그인 폼을 구성하기 위해 호출.
                .loginPage("/login")//커스텀 로그인 페이지의 경로를 지정
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .csrf();
        
    }

    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception
    {


        auth
        .userDetailsService(userDetailsService)
        .passwordEncoder(encoder());//비밀번호가 암호화 되어 데이터베이스에 저장될 수 있도록 암호화 해주는것
        /*
        auth.inMemoryAuthentication()
                .withUser("user1")
                .password("{noop}password1")
                .authorities("ROLE_USER")
                .and()
                .withUser("user2")
                .password("{noop}password2")
                .authorities("ROLE_USER");

         */


//        auth.jdbcAuthentication().dataSource(dataSource)
//        .usersByUsernameQuery(
//                "select username, password, enabled from users "+"where username=?")
//        .authoritiesByUsernameQuery(
//                "select username, authority from authorities "+ "where username=?"
//        )
//        .passwordEncoder(new NoEncodingPasswordEncoder());
//        auth.ldapAuthentication()
//                .userSearchBase("ou=people")//사용자를 검색하기 위한 기준점 쿼리 제공(//따라서 루트부터 검색안함)
//                .userSearchFilter("(uid={0})")//사용자를 검색하기 위해서
//                .groupSearchBase("ou=groups")//그룹을 찾기 위한 기준점 쿼기 제공(따라서 루트부터 검색안함)
//                .groupSearchFilter("member={0}")//그룹을 검색하기 위해서
//                .passwordCompare()// 비밀번호를 비교하는 방법으로 LDAP 인증을 하고자 할 때
//                .passwordEncoder(new BCryptPasswordEncoder())
//                .passwordAttribute("userPasscode")
//                .contextSource()
//                .root("dc=tacocloud,dc=com");

//        auth.ldapAuthentication()
//                .userSearchBase("ou=people")//사용자를 검색하기 위한 기준점 쿼리 제공(//따라서 루트부터 검색안함)
//                .userSearchFilter("(uid={0})")//사용자를 검색하기 위해서
//                .groupSearchBase("ou=groups")//그룹을 찾기 위한 기준점 쿼기 제공(따라서 루트부터 검색안함)
//                .groupSearchFilter("member={0}")//그룹을 검색하기 위해서
//                .contextSource()
//                .root("dc=tacocloud,dc=com")
//                .ldif("classpath:users.ldif")//classpath의 루트에서 users.ldif 파일 찾아서 LDAP 서버로 데이터를 로드하라고 요청
//                .and()
//                .passwordCompare()
//                .passwordEncoder(new BCryptPasswordEncoder())
//                .passwordAttribute("userPasscode");

    }

}

```



#### CSRF 공격 방어하기

* 스프링 시큐리티에는 내장된 CSRF 방어기능 존재하며 활성화 되어있음.

* 개발자가 별도로 구성할 필요없이 CSRF 토큰을 넣을 _csrf 라는 이름의 필드를 애플리케이션이 제출하는 폼에 포함시키면 됨.

* _csrf라는 이름의 요청 속성에 넣으면 된다.

* 만일 스프링 MVC의 JSP 태그 라이브러리 또는 Thymeleaf를 스프링 시큐리티 dialect와 함께 사용중이라면 숨김 필드 조차도 자동으로 생성됨

* 예를 들어 Thymeleaf가 숨김필드를 포함하기 위해서는 다음과 같이 th:action 속성만 지정하면 됨

* ```html
  <form method="post" th:action="@{/login}" id="loginForm"
  ```

* CSRF 지원은 거의 왠만하면 절대 비활성화 하지말자. (단, REST API서버로 실행되는 애플리케이션의 경우 CSRF를 disable 해야함.)

```java
.and()
.csrf()
.disable()
```







### 사용자 인지하기

* OrderController에서 주문 폼과 바인딩 되는 Order 객체를 최초 생성할 때 해당 주문을 하는 사용자의 이름과 주소를 주문 폼에 미리 넣을 수 있는것
* 사용자 주문 데이터를 데이터베이스에 저장할 때 주문이 생성되는 User와 Order를 연관시킬 수 있어야 한다. 
* DB에서 Order 개체와 User 개체를 연관시키기 위해서는 Order 클래스에 새로운 속성을 추가해야한다.

```java
@Data
@Entity
@Table(name="Taco_Order")
public class Order implements Serializable  {
	
	private static final long serialVersionUID = 1L;
	......
  @ManyToOne //한건의 주문이 한명의 사용자에속한다는 것을 나타낸다. 즉, 한명의 사용자는 여러주문을 가질 수 있다.
	private User user;
}
```



> 즉 한명의 사용자가 여러주문을 가질 수 있는 것이다. 



사용자가 누구인지 결정하는 방법

* Principal 객체를 컨트롤러 메서드에 주입한다.

  * ```java
    @PostMapping
    public String processOrder(@Valid Order order, Errors errors, 
                               SessionStatus sessionStatus,
                               Principal principal)
    {
      ...
        User user=userRepository.findByUsername(principal.getName())
    }
    ```

    

* Authentication 객체를 컨트롤러 메서드에 주입한다

* SecurityContextHolder를 사용해서 보안 컨텍스트를 얻는다

* @AuthenticationPrincipal 애노테이션을 메서드에 지정한다. 









__TIP__ 

* 로그인 같은 테스트를 진행할 시 Incognito 모드를 이용하는 것이 좋다. 	
  * 쿠키, 임시 인터넷 파일등을 저장하지 못하기 때문이다. 따라서 보안 관련 변경이 확실하게 적용되었는지 알 수 있다.
* 보안에서 username은 ID에 해당한다.





## 참고자료

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

* @ Entity
  *  일반적으로 **RDBMS에서 Table을 객체화** 시킨 것으로 보면 된다.
  *  Entity 어노테이션을 클래스에 선언하면 그 클래스는 JPA가 관리한다.
  *  그러므로 DB의 테이블과 Class와 맵핑한다면 반드시 @Entity를 붙여주어야 한다.
     *  제약사항
        *  1. 필드에 final, enum, interface, class를 사용할 수 없습니다.
           2. 생성자중 기본 생성자가 반드시 필요합니다.
     *  속성
        *  name : 엔티티 이름을 지정합니다. 기본값으로 클래스 이름을 그대로 사용한다.

![Screen Shot 2020-09-05 at 7.01.44 PM](/Users/nayeong-yun/Library/Application Support/typora-user-images/Screen Shot 2020-09-05 at 7.01.44 PM.png)



* @ Data
  
  * ![image-20200905195726463](/Users/nayeong-yun/Library/Application Support/typora-user-images/image-20200905195726463.png)
  
* @Table

  * Table 어노테이션은 맵핑할 테이블을 지정합니다.
  * @Table의 속성
    * name : 매핑할 테이블의 이름을 지정합니다.
    * catalog : DB의 catalog를 맵핑합니다.
    * schema : DB 스키마와 맵핑합니다.
    * uniqueConstraint : DDL 쿼리를 작성할 때 제약 조건을 생성합니다.

  

* @NoArgsConstructor
  * 파라미터가 없는 기본 생성자를 생성한다.
  * ![image-20200907162653420](/Users/nayeong-yun/Library/Application Support/typora-user-images/image-20200907162653420.png)



* @RequiredArgsConstructor 
  *  생성자를 자동 생성하지만, 필드명 위에 @NonNull로 표기된 경우만 생성자의 매개변수로 받는다. 
  
    
  
* @ID

  * primary key를 가지는 변수를 선언하는 것을 뜻한다.  



* @GeneratedValue

  * 해당 Id 값을 어떻게 자동으로 생성할지 전략을 선택하는것. 여기서 선택한 전략은 "AUTO"이다.

    

* @ModelAttribute

  * @ModelAttribute 선언 후 자동으로 진행되는 작업들은 다음과 같다.

    * `@RequestParam`과 비슷한데 1:1로 파라미터를 받을경우는 `@RequestParam`를 사용하고, 도메인이나 오브젝트로 파라미터를 받을 경우는 `@ModelAttribute`으로 받을수 있다. 또한 이 어노테이션을 사용하면 검증(Validation)작업을 추가로 할수 있는데 예로들어 null이라던지, 각 멤버변수마다 valid옵션을 줄수가 있고 여기서 에러가 날 경우 BindException 이 발생한다.

    * Using @ModelAttribute on a method argument data binding in Spring MVC, a very useful mechanism that saves you from having to __parse each form field individually__.

      


* CrudRepository

  * org.springframework.data.repository.CrudRepository,CRUD 기능을 제공하는 리파지토리 인터페이스

  * 메소드

    ![image-20200905202612829](/Users/nayeong-yun/Library/Application Support/typora-user-images/image-20200905202612829.png)



* remember-me
  * 이전 로그인 정보를 쿠키나 데이터베이스로 저장한 후 일정기간내에 다시 접근 시 저장된 정보로 자동 로그인이 된다.



* CSRF(Cross-Site Request Forgery)
  * 크로스 사이트 요청 위조
  * 사용자가 웹사이트에 로그인한 상태에서 악의적인 ㅗㅋ드가 삽입된 페이지를 열면 공격대상이 되는 웹사이트에 자동으로 폼이 제출되고 이사이트는 위조된 공격명령이 믿을 수 있는 사용자로부터 제출된 것으로 판단 되어 공격에 노출됨 















