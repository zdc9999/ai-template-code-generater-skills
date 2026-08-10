# Java/Spring Boot 模板提炼参考指南

## 提炼流程概览

```
源项目 src/main/java/com/example/demo/
├── entity/User.java          ──▶  entity.java.tera
├── controller/UserController.java  ──▶  controller.java.tera
├── service/UserService.java        ──▶  service.java.tera
├── service/impl/UserServiceImpl.java  ──▶  serviceImpl.java.tera
├── dao/UserDao.java                ──▶  dao.java.tera
└── src/main/resources/mapper/UserDao.xml  ──▶  mapper.xml.tera
```

## 提炼对照示例

### 1. Entity → entity.java.tera

**源文件 (User.java):**
```java
package com.example.demo.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.io.Serializable;
import java.util.Date;

@Data
@TableName("sys_user")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    /** 主键ID */
    private Long id;

    /** 用户名 */
    private String userName;

    /** 创建时间 */
    private Date createTime;
}
```

**提炼后 (entity.java.tera):**
```tera
#save("/entity", ".java")
#setPackageSuffix("entity")

import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.io.Serializable;

#tableComment("实体类")
@Data
@TableName("{{ tableInfo.obj.name }}")
public class {{ tableInfo.name }} implements Serializable {
    private static final long serialVersionUID = {{ serialUID }};
{% for column in tableInfo.fullColumn %}
    {% if column.comment %}/**
     * {{ column.comment }}
     */{% endif %}

    private {{ column.shortType }} {{ column.name }};
{% endfor %}
}
```

### 2. Controller → controller.java.tera

**源文件 (UserController.java):**
```java
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import org.springframework.web.bind.annotation.*;
import javax.annotation.Resource;

@RestController
@RequestMapping("/user")
public class UserController {
    @Resource
    private UserService userService;

    @GetMapping("/{id}")
    public User getById(@PathVariable Long id) {
        return userService.getById(id);
    }
}
```

**提炼后 (controller.java.tera):**
```tera
$!callback.setFileName($tool.append($tableInfo.name, "Controller.java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))

{% if tableInfo.pkColumn | length > 0 %}
    {% set pk = tableInfo.pkColumn[0] %}
{% endif %}

{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}controller;

import {{ tableInfo.savePackageName }}.entity.{{ tableInfo.name }};
import {{ tableInfo.savePackageName }}.service.{{ tableInfo.name }}Service;
import org.springframework.web.bind.annotation.*;
import javax.annotation.Resource;

@RestController
@RequestMapping("{{ tableInfo.nameLowerFirst }}")
public class {{ tableInfo.name }}Controller {
    @Resource
    private {{ tableInfo.name }}Service {{ tableInfo.nameLowerFirst }}Service;

    @GetMapping("/{id}")
    public {{ tableInfo.name }} getById(@PathVariable {{ pk.shortType }} id) {
        return {{ tableInfo.nameLowerFirst }}Service.getById(id);
    }
}
```

### 提炼关键点

1. **类名替换**: `User` → `{{ tableInfo.name }}`
2. **包名替换**: `com.example.demo` → `{{ tableInfo.savePackageName }}`
3. **字段替换**: 硬编码字段 → `{% for column in tableInfo.fullColumn %}`
4. **主键提取**: `Long id` → `{% set pk = tableInfo.pkColumn[0] %}` → `{{ pk.shortType }} {{ pk.name }}`
5. **添加宏**: 文件开头添加 `#save` 或 `$!callback`
6. **注释保留**: 中文注释保留原样，放在 `{% if column.comment %}` 条件中