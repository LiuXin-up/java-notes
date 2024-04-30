# AOP与自定义注解

## 题目描述

spring boot实现两个注解，要求如下：

1. 注解一用于对象的参数，作用为：是否对参数开启拼接，如果开启，则获取到参数值，并通过注解二拼接成字符串。
2. 注解二用于接口上，作用为：获取到注解一的参数值，并将其拼接成字符串。

## 思路

1. 注解一作用于入参上，注解二作用于方法上
2. 获取参数值需要使用拦截器或aop实现

## 作用

1. 可配合Redis等分布式锁实现接口防重放功能
2. 可对参数进行二次加工，实现入参及进行md5加密等操作

## 代码实现

`@ParamConcat`自定义注解实现代码如下，注解作用于参数上

```java
package code.spring.service.concat;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 自定义参数注解
 * 配合aop切面，实现参数获取
 *
 * @author lx201
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface ParamConcat {

    /**
     * 默认不开启参数获取
     *
     * @return
     */
    boolean value() default false;

}

```

`@ConcatString`自定义注解实现代码如下，注解作用于方法上

```java
package code.spring.service.concat;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 自定义方法注解
 * 配合aop切面，实现参数拼接
 *
 * @author lx201
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ConcatString {

    /**
     * 默认参数值为空字符串
     *
     * @return
     */
    String value() default "";
}
```

使用aop进行参数获取及拼接，代码实现如下

```java
package code.spring.service.concat;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Parameter;

/**
 * aop切面参数操作类
 * 配合@ParamConcat注解、@ConcatString注解使用
 * 组合使用后，可实现参数的获取拼接等功能
 * aop切面在方法执行前生效
 *
 * @author lx201
 */
@Aspect
@Component
public class ParamConcatInterceptor {

    /**
     * aop切面方法
     * 作用于 @ConcatString注解
     *
     * @param joinPoint    连接点(获取到的参数值)
     * @param concatString 拦截注解(自定义方法注解以及注解的参数)
     * @return 返回结果
     * @throws Throwable 异常
     */
    @Around("@annotation(concatString)")
    public Object concatString(ProceedingJoinPoint joinPoint, ConcatString concatString) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        StringBuilder result = new StringBuilder();

        Parameter[] parameters = signature.getMethod().getParameters();
        Object[] args = joinPoint.getArgs();
        for (int i = 0; i < parameters.length; i++) {
            Parameter parameter = parameters[i];
            // 判断参数是否使用了@ParamConcat注解
            if (parameter.isAnnotationPresent(ParamConcat.class)) {
                // 直接使用循环变量i作为索引
                Object value = args[i];
                if (value != null) {
                    // 这里简化为直接拼接，可以根据需要添加分隔符等
                    result.append(value);
                    result.append(concatString.value());
                }
            }
        }
        // 这里可以打印结果或将其用于其他逻辑
        System.out.println("获取到的参数值: " + result);

        // 继续执行原方法
        return joinPoint.proceed();
    }
}
```

测试接口

```java
package code.spring.service.concat;

import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * aop与自定义注解示例类
 *
 * @author liuxin
 * @date 2024/04/30
 */
@RestController("/aopDemo")
public class AopController {

    /**
     * aop测试接口
     * ConcatString 注解为自定义注解，aop切面作用于该注解，将参数拼接成字符串返回，拼接的字符串为：name#age
     *
     * @param name
     * @param age
     * @return
     */
    @ApiOperation(value = "aop切面与自定义注解示例", notes = "aop切面与自定义注解示例")
    @GetMapping("/aop")
    @ConcatString("#")
    public String demo(@ParamConcat(value = true) @RequestParam String name, 
                       @ParamConcat(value = true) @RequestParam int age) {
        return "success";
    }
}
```