---
title: 【SpringBoot篇】自定义异常处理
date: '2024-06-06 18:27'
tag: SpringBoot
categories: Java
cover: 'https://s21.ax1x.com/2024/07/17/pkI5pCQ.png'
abbrlink: 2677420933
---



## 🎈异常处理

在 Java 中，自定义异常是指用户根据自己的需求创建的异常类。Java 提供了一些预定义的异常类，如 **NullPointerException**、**ArrayIndexOutOfBoundsException**等，但有时这些预定义的异常类并不能完全满足我们的需求。在这种情况下，我们可以通过创建自定义异常类来表示特定的异常情况。

自定义异常类通常继承自 Exception 类或 RuntimeException 类，以及它们的子类，并根据需要添加相应的构造方法和其他方法以满足特定的异常处理需求，自定义异常类可以包含额外的属性和方法，以提供更多的信息和功能。

使用自定义异常类时，通常的做法是在方法中使用 throw 语句来抛出自定义异常，然后在调用该方法的地方使用 try-catch 语句块来捕获并处理异常。

![异常类](https://s21.ax1x.com/2024/07/16/pkI2JUJ.png)

> 我们想让异常结果也显示为统一的返回结果对象，并且统一处理系统的异常信息，那么，需要统一异常处理
## 🎆统一异常处理器
>@ExceptionHandler(Exception.class)
>@ControllerAdvice

```java
	/**
	 * 统一异常处理类
	 */
	@ControllerAdvice
	public class GlobalExceptionHander {


	@ExceptionHandler(Exception.class)
	@ResponseBody
	public Result error(Exception e){
		e.printStackTrace();
		return  Result.build(null,201,"出现了异常");
	}
	}

    /**
     * 特定的异常处理
     */

    @ExceptionHandler(ArithmeticException.class)
    @ResponseBody
    public Result error(ArithmeticException e){
        e.printStackTrace();
        return  Result.build(null,202,"特定的异常");
    }

```
## 🎀自定义异常
```java

@Data
public class GuiguException  extends  RuntimeException {

    private Integer CODE;
    private String  message;
    private ResultCodeEnum resultCodeEnum;

    public GuiguException() {
    }

    public GuiguException(Integer CODE, String message) {
        this.CODE = CODE;
        this.message = message;
    }

    public GuiguException(ResultCodeEnum resultCodeEnum) {
        this.CODE = resultCodeEnum.getCode();
        this.message = resultCodeEnum.getMessage();
        this.resultCodeEnum = resultCodeEnum;
    }
}

```
```java
/**
 * 自定义异常
 */
@ExceptionHandler(GuiguException.class)
@ResponseBody
public Result error(GuiguException e)
{
    e.printStackTrace();
    return Result.build(null,e.getCODE(),e.getMessage());
}
```

```

```