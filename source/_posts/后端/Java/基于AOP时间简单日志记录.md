---
title: 基于AOP实现简单日志记录
date: 2024-06-14 10:08:40
tags: [ Aop , Log]
categories: [ 后端 , Java , Spring]

---





```java

package org.example.springaoplog.log;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import java.util.Date;

/**
 * 日志切面
 * @author 14237
 */
@Component
@Aspect
public class LogAspect {
    @Pointcut ( "@annotation(org.example.springaoplog.log.Log)")
    public void pointcut() {}

    @Resource
    HttpServletRequest request;
    final ObjectMapper mapper = new ObjectMapper();

    @Around ( "pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws JsonProcessingException {
        final MethodSignature signature = ((MethodSignature) pjp.getSignature());
        final Log annotation = signature.getMethod().getAnnotation(Log.class);
        final StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        String errorMsg = "";
        final Object proceed;
        String result = "";
        try {
            proceed = pjp.proceed();
            result = mapper.writeValueAsString(proceed);//返回结果
            return proceed;
        } catch(Throwable e) {
            errorMsg = e.getLocalizedMessage();
            throw new RuntimeException(e);
        } finally {
            stopWatch.stop();

            final SysLog sysLog = new SysLog()
                    .setId(null)
                    .setRequestMethod(request.getMethod())
                    .setName(annotation.value())
                    .setType(annotation.type().getValue())
                    .setOperator("sessionId: " + request.getSession().getId())
                    .setOperatingTime(new Date())
                    .setIp(request.getRemoteHost())
                    .setUrl(request.getRequestURL().toString())
                    .setMethod(signature.toString())
                    .setArgs(mapper.writeValueAsString(pjp.getArgs()))
                    .setMillisecond(stopWatch.getTotalTimeMillis())
                    .setResult(result)
                    .setError(errorMsg);
            //记录日志
            recordLog(sysLog);
        }

    }

    /**
     * 记录日志
     * @param log 日志
     */
    public void recordLog(SysLog log) {
        //保存到数据库
        System.err.println(log);
    }
}
```

```java
package org.example.springaoplog.log;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention ( RetentionPolicy.RUNTIME)
public @interface Log {
    /**
     * 日志类型
     * @return {@link LogType }
     */
    LogType type() default LogType.DEFAULT;

    /**
     * 日志名称
     * @return {@link String }
     */
    String value() default "";
}
	
```

```java
package org.example.springaoplog.log;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public enum LogType {
    DEFAULT("默认"),
    SYSTEM("系统日志"),
    BUSINESS("业务日志"),
    OPERATE("操作日志"),
    TEST("测试日志");
    final String value;
}

```

```java
package org.example.springaoplog.log;

import lombok.Data;
import lombok.experimental.Accessors;

import java.util.Date;

/**
 * 系统日志
 * @author inyxin
 */
@Data
@Accessors ( chain = true)
public class SysLog {
    /** 日志ID */
    private Integer id;
    /** 日志名字 */
    private String name;
    /** 日志类型 */
    private String type;
    /** 请求路径 */
    private String url;
    /** 请求方式 */
    private String requestMethod;
    /** 操作者 */
    private String operator;
    /** 方法 */
    private String method;
    /** 参数 */
    private String args;
    /** 远程IP */
    private String ip;
    /**
     * 操作时间
     */
    private Date operatingTime;
    /** 异常信息 */
    private String error;
    /** 结果 */
    private String result;
    /** 方法执行时间 毫秒 */
    private long millisecond;

}

```

测试:

```java
/*
 * Copyright 2013-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.example.springaoplog.demos.web;

import org.example.springaoplog.log.Log;
import org.example.springaoplog.log.LogType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * @author <a href="mailto:chenxilzx1@gmail.com">theonefx</a>
 */
@Controller
public class BasicController {

    // http://127.0.0.1:8080/hello?name=lisi
    @RequestMapping ( "/hello")
    @ResponseBody
    @Log (
            value = "Hello",
            type = LogType.TEST
    )
    public String hello(
            @RequestParam (
                    name = "name",
                    defaultValue = "unknown user"
            ) String name) {
        return "Hello " + name;
    }

    // http://127.0.0.1:8080/errortest
    @RequestMapping ( "/errortest")
    @ResponseBody
    @Log (
            value = "异常测试",
            type = LogType.TEST
    )
    public String errortest() {
        System.out.println(1 / 0);
        return "12321";
    }

    // http://127.0.0.1:8080/user
    @RequestMapping ( "/user")
    @ResponseBody
    @Log ( value = "查询用户")
    public User user() {
        User user = new User();
        user.setName("theonefx");
        user.setAge(666);
        return user;
    }


    // http://127.0.0.1:8080/save_user?name=newName&age=11
    @RequestMapping ( "/save_user")
    @ResponseBody
    @Log (
            value = "保存用户",
            type = LogType.OPERATE
    )
    public String saveUser(User u) {
        return "user will save: name=" + u.getName() + ", age=" + u.getAge();
    }

    // http://127.0.0.1:8080/html
    @RequestMapping ( "/html")
    public String html() {
        return "index.html";
    }

    @ModelAttribute
    public void parseUser(
            @RequestParam (
                    name = "name",
                    defaultValue = "unknown user"
            ) String name
            ,
            @RequestParam (
                    name = "age",
                    defaultValue = "12"
            ) Integer age, User user) {
        user.setName("zhangsan");
        user.setAge(18);
    }
}

```

输出:

```java
SysLog(id=null, name=查询用户, type=默认, url=http://127.0.0.1:8080/user, requestMethod=GET, operator=sessionId: 2F552A00DA410EEE3047B4AEE36927B1, method=User org.example.springaoplog.demos.web.BasicController.user(), args=[], ip=127.0.0.1, operatingTime=Fri Jun 14 10:02:11 CST 2024, error=, result={"name":"theonefx","age":666}, millisecond=7)
SysLog(id=null, name=查询用户, type=默认, url=http://10.120.1.193:8080/user, requestMethod=GET, operator=sessionId: 39EA319EFCC022E1973A1C9DA452DAE2, method=User org.example.springaoplog.demos.web.BasicController.user(), args=[], ip=10.120.1.193, operatingTime=Fri Jun 14 10:06:46 CST 2024, error=, result={"name":"theonefx","age":666}, millisecond=0)

```

