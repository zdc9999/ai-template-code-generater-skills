---
name: "template-generator"
description: "从 Java/Spring Boot、Vue2、Vue3 项目中提炼代码模板，生成适配 code-generator 的 .tera 模板文件。当用户需要从现有项目中提取代码模式、创建可复用的 Tera 模板、或为 code-generator 添加新的模板组时调用。"
---

# 模板生成器 (Template Generator)

从现有项目代码中反向提炼 Tera 模板，为 [code-generator](../code-generator/SKILL.md) 技能提供模板支持。支持 Java/Spring Boot、Vue2、Vue3 三种技术栈。

## 目录结构

```
skills/template-generator/
├── SKILL.md                    # 本文件
└── examples/                   # 提炼示例（参考用）
    ├── java-spring-boot/       # Java/Spring Boot 提炼示例
    └── vue/                    # Vue2/Vue3 提炼示例
```

## 核心概念

### 提炼原理

模板提炼的本质是**参数化替换**：将项目中具体的类名、字段名、包名、路径等替换为 code-generator 识别的模板变量和 EasyFrame 宏，使其成为可复用的代码生成模板。

### 三层参数化

| 层级 | 说明 | 示例 |
|------|------|------|
| **表级参数** | 整表相关的变量 | `{{ tableInfo.name }}`、`{{ tableInfo.comment }}`、`{{ tableInfo.savePackageName }}` |
| **列级参数** | 字段相关的变量 | `{{ column.name }}`、`{{ column.shortType }}`、`{{ column.comment }}` |
| **全局参数** | 跨模板的变量 | `${basePackage}`、`${commonImports}`、`${author}` |

### 与 code-generator 的关系

```
现有项目代码 ──[template-generator 提炼]──▶ .tera 模板文件
                                                    │
                    code-generator 的 template/ 目录 ◀┘
                                                    │
              数据模型 JSON ──[code-generator 生成]──▶ 新项目代码
```

## 使用方法

### 交互流程

当用户触发此 Skill 时，严格按以下步骤操作：

#### 步骤 1：确认提炼目标

使用 **`AskUserQuestion`** 工具收集信息：

**问题 1 — 选择技术栈**（单选）：
- `Java/Spring Boot` — Spring Boot + MyBatis/MyBatis-Plus 项目
- `Vue2` — Vue2 + Options API 项目
- `Vue3` — Vue3 + Composition API 项目

**问题 2 — 源项目路径**（文本输入）：
- 让用户输入要提炼的源项目根目录绝对路径

**问题 3 — 目标模板组名称**（文本输入）：
- 模板组将保存到 `f:\code\ai-code-generater\.trae\skills\code-generator\template\{模板组名}\`
- 示例：`MyProject`、`VueAdmin`

#### 步骤 2：扫描源项目

根据技术栈，扫描并分析源项目代码结构，识别所有可提炼的模板文件。扫描完成后，**不要直接开始提炼**，先进入步骤 3。

**Java/Spring Boot 项目扫描：**
1. 定位实体类（Entity/Model/POJO）→ 可提炼 entity.java.tera
2. 定位控制器类（Controller）→ 可提炼 controller.java.tera
3. 定位服务接口（Service）→ 可提炼 service.java.tera
4. 定位服务实现（ServiceImpl）→ 可提炼 serviceImpl.java.tera
5. 定位数据访问层（Dao/Mapper/Repository）→ 可提炼 dao.java.tera
6. 定位 MyBatis XML 映射文件 → 可提炼 mapper.xml.tera
7. 提取字段类型信息 → 可生成 type-mapping.json

**Vue2/Vue3 项目扫描：**
1. 定位页面组件（views/pages）→ 可提炼 page.vue.tera
2. 定位业务组件（components）→ 可提炼 component.vue.tera
3. 定位 API 服务文件（api/services）→ 可提炼 api.js.tera / api.ts.tera
4. 定位状态管理（store/vuex/pinia）→ 可提炼 store.js.tera / store.ts.tera
5. 定位路由配置（router）→ 可提炼 router.js.tera / router.ts.tera

#### 步骤 3：选择生成模板（人机交互）

扫描完成后，使用 **`AskUserQuestion`** 工具让用户选择要生成哪些模板。模板分为两类：

**分类说明：**

| 分类 | 说明 | 示例 |
|------|------|------|
| **常规文件** | 业务代码模板，用于生成实际的业务代码文件 | entity.java.tera、controller.java.tera、page.vue.tera、api.js.tera 等 |
| **类型配置文件** | 类型映射配置，用于定义字段类型与 Java/TypeScript 类型的对应关系 | type-mapping.json（Java）、component-types.json（Vue） |

**交互问题设计：**

**问题 1 — 选择要生成的常规文件模板**（多选）：
- 列出所有扫描到的常规文件模板，标注每个模板的用途
- 默认全选，用户可取消不需要的模板
- 每个选项格式：`{模板文件名} — {用途说明}`
- **不选择时，默认全部生成**

**Java/Spring Boot 选项示例：**
- `entity.java.tera` — 实体类模板（数据库表映射）
- `controller.java.tera` — REST 控制器模板（API 接口层）
- `service.java.tera` — 服务接口模板（业务逻辑接口）
- `serviceImpl.java.tera` — 服务实现模板（业务逻辑实现）
- `dao.java.tera` — 数据访问层模板（DAO/Mapper 接口）
- `mapper.xml.tera` — MyBatis XML 映射模板（SQL 映射）

**Vue2/Vue3 选项示例：**
- `page.vue.tera` / `page.vue3.tera` — 页面组件模板（列表/表单页）
- `component.vue.tera` — 业务组件模板（可复用组件）
- `api.js.tera` / `api.ts.tera` — API 服务模板（接口封装）
- `store.js.tera` / `store.ts.tera` — 状态管理模板（Vuex/Pinia）
- `router.js.tera` / `router.ts.tera` — 路由配置模板（路由定义）

**问题 2 — 是否生成类型配置文件**（单选）：
- `是，生成类型配置文件` — 从源项目提取字段类型映射，生成 type-mapping.json 等
- `否，跳过类型配置` — 不生成类型配置文件

**交互规则：**
1. 只展示扫描中实际找到的模板（未找到对应源文件的模板不展示）
2. 如果用户取消所有常规文件且不生成类型配置，提示至少选择一个模板
3. 用户确认后，只生成选中的模板文件

#### 步骤 4：逐文件提炼

**仅对步骤 3 中用户选中的模板**执行提炼。对每个选中的模板类型，找到对应的源文件并执行以下提炼流程：

1. **读取源文件**，理解其结构和模式
2. **识别可变部分**：类名、字段名、包名、路径、注释等
3. **识别固定部分**：框架代码、注解、导入语句、业务逻辑骨架
4. **应用参数化规则**（见下方各技术栈的详细规则）
5. **添加 EasyFrame 宏**：`#save`、`#setPackageSuffix`、`$!callback` 等
6. **生成 .tera 文件**，保存到目标模板组目录

如果用户选择了生成类型配置文件，在常规文件提炼完成后，从实体类中提取所有字段类型，生成类型配置文件。

#### 步骤 5：验证模板

1. 运行 `./scripts/easyframe-codegen --list-groups` 确认新模板组可识别
2. 使用示例数据模型运行 `--dry-run` 预览生成结果
3. 抽查生成内容是否与源项目风格一致

---

## Java/Spring Boot 提炼规则

### 1. 实体类 → entity.java.tera

**识别特征：** 包含 `@Entity`、`@Table`、`@TableName` 注解，包含字段声明和 getter/setter。

**提炼步骤：**

1. 将类名替换为 `{{ tableInfo.name }}`
2. 将包声明替换为 `{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}#setPackageSuffix定义的包后缀;`
3. 在文件开头添加 `#save("/entity", ".java")` 和 `#setPackageSuffix("entity")`
4. 将类注释替换为 `#tableComment("实体类")`
5. 将字段列表包装在 `{% for column in tableInfo.fullColumn %} ... {% endfor %}` 循环中
6. 字段名 → `{{ column.name }}`，字段类型 → `{{ column.shortType }}`
7. 字段注释 → `{% if column.comment %}/** {{ column.comment }} */{% endif %}`
8. getter/setter 方法 → `#getSetMethod($column)`
9. 保留源项目特有的注解（如 `@Data`、`@Builder` 等），它们成为全局变量 `${classAnnotation}` 的候选

**示例：**

源文件 `User.java`:
```java
package com.example.demo.entity;

import lombok.Data;
import java.io.Serializable;
import java.util.Date;

@Data
public class User implements Serializable {
    private Long id;
    private String userName;
    private Date createTime;
}
```

提炼后 `entity.java.tera`:
```tera
#save("/entity", ".java")
#setPackageSuffix("entity")

import lombok.Data;
import java.io.Serializable;
import java.util.Date;

#tableComment("实体类")
@Data
public class {{ tableInfo.name }} implements Serializable {
{% for column in tableInfo.fullColumn %}
    {% if column.comment %}/**
     * {{ column.comment }}
     */{% endif %}

    private {{ column.shortType }} {{ column.name }};
{% endfor %}
}
```

### 2. 控制器 → controller.java.tera

**识别特征：** 包含 `@RestController`、`@Controller` 注解，包含 `@RequestMapping`、`@GetMapping` 等。

**提炼步骤：**

1. 在文件开头添加 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)`
2. 提取主键列：`{% if tableInfo.pkColumn | length > 0 %}{% set pk = tableInfo.pkColumn[0] %}{% endif %}`
3. 类名 → `{{ tableInfo.name }}Controller`
4. 包声明 → `{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}controller;`
5. RequestMapping → `@RequestMapping("{{ tableInfo.nameLowerFirst }}")`
6. 服务类引用 → `{{ tableInfo.name }}Service`
7. 服务字段名 → `{{ tableInfo.nameLowerFirst }}Service`
8. 实体参数名 → `{{ tableInfo.nameLowerFirst }}`
9. 主键类型 → `{{ pk.shortType }}`，主键名 → `{{ pk.name }}`
10. 作者 → `{{ author }}`，时间 → `{{ currentTime }}`

### 3. 服务接口 → service.java.tera

**识别特征：** 以 `Service` 结尾的接口，定义 CRUD 方法签名。

**提炼步骤：**

1. 添加 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)`
2. 接口名 → `{{ tableInfo.name }}Service`
3. 包声明 → `{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}service;`
4. 方法返回类型中的实体类 → `{{ tableInfo.name }}`
5. 方法参数中的实体类 → `{{ tableInfo.name }}`，参数名 → `{{ tableInfo.nameLowerFirst }}`
6. 主键参数 → `{{ pk.shortType }} {{ pk.name }}`

### 4. 服务实现 → serviceImpl.java.tera

**识别特征：** 以 `ServiceImpl` 结尾的类，实现对应 Service 接口。

**提炼步骤：**

1. 添加 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)`
2. 类名 → `{{ tableInfo.name }}ServiceImpl`
3. `@Service` 注解值 → `"{{ tableInfo.nameLowerFirst }}Service"`
4. DAO 字段 → `{{ tableInfo.name }}Dao`，字段名 → `{{ tableInfo.nameLowerFirst }}Dao`
5. 方法中的 DAO 调用保持原样，实体参数替换为模板变量

### 5. DAO 接口 → dao.java.tera

**识别特征：** 以 `Dao`/`Mapper`/`Repository` 结尾的接口，定义数据库操作方法。

**提炼步骤：**

1. 添加 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)`
2. 接口名 → `{{ tableInfo.name }}Dao`
3. 包声明 → `{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}dao;`
4. 所有实体引用 → `{{ tableInfo.name }}`

### 6. Mapper XML → mapper.xml.tera

**识别特征：** MyBatis XML 映射文件，包含 `<mapper>`、`<select>`、`<insert>`、`<update>`、`<delete>` 标签。

**提炼步骤：**

1. 添加 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)`
2. namespace → `{{ tableInfo.savePackageName }}.dao.{{ tableInfo.name }}Dao`
3. resultMap type → `{{ tableInfo.savePackageName }}.entity.{{ tableInfo.name }}`
4. resultMap id → `{{ tableInfo.name }}Map`
5. 列映射包装在 `{% for column in tableInfo.fullColumn %}` 中：
   - property → `{{ column.name }}`
   - column → `{{ column.obj.name }}`
   - jdbcType → `{{ column.ext.jdbcType }}`
6. 表名 → `{{ tableInfo.obj.name }}`
7. 所有列 → `#allSqlColumn()`
8. 主键列 → `{{ pk.obj.name }}`，主键值 → `#{{ pk.name }}`
9. 非主键列 → `{% for column in tableInfo.otherColumn %}`
10. String 类型判断 → `{% if column.type == "java.lang.String" %}`

### 7. 类型映射 → type-mapping.json

从源项目的实体类中提取所有字段类型，生成类型映射配置：

```json
{
  "Long": ["Long", "java.lang.Long"],
  "String": ["String", "java.lang.String"],
  "Date": ["Date", "java.util.Date"],
  ...
}
```

---

## Vue2/Vue3 提炼规则

### 通用原则

Vue 模板提炼的目标是生成 `.vue.tera` 文件，支持组件级代码生成。与 Java 后端不同，Vue 前端模板的数据模型更灵活。

**数据模型格式（Vue 模板使用）：**

```json
{
  "componentName": "UserList",
  "componentNameKebab": "user-list",
  "routePath": "/user",
  "pageTitle": "用户管理",
  "apiModule": "user",
  "props": [
    {"name": "userId", "type": "String", "required": true, "default": ""}
  ],
  "tableColumns": [
    {"prop": "userName", "label": "用户名", "width": 120, "sortable": true}
  ],
  "formFields": [
    {"prop": "userName", "label": "用户名", "type": "input", "required": true}
  ],
  "searchFields": [
    {"prop": "userName", "label": "用户名", "type": "input"}
  ]
}
```

### 1. 页面组件 → page.vue.tera / page.vue3.tera

**提炼步骤：**

**模板部分：**
1. 将组件内的具体字段名替换为 `{% for field in formFields %}` 循环
2. 表格列替换为 `{% for col in tableColumns %}`
3. 搜索表单替换为 `{% for field in searchFields %}`
4. 组件名标签 → `{{ pageTitle }}`

**脚本部分（Vue2 Options API）：**
1. `name` → `"{{ componentName }}"`
2. `data()` 中的字段 → 使用 `{% for field in formFields %}` 生成
3. `methods` 中的 API 调用 → 替换为 `api.{{ apiModule }}.xxx`
4. 路由跳转 → `this.$router.push("{{ routePath }}")`

**脚本部分（Vue3 Composition API）：**
1. `defineOptions({ name: "{{ componentName }}" })`
2. `ref`/`reactive` 字段 → 使用循环生成
3. `useRouter().push("{{ routePath }}")`

### 2. 业务组件 → component.vue.tera

**提炼步骤：**

1. 组件名 → `{{ componentName }}`
2. Props 定义 → `{% for prop in props %}{{ prop.name }}: { type: {{ prop.type }}, ... }{% endfor %}`
3. Emits 定义 → `{% for event in emits %}...{% endfor %}`
4. Slots → `{% for slot in slots %}...{% endfor %}`

### 3. API 服务 → api.js.tera / api.ts.tera

**提炼步骤：**

1. 基础路径 → `{{ apiBasePath }}`
2. 每个 API 方法 → 提取为 `{% for api in apis %}` 循环
3. URL 中的具体 ID → `{{ api.url }}`
4. 请求方法 → `{{ api.method }}`

### 4. 状态管理 → store.js.tera / store.ts.tera

**Vuex 提炼：**
1. `state` → `{% for field in stateFields %}`
2. `mutations` → `{% for mutation in mutations %}`
3. `actions` → `{% for action in actions %}`

**Pinia 提炼：**
1. `defineStore("{{ storeName }}", ...)`
2. `state` → `() => ({ {% for field in stateFields %}...{% endfor %} })`
3. `getters` → `{% for getter in getters %}`
4. `actions` → `{% for action in actions %}`

### 5. 路由配置 → router.js.tera / router.ts.tera

**提炼步骤：**

1. 每个路由条目 → `{% for route in routes %}`
2. `path` → `"{{ route.path }}"`
3. `name` → `"{{ route.name }}"`
4. `component` → `() => import("{{ route.componentPath }}")`
5. `meta.title` → `"{{ route.title }}"`

---

## EasyFrame 宏使用规范

### 输出路径指定

每个 .tera 文件必须包含输出路径指令，二选一：

**Save 模式**（适用于固定路径的文件）：
```tera
#save("/entity", ".java")         # 输出到 /entity/ 目录，后缀 .java
#setPackageSuffix("entity")       # 设置包后缀
```

**Callback 模式**（适用于动态路径的文件）：
```tera
$!callback.setFileName($tool.append($tableInfo.name, "Controller.java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))
```

### 宏速查表

| 宏 | 用途 | 示例 |
|---|------|------|
| `#save("/path", ".ext")` | 指定输出路径和后缀 | `#save("/entity", ".java")` |
| `#setPackageSuffix("suffix")` | 设置包后缀 | `#setPackageSuffix("entity")` |
| `#setTableSuffix("suffix")` | 设置表名后缀 | `#setTableSuffix("")` |
| `#tableComment("desc")` | 生成类注释 | `#tableComment("实体类")` |
| `#getSetMethod($column)` | 生成 getter/setter | `#getSetMethod($column)` |
| `#allSqlColumn()` | 生成所有列名 | `#allSqlColumn()` |
| `$!callback.setFileName(...)` | 设置输出文件名 | 见上方示例 |
| `$!callback.setSavePath(...)` | 设置输出路径 | 见上方示例 |

### 预计算变量速查表

| 变量 | 说明 |
|------|------|
| `{{ tableInfo.name }}` | 表名（首字母大写驼峰） |
| `{{ tableInfo.nameLowerFirst }}` | 表名（首字母小写驼峰） |
| `{{ tableInfo.comment }}` | 表注释 |
| `{{ tableInfo.savePackageName }}` | 包名 |
| `{{ tableInfo.savePath }}` | 保存路径 |
| `{{ tableInfo.obj.name }}` | 原始表名 |
| `{{ tableInfo.fullColumn }}` | 所有列 |
| `{{ tableInfo.pkColumn }}` | 主键列 |
| `{{ tableInfo.otherColumn }}` | 非主键列 |
| `{{ column.name }}` | 列名（首字母小写驼峰） |
| `{{ column.shortType }}` | 列短类型 |
| `{{ column.type }}` | 列全限定类型 |
| `{{ column.comment }}` | 列注释 |
| `{{ column.obj.name }}` | 原始列名 |
| `{{ column.ext.jdbcType }}` | JDBC 类型 |
| `{{ column.nameUpperFirst }}` | 列名首字母大写 |
| `{{ column.nameUnderline }}` | 列名下划线形式 |
| `{{ serialUID }}` | 随机 serialVersionUID |
| `{{ currentTime }}` | 当前时间 |
| `{{ author }}` | 作者 |
| `{{ modulePath }}` | 模块路径 |
| `{{ projectPath }}` | 项目路径 |

---

## 全局变量提取

在提炼过程中，识别以下可提取为全局变量的内容：

1. **基础包名** → `${basePackage}`
2. **公共导入** → `${commonImports}`（如 `import lombok.Data;`）
3. **类注解** → `${classAnnotation}`（如 `@Data\n@Builder`）
4. **公共方法** → `${commonMethods}`
5. **作者信息** → `${author}`

全局变量配置写入 `global-config.json` 或数据模型 JSON 的 `globalConfig` 字段。

---

## 质量检查清单

提炼完成后，必须逐项检查：

### Java/Spring Boot 模板检查

- [ ] 每个 .tera 文件包含输出路径宏（`#save` 或 `$!callback`）
- [ ] 类名、接口名使用 `{{ tableInfo.name }}` 而非硬编码
- [ ] 包声明使用 `{{ tableInfo.savePackageName }}` 而非硬编码
- [ ] 字段使用 `{% for column in tableInfo.fullColumn %}` 循环
- [ ] 主键引用使用 `{% set pk = tableInfo.pkColumn[0] %}` 提取
- [ ] 字段类型使用 `{{ column.shortType }}`
- [ ] 注释使用 `{{ column.comment }}` 或 `{{ tableInfo.comment }}`
- [ ] 时间使用 `{{ currentTime }}`，作者使用 `{{ author }}`
- [ ] getter/setter 使用 `#getSetMethod($column)` 宏
- [ ] Mapper XML 中列名使用 `{{ column.obj.name }}`（原始列名）
- [ ] type-mapping.json 覆盖所有字段类型

### Vue 模板检查

- [ ] 组件名使用 `{{ componentName }}`
- [ ] 字段使用 `{% for field in ... %}` 循环
- [ ] API 路径使用模板变量
- [ ] 路由路径使用 `{{ routePath }}`
- [ ] 区分 Vue2 Options API 和 Vue3 Composition API 的语法差异

---

## 注意事项

1. **保留框架特征**：提炼时保留源项目的框架特有注解和代码风格，不要过度简化
2. **全局变量优先**：公共的、跨模板的代码片段优先提取为全局变量
3. **模板组命名**：使用驼峰命名，如 `MyProject`、`VueAdmin`
4. **type-mapping.json**：Java 模板组必须包含类型映射，覆盖源项目中所有字段类型
5. **编码**：所有 .tera 文件使用 UTF-8 编码
6. **Tera 语法兼容**：确保生成的模板使用 Tera 语法（`{{ }}`、`{% %}`），而非 Velocity 语法
7. **EasyFrame 宏前置**：`#save`、`$!callback` 等宏必须在文件最开头
8. **注释保留**：源文件中的中文注释、业务逻辑注释应保留并参数化