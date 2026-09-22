---
title: "Spec-kit使用教程"
published: 2026-05-13
description: "Spec Kit 使用教程，整理安装配置、规范驱动开发流程、命令用法和项目落地步骤。"
category: 工具
tags: [AI,SDD]
---

# Spec- Kit 
[spec-kit](https://github.com/github/spec-kit) github 官方开发维护的一个 [SDD](https://github.com/github/spec-kit/blob/main/spec-driven.md) 工具

## 1. 安装

1. 需要先安装 Python 包管理工具 uv： 

``` 
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

2. 安装 spec-kit 工具。 X.Y.Z为版本

```
   uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

3. 检查你安装的工具

```
   specify check
```

## 2. 使用

### 1. 初始化项目

1. 创建新项目，会弹出选择框选择你需要用到的 AI 工具
   
```
   specify init <PROJECT_NAME>
```

2. 如果项目已经存在

```
   specify init . --integration <AI code agent>
   # or
   specify init --here --integration <AI code agent>
```

在项目里会生成 `.specify` 文件夹，`.specify/templates/` 文件夹下会存放模板文档，后续的文档生成依据此模板生成，默认使用官方模板，也可以自定义模板。

### 2. 项目里添加/移除 AI 集成

1. 新安装其他 AI 工具，一个项目是可以允许使用多个 AI 工具

```
   specify integration install <agent>
```

2. 卸载 AI agent

```
   specify integration uninstall <agent>
```

   > 如果所使用的 AI 工具支持 skills 的话，则默认安装的是skills集，在 AI 工具中根据提示词自动激发使用，反之安装的是 speckit 指令集，使用 `/`调用相关指令。后续以调用 skills 集为例
   >
   > （如出现未自动激发skills，可手动指定skills， codex中使用为 `$skills-name`，其他AI工具使用其对应工具的方式即可，下面教程示例中都为 codex 手动触发 skills）

[其他 AI 集成指令参考](https://github.github.io/spec-kit/reference/integrations.html)

### 3.  制定项目宪章 (`constitution.md`)

constitution 这是项目级别的 "宪法"，定义了不可违背的约束条件，所有 Spec 都必须遵守，是整个项目的管理原则

创建或更新项目的管理原则和发展指南

**使用：**

```
   $speckit-constitution   
```

**产出：**`constitution.md`

生成的文档为 `constitution.md`，存放在项目目录的 `./.specify/memory/constitution.md`

**示例：**
```text
<!--
Sync Impact Report
Version change: 1.0.0 → 1.0.1
Modified principles:
- I. Detection Core First → I. 检测内核优先
- II. Shared KMP Contracts Before Platform Code → II. 共享 KMP 契约先于平台代码
- III. Kotest Verification Is Mandatory → III. Kotest 验证强制执行
- IV. Runtime Observability and Evidence → IV. 运行时可观测性与证据
- V. Scoped Simplicity and Delivery Boundaries → V. 范围克制、简单交付
Added sections:
- None
Removed sections:
- None
Templates requiring updates:
- ✅ .specify/templates/plan-template.md
- ✅ .specify/templates/spec-template.md
- ✅ .specify/templates/tasks-template.md
- ⚠ .specify/templates/commands/*.md not present in this project
- ✅ README.md
- ✅ docs/testing/kotest-allure.md
Follow-up TODOs:
- None
-->

# monitor_detection 项目宪章

## 核心原则

### I. 检测内核优先

本项目 MUST 优先服务监控识别系统本体：输入源接入、推理调用、场景识别、
规则判定、告警聚合、证据输出与运行时状态。Web UI、Android 展示 UI、
告警大屏和管理后台均不属于当前项目范围，除非未来通过宪章修订明确改变
项目边界。

理由：本仓库的目标是交付可复用的检测内核与运行时，而不是被展示层或管理端
分散核心部署目标。

### II. 共享 KMP 契约先于平台代码

核心模型、规则契约、场景处理器、DTO、运行时契约和业务行为，只要能被 JVM
与 Android 共用，就 MUST 放在 Kotlin Multiplatform 共享 source set 中。
平台 source set MUST 只承载输入源、模型执行、服务、存储和生命周期等适配
职责。任何契约变更 MUST 同时考虑 JVM 与 Android 装配，避免双端行为漂移。

理由：项目价值在于一套共享检测行为服务两个部署运行时。把业务逻辑复制到
平台侧会造成长期分叉和维护风险。

### III. Kotest 验证强制执行

每个行为变更 MUST 在最小有效层级补充 Kotest 测试：共享逻辑使用 common
测试，JVM 适配器和运行时使用 JVM 测试，Android 运行时行为使用 Android host
测试。测试 MUST 使用 Kotest 风格编写，测试报告 MUST 通过 Allure 生成以便
本地和 CI 审查。相关 compile、ktlint、Kotest 和 Allure report 命令通过前，
变更不得视为完成；如无法执行，MUST 明确记录原因。

理由：检测规则、运行时契约和平台装配容易回归，测试与报告是验收记录。

### IV. 运行时可观测性与证据

运行时代码 MUST 通过共享契约暴露健康状态、源连接状态、最后处理帧时间、
告警记录和证据元数据。错误 MUST 更新运行时健康状态，并提供可行动的错误
信息。证据存储可以先以 no-op 元数据形式实现，但公开契约 MUST 为后续文件
或对象存储实现保持稳定。

理由：监控部署必须在没有 UI 或调试器的情况下可诊断、可复盘。

### V. 范围克制、简单交付

变更 MUST 严格限定在当前需求或失败验证所需范围内。新增抽象 MUST 解决真实
重复或维护共享契约，MUST NOT 为未来假设增加投机性灵活性。平台能力 SHOULD
优先在现有契约背后增量实现，而不是用无关框架替换既有架构。

理由：项目已经是多平台运行时。小而可验证的变更能保护共享模型并降低交付
风险。

## 项目约束

- Kotlin Multiplatform 是共享逻辑的强制实现模型。
- JVM 与 Android 是当前有效部署目标。
- Kotest 是强制测试框架；JUnit 仅作为 JVM 测试执行平台，不作为测试编写风格。
- Allure 是测试报告的强制格式。
- MUST 保持 `ktlint` 格式化和现有 Gradle version catalog 约定。
- `com.smartwater.ai.*` 直接运行时链路是当前有效路径；MUST NOT 重新引入旧
  bridge 或重复 pipeline 路径。

## 开发流程

1. 用检测或运行时行为定义需求范围，并明确非目标。
2. 共享行为优先修改 common source set；平台 source set 仅用于适配职责。
3. 实现前或实现同时补充 Kotest 覆盖。
4. 运行相关验证命令：
    - `./gradlew :monitor_detection:jvmTest`
    - Android 运行时行为变化时运行
      `./gradlew :monitor_detection:testAndroidHostTest`
    - `./gradlew :monitor_detection:allureReport --no-configuration-cache`
    - `./gradlew :monitor_detection:ktlintCheck`
    - 与触达目标相关的 compile 任务
5. 命令、契约、运行时行为或项目边界变化时，MUST 更新 README 或 docs。

## 治理

本宪章优先于非正式项目习惯。所有 specification、plan、task list、code review
和 implementation 都 MUST 检查是否符合核心原则与项目约束。

修订本宪章 MUST 满足：

1. 对本文件进行书面修改。
2. 在 Sync Impact Report 中列出受影响原则、模板和文档。
3. 在同一变更中同步更新依赖模板和运行时指导文档。
4. 按语义版本规则升级版本：
    - MAJOR：不兼容的治理或原则重定义、删除。
    - MINOR：新增原则、章节或实质扩展指导。
    - PATCH：澄清、文字调整、非语义变更。

实现规划前和交付前 MUST 进行合规审查。任何违反宪章的情况 MUST 写入 plan，
并给出具体理由和被拒绝的更简单替代方案。

**Version**: 1.0.1 | **Ratified**: 2026-05-11 | **Last Amended**: 2026-05-11
```


### 4. 制定项目规格 (`spec.md`)

spec 这是 "唯一真实来源"。它回答 "***做什么**" 和 "**为什么做**"，不涉及 "怎么做"。尽可能明确地说明你想打造*什么*以及*为什么要做*。**现在不要专注于技术栈**

定义你想构建的内容（需求和用户故事）

**使用：**

```
$speckit-specify
```

**产出：**`spec.md`， `checklist`

使用之后会自动新建一个分支，并生成相关文档，生成的文档为规格文档 `spec.md` 和 质量检查清单 `checklist`，存放在项目目录的 `./spec/` 目录下

质量检查清单用于检查 `spec.md` 文档是否符合要求，验证需求的完整性、清晰性和一致性,以及是否引入了不必要的技术栈。

**spec文档示例：**
```text
# Feature Specification: JVM 与 Android 多平台监控识别报警

**Feature Branch**: `001-kmp-monitor-alerts`
**Created**: 2026-05-12
**Status**: Draft
**Input**: User description: "
帮我生成一份需求文档，主要用于JVM和Android的KMP多平台的监控识别报警项目，主要识别场景有工程机械作业预警、水面异物预警、非运维人员入侵预警、监测设施位移预警、水体颜色变化预警、船舶异常停留预警、闸坝开启/关闭状态识别、非法排污预警"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 多平台统一识别八类监控场景 (Priority: P1)

运维负责人需要在 JVM 部署环境和 Android 设备环境中使用同一套监控识别能力，
对水利、水域或设施监控画面进行实时分析，并识别八类业务场景：工程机械作业、
水面异物、非运维人员入侵、监测设施位移、水体颜色变化、船舶异常停留、闸坝
开启/关闭状态、非法排污。

**Why this priority**: 八类场景识别是系统核心价值；若不能统一识别并输出结果，
后续告警、证据和状态能力都无法成立。

**Independent Test**: 准备包含八类场景的样例输入，分别在 JVM 和 Android 目标中
触发识别流程，验证每类场景均能产生对应的场景事件或正常状态结果。

**Acceptance Scenarios**:

1. **Given** 一个启用工程机械作业预警的监控任务，**When** 画面中出现工程机械
   作业目标，**Then** 系统输出工程机械作业预警事件。
2. **Given** 一个启用闸坝状态识别的监控任务，**When** 画面中出现闸门开启或
   关闭状态，**Then** 系统输出对应的闸坝状态识别结果。
3. **Given** 同一业务场景在 JVM 与 Android 两端运行，**When** 两端接收等价
   输入，**Then** 两端输出的场景类型、任务标识和告警语义保持一致。

---

### User Story 2 - 按规则生成预警并聚合告警 (Priority: P2)

运维值守人员需要系统根据任务规则判断识别结果是否达到预警条件，并将连续或
重复出现的同类事件聚合为可跟踪的告警记录，避免同一风险在短时间内产生大量
重复告警。

**Why this priority**: 识别结果只有经过规则判定和告警聚合，才能成为可处置的
运维事件。

**Independent Test**: 为每类场景配置启用状态、目标类别、区域条件和持续命中
要求，输入命中与未命中的样例，验证告警创建、更新和未触发路径。

**Acceptance Scenarios**:

1. **Given** 非运维人员入侵预警任务要求连续命中，**When** 入侵目标持续出现并
   满足规则，**Then** 系统创建入侵告警记录。
2. **Given** 同一船舶异常停留事件已经存在打开的告警，**When** 后续画面继续
   满足同一事件条件，**Then** 系统更新该告警的最后发现时间，而不是创建重复告警。
3. **Given** 水面异物检测目标未进入指定关注区域，**When** 系统完成识别，
   **Then** 不生成水面异物预警告警。

---

### User Story 3 - 输出证据、任务状态和设备健康 (Priority: P3)

平台对接方和现场运维人员需要查询每个任务的运行状态、设备健康、告警记录和
证据信息，以便确认系统是否正常工作，并在发生预警后进行复核。

**Why this priority**: 监控识别项目通常部署在边缘或无人值守环境中；没有状态
和证据输出时，告警可信度和运维可用性不足。

**Independent Test**: 启动一个任务并模拟识别、告警、异常和停止过程，验证任务
状态、设备健康、告警列表和证据元数据均可被查询且字段完整。

**Acceptance Scenarios**:

1. **Given** 一个监控任务正在处理输入，**When** 查询任务状态，**Then** 返回
   运行状态、源连接状态、最近心跳和最近处理帧时间。
2. **Given** 一个非法排污预警已经触发，**When** 查询告警详情，**Then** 返回
   告警类型、等级、任务、设备、首次发现时间、最后发现时间和证据信息。
3. **Given** 输入源异常中断，**When** 查询设备健康，**Then** 返回非健康状态和
   可用于定位问题的错误信息。

---

### Edge Cases

- 输入画面无任何目标时，系统必须保持任务运行状态并不生成告警。
- 同一画面中同时出现多个场景目标时，系统必须按已启用任务分别识别和输出。
- 目标出现在关注区域外时，系统必须保留识别事实，但不触发要求区域命中的告警。
- 目标短暂出现后消失时，系统必须重置连续命中状态，避免误报持续性预警。
- 设备或输入源断开时，系统必须更新健康状态，并避免输出伪造的场景告警。
- 同一告警持续出现时，系统必须更新既有告警，而不是无限制创建重复记录。
- 场景规则未配置时，系统必须使用统一默认规则：任务启用、场景默认目标标签、置信度 ≥ 0.7、连续命中 ≥
  2；未配置关注区域时默认全画面有效。

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系统 MUST 支持 JVM 和 Android 两类部署目标，并保持共享业务语义一致。
- **FR-002**: 系统 MUST 支持工程机械作业预警，识别工程机械相关作业目标并输出预警事件。
- **FR-003**: 系统 MUST 支持水面异物预警，识别水面漂浮物、垃圾或异常异物并输出预警事件。
- **FR-004**: 系统 MUST 支持非运维人员入侵预警，识别非授权人员进入监控区域并输出预警事件。
- **FR-005**: 系统 MUST 支持监测设施位移预警，识别监测设施位置异常或疑似位移并输出预警事件。
- **FR-006**: 系统 MUST 支持水体颜色变化预警，识别水体颜色异常变化并输出预警事件。
- **FR-007**: 系统 MUST 支持船舶异常停留预警，识别船舶在关注区域内异常停留并输出预警事件。
- **FR-008**: 系统 MUST 支持闸坝开启/关闭状态识别，并输出明确的开启、关闭或未知状态。
- **FR-009**: 系统 MUST 支持非法排污预警，识别排污口、排放行为或疑似排污异常并输出预警事件。
- **FR-010**: 系统 MUST 支持为每个监控任务配置启用状态、场景类型、关注区域和规则条件。
- **FR-011**: 系统 MUST 根据任务规则判断识别结果是否触发告警，包括目标类别、置信程度、关注区域和持续命中条件。
- **FR-011a**: 系统 MUST 在场景规则未配置时使用统一默认规则：任务启用、场景默认目标标签、置信度 ≥
  0.7、连续命中 ≥ 2，且未配置关注区域时全画面有效。
- **FR-012**: 系统 MUST 将同一任务、同一设备、同一风险类型的持续事件聚合为同一告警记录。
- **FR-013**: 系统 MUST 为告警记录输出首次发现时间、最后发现时间、告警等级、标题、任务、站点和设备信息。
- **FR-014**: 系统 MUST 为每次告警输出证据元数据，至少包括任务、输入源、时间和关联告警。
- **FR-015**: 系统 MUST 输出任务运行状态，包括启动、运行、停止、错误、源连接状态和最近心跳。
- **FR-016**: 系统 MUST 输出设备健康状态，并在输入源或运行异常时提供可诊断的错误信息。
- **FR-017**: 系统 MUST 提供任务列表、任务状态、告警列表、设备健康、规则配置和证据信息的统一输出。
- **FR-018**: 系统 MUST 在无目标、未满足规则或任务未启用时不产生告警。
- **FR-019**: 系统 MUST 保证 JVM 与 Android 对同一场景类型使用一致的场景命名、告警语义和输出字段。
- **FR-020**: 系统 MUST 为八类场景分别提供可独立验证的测试样例和验收结果。

### Project Boundary Requirements

- **PBR-001**: 功能 MUST 保持检测内核范围；本规格不包含 Web UI、Android 展示 UI、告警大屏或管理后台。
- **PBR-002**: 共享行为 MUST 先按 Kotlin Multiplatform common 行为描述，再描述平台适配行为。
- **PBR-003**: 验证方案 MUST 标明 Kotest 覆盖和用于证明功能的 Allure 报告命令。

### Key Entities *(include if feature involves data)*

- **监控任务**: 表示一个运行中的识别配置，包含任务标识、站点、设备、场景类型、启用状态、关注区域和规则。
- **输入帧**: 表示一次来自监控源的画面输入，包含来源、时间、宽高和可选原始数据引用。
- **检测目标**: 表示画面中识别出的对象或状态，包含标签、置信程度、位置、跟踪标识和附加属性。
- **关注区域**: 表示用户关心的监控区域，用于判断目标是否进入有效告警范围。
- **场景观测**: 表示某个任务在一次或多次输入中形成的业务观测结果，包括场景类型、命中目标、连续命中和区域信息。
- **场景事件**: 表示规则判定前后形成的业务事件，用于后续告警聚合。
- **告警记录**: 表示可处置的预警结果，包含告警标识、场景类型、任务、设备、等级、状态和时间信息。
- **证据包**: 表示告警关联的复核信息，包含截图、视频片段或元数据。
- **任务运行状态**: 表示任务当前是否启动、运行、停止或错误，以及源连接和最近心跳信息。
- **设备健康状态**: 表示设备或运行环境是否健康，以及异常原因。

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 八类场景均具备至少 1 个可独立验收的成功识别样例，验收结果覆盖 JVM 和 Android 两类部署目标。
- **SC-002**: 对同一组等价输入，JVM 与 Android 输出的场景类型和告警关键字段一致率达到 100%。
- **SC-003**: 对未满足规则的样例输入，系统不产生告警的准确率达到 100%。
- **SC-004**: 同一持续事件在 10 次连续命中内只产生 1 条打开告警记录，并持续更新最后发现时间。
- **SC-005**: 任务状态和设备健康查询在任务启动、运行、错误、停止四类状态下均返回完整字段。
- **SC-006**: 每条告警记录均能关联到证据元数据，关联完整率达到 100%。
- **SC-007**: 规格覆盖的核心用户场景均可通过测试报告复核，且报告中能看到通过、失败和执行时间信息。
- **SC-008**: 新增或变更场景后，已有场景的验收样例仍全部通过。

## Assumptions

- 监控输入源、模型来源和外部对接系统由部署方提供，本规格只定义识别报警项目的业务能力和输出要求。
- 八类场景都可以通过目标、状态或属性识别结果映射到统一场景事件。
- 默认告警等级为预警级别；具体等级升级策略可在后续规则增强规格中细化。
- 关注区域用于减少误报；未配置关注区域时，任务默认在整个画面范围内生效。
- 告警证据可以先以元数据形式交付，截图或视频片段落盘属于后续增强能力。
- 本规格不包含展示端、管理后台、权限系统或人工处置流程。
```


### 5. spec 功能规范澄清

在建立基础规格后，可以继续说明第一次尝试中未完全捕捉到的任何要求。先运行结构化澄清流程，再制定 plan 以减少下游重做的技术计划

>  官方建议优先顺序：
>
> 1. 使用（结构化）——顺序性、基于覆盖的问题，将答案记录在澄清部分（使用 `$speckit-clarify)
> 2. 如果某些内容仍然模糊，可以选择性地进行自由形式的细化。（手动修改文档）

**使用：**

```
$speckit-clarify
```

使用之后 AI 会对模糊点进行提问和建议，可以选择接受 AI 建议 或者 提供简短回答，之后会将澄清内容写入 `.spec`文档

也可以手动修改文档进行模糊点的细化完善。

**示例：**
```text
## Clarifications

### Session 2026-05-13

- Q: 场景规则未配置时系统应如何处理？ → A: 使用统一默认规则：任务启用、场景默认目标标签、置信度 ≥
  0.7、连续命中 ≥ 2、未配置 ROI 时全画面有效。
```

### 6.  质量检查(`checklist.md`)

澄清或修改了 spec 文档之后，建议对 spec 文档重新进行质量检查，生成质量检查清单。

使用：

```
$speckit-checklist
```

生成之后查看 检查结果文档确认是否检查通过。

**示例：**
```text
# Specification Quality Checklist: JVM 与 Android 多平台监控识别报警

**Purpose**: Validate specification completeness and quality before proceeding to planning
**Created**: 2026-05-12
**Feature**: [spec.md](../spec.md)

## Content Quality

- [x] No implementation details (languages, frameworks, APIs)
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders
- [x] All mandatory sections completed

## Requirement Completeness

- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous
- [x] Success criteria are measurable
- [x] Success criteria are technology-agnostic (no implementation details)
- [x] All acceptance scenarios are defined
- [x] Edge cases are identified
- [x] Scope is clearly bounded
- [x] Dependencies and assumptions identified

## Feature Readiness

- [x] All functional requirements have clear acceptance criteria
- [x] User scenarios cover primary flows
- [x] Feature meets measurable outcomes defined in Success Criteria
- [x] No implementation details leak into specification

## Notes

- 自检通过。规格可进入 `/speckit-plan`。

```


### 7.  制定计划(`plan.md`等)

这一步就是根据需求指定项目架构，技术栈和其他技术需求。

使用：

```
$speckit-plan
```

这一步的输出将包含**若干实现细节文档**，可以查看文档，确保使用了正确的技术栈，符合你的说明
具体会包含计划文档`plan.md`,调研文档`research.md`，数据模型文档`data-model.md`，接口文档`contracts`

**示例：**
```text

# Implementation Plan: JVM 与 Android 多平台监控识别报警

**Branch**: `001-kmp-monitor-alerts` | **Date**: 2026-05-12 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/001-kmp-monitor-alerts/spec.md`

## Summary

本功能把 `monitor_detection` 完整规划为 JVM 与 Android 双端共享的 KMP 监控识别
报警能力。实现策略是在现有单 KMP 模块中补齐共享检测内核：八类场景处理器、
统一规则判定、ROI/连续命中、告警聚合、证据元数据、任务状态与设备健康输出；
JVM 与 Android 仅保留输入源、推理引擎、生命周期和平台服务适配。

## Technical Context

**Language/Version**: Kotlin Multiplatform，Kotlin 2.3.20
**Primary Dependencies**: Kotlin Coroutines、kotlinx.serialization、kotlinx.datetime、ONNX
Runtime、TensorFlow Lite、JavaCV/FFmpeg、RootEncoder、Kotest、Allure
**Storage**: 当前以共享内存仓库与证据元数据为交付基线；文件证据落盘作为后续增强
**Testing**: Kotest 与 Allure 报告
**Target Platform**: JVM 部署版与 Android 设备部署版
**Project Type**: 单模块 Kotlin Multiplatform 检测运行时库
**Performance Goals**: 任务状态与告警输出保持实时可查询；每个目标平台至少能完成八类场景样例验证
**Constraints**: 不引入 Web UI、Android 展示 UI、告警大屏或管理后台；共享行为必须在 common source
set；平台代码仅做适配
**Scale/Scope**: 8 类场景、JVM/Android 双端、任务配置、规则判定、告警聚合、证据元数据、运行状态和健康输出

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **检测内核范围**：PASS。计划仅覆盖输入、推理、场景识别、规则、告警、证据、
  运行时状态和共享契约；明确排除 UI、大屏、管理后台。
- **共享 KMP 优先**：PASS。核心模型、规则、场景处理、DTO、状态和证据契约均规划
  在 `commonMain`；JVM/Android 只保留适配器和运行时装配。
- **Kotest 与 Allure**：PASS。计划要求 common/JVM/Android host Kotest 覆盖，并用
  `:monitor_detection:allureReport --no-configuration-cache` 生成报告。
- **可观测性与证据**：PASS。计划包含任务状态、设备健康、告警记录和证据元数据。
- **范围克制与简单性**：PASS。保留现有单模块 KMP 架构，不新增 UI、后台或额外模块。

## Project Structure

### Documentation (this feature)


specs/001-kmp-monitor-alerts/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── runtime-query-contract.md
│   └── scenario-output-contract.md
└── checklists/
    └── requirements.md


### Source Code (repository root)


monitor_detection/
├── src/
│   ├── commonMain/kotlin/com/smartwater/ai/
│   │   ├── core/model/
│   │   ├── core/rule/
│   │   ├── core/alarm/
│   │   ├── data/contract/
│   │   ├── runtime/contract/
│   │   └── scenario/
│   ├── commonTest/kotlin/com/smartwater/ai/
│   ├── jvmMain/kotlin/
│   ├── jvmTest/kotlin/
│   ├── androidMain/kotlin/
│   └── androidHostTest/kotlin/
└── build.gradle.kts


**Structure Decision**: 继续使用现有单 KMP 模块 `:monitor_detection`。共享业务能力进入
`commonMain`，共享测试进入 `commonTest`；JVM 和 Android 仅补平台适配与运行时验证。

## Complexity Tracking

无宪章违反项。

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|--------------------------------------|
```



### 8. 任务分解(`task.md`)

可以将计划拆分为具体且可执行的任务，并按正确顺序执行

**使用**：

```
$speckit-tasks
```

**生成文档**：`task.md`

**示例：**
```text
# Tasks: JVM 和 Android 多平台监控识别报警

**Input**: Design documents from `specs/001-kmp-monitor-alerts/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`,
`quickstart.md`

**Tests**: 本功能受项目原则约束，所有行为变更必须包含 Kotest 测试，并通过 Allure 报告复核。
**Organization**: 任务按用户故事分组，确保每个故事可独立实现和验证。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可并行执行，涉及不同文件且不依赖未完成任务
- **[Story]**: 用户故事标签，仅用于用户故事阶段
- 每个任务描述都包含明确文件路径

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 确认 KMP、Kotest、Allure 和文档入口满足本功能交付前提。

- [X] T001 校验 Kotest 与 Allure 依赖和任务入口，必要时调整 `gradle/libs.versions.toml` 和
  `monitor_detection/build.gradle.kts`
- [X] T002 [P] 校验 Kotest 全局配置和 Allure 标签输出，必要时调整
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/test/KotestProjectConfig.kt`
- [X] T003 [P] 对齐测试报告说明和命令，更新 `docs/testing/kotest-allure.md`
- [X] T004 [P] 对齐功能验收命令和报告路径，更新 `specs/001-kmp-monitor-alerts/quickstart.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 建立所有用户故事共用的模型、规则、场景引擎、DTO 和测试夹具。

**CRITICAL**: 完成本阶段前，不应开始任一用户故事实现。

- [X] T005 [P] 校验八类场景枚举、告警级别和运行状态定义，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Enums.kt`
- [X] T006 [P] 校验监控任务、场景观测、告警记录和设备健康字段，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Task.kt`、
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Scenario.kt`、
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Alarm.kt` 和
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Infrastructure.kt`
- [X] T007 [P] 补齐关注区域默认全画面、区域命中和非法区域校验逻辑，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/model/Geometry.kt`
- [X] T008 [P] 补齐规则默认值、置信度阈值、目标标签、连续命中和区域条件模型，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/rule/RuleEngine.kt`
- [X] T009 实现场景连续命中状态、未命中重置和事件键辅助逻辑，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/engine/ScenarioSupport.kt`
- [X] T010 注册并校验八类场景处理器工厂，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/engine/DefaultScenarioRegistryFactory.kt`
- [X] T011 [P] 对齐运行时查询和场景输出 DTO 字段，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/data/contract/ApiContracts.kt`
- [X] T012 [P] 对齐平台服务、视觉引擎、视频源和运行时契约，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/RuntimeAdapters.kt` 和
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/PlatformServices.kt`
- [X] T013 [P] 建立八类场景共享测试夹具和样例输入，新增
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/test/ScenarioTestFixtures.kt`

**Checkpoint**: Foundation ready - 用户故事实现可并行启动。

---

## Phase 3: User Story 1 - 多平台统一识别八类监控场景 (Priority: P1) MVP

**Goal**: JVM 和 Android 使用同一套共享场景识别语义，识别工程机械、水面异物、非运维人员入侵、设施位移、水体颜色变化、船舶异常停留、闸坝状态和非法排污。

**Independent Test**: 使用八类样例输入分别运行 common 场景处理器、JVM runtime 和 Android host
runtime，验证场景类型、任务标识和输出语义一致。

### Tests for User Story 1 (Kotest first)

- [X] T014 [P] [US1] 为八类场景处理器补齐成功识别和无目标输入 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/scenario/ScenarioProcessorTests.kt`
- [X] T015 [P] [US1] 为场景标签归一化、属性映射和连续命中辅助逻辑补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/scenario/engine/ScenarioSupportTest.kt`
- [X] T016 [P] [US1] 为 ScenarioEngine 输出八类场景观测补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/scenario/engine/ScenarioEngineTest.kt`
- [X] T017 [P] [US1] 为 JVM runtime 八类场景识别烟测补齐 Kotest 覆盖，更新
  `monitor_detection/src/jvmTest/kotlin/com/smartwater/ai/runtime/jvm/JvmMonitoringRuntimeTest.kt`
- [X] T018 [P] [US1] 为 Android host runtime 八类场景识别烟测补齐 Kotest 覆盖，更新
  `monitor_detection/src/androidHostTest/kotlin/com/smartwater/ai/runtime/android/AndroidMonitoringRuntimeTest.kt`

### Implementation for User Story 1

- [X] T019 [P] [US1] 完成工程机械作业目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/machinery/MachineryScenarioProcessor.kt`
- [X] T020 [P] [US1] 完成水面异物目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/floating/FloatingObjectScenarioProcessor.kt`
- [X] T021 [P] [US1] 完成非运维人员入侵目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/intrusion/IntrusionScenarioProcessor.kt`
- [X] T022 [P] [US1] 完成监测设施位移目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/displacement/FacilityDisplacementScenarioProcessor.kt`
- [X] T023 [P] [US1] 完成水体颜色变化目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/watercolor/WaterColorScenarioProcessor.kt`
- [X] T024 [P] [US1] 完成船舶异常停留目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/vessel/VesselStayScenarioProcessor.kt`
- [X] T025 [P] [US1] 完成闸坝开启、关闭和未知状态识别输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/gate/GateStatusScenarioProcessor.kt`
- [X] T026 [P] [US1] 完成非法排污目标标签、事件标题和属性输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/discharge/IllegalDischargeScenarioProcessor.kt`
- [X] T027 [US1] 串联处理器注册、观测生成和无目标输入空结果路径，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/engine/ScenarioEngine.kt`
- [X] T028 [US1] 对齐 JVM 和 Android 默认视觉输出标签映射，更新
  `monitor_detection/src/jvmMain/kotlin/DefaultVisionEngine.jvm.kt` 和
  `monitor_detection/src/androidMain/kotlin/platform/DefaultVisionEngine.android.kt`
- [X] T029 [US1] 校验 JVM 与 Android runtime 默认任务装配使用同一场景注册表，更新
  `monitor_detection/src/jvmMain/kotlin/com/smartwater/ai/runtime/jvm/JvmMonitoringRuntime.kt` 和
  `monitor_detection/src/androidMain/kotlin/com/smartwater/ai/runtime/android/AndroidMonitoringRuntime.kt`

**Checkpoint**: User Story 1 可独立运行和验证，是本功能 MVP。

---

## Phase 4: User Story 2 - 按规则生成预警并聚合告警 (Priority: P2)

**Goal**: 根据任务规则判断识别结果是否触发预警，并将同一任务、设备和风险类型的持续事件聚合为同一条打开告警。

**Independent Test**: 为命中、未命中、区域外、连续命中不足和重复事件输入样例，验证告警创建、更新和不触发路径。

### Tests for User Story 2 (Kotest first)

- [X] T030 [P] [US2] 为置信度阈值、目标标签、区域条件和连续命中规则补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/core/rule/DefaultRuleEvaluatorTest.kt`
- [X] T031 [P] [US2] 为同一 eventKey 告警聚合和 lastSeenAt 更新补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/core/alarm/AlarmAggregatorTest.kt`
- [X] T032 [P] [US2] 为 ScenarioEngine 规则判定、区域外不告警和重复事件聚合补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/scenario/engine/ScenarioEngineTest.kt`
- [X] T033 [P] [US2] 为告警 DTO 输出字段和 eventKey 合同补齐 Kotest 覆盖，新增
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/data/contract/AlarmDtoPublishersTest.kt`

### Implementation for User Story 2

- [X] T034 [US2] 实现规则判定的默认规则、禁用任务、目标过滤、阈值、区域和连续命中逻辑，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/rule/RuleEngine.kt`
- [X] T035 [US2] 统一八类场景 eventKey、title、details 和 alarmLevel 生成方式，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/scenario/engine/ScenarioSupport.kt`
- [X] T036 [US2] 实现同一 eventKey 打开告警复用和 lastSeenAt 更新，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/core/alarm/AlarmAggregator.kt`
- [X] T037 [US2] 输出符合合同的告警记录 DTO 和内存发布器行为，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/data/contract/AlarmDtoPublishers.kt`
- [X] T038 [US2] 为八类场景提供可解释默认规则集，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/RuleSetProvider.kt`
- [X] T039 [US2] 将规则判定、告警聚合和发布器接入运行时主流程，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/DirectMonitoringRuntime.kt`

**Checkpoint**: User Story 1 和 2 均可独立验证，识别结果能转化为可查询告警。

---

## Phase 5: User Story 3 - 输出证据、任务状态和设备健康 (Priority: P3)

**Goal**: 提供任务列表、任务状态、设备健康、告警列表、规则配置和证据元数据输出，支持平台对接与现场复核。

**Independent Test**: 启动任务并模拟识别、告警、异常和停止过程，验证任务状态、设备健康、告警和证据字段完整。

### Tests for User Story 3 (Kotest first)

- [X] T040 [P] [US3] 为任务列表、任务状态、设备健康、规则配置和证据查询补齐 Kotest 覆盖，更新
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/runtime/contract/PlatformServicesTest.kt`
- [X] T041 [P] [US3] 为运行时启动、心跳、错误和停止状态转移补齐 Kotest 覆盖，新增
  `monitor_detection/src/commonTest/kotlin/com/smartwater/ai/runtime/contract/DirectMonitoringRuntimeTest.kt`
- [X] T042 [P] [US3] 为 JVM bootstrap 默认任务、规则和健康输出补齐 Kotest 覆盖，更新
  `monitor_detection/src/jvmTest/kotlin/com/smartwater/ai/runtime/jvm/JvmMonitoringRuntimeTest.kt`
- [X] T043 [P] [US3] 为 Android bootstrap 默认任务、规则和健康输出补齐 Kotest 覆盖，更新
  `monitor_detection/src/androidHostTest/kotlin/com/smartwater/ai/runtime/android/AndroidMonitoringRuntimeTest.kt`

### Implementation for User Story 3

- [X] T044 [US3] 实现 ApiExposureService 对任务、状态、健康、规则、告警和证据的统一查询输出，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/PlatformServices.kt`
- [X] T045 [US3] 在运行时维护任务状态、心跳、最近帧、错误和证据元数据关联，更新
  `monitor_detection/src/commonMain/kotlin/com/smartwater/ai/runtime/contract/DirectMonitoringRuntime.kt`
- [X] T046 [US3] 对齐 JVM 默认视频源、视觉引擎、任务和规则装配，更新
  `monitor_detection/src/jvmMain/kotlin/com/smartwater/ai/runtime/jvm/JvmRuntimeBootstrap.kt`
- [X] T047 [US3] 对齐 Android 默认视频源、视觉引擎、任务、规则和前台服务装配，更新
  `monitor_detection/src/androidMain/kotlin/com/smartwater/ai/runtime/android/AndroidRuntimeBootstrap.kt`
  和 `monitor_detection/src/androidMain/kotlin/service/AlertForegroundService.kt`

**Checkpoint**: 三个用户故事均可独立运行，运行状态和证据输出可供复核。

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 文档、合同、格式和验收报告收尾。

- [X] T048 [P] 根据最终实现同步场景输出合同，更新
  `specs/001-kmp-monitor-alerts/contracts/scenario-output-contract.md`
- [X] T049 [P] 根据最终实现同步运行时查询合同，更新
  `specs/001-kmp-monitor-alerts/contracts/runtime-query-contract.md`
- [X] T050 [P] 更新项目说明中的功能范围和测试入口，更新 `README.md`
- [X] T051 运行
  `./gradlew :monitor_detection:compileCommonMainKotlinMetadata :monitor_detection:compileKotlinJvm :monitor_detection:compileAndroidMain`
  并将结果记录到 `specs/001-kmp-monitor-alerts/quickstart.md`
- [X] T052 运行 `./gradlew :monitor_detection:jvmTest` 并将结果记录到
  `specs/001-kmp-monitor-alerts/quickstart.md`
- [X] T053 运行 `./gradlew :monitor_detection:testAndroidHostTest` 并将结果记录到
  `specs/001-kmp-monitor-alerts/quickstart.md`
- [X] T054 运行 `./gradlew :monitor_detection:allureReport --no-configuration-cache` 并确认报告目录
  `monitor_detection/build/reports/allure-report/allureReport`
- [X] T055 运行 `./gradlew :monitor_detection:ktlintCheck` 并将结果记录到
  `specs/001-kmp-monitor-alerts/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 无依赖，可立即执行
- **Foundational (Phase 2)**: 依赖 Setup，阻塞全部用户故事
- **User Story 1 (Phase 3)**: 依赖 Foundational，是 MVP
- **User Story 2 (Phase 4)**: 依赖 Foundational，可在 US1 后集成验证，也可与 US1 并行开发
- **User Story 3 (Phase 5)**: 依赖 Foundational，可在 US1/US2 之后做端到端复核，也可并行开发运行时输出
- **Polish (Phase 6)**: 依赖选定交付范围完成

### User Story Dependencies

- **US1 (P1)**: 八类场景识别，MVP 范围，无其他用户故事依赖
- **US2 (P2)**: 规则与告警聚合，依赖共享模型和规则基础，可独立用样例事件验证
- **US3 (P3)**: 状态、健康和证据输出，依赖共享运行时契约，可独立模拟任务生命周期验证

### Within Each User Story

- 先编写或补齐 Kotest 测试，并确认测试在实现前能暴露缺口
- 先 commonMain 共享行为，再 JVM/Android 适配
- 先场景观测，再规则判定，再告警聚合，再 DTO/运行时输出
- 每个用户故事完成后运行对应 Kotest，再进入下一优先级

### Parallel Opportunities

- T002、T003、T004 可并行处理配置和文档
- T005、T006、T007、T008、T011、T012、T013 可在不同文件中并行推进
- US1 的 T019 到 T026 可由不同人员分别处理八类场景处理器
- US2 的规则测试、聚合测试和 DTO 测试可并行编写
- US3 的 common、JVM、Android host 测试可并行编写

---

## Parallel Example: User Story 1

Task: "T014 [P] [US1] 更新 ScenarioProcessorTests.kt"
Task: "T015 [P] [US1] 更新 ScenarioSupportTest.kt"
Task: "T017 [P] [US1] 更新 JvmMonitoringRuntimeTest.kt"
Task: "T018 [P] [US1] 更新 AndroidMonitoringRuntimeTest.kt"

Task: "T019 [P] [US1] 更新 MachineryScenarioProcessor.kt"
Task: "T020 [P] [US1] 更新 FloatingObjectScenarioProcessor.kt"
Task: "T021 [P] [US1] 更新 IntrusionScenarioProcessor.kt"
Task: "T022 [P] [US1] 更新 FacilityDisplacementScenarioProcessor.kt"
Task: "T023 [P] [US1] 更新 WaterColorScenarioProcessor.kt"
Task: "T024 [P] [US1] 更新 VesselStayScenarioProcessor.kt"
Task: "T025 [P] [US1] 更新 GateStatusScenarioProcessor.kt"
Task: "T026 [P] [US1] 更新 IllegalDischargeScenarioProcessor.kt"


## Parallel Example: User Story 2


Task: "T030 [P] [US2] 更新 DefaultRuleEvaluatorTest.kt"
Task: "T031 [P] [US2] 更新 AlarmAggregatorTest.kt"
Task: "T033 [P] [US2] 新增 AlarmDtoPublishersTest.kt"


## Parallel Example: User Story 3


Task: "T040 [P] [US3] 更新 PlatformServicesTest.kt"
Task: "T041 [P] [US3] 新增 DirectMonitoringRuntimeTest.kt"
Task: "T042 [P] [US3] 更新 JvmMonitoringRuntimeTest.kt"
Task: "T043 [P] [US3] 更新 AndroidMonitoringRuntimeTest.kt"


---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1 和 Phase 2
2. 完成 Phase 3 中 US1 的 Kotest 测试和八类场景处理器
3. 运行 `./gradlew :monitor_detection:jvmTest`
4. 运行 `./gradlew :monitor_detection:testAndroidHostTest`
5. 生成 `./gradlew :monitor_detection:allureReport --no-configuration-cache`

### Incremental Delivery

1. Setup + Foundational: 保证 KMP 共享模型、规则、场景引擎和测试夹具可用
2. US1: 交付八类场景识别 MVP，并验证 JVM/Android 语义一致
3. US2: 增加规则判定和告警聚合，验证不重复告警和不触发路径
4. US3: 增加任务状态、设备健康和证据元数据输出
5. Polish: 同步合同、文档、格式检查和 Allure 报告

### Validation Commands


./gradlew :monitor_detection:compileCommonMainKotlinMetadata :monitor_detection:compileKotlinJvm :monitor_detection:compileAndroidMain
./gradlew :monitor_detection:jvmTest
./gradlew :monitor_detection:testAndroidHostTest
./gradlew :monitor_detection:allureReport --no-configuration-cache
./gradlew :monitor_detection:ktlintCheck


## Notes

- [P] 任务必须保持不同文件或无直接依赖，避免同一文件并发修改冲突
- 用户故事阶段的任务均带有 [US1]、[US2] 或 [US3] 标签
- 本任务清单不包含 Web UI、Android 展示 UI、告警大屏或管理后台
- 行为变更以 Kotest 为准，交付复核以 Allure 报告为准
```


### 9.  **最后：AI 实施生成代码**

SDD流程的 spec 和 plan 都完成后开始让AI生成项目代码

**使用：**

```
$specify-implement
```



### 10.  一致性和覆盖率分析

代码生成之后检查实际和 `spec.md`, `plan.md`, `tasks.md `，`constitution.md` 描述内容是否一致以及是否覆盖到

**使用：**

```
$specify-analyze
```

目前默认分析结果是在对话窗口中输出的，可以指定让AI帮忙把结果沉淀到文档里



## 3. 参考

[spec-kit](https://github.com/github/spec-kit/tree/main)

[完整的规格驱动开发(SDD)方法论](https://github.com/github/spec-kit/blob/main/spec-driven.md)

[Supported AI Coding Agent Integrations](https://github.github.io/spec-kit/reference/integrations.html)