---
name: java-backend-unified-spec
description: Java 后端项目设计、实现、重构和代码审查的统一规范。凡是 AI 编码代理（Claude Code、Codex、Cursor、Aider 等）需要构建或修改 Java 后端，并应遵循模块优先组织、首选顶层模块命名 `infrastructure / persistence / business / web`、多入口项目使用 `interfaces`、大型第三方集成可选 `integration`、详细命名与注释规则、明确安全基线、可维护性治理，以及每个模块完成后全量回归该模块 API 时，使用本技能。
---

# Java 后端统一规范

## 1. 如何应用本规范

> 本文档定义 Java 后端项目的**架构边界和命名基线**，不是工程判断的替代品。应用这些规则的 AI 代理应像资深后端工程师一样思考：先设计，再实现，并在下列边界内选择专家级方案。规则告诉你“墙在哪里”，房间仍然需要你来设计。

对每个非平凡任务，遵循以下心智模型：

1. **架构先于代码。** 写第一个类之前，先用一两句话说明：本次变更归属哪个模块、跨越哪条边界、复用哪些现有基础能力，以及需要扩展什么。如果答不上来，先停下来设计。
2. **把规则当边界，不当脚本。** 当规则和更清晰的业务表达冲突时，优先选择更清晰的业务表达，并明确说明偏离点。机械合规如果损害可读性或正确性，就是退步。
3. **扩展架构，不模仿反模式。** 如果现有代码存在违规做法，例如 `Service` 直接持有 `BaseMapper`、`Controller` 编排回滚、魔法字符串错误码，应产出符合规范的版本，即使它与相邻文件不一致，也要标明这种不一致。
4. **选择专家级方案，而不是堆样板。** 不要在周边契约已经排除 `null` 的地方塞防御性空判断。不要写只复述方法签名的 Javadoc。直接调用更清晰时，不要提前引入 `Manager / Helper / Util` 包装。三行相似代码胜过过早抽象；但一个清晰抽象胜过三份微妙逻辑的复制。
5. **先用 2 到 3 句话说明取舍，再选择方案。** 当任务存在多个有效架构选择，例如 AOP 与显式调用点、拦截器与包装器、同步写入与事件，先简短说明取舍，再选择最符合项目现有并发、规模和可观测性设计的方案。
6. **按风险匹配严谨度。** 账单、鉴权、资金等热点链路应使用 `multiplyExact`、穷尽式 `switch`，并使用明确单位后缀（`_x10000`、`_credits`、`_ms`）。一次性后台配置写入不需要同等强度。所有系统边界都严格应用安全基线；系统内部的人体工学规则优先按判断力落地。
7. **优先删除，而不是堆积。** 重构时删除死字段、陈旧注释、无用包装器和不再使用的 import。不要给自己刚引入的问题留下 `TODO`；要么完成，要么不要加入。
8. **暴露假设和缺口。** 当规范要求“使用 X helper”，但项目里没有 X 时，提出创建它，而不是静默内联该模式。当规则与无法满足的约束冲突，例如历史 API 契约、第三方 schema，说明约束并有意识地推进。
9. **质疑每个静默兜底。** 空 `try / catch`、解析失败时 `return true`、鉴权失败时 `return null`、`if (x == null) {}` 没有 `else`，这些都是决策，不是安全。将其显式化为 `fail-closed`、`fail-open with audit log` 或抛出类型化异常；若不显而易见，用一条注释说明理由。
10. **把代码写成会被更聪明的人审查的样子。** 名称携带单位（`durationSec`、`priceX10000`）；分支携带意图（`if (isTerminal(status))`，而不是 `if (status.equals("SUCCESS") || status.equals("FAILED"))`）；模块承载职责（一个类做五件事，就在提交前拆开）。

下列参考文件定义架构和命名基线。在这些基线内，**思考、设计、扩展，并交付专家级代码**，不要机械转录规则。

## 2. 规则等级

参考文件中的规则等级如下：

- `MUST`：新项目以及既有项目中被修改区域的默认强制基线
- `RECOMMENDED`：首选默认方案；历史项目可根据成本和风险渐进采用
- `OPTIONAL`：根据业务类型、流量模型、团队规模和合规要求选择

## 3. 工作流程

1. 先按项目目标或业务需求拆分模块。即使项目最终只落在一个 Maven 模块和一个 `src/main/java` 中，也要先按业务模块拆包，再在模块内分层。不要把单个 `src` 拍平为全局 `controller / service / repo / mapper / dto` 结构。
2. 设计或修改代码前，只读取相关参考文件：
   - 架构与模块边界：[references/architecture-and-boundaries.md](references/architecture-and-boundaries.md)
   - 命名、注释、校验和代码质量：[references/coding-standards.md](references/coding-standards.md)
   - 安全、第三方集成、并发、测试和交付：[references/security-integration-and-delivery.md](references/security-integration-and-delivery.md)
   - 密钥、敏感数据、上传下载、回调保护和发布检查的安全基线：[references/secure-baseline.md](references/secure-baseline.md)
   - 兼容性、ADR、模块 README 和质量门禁等可维护性治理：[references/evolution-and-governance.md](references/evolution-and-governance.md)
   - PR、评审、交付和发布检查清单的可复用模板及唯一事实来源：[references/templates-and-checklists.md](references/templates-and-checklists.md)
   - 模板和示例实现：[references/code-examples.md](references/code-examples.md)
3. 应用首选顶层结构：`infrastructure / persistence / business / web`。当项目存在 HTTP、MQ、定时任务、Webhook 等多种入站入口时，使用 `interfaces` 替代 `web`。
4. 只有当第三方集成规模大到值得拥有独立基础设施模块时，才添加 `integration`。
5. HTTP 请求和响应对象只放在 `web` 或 `interfaces/http` 中。让 `business` 编排用例。将强业务规则下沉到 `domain`。让 `persistence` 专注于数据访问和原始查询、写入结果。`Repo` 或 `RepoImpl` 不应把更新失败、不存在、状态非法等语义翻译成业务异常；需要稳定业务错误码时，由 `ServiceImpl` 或项目级业务断言、helper 显式转换这些结果。
6. 严格执行命名：
   - 常量按类别命名：`XxxErrorCodes`、`XxxPermissionCodes`、`XxxRiskRuleCodes`、`XxxNoRepeatKeys`、`XxxLockKeys`、`XxxApiPaths`、`XxxSecurityPaths`、`XxxErrorMessages`
   - 枚举以 `Enum` 结尾
   - API 路径使用短小的 hyphen-case，默认以 `/api/**` 为前缀；只有项目存在独立后台入口时才引入 `/admin/**`，且此时后台 Controller、Assembler 和常量必须显式加 `Admin` 前缀
   - 根包默认使用 `com.{org}.{projectName}`，其中 `projectName` 是去掉连字符、下划线和空格并转为小写后的项目名，例如 `vibe-coding-platform` 变为 `com.{org}.vibecodingplatform`；Maven `groupId` 必须与根包一致；不要使用 `com.example.*`、`com.demo.*`、`com.test.*` 等脚手架默认包名，也不要占用 `java.*`、`javax.*`、`jakarta.*`、`org.springframework.*` 等保留命名空间
   - 模块、包、目录和类命名应遵循参考文档中的详细规则
7. 默认将 `OSS`、`TTS`、`AI`、短信、支付、推送等第三方集成客户端放在 `infrastructure/client`。如果集成规模变大，可拆到 `integration/client`，但仍保持在基础设施边界内。
8. 业务编排留在 `business`；回调、Webhook、MQ 或任务入口放在 `web` 或 `interfaces`。
9. 保持 `Assembler` 和 `Convert` 为不同层。`Assembler` 属于 `web / interfaces`，处理入口层适配，例如 `Request -> Command`、`VO -> Response`。`Convert` 属于 `business`，处理业务层跨对象组装，例如 `DO -> VO`、`DO -> DTO` 和内部快照转换。只有在对象组装确实是一次性的、很短且能用局部 `Builder` 清晰表达时，才跳过 `convert`。
10. 禁止在业务代码中使用魔法字符串和魔法数字。不要使用 `Constants` 或 `CommonConstants` 之类兜底常量类。即使是很短的业务前缀、版本前缀和对象 key 路径片段，例如 `"v"` 或 `"video-cut/export/"`，只要承载业务含义，就属于魔法字符串。
11. 为每条业务链选择单一且明确的并发策略，并保持集中管理。不要在同一工作流中混用多种临时并发风格。
12. AI 代理生成或修改 Java 后端代码时，类、`public` / `protected` 方法、Controller 类、公开端点方法、Repo 方法、Convert 方法、关键业务字段和不显而易见的业务分支必须写中文注释。`@PostMapping` 或 `@GetMapping` 等 Controller 注解不能替代方法注释。字段注释应在名称不足以说明时解释清楚业务含义、单位、状态语义、来源或使用边界。
13. 对 `Request` / `Response` / `Command` / `Query` / `VO` 等纯承载对象，优先使用 Lombok 注解去除样板访问器。需要读写访问时优先使用 `@Data`；只需要读访问时优先使用 `@Getter`。对主要在一个地方组装的命令载荷、DTO、VO、快照载荷或转换目标，优先使用 `@Builder`；如果 `Command`、`Query` 或 `VO` 在当前项目中必须作为常规可变载体并需要日常 getter/setter 访问，仍优先使用 `@Data`。枚举默认只使用 `@Getter`，不要添加 `@Setter`。除非框架约束或非平凡逻辑要求，不要手写简单 getter/setter。
14. 对基础非空存在性、重复提交、活跃任务冲突和直接前置条件等简单守卫式校验，默认使用项目级统一校验 helper，通常暴露为 `Validate.notNull(...)`、`Validate.notBlank(...)`、`Validate.isTrue(...)`、`Validate.equal(...)` 和 `Validate.notEqual(...)`。不要在业务方法中手工组合重复的低层模板，例如 `Validate.isTrue(ObjectUtil.isNotNull(...), "...不能为空")`、`Validate.isTrue(ObjectUtil.equal(...), "...")` 或 `Validate.isTrue(ObjectUtil.notEqual(...), "...")`。一般经验是：当一个业务语句同时组合多个低层 helper，并带有面向业务的消息、错误码或异常语义时，默认视为应抽取候选，而不是内联表达式。如果项目同时标准化 Apache Commons Lang3 和 Hutool，应把它们封装在 helper 后面，或只在真正低层的工具代码中使用，而不是在业务层到处重复原始组合。对普通条件组合和通用 helper 代码，默认使用 Hutool，而不是手写小工具：对象空判断使用 `ObjectUtil.isNull(...)` / `isNotNull(...)`，空安全相等使用 `ObjectUtil.equal(...)` / `notEqual(...)`，字符串使用 `StrUtil.isBlank(...)` / `isNotBlank(...)` 或 `isEmpty(...)` / `isNotEmpty(...)`，集合使用 `CollUtil.isEmpty(...)` / `isNotEmpty(...)`，装箱或三态布尔使用 `BooleanUtil.isTrue(...)` / `isFalse(...)`，常见工具和轻量 HTTP 场景可使用 `MapUtil`、`ArrayUtil`、`BeanUtil`、`JSONUtil`、`DateUtil`、`IdUtil`、`URLUtil`、`HttpUtil` 或 `HttpRequest` 等 Hutool helper。URL 和 HTTP helper 的使用必须留在 `infrastructure/client` 或 `integration/client` 边界内，并仍满足超时、重试、协议校验、白名单和 SSRF 防护要求。不要为简单前置条件失败散落原始 `if (...) { throw ... }`，不要反复手写 `x != null && !x.isBlank()`、`Objects.equals(x, y)` 或 `Boolean.TRUE.equals(flag)` 等模式，也不要为 Hutool 已经很好覆盖的能力发明局部 helper。只有在既有项目基础设施、兼容性约束或更强框架抽象已经稳定且被明确要求时，才允许偏离。对于必须暴露稳定业务错误码的失败，例如不存在、更新失败或状态非法语义，应使用 `BizException` 或项目级业务断言、helper（如 `BizAssert`），而不是裸用 `Validate`。Repo 写方法通常返回 `boolean`、影响行数、已保存实体标识或类型化结果；由 `ServiceImpl` 负责把这些结果转换成业务失败语义。校验注解消息应使用面向业务的中文，例如 `项目不能为空` 或 `时间轴不能为空`，而不是面向字段名的措辞，例如 `projectId不能为空` 或 `timelineDslJson不能为空`。
15. 直接删除无用目录、空占位目录、废弃 demo 目录和临时包树。不要保留 `temp`、`tmp`、`demo`、`test2`、`misc`、`backup` 或兜底 `common` 风格目录，除非它们具有稳定且可审查的职责。
16. 有意识地维护仓库卫生和 `.gitignore`。不要提交 IDE 文件、编辑器缓存、构建产物、临时调试文件、一次性导出产物、下载的凭证，或不属于项目交付物的临时脚本。如果某个文件只用于本地排查且不应参与长期协作，应删除或忽略它，而不是留在仓库中。注意：Spring Boot 环境配置 `application.yml` / `application-local.yml` / `application-test.yml` / `application-prod.yml` 是项目交付物的一部分，应保留在仓库中；它们不被视为本地秘密材料。
17. 密钥、令牌、证书、签名 key、数据库密码、MQ 凭证、对象存储 key 和第三方 `AK/SK` 存放在 Spring Boot 配置文件 `application-{profile}.yml` 中，并由业务代码通过 `@ConfigurationProperties` 绑定的 `XxxProperties` 类读取。不要在 Java 源码、`@Value` 默认值、示例代码、测试或注释中硬编码密钥。环境配置精确拆分为四个文件：`application.yml`（共享默认值，仅非敏感）、`application-local.yml`、`application-test.yml`、`application-prod.yml`。四个文件全部纳入仓库；密钥可见性由仓库访问权限控制，因此必须使用私有且有访问控制的仓库。未经事先评审，不要引入 `application-dev.yml` 或 `application-staging.yml` 等额外环境文件。
18. 每个模块完成后，回归测试该模块所有 API，而不仅是当前任务改动过的端点。

## 4. 参考导航

按需读取以下文件：

- [references/architecture-and-boundaries.md](references/architecture-and-boundaries.md)
- [references/coding-standards.md](references/coding-standards.md)
- [references/security-integration-and-delivery.md](references/security-integration-and-delivery.md)
- [references/secure-baseline.md](references/secure-baseline.md)
- [references/evolution-and-governance.md](references/evolution-and-governance.md)
- [references/templates-and-checklists.md](references/templates-and-checklists.md)
- [references/code-examples.md](references/code-examples.md)
- [references/java-backend-standard.md](references/java-backend-standard.md)，仅用于快速导航
