# Java Backend Fusion Specification

面向 AI 编码代理的 Java 后端统一规范 skill，用于指导 Java 后端项目的设计、实现、重构和代码审查。

本仓库是一个 Codex skill 目录。AI 代理实际读取和触发的是 [SKILL.md](SKILL.md)，`README.md` 只作为人类维护者和使用者的入口说明。

## 适用场景

当 AI 编码代理需要处理以下任务时，使用本 skill：

- 设计新的 Java 后端项目结构或模块边界
- 修改、重构既有 Java 后端代码
- 审查 Java 后端 PR 或已有实现
- 统一 Controller、Service、Repo、Domain、Client 等分层职责
- 规范命名、注释、校验、异常、日志、事务、幂等、并发和交付检查
- 为模块 README、ADR、PR 自检、代码评审和上线检查提供统一模板

本 skill 的核心目标不是让代理机械套模板，而是在明确架构边界和命名基线后，交付符合项目上下文的专家级实现。

## 核心约定

默认顶层模块优先使用：

```text
infrastructure
persistence
business
web
```

当项目存在 HTTP、MQ、定时任务、Webhook 等多种入站入口时，使用：

```text
interfaces
```

当第三方集成规模足够大时，可额外引入：

```text
integration
```

关键边界：

- `web` / `interfaces` 只负责入口适配、参数校验、Assembler 和统一响应
- `business` 负责编排业务用例、事务、Repo 调用和领域协作
- `domain` 承载强业务规则、状态机、策略和不变量校验
- `persistence` 只负责数据访问语义，不直接翻译业务异常
- `infrastructure/client` 或 `integration/client` 负责第三方 SDK、签名、鉴权、超时、重试和异常映射

## 目录结构

```text
.
├─ SKILL.md
└─ references/
   ├─ architecture-and-boundaries.md
   ├─ coding-standards.md
   ├─ security-integration-and-delivery.md
   ├─ secure-baseline.md
   ├─ evolution-and-governance.md
   ├─ templates-and-checklists.md
   ├─ code-examples.md
   └─ java-backend-standard.md
```

## 参考文件导航

按任务读取对应文件，不建议一次性通读全部参考文档。

- 架构设计、模块拆分、分层边界：`references/architecture-and-boundaries.md`
- 命名、注释、字段说明、校验、异常和代码质量：`references/coding-standards.md`
- 权限、安全、第三方接入、并发、测试和交付：`references/security-integration-and-delivery.md`
- 密钥、敏感数据、上传下载、回调验签、发布前安全检查：`references/secure-baseline.md`
- 兼容性、废弃策略、模块 README、ADR、质量门禁：`references/evolution-and-governance.md`
- 模板、PR 自检、代码评审、上线检查：`references/templates-and-checklists.md`
- 目录模板、类模板、Java 示例实现：`references/code-examples.md`
- 快速导航总览：`references/java-backend-standard.md`

## 安装方式

将本目录放入 Codex skills 目录，例如：

```powershell
$env:CODEX_HOME = "$HOME\.codex"
Copy-Item -Recurse . "$env:CODEX_HOME\skills\java-backend-fusion-specification"
```

如果未设置 `CODEX_HOME`，通常使用：

```text
~/.codex/skills/java-backend-fusion-specification
```

安装后，新的 Codex 会话即可根据 `SKILL.md` frontmatter 中的 `name` 和 `description` 自动判断是否触发。

## 使用示例

可用类似请求触发：

```text
使用 java-backend-fusion-specification 规范，帮我设计订单模块的后端结构。
```

```text
按这个 skill 审查当前 Java 后端改动，重点看分层边界、命名、注释、事务和幂等。
```

```text
用 Java 后端统一规范重构这个 Controller 和 Service。
```

## 维护原则

- `SKILL.md` 保持精炼，只放触发后必须立刻知道的工作流和核心规则
- 细节规则优先放入 `references/`，并从 `SKILL.md` 明确导航
- 模板和检查清单只维护在 `references/templates-and-checklists.md`
- 示例代码只维护在 `references/code-examples.md`
- 更新规则时同步检查 `SKILL.md` 的导航是否仍然准确
- 不要在多个文件重复维护同一份规则，避免后续漂移

## 规则等级

- `MUST`：新项目以及既有项目中被修改区域的默认强制基线
- `RECOMMENDED`：首选默认方案；历史项目可根据成本和风险渐进采用
- `OPTIONAL`：根据业务类型、流量模型、团队规模和合规要求选择

