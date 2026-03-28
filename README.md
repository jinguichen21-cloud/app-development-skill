# 应用开发技能

通过 `dws aiapp` CLI 快速创建应用、小工具、业务系统。

## 适用场景

- **小工具**: 计算器、待办工具、表单应用、信息收集、报表应用
- **业务系统**: CRM、OA、库存管理、仓库管理、审批系统、进销存
- **通用应用**: 管理后台、数据录入工具、查询工具
- **图片输入**: 根据截图/照片创建类似系统
- **Excel输入**: 根据表格数据创建管理系统

## 快速开始

### 1. 新建应用

```bash
dws aiapp create \
  --prompt "帮我创建一个计算器应用" \
  --format json
```

### 2. 带附件创建

```bash
dws aiapp create \
  --prompt "根据这个截图创建类似的系统" \
  --attachments '[{"name":"截图.png","type":"image/png","url":"https://...","size":102400}]' \
  --format json
```

### 3. 修改应用

```bash
dws aiapp modify \
  --thread-id <从create返回中获取> \
  --prompt "把首页改得更简洁" \
  --format json
```

### 4. 查询任务状态

```bash
dws aiapp query \
  --task-id <从create/modify返回中获取> \
  --format json
```

## 技能特点

这是一个**通用基础技能**:
- 不需要特定的 `--skills` 参数
- 支持图片、Excel、文档等附件输入
- 可与其他技能组合使用

## 与其他技能组合

当需要更好的视觉效果时，可以组合视觉设计技能：

```bash
dws aiapp create \
  --prompt "创建界面美观的管理工具" \
  --skills official_b3f7380095ef4e31 \
  --format json
```

## 文件结构

```
app-development-skill/
├── SKILL.md                # AI Agent 指导文件
├── package.json            # 元数据
├── README.md               # 本文件
├── references/
│   ├── api-reference.md    # dws aiapp 命令完整参数
│   └── error-codes.md      # 错误码及处理方式
└── tests/
    └── testcases.json      # 评测用例
```

## 参考链接

- [dws aiapp 命令参考](./references/api-reference.md)
- [错误码说明](./references/error-codes.md)
