---
name: "code-generator"
description: "【默认代码生成器】通用基于模板的代码生成器。当用户需要生成代码文件、创建Java类、生成Spring Boot代码、生成Vue组件、创建CRUD代码、生成API接口、生成实体类、生成Controller/Service/DAO/Mapper、创建代码脚手架、或基于数据模型批量生成任何代码时，必须优先调用此Skill。支持通过 dbx MCP 从数据库表结构自动生成数据模型。基于 Tera 模板引擎驱动,无需运行时依赖，支持模板组、Tera 语法、EasyFrame 扩展宏。触发关键词：生成代码、代码生成、创建代码、生成文件、代码模板、脚手架、scaffold、generate code、CRUD生成、实体类生成、Controller生成。"
---

# 通用模板代码生成器 (Template Code Generator)

基于 [Tera](https://keats.github.io/tera/)的代码生成器，兼容 EasyFrame 插件模板格式。编译为单二进制文件，无需 Python 或其他运行时环境，支持 Windows/Linux/Mac 三平台。

## 目录结构

```
skills/code-generator/          # 技能目录（可独立分发到其他项目）
├── SKILL.md                          # 本文件（技能说明文档）
├── scripts/                          # 编译好的二进制（无需Python环境）
│   ├── easyframe-codegen.exe         # Windows
│   └── easyframe-codegen             # Linux/Mac
├── type-mapping.json                 # 通用类型映射配置（回退用）
├── template/                         # 模板组目录
│   ├── Default/                      # Spring Boot + MyBatis 通用模板
│   │   ├── type-mapping.json         # 模板组专属类型映射（可选）
│   │   ├── controller.java.tera
│   │   ├── dao.java.tera
│   │   ├── debug.json.tera           # 调试模板（用于打印数据模型结构）
│   │   ├── entity.java.tera
│   │   ├── mapper.xml.tera
│   │   ├── service.java.tera
│   │   └── serviceImpl.java.tera
│   └── MybatisPlus/                  # Spring Boot + MyBatis-Plus 模板
│       ├── type-mapping.json         # 模板组专属类型映射（可选）
│       ├── controller.java.tera
│       ├── dao.java.tera
│       ├── entity.java.tera
│       ├── service.java.tera
│       └── serviceImpl.java.tera
└── example/                          # 示例数据模型
    └── data-model.json               # 示例数据模型
```

## 使用方法

### 命令行

```bash
# 列出所有可用模板组
./scripts/easyframe-codegen --list-groups

# 交互模式（无参数时自动列出模板组并提示选择）
./scripts/easyframe-codegen

# 使用 Default 模板生成代码
./scripts/easyframe-codegen --group Default --data example/data-model.json --output ./output

# 使用 MybatisPlus 模板生成代码
./scripts/easyframe-codegen --group MybatisPlus --data example/data-model.json --output ./output

# 预览模式（不写入文件）
./scripts/easyframe-codegen --group Default --data example/data-model.json --output ./output --dry-run

# 使用全局变量配置
./scripts/easyframe-codegen --group Default --data example/data-model.json --output ./output --global-config global-config.json
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--list-groups` | 列出所有可用模板组 |
| `--group` / `-g` | 模板组名称 |
| `--data` / `-d` | 数据模型 JSON 文件路径 |
| `--output` / `-o` | 输出根目录（默认 `./output`） |
| `--global-config` | 全局变量配置 JSON 文件路径（也可在数据模型 JSON 中通过 `globalConfig` 字段配置） |
| `--dry-run` | 仅预览，不写入文件 |

## 模板语法

模板使用 Tera 模板语言。模板引擎负责变量替换、条件判断和循环控制。

### 变量引用
- `{{ var }}` — 变量引用
- `{{ obj.prop }}` — 对象属性访问
- `{{ obj.nested.prop }}` — 嵌套属性访问

### 控制指令
- `{% set var = value %}` — 变量赋值
- `{% for item in list %} ... {% endfor %}` — 循环
- `{% if cond %} ... {% elif cond %} ... {% else %} ... {% endif %}` — 条件
- `{% break %}` — 跳出循环
- `{# comment #}` — 行注释

### EasyFrame 扩展宏

以下宏是 EasyFrame 特有的，不属于任何模板引擎语法，在 Tera 渲染前由预处理器处理，**模板中保留原样**：

| 宏 | 说明 | 处理方式 |
|---|------|---------|
| `#save("/path", ".ext")` | 指定输出路径和文件后缀 | 提取输出路径和后缀，从模板中移除。文件名 = `{tableInfo.name}{后缀}` |
| `#setPackageSuffix("suffix")` | 设置包后缀 | 提取包后缀，从模板中移除 |
| `#setTableSuffix("suffix")` | 设置表名后缀，上下文注入 `{{ tableName }}` 变量 | 提取表名后缀，设置 `tableName` = `tableInfo.name` + 后缀，用于类名引用 |
| `#tableComment("description")` | 生成表注释块 | 替换为 Java 注释块字面量 |
| `#getSetMethod($column)` | 生成 getter/setter 方法 | 替换为 getter/setter 代码 |
| `#allSqlColumn()` | 生成所有 SQL 列名 | 替换为逗号分隔的列名字面量 |
| `$!callback.setFileName(...)` | 动态设置输出文件名 | 提取文件名，从模板中移除 |
| `$!callback.setSavePath(...)` | 动态设置输出文件路径 | 提取保存路径，从模板中移除 |
| `$!callback.setWriteFile(false)` | 禁止将生成结果写入文件（仅预览） | 设置写入标志，从模板中移除 |
| `$!tool.append(var, "str")` | 拼接变量与字符串字面量 | 预处理为字符串拼接结果 |
| `$!tool.debug(obj)` | 调试输出对象结构到控制台 | 打印对象信息，从模板中移除 |
| `$!tool.getField(obj, "fieldName")` | 反射获取对象字段值 | 预处理为字段值 |

### 输出路径模式

模板支持两种指定输出路径的方式，同一模板中只能使用一种：

#### 1. Save 模式（`#save` 宏）

通过 `#save("/path", ".ext")` 静态指定输出路径和文件后缀。**生成的文件名固定为 `{tableInfo.name}{后缀}`**（即使用原始表名，不受 `#setTableSuffix` 影响）。

```tera
#save("/entity", ".java")
#setPackageSuffix("entity")
```

输出路径 = `{output}/entity/{表名}.java`

> 适用于 Default/entity.java.tera、MybatisPlus 全部模板

**配合 `#setTableSuffix` 使用**（MybatisPlus 模板典型用法）：

```tera
#setTableSuffix("Controller")
#save("/controller", "Controller.java")
#setPackageSuffix("controller")
```

此时：`tableName` = `UserController`（用于类名），文件路径 = `{output}/controller/UserController.java`

> 注意：`#save` 的第二个参数 `"Controller.java"` 是拼接在 `tableInfo.name` 之后的，而非 `tableName`。例如 `tableInfo.name` = `User`，则文件名为 `UserController.java`。

#### 2. Callback 模式（`$!callback` 语法）

通过 `$!callback.setFileName(...)` 和 `$!callback.setSavePath(...)` 动态拼接输出路径。使用 `$tool.append(var, "literal")` 拼接变量和字符串。

```tera
$!callback.setFileName($tool.append($tableInfo.name, "Controller.java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))
```

输出路径 = `{output}/controller/{表名}Controller.java`

> 适用于 Default/controller.java.tera、Default/dao.java.tera、Default/service.java.tera、Default/serviceImpl.java.tera、Default/mapper.xml.tera

### 预计算上下文字段

模板中不直接调用工具方法，而是使用预计算并注入到上下文的字段：

| 上下文字段 | 说明 | 替代的方法调用 |
|-----------|------|--------------|
| `{{ column.shortType }}` | 短类型名（如 `String`） | `$tool.getClsNameByFullName($column.type)` |
| `{{ column.nameUpperFirst }}` | 列名首字母大写 | `$tool.firstUpperCase($column.name)` |
| `{{ column.nameUnderline }}` | 列名驼峰转下划线 | `$tool.hump2Underline($column.name)` |
| `{{ column.obj.name }}` | 列原始数据库名（如 `user_name`） | `$column.obj.name` |
| `{{ tableInfo.nameLowerFirst }}` | 表名首字母小写 | `$tool.firstLowerCase($tableInfo.name)` |
| `{{ serialUID }}` | 随机 serialVersionUID | `$tool.serial()` |
| `{{ currentTime }}` | 当前时间（`yyyy-MM-dd HH:mm:ss`） | `$time.currTime()` |

> 💡 **主键列快捷引用：** 模板中通常通过 `{% set pk = tableInfo.pkColumn[0] %}` 获取主键列，之后可使用 `{{ pk.name }}`、`{{ pk.shortType }}`、`{{ pk.nameUpperFirst }}`、`{{ pk.obj.name }}` 等全部 ColumnInfo 属性。

### 模板示例

#### Save 模式 — Default 实体类（entity.java.tera）

使用 `#save` 宏指定输出路径和后缀：

```tera
#save("/entity", ".java")
#setPackageSuffix("entity")
import java.io.Serializable;

#tableComment("实体类")
public class {{ tableInfo.name }} implements Serializable {
    private static final long serialVersionUID = {{ serialUID }};
{% for column in tableInfo.fullColumn %}
    {% if column.comment %}/**
     * {{ column.comment }}
     */{% endif %}

    private {{ column.shortType }} {{ column.name }};
{% endfor %}

{% for column in tableInfo.fullColumn %}
#getSetMethod($column)
{% endfor %}
}
```

#### Save 模式 — MybatisPlus 实体类（entity.java.tera）

使用 `#setTableSuffix` 不设置后缀（继承 Model 后类名不变），使用 `{% break %}` 跳出主键循环：

```tera
#save("/entity", ".java")
#setPackageSuffix("entity")

import com.baomidou.mybatisplus.extension.activerecord.Model;
import java.io.Serializable;

#tableComment("表实体类")
@SuppressWarnings("serial")
public class {{ tableInfo.name }} extends Model<{{ tableInfo.name }}> {
{% for column in tableInfo.fullColumn %}
    {% if column.comment %}/*{{ column.comment }}*/{% endif %}

    private {{ column.shortType }} {{ column.name }};
{% endfor %}

{% for column in tableInfo.fullColumn %}
#getSetMethod($column)
{% endfor %}

{% for column in tableInfo.pkColumn %}
    @Override
    protected Serializable pkVal() {
        return this.{{ column.name }};
    }
    {% break %}
{% endfor %}
}
```

#### Callback 模式（controller.java.tera）

使用 `$!callback.setFileName` 和 `$!callback.setSavePath` 动态指定输出路径：

```tera
{# 拿到主键 #}
{% if tableInfo.pkColumn | length > 0 %}
    {% set pk = tableInfo.pkColumn[0] %}
{% endif %}

$!callback.setFileName($tool.append($tableInfo.name, "Controller.java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))

{% if tableInfo.savePackageName %}package {{ tableInfo.savePackageName }}.{% endif %}controller;

import {{ tableInfo.savePackageName }}.entity.{{ tableInfo.name }};
import {{ tableInfo.savePackageName }}.service.{{ tableInfo.name }}Service;
// ...

@RestController
@RequestMapping("{{ tableInfo.nameLowerFirst }}")
public class {{ tableInfo.name }}Controller {
    @Resource
    private {{ tableInfo.name }}Service {{ tableInfo.nameLowerFirst }}Service;

    @GetMapping("{id}")
    public ResponseEntity<{{ tableInfo.name }}> queryById(@PathVariable("id") {{ pk.shortType }} id) {
        return ResponseEntity.ok(this.{{ tableInfo.nameLowerFirst }}Service.queryById(id));
    }
    // ...
}
```

### 全局变量系统

全局变量可在模板渲染前注入，具有**优先处理权限**。全局变量使用 `${varName}` 语法（与 Tera 的 `{{ }}` 区分），在 Phase 1 预处理阶段替换。

> ⚠️ **注意：** 全局变量具有优先处理权限，所以在模板中一定不要定义与全局变量同名的变量。

#### 使用案例一：简单模式
名称为变量名，内容为变量值。在模板中直接使用 `${变量名}` 引用变量。

```json
{
  "globalConfig": [
    {"name": "basePackage", "value": "com.example.demo"},
    {"name": "commonAuthor", "value": "developer@example.com"}
  ]
}
```

模板中使用：
```
package ${basePackage}.entity;
```

#### 使用案例二：复杂模式（模板嵌入）
名称为变量名，内容为模板内容。在模板任意位置使用 `${变量名}` 引入模板，引入的内容会被后续的 Tera 引擎进一步渲染。

```json
{
  "globalConfig": [
    {
      "name": "commonImports",
      "value": "import lombok.Data;\nimport lombok.EqualsAndHashCode;\nimport java.io.Serializable;"
    },
    {
      "name": "classAnnotation",
      "value": "@Data\n@EqualsAndHashCode(callSuper = false)"
    }
  ]
}
```

模板中使用：
```java
${commonImports}

${classAnnotation}
public class {{ tableInfo.name }} {
    // ...
}
```

#### 配置方式

全局变量可通过两种方式配置：

1. **命令行参数**：`--global-config global-config.json`
2. **数据模型 JSON 内嵌**：在数据模型 JSON 文件中添加 `globalConfig` 字段

```json
{
  "tableName": "User",
  "tableComment": "用户表",
  "globalConfig": [
    {"name": "basePackage", "value": "com.example"}
  ],
  "columns": [...]
}
```

### 内置变量

| 变量 | 说明 |
|------|------|
| `{{ tableInfo.name }}` | 表名（转换后的首字母大写类名，如 `User`） |
| `{{ tableInfo.comment }}` | 表注释 |
| `{{ tableInfo.preName }}` | 表前缀 |
| `{{ tableInfo.saveModelName }}` | 保存的 model 名称 |
| `{{ tableInfo.savePackageName }}` | 包名 |
| `{{ tableInfo.savePath }}` | 保存路径 |
| `{{ tableInfo.obj }}` | 表原始对象 |
| `{{ tableInfo.obj.name }}` | 原始表名（如 `sys_user`），常用于 SQL 语句中 |
| `{{ tableInfo.fullColumn }}` | 所有列 `List<ColumnInfo>` |
| `{{ tableInfo.pkColumn }}` | 主键列 `List<ColumnInfo>` |
| `{{ tableInfo.otherColumn }}` | 非主键列（除主键以外的列）`List<ColumnInfo>` |
| `{{ tableName }}` | 表名 + 表后缀（由 `#setTableSuffix` 宏注入）。若无 `#setTableSuffix` 则等于 `tableInfo.name` |
| `{{ tableInfoList }}` | 所有选中的表 `List<TableInfo>` |
| `{{ importList }}` | 所有需要导入的包集合 `Set<String>` |
| `{{ author }}` | 作者 |
| `{{ modulePath }}` | 模块路径 |
| `{{ projectPath }}` | 项目绝对路径 |
| `{{ serialUID }}` | 随机 serialVersionUID（预计算） |
| `{{ currentTime }}` | 当前时间（预计算，格式 `yyyy-MM-dd HH:mm:ss`） |

### ColumnInfo 列对象属性

| 属性 | 说明 |
|------|------|
| `column.name` | 列名（首字母小写驼峰，如 `userName`） |
| `column.comment` | 列注释 |
| `column.type` | 列类型（类型全名，如 `java.lang.String`）。模板中类型比较时用全名，如 `{% if column.type == "java.lang.String" %}` |
| `column.shortType` | 列类型（短类型，如 `String`） |
| `column.custom` | 是否附加列 `Boolean` |
| `column.ext` | 附加字段 `Map<String, Object>`（含 `jdbcType`, `nameUpperFirst`, `nameUnderline` 等） |
| `column.obj` | 列原始对象 |
| `column.obj.name` | 原始列名（数据库字段名，如 `user_name`），常用于 SQL 语句 |
| `column.obj.dataType` | 原始 JDBC 数据类型对象 |
| `column.obj.dataType.typeName` | JDBC 类型名称（如 `VARCHAR`、`BIGINT`、`TIMESTAMP`） |
| `column.obj.dataType.getLength()` | JDBC 类型长度（需通过 `$!tool.getField` 或方法调用获取） |
| `column.nameUpperFirst` | 列名首字母大写（预计算，在 `ext` 中） |
| `column.nameUnderline` | 列名驼峰转下划线（预计算，在 `ext` 中） |

## 数据模型格式

### 简化格式（推荐）

```json
{
  "tableName": "User",
  "tableComment": "用户表",
  "originalTableName": "sys_user",
  "packageName": "com.example.demo",
  "modulePath": "",
  "author": "developer",
  "columns": [
    {
      "name": "id",
      "comment": "主键ID",
      "type": "Long",
      "originalName": "id",
      "jdbcType": "BIGINT",
      "isPrimaryKey": true
    },
    {
      "name": "userName",
      "comment": "用户名",
      "type": "String",
      "originalName": "user_name",
      "jdbcType": "VARCHAR",
      "isPrimaryKey": false
    },
    {
      "name": "createTime",
      "comment": "创建时间",
      "type": "Date",
      "originalName": "create_time",
      "jdbcType": "TIMESTAMP",
      "isPrimaryKey": false
    }
  ]
}
```

### 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `tableName` | ✅ | 表名转换后的类名（首字母大写驼峰，如 `User`） |
| `tableComment` | 否 | 表注释 |
| `originalTableName` | 否 | 原始表名（如 `sys_user`），不提供则使用 `tableName` 的下划线形式 |
| `packageName` | ✅ | 包名（如 `com.example.demo`） |
| `modulePath` | 否 | 模块路径（多模块项目使用） |
| `author` | 否 | 作者名，默认为 `developer` |
| `globalConfig` | 否 | 全局变量配置数组，见 [全局变量系统](#全局变量系统) |
| `columns` | ✅ | 列定义数组 |

**列字段说明：**

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | ✅ | 列名（驼峰，如 `userName`） |
| `comment` | 否 | 列注释 |
| `type` | ✅ | 列类型，使用短类型名（如 `Long`、`String`、`Date`），参见 [类型映射配置](#类型映射配置) |
| `originalName` | 否 | 原始列名（数据库字段名，如 `user_name`），不提供则自动转换 |
| `jdbcType` | 否 | JDBC 类型（如 `BIGINT`、`VARCHAR`），用于 Mapper XML 的 `jdbcType` 属性 |
| `isPrimaryKey` | 否 | 是否主键，默认 `false` |
| `custom` | 否 | 是否附加列，默认 `false`。附加列不会出现在 Mapper XML 的 insert/update 语句中 |
| `ext` | 否 | 附加字段对象，可包含 `jdbcType`、`nameUpperFirst`、`nameUnderline` 等 |

### 类型映射配置

类型映射控制数据模型中 `columns[].type` 到 Java 类型（短类型 + 全限定名）的转换。映射配置独立于代码，通过 JSON 文件管理。

**配置文件加载优先级：**
1. `template/{模板组名}/type-mapping.json` — 模板组专属配置（优先）
2. `type-mapping.json` — 通用配置（回退）

**JSON 格式：**
```json
{
  "类型名": ["短类型", "全限定类型"],
  "Long": ["Long", "java.lang.Long"],
  "String": ["String", "java.lang.String"]
}
```

**通用配置默认支持的字段类型：**

| 类型 | 短类型 | 全限定类型 |
|------|--------|-----------|
| `Long` | `Long` | `java.lang.Long` |
| `Integer` | `Integer` | `java.lang.Integer` |
| `int` | `int` | `int` |
| `String` | `String` | `java.lang.String` |
| `Boolean` | `Boolean` | `java.lang.Boolean` |
| `boolean` | `boolean` | `boolean` |
| `Date` | `Date` | `java.util.Date` |
| `LocalDateTime` | `LocalDateTime` | `java.time.LocalDateTime` |
| `LocalDate` | `LocalDate` | `java.time.LocalDate` |
| `BigDecimal` | `BigDecimal` | `java.math.BigDecimal` |
| `Double` | `Double` | `java.lang.Double` |
| `double` | `double` | `double` |
| `Float` | `Float` | `java.lang.Float` |
| `float` | `float` | `float` |
| `Byte` | `Byte` | `java.lang.Byte` |
| `byte` | `byte` | `byte` |
| `Short` | `Short` | `java.lang.Short` |
| `short` | `short` | `short` |
| `Object` | `Object` | `java.lang.Object` |

> 💡 **自定义类型映射：** 在模板组目录下创建 `type-mapping.json`，添加该模板组需要的额外类型。模板组专属配置会覆盖通用配置中的同名类型。

## 从数据库生成数据模型（dbx MCP）

当需要为已有数据库表生成代码时，可以使用 `dbx` MCP 工具直接从数据库读取表结构，自动生成符合 `data-model.json` 格式的数据模型文件，无需手动编写。

### 工作流程

#### 1. 添加数据库连接

使用 `dbx_add_connection` 添加数据库连接：

```
run_mcp --server_name mcp_dbx --tool_name dbx_add_connection --args '{"name": "my-mysql", "db_type": "mysql", "host": "localhost", "port": 3306, "username": "root", "password": "123456", "database": "my_database"}'
```

支持的 `db_type`：`mysql`、`postgresql`、`sqlite`、`sqlserver`、`oracle` 等。

#### 2. 列出数据库中的表

使用 `dbx_list_tables` 查看可用的表：

```
run_mcp --server_name mcp_dbx --tool_name dbx_list_tables --args '{"connection_name": "my-mysql", "database": "my_database"}'
```

#### 3. 获取表结构信息

使用 `dbx_describe_table` 获取指定表的列定义：

```
run_mcp --server_name mcp_dbx --tool_name dbx_describe_table --args '{"connection_name": "my-mysql", "table": "sys_user", "database": "my_database"}'
```

返回结果包含每列的：列名（`name`）、数据类型（`type`）、是否可空（`nullable`）、是否主键（`primary_key`）、注释（`comment`）等信息。

#### 4. 转换为 data-model.json 格式

将 `dbx_describe_table` 返回的列信息转换为 `data-model.json` 格式：

| 来源字段 | 目标字段 | 转换规则 |
|---------|---------|---------|
| 表名（如 `sys_user`） | `originalTableName` | 保持原样 |
| 表名（如 `sys_user`） | `tableName` | 下划线转驼峰，首字母大写（如 `User`） |
| 表注释 | `tableComment` | 保持原样 |
| 用户指定 | `packageName` | 需用户手动提供（如 `com.example.demo`） |
| 用户指定 | `author` | 需用户手动提供 |
| 列名（如 `user_name`） | `columns[].originalName` | 保持原样 |
| 列名（如 `user_name`） | `columns[].name` | 下划线转驼峰，首字母小写（如 `userName`） |
| 列注释 | `columns[].comment` | 保持原样 |
| 列数据类型 | `columns[].type` | 按 JDBC 类型映射为 Java 短类型 |
| 列数据类型 | `columns[].jdbcType` | 映射为 JDBC 类型名 |
| 是否主键 | `columns[].isPrimaryKey` | 布尔值 |

#### JDBC 类型到 Java 类型映射

| JDBC 类型 | Java 短类型 | 全限定类型 |
|-----------|------------|-----------|
| BIGINT / BIGSERIAL | `Long` | `java.lang.Long` |
| INTEGER / INT / SERIAL | `Integer` | `java.lang.Integer` |
| SMALLINT / SMALLSERIAL | `Short` | `java.lang.Short` |
| TINYINT | `Byte` | `java.lang.Byte` |
| VARCHAR / CHAR / TEXT / LONGTEXT / MEDIUMTEXT | `String` | `java.lang.String` |
| TIMESTAMP / DATETIME | `Date` | `java.util.Date` |
| DATE | `LocalDate` | `java.time.LocalDate` |
| TIME | `LocalTime` | `java.time.LocalTime` |
| BOOLEAN / BIT / BOOL | `Boolean` | `java.lang.Boolean` |
| DECIMAL / NUMERIC / NUMBER | `BigDecimal` | `java.math.BigDecimal` |
| DOUBLE / DOUBLE PRECISION | `Double` | `java.lang.Double` |
| FLOAT / REAL | `Float` | `java.lang.Float` |
| BLOB / LONGBLOB / BYTEA | `byte[]` | `byte[]` |

> 💡 不同数据库的类型名称可能略有差异（如 MySQL 的 `INT` vs PostgreSQL 的 `INTEGER`），需要根据实际返回的类型名进行映射。

#### 5. 保存为 JSON 文件并执行生成

将转换后的 JSON 写入文件（如 `skills/code-generator/example/data-model.json`），然后按正常流程执行代码生成。

> 💡 **完整示例**：参见 [example/data-model.json](file:///f:/gitee-project/ai-template-code-generater-skills/skills/code-generator/example/data-model.json)。

## 执行流程

> ⚠️ **交互原则：** 每个需要用户决策的步骤，必须使用 `AskUserQuestion` 工具弹出交互式选择卡片，让用户点击选择，**不能只输出文本让用户打字回复**。

当用户触发此 Skill 时，严格按以下步骤操作：

### 步骤 1：列出模板组 + 收集用户选择

1. 运行 `./scripts/easyframe-codegen --list-groups` 获取可用模板组列表
2. 使用 **`AskUserQuestion`** 工具，弹出交互式选择卡片，包含两个问题：

   **问题 1 — 选择模板组**（单选）：
   - 选项格式：`Default (Recommended)` — Spring Boot + MyBatis 通用模板（含 MyBatis XML）
   - 选项格式：`MybatisPlus` — Spring Boot + MyBatis-Plus 模板

   **问题 2 — 输出目录**（文本输入）：
   - 让用户输入项目根目录的绝对路径（代码将生成到该目录下）
   - 示例：`d:\my-project`

### 步骤 2：确认数据模型

有两种方式获取数据模型：

**方式一：从数据库自动生成（推荐）**
如果用户有可访问的数据库，使用 dbx MCP 工具直接从数据库表结构生成数据模型 JSON。详见 [从数据库生成数据模型（dbx MCP）](#从数据库生成数据模型dbx-mcp)。

**方式二：手动创建数据模型**
1. 检查用户是否已有数据模型 JSON 文件
2. 如果有，使用 **`AskUserQuestion`** 确认是否使用现有文件
3. 如果没有，使用 **`AskUserQuestion`** 收集信息：
   - 表名、表注释
   - 包名（如 `com.example.demo`）
   - 作者名
   - 然后逐字段收集：字段名、类型、注释、是否主键等

### 步骤 3：执行生成

根据用户选择，运行二进制：
```bash
./scripts/easyframe-codegen \
  --group <用户选择的模板组> \
  --data <数据模型JSON路径> \
  --output <用户指定的输出目录>
```

### 步骤 4：验证输出

1. 列出生成的文件清单
2. 抽查 1-2 个关键文件（如 Entity、Controller），确认内容正确
3. 总结生成结果，告知用户

## 内置模板组

| 模板组 | 说明 | 生成文件 |
|--------|------|----------|
| `Default` | Spring Boot + MyBatis 通用模板 | Entity, Dao, Mapper XML, Service, ServiceImpl, Controller, Debug |
| `MybatisPlus` | Spring Boot + MyBatis-Plus 模板 | Entity, Dao, Service, ServiceImpl, Controller |

### 调试模板（debug.json.tera）

`Default` 模板组中包含 `debug.json.tera`，用于调试和了解数据模型结构。它不会生成输出文件（通过 `$!callback.setWriteFile(false)` 禁止写入），而是将数据模型对象结构打印到控制台。

```tera
// 禁止将生成结果写入到文件
$!callback.setWriteFile(false)

// 调试表原始对象
$!tool.debug($tableInfo.obj)

// 调试列原始对象
$!tool.debug($tableInfo.fullColumn.get(0).obj)

// 调试列原始列类型
$!tool.debug($tableInfo.fullColumn.get(0).obj.dataType)

// 获取原始列类型中的字段
sqlType = $!tool.getField($tableInfo.fullColumn.get(0).obj.dataType, "typeName")

// 执行原始列类型中的方法
sqlTypeLen = $!tableInfo.fullColumn.get(0).obj.dataType.getLength()
```

> 💡 调试模板展示了 `$!tool.debug()`、`$!tool.getField()` 和对象方法调用的用法，编写自定义模板时可参考。

## 添加新模板组

1. 在 `template/` 下创建新目录，如 `template/my-framework/`
2. 创建 `.tera` 模板文件（使用 Tera 语法 + EasyFrame 宏）
3. （可选）创建 `type-mapping.json` 定义模板组专属的类型映射，未配置则回退到通用 `type-mapping.json`
4. 运行 `./scripts/easyframe-codegen --list-groups` 验证新模板组是否可识别

模板文件命名规范：
- 文件名格式：`<类型>.<语言扩展名>.tera`，如 `entity.java.tera`、`mapper.xml.tera`
- 每个模板组只需 `.tera` 文件即可运行，`type-mapping.json` 为可选配置
- 建议添加 `debug.json.tera` 调试模板，便于排查数据模型问题

## 注意事项

1. **无需运行时环境**：编译产物为独立二进制文件，无需安装 Python、Node.js 等
2. **模板语法**：模板使用 Tera 语法（`{{ }}`、`{% %}`），不是 Velocity 语法
3. **EasyFrame 宏**：EasyFrame 特有宏由预处理器处理，在 Tera 渲染前转换，模板中保留原样
4. **预计算字段**：模板中不调用方法，所有工具方法结果已预计算为上下文字段
5. **路径处理**：自动创建输出目录
6. **编码**：所有文件使用 UTF-8 编码
7. **已有文件**：生成脚本会覆盖已存在的文件，请确认输出目录
8. **模板内容**：`template/` 目录下的模板文件不可修改，但可以添加新的模板组
9. **跨平台**：编译产物支持 Windows (.exe)、Linux 和 Mac