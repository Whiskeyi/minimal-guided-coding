# minimal-guided-coding

语言：[English](./README.md) | [简体中文](./README.zh-CN.md)

## 目录

- [痛点](#痛点)
- [定位](#定位)
- [适用场景](#适用场景)
- [工作方式](#工作方式)
- [最小范围原则](#最小范围原则)
- [注释规则](#注释规则)
- [评估设计](#评估设计)
- [评估结果](#评估结果)
- [参考文档](#参考文档)

## 痛点

在 vibe coding / guided programming 中，增量调整很容易从“小改动”膨胀成“大改动”：

- 一个局部需求触发额外 helper、配置、抽象或跨文件重构。
- 用户已经否定方案 A，转向方案 B/C，但旧代码、旧测试、fixture、注释或文案仍被保留。
- 为了“保险”同时保留新旧逻辑路径，导致最终代码偏离最新需求。
- CR 描述只写“done”，审阅者无法判断为什么改这些文件、旧方案残留是否清理干净、验证是否可复现。
- 明显逻辑被添加“翻译式”注释，只增加噪音。
- 为了解释修改意图，把用户指令、任务上下文或“按要求修改”等元信息写进代码、注释、测试名或用户可见文案。
- 在没有要求补测试、且仓库类似代码也没有测试时，把临时验证测试留在最终 CR 中。

`minimal-guided-coding` 用来解决这类收敛问题：让最终代码只服务用户的最新需求，在“最小但完整”的闭环中完成改动，并清理已被否定或过期方案的残留。

## 定位

这个 Skill 面向对现有代码的 guided modification。

这里的 “Guided” 指：用户在连续对话中明确指定了如何调整现有实现。Agent 的任务是按照最新要求收敛，而不是自行设计大型增量功能、重构架构或从零搭建新模块。

## 适用场景

适合：

- 当前任务是对现有代码的聚焦调整。
- 用户表达了收敛意图，例如“不要过度修改”“diff 太大”“上一个方案不对”“回退旧改动”“清理残留”“CR 要清楚”。
- 已经观察到或强烈怀疑存在 diff 范围膨胀、旧方案沉积、注释噪音或最终描述不可审计的问题。

不适合：

- 可预见的大功能、大重构或新模块脚手架。
- 常规增量开发，且用户没有表达控制变更范围或清理旧方案的意图。
- 边界清晰、没有旧方案残留、没有 diff 发散风险的单个小修复。
- 需要产品探索、方案发散或 agent 主动创意扩展的场景。

## 工作方式

这个 Skill 将“最小实现”拆成 4 个轻量约束。

| 约束 | 目的 |
| --- | --- |
| 最新契约优先 | 用户最新要求覆盖旧上下文；被否定的旧方案默认成为清理候选。 |
| 实现前小预算 | 编辑代码前，用一句话或短列表写明精确路径、动作和原因，避免执行中范围蔓延。 |
| 残留闭环清理 | 追踪 imports、helpers、types、configs、fixtures、tests、comments 和 copy，检查旧方案标识是否仍存在。 |
| 可审计 CR | 最终回复说明修改范围、旧方案清理、关键边界和可复现的最新验证结果。 |

预算和验证计划是 agent 的执行约束，不是等待用户再次确认的 gate；当用户最新要求已经足够清晰时，写完预算后应直接进入实现。

复杂或多文件残留场景可以使用以下预算格式：

```markdown
## Change Budget (pre-implementation)
- `exact/path`: modify/delete/retain - necessary reason
- No change: `exact/path` - reason
```

最终回复应覆盖：

- 修改范围：为什么每个路径必须变化。
- 旧方案清理：删除了什么，或确认没有旧方案残留。
- 关键边界：可 grep 的字段、状态、枚举、阈值、API 或配置键。
- 验证：实际命令、退出码、测试总数、通过数和 0 failures，且来自最新运行。

## 最小范围原则

“最小”不是文件数最少，而是当前契约的最小完整闭环。

如果单文件即可闭环，就只改单文件；如果契约天然横跨 parser、state boundary、renderer 和 tests，就应把这些必要文件一起纳入预算。反过来，泛化 helper、配置层、兼容分支和不属于当前闭环的旧测试，不应因为“以后可能有用”而保留。

## 注释规则

注释是预算项，不是礼貌项。

只在以下情况下保留或新增注释：

- 业务规则、兼容约束或边界条件无法从代码结构明显看出。
- 逻辑足够复杂，删除注释会让审阅者难以理解为什么这样写。
- 类型、枚举或公开常量承载业务语义，需要解释取值含义或外部契约。

必须删除对显然分支、简单赋值、直观函数或局部变量的“翻译式”注释。

## 评估设计

评估用例围绕 4 个压力场景设计，对应真实工作中常见的失控风险。

| Case | 压力场景 | 主要检查点 |
| --- | --- | --- |
| multi-turn-latest-intent | 方案 A 和 B 在连续对话中都被否定，最终只应保留方案 C。 | 是否只实现最新访问规则？A/B helpers、configs、旧 enums 和旧 tests 是否删除？是否没有保留兼容 fallback？ |
| hidden-scattered-residue | 主行为正确，但被否定方案散落在 policy、configs、types、fixtures、tests 和 copy 中。 | 人工审查残留是否清理？当前行为仍使用的 audit 是否保留？主流程是否不被扰动？ |
| tempting-shared-abstraction | 之前为一个小规则抽出了 generic engine、config 和解释性注释；审查要求缩小范围，且很容易把审查请求写进注释或测试名。 | 是否恢复 list/detail 的本地判断？是否删除 generic rule layer 和冗余注释？是否避免用户指令元描述？特殊编码原因是否限于业务动机且简短？ |
| necessary-cross-file-closure | 状态值从 `waiting` 改为 `queued`，但真实契约横跨 parser、renderer、export state 和 tests。 | 必要的跨文件闭环是否完整？`running` 和 `done` 原行为是否保留？旧 `waiting` 残留是否删除？ |

每个 case 都会在 with-skill 和 baseline 两种配置下运行。

| 配置 | 指导方式 |
| --- | --- |
| baseline | 使用同一 fixture 和业务 prompt，但只要求标准 coding completion；不提供 `minimal-guided-coding` Skill 路径、实现前预算、关键边界模板或残留闭环流程。 |
| with-skill | 在同一业务 prompt 上，明确要求先阅读 Skill，并按 Skill 的预算、清理和最终 CR 契约执行。 |

这个设计确保差异主要来自 Skill 是否介入，而不是不同业务需求。

自动断言不仅检查 `node --test` 是否通过，还检查：

- 行为是否满足最新契约。
- 旧方案标识、文件、测试、fixture 或 copy 是否仍有残留。
- 代码、注释、fixture、测试名或用户可见 copy 是否包含用户指令元描述；特殊编码动机是否业务化且简短。
- 修改文件是否只落在必要闭包内。
- 测试是否真正覆盖关键边界，而不是只包含关键词。
- 是否存在编辑前 change budget。
- 最终 CR 是否命名关键边界，并提供最新 `node --test` 证据。
- 代码指标是否使用 LCS 行级 diff 计数，避免低估重复行。

## 评估结果

当前复杂评估使用 final-curated caliber，来自 4 个复杂 case 的有效最新 agent runs；代码指标使用 LCS 行级 changed-line count。

| Metric | with-skill final | baseline realistic | Delta |
| --- | ---: | ---: | ---: |
| Assertion pass rate | 100% (30/30) | 63.3% (19/30) | +36.7% |
| Changed files | 19 | 20 | -1 |
| Lines added | +14 | +25 | -11 |
| Lines deleted | -121 | -118 | +3 |

逐 case 结果：

| Case | with-skill final | baseline realistic | 主要差异 |
| --- | ---: | ---: | --- |
| multi-turn-latest-intent | 7/7 | 3/7 | Baseline 保留 A/B 旧标识，缺少预算和 CR 证据。 |
| hidden-scattered-residue | 8/8 | 6/8 | Baseline 清理了主行为，但缺少编辑前预算和可复现 CR 证据。 |
| tempting-shared-abstraction | 8/8 | 6/8 | Baseline 完成代码收窄，但缺少预算和精确验证报告。 |
| necessary-cross-file-closure | 7/7 | 4/7 | Baseline 保留 `waiting` 残留，CR 边界描述不完整。 |

这里的 `100%` 指 4 个复杂 case、30 个自动断言下的最终 curated re-evaluation 结果，并不表示所有真实场景都能在单次执行中达到 100%。

这个结果表明：该 Skill 的主要收益不是机械减少文件数，而是清除旧方案残留、减少不必要新增，并让最终 CR 和验证证据可审计。

## 参考文档

- [SKILL.md](./SKILL.md)：Skill 入口和执行契约。
- [Change Budget](./references/budget.md)：变更预算判断规则和详细步骤。
- [Cleanup Rules](./references/cleanup.md)：方案切换清理、写入规则和注释规则。
- [Verification](./references/verification.md)：CR 清晰度检查、验证清单和最终回复结构。
- [Examples](./references/examples.md)：示例、反模式和风险信号。
