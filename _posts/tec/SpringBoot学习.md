---
title: SpringBoot学习
date: 2025-03-30
tags: ['Java']
categories: ['技术']
author: QuHou
---

# CRUD流程:
## 1. 在pojo里写User表
```Java
package com.learn.learn.pojo;

import jakarta.persistence.*;

@Table(name="tb_user")
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    private Integer userId;
    @Column(name = "user_name")
    private String userName;
    @Column(name = "password")
    private String password;
    @Column(name = "email")
    private String email;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}

```
## 2. 在pojo.dto里写UserDTO，表示从前端接收的字段名

```Java
package com.learn.learn.pojo.dto;

import jakarta.persistence.Column;

public class UserDTO {
    private String userName;
    private String password;
    private String email;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "UserDTO{" +
                "userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}

```

## 3. 在pojo里写ResponseMessage泛型类，表示要返回给前端的数据结构

```Java
package com.learn.learn.pojo;

import org.springframework.http.HttpStatus;

public class ResponseMessage<T> {
    private final Integer code;
    private final String message;
    private final T data;

    public ResponseMessage(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    public static <T> ResponseMessage<T> success(T data) {
        return new ResponseMessage(HttpStatus.OK.value(), "success", data);
    }
}

```

## 4. 在controller里写UserController类用来控制实际的请求

```Java
package com.learn.learn.controller;

import com.learn.learn.pojo.User;
import com.learn.learn.pojo.dto.UserDTO;
import com.learn.learn.service.IUserService;
import com.learn.learn.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController //接口返回的对象转换为json文本
@RequestMapping("/user") //访问localhost:8080/user 访问user接口
public class UserController {

    @Autowired
    IUserService userService;

//    增加
    @PostMapping
    public String add(@RequestBody UserDTO user){
        userService.add(user);
        return  "success!";
    }
//    查询
//    修改
//    删除

}
```

## 5. 在service里写UserService类用来操作数据

```Java
package com.learn.learn.service;

import com.learn.learn.pojo.User;
import com.learn.learn.pojo.dto.UserDTO;
import com.learn.learn.repository.UserRepository;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service //spring bean
public class UserService implements IUserService {
    @Autowired
    UserRepository userRepository;

    @Override
    public void add(UserDTO user) {
//        调用数据访问类
        User userPojo = new User();
        BeanUtils.copyProperties(user,userPojo);
        userRepository.save(userPojo);
    }
}
```

## 6. 在repository里写UserRepository类，用来真正对数据库进行操作

```Java
package com.learn.learn.repository;

import com.learn.learn.pojo.User;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class UserRepository implements CrudRepository<User, Integer> {

    @Override
    public <S extends User> S save(S entity) {
        return null;
    }

    @Override
    public <S extends User> Iterable<S> saveAll(Iterable<S> entities) {
        return null;
    }

    @Override
    public Optional<User> findById(Integer integer) {
        return Optional.empty();
    }

    @Override
    public boolean existsById(Integer integer) {
        return false;
    }

    @Override
    public Iterable<User> findAll() {
        return null;
    }

    @Override
    public Iterable<User> findAllById(Iterable<Integer> integers) {
        return null;
    }

    @Override
    public long count() {
        return 0;
    }

    @Override
    public void deleteById(Integer integer) {

    }

    @Override
    public void delete(User entity) {

    }

    @Override
    public void deleteAllById(Iterable<? extends Integer> integers) {

    }

    @Override
    public void deleteAll(Iterable<? extends User> entities) {

    }

    @Override
    public void deleteAll() {

    }
}
```