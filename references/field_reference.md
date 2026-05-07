# 报名字段参考文档

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