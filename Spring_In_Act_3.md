# JDBC를 사용해서 데이터 읽고 쓰기
   
## 알아둘 것
   
**1. JDBC ?**  
: Java Database Connectivity, 자바에서 데이터베이스로 접근하게 해주는 API이다. 쉽게 말해 자바로 데이터를 CRUD 할 수 있게 해주는 애
   
**2. JdbcTemplate ?**     
: JDBC를 지원해주는 클래스

**2-1. JdbcTemplate를 사용하지 않고 DB 쿼리**
[참고](https://happynewmind.tistory.com/55)
```java
@Override
public Ingredient findById(String id){
   Connection connection = null;
   PreparedStatement statement = null;
   ResultSet resultSet = null;
try{
   connection = dataSource.getConnection();
   statement = connection.prepareStatement(                          //prepareStatement 생성
      "select id , name, type from Ingredient shere id = ?");        //물음표 값 나중에 넣겠다. 쿼리문 재사용할거다!
   statement.setString(1,id);                                        //쿼리문의 첫번째 물음표에 id를 넣겠다!
   resultSet = statement.executeQuery();                               //쿼리 실행
   Ingredient = ingredient = null;
   if (resultSet.next()){                                            //다음 레코드 읽기(여기서는 1번째 레코드 읽는것,처음에 resultSet은 0번째를 가리키고 있음)
      ingredient = new Ingredient(
         resultSet.getString("id"),                                  //첫번째 레코드의 id값가져와서 Ingredient 객체에 넣어줌
         resultSet.getString("name").
         Ingredient.Type.valueOf(resultSet.getString("type")));
}
   return ingredient;
} catch (SQLException e){                                            //DB 연결,쿼리문, 결과 오류 잡기
} finally {                                                          //prepareStatement 닫기
   if (resultSet != null){
      try {
         resultSet.close();
      } catch (SQLException e) {}
   }
   if(statement != null){
      try { 
      statement.close();
      } catch(SQLException e){}
   }
   if (connection != null) {
      try{
      connection.close();
      } catch (SQLException e) {}
      }
   }
   return null;
}
```
**2-2. JdbcTemplate 이용해서 DB 쿼리하기**
```java
private JdbcTemplate jdbc;

@Override
public Ingredient findById(String id){
   return jdbc.queryForObject(								//쿼리 실행결과가 1개일때 queryForObject() 사용, 여러개면 query()사용
      "select id, name, type from Ingredient shere id =?"
      this::mapRowToIngredient, id);
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
   throws SQLException{
 return new Ingredient(
         rs.getString("id"),
         rs.getString("name"),
         Ingredient.Type.valueOf(rs.getString("type")));
}
```

**JdbcTemplate을 썼을 때**
```
- 쿼리문, 연결 담는 객체 생성 안했다.
- 결과 담는 객체는 있음! (mapRowToIngredient함수가 만들어줌)
- 예외 처리하는 catch도 없다.
```
  
    
    
## 프로젝트 적용
* setting
-> dependency 설정 : jdbctemplate 와 h2databse(내장 데이터베이스)
  
* 도메인 수정(식별자 id 만들어줌)    
: Taco, Order 수정. 
**#main/tacos/Taco.java**
```java
...
@Data
public class Taco {
	
	private Long id;
	private Date createdAt;

...
}
```
**#main/tacos/Order.java**
```java
...
@Data
public class Order {
	
	private Long id;
	private Date placedAt;
...
}
```

```
패턴
- ingredient 데이터 다룰 함수들 간단히 적은 interface 생성 (IngredientRepository)
- interface에 적은 함수들 JdbcingredientRepository에 구현
- DesignController에 주입
```

1. Ingredient
: DesignController에서 ingredient를 정의해 주는거 말고 DB에서 가져와 볼 것이다!   

1-1. IngredientRepository   
:인터페이스 생성 [참고](https://wikidocs.net/217)  
**#main/data/IngredientRepository.java**
```java
package tacos.data;

import tacos.Ingredient;

public interface IngredientRepository {

	Iterable<Ingredient> findAll();

	Ingredient findById(String id);

	Ingredient save(Ingredient ingredient);

}
```
1-2. JdbcIngredientRepository  
: ingredient 데이터 가져오고 저장하는 함수들 정의.  
```java
package tacos.data;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.sql.ResultSet;
import java.sql.SQLException;

import tacos.Ingredient;

@Repository										//Component annotation 중 db 관련 annotation
public class JdbcIngredientRepository implements IngredientRepository {			//3-1에서 구현한 ingredientRepository 인터페이스 구현

	private JdbcTemplate jdbc;

	@Autowired									//Autowired annotation으로 jdbcTemplate에 연결
	public JdbcIngredientRepository(JdbcTemplate jdbc) {
	  this.jdbc = jdbc;
	}

	@Override
	  public Iterable<Ingredient> findAll() {
	    return jdbc.query("select id, name, type from Ingredient",			//jdbc.query함수는 List형태로 쿼리결과를 return, 
	      this::mapRowToIngredient);						//쿼리 결과 행 개수만큼 호출됨. 한 번 호출할 때 마다 ingredient 객체 생성
	  }										//모든 객체 만들어지면 List로 return됨

	  @Override
	  public Ingredient findById(String id) {
	    return jdbc.queryForObject(
	      "select id, name, type from Ingredient where id=?",
	        this::mapRowToIngredient, id);
	  }

	  private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
	    throws SQLException {
	      return new Ingredient(
		    rs.getString("id"),
		    rs.getString("name"),
		    Ingredient.Type.valueOf(rs.getString("type")));
	  }

	  @Override
	  public Ingredient save(Ingredient ingredient) {				//쿼리 결과를 저장
	    jdbc.update(
	        "insert into Ingredient (id, name, type) values (?, ?, ?)",
	        ingredient.getId(),							//첫번째 물음표에 들어갈 값
	        ingredient.getName(),							//두번째
	        ingredient.getType().toString());					//세번째
	    return ingredient;
	  }

}
```
1-3. DesignTacoController 수정(p83)   
```
- chapter2에서 하드코딩 했던 ingredients List를 삭제해주기  
- 데이터 다루는 함수 쓰기 위해 IngredientRepository 객체 생성해주기  
- DB에서 ingredient 가져와서 List만들어주기  
```

* Table 정의
```
- classpath 루트 경로에 schema.sql 생성 (src/main/resources폴더에)
```
```sql
create table if not exists Ingredient (
  id varchar(4) not null,
  name varchar(25) not null,
  type varchar(10) not null
);

create table if not exists Taco (
  id identity,
  name varchar(50) not null,
  createdAt timestamp not null
);

create table if not exists Taco_Ingredients (
  taco bigint not null,
  ingredient varchar(4) not null
);

alter table Taco_Ingredients
    add foreign key (taco) references Taco(id);
alter table Taco_Ingredients
    add foreign key (ingredient) references Ingredient(id);

create table if not exists Taco_Order (
  id identity,
  deliveryName varchar(50) not null,
  deliveryStreet varchar(50) not null,
  deliveryCity varchar(50) not null,
  deliveryState varchar(2) not null,
  deliveryZip varchar(10) not null,
  ccNumber varchar(16) not null,
  ccExpiration varchar(5) not null,
  ccCVV varchar(3) not null,
  placedAt timestamp not null
);

create table if not exists Taco_Order_Tacos (
  tacoOrder bigint not null,
  taco bigint not null
);

alter table Taco_Order_Tacos
    add foreign key (tacoOrder) references Taco_Order(id);
alter table Taco_Order_Tacos
    add foreign key (taco) references Taco(id);
```
3-5. Data 저장
```
- src/main/resource 폴더에 data.sql 파일 생성 후 insert 이용해서 데이터 추가
```
```sql
delete from Taco_Order_Tacos;
delete from Taco_Ingredients;
delete from Taco;
delete from Taco_Order;

delete from Ingredient;
insert into Ingredient (id, name, type)
    values ('FLTO', 'Flour Tortilla 토르티아', 'WRAP');
insert into Ingredient (id, name, type)
    values ('COTO', 'Corn Tortilla', 'WRAP');
insert into Ingredient (id, name, type)
    values ('GRBF', 'Ground Beef', 'PROTEIN');
insert into Ingredient (id, name, type)
    values ('CARN', 'Carnitas', 'PROTEIN');
insert into Ingredient (id, name, type)
    values ('TMTO', 'Diced Tomatoes', 'VEGGIES');
insert into Ingredient (id, name, type)
    values ('LETC', 'Lettuce', 'VEGGIES');
insert into Ingredient (id, name, type)
    values ('CHED', 'Cheddar', 'CHEESE');
insert into Ingredient (id, name, type)
    values ('JACK', 'Monterrey Jack', 'CHEESE');
insert into Ingredient (id, name, type)
    values ('SLSA', 'Salsa', 'SAUCE');
insert into Ingredient (id, name, type)
    values ('SRCR', 'Sour Cream', 'SAUCE');
```
2. Taco 
2-1. TacoRepository   
: Taco 정보 저장해 줄 함수 적은 인터페이스
```java
package tacos.data;

import tacos.Taco;

public interface TacoRepository {

	Taco save(Taco design);
	
}
```
2-2. JdbcTacoRepository   
: TacoRepository(3-5-1-1)의 save함수 구현   
```
상황
- 브라우저에서 Taco 정보를 입력받아서 Taco 객체에 정보를 담은 상태

순서
- Taco 테이블에 column name과 createdAt에 데이터 저장(saveTacoInfo함수), 고유 id도 저장(saveTacoInfo함수 리턴값으로 변수에 저장됨 나중에 쓰이므로)
- Taco에 부여된 고유 id와 Taco를 만든 Ingredients id들을 Taco_Ingredients 테이블에 저장
```
```java
package tacos.data;

import java.sql.Timestamp;
import java.sql.Types;
import java.util.Arrays;
import java.util.Date;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.core.PreparedStatementCreatorFactory;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;

import tacos.Ingredient;
import tacos.Taco;

@Repository
public class JdbcTacoRepository implements TacoRepository {

	private JdbcTemplate jdbc;

	  public JdbcTacoRepository(JdbcTemplate jdbc) {
	    this.jdbc = jdbc;
	  }

	  @Override
	  public Taco save(Taco taco) {									//taco정보를 저장해보자
	    long tacoId = saveTacoInfo(taco);								//새로운 taco 정보 저장 위한 함수(taco객체에 이름과 생성시간 데이터 입력해줌)
	    taco.setId(tacoId);										//saveTacoInfo에서 전달받은 taco생성시 자동 생성되는 고유 Id도 데이터 입력해줌
	    										//왜 saveTacoInfo함수 실행시 고유id까지 저장안해주고 따로 return을 해서 입력해주나?
											//왜냐면 밑의 for문에서 saveIngredientToTaco 실행할때 tacoId 값을 써야하니까
	    // 스프링 Converter를 우리가 구현한 IngredientByIdConverter의 Convert() 메서드가 이때 자동 실행된다.
	    for (Ingredient ingredient : taco.getIngredients()) { 							//새로 taco만들 때 어떤 재료들로 만드는지도 같이 저장해줌
	      saveIngredientToTaco(ingredient, tacoId);									//saveIngredientToTaco함수를 이용해서 재료 정보를 타코에 저장
	    }

	    return taco;
	  }

	  private long saveTacoInfo(Taco taco) {
	    taco.setCreatedAt(new Date());							//Taco class에서 @Data로 만들어진 getter setter 함수 중 하나로 생성 시간 입력해줌
	    PreparedStatementCreator psc =							//psc 객체에 밑에 오는 쿼리문을 저장해줌(PreparedStatementCreatorFactory 생성자)
	        new PreparedStatementCreatorFactory(						
	            "insert into Taco (name, createdAt) values (?, ?)",				
	            Types.VARCHAR, Types.TIMESTAMP						//첫번째와 두번째 물음표 타입 정해줌
	        ).newPreparedStatementCreator(							// ??						
	           Arrays.asList(								//물음표들에 들어갈 값을 Taco객체에서 List형태로 가져와서 DB에 저장할 준비
	               taco.getName(),
	               new Timestamp(taco.getCreatedAt().getTime())));

	    KeyHolder keyHolder = new GeneratedKeyHolder();					//Taco가 새로 생겨날때마다 부여되는 고유 Id를 생성하는애
	    jdbc.update(psc, keyHolder);							//PreparedStatementCreatorFactory로 만든 쿼리문, 저장할 데이터가 담긴 객체 psc와 고유 id생성하는 keyholder를 인자로 전달해서 DB에 저장함

	    return keyHolder.getKey().longValue();						//Taco 고유 id 리턴
	  }

	  private void saveIngredientToTaco( 							//각 Taco id와 그 Taco를 만드는 ingredeint id를 Taco_Ingredients 테이블에 저장하는 함수
	          Ingredient ingredient, long tacoId) {	
	    jdbc.update(				
	        "insert into Taco_Ingredients (taco, ingredient) " +
	        "values (?, ?)",
	        tacoId, ingredient.getId());							//Taco id와 ingredient id를 테이블에 저장
	  }

}
```
2-3. DesignController 파일 수정 
```
할일 
1. DesignController가 위에서 만든 TacoRepository 사용할 수 있게 연결시켜주기
2. taco 모델과 order모덷 생성해주기
3. order은 taco여러개를 주문하는 경우 한 주문에 여러개 타코 정보가 들어가므로 이때 여러번의 HTTP 요청이 발생한다 -> sessionAttributes 어노테이션을 써줌
4. 브라우저에서 받은 Taco 정보를 유효성 검사하고 DB에 저장
5. 이 Taco 객체를 Order 객체에도 넣어줌
```
```java
package tacos.web;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

import javax.validation.Valid;

import org.springframework.validation.Errors;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.SessionAttributes;

import tacos.Order;
import tacos.data.IngredientRepository;


import lombok.extern.slf4j.Slf4j;
import tacos.Taco;
import tacos.Ingredient;
import tacos.Ingredient.Type;
import tacos.data.TacoRepository;

@Slf4j
@Controller
@RequestMapping("/design")
@SessionAttributes("order")										//3. order은 여러번의 http 요청 발생하므로 이 어노테이션을 
public class DesignTacoController {

	private final IngredientRepository ingredientRepo;
	
	private TacoRepository tacoRepo;								//1. tocorepository 연결

	@Autowired
	public DesignTacoController(
			IngredientRepository ingredientRepo, TacoRepository tacoRepo) {
	  this.ingredientRepo = ingredientRepo;
	  this.tacoRepo = tacoRepo;
	}

	@GetMapping
	  public String showDesignForm(Model model) {
	    
		List<Ingredient> ingredients = new ArrayList<>();
	    ingredientRepo.findAll().forEach(i -> ingredients.add(i));

	    Type[] types = Ingredient.Type.values();
	    for (Type type : types) {
	      model.addAttribute(type.toString().toLowerCase(),
	          filterByType(ingredients, type));
	    }

	    model.addAttribute("taco", new Taco());

	    return "design";
	  }
	
	  private List<Ingredient> filterByType(
	      List<Ingredient> ingredients, Type type) {
	    return ingredients
	              .stream()
	              .filter(x -> x.getType().equals(type))
	              .collect(Collectors.toList());
	  }

	  @ModelAttribute(name = "order")						//2. order 모델 생성
	  public Order order() {
	    return new Order();
	  }

	  @ModelAttribute(name = "taco")						//2. taco 모델 
	  public Taco taco() {
	    return new Taco();
	  }

	  @PostMapping
	  public String processDesign(
			  @Valid Taco design, 							//4. 브라우저에서 Taco주문 시 Taco정보 입력한 것을 유효성 검사
			  Errors errors, @ModelAttribute Order order) {				
		  if (errors.hasErrors()) {
			 return "design";
		  }

		  Taco saved = tacoRepo.save(design);	//4. 3-5-1-2(바로위) 에서 구현했던 save함수를 사용해서 입력받은 Taco정보를 db에 저장 후(return Taco) saved 변수에 저장  
		  order.addDesign(saved);			//5. order객체에 Taco객체 저장 (Order 도메인(클래스)에 아직 addDesign함수 정의 안돼있어서 지금은 오류남)

		  return "redirect:/orders/current";
	  }

}
```
2-4. Order 도메인 수정    
: addDesign 함수 추가해줌(바로 위 코드에서 오류났던것)   
```java
package tacos;

import java.util.ArrayList;
import java.util.List;
import java.util.Date;

import javax.validation.constraints.Digits;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.NotBlank;
import org.hibernate.validator.constraints.CreditCardNumber;

import lombok.Data;

@Data
public class Order {
	
	private Long id;
    private Date placedAt;


	@NotBlank(message="Name is required")
	private String deliveryName;
	
	@NotBlank(message="Street is required")
	private String deliveryStreet;
	
	@NotBlank(message="City is required")
	private String deliveryCity;
	
	@NotBlank(message="State is required")
	private String deliveryState;
	
	@NotBlank(message="Zip code is required")
	private String deliveryZip;
	
	@CreditCardNumber(message="Not a valid credit card number")
	private String ccNumber;
	
	@Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
	           message="Must be formatted MM/YY")
	private String ccExpiration;
	
	@Digits(integer=3, fraction=0, message="Invalid CVV")
	private String ccCVV;
	
	private List<Taco> tacos = new ArrayList<>();
	
	public void addDesign(Taco design) {
	  this.tacos.add(design);
	}

}
```
3. Order   
```
할일
- Taco 정보를 DB에 저장해줬던 것과 마찬가지로 Taco_Order 테이블에 주문에만 관련된 데이터를 저장해주고 
- Taco_Order_Tacos 테이블에 Order id와 Taco id를 저장해줄것이다. 

다룰 것
- Taco 정보를 DB에 입력해줄때 아까 PreparedStatementCreator를 썼다면 지금은 좀 더 쉬운 SimpleJdbcInsert를 사용할 것이다.
``` 
3-1. OrderRepository   
: Order 정보를 저장해주는 함수를 적은 인터페이스 
```java
package tacos.data;

import tacos.Order;

public interface OrderRepository {

	Order save(Order order);
	
}
```
3-2. JdbcOrderRepository   
: Order의 save함수 구현   
```java
package tacos.data;

import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import org.springframework.stereotype.Repository;

import com.fasterxml.jackson.databind.ObjectMapper;

import tacos.Taco;
import tacos.Order;

@Repository
public class JdbcOrderRepository implements OrderRepository {

	private SimpleJdbcInsert orderInserter;
	private SimpleJdbcInsert orderTacoInserter;
	private ObjectMapper objectMapper;

	@Autowired
	public JdbcOrderRepository(JdbcTemplate jdbc) {
	  this.orderInserter = new SimpleJdbcInsert(jdbc)				//원래 private JdbcTemplate jdbc; 한 다음에 주입시켜 주던것을 (p89참고) 이렇게 바꿈
	      .withTableName("Taco_Order")						//Taco_Order 테이블에 데이터를 추가하기 위한 orderInserter 인스턴스 생성
	      .usingGeneratedKeyColumns("id");						//db에서 자동 생성되는 id 사용

	  this.orderTacoInserter = new SimpleJdbcInsert(jdbc)				//Taco_Order_Tacos 테이블에 데이터 추가하기 위한 orderTacoInserter 인스턴스 생성
	      .withTableName("Taco_Order_Tacos");

	  this.objectMapper = new ObjectMapper();					//Order 객체와 Taco 객체를 DB에 저장할 때 이용할 인스턴스
	}

	@Override
	public Order save(Order order) {						//Order 정보 저장하는 save함수 구현
	  order.setPlacedAt(new Date());						//브라우저에서 Order 정보 받아올 때 시간 정보는 받지 않으므로 시스템에서 해줘야함
	  long orderId = saveOrderDetails(order);					//브라우저에서 받은 Order정보를 DB에 저장해주는 saveOrderDetails함수 실행
	  order.setId(orderId);
	  List<Taco> tacos = order.getTacos();
	  
	  for (Taco taco : tacos) {
	    saveTacoToOrder(taco, orderId);
	  }

	  return order;
	}

	private long saveOrderDetails(Order order) {					
	  @SuppressWarnings("unchecked")
	  Map<String, Object> values =							//브라우저에서 받은 Order 객체의 정보를 Map 객체 생성해서 Map에 넣어줌
	      objectMapper.convertValue(order, Map.class);
	  values.put("placedAt", order.getPlacedAt());					//Map으로 변환하는 과정에서 placedAt이 타입이 long으로 바뀌기 때문에 위에서 정의해준 Date 타입이 아니므로 위에서 생성해준 PlacedAt 데이터를 가져와서 그걸로 Map의 PlacedAt를 바꿔줘야함

	  long orderId =
	      orderInserter
	          .executeAndReturnKey(values)						//저 위에서 만들어줬던 Taco_Order 테이블에 데이터 추가하는데 사용하는 orderInserter 실행
	          .longValue();					//executeAndReturnKey 함수 실행 결과 DB에서 자동생성해주는 Id를 Number 객체로 return, 그걸 long 타입으로 변환
	  return orderId;					
	}

	private void saveTacoToOrder(Taco taco, long orderId) {				//Taco_Order_Tacos 테이블에 data저장하는 함수
	  Map<String, Object> values = new HashMap<>();					//Map 인스턴스를 생성
	  values.put("tacoOrder", orderId);						//orderid값을 Map에 tacoOrder column에 넣음
	  values.put("taco", taco.getId());
	  orderTacoInserter.execute(values);						//DB에 저장
	}

}
```
3-3. OrderController 수정   
```java
package tacos.web;

import javax.validation.Valid;

import org.springframework.validation.Errors;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.bind.support.SessionStatus;

import lombok.extern.slf4j.Slf4j;
import tacos.Order;
import tacos.data.OrderRepository;

@Slf4j
@Controller
@RequestMapping("/orders")
@SessionAttributes("order")						//한 주문에 여러개 Taco 정보를 담고 있어야 하므로 session 필요
public class OrderController {

	private OrderRepository orderRepo;

	public OrderController(OrderRepository orderRepo) {
	  this.orderRepo = orderRepo;
	}

	@GetMapping("/current")
	  public String orderForm() {
	    return "orderForm";
	  }

	@PostMapping
	  public String processOrder(@Valid Order order, 
			  Errors errors, SessionStatus sessionStatus) {
		if (errors.hasErrors()) {
		    return "orderForm";
		}

		orderRepo.save(order);						//주문 DB저장
	    sessionStatus.setComplete();					//한 주문이 끝나면 session clear해줌

	    return "redirect:/";
	  }

}
```
4. 브라우저에서 입력한 Ingredient(String 타입)을 Ingredient 객체 타입으로 변환해야할 일이 생기면 자동으로 실행되는 convert()함수 Override   
: tacos/web 에 IngredientByIdConverter.java 클래스 만들기
```java
package tacos.web;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;

import tacos.Ingredient;
import tacos.data.IngredientRepository;

@Component
public class IngredientByIdConverter 
		implements Converter<String, Ingredient> {

	private IngredientRepository ingredientRepo;

	@Autowired
	public IngredientByIdConverter(IngredientRepository ingredientRepo) {
	  this.ingredientRepo = ingredientRepo;
	}
	  
	@Override
	public Ingredient convert(String id) {
	  return ingredientRepo.findById(id);				//ingredient id를 가지고 DB에서 해당 row 찾아낸 후 ingredient 객체로 생성시켜서 return 해줌
	}

}
```
