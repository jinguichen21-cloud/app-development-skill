---
name: app-development-skill
description: 通用应用开发技能。当用户需要创建应用、小工具、业务系统、管理后台时使用。不要在仅需官网、活动页面、品牌展示时触发。
---

# 应用开发技能

从想法到实现，一句话随口说，一张图随手拍/随手截，一个Excel随手传。均能根据你的需求，为你定制开发你的系统。

## 严格禁止 (NEVER DO)

- **不要**等待用户再次确认，选择此技能后直接执行 `dws aiapp` 命令
- **不要**忘记 `--format json` 参数
- **不要**编造 threadId，必须从之前的返回中提取
- **不要**在 query 命令中附加多余参数
- **不要**在 modify 命令的 `--attachments` 参数中使用非图片类型附件，modify 仅支持图片附件
- **不要**把"轮询查询状态"理解成自动化/定时任务/周期任务/cron/background job
- **不要**调用任何内置 automation、定时任务、提醒、计划任务能力来代替当前会话内的轮询
- **不要**在创建完成前结束当前轮询流程，除非任务已经 succeeded / failed
- **不要**在错误工作目录下直接运行 `./scripts/poll_aiapp_status.py`
- **不要**省略定位 skill 根目录这一步；必须先拿到脚本绝对路径再执行
- **不要**用 `python` 直接执行脚本；统一使用 `uv run python`
- **不要**为 `query` 编造 `verbose` 等未声明参数

## 命令总览

| 命令 | 用途 | 必填参数 | 可选参数 |
|------|------|---------|----------|
| `dws aiapp create` | 新建应用 | `--prompt`, `--format json` | `--attachments`, `--skills` |
| `dws aiapp modify` | 修改已有应用 | `--thread-id`, `--prompt`, `--format json` | `--skills`, `--attachments`（仅图片） |
| `dws aiapp query` | 查询任务状态 | `--task-id`, `--format json` | - |
| `uv run python <绝对脚本路径>` | 当前会话内长轮询状态 | - | - |

> **注意**: `modify` 命令仅支持**图片类型**附件，`create` 支持所有类型附件

## 意图判断决策树

```text
用户说 "应用/系统/工具/后台/页面":
├─ 创建/做一个/帮我做/新建 → dws aiapp create
│   ├─ 用户上传了图片/Excel/文档 → 添加 --attachments
│   └─ 无附件 → 不加 --attachments
├─ 修改/改一下/优化 → dws aiapp modify [需要 threadId]
│   ├─ 用户提供了图片 → 添加 --attachments（仅限图片类型）
│   └─ 用户提供了非图片附件 → 提示 modify 仅支持图片，非图片附件请重新创建
└─ 查询/看进度/状态/创建好了没 → 先 `dws aiapp query` 获取最新状态 → 再启动 Python 长轮询直到完成

用户说 "CRM/OA/库存/仓库/审批/进销存":
├─ 新建 → dws aiapp create
└─ 改一下 → dws aiapp modify [需要 threadId]

用户上传图片/Excel:
├─ 创建意图 → dws aiapp create + --attachments
└─ 修改意图 + 图片 → dws aiapp modify + --attachments（仅图片）
    └─ 修改意图 + 非图片 → 提示用户 modify 仅支持图片附件

关键区分:
- 无 threadId + 创建意图 → create
- 有 threadId + 修改意图 → modify
- 有 threadId + 修改意图 + 图片 → modify + --attachments（仅限图片）
- 有 threadId + 修改意图 + 非图片附件 → 提示用户 modify 仅支持图片附件
- 有 taskId + 查询意图 → 定位 skill 脚本绝对路径后启动 `uv run python` 轮询脚本
```

## 核心工作流

```bash
# 场景1: 新建 (无附件)
dws aiapp create --prompt "<用户描述>" --format json
# → 返回: threadId, taskId, threadViewUrl

# 场景1b: 新建 (带图片/Excel/文档，支持多附件)
dws aiapp create --prompt "<描述>" --attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400},{"name":"数据表.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]' --format json

# 场景2: 修改 [HARD-GATE: 必须有 threadId] [仅支持图片附件]
dws aiapp modify --thread-id <从create返回> --prompt "<修改要求>" --format json

# 场景2b: 修改 (带图片附件)
dws aiapp modify --thread-id <从create返回> --prompt "<修改要求>" --attachments '[{"name":"新截图.png","type":"image/png","url":"https://...","size":102400}]' --format json

# 场景3: 查询
dws aiapp query --task-id <从create/modify返回> --format json

# 场景4: 定位 skill 脚本绝对路径，再启动长轮询脚本
# 先拿到 skill 根目录，再拼出绝对脚本路径:
# <skill_root>/scripts/poll_aiapp_status.py
TASK_ID="<taskId>" THREAD_ID="<threadId>" THREAD_VIEW_URL="<threadViewUrl>" \
uv run python "<skill_root>/scripts/poll_aiapp_status.py"
```

## 工具边界

- 本技能里的"轮询"指的是**当前会话窗口内的前台阻塞轮询**
- 这不是自动化任务，也不是提醒事项，不允许创建 recurring automation
- 只要需要连续查询直到完成，必须执行 skill 包里的 `scripts/poll_aiapp_status.py`
- 必须先解析 skill 根目录，再用脚本绝对路径执行，不要假设当前目录就是 skill 目录
- 统一使用 `uv run python <绝对脚本路径>`
- 如果宿主环境支持内置 automation / reminder / schedule，也**一律不要使用**

## 创建后立即返回的信息

执行 `create` 或 `modify` 后，先立刻把下面信息返回给用户，再进入后续轮询。**这是硬门槛，没发出这条用户可见回复之前，不要继续执行 `query`、定位脚本路径或启动轮询脚本。**

- `threadViewUrl`：最重要，必须优先展示，告诉用户这是应用创建过程链接
- 当前状态：如果返回里已有 `status`，一并展示；没有就说明"任务已启动"
- `threadId`、`taskId` 只用于 agent 内部后续调用，不要在普通回复里展示给用户，除非用户明确索要

## 创建后首条回复模板

拿到 `create` 返回后，必须立刻发送一条普通回复，至少包含以下内容：

```text
任务已启动。
应用创建过程链接: <threadViewUrl>
任务状态: <status>
```

规则：

- `threadViewUrl` 必须出现在普通回复正文里，不能只停留在工具输出或折叠面板里
- 这条回复必须发生在任何后续 `query`、脚本定位、脚本启动之前
- 如果返回里已有 `threadViewUrl`，不要再额外查询一次才回复用户
- 只有在 `create` 返回里确实缺少 `threadViewUrl` 时，才允许先执行一次 `query` 补取
- `threadId`、`taskId` 仅保留在内部上下文，不要主动展示给用户

如果 `create` 返回里暂时没有 `threadViewUrl`，立刻补一次：

```bash
dws aiapp query --task-id <taskId> --format json
```

拿到 `threadViewUrl` 后再向用户汇报。

## 上下文传递表

| 操作 | 从返回中提取 | 用于 |
|------|-------------|------|
| `create` | `threadId` | `modify --thread-id` |
| `create` | `taskId` | `query --task-id` |
| `modify` | `taskId` | `query --task-id` |

## 数据格式

```bash
# [正确] 完整参数
dws aiapp create --prompt "帮我创建一个计算器应用" --format json

# [正确] modify 带图片附件
dws aiapp modify --thread-id abc123 --prompt "换一张新的截图" --attachments '[{"name":"新截图.png","type":"image/png","url":"https://...","size":102400}]' --format json

# [错误] 缺少 --format json
dws aiapp create --prompt "帮我创建一个计算器应用"

# [错误] modify 带非图片附件（不支持）
dws aiapp modify --thread-id abc123 --prompt "更新数据表" --attachments '[{"name":"数据.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]' --format json
```

### 附件参数格式 (--attachments)

JSON 数组，每个对象必须包含 `name`/`type`/`url`/`size` 四个字段，支持多附件：

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 文件名，如 `"截图.png"` |
| `type` | string | MIME 类型，如 `image/png`、`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` |
| `url` | string | 文件访问 URL |
| `size` | number | 文件大小 (字节) |

```bash
# 单个附件
--attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400}]'

# 多附件
--attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400},{"name":"数据表.xlsx","type":"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","url":"https://...","size":51200}]'
```

> 详见 [api-reference.md](./references/api-reference.md#附件参数格式)

## 轮询查询应用状态

执行 `create` 或 `modify` 后，必须在当前活跃窗口继续跟踪任务。轮询必须通过 `scripts/poll_aiapp_status.py` 执行，使用 `uv run python <绝对脚本路径>` 启动。脚本内部每 **30 秒**调用一次 `dws aiapp query --task-id <taskId> --format json`，每 **300 秒**输出一次心跳，最长跟踪 **60 分钟**。

> 详见 [references/polling-guide.md](./references/polling-guide.md)

## 错误处理

| 错误场景 | 处理方式 |
|---------|---------|
| 无 threadId 但要 modify | 询问用户是否新建，或请用户提供之前的会话 |
| 返回超时 | 使用 `query --task-id` 查询状态 |
| 任务失败 | 展示错误信息给用户，询问是否重试 |
| 权限错误 | 提示用户检查登录状态 |

## 详细参考 (按需读取)

- [dws aiapp 命令参考](./references/api-reference.md)
- [错误码说明](./references/error-codes.md)

## 触发关键词

应用/系统/工具/后台/页面/CRM/OA/库存/仓库/审批/进销存/计算器/待办/表单/报表/截图/Excel/表格

## 与其他技能合并

当用户需求包含"美观/好看/界面优化"时，附加视觉设计技能:

```bash
dws aiapp create --prompt "创建界面美观的管理工具" --skills official_b3f7380095ef4e31 --format json
```

## 结果展示

执行命令后，汇总展示: 应用名称（使用 `threadTitle`）、任务状态、threadViewUrl（应用创建过程链接，必须在正文展示）、appPreviewUrl（成功后展示）。`threadId`、`taskId` 仅保留给 agent 内部使用，不主动展示给用户。
