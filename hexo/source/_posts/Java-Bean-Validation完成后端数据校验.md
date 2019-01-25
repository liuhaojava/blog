---
title: Java Bean Validation完成后端数据校验
copyright: true
date: 2019-01-24 16:04:09
tags: Validation完成后端数据校验
categories: Java
---

# 前言
　　数据的校验是交互式网站一个不可或缺的功能，前端的js校验可以涵盖大部分的校验职责，如用户名唯一性，生日格式，邮箱格式校验等等常用的
校验。但是为了避免用户绕过浏览器，使用http工具直接向后端请求一些违法数据，服务端的数据校验也是必要的，可以防止脏数据落到数据库中，如
果数据库中出现一个非法的邮箱格式，也会让运维人员头疼不已。我在之前保险产品研发过程中，系统对数据校验要求比较严格且追求可变性及效率，
曾使用drools作为规则引擎，兼任了校验的功能。而在一般的应用，可以使用本文将要介绍的validation来对数据进行校验。
## JSR303/JSR-349
　　简述JSR303/JSR-349，Hibernate Validation，Spring Validation之间的关系。JSR303是一项标准,JSR-349是其的升级版本，添加了一些
新特性，他们规定一些校验规范即校验注解，如@Null，@NotNull，@Pattern，他们位于javax.validation.constraints包下，只提供规范不提供
实现。而hibernate validation是对这个规范的实践（不要将hibernate和数据库orm框架联系在一起），他提供了相应的实现，并增加了一些其他校
验注解，如@Email，@Length，@Range等等，他们位于org.hibernate.validator.constraints包下。而万能的Spring为了给开发者提供便捷，对
Hibernate Validation进行了二次封装，显示校验validated bean时，你可以使用Spring Validation或者Hibernate Validation，而Spring 
validation另一个特性，便是其在SpringMVC模块中添加了自动校验，并将校验信息封装进了特定的类中。这无疑便捷了我们的web开发。本文主要介
绍在SpringMVC中自动校验的机制。注解如下：
### JSR提供的校验注解：
| 注解        | 注释   |
| --------   | ------  |
|@Null | 被注释的元素必须为 null|
|@NotNull | 被注释的元素必须不为 null|
|@AssertTrue | 被注释的元素必须为 true|
|@AssertFalse | 被注释的元素必须为 false|
|@Min(value) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值|
|@Max(value) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值|
|@DecimalMin(value) | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值|
|@DecimalMax(value) | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值|
|@Size(max=, min=) | 被注释的元素的大小必须在指定的范围内|
|@Digits (integer, fraction)  | 被注释的元素必须是一个数字，其值必须在可接受的范围内|
|@Past | 被注释的元素必须是一个过去的日期 |
|@Future | 被注释的元素必须是一个将来的日期|
|@Pattern(regex=,flag=) | 被注释的元素必须符合指定的正则表达式|


### Hibernate Validator提供的校验注解： 
| 注解        | 注释   |
| --------   | ------  | 
|@NotBlank(message =) | 验证字符串非null，且长度必须大于0|
|@Email | 被注释的元素必须是电子邮箱地址|
|@Length(min=,max=) | 被注释的字符串的大小必须在指定的范围内|
|@NotEmpty | 被注释的字符串的必须非空|
|@Range(min=,max=,message=) | 被注释的元素必须在合适的范围内|

# 代码实现
## 添加JAR包依赖
```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.7.Final</version>
    <!--<classifier>sources</classifier>-->
</dependency>
```
## 简单校验
### 1.在pojo中指定校验规则
```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Getter;
import lombok.Setter;
import org.hibernate.validator.constraints.Length;
import org.springframework.format.annotation.DateTimeFormat;

import javax.validation.constraints.*;
import java.util.Date;

@ApiModel
@Getter
@Setter
public class UserInfo {
    @ApiModelProperty(value = "姓名")
    @NotEmpty(message = "姓名不能为空！")
    @Max(value = 5, message = "姓名长度不能超过5！")
    private String name;
    @Length(max = 10, message = "昵称长度不能超过10！")
    @ApiModelProperty(value = "昵称")
    private String nickname;
    @Pattern(regexp = "[男|女]", message = "性别只能在男或女中选择！")
    @ApiModelProperty(value = "性别")
    private String sex;
    @Digits(integer = 18, fraction = 28, message = "年龄必须在18-28之间！")
    @ApiModelProperty(value = "年龄")
    private Integer age;
    @Past(message = "生日必须在过去的时间里！")
    @ApiModelProperty(value = "生日")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
    @ApiModelProperty(value = "籍贯")
    private String nativePlace;
    @Pattern(regexp = "^0\\d{2,3}-\\d{7,8}$", message = "固定电话格式不正确！")
    @ApiModelProperty(value = "固定电话")
    private String telephone;
    @Pattern(regexp = "^1\\d{10}$", message = "移动电话格式不正确！")
    @ApiModelProperty(value = "移动电话")
    private String phone;
    @Email(message = "邮箱格式不正确！")
    @ApiModelProperty(value = "邮箱")
    private String email;

    @Override
    public String toString() {
        return "UserInfo{" +
                "name='" + name + '\'' +
                ", nickname='" + nickname + '\'' +
                ", sex='" + sex + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                ", nativePlace='" + nativePlace + '\'' +
                ", telephone='" + telephone + '\'' +
                ", phone='" + phone + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```
### 2.controller中对其校验绑定进行使用
```java
import io.swagger.annotations.Api;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import qgs.csmp.zzz.UserInfo;
import qgs.framework.core.annotation.AuthPassport;
import qgs.framework.core.common.BaseController;

@Api(tags = "validation校验demo")
@Controller
@RequestMapping("/validation")
public class DemoController extends BaseController {

    @AuthPassport(value = false)
    @ResponseBody
    @RequestMapping(method = RequestMethod.GET)
    public String save(@Validated UserInfo userInfo, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            StringBuilder sb = new StringBuilder();
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                sb.append(fieldError.getDefaultMessage()).append(",\n");
            }
            return sb.toString();
        }
        return "success";
    }
}
```
- 注：
> 1、@Validated作用就是将pojo内的注解数据校验规则(@NotNull等)生效，如果没有该注解的声明，pojo内有注解数据校验规则也不会生效   
> 2、BindingResult对象用来获取校验失败的信息(@NotNull中的message)，与@Validated注解必须配对使用，一前一后  

## 对BindingResult统一异常拦截
　　统一异常拦截后，不必每次都对controller接口增加参数BindingResult。代码实现如下：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import qgs.framework.util.utilty.StringUtil;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Set;

@ControllerAdvice
public class ExceptionLogInterceptor {

    @SuppressWarnings("rawtypes")
    @ResponseBody
    @ExceptionHandler(Exception.class)
    public void resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception e) {
        Logger logger = LoggerFactory.getLogger(getClass());

        StringBuilder errorMsg = new StringBuilder();
        StringBuilder errorLog = new StringBuilder();
        if (e instanceof BindingResult || e instanceof MethodArgumentNotValidException ||
                e instanceof ConstraintViolationException) {
            BindingResult bindingResult = null;
            if (e instanceof BindingResult) {
                bindingResult = (BindingResult) e;
            }
            //实体类中包含其他实体
            if (e instanceof MethodArgumentNotValidException) {
                MethodArgumentNotValidException validException = (MethodArgumentNotValidException) e;
                bindingResult = validException.getBindingResult();
            }
            if (bindingResult != null && bindingResult.getAllErrors() != null && !bindingResult.getAllErrors().isEmpty()) {
                errorMsg = new StringBuilder(bindingResult.getAllErrors().get(0).getDefaultMessage());
            }
            if (e instanceof ConstraintViolationException) {
                Set<ConstraintViolation<?>> violations = ((ConstraintViolationException) e).getConstraintViolations();
                for (ConstraintViolation<?> violation : violations) {
                    errorMsg.append(violation.getMessage()).append(", ");
                }
            }
        } else {
            errorMsg.append(StringUtil.isNullOrEmpty(e.getMessage()) ? e.toString() : e.getMessage());
            errorLog.append(StringUtil.isNullOrEmpty(e.getMessage()) ? e.toString() : e.getMessage());
            errorLog.append("\r\n");
            for (StackTraceElement traceElement : e.getStackTrace()) {
                errorLog.append(traceElement.toString());
                errorLog.append("\r\n");
            }
        }
        logger.error(errorLog.toString());
        try {
            response.setStatus(500);
            response.setContentType("application/json; charset=utf-8");
            PrintWriter out = response.getWriter();
            out.append("{\"msg\":\"").append(errorMsg.toString()).append("\"}");
            out.close();
        } catch (IOException ignored) {
        }
    }

}
```
- 注：
> 这里只对一处不符合规则的错误信息输出

此时，controller代码可更改为：
```java
public class DemoController extends BaseController {
    @AuthPassport(value = false)
    @ResponseBody
    @RequestMapping(method = RequestMethod.GET)
    public String save(@Validated UserInfo userInfo) {
        return "success";
    }
}
```
或者
```java
public class DemoController extends BaseController {
    @AuthPassport(value = false)
    @ResponseBody
    @RequestMapping(method = RequestMethod.GET)
    public String save(@Valid UserInfo userInfo) {
        return "success";
    }
}
```
@Validated或者@Valid均可

## 分组校验
### 1.什么是分组校验？
　　校验规则是在pojo制定的，而同一个pojo可以被多个Controller使用，此时会有问题，即：不同的Controller方法对同一个pojo进行校验，此时
这些校验信息是共享在这不同的Controller方法中，但是实际上每个Controller方法可能需要不同的校验，在这种情况下，就需要使用分组校验来
解决这种问题，通俗的讲，一个pojo中有很多属性，controller中的方法1可能只需要校验pojo中的属性1，controller中的方法2只需要校验pojo中
的属性2，但是pojo中的校验注解有很多，怎样才能使方法1只校验属性1，方法二只校验属性2呢？就需要用分组校验来解决了。
### 2.定义分组
　　1.定义空的接口
```java
public interface ValidationGroup1 {
}
```
```java
public interface ValidationGroup2 {
}
```
　　2.修改pojo，注解增加参数groups
```java
public class UserInfo {
    @ApiModelProperty(value = "姓名")
    @NotEmpty(message = "姓名不能为空！", groups = ValidationGroup1.class)
    @Max(value = 5, message = "姓名长度不能超过5！")
    private String name;
    @Length(max = 10, message = "昵称长度不能超过10！", groups = ValidationGroup2.class)
    @ApiModelProperty(value = "昵称")
    private String nickname;
    @Pattern(regexp = "[男|女]", message = "性别只能在男或女中选择！")
    @ApiModelProperty(value = "性别")
    private String sex;
    @Digits(integer = 18, fraction = 28, message = "年龄必须在18-28之间！")
    @ApiModelProperty(value = "年龄")
    private Integer age;
    @Past(message = "生日必须在过去的时间里！")
    @ApiModelProperty(value = "生日")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
    @ApiModelProperty(value = "籍贯")
    private String nativePlace;
    @Pattern(regexp = "^0\\d{2,3}-\\d{7,8}$", message = "固定电话格式不正确！")
    @ApiModelProperty(value = "固定电话")
    private String telephone;
    @Pattern(regexp = "^1\\d{10}$", message = "移动电话格式不正确！")
    @ApiModelProperty(value = "移动电话")
    private String phone;
    @Email(message = "邮箱格式不正确！")
    @ApiModelProperty(value = "邮箱")
    private String email;

    @Override
    public String toString() {
        return "UserInfo{" +
                "name='" + name + '\'' +
                ", nickname='" + nickname + '\'' +
                ", sex='" + sex + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                ", nativePlace='" + nativePlace + '\'' +
                ", telephone='" + telephone + '\'' +
                ", phone='" + phone + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```
　　2.修改controller
```java
public class DemoController extends BaseController {
    @AuthPassport(value = false)
    @ResponseBody
    @RequestMapping(method = RequestMethod.GET)
    public String save(@Validated(value = {ValidationGroup1.class}) UserInfo userInfo) {
        return "success";
    }
}
```
- 注：
> 此时只能使用@Validated注解    
> 如上，只校验pojo中groups为ValidationGroup1的属性，如name有两处校验，只会校验是否为空而不会校验长度是否大于5

## 自定义注解校验
　　业务需求总是比框架提供的这些简单校验要复杂的多，我们可以自定义校验来满足我们的需求。自定义Spring Validation非常简单，主要分为两步。
　　1.自定义校验注解
我们尝试添加一个“字符串不能包含空格”的限制。
```java
import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {CannotHaveBlankValidator.class})

public @interface CannotHaveBlank {
    
    String message() default "不能包含空格";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};

    //指定多个时使用
    @Target({FIELD, METHOD, PARAMETER, ANNOTATION_TYPE})
    @Retention(RUNTIME)
    @Documented
    @interface List {
        CannotHaveBlank[] value();
    }
}
```
　　2 编写校验类
```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class CannotHaveBlankValidator implements ConstraintValidator<CannotHaveBlank, String> {

    @Override
    public void initialize(CannotHaveBlank constraintAnnotation) {
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        //null时不进行校验
        if (value != null && value.contains(" ")) {
            //获取默认提示信息
            String defaultConstraintMessageTemplate = context.getDefaultConstraintMessageTemplate();
            System.out.println("default message :" + defaultConstraintMessageTemplate);
            //禁用默认提示信息
            context.disableDefaultConstraintViolation();
            //设置提示语
            context.buildConstraintViolationWithTemplate("can not contains blank").addConstraintViolation();
            return false;
        }
        return true;
    }
}
```
- 注:
> 所有的验证者都需要实现ConstraintValidator接口，它的接口包含一个初始化事件方法，和一个判断是否合法的方法。    
> ConstraintValidatorContext 这个上下文包含了认证中所有的信息，我们可以利用这个上下文实现获取默认错误提示信息，禁用错误提示信息，
改写错误提示信息等操作。 
<div style="display:none;">
## 基于方法校验(controller层方法中单个参数校验)

```java
@Api(tags = "validation校验demo")
@Controller
@RequestMapping("/validation")
@Validated
public class DemoController extends BaseController {

    @AuthPassport(value = false)
    @ResponseBody
    @RequestMapping(method = RequestMethod.GET)
    public String save(@NotNull(message = "不能为空") Integer id) {
        return "success";
    }
}
```
1.为类添加@Validated注解
2.校验方法的返回值和入参  
</div>
