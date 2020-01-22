---
title: Spring单元测试编写
date: 2020-01-22 14:09:01
tags: 单元测试
---
# SpringBoot单元测试
## Controller层
### 登录校验(一般情况不使用)

```java
@RunWith(SpringRunner.class)
@Transactional//测试结束后rollback
@SpringBootTest
public abstract class BaseTest {
    private static final LoginBean LOGINBEAN;
    static {
        LoginBean.Builder builder = new LoginBean.Builder();
        builder.setAccountID("7f2b815be57541549b7c4cd1e81f2923");
        builder.setCompanyID("08d181119a7b4c0e94ff368942fd4420");
        builder.setLoginName("utry");
        builder.setRealName("远传");
        LOGINBEAN = builder.build();
        LoginInfo.putSp(LOGINBEAN);
    }

    @Autowired
    protected ICacheService cache;
    
    {
        init();
    }

    private static void init() {
        preloadModules();
        splitMapper();
    }

    private static void preloadModules() {
        UtryCloudModuleManager.initModules();
    }

    private static void splitMapper() {
        new MybatisMapperSplitListener("mapper").onApplicationEvent(null);
    }

    @Before
    public void setUp() {
        LoginInfo.setThreadLocal(cache, LOGINBEAN);
    }
}
```

### 注解

- 测试类注解
```java
@RunWith(SpringRunner.class)
@SpringBootTest
```
- 模拟MVC环境注解以及代码
```java
private MockMVC mockMVC
@Autowired
private WebApplicationContext webApplicationContext;

```
- 通过@Before方法，提前准备测试环境

`注意（此处的MockMvc 实例化是通过手工方式创建，如果想通过spring的bean注入方式的话，在类上加@AutoConfigureMockMvc/@Controller等注解，只要等同于@Component效果即可，然后在上面的第2步中进行注入，即在成员变量mockMvc上加注解@Autowired）`

```java
@Before
public void setup(){
    /*通过bean注入方式
    mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    */

    /*手动指定需要测试controller类*/
    mockMvc = MockMvcBuilders.standaloneSetup(new RecordTapDataImportController()).build();
}

```

### 编写测试方法@Test
- 普通的测试方法
```java
/**
     * 获取最新app信息
     */
	@Test
	public void TestGetAppLatestVersion() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = post("/appProducer/getAppLatestVersion") 
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().isOk())//返回HTTP状态为200
		        .andExpect(jsonPath("$.status", not("E")))//使用jsonPath解析JSON返回值，判断具体的内容, 此处不希望status返回E
		        .andDo(print());//打印结果
		        //.andReturn();//想要返回结果，使用此方法
		
	}
	
	/**
	 * 用户通过微信方式扫描下载APP，则会提示使用浏览器打开地址
	 */
	@Test
	public void TestDownloadAppByMicroMessenger() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/downloadApp")
				.header("user-agent", "MicroMessenger")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().isOk())//返回HTTP状态为200
		        .andExpect(content().string(containsString("选择浏览器打开即可")))//返回结果中需包含的文字
		        .andDo(print());//打印结果
		
	}
	
	/**
	 * 用户通过Android扫描下载APP
	 */
	@Test
	public void TestDownloadAppByAndroid() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/downloadApp")
				.header("user-agent", "Android")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().isOk())//返回HTTP状态为200
		        .andDo(print());//打印结果
		
	}
	
	/**
	 * 用户通过iPhone扫描下载APP，则会重定向至评苹果APP官网
	 */
	@Test
	public void TestDownloadAppByIphone() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/downloadApp")
				.header("user-agent", "iPhone")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
				.andExpect(status().is3xxRedirection())//表示页面被重定向
				.andExpect(redirectedUrl("https://www.apple.com/cn/itunes/charts/"))//验证处理完请求后重定向的url
		        .andDo(print());//打印结果
		
	}
	
	/**
	 * 用户通过iPad扫描下载APP，则会重定向至评苹果APP官网
	 */
	@Test
	public void TestDownloadAppByIPad() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/downloadApp")
				.header("user-agent", "iPad")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().is3xxRedirection())//表示页面被重定向
		        .andExpect(redirectedUrl("https://www.apple.com/cn/itunes/charts/"))//验证处理完请求后重定向的url
		        .andDo(print());//打印结果
		
	}
	
	/**
	 * 用户通过其他方式扫描下载APP，则会提示仅支持的下载方式
	 */
	@Test
	public void TestDownloadAppByOther() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/downloadApp")
				.header("user-agent", "other")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().isOk())//返回HTTP状态为200
		        .andExpect(content().string(containsString("<h1>出现该页面可能是以下原因</h1>")))
		        .andDo(print());//打印结果
		
	}
	
	/**
	 * APP升级
	 */
	@Test
	public void TestUpgradeApp() throws Exception{
		RequestBuilder request = null;
		//构造请求
		request = get("/appProducer/upgradeApp")
				.param("appId", "1001"); 
		//执行请求
		mockMvc.perform(request) 
		        .andExpect(status().isOk())//返回HTTP状态为200
		        .andDo(print());//打印结果
		
	}
```

```java
@Test
public void controllerTest() throws Exception{

    VoiceData voiceData = new VoiceData();
    voiceData.setId("123");
    voiceData.setServiceName("jack");

    VoiceDataBo voiceDataBo = new VoiceDataBo();
    voiceDataBo.setCallId("123");
    voiceDataBo.setSeatName("tom");

    List dataList = new ArrayList();
    dataList.add(voiceData);

    /*构造一个请求*/
    RequestBuilder request = null;
    request = MockMvcRequestBuilders.post("/recordTapImport/voiceData")
            .contentType(MediaType.APPLICATION_JSON).content(JSON.toJSONString(dataList));

    /*执行一个请求*/
    mockMvc.perform(request).andDo(print()).andReturn();
}
```

- 当发送一个被@ResponceBody标识的参数，遇到400的返回错误。
    `注意上面contentType需要设置成MediaType.APPLICATION_JSON，即声明是发送“application/json”格式的数据。使用content方法，将转换的json数据放到request的body中。`

