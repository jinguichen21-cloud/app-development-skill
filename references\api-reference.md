# dws aiapp 命令参考

> 完整的 `dws aiapp` 命令参数说明，供 AI Agent 按需加载。

## 命令概览

| 命令 | 用途 |
|------|------|
| `dws aiapp create` | 新建 AI 应用 |
| `dws aiapp modify` | 修改已有 AI 应用 |
| `dws aiapp query` | 查询任务状态 |

---

## dws aiapp create

新建 AI 应用。

### 语法

```bash
dws aiapp create \
  --prompt <用户描述> \
  --format json \
  [--attachments <附件信息>] \
  [--skills <技能ID列表>]
```

### 参数说明

| 参数 | 必填 | 说明 |
|------|:---:|------|
| `--prompt` | ✅ | 用户对应用的描述，自然语言 |
| `--format json` | ✅ | 输出格式，必须为 json |
| `--attachments` | ❌ | 附件信息 (仅 create 支持，见下方详细说明) |
| `--skills` | ❌ | 技能 ID，多个用逗号分隔 (可选，用于组合其他技能) |

### 附件参数格式

`--attachments` 参数值为 JSON 数组，每个对象必须包含以下 4 个字段：

| 字段 | 类型 | 必填 | 说明 |
|------|------|:---:|------|
| `name` | string | ✅ | 文件名，如 `"截图.png"` |
| `type` | string | ✅ | MIME 类型 |
| `url` | string | ✅ | 文件访问 URL，必须是可公开访问的地址 |
| `size` | number | ✅ | 文件大小 (字节) |

**正确示例:**

```bash
# 单个附件 (图片)
--attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400}]'

# 单个附件 (Excel)
--attachments '[{"name":"数据表.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]'

# 多个附件 (图片 + Excel)
--attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400},{"name":"数据表.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]'
```

**支持的文件类型:**

| 类型 | MIME 类型 | 说明 |
|------|----------|------|
| PNG 图片 | `image/png` | 截图、原型图、流程图 |
| JPEG 图片 | `image/jpeg` | 照片、截图 |
| Excel 表格 | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | 数据表、需求表 |
| PDF 文档 | `application/pdf` | 需求文档、PRD |
| Word 文档 | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | 需求说明 |

**错误示例:**

```bash
# [错误] 缺少必填字段 (缺少 type 和 size)
--attachments '[{"name":"截图.png","url":"https://..."}]'

# [错误] 使用简写类型而非 MIME 类型
--attachments '[{"name":"截图.png","type":"png","url":"https://...","size":102400}]'

# [错误] 不是 JSON 数组格式
--attachments "截图.png"
```

### 返回值

```json
{
  "success": true,
  "data": {
    "threadId": "thread_abc123",
    "taskId": "task_xyz789",
    "status": "running",
    "threadViewUrl": "https://example.dingtalk.com/app/xxx"
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `threadId` | string | 会话标识，用于后续 modify |
| `taskId` | string | 任务标识，用于 query 查询状态 |
| `status` | string | 任务状态: `running`, `completed`, `failed` |
| `threadViewUrl` | string | 应用访问链接 |

### 示例

```bash
# 基础创建
dws aiapp create \
  --prompt "帮我创建一个计算器应用" \
  --format json

# 带截图创建
dws aiapp create \
  --prompt "根据这个截图创建类似的系统" \
  --attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400}]' \
  --format json

# 带 Excel 创建
dws aiapp create \
  --prompt "根据这个表格创建管理系统" \
  --attachments '[{"name":"数据表.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]' \
  --format json

# 组合视觉设计技能
dws aiapp create \
  --prompt "创建界面美观的管理工具" \
  --skills official_b3f7380095ef4e31 \
  --format json
```

---

## dws aiapp modify

修改已有 AI 应用。

> **注意**: modify 命令**不支持**附件参数，如需更换素材请重新 create。

### 语法

```bash
dws aiapp modify \
  --thread-id <threadId> \
  --prompt <修改要求> \
  --format json \
  [--skills <技能ID列表>]
```

### 参数说明

| 参数 | 必填 | 说明 |
|------|:---:|------|
| `--thread-id` | ✅ | 会话标识，从 create 返回中获取 |
| `--prompt` | ✅ | 修改要求，自然语言 |
| `--format json` | ✅ | 输出格式，必须为 json |
| `--skills` | ❌ | 技能 ID (可选) |

### 返回值

```json
{
  "success": true,
  "data": {
    "threadId": "thread_abc123",
    "taskId": "task_new456",
    "status": "running"
  }
}
```

### 示例

```bash
dws aiapp modify \
  --thread-id thread_abc123 \
  --prompt "把首页改得更简洁" \
  --format json
```

---

## dws aiapp query

查询任务执行状态。

### 语法

```bash
dws aiapp query \
  --task-id <taskId> \
  --format json
```

### 参数说明

| 参数 | 必填 | 说明 |
|------|:---:|------|
| `--task-id` | ✅ | 任务标识，从 create/modify 返回中获取 |
| `--format json` | ✅ | 输出格式，必须为 json |

### 返回值

```json
{
  "success": true,
  "data": {
    "taskId": "task_xyz789",
    "status": "completed",
    "result": {
      "threadViewUrl": "https://example.dingtalk.com/app/xxx"
    }
  }
}
```

### 示例

```bash
dws aiapp query \
  --task-id task_xyz789 \
  --format json
```

---

## 上下文传递链

```
create → 返回 threadId, taskId, threadViewUrl
   ↓
modify (使用 threadId) → 返回新 taskId
   ↓
query (使用 taskId) → 返回执行结果
```

| 操作 | 提取字段 | 传递给 |
|------|---------|--------|
| `create` | `threadId` | `modify --thread-id` |
| `create` | `taskId` | `query --task-id` |
| `modify` | `taskId` | `query --task-id` |
