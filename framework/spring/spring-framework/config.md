
# 多环境配置

在实际项目中，大多包括多种环境，例如线上环境prod(product)、开发环境dev(development)、测试环境test、单元测试unitest等等。不同的环境需要进行不同的配置，或者执行不同的逻辑。Spring Boot为此提供了非常友好的支持，我们可以从 `@Profile` 和多资源文件配置两个方面开始介绍。

## @Profile

`@Profile` 注解使用在类和方法上，其 `value` 参数用于指定可以使用的环境有哪些，如下示例：

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @Profile({"dev", "test"})
    public DataSource devDataSource() {
        ...
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        ...
    }
}
```

对于实际使用那个环境，有以下几种激活方式：

1. 配置属性指定。使用 `spring.profiles.active` ，若未配置使用 `spring.profiles.default` 。
2. 在web.xml中配置：
```xml
<context-param>
	<param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
<servlet>
    <servlet-name>zszxzServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>spring.profiles.default</param-name>
            <param-value>dev</param-value>
        </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>zszxzServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
3. 使用 `@ActiveProfiles` 注解。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = DataSourceConfig.class)
@ActiveProfiles("dev")
public class ProfileTest {

    @Autowired
    private DataSource dataSource;

    @Test
    public void sheetTest(){
        ...
    }
}
```

## 多资源配置



@Transactional

@Value

@ConfigurationProperties

@PropertySource

# 参考资料