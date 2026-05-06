---
name: aixiu-activity-operation
description: 对话式运营助手，辅助创建、编辑爱秀活动。触发词（中文）：创建爱秀活动、创建报名活动、发布报名表、编辑爱秀活动、修改报名活动、修改活动字段、关联抽奖、活动报名；触发词（English）：create activity、create signup event、edit activity、modify registration form、link raffle、AIxiu activity。支持报名表单设计、海报配置、配色方案、抽奖关联等完整工作流。适用于快速创建报名活动、收集用户信息、管理活动信息的场景，支持与抽奖功能联动提升活动传播势能。
---

# 爱秀活动管理 Skill

## 功能概述

- 管理"爱秀活动"
- 自动解析用户自然语言中的活动类型与字段
- 支持预设模板和风格
- 支持多轮对话补充活动参数
- 提供完整的创建与编辑工作流

---

## 触发条件

当用户表达以下意图时激活本 Skill：

| 意图类型     | 触发关键词示例             |
|----------|---------------------|
| **创建活动** | 创建爱秀活动、创建报名活动、发布报名表 |
| **编辑活动** | 编辑爱秀活动、修改报名活动、修改报名表 |

---

## ⛔ 全局执行规则（最高优先级，通读后再执行任何操作）

> 以下规则贯穿全流程，任何步骤均不得违反。

### 输出规范
1. **禁止步骤标签**：严禁输出任何步骤标签（如「create_step_1:」「create_step_2:」）、步骤名称（如「模板匹配:」）或 Markdown 结构名称。
2. **输出 = 话术，仅话术**：每一步的输出必须且只能包含：展示数据（表格/文本）+ 提问。除此之外的任何内容（推导过程、执行说明、步骤状态）**一律不输出**。
3. **屏蔽执行细节**：AI 具体的执行过程和思考过程不对用户输出。

### 时区规范
1. **所有日期基于北京时间（UTC+8）**：如系统时区非 UTC+8，所有日期/时间操作必须先转换。

### 输出禁区（出现即违规）

以下句式严禁出现在任何输出中：

- "正在执行..."、"正在匹配..."、"正在处理..."
- "匹配到模板"、"匹配成功"、"已匹配"
- "我将..."、"让我..."、"接下来..."
- 任何步骤名称（create_step_X、step_X）
- 任何推导说明（"因为..."、"根据..."）

✅ 正确输出 = 直接展示结果 + 提问

---

## 工作流程概览

### 创建活动流程

```
create_step_1（解析基础信息）
       │
       ├─ 匹配到模板 → create_step_7（直接创建）
       │
       └─ 未匹配模板 → create_step_2（生成报名字段）
                         │
                         └─ create_step_3（补充报名字段）
                             │
                             └─ create_step_4（海报处理）
                                 │
                                 └─ create_step_5（风格与配色生成）
                                     │
                                     └─ create_step_6（确认活动）
                                         │
                                         └─ create_step_7（创建活动）
```

> 各步骤详细规则见 `references/create_workflow.md`

### 编辑活动流程

```
edit_step_1（获取活动信息 + 提取修改意图）
       │
       ├─ 已提取修改意图 → edit_step_3（修改意图解析与执行）
       │
       └─ 未提取修改意图 → edit_step_2（获取修改需求）
                             │
                             └─ edit_step_3（修改意图解析与执行）
                                 │
                                 └─ edit_step_4（确认修改活动）
                                     │
                                     └─ edit_step_5（提交更新）
```

> 各步骤详细规则见 `references/edit_workflow.md`

---

## 工具列表与调用方式

| 工具/脚本             | 功能说明                   | 参考文档                            |
|-------------------|------------------------|---------------------------------|
| `create_event.py` | 创建活动，读取临时文件并调用后端接口     | `references/create_workflow.md` |
| `update_event.py` | 更新活动信息，读取临时文件并调用后端接口   | `references/edit_workflow.md`   |
| `get_activity.py` | 获取活动详情，返回完整活动配置 JSON   | `references/edit_workflow.md`   |
| `designh5.py`     | 报名活动字段映射、模板匹配等         | `references/field_reference.md` |
| `raffle.py`       | 抽奖功能处理逻辑               | `references/create_workflow.md` |
| `helper.py`       | 公共辅助函数：用户ID读取、配置加载、日志等 | -                               |

### 调用方式

```bash
# 创建活动
python scripts/create_event.py '{"temp_file_path": "xxx"}'

# 更新活动
python scripts/update_event.py '{"temp_file_path": "xxx"}'

# 获取活动详情
python scripts/get_activity.py '{"act_id": "xxx"}'
```

---

## 数据模型

### 活动对象（Activity）

```
Activity
├── 基本信息：act_type, title, brief, start_time, end_time
├── 模板信息：template_id, mark
├── 海报：post_img { url, size, desc_str }
├── 配色：scheme { page_config, form, title, text, long_text }
├── 报名字段：fields[] { label, type, id, cid, placeholder, isRequire, options }
└── 抽奖：is_raffle (0/1)
```

### 核心嵌套关系

- `fields[]`：报名字段数组，每个字段含 `label`（显示名）、`type`（类型）、`id`（前端标识）、`cid`（后端关联ID）
- `post_img`：海报结构，固定 `size: "818*1404"`
- `scheme`：配色配置，所有颜色为 RGBA 格式

> 字段类型映射及结构规范详见 `references/field_reference.md`

---

## 常见工作流

### 创建活动

1. **create_step_1**：解析用户输入，提取 `act_type`/`title`/`brief`/`start_time`/`end_time`/`fields`/`is_raffle`
2. **模板匹配**：遍历 `config/designh5/template_list.json`，按关键词匹配模板
   - 匹配成功（≥1个）→ 展示信息 → 确认后直接 **create_step_7**
   - 匹配失败（0个）→ **create_step_2**
3. **create_step_2**：首次生成 `fields`（若不存在）
4. **create_step_3**：补充/兜底字段（最多一轮询问）
5. **create_step_4**：海报处理（询问链接或生成描述）
6. **create_step_5**：配色生成（参考 `template/designh5.json`）
7. **create_step_6**：数据预校验 → 展示活动数据 → 等待确认
8. **create_step_7**：调用 `create_event.py` 创建 → 展示结果 → 询问关联抽奖

> 详细规则、分支处理、约束条件见 `references/create_workflow.md`

### 编辑活动

1. **edit_step_1**：获取活动信息（ID/链接/标题定位）→ 提取修改意图 → 保存 `original_data`
   - 已提取修改意图 → 跳过 edit_step_2，直接进入 edit_step_3
   - 未提取 → 进入 edit_step_2
2. **edit_step_2**：收集用户修改需求
3. **edit_step_3**：解析修改意图（标题/介绍/时间/海报/配色/字段/抽奖）→ 逐项执行 → 循环直到用户结束
   - **强制约束**：修改完成后必须进入 edit_step_4 确认，严禁直接提交
4. **edit_step_4**：对比 `original_data` 展示变更 → 数据预校验 → 等待用户确认
5. **edit_step_5**：调用 `update_event.py` 提交更新 → 展示结果

> 详细规则、字段修改约束、校验逻辑见 `references/edit_workflow.md`

---

## 模板匹配规则（精简版）

**数据来源**：`config/designh5/template_list.json`

**匹配逻辑**：
1. 提取用户 `brief` 中的实义词（去除停用词）为关键词集合
2. 遍历模板 `des` 字段，计算匹配词数量
3. 匹配词数量 ≥ 1 即为匹配到，选匹配最多的模板
4. 匹配成功 → 提取 `template_id` + `mark` → 直接进入 create_step_7
5. 匹配失败 → 进入 create_step_2

> ⚠️ 只做关键词字面匹配，不做语义联想。详细规则见 `references/create_workflow.md`

---

## 字段类型速查

| 类型                                | 用途    | 示例                    |
|-----------------------------------|-------|-----------------------|
| `Text`                            | 文本输入  | 姓名、学校、学号              |
| `Mobile`                          | 手机号码  | 手机号、联系方式              |
| `IDCard`                          | 身份证   | 身份证号                  |
| `MyUpload`                        | 文件上传  | 照片、文档（含 `fileType`）   |
| `MySelect`/`MyRadio`/`MyCheckbox` | 选择类   | 单选、多选、下拉（含 `options`） |
| `Textarea`                        | 多行文本  | 简介、建议                 |
| `Date`                            | 日期选择  | 出生日期、入职日期             |

> 完整字段结构、默认值规范、fileType 映射见 `references/field_reference.md`

---

## 常见错误码

| 错误场景      | 处理方式                    |
|-----------|-------------------------|
| 活动未查询到    | 确认活动ID或链接后重试            |
| 字段修改未生效   | 确保已在 edit_step_4 确认修改   |
| 配色未变化     | 清除浏览器缓存后重试              |
| 删除字段后异常   | 删除前系统会要求确认，删除不可逆        |
| 模板匹配失败    | 进入 create_step_2 手动补充字段 |
| 活动ID重复    | 提示"活动ID已存在，请检查后重试"      |
| 图片URL无效   | 提示"海报图片链接无效，请重新提供"      |
| 字段类型冲突    | 提示"字段类型冲突，请明确指定字段类型"    |
| 接口超时/网络异常 | 提示"网络异常，请稍后重试"，不得重复提交   |

---

## 资源引用说明

### scripts/ 目录

| 文件                 | 用途                                                  |
|--------------------|-----------------------------------------------------|
| `create_event.py`  | 创建活动，读取临时文件并调用后端接口，参数：`{"temp_file_path": "xxx"}`   |
| `update_event.py`  | 更新活动信息，读取临时文件并调用后端接口，参数：`{"temp_file_path": "xxx"}` |
| `get_activity.py`  | 获取活动详情，返回完整活动配置JSON，参数：`{"act_id": "xxx"}`          |
| `designh5.py`      | 报名活动（designh5）相关处理逻辑，包含字段映射、模板匹配等                   |
| `raffle.py`        | 抽奖功能相关处理逻辑，包含抽奖状态更新、权益配置等                           |
| `helper.py`        | 公共辅助函数：用户ID读取（`~/.aixiu_user_id`）、配置加载、日志等          |

### config/ 目录

| 文件                            | 用途                                |
|-------------------------------|-----------------------------------|
| `designh5/template_list.json` | 报名活动模板列表，用于 `create_step_1` 的模板匹配 |

### template/ 目录

| 文件              | 用途                                  |
|-----------------|-------------------------------------|
| `designh5.json` | 报名活动页面配置示例，用于 `create_step_5` 的配色参考 |

---

## 注意事项

- **title 长度**：2-15 字，超出强制截断，不足须询问补充
- **brief 长度**：50-150 字，超出须提示精简
- **fields 上限**：单次活动最多 20 个字段
- **id/cid 保护**：编辑时严禁修改或丢失已有字段的 `id` 和 `cid`
- **update_logic 整体覆盖**：每次调用 `update_event.py` 会覆盖所有已有逻辑，追加需先获取当前数据
- **非幂等写操作**：`create_event.py` 每次调用创建新活动，`update_event.py` 每次调用覆盖更新
- **scheme 默认值**：未经过配色生成步骤时，保持默认值 `{}`
