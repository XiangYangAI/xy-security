## 使用Spring MVC 开发 RESTful API

```markdown
400 : 请求格式错误，比如controller要求传一个参数，但没有传
405 ：请求的get，post...不匹配
404 ：请求方法不存在
```



### 查询服务

>@RestController 标明此Controller提供RestAPI
>@RequestMapping 及其变体。映射http请求url到java方法
>@RequestParam 映射请求参数到java方法的参数
>@PageableDefault 制定分页参数默认值
>
>DTO 包下编写User对象，DTO包下的对象都是用来封装Restful API输入输出的数据的

* pom加入依赖

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-test</artifactId>
  </dependency>
  ```

* 在src/test/java下新建测试用例

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class UserControllerTest {
      
  	@Autowired
  	private WebApplicationContext wac;
  	private MockMvc mockMvc;
      
  	//在测试用例执行之前执行
      //伪造一个mvc环境
  	@Before
  	public void setup() {
  		mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
  	}
  
      //用户查询的测试用例
  	@Test
  	public void whenQurySuccess() throws Exception {
  		mockMvc.perform(get("/user")
                  .param("username", "jojo")
                  .contentType(MediaType.APPLICATION_JSON_UTF8))
                  .andExpect(status().isOk())
  			   .andExpect(jsonPath("$.length()").value(3)); //jsonPath在github上有详解
  	}
  }
  ```

  `Tips：在preference中找到favourite加入常用的类，之后在代码中可以直接写方法，不用写类名,类的方法采用静态导入的方式`

  ```java
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
  ```

* 用户查询controller

  ```java
  //用户查询controller
  //@RequestMapping(value = "/user", method = RequestMethod.GET)
  @GetMapping
  @JsonView(User.UserSimpleView.class)
  public List<User> query(@RequestParam String username){
  	List<User> users = new ArrayList<User>();
  	users.add(new User());
  	users.add(new User());
  	users.add(new User());		
  	return users;
  }
  ```

  * @RequestParam属性

    * required = false ：不是必填

    * name ：当http请求的参数和java方法中的参数名不一样时，指定http中的参数名称

    * defaultValue ：没有传参的默认值
  * 关于查询可以在DTO中加入一个类UserQueryCondition方便查询

    

### 详情服务

> @PathVariable 映射url片段到 java 方法的参数
>
> @JsonView 控制json输出内容
>
> 在url声明中使用正则表达式

* 测试先行

  ```java
  @Test
  public void whenGetInfoSuccess() throws Exception {
  	mockMvc.perform(get("/user/1")
  			.contentType(MediaType.APPLICATION_JSON_UTF8))
  			.andExpect(status().isOk())
  			.andExpect(jsonPath("$.username").value("tom"));
  }
  //测试传入的id必须为数字，而不是字母
  @Test
  public void whenGetInfoFail() throws Exception {
  	mockMvc.perform(get("/user/a").contentType(MediaType.APPLICATION_JSON_UTF8))
  			.andExpect(status().is4xxClientError());
  }
  
  ```

* 用户详情controller

  ```java
  @GetMapping("/{id:\\d+}") 
  @JsonView(User.UserDetailView.class)
  public User getInfo(@PathVariable(name = "id") String id) {
  	User user = new User();
  	user.setUsername("tom");
  	return user;
  }
  ```

  - 在其他语言中`\\`表示：我想要在正则表达式中插入一个普通的（字面上的）反斜杠，请不要给他任何特殊的意义；
  - 在java中，`\\`表示：我要插入一个正则表达式的反斜线，所以其后的字符具有特殊的意义。
  - 所以，在其他的语言中（如Perl），一个反斜杠 `\` 就足以具有转义的作用，而在 Java 中正则表达式中则需要有两个反斜杠才能被解析为其他语言中的转义作用。也可以简单的理解在 Java 的正则表达式中，两个` \\` 代表其他语言中的一个` \`，这也就是为什么表示一位数字的正则表达式是` \\d`，而表示一个普通的反斜杠是` \\\\`。 

* @JsonView 使用步骤

  > 注意到上面controller代码中的注释`@JsonView`，它使用的场景：虽然返回的是同一个类的对象（如User），但在不同业务场景下需要返回对象不同的属性，如有些业务需要password字段，有些不需要。

  * 使用接口声明多个视图，在对象的属性get方法上指定视图

    ```java
    public class User {
        //简单视图
    	public interface UserSimpleView {
    	};
    	//详细视图继承简单视图
    	public interface UserDetailView extends UserSimpleView {
    	};
    
    	private String id;	
    	private String username;	
    	private String password;
    	private Date birthday;
    
    	@JsonView(UserSimpleView.class)
    	public String getId() {
    		return id;
    	}
    
    	@JsonView(UserDetailView.class)
    	public String getPassword() {
    		return password;
    	}
    
    	@JsonView(UserSimpleView.class)
    	public String getUsername() {
    		return username;
    	}
        
    	@JsonView(UserSimpleView.class)
    	public Date getBirthday() {
    		return birthday;
    	}
        //属性set方法在此省略
    }
    
    ```

  * 在Controller方法上指定视图

    ```java
    见controller方法上的@JsonView注解
    ```

### 创建服务

> @RequestBody 映射请求体到java方法的参数
>
> 日期类型参数的处理
>
> @Valid 注解和 BindingResult 验证请求参数的合法性并处理校验结果

* 测试先行

  ```java
  @Test
  public void whenCreateSuccess() throws Exception {
  	Date date = new Date();
  	System.out.println(date.getTime());
  	String content = "{\"username\":\"tom\",\"password\":null,\"birthday\":" + date.getTime() + "}";
  	String result = mockMvc.perform(post("/user").contentType(MediaType.APPLICATION_JSON_UTF8)
  			.content(content))
  			.andExpect(status().isOk())
  			.andExpect(jsonPath("$.id").value("1"))
  			.andReturn().getResponse().getContentAsString();
  	System.out.println(result);
  }
  ```

* 创建服务controller

  ```java
  @PostMapping
  public User create(@Valid @RequestBody User user, BindingResult errors) {
  		
  	if(errors.hasErrors()) {
  		errors.getAllErrors().stream().forEach(error -> System.out.println(error.getDefaultMessage()));
  	}
  	System.out.println(user.getId());
  	System.out.println(user.getUsername());
  	System.out.println(user.getPassword());
  	System.out.println(user.getBirthday());
  		
  	user.setId("1");
  	
  	return user;		
  }
  ```

  - @RequestBody 与 @RequestParam 区别与应用场景，这里如果将 @RequestBody 换成@RequestParam会有什么意想不到的结果？
  - Date 在开发中的处理方式之一：传给前台不带格式的时间戳，是不是最妥的方法？