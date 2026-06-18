# Java 后端统一规范导航

本目录按主题拆分为多个参考文件，默认不要整本通读，按当前任务选择相关文件即可。

## 规则等级说明

- `必须`：新项目默认强制执行；老项目对本次修改范围内代码和结构必须尽量对齐
- `推荐`：优先采用；老项目可按改造窗口逐步落地
- `可选`：按项目类型、团队规模、合规要求、系统复杂度自行决定

## 阅读建议

- 做新项目结构设计、模块拆分、分层边界确认时，读 [architecture-and-boundaries.md](architecture-and-boundaries.md)
- 做命名、注释、字段注释、校验、异常、代码质量约束时，读 [coding-standards.md](coding-standards.md)
- 做权限、安全、第三方接入、并发策略、提测与交付要求时，读 [security-integration-and-delivery.md](security-integration-and-delivery.md)
- 做密钥管理、敏感数据、上传下载、回调验签、发布前安全检查时，读 [secure-baseline.md](secure-baseline.md)
- 做兼容性、废弃策略、模块 README、ADR、静态检查与质量门禁时，读 [evolution-and-governance.md](evolution-and-governance.md)
- 需要直接复用模板或统一检查清单时，读 [templates-and-checklists.md](templates-and-checklists.md)
- 需要参考目录模板、类模板或 Java 示例实现时，读 [code-examples.md](code-examples.md)

## 文件导航

- [architecture-and-boundaries.md](architecture-and-boundaries.md)
  - `目标`
  - `通用技术原则`
  - `使用方式`
  - `顶层模块命名与工程结构`
  - `模块拆分规则`
  - `分层职责与边界`
- [coding-standards.md](coding-standards.md)
  - `命名规范`
  - `注释规范`
  - `数据库规范`
  - `异常、日志与响应规范`
  - `代码质量与可维护性规范`
- [security-integration-and-delivery.md](security-integration-and-delivery.md)
  - `安全、权限与风控规范`
  - `第三方接入规范`
  - `财务项目附加规则`
  - `测试与交付规范`
  - `团队落地要求`
- [secure-baseline.md](secure-baseline.md)
  - `密钥与配置安全`
  - `输入输出与边界安全`
  - `敏感数据治理`
  - `供应链与发布安全`
  - `与总清单的关系`
- [evolution-and-governance.md](evolution-and-governance.md)
  - `兼容性与演进策略`
  - `模块 README 与 ADR`
  - `静态检查与自动化门禁`
  - `评审与规范治理`
- [templates-and-checklists.md](templates-and-checklists.md)
  - `模块 README 模板`
  - `ADR 模板`
  - `PR 自检清单`
  - `代码评审清单`
  - `交付与上线总清单`
- [code-examples.md](code-examples.md)
  - `类模板清单`
  - `单流程标准写法`
  - `代码示例`
