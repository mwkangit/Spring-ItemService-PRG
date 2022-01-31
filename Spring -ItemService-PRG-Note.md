# Practice Setting



```groovy
plugins {
	id 'org.springframework.boot' version '2.6.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

```

- #### 이전과 설정은 동일하게 하며 Jar 파일을 이용한다.



# Practice Item-Service



## Flow & Domain & Repository



![service flow](/media/mwkang/Klevv/Spring 일지/MVC1/01.19/service flow.png)

- #### 백엔드 개발자는 HTML 화면이 나오기 전까지 시스템을 설계하고 핵심 비즈니스 모델을 개발해놔야 한다. 이후 HTML이 나오면 HTML을 뷰 템플릿으로 변환하여 동적 화면을 생성하고 웹 화면의 흐름을 제어한다.



```java
// Domain

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

- #### 이러한 핵심 도메인 상황에는 @Data가 위험하다. 즉, 필요에 따라 @Getter, @Setter을 선언하는 것이 좋다.

- #### DTO는 단순 데이터를 다루기 때문에 @Data를 사용해도 된다.

- #### Price와 같이 null일 수는 있지만 입력하지 않았을 경우 0이 되면 안돼는 객체는 int 대신 Integer로 선언해야 한다.



```java
// Respository

@Repository
public class ItemRepository {

    // static으로 유일하게 만들면서 전역으로 사용한다
    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;

    public Item save(Item item){
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id){
        return  store.get(id);
    }

    public List<Item> findAll(){
        return new ArrayList<>(store.values());
    }

    public void update(Long itemId, Item updateParam){
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    public void clearStore(){
        store.clear();
    }
}
```

- #### 저장 변수에 static을 선언하여 전역으로 사용하며 빈을 이용하는 효과를 볼 수 있다.

- #### 멀티스레드 환경에서 동시에 여러 스레드가 접근하면 map과 long은 오류가 발생할 수 있다. 이때 오류를 방지하기 위해 ConcurrentHashMap, AtomicLong을 사용하는 것이 좋다.

- #### Update() 메소드의 경우 파라미터로 보통 업데이트 값만 다루는 DTO를 사용한다. 중복보다는 항상 명확성을 중요하게 생각하여 DTO를 다루는 것이 좋다.



## Product List



```java
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model){
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);

        return "basic/items";
    }


	/**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init(){
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));

    }
}
```

- #### @RequiredArgsConstructor이 없으면 @Autowired로 생성자를 주입해야 한다.

- #### 저장된 데이터가 없으면 로직이 잘 실행되는지 테스트를 하기 번거롭기 때문에 @PostConstruct를 이용하여 더미 데이터를 저장했다. 빈의 의존관계가 모두 주입된 후에 @PostContruct가 호출된다.



```html
<!--items.html-->

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">

    <!--th:href-->
    <link th:href="@{/css/bootstrap.min.css}"
            href="../css/bootstrap.min.css" rel="stylesheet">
    
    <!--th:onclick-->
    <button class="btn btn-primary float-end"
            th:onclick="|location.href='@{/basic/items/add}'|"
            onclick="location.href='addForm.html'" 
            type="button">상품등록
    </button>
    
    <table class="table">
        <thead>
            <tr>
                <th>ID</th>
                <th>상품명</th>
                <th>가격</th>
                <th>수량</th>
            </tr>
        </thead>
        <tbody>
            <!--th:each-->
            <tr th:each="item : ${items}">
                <td><a href="item.html" th:href="@{/basic/items/${itemId}(itemId=${item.id})}" th:text="${item.id}">회원id</a></td>
                <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}" th:text="${item.itemName}">상품명</a></td>
                <!--th:text-->
                <td th:text="${item.price}">10000</td>
                <td th:text="${item.quantity}">10</td>
            </tr>
        </tbody>
    </table>
</html>
```

- #### 타임리프의 동적 html 생성에 초점을 맞춰서 기존의 html을 타임리프로 변경한 부분만 다룰 것이다.

- #### 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다. 대부분의 html 속성을 th:xxx로 변경하는 것이 가능하다. Th:xxx가 없으면 기존 html이 xxx 속성을 그대로 사용한다.

- #### Html을 파일로 직접 열었을 때 th:xxx가 있어도 웹 브라우저는 th: 속성을 모르기 때문에 무시하고 기존의 xxx를 실행한다. 즉, html을 파일 보기를 유지하여 순수 html 화면 수정이 용이하고 템플릿 기능도 수행하는 것이 가능하다. 이렇게 순수 html을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 natural template이라고 한다(JSP 경우 웹 브라우저에서 열면 JSP 소스코드와 html이 뒤죽박죽 되어서 정상적인 확인이 불가능하다).

- #### 타임리프 사용 선언

  - #### ex) `<html xmlns:th="http://www.thymeleaf.org">`

- #### 속성 변경 - th:href

  - #### HTML을 그래도 볼 때는 `href` 속성이 사용되고 뷰 템플릿을 거치면 `th:href`의 값이 `href`를 대체하면서 동적으로 변경한다.

  - #### ex) `th:href="@{/css/bootstrap.min.css}"`

- #### 속성 변경 - th:onclick

  - #### ex) `th:onclick="|location.href='@{/basic/items/add}'|"`

- #### 반복 출력 - th:each

  - #### For-each문과 유사한 성질을 가지고 모델에서 넘어온 객체를 사용할 수 있다.

  - #### 컬렉션 수 만큼 `<tr>..</tr>`이 하위 태그를 포함해서 생성된다.

  - #### ex) `<tr th:each="item : ${items}">...</tr>`

- #### 내용 변경 - th:text

  - #### 내용의 값을 동적으로 변경 가능하다.

  - #### 위 코드에서 price는 default로 10000이지만 타임리프로 동적으로 변경된다.

  - #### ex) `<td th:text="${item.price}">10000</td>`

- #### URL 링크 표현식 - @{...}

  - #### 타임리프는 url 링크를 사용하는 경우 @{...}를 사용하며 이것을 URL 링크 표현식이라고 한다.

  - #### URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다(스프링 부트가 내장하면서 잘 사용하지 않게 되었다).

  - #### ex) `th:href="@{/css/bootstrap.min.css}"`

- #### URL 링크 표현식2 - @{...}

  - #### URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.

  - #### 경로 변수 `{itemId}`뿐만 아니라 쿼리 파라미터도 생성할 수 있다.

  - #### ex) `th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"`

    #### 생성 링크 : `http://localhost:8080/basic/items/1?query=test`

- #### URL 링크 간단히

  - #### 리터럴 대체 문법을 활용해서 간단히 사용할 수 있다.

  - #### 명확하게 볼 필요없는 간단한 경우에는 이렇게 사용하는 것도 가능하다.

  - #### ex) `th:href="@{|/basic/items/${item.id}|}"`

- #### 리터럴 대체 - |...|

  - #### 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다(자바 문법과 비슷하다).

    - #### ex) `<span th:text="'Welcome to our application, ' + ${user.name} + '!'">`

      #### `th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"`

  - #### 리터럴 대체 문법을 사용하면 더하기 없이 편리하게 사용하는 것이 가능하다.

    - #### `<span th:text="|Welcome to our application, ${user.name}!|">`

      #### `th:onclick="|location.href='@{/basic/items/add}'|"`

- #### 변수 표현식 - ${...}

  - #### 모델의 포함된 값이나 타임리프 변수로 선언한 값을 조회할 수 있다.

  - #### 프로퍼티 접근법을 사용한다(getter, setter).

  - #### ex) `<td th:text="${item.price}">10000</td>`



## Product Details



```java
@GetMapping("/{itemId}")
public String item(@PathVariable long itemId, Model model){
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/item";
}
```

- #### 상품에 대한 상세한 내용을 확인하게 해주는 것으로 모델에 객체를 넣어서 상세 정보를 전달한다.

- #### 이때 DTO 객체를 넣는 것이 좋다.



```html
<!--item.html-->

<!--th:value-->
<input type="text" id="itemId" name="itemId" class="form-control" value="1" th:value="${item.id}" readonly>
```

- #### 속성 변경 - th:value

  - #### 모델에 있는 상품 정보를 획득하여 프로퍼티 접근법으로 출력한다.

  - #### ex) `th:value="${item.id}"`



## Product Registration



### GET Form



```java
@GetMapping("/add")
public String addForm(){
    return "basic/addForm";
}
```

- #### 폼을 보여주고 등록하는 곳은 아니므로 GET 메소드를 사용한다.



```html
<!--addForm.html-->

<!--th:action-->
<form action="item.html" th:action method="post">
```

- #### 속성 변경 - th:action

  - #### HTML form에서 action에 값이 없으면 현재 url로 지정한 HTTP 메소드로 데이터를 전송한다. 나중에 렌더링 하면 `action = ""`로 표시된다.

  - #### ex) `th:action`



### POST Logic



#### View Template



```java
// @RequestParam 사용
@PostMapping("/add")
public String addItemV1(@RequestParam String itemName,
                        @RequestParam int price,
                        @RequestParam Integer quantity,
                        Model model
                       ){
    Item item = new Item();
    item.setItemName(itemName);
    item.setPrice(price);
    item.setQuantity(quantity);

    itemRepository.save(item);

    model.addAttribute("item", item);

    return "basic/item";
}

// @ModelAttribute 사용
@PostMapping("/add")
public String addItemV2(@ModelAttribute("item") Item item, Model model){

    itemRepository.save(item);

    // @ModelAttribute에 이름을 지정하여 바로 모델에 추가하여 아래 코드 생략
    //        model.addAttribute("item", item);

    return "basic/item";
}

@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item){

    itemRepository.save(item);

    // @ModelAttribute에 name이 없으면 객체명 첫글자를 소문자로 바꾼 이름을 key로 하여 모델에 저장한다
    //        model.addAttribute("item", item);

    return "basic/item";
}

// redirect가 아니어서 새로고침 시 중복으로 추가 요청이 되는 문제가 발생한다
@PostMapping("/add")
public String addItemV4(Item item){

    // 객체 경우 자동으로 @ModelAttribute로 적용되어 자동으로 모델에 저장된다
    // 이 경우도 Item -> item 으로 객체명을 바꿔서 key로 사용한다
    itemRepository.save(item);

    return "basic/item";
}
```

- #### V1은 기본적인 방법으로 @RequestParam을 사용한다.

- #### V2부터 V4까지는 @ModelAttribute를 사용하게 된다.

- #### V2에서 보면 @ModelAttribute 선언 시 이름도 함께 선언하면 그 이름으로 자동으로 객체가 모델에 저장되어 저장코드를 만들지 않아도 된다.

- #### V3에서 보면 @ModelAttribute 선언 시 이름을 선언하지 않는다면 선언한 객체의 클래스 이름의 첫글자만 소문자로 만들어서 모델에 자동으로 등록한다.

- #### V4에서 보면 @ModelAttribute를 선언하지 않아도 V3와 같은 방법으로 모델에 객체가 자동으로 저장된다.



#### PRG



![PRG](/media/mwkang/Klevv/Spring 일지/MVC1/01.19/PRG.png)

```java
// "/add"를 redirect로 매핑하여 url을 변경하고 GET 메소드로 바꾼다
//    @PostMapping("/add")
public String addItemV5(Item item){
    itemRepository.save(item);
    return "redirect:/basic/items/" + item.getId();
}

// RedirectAttributes
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes){
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);

    return "redirect:/basic/items/{itemId}";
}
```

```html
<!--item.html-->

<!-- /add에서 상품이 잘 등록되었는지 확인하는 구간 -->
<!-- addItemV6에서 사용하며 status=true 오면 add가 잘 처리된 것이다 -->
<h2 th:if="${param.status}" th:text="'저장 완료'"> </h2>
```



- #### 위와 같이 뷰 템플릿으로 이동하여 상품 상세를 보여주는 방법에는 오류가 발생한다.

  - #### 뷰 템플릿으로 이동은 하지만 아직 마지막 요청이 `POST /add`로 설정되어 있다.

  - #### 새로고침하면 `POST /add`가 실행되어 다른 상품 아이디로 상품이 계속 등록된다.

  - #### 즉, 계속 원하지 않는 로직이 처리되는 것이다.

- #### 이러한 오류를 보완하려면 redirect를 사용하여 url과 HTTP 메소드를 GET으로 바꿔야 한다. 이러한 방법을 PRG(POST Redirect GET)이라고 한다.

- #### 반환 시 문자열에 템플릿 형식으로 넣을 수 있는 데이터는 보통 모델에 저장된 데이터, pathvariable이다.

- #### 자료형이 문자가 아니어도 그대로 반환형 String에 붙여서 경로를 만들 수 있다.

  - #### ex) `return "redirect:/basic/items/" + item.getId();`

- #### RedirectAttributes는 url 인코딩 해주고, pathvariable, 쿼리 파라미터까지 처리해준다.

  - #### PathVariable에 저장된 데이터를 바인딩할 수 있으며 사용하지 않은 나머지 데이터는 쿼리 파라미터로 전송된다.

- #### 조건 - th:if

  - #### 해당 조건이 참이면 실행하는 if문이라고 생각하면 된다.

  - #### ${param.status} 는 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능이다.

  - #### 기본 html이 없다면 정적 리소스로 웹 브라우저에서 소스보기를 했을 때 이 구문은 비어있다.

  - #### ex) `<h2 th:if="${param.status}" th:text="'저장 완료'"> </h2>`



## Product Modification



### GET Form



```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model){
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/editForm";
}
```

- #### ID 값으로 조회하여 폼에 이미 저장되어 있는 값을 보여주는 역할만 수행하기 때문에 GET 메소드를 이용한다.



### POST Logic



#### PRG



```java
@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item){

    itemRepository.update(itemId, item);
    return "redirect:/basic/items/{itemId}";
}
```

- #### Redirect 시 url을 완전히 변경하며 @Controller이 작동 중이어도 뷰를 호출하지 않고 url을 호출하게 된다.

- #### F12로 확인했을 때 상태코드 302(redirect)가 표시된다. 즉, 서바가 url을 클라이언트에게 주고 클라이언트가 받은 url로 GET 메소드 호출을 한 것이다.



# Additional Knowledge



- #### Static HTML 리소스를 확인하는 두 가지 방법이 있다.

  - #### Intellij에서 static에 위치한 html 파일 우클릭 - Copy Path/Reference -  Absolute Path 후 브라우저에 검색하면 렌더링된 html 파일을 화면에서 볼 수 있다.

  - #### 서버를 실행하여 확인하는 방법이 있다(스프링은 정적 리소스를 GET 메소드로만 호출되게 해놓아서 POST로 접근해야하는 리소스는 오류가 발생하여 확인할 수 없다).

