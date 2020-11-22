# SpringBoot2单元测试

## 引入maven依赖

引入powermock是为了解决静态方法mock的问题。
```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>2.0.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>2.0.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.28.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.11.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>2.0.2-beta</version>
    <scope>test</scope>
</dependency>
```

## 构建单元测试目录

标准的maven单元测试目录一样，在resources目录里面添加application.yml内容如下：
```yaml
spring:
  profiles:
    active: test

---
spring:
  profiles: test

dc:
  security:
    auditlog:
      module: "System Manage"        #模块名
logging:
  level:
    root: INFO
```

## 定义Application入口类

```java
@ActiveProfiles("test")
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class, MultipartAutoConfiguration.class})
public class Application4Test extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application4Test.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application4Test.class, args);
    }
}
```

## 抽象测试父类

```java
@RunWith(PowerMockRunner.class)
@PowerMockRunnerDelegate(SpringRunner.class)
@PowerMockIgnore({"javax.management.*","javax.net.*", "javax.net.ssl.*"})
@SpringBootTest(classes = Application4Test.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@ContextConfiguration(initializers = {ApplicationInitializer4Test.class})
public abstract class AbstractSpringTest {

    @MockBean
    protected ISecurityContextService securityContextService;
    @Autowired
    protected Map<String, HttpRequestInterceptor> HttpRequestInterceptorMap;
    @SpyBean
    protected HttpClientProperties httpClientProperties;
    @Autowired
    @Qualifier("restTemplate")
    protected RestTemplate eurekaRestTemplate;
    @Autowired
    @Qualifier("simpleRestTemplate")
    protected RestTemplate simpleRestTemplate;
}
```

## 启动覆盖yml配置

如果yml文件中的配置项需要覆盖，可实现ApplicationContextInitializer

```java
public class ApplicationInitializer4Test implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    private static final Logger logger = LoggerFactory.getLogger(ApplicationInitializer4Test.class);

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        Resource rootCertResource = new ClassPathResource("cert/root.p12");
        Resource serverCertResource = new ClassPathResource("cert/server.p12");
        String rootCertPath = "";
        String clientCertPath = "";
        try {
            File rootCertFile = rootCertResource.getFile();
            File serverCertFile = serverCertResource.getFile();
            rootCertPath = rootCertFile.getCanonicalPath().replaceAll("\\\\", "/");
            clientCertPath = serverCertFile.getCanonicalPath().replaceAll("\\\\", "/");
            logger.info("rootCertFilePath=" + rootCertPath);
            logger.info("clientCertFilePath=" + clientCertPath);
        } catch (IOException e) {
            e.printStackTrace();
        }
        TestPropertySourceUtils.addInlinedPropertiesToEnvironment(
            applicationContext, "server.ssl.keyStore=" + clientCertPath);
        TestPropertySourceUtils.addInlinedPropertiesToEnvironment(
            applicationContext, "server.ssl.trustStore=" + rootCertPath);
    }
}
```

## 实际单元测试用例

```java
@PrepareForTest({SecurityContextHolder.class, ClientContextHolder.class})
public class ApiOperationLogServiceTest extends AbstractSpringTest {

    private ApiOperationLogService operationLogService;

    @Before
    public void setUp() throws Exception {
        System.out.println("###################test start##########################");
        operationLogService = new ApiOperationLogService(properties, restTemplate);
    }

    @After
    public void tearDown() throws Exception {
        System.out.println("###################test end##########################");
    }

    @Test
    public void save() {
        ResultVo resultVo = new ResultVo();
        resultVo.setResultCode(0);
        resultVo.setResultMessage("save log success");
        ResponseEntity<ResultVo> response = new ResponseEntity<>(resultVo, HttpStatus.OK);
        doReturn(response).when(restTemplate).exchange(eq(properties.getOperationLogUrl()), eq(HttpMethod.POST),
            isA(HttpEntity.class), eq(ResultVo.class));
        operationLogService.save(new OperationLog());
    }

    @Test
    public void testLoadOperatorId() {
        // PowerMockito打桩，模拟静态方法
        PowerMockito.mockStatic(SecurityContextHolder.class);
        SecurityContextImpl securityContext = new SecurityContextImpl();
        securityContext.setAuthentication(
            new UsernamePasswordAuthenticationToken("xiongneng", "123456"));
        PowerMockito.when(SecurityContextHolder.getContext()).thenReturn(securityContext);
        assertEquals(operationLogService.loadOperatorId(), "xiongneng");
    }

    @Test
    public void testLoadClientIpWhenRemoteUserIp() {
        // PowerMockito打桩，模拟静态方法
        PowerMockito.mockStatic(ClientContextHolder.class);
        ClientContext clientContext = new ClientContext();
        clientContext.setClientIP("192.168.20.22");
        clientContext.setRemoteUserIP("30.200.12.22");
        PowerMockito.when(ClientContextHolder.getContext()).thenReturn(clientContext);
        assertEquals(operationLogService.loadClientIp(), "30.200.12.22");
    }

    @Test
    public void testLoadClientIpWhenNoRemoteUserIp() {
        // PowerMockito打桩，模拟静态方法
        PowerMockito.mockStatic(ClientContextHolder.class);
        ClientContext clientContext = new ClientContext();
        clientContext.setClientIP("192.168.20.22");
        PowerMockito.when(ClientContextHolder.getContext()).thenReturn(clientContext);
        assertEquals(operationLogService.loadClientIp(), "192.168.20.22");
    }
}
```

## 调用Controller接口测试

```java
public class RestTemplateTest extends AbstractSpringTest {

    @LocalServerPort
    private int port;

    private URL base;

    @Before
    public void setUp() throws Exception {
        this.base = new URL("https://localhost:" + port);
    }

    @Test
    public void testBidirectionCertificate() {
        ResponseEntity<String> response = simpleRestTemplate.getForEntity(base.toString() + "/welcome", String.class);
        assertEquals(response.getBody(), "welcome");
    }
}
```

## MockBean和SpyBean区别

spy对象和mock对象的两点区别：

1、默认行为的不同

对于未指定mock的方法，spy默认会调用真实的方法，有返回值的返回真实的返回值，而mock默认不执行，有返回值的，默认返回null。

2、mock的使用方式不同

mock对象的使用方式如下，注意spy对象这样使用会直接调用该方法，所以无法这样使用。比如：
``` java
Mockito.when(obj.domethod(parm1, param2)).thenReturn(result);
```

spy对象的使用方式，要先执行do等方法，mock对象也可以这样使用，比如：
``` java
Mockito.doReturn(info).when(obj).domethod(param1, param2);
```

`@Spy` 和 `@SpyBean` 的区别，`@Mock` 和 `@MockBean`的区别

1. spy和mock生成的对象不受spring管理
2. spy调用真实方法时，其它bean是无法注入的，要使用注入，要使用SpyBean
3. SpyBean和MockBean生成的对象受spring管理，相当于自动替换对应类型bean的注入，比如@Autowired等注入

## 模拟void方法

对void方法的模拟有两种方式，一种是通过抛出异常，一种是通过Answer来指定void的执行过程。

抛出期望的异常：
```java
doThrow(RuntimeException.class).when(daoMock).updateEmail(any(Customer.class), any(String.class));
```

指定void的执行过程：
```java
doAnswer((Answer<Void>) invocation -> {
    Object[] args = invocation.getArguments();
    System.out.println("restTemplate.exchange called with arguments: " + Arrays.toString(args));
    return null;
}).when(restTemplate).exchange(anyString(), eq(HttpMethod.POST),
    isA(HttpEntity.class), eq(ResultVo.class));

// 执行真实方法
doAnswer(Answers.CALLS_REAL_METHODS.get()).when(mock).voidMethod(any(SomeParamClass.class));

```
