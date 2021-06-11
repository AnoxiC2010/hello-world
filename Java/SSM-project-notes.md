

# 跨域配置

```java
 * Function:ajax跨域请求配置
 */
@Configuration
public class CorsConfig {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 1 设置访问源地址
        corsConfiguration.addAllowedHeader("*"); // 2 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 3 设置访问源请求方法
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 4 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```



# PageHelper



# Mybatis-Generator

逆向工程

# TypeHandler

## Integer[] <=> String

```java
@Data
public class Admin {
	    private Integer[] roleIds;//配置类型转换器
    //数据库 [1,2,4]
}
```

```java
package com.cskaoyan.typehandler;

import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.SneakyThrows;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;
import org.apache.ibatis.type.TypeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@MappedTypes(Integer[].class)
@MappedJdbcTypes(JdbcType.VARCHAR)
public class IntegerArrayTypeHandler implements TypeHandler<Integer[]> {

    //Json
    ObjectMapper objectMapper = new ObjectMapper();

    //输入映射 Integer[] → String
    //index:输入映射过程中，预编译 占位符的顺序
    @SneakyThrows
    @Override
    public void setParameter(PreparedStatement preparedStatement, int index, Integer[] integers, JdbcType jdbcType) throws SQLException {
        String value = objectMapper.writeValueAsString(integers);
        preparedStatement.setString(index,value);
    }
    //输出映射 String → Integer[]
    @Override
    public Integer[] getResult(ResultSet resultSet, String columnName) throws SQLException {
        String result = resultSet.getString(columnName);
        return transfer(result);
    }

    @Override
    public Integer[] getResult(ResultSet resultSet, int index) throws SQLException {
        String result = resultSet.getString(index);
        return transfer(result);
    }

    @Override
    public Integer[] getResult(CallableStatement callableStatement, int index) throws SQLException {
        String result = callableStatement.getString(index);
        return transfer(result);
    }

    @SneakyThrows
    private Integer[] transfer(String result) {
        if (result == null || "".equals(result)) {
            return new Integer[0];
        }
        Integer[] integers = objectMapper.readValue(result, Integer[].class);
        return integers;
    }
}

```

```yaml
# application.yml
# jdbc类型转换器
mybatis:
  type-handlers-package: com.cskaoyan.typehandler
```

## String[] <=> String

```java
@MappedTypes(String[].class)
@MappedJdbcTypes(JdbcType.VARCHAR)
public class StringArrayTypeHandler implements TypeHandler<String[]> {

    //Json
    ObjectMapper objectMapper = new ObjectMapper();

    //输入映射 String[] → String
    //index:输入映射过程中，预编译 占位符的顺序
    @SneakyThrows
    @Override
    public void setParameter(PreparedStatement preparedStatement, int index, String[] strings, JdbcType jdbcType) throws SQLException {
        String value = objectMapper.writeValueAsString(strings);
        preparedStatement.setString(index,value);
    }
    //输出映射 String → String[]
    @Override
    public String[] getResult(ResultSet resultSet, String columnName) throws SQLException {
        String result = resultSet.getString(columnName);
        return transfer(result);
    }

    @Override
    public String[] getResult(ResultSet resultSet, int index) throws SQLException {
        String result = resultSet.getString(index);
        return transfer(result);
    }

    @Override
    public String[] getResult(CallableStatement callableStatement, int index) throws SQLException {
        String result = callableStatement.getString(index);
        return transfer(result);
    }

    @SneakyThrows
    private String[] transfer(String result) {
        if (result == null || "".equals(result)) {
            return new String[0];
        }
        String[] strings = objectMapper.readValue(result, String[].class);
        return strings;
    }
}
```

配置同Integer[]类型转换

# BeanUtils转换bean

查查询的pojo了和前端需要的参数不一致

org.springframework.beans.BeanUtils

转换需要属性名和类型对应

```java
BeanUtils.copyProperties(bo, admin);
```



# 文件上传

[Spring Boot应用上传文件时报错 - nuccch - 博客园 (cnblogs.com)](https://www.cnblogs.com/nuccch/p/11546494.html)

```yaml
# application.yml
# 文件上传配置
  servlet:
    multipart:
      # 启用
      enabled: true
      max-file-size: 1MB
      # 上传是产生的临时文件目录 位于 例如/tmp/tomcat.6574404581312272268.18333/work/Tomcat/localhost/ROOT + /temp
      location: /temp
# 静态资源映射
  web:
    resources:
      # 映射文件夹
      static-locations: file:${mall.storage-base-path}
  mvc:
    # 映射url
    static-path-pattern: ${mall.static-path-pattern-prefix}*
# 文件本地存储路径 个人可按照自己需要修改(注释人只有C盘)
mall:
  # 域名
  domain: http://localhost:8083
  # 图片url映射前缀
  static-path-pattern-prefix: /wx/storage/fetch/
  # 图片本地存储路径
  storage-base-path: c:/D/mall/img
```



# 密码散列(注解+切面)

```java
/**
 * 注解
 * 添加到需要散列密码的方法上
 * 检查参数里的psssword属性
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface HashPasswordAnno {

}
```



```java
@Aspect
@Component
public class HashPasswordAspect {
    //切点
    @Pointcut("@annotation(com.cskaoyan.anno.HashPasswordAnno)")
    public void hashPasswordPointCut() {
    }

    /**
     * 在执行Handler方法之前，检查参数中名为password的属性，并散列
     * 在需要散列密码的Handler方法上加 注解 @HashPasswordAnno 即可
     * 散列方法位于util.StringUtils.sha250(string raw)
     * 可以按需要修改散列方法的内部实现
     * @param joinPoint
     */
    @Before("hashPasswordPointCut()")
    public void hashPasswordBeforeController(JoinPoint joinPoint) {
        //获得方法参数
        Object[] args = joinPoint.getArgs();
        //检查password（如果有的话）属性并散列
        try {
            for (Object arg : args) {
                Class<?> aClass = arg.getClass();
                Field passwordField = aClass.getDeclaredField("password");
                if (passwordField != null) {
                    passwordField.setAccessible(true);
                    String rawPassword = (String) passwordField.get(arg);
                    String hashPassword = StringUtils.sha250(rawPassword);
                    passwordField.set(arg, hashPassword);
                    break;
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }

    }
}
```



# 统一参数校验响应（切面）

```java
/**
 * HJL
 * 参数检验失败响应切面
 * 全面接管所有参数校验失败的Handler方法的响应
 * 不需要在每个需要参数校验的Handler方法内都做失败响应。
 *
 */
@Aspect
@Component
public class ParamValidAspect {
    //切点
    @Pointcut("execution(* com.cskaoyan.controller.*.*(..,org.springframework.validation.BindingResult,..))")
    public void paramValidPointcut() {
    }

    /**
     * 检查参数包含BindingResult的方法，接管校验失败的响应
     * @param joinPoint
     * @return
     */
    @Around("paramValidPointcut()")
    public Object paramValidCheck(ProceedingJoinPoint joinPoint) throws Throwable {
        //校验失败响应
        Object[] args = joinPoint.getArgs();
        for (Object arg : args) {
            if (arg instanceof BindingResult) {
                BindingResult bindingResult = (BindingResult) arg;
                if (bindingResult.hasFieldErrors()) {
                    return ValidateUtils.valid(bindingResult);
                }
            }
        }
        //校验通过，正常响应
        return joinPoint.proceed();
    }


}
```

# 异常响应 主要用于事务



# 操作日志记录（切面）

HandlerInterceptor

`/**`



# 静态资源映射配置

springBoot不能像在配置类中加addResourceHandler一样配置多组静态资源映射url和静态资源映射路径

```java
//例如这样可以注册多组
@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/aligenie/**").addResourceLocations("classpath:/aligenie/");
       //在这里注册多组
    }

```

application.yml这里的location和pattern不能配置多组，只有location能配置多个

```yaml
spring:
# 文件上传配置
  servlet:
    multipart:
      # 启用
      enabled: true
      max-file-size: 1MB
      # 上传是产生的临时文件目录 位于 例如/tmp/tomcat.6574404581312272268.18333/work/Tomcat/localhost/ROOT + /temp
      location: /temp
# 静态资源映射
  web:
    resources:
      # 映射文件夹
      static-locations: file:${mall.storage-base-path}
  mvc:
    # 映射url
    static-path-pattern: ${mall.static-path-pattern-prefix}*
```



# shiro权限管理配置





# 事务咨询老师

为解决的事务 死锁

事务的propagation



一个方法内 几个修改语句是那种传播

一个方法内 调用另一个方法 两个方法都加了事务注解 那种传播

```java
@Transactional
public Object delete(LitemallGoods goods) {
    Integer id = goods.getId();
    if (id == null) {
        return ResponseUtil.badArgument();
    }

    Integer gid = goods.getId();
    goodsService.deleteById(gid);
    specificationService.deleteByGid(gid);
    attributeService.deleteByGid(gid);
    productService.deleteByGid(gid);
    return ResponseUtil.ok();
}
```



# Redis 存储固定数据

```xml
<!--
pom.xml 
redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



```yaml
spring:
# redis配置 这些是瞎配的
  redis:
    host: localhost
    port: 6379
    timeout: 10000
    lettuce:
      pool:
        max-active: 8
        min-idle: 10
        max-idle: 30
        max-wait: -1ms
# 日志
logging:
  level:
    com.cskaoyan.mapper: debug
    # 不配这个日志级别 眼睛都被lettuce的重连晃瞎了
    io.lettuce.core.protocol: error
```

不配lettuce的日志级别会反复报重连错误，没几秒一次

```
io.lettuce.core.protocol.CommandHandler connection was forcibly closed by the remote host
```

```java
//加载需要返回固定配置数据的service方法上
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RedisCache {
}
```

```java
@Aspect
@Component
public class RedisCacheAspect {
    private static final Logger logger = Logger.getGlobal();
    @Autowired
    private RedisService redisService;

    @Pointcut("@annotation(com.cskaoyan.anno.RedisCache)")
    public void requireRedisCache() {
    }

    @Around("requireRedisCache()")
    public Object redisCacheSupport(ProceedingJoinPoint joinPoint) throws Throwable {
        //从缓存取
        HttpServletRequest request = HttpUtils.getHttpRequest();
        String methodName = joinPoint.getSignature().getName();
        String permission = StringUtils.uriToPermission(request.getRequestURI());
        String key = permission + ":" + methodName;//key 格式 admin:role:permission:service方法名

        Object value = redisService.get(key);
        if (value != null) {
            logger.info("使用缓存 " + key);
            return value;
        }

        //从数据库取
        Object result = joinPoint.proceed();

        //放入缓存
        redisService.set(key, result);
        logger.info("使用缓存失败，从数据库获取 " + key);

        return result;
    }

}

```

```java
@Service
public class RedisServiceImpl implements RedisService {

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void set(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }

    @Override
    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }
}
```



# JsonFromat

```
@JsonInclude(JsonInclude.Include.NON_NULL) //忽略掉null的成员变量
```

时间格式通用配置

```yaml
# application.yml
spring:
  # jackson时间通用配置
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```



# 任意处获取HttpServletRequest

[SpringBoot在AOP中获取HttpServletRequest信息_Mr.Xie的博客-CSDN博客](https://blog.csdn.net/lihua5419/article/details/98479793)

 AOP中获取HttpServletRequest信息

```java
//获取当前登录人信息
//Subject subject = SecurityUtils.getSubject();
//SysUser user = (SysUser)subject.getPrincipal();
//获取RequestAttributes
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
//从获取RequestAttributes中获取HttpServletRequest的信息
HttpServletRequest request = (HttpServletRequest) requestAttributes.resolveReference(RequestAttributes.REFERENCE_REQUEST);
String token = request.getHeader("token");
Long userid= jwtUtil.parseToken(token).getId();
SysUser user=userService.selectUserById(userid);
```

 AOP中获取HttpSession信息

```java
HttpSession session = (HttpSession) requestAttributes.resolveReference(RequestAttributes.REFERENCE_SESSION);
```







# 短信服务



aliyun文本短信 签名: stone4j

模板code: SMS_173765187



# oos

bucket: cskaoyan

endPoint: oos-cn-beijing.aliyuncs.com 

# 疑问咨询老师

1. 是否每个方法都要判空 判属性空 controller service...



# bug

public int deleteAdmin(DeleteAdminBO bo) {

两个update语句存在时

 Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction





系统管理演示摘要

文件上传： 阿里云

- 异常控制通知

  service层抛SQL异常主动回滚事务。同时异常message响应为BasResVo并写入日志

- 切面

  - md5密码散列切面+注解配置（没法演示）

    被注解的方法进入时参数里的密码就已经被md5散列值取代

  - 统一全局参数校验响应切面（修改管理员密码演示 长度限制6-9）

    检测所有包含(BindingResult)参数，在进入hanler方法前返回错误信息，并录入日志

  - 管理员日志记录切面（需要在演示 修改密码参数错误 和 liming@qq.com最为用户名时抛异常 后看日志）

    用admin:order:list的格式记录所有 result=BasRespVo.msg , comment = 异常信息

  - Redis缓存切面+注解（注解加载查询系统固定配置的方法上，优先从缓存取）

    有从redis取，没有库里查，在存如redis

    如admin/admin/permissions返回数据库系统配置

- Redis

  1. 注解缓存固定配置数据
  2. 短信验证码利用redis的set来存储和保证是最新的





1. 管理员
2. 操作日志
3. 角色管理
4. 对象存储
5. 