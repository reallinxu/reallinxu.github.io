---
title: " QLExpress规则引擎实例"
id: 20200117
categories:
  - QLExpress
date: 2020-1-17 11:30:35
tags: QLExpress
---

## what is QLExpress
QLExpress是阿里开源的一个规则引擎，对java支持良好，具体描述及用法直接移步github：https://github.com/alibaba/QLExpress

## just do it!
github中对基本用法已经描述很清楚，此处不再赘述，我们直接通过一个入门案例来帮忙理解，

### fisrt 
我们先来看一段java伪代码
```java
if(新用户) {
  赠送优惠券;
} else {
  赠送积分;
}
if(有手机号){
  发送短信;
}
```
我们代码中都会出现类似的逻辑，如果此时我想要修改为新用户增送积分，否则赠送优惠券呢，这时候就需要修改if-else，此时我们可以使用规则引擎来轻松实现逻辑，并通过修改规则引擎描述就可以直接修改功能，咋做嘞，往下瞅~

### show you code
```java
package com.qlexpress.demo;

import com.ql.util.express.DefaultContext;
import com.ql.util.express.ExpressRunner;

public class Test {

    public static void main(String[] args) throws Exception {
        ExpressRunner runner = new ExpressRunner();

//      如果 新用户 赠送代金券  否则  赠送积分
//      如果 有手机号  发送短信

//      定义逻辑
        runner.addOperatorWithAlias("如果", "if", null);
        runner.addOperatorWithAlias("则", "then", null);
        runner.addOperatorWithAlias("否则", "else", null);

//      定义执行方法
        runner.addFunctionOfClassMethod("赠送优惠券", Test.class.getName(),
                "sendCoupon", new Class[]{Integer.class}, null);
        runner.addFunctionOfClassMethod("赠送积分", Test.class.getName(),
                "sendIntegral", new Class[]{Integer.class}, null);
        runner.addFunctionOfServiceMethod("发送短信", new Test(), "sendMsg",
                new String[]{"String"}, null);

//      定义逻辑
        runner.addFunctionOfServiceMethod("是否新用户", new Test(), "isNewAcct",
                new Class[]{Integer.class}, null);
        runner.addFunctionOfServiceMethod("是否有手机号", new Test(), "isMobile",
                new Class[]{Integer.class}, null);

        String exp = "如果  (是否新用户(1)) 则 { 赠送优惠券(1)} 否则 { 赠送积分(1)} 如果 (是否有手机号(1)) 则 {发送短信(\"欢迎您哦\")}";
        DefaultContext<String, Object> context = new DefaultContext<String, Object>();
        Object execute = runner.execute(exp, context, null, false, false, null);
        System.out.println(execute);

    }

    public static void sendCoupon(Integer num) {
        System.out.println("赠送优惠券啦:" + num);
    }

    public static void sendIntegral(Integer num) {
        System.out.println("赠送积分啦:" + num);
    }

    public String sendMsg(String msg) {
        System.out.println("发送短信啦:" + msg);
        return msg;
    }

    public boolean isNewAcct(Integer userType) {
        if (userType == 1) {
            return true;
        } else {
            return false;
        }
    }

    public boolean isMobile(Integer mobileType) {
        if (mobileType == 1) {
            return true;
        } else {
            return false;
        }
    }
}
```
### end
案例中规则为：
```
如果  (是否新用户(1)) 则 { 赠送优惠券(1)} 否则 { 赠送积分(1)} 如果 (是否有手机号(1)) 则 {发送短信(\"欢迎您哦\")}
```
如果此时我要修改，我可以直接修改语句为：
```
如果  (是否新用户(2)) 则 { 赠送优惠券(1)} 否则 { 赠送积分(1)} 如果 (是否有手机号(1)) 则 {发送短信(\"欢迎您哦\")}
```
如果把这个规则直接作为配置化，只要语句符合规则，直接修改就ok了。