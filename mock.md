#1、什么是Mock?
mock是在测试过程中，对于一些不容易构造/获取的对象，创建一个mock对象来模拟对象的行为。比如说你需要调用B服务，可是B服务还没有开发完成，那么你就可以将调用B服务的那部分给Mock掉，并编写你想要的返回结果。
Mock有很多的实现框架，例如Mockito、EasyMock、Jmockit、PowerMock、Spock等等，SpringBoot默认的Mock框架是Mockito，和junit一样，只需要依赖spring-boot-starter-test就可以了。本文代码基于jdk8、junit5、Mockito3
##1.1、	Mockito中文文档
Mockito是mocking框架，它让你用简洁的API做测试。而且Mockito简单易学，它可读性强和验证语法简洁。Mockito是GitHub上使用最广泛的Mock框架,并与JUnit结合使用.Mockito框架可以创建和配置mock对象.使用Mockito简化了具有外部依赖的类的测试开发!
Mockito具体使用方法见文档https://github.com/hehonghui/mockito-doc-zh#0

##1.2、Mockito基本使用方法简介
###1）、静态导入会使代码更简洁
```
import static org.mockito.Mockito.*;
```
**举例：**
```
//创建mock对象，mock一个List接口
List mockedList = mock(List.class);
//如果不使用静态导入,则必须使用Mockito调用
List mockList = Mockito.mock(List.class);
```
###2）、验证某些行为
```
//你可以mock一个具体的类型，而不仅是接口
LinkedList mockedList = mock(LinkedList.class);

mockedList.add("one");


//验证
verify(mockedList).add("one");
```
>一旦mock对象被创建了，mock对象会记住所有的交互。然后你就可能选择性的验证你感兴趣的交互。

###3）、如何做一些测试桩
```
//测试桩
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());

当调用mockList.get(0)的时候，返回first
当调用mockList.get(1)的时候，抛出一个运行时异常
```
###4）、其他使用见上面文档

#2、MockMVC基于RESTful风格的测试
对于前后端分离的项目而言，无法直接从前端静态代码中测试接口的正确性，因此可以通过MockMVC来模拟HTTP请求。基于RESTful风格的SpringMVC的测试，我们可以测试完整的Spring MVC流程，即从URL请求到控制器处理，再到视图渲染都可以测试。

##2.1、初始化MockMvc对象
```
@Autowired
private WebApplicationContext webApplicationContext;
private MockMvc mockMvc;

//在每个测试方法执行之前都初始化MockMvc对象
@BeforeEach
public void setupMockMvc() {
    mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
}

```

##2.2、完成一些接口的测试
###1）、尝试测试一个不存在的请求 /user/1
```
/**
 * @DisplayName 自定义测试方法展示的名称
 * @throws Exception
 */
@DisplayName("测试根据Id获取User")
@Test
void contextLoads() throws Exception {

    //perform,执行一个RequestBuilders请求，会自动执行SpringMVC的流程并映射到相应的控制器执行处理
    mockMvc.perform(MockMvcRequestBuilders
        //构造一个get请求
        .get("/user/1")
        //请求类型 json
        .contentType(MediaType.APPLICATION_JSON))
        // 期待返回的状态码是4XX，因为我们并没有写/user/{id}的get接口
        .andExpect(MockMvcResultMatchers.status().is4xxClientError());

}
```
**展示结果：**
 
![image.png](https://upload-images.jianshu.io/upload_images/8020666-93c1ff8d696cc720.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###2）、在Controller中完成 /user/{id}
```
/**
 * id:\\d+只匹配数字
 * @param id
 * @return
 */
@GetMapping("/user/{id:\\d+}")
public User getUserById(@PathVariable Long id) {

    return userService.getById(id);
}
```
>修改一下测试类：期待返回的结果是200

```
@Test
void getUserById() throws Exception {
    //perform,执行一个RequestBuilders请求，会自动执行SpringMVC的流程并映射到相应的控制器执行处理
    mockMvc.perform(MockMvcRequestBuilders
            //构造一个get请求
            .get("/user/1")
            //请求类型 json
            .contentType(MediaType.APPLICATION_JSON))
            // 期望的结果状态 200
            .andExpect(MockMvcResultMatchers.status().isOk());
}
```
**结果展示：**

 ![image.png](https://upload-images.jianshu.io/upload_images/8020666-4d4a98e576884e52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###3）、我们可以把结果打印到控制台
```
// 期望的结果状态 200
.andExpect(MockMvcResultMatchers.status().isOk())
//添加ResultHandler结果处理器，比如调试时 打印结果(print方法)到控制台
.andDo(MockMvcResultHandlers.print());
```
**运行结果：可以看到并没有返回结果**
 ![image.png](https://upload-images.jianshu.io/upload_images/8020666-7eb445f2b42c9334.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###4）、结合Mockito构建自定义返回结果
>这里就用到了Mockito的应用场景，userService.getById并没有返回结果，但是我们的测试并不关心userService.getById这个方法是否正常，只是在我们的测试中需要用到这个方法，所以我们可以Mock掉UserService的getById方法，自己定义返回的结果，继续我们的测试。
```
@MockBean
private UserService userService;

@Test
void getUserById() throws Exception {

    User user = new User();
    user.setId(1);
    user.setNickname("yunqing");
    //Mock一个结果，当userService调用getById的时候，返回user
    doReturn(user).when(userService).getById(any());
    
    //perform,执行一个RequestBuilders请求，会自动执行SpringMVC的流程并映射到相应的控制器执行处理
    mockMvc.perform(MockMvcRequestBuilders
            //构造一个get请求
            .get("/user/1")
            //请求类型 json
            .contentType(MediaType.APPLICATION_JSON))
            // 期望的结果状态 200
            .andExpect(MockMvcResultMatchers.status().isOk())
            //添加ResultHandler结果处理器，比如调试时 打印结果(print方法)到控制台
            .andDo(MockMvcResultHandlers.print());
}
```
**运行结果**

 ![image.png](https://upload-images.jianshu.io/upload_images/8020666-ae4a129620e88282.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###5）、传参数
```
@Test
void getUserByUsername() throws Exception {
    // perform : 执行请求 ;
    mockMvc.perform(MockMvcRequestBuilders
            //MockMvcRequestBuilders.get("/url") ： 构造一个get请求
            .get("/user/getUserByName")
            //传参
            .param("username","admin")
            // 请求type : json
            .contentType(MediaType.APPLICATION_JSON))
            // 期望的结果状态 200
            .andExpect(MockMvcResultMatchers.status().isOk());
}

```



###6）、期望返回结果集有两个元素
```
@Test
void getAll() throws Exception {
    User user = new User();
    user.setNickname("yunqing");
    List<User> list = new LinkedList<>();
    list.add(user);
    list.add(user);
    //Mock一个结果，当userService调用list的时候，返回user
    when(userService.list()).thenReturn(list);
    //perform,执行一个RequestBuilders请求，会自动执行SpringMVC的流程并映射到相应的控制器执行处理
    mockMvc.perform(MockMvcRequestBuilders
            //构造一个get请求
            .get("/user/list")
            //请求类型 json
            .contentType(MediaType.APPLICATION_JSON))
            // 期望的结果状态 200
            .andExpect(MockMvcResultMatchers.status().isOk())
            //期望返回的结果集合有两个元素
            .andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(2))
            //添加ResultHandler结果处理器，比如调试时 打印结果(print方法)到控制台
            .andDo(MockMvcResultHandlers.print());
}
```
**运行结果：**

![image.png](https://upload-images.jianshu.io/upload_images/8020666-f1410735ab7ba100.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 




###7）、测试Post请求
```
@Test
void insert() throws Exception {

    User user = new User();
    user.setNickname("yunqing");
    String jsonResult = JSONObject.toJSONString(user);
    //直接自定义save返回true
    when(userService.save(any())).thenReturn(true);
    // perform : 执行请求 ;
    MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders
            //MockMvcRequestBuilders.post("/url") ： 构造一个post请求
            .post("/user/insert")
            .accept(MediaType.APPLICATION_JSON)
            //传参,因为后端是@RequestBody所以这里直接传json字符串
            .content(jsonResult)
            // 请求type : json
            .contentType(MediaType.APPLICATION_JSON))
            // 期望的结果状态 200
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andDo(MockMvcResultHandlers.print())
            .andReturn();//返回结果

    int statusCode = mvcResult.getResponse().getStatus();
    String result = mvcResult.getResponse().getContentAsString();
    //单个断言
    Assertions.assertEquals(200, statusCode);
    //多个断言，即使出错也会检查所有断言
    assertAll("断言",
            () -> assertEquals(200, statusCode),
            () -> assertTrue("true".equals(result))
    );
```
#3、一些常用API总结

##常用的期望：
```
//使用jsonPaht验证返回的json中code、message字段的返回值
.andExpect(MockMvcResultMatchers.jsonPath("$.code").value("00000"))
.andExpect(MockMvcResultMatchers.jsonPath("$.message").value("成功"))
//body属性不为空
.andExpect(MockMvcResultMatchers.jsonPath("$.body").isNotEmpty())
// 期望的返回结果集合有2个元素 ， $: 返回结果
.andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(2));
```
##附带常用API解释：
###RequestBuilder/MockMvcRequestBuilders：
```
//根据uri模板和uri变量值得到一个GET请求方式的MockHttpServletRequestBuilder；
MockHttpServletRequestBuilder get(String urlTemplate, Object... urlVariables)
//同get类似，但是是POST方法；
MockHttpServletRequestBuilder post(String urlTemplate, Object... urlVariables)
//同get类似，但是是PUT方法；
MockHttpServletRequestBuilder put(String urlTemplate, Object... urlVariables)
//同get类似，但是是DELETE方法；
MockHttpServletRequestBuilder delete(String urlTemplate, Object... urlVariables)
//同get类似，但是是OPTIONS方法；
MockHttpServletRequestBuilder options(String urlTemplate, Object... urlVariables)
//提供自己的Http请求方法及uri模板和uri变量，如上API都是委托给这个API；
MockHttpServletRequestBuilder request(HttpMethod httpMethod, String urlTemplate, Object... urlVariables)
//提供文件上传方式的请求，得到MockMultipartHttpServletRequestBuilder；
MockMultipartHttpServletRequestBuilder fileUpload(String urlTemplate, Object... urlVariables)
//创建一个从启动异步处理的请求的MvcResult进行异步分派的RequestBuilder；
RequestBuilder asyncDispatch(final MvcResult mvcResult)
```
###MockHttpServletRequestBuilder：
```
//：添加头信息；
MockHttpServletRequestBuilder header(String name, Object... values)/MockHttpServletRequestBuilder headers(HttpHeaders httpHeaders)
//：指定请求的contentType头信息；
MockHttpServletRequestBuilder contentType(MediaType mediaType)
//：指定请求的Accept头信息；
MockHttpServletRequestBuilder accept(MediaType... mediaTypes)/MockHttpServletRequestBuilder accept(String... mediaTypes)
//：指定请求Body体内容；
MockHttpServletRequestBuilder content(byte[] content)/MockHttpServletRequestBuilder content(String content)
//：请求传入参数
MockHttpServletRequestBuilder param(String name,String... values)
//：指定请求的Cookie；
MockHttpServletRequestBuilder cookie(Cookie... cookies)
//：指定请求的Locale；
MockHttpServletRequestBuilder locale(Locale locale)
//：指定请求字符编码；
MockHttpServletRequestBuilder characterEncoding(String encoding)
//：设置请求属性数据；
MockHttpServletRequestBuilder requestAttr(String name, Object value) 
//：设置请求session属性数据；
MockHttpServletRequestBuilder sessionAttr(String name, Object value)/MockHttpServletRequestBuilder sessionAttrs(Map<string, object=""> sessionAttributes)
//指定请求的flash信息，比如重定向后的属性信息；
MockHttpServletRequestBuilder flashAttr(String name, Object value)/MockHttpServletRequestBuilder flashAttrs(Map<string, object=""> flashAttributes)
//：指定请求的Session；
MockHttpServletRequestBuilder session(MockHttpSession session) 
// ：指定请求的Principal；
MockHttpServletRequestBuilder principal(Principal principal)
//：指定请求的上下文路径，必须以“/”开头，且不能以“/”结尾；
MockHttpServletRequestBuilder contextPath(String contextPath) 
//：请求的路径信息，必须以“/”开头；
MockHttpServletRequestBuilder pathInfo(String pathInfo) 
//：请求是否使用安全通道；
MockHttpServletRequestBuilder secure(boolean secure)
//：请求的后处理器，用于自定义一些请求处理的扩展点；
MockHttpServletRequestBuilder with(RequestPostProcessor postProcessor)
```
###MockMultipartHttpServletRequestBuilder
```
//：指定要上传的文件；
MockMultipartHttpServletRequestBuilder file(String name, byte[] content)/MockMultipartHttpServletRequestBuilder file(MockMultipartFile file)
```
###ResultActions
```
//：添加验证断言来判断执行请求后的结果是否是预期的；
ResultActions andExpect(ResultMatcher matcher) 
//：添加结果处理器，用于对验证成功后执行的动作，如输出下请求/结果信息用于调试；
ResultActions andDo(ResultHandler handler) 
//：返回验证成功后的MvcResult；用于自定义验证/下一步的异步处理；
MvcResult andReturn() 
```
###ResultMatcher/MockMvcResultMatchers
```
//：请求的Handler验证器，比如验证处理器类型/方法名；此处的Handler其实就是处理请求的控制器；
HandlerResultMatchers handler()
//：得到RequestResultMatchers验证器；
RequestResultMatchers request()
//：得到模型验证器；
ModelResultMatchers model()
//：得到视图验证器；
ViewResultMatchers view()
//：得到Flash属性验证；
FlashAttributeResultMatchers flash()
//：得到响应状态验证器；
StatusResultMatchers status()
//：得到响应Header验证器；
HeaderResultMatchers header()
//：得到响应Cookie验证器；
CookieResultMatchers cookie()
//：得到响应内容验证器；
ContentResultMatchers content()
//：得到Json表达式验证器；
JsonPathResultMatchers jsonPath(String expression, Object ... args)/ResultMatcher jsonPath(String expression, Matcher matcher)
//：得到Xpath表达式验证器；
XpathResultMatchers xpath(String expression, Object... args)/XpathResultMatchers xpath(String expression, Map<string, string=""> namespaces, Object... args)
//：验证处理完请求后转发的url（绝对匹配）；
ResultMatcher forwardedUrl(final String expectedUrl)
//：验证处理完请求后转发的url（Ant风格模式匹配，@since spring4）；
ResultMatcher forwardedUrlPattern(final String urlPattern)
//：验证处理完请求后重定向的url（绝对匹配）；
ResultMatcher redirectedUrl(final String expectedUrl)
//：验证处理完请求后重定向的url（Ant风格模式匹配，@since spring4）；
ResultMatcher redirectedUrlPattern(final String expectedUrl)
```
