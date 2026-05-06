# 字段类型映射与结构规范

> 本文件包含报名字段的完整类型映射、JSON 结构规范、默认值规则。
> 供 `create_step_2`（首次生成）和 `edit_step_3`（字段修改）参考使用。

---

## 字段类型映射表

### 1. 文本输入类（Text）

适用于：姓名、学校、学号、公司、职务、地址等

```json
{
  "label": "字段名称",
  "type": "Text",
  "isRequire": true,
  "placeholder": "请输入XXX"
}
```

---

### 2. 上传类（MyUpload）

当字段包含"上传/提交文件"语义时使用

```json
{
  "label": "字段名称",
  "type": "MyUpload",
  "isRequire": true,
  "placeholder": "请上传XXX",
  "fileType": "<类型>"
}
```

**fileType 映射**：

| 用户描述关键词       | fileType 值  |
|---------------|-------------|
| 图片、照片、头像      | `picture`   |
| 视频、录像         | `video`     |
| 音频、录音         | `audio`     |
| 文档、表格、证明      | `doc`       |

---

### 3. 选择类（MySelect / MyRadio / MyCheckbox）

当字段包含"选择/选项"语义时使用

```json
{
  "label": "字段名称",
  "type": "MySelect",
  "isRequire": true,
  "placeholder": "请选择XXX"
}
```

---

### 4. 手机号码类（Mobile）

当字段包含"手机/手机号码/通讯号码/号码"语义时使用

```json
{
  "label": "字段名称",
  "type": "Mobile",
  "isRequire": true,
  "placeholder": "请输入XXX"
}
```

---

### 5. 身份证（IDCard）

当字段包含"身份证"语义时使用

```json
{
  "label": "字段名称",
  "type": "IDCard",
  "isRequire": true,
  "placeholder": "请输入XXX"
}
```

---

### 6. 介绍类（Textarea）

当字段包含"简介/介绍/描述"语义时使用

```json
{
  "label": "字段名称",
  "type": "Textarea",
  "isRequire": true,
  "placeholder": "请输入XXX"
}
```

---

### 7. 日期类（Date）

当字段包含"日期"语义时使用

```json
{
  "label": "字段名称",
  "type": "Date",
  "isRequire": true,
  "placeholder": "请选择XXX"
}
```

---

## 字段默认值规范

| 属性            | 默认值    | 说明                                                               |
|---------------|--------|------------------------------------------------------------------|
| `isRequire`   | `true` | 所有字段默认必填                                                         |
| `placeholder` | 按类型生成  | 文本类："请输入XXX"，上传类："请上传XXX"，选择类："请选择XXX"                           |
| `options`     | 按类型生成  | 对象数组：[{"label":"选项1","value":"1"},{"label":"选项2","value":"2"}]   |

---

## 字段结构示例

### 完整字段示例

```json
[
  {
    "label": "姓名",
    "type": "Text",
    "isRequire": true,
    "placeholder": "请输入姓名"
  },
  {
    "label": "手机号",
    "type": "Mobile",
    "isRequire": true,
    "placeholder": "请输入手机号"
  },
  {
    "label": "上传照片",
    "type": "MyUpload",
    "isRequire": true,
    "placeholder": "请上传照片",
    "fileType": "picture"
  },
  {
    "label": "兴趣爱好",
    "type": "MySelect",
    "isRequire": true,
    "placeholder": "请选择兴趣爱好",
    "options": [
      {"label": "阅读", "value": "1"},
      {"label": "运动", "value": "2"},
      {"label": "音乐", "value": "3"}
    ]
  }
]
```

---

## 字段提取规则

将用户提到的每个"收集信息项"转为字段：

| 用户描述             | 字段示例                                        |
|------------------|---------------------------------------------|
| "姓名"、"手机号"、"学号"  | `{"label": "姓名", ...}`                      |
| "上传照片"           | `{"label": "照片", "type": "MyUpload", ...}`  |
| "选择部门"           | `{"label": "部门", "type": "MySelect", ...}`  |
| "填写简介"           | `{"label": "简介", "type": "Textarea", ...}`  |
| "选择日期"           | `{"label": "日期", "type": "Date", ...}`      |
| "身份证号"           | `{"label": "身份证", "type": "IDCard", ...}`   |

---

## 字段数量限制

- 单次活动 `fields` 数量上限：**20 个**
- 若超过 20 个：提示用户精简，不得静默截断

---

## 特殊处理规则

### 情况 1：字段信息不完整（如缺少 options）

**必须询问用户**：
> "该字段有哪些可选项？"

**处理**：
- 用户补充前不得生成完整字段结构
- 可先记录字段名称，等待补充

### 情况 2：用户未提供任何字段

- `fields` 初始化为空数组：`[]`
- 由 `create_step_3` 兜底处理

### 情况 3：弱语义

如："填一些基本信息"、"正常报名信息就行"

👉 处理：
- ❌ 不直接生成字段
- ✅ 留给 `create_step_3` 处理（统一兜底）

---

## 编辑时的字段约束

### 已有字段修改规则

- **仅允许修改**：`label`、`type`、`placeholder`、`isRequire`、其他展示属性
- **必须保留**：`id`（不变）、`cid`（不变）
- **严禁**：删除或生成新的 `id`/`cid`、替换整个字段对象

### 新增字段规则

- 使用 `create_step_2` 规则生成结构
- ⚠️ **新增字段不得包含 `id` 和 `cid`**
- 追加到 `fields` 列表

### 字段定位规则

1. **精确匹配优先**：`label` 完全相同 → 直接定位
2. **模糊匹配阈值**：`label` 相似度 ≥ 70% 才视为匹配
3. **匹配失败 Fallback**：列出所有字段 `label`，请用户明确指定
4. **禁止行为**：
   - 不得仅凭情绪词或通用词（"信息"、"资料"、"字段"）定位字段
   - 不得匹配到多个字段时不加区分地全部修改

### 一致性校验

每次字段修改后校验：

| 校验项     | 要求                       |
|---------|--------------------------|
| 已有字段    | 必须包含 `id` 和 `cid`，且值不变   |
| 新增字段    | 不得包含 `id` 和 `cid`        |
| 整体结构    | 符合本文件格式规范                |
| 禁止      | 字段结构错乱、缺失关键属性、类型异常       |

---

## 默认字段兜底

当 `fields` 为空且用户未有效补充时，使用默认字段：

```json
[
  {
    "label": "姓名",
    "type": "Text",
    "isRequire": true,
    "placeholder": "请输入姓名"
  },
  {
    "label": "手机号",
    "type": "Mobile",
    "isRequire": true,
    "placeholder": "请输入手机号"
  }
]
```
