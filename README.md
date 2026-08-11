# AI Template Code Generator Skills — 代码生成工具链

基于 Tera 模板引擎的 AI 驱动代码生成工具链，包含两个核心 Skill：

| Skill | 用途 | 方向 | 详细文档 |
| --- | --- | --- | --- |
| **code-generator** | 基于模板生成代码 | 模板 → 代码 | [SKILL.md](skills/code-generator/SKILL.md) |
| **template-generator** | 从现有项目提炼模板 | 代码 → 模板 | [SKILL.md](skills/template-generator/SKILL.md) |

* * *

 [![Gitee Stars](https://gitee.com/pretend-work/ai-template-code-generater-skills/badge/star.svg?theme=dark)](https://gitee.com/pretend-work/ai-template-code-generater-skills)[![Gitee Forks](https://gitee.com/pretend-work/ai-template-code-generater-skills/badge/fork.svg?theme=dark)](https://gitee.com/pretend-work/ai-template-code-generater-skills)

> 🚀 **如果这个项目对你有帮助，请在 [Gitee](https://gitee.com/pretend-work/ai-template-code-generater-skills) 上给个 Star ⭐，让更多人看到！** 你的支持是我们持续改进的动力！

## 工作流程

```
现有项目代码 ──[template-generator 提炼]──▶ .tera 模板文件
                                                  │
              code-generator 的 template/ 目录 ◀───┘
                                                  │
        数据模型 JSON ──[code-generator 生成]──────▶ 新项目代码
```

* * *

# code-generator

基于 [Tera](https://keats.github.io/tera/) 模板引擎的通用代码生成器。编译为单二进制文件，无运行时依赖，支持 Windows/Linux/Mac。

> 📖 完整语法、变量参考、EasyFrame 宏说明见 [code-generator/SKILL.md](skills/code-generator/SKILL.md)

## 快速开始

### 1\. 准备数据模型

```json
{
  "tableName": "User",
  "tableComment": "用户表",
  "originalTableName": "sys_user",
  "packageName": "com.example.demo",
  "author": "developer",
  "columns": [
    { "name": "id", "comment": "主键ID", "type": "Long", "originalName": "id", "jdbcType": "BIGINT", "isPrimaryKey": true },
    { "name": "userName", "comment": "用户名", "type": "String", "originalName": "user_name", "jdbcType": "VARCHAR", "isPrimaryKey": false }
  ]
}
```

### 2\. 运行生成

```bash
# 列出可用模板组
./scripts/easyframe-codegen --list-groups

# 预览模式
./scripts/easyframe-codegen --group Default --data user-model.json --output ./output --dry-run

# 生成代码
./scripts/easyframe-codegen --group Default --data user-model.json --output ./output
```

### 3\. 在 AI 对话中使用

直接提出需求，AI 会自动调用 Skill：

-   "根据 user-model.json 生成代码，保存到 d:\\my-project"

## 命令行参数

| 参数 | 简写 | 说明 |
| --- | --- | --- |
| `--list-groups` | — | 列出所有可用模板组 |
| `--group` | `-g` | 模板组名称（`Default` / `MybatisPlus`） |
| `--data` | `-d` | 数据模型 JSON 文件路径 |
| `--output` | `-o` | 输出根目录（默认 `./output`） |
| `--global-config` | — | 全局变量配置 JSON 文件路径 |
| `--dry-run` | — | 仅预览，不写入文件 |

## 内置模板组

| 模板组 | 技术栈 | 生成文件 |
| --- | --- | --- |
| `Default` | Spring Boot + MyBatis | Entity、Dao、Mapper XML、Service、ServiceImpl、Controller |
| `MybatisPlus` | Spring Boot + MyBatis-Plus | Entity、Dao、Service、ServiceImpl、Controller |

## AI 交互流程

1.  **列出模板组 + 收集选择**：选择模板组和输出目录
2.  **确认数据模型**：有现成 JSON 则复用，否则逐字段收集
3.  **执行生成**：运行二进制生成代码
4.  **验证输出**：列出文件清单，抽查关键文件

## 添加新模板组

1.  在 `template/` 下创建新目录，放入 `.tera` 模板文件
2.  （可选）创建 `type-mapping.json` 定义专属类型映射
3.  运行 `--list-groups` 验证

* * *

# template-generator

从现有项目代码中反向提炼 Tera 模板，为 code-generator 提供模板支持。支持 Java/Spring Boot、Vue2、Vue3。

> 📖 完整提炼规则、各文件详细步骤、检查清单见 [template-generator/SKILL.md](skills/template-generator/SKILL.md)

## 提炼原理

**参数化替换**：将项目中具体的类名、字段名、包名替换为模板变量和 EasyFrame 宏。

| 层级 | 示例 |
| --- | --- |
| 表级参数 | `{{ tableInfo.name }}`、`{{ tableInfo.comment }}` |
| 列级参数 | `{{ column.name }}`、`{{ column.shortType }}` |
| 全局参数 | `${basePackage}`、`${commonImports}` |

## 支持的技术栈

| 技术栈 | 可提炼的模板 |
| --- | --- |
| **Java/Spring Boot** | entity、controller、service、serviceImpl、dao、mapper.xml、type-mapping.json |
| **Vue2** | page.vue、component.vue、api.js、store.js、router.js |
| **Vue3** | page.vue3、component.vue、api.ts、store.ts、router.ts |

## AI 交互流程

1.  **确认提炼目标**：选择技术栈、源项目路径、目标模板组名称
2.  **扫描源项目**：扫描项目结构，识别可提炼的模板文件
3.  **选择生成模板**（人机交互）：
    -   常规文件（多选）：entity、controller、page.vue 等
    -   类型配置文件（单选）：是否生成 type-mapping.json
4.  **逐文件提炼**：参数化替换，生成 `.tera` 文件
5.  **验证模板**：`--list-groups` 确认可识别，`--dry-run` 预览结果

## 提炼示例

**源文件 `User.java`** → **`entity.java.tera`**：

```java
// 源文件
@Data
public class User implements Serializable {
    private Long id;
    private String userName;
}
```

```tera
// 提炼后模板
#save("/entity", ".java")
#setPackageSuffix("entity")
#tableComment("实体类")
@Data
public class {{ tableInfo.name }} implements Serializable {
{% for column in tableInfo.fullColumn %}
    private {{ column.shortType }} {{ column.name }};
{% endfor %}
}
```

* * *

## 👨‍💻 作者

| 作者 | 联系微信 | 联系方式 |
| --- | --- | --- |
| **city-space** | zgr-zhonguo | [121051390@qq.com](mailto:121051390@qq.com) |

> 💡 欢迎提交 Issue 和 PR，一起完善这个工具链！如果你有好的模板组，也欢迎贡献到项目中。

## 📱 交流群

| 个人微信 | QQ群 |
| --- | --- |
| ![个人微信](share/weixingeren.jpg) | ![QQ群](share/qqqun.jpg) |

* * *

# 目录结构

```
f:\code\ai-code-generater\
├── readme.md
├── .trae\skills\
│   ├── code-generator\
│   │   ├── SKILL.md                    # 详细文档
│   │   ├── scripts\                    # 编译好的二进制
│   │   ├── type-mapping.json           # 通用类型映射
│   │   ├── template\                   # 模板组
│   │   │   ├── Default\                # Spring Boot + MyBatis
│   │   │   └── MybatisPlus\            # Spring Boot + MyBatis-Plus
│   │   └── example\data-model.json     # 示例数据模型
│   └── template-generator\
│       ├── SKILL.md                    # 详细文档
│       └── examples\                   # 提炼示例
```