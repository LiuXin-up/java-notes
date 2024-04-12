# Spring常用注解

## 增删查改常用注解

### @Controller、@RestController

> 在controller层类上使用，表示这是一个控制层。
>
> @RestController包含@Controller与@ResponseBody；在处理请求时，会自动将返回值转为json格式

### @Service、@Mapper

> @Service表示这是一个服务层
>
> @Mapper表示这是一个Dao层接口

### @ResponseBody

> 将返回值自动转为json格式

### @PostMapping、@GetMapping、@PutMapping、@DeleteMapping

> @PostMapping：表示这是一个post请求，常用于添加操作
>
> @GetMapping：表示这是一个Get请求，常用于查询操作
>
> @PutMapping：表示这是一个Put请求，常用于更新操作
>
> @DeleteMapping：表示这是一个Delete请求，常用于删除操作



## 事务

### @Transactional

> spring 的事务注解，使用`@Transactional`表示开启spring 事务，事务中抛出异常，则对整个事务进行回滚
>
> `@Transactional`可以用于类、方法、接口。一般情况下只用于public 方法上，private 等其他方法使用注解，会被忽略，不会抛出异常

#### 用法示例：

```java
 	/**
     * 新增
     */
	@Transactional(rollbackFor = Exception.class)
    public void insertData(BulletinBoardPO po) {
        po.setCreateTime(new Date());
        this.poDao.insert(po);
    }
```

#### @Transactional的属性

| 属性                   | 类型                               | 描述                                   |
| :--------------------- | ---------------------------------- | -------------------------------------- |
| value                  | String                             | 可选的限定描述符，指定使用的事务管理器 |
| propagation            | enum: Propagation                  | 可选的事务传播行为设置                 |
| isolation              | enum: Isolation                    | 可选的事务隔离级别设置                 |
| readOnly               | boolean                            | 读写或只读事务，默认读写               |
| timeout                | int (in seconds granularity)       | 事务超时时间设置                       |
| **rollbackFor**        | Class对象数组，必须继承自Throwable | **最常用**，导致事务回滚的异常类数组   |
| rollbackForClassName   | 类名数组，必须继承自Throwable      | 导致事务回滚的异常类名字数组           |
| noRollbackFor          | Class对象数组，必须继承自Throwable | 不会导致事务回滚的异常类数组           |
| noRollbackForClassName | 类名数组，必须继承自Throwable      | 不会导致事务回滚的异常类名字数组       |

## 缓存

### @Cacheable、@CacheEvict

> **@Cacheable**：方法的返回值存储在指定的缓存中。后续使用相同参数调用此方法将从缓存中获取结果，而不是执行该方法
>
> **@CacheEvict**：删除指定缓存

#### @Cacheable

参数：

- `value`：存储结果的缓存的名称，必填属性。
- `key`：指定缓存的键，用于动态计算密钥的 SpEL 表达式，必填属性。
- `condition`：指定缓存的条件，可以使用SpEL 表达式。

用法示例：

①需要在启动类加上 **@EnableCaching** 注解，表示开启基于注解的缓存

②在方法上使用 **@Cacheable** 注解，使用注解缓存。

③key可以使用变量key，例如`key = "#id"`，#XX与传入参数名相同；也可以使用常量key，例如：`key="'onlyOne'"`

```java
/**
 * 根据id获取数据
 */
@Cacheable(value = "CACHE_USER", key = "#id")
public BulletinBoardPO index(Long id) {
    UserPO user = this.poDao.getById(id);
    return user;
}
```

#### @CacheEvict

参数：

- `value`：指定缓存名称，必填属性。
- `key`：指定缓存的键，可以使用SpEL表达式，必填属性。
- `allEntries`：是否删除所有缓存，默认为false。
- `beforeInvocation`：在方法调用前删除缓存，默认为false，表示在方法调用后删除缓存。

用法示例：

```java
  	@CacheEvict(value = "user", key = "#id")
    public void updateUser(Long id, User user) {
        // 更新数据库或其他操作
    }
```



## 微服务调用

### @EnableFeignClients、@FeignClient

`@EnableFeignClients`注解的作用是**启用Feign客户端**，在springboot启动类上使用。

示例：

```java
@SpringBootApplication
@EnableFeignClients
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(ErpApplication.class, args);
    }
}
```

`@FeignClient`注解的作用是**声明一个Feign客户端**。在远程接口上使用。

示例：

```java
@FeignClient(name = "RemoteService", url = "ip:端口/xxx", configuration = FeignConfig.class)
public interface RemoteService {
    @GetMapping("/getById")
    UserPO getById(@RequestParam("id") Long id);
}
```

常用参数解释：

- **value**：必须属性，用于指定远程服务的名称。当使用服务发现机制（如Nacos）时，可以省略该属性，因为Feign会自动解析服务名称。
- **url**：指定远程服务的URL。
- **configuration**：用于指定一个配置类，该配置类可以包含Feign客户端的自定义配置。
- **fallback**和**fallbackFactory**：用于指定熔断时的回调接口或工厂类，当远程服务不可用或响应超时时，会调用这些回调。



## 异步

### @EnableAsync、@Async

`@EnableAsync`注解的作用是**启动异步执行功能**，在springboot启动类上使用。

`@Async`注解的作用是**将方法标记为异步执行**，在方法上使用。

示例：

```java
	@Async
    public void asyncMethod() {
        // 异步执行的任务逻辑
        System.out.println("异步方法开始执行");
        // 模拟耗时操作
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("异步方法执行完成");
    }
```



## 全局处理

### @ControllerAdvice

`@ControllerAdvice：`的增强版@Controller；用于全局处理、声明全局性（例如：声明全局异常、全局数据处理、请求参数预处理）。

### @ExceptionHandler（全局处理使用异常）

`@ExceptionHandler`注解标注的方法：用于捕获Controller中抛出的不同类型的异常，进行异常全局处理。

示例：

```Java
@ControllerAdvice
@ResponseBody
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ExceptionControllerAdvice {
 
    @ExceptionHandler(BaseException.class)
    public ResultVO<Object> baseException(BaseException ex) {
        return new ResultVO<>(ex.getCode(), ex.getMsg());
    }
 
    @ExceptionHandler({Exception.class})
    public ResultVO<Object> exception(Exception ex, HttpServletRequest request) {
        this.log.error("未处理到异常", ex);
        return getErrorResult(EnumResultMsg.SYSTEM_ERROR, (String)null);
    }
}
```

```java
@RestController
@RequestMapping("/test")
public class TestController {
 
    @GetMapping("/exception")
    public void testException() {
        throw new BusinessException("异常测试");
    }
}
```

`BusinessException`继承自`BaseException`，业务逻辑中抛出异常，最终会走到`ExceptionControllerAdvice`中的`baseException`方法中，最终封装成`ResultVO`对象返回给前台

### @ModelAttribute（可用于全局数据处理）

`@ModelAttribute`注解的作用是将请求参数绑定到指定的模型对象上，此方法会在执行目标Controller方法之前执行

示例：将参数绑定到model中，通过model的asMap方法获取到数据

```java

/**
 * 全局数据处理类
 *
 * @Author 刘新
 * @Date 2024/4/7 10:29
 */
@ControllerAdvice
public class GlobalDataConfig {

    /**
     * 全局数据处理示例：增加一个userInfo数据
     * ModelAttribute：使用注解
     *
     * @return
     */
    @ModelAttribute(value = "info")
    public Map<String, String> userInfo() {
        HashMap<String, String> map = new HashMap<>();
        map.put("username", "张三");
        map.put("gender", "男");
        return map;
    }
}
```

```java
    @GetMapping("/getInfo")
    @ResponseBody
    public void hello(Model model) {
        Map<String, Object> map = model.asMap();
        for (String s : map.keySet()) {
            System.out.println("key:" + s + ",value:" + map.get(s));
        }
    }
```

输出结果：

```bash
key:info,value:{gender=男, username=张三}
```



## @InitBinder（请求参数预处理）

`@InitBinder`作用于一个controller类，用于对请求参数进行预处理，如：参数类型转换、参数绑定等操作

示例：

```java
/**
 * 全局处理示例类
 *
 * @Author 刘新
 * @Date 2024/4/7 10:31
 */
@RestController
@RequestMapping("global")
public class GlobalController {

    /**
     * 参数预处理（接口调用之前生效）
     * @param binder
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // 设置需要预处理的属性，这里以日期类型为例
        binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }

    /**
     * 参数类型转换
     * 当接口传参为string类型的"2024-01-01“时
     * 如果不使用InitBinder进行参数预处理，会提示类型错误
     *
     * @param date
     * @return
     */
    @RequestMapping("/test")
    public String test(Date date) {
        return "Received date: " + date;
    }
}
```

