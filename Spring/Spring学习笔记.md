# Spring

## 1.1 简介

- SSH：
- SSM：SpringMVC + Spring +Mybatis

## 1.2优点

- Spring是一个开源免费的框架（容器）
- 是一个轻量级、非入侵市的框架
- 控制反转（IOC）、面向切面编程（AOP）
- 支持事务处理和对框架整合的支持

==总的来说就是Spring是一个轻量级的控制反转（IOC）、面向切面编程（AOP）的框架 ==

## 1.3 Spring的组成

![1666089007407](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1666089007407.png)

## 1.4 拓展

- Spring Boot
  - 一个快速开发的脚手架
  - 基于SpringBoot可以快速的开发单个微服务
  - 约定大于配置！！
- Sping Cloud
  - SpingCloud是基于SpringBoot实现的



# IOC理论推导

## DAO

- UserDao接口
- UserDaoImpl实现类
- UserServic业务接口
- UserServiceImpl业务实现类

总结：用户的需求可能会影响我们的源代码，而我们根据需求去修改源代码！！这样会破环程序的完整性



我们使用set接口实现

``` java
public class UserServiceImpl implements UserService{
    private UserDao userDao;

    //set注入，进行动态实现值的注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void getUser() {
        userDao.getUser();
    }
}

```

