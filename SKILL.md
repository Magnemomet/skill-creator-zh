---
name: skill-creator-zh
description: >
  Create new skills, modify and improve existing skills, evaluate skill quality, run skill evals, benchmark skill behavior, optimize skill descriptions, compare skill versions, validate whether a translated or localized skill preserves the original capability, and package skills. Use this skill whenever the user wants to create a skill from scratch, update an existing skill, translate or localize a skill safely, test whether a translated skill remains equivalent to the original, improve triggering accuracy, run quantitative or qualitative evaluations, compare versions, or package a skill for installation.
  创建新 skill、修改和改进已有 skill、评估 skill 质量、运行 skill eval、做 benchmark、优化 description 触发准确率、比较 skill 版本、验证翻译/本地化后的 skill 是否保持原能力、打包 skill。当用户想要从零创建 skill、更新或优化已有 skill、安全中文化/本地化 skill、测试翻译后的 skill 是否与原版能力等价、设计测试用例、做定量或定性评估、比较不同版本、或生成可安装 skill 时必须使用。
  MANDATORY TRIGGERS: create skill, new skill, improve skill, update skill, translate skill, localize skill, skill eval, benchmark skill, validate skill parity, optimize skill description, package skill, 创建技能, 新建技能, 改进技能, 更新技能, 翻译技能, 中文化技能, 本地化技能, 安全中文化, 等价性评估, 能力一致性, 评估技能, 技能测试, 优化技能描述, 技能打包
---

# Skill Creator 中文安全版

用于创建新 skill、改进已有 skill、评估 skill 表现，并在不破坏工具链约定的前提下进行中文化或本地化改造。

本 skill 是 `skill-creator` 的隔离中文化版本。正文以中文为主，但保留所有会被脚本、工具、viewer、JSON parser、路径解析或 skill loader 依赖的英文接口。使用时遵循一个底层原则：**中文化说明，保留接口**。

---

## 安全中文化原则

翻译或改造 skill 时，先区分两类内容：

1. **说明性内容**：给模型或用户阅读的自然语言，可以翻译、重写、压缩或扩展。
2. **协议性内容**：会被工具链、脚本、文件系统、JSON parser、viewer、命令行或 skill loader 识别的内容，必须保持原样。

不要翻译以下内容：

- skill 目录结构、文件名和目录名，例如 `SKILL.md`、`evals/evals.json`、`eval_metadata.json`、`timing.json`、`grading.json`、`benchmark.json`、`feedback.json`、`scripts/`、`references/`、`assets/`、`outputs/`。
- JSON 字段名，例如 `skill_name`、`evals`、`id`、`prompt`、`expected_output`、`files`、`assertions`、`text`、`passed`、`evidence`、`reviews`、`feedback`、`timestamp`、`status`。
- 命令、脚本名、参数名、占位符，例如 `python -m scripts.aggregate_benchmark`、`--skill-name`、`__EVAL_DATA_PLACEHOLDER__`。
- 作为固定输出格式的一部分被 viewer 或脚本读取的英文目录名，例如 `with_skill/`、`without_skill/`、`old_skill/`。

可以翻译或改写以下内容：

- 标题、段落、用户沟通话术、流程解释、注意事项。
- 示例里的自然语言描述，但示例代码、字段名、命令和路径保持原样。

### 路径可迁移性

不要在可复用 skill 的正文、示例命令或脚本中写死当前机器的绝对路径，例如 `C:\Users\...`、`/Users/...`、`/home/...`。这类路径只适合临时测试记录，不适合作为 skill 的长期说明。

优先使用：

- `<skill-dir>/scripts/...`
- `<workspace>/...`
- 相对路径，例如 `scripts/create_report.py`
- “先定位当前 skill 目录，再拼接 `scripts/...`”这样的步骤说明

只有在记录当前测试结果、临时 workspace、或用户明确指定本机路径时，才可以写绝对路径。生成可迁移 skill 时，示例命令应使用占位符或相对路径，并说明如何解析到真实路径。

---

## 总体工作流

创建或改进 skill 时，按这个循环推进：

1. 明确 skill 要解决什么问题，以及大致如何解决。
2. 写出 `SKILL.md` 草稿，必要时创建 `scripts/`、`references/`、`assets/`。
3. 设计 2-3 个真实测试提示词，让用户确认是否符合预期。
4. 运行测试：有条件时比较 `with_skill` 和 baseline；没有条件时做最小可行的人工/单次测试。
5. 让用户同时看定性输出和定量指标。
6. 根据用户反馈和 benchmark 暴露的问题改写 skill。
7. 重复测试和改进，直到用户满意、反馈为空，或继续迭代已经没有明显收益。
8. 完成后，可以优化 frontmatter 的 `description`，提升触发准确率。
9. 如果需要交付，打包并呈现 `.skill` 文件。

使用本 skill 时，你的职责是判断用户处在这个流程的哪一步，然后接上去推进。用户可能只是说“我想做一个 X 的 skill”，也可能已经有草稿让你改，或者只想让你帮忙中文化、优化触发描述、加测试、打包。

保持灵活。如果用户明确说“不需要跑评估，只想一起打磨一下”，可以跳过完整 benchmark，采用轻量人工检查。

---

## 与用户沟通

用户对代码和评估术语的熟悉程度差异很大。根据上下文选择表达方式。

默认做法：

- 可以使用“评估”“benchmark/基准测试”等词，但第一次出现时可简短解释。
- 使用 `JSON`、`assertion`、`schema`、`frontmatter` 这类术语前，观察用户是否熟悉；不确定时，用一句话解释。
- 避免为了显得专业而堆术语。skill 的价值是把可靠流程固化下来，不是让用户学习一堆工具名。
- 关键风险要说清楚，尤其是文件路径、字段名、命令参数不能随意翻译或改名。

---

## 创建 skill

### 1. 捕捉意图

先理解用户真正想固化的工作流。当前对话里可能已经包含了完整流程，例如用户说“把这个变成 skill”。如果有上下文，先从历史里提取：

- 用过哪些工具；
- 步骤顺序是什么；
- 用户做过哪些纠正；
- 输入和输出格式是什么；
- 哪些结果算成功，哪些结果算失败。

必要时让用户补充，并在进入下一步前获得确认。

至少弄清楚这几个问题：

1. 这个 skill 应该让模型能做什么？
2. 它应该在什么用户表述或场景下触发？
3. 预期输出格式是什么？
4. 是否需要测试用例验证它有效？

判断是否建议测试：

- 文件转换、数据抽取、代码生成、固定流程自动化等客观任务，通常应该有测试用例。
- 写作风格、创意方向、审美判断等主观任务，可以更多依赖用户定性反馈。

### 2. 访谈与调研

主动询问边界条件、输入输出格式、示例文件、成功标准和依赖项。

如果需要查文档或找相似做法，可以使用可用的搜索工具或 subagent 进行调研。调研的目的不是把负担丢给用户，而是带着上下文回来，让用户只确认关键选择。

### 3. 编写 `SKILL.md`

一个 skill 至少包含：

- `name`：skill 标识符，保持简短、稳定、英文 kebab-case。
- `description`：触发机制的核心。写清楚这个 skill 做什么，以及什么时候必须使用。
- Markdown 正文：触发后给模型执行的具体说明。
- 可选资源：`scripts/`、`references/`、`assets/`。

`description` 是最重要的触发入口。所有“什么时候使用这个 skill”的信息都应该放进 `description`，不要只藏在正文里。

当前模型有时会 undertrigger，也就是该用 skill 时没有用。为了降低这个风险，`description` 可以写得稍微主动一些。例如：

```yaml
description: >
  Build simple dashboards and internal data visualizations. Use this skill whenever the user mentions dashboards, charts, metrics, internal data, reporting, or wants to display company data visually, even if they do not explicitly say "dashboard".
  构建仪表盘和内部数据可视化。当用户提到 dashboard、图表、指标、报表、内部数据展示，或想把公司数据可视化时必须使用，即使用户没有明确说“仪表盘”。
```

### 4. 说明与其他 skill 的关系

如果新 skill 与已有 skill 的能力有交叉，要在 `description` 或正文中说明主次关系。主次关系应由用户的主要目标决定，而不是由关键词数量决定。

判断原则：

- 用户的主要目标决定主 skill；
- 文件格式、网页抓取、图片生成、Office 读写等能力可以作为辅助 skill；
- 不要让辅助工具的关键词抢走整个任务的主流程；
- 如果需要调用另一个 skill 或它的脚本，先定位真实源目录，不要猜测 session snapshot 或硬编码路径。

例如，用户说“收集 AI 新闻并写成 Word 日报”时，主目标是 AI 新闻日报，Word 只是输出格式。此时日报 skill 应作为主流程，`office-documents` 只用于读取、验证或编辑 `.docx`。

---

## Skill 写作指南

#### skill架构

一个 skill 的基本结构如下：

```text
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

`SKILL.md` 是必需文件，包含 YAML frontmatter 和正文说明。`scripts/`、`references/`、`assets/` 是可选 bundled resources，用于承载可执行脚本、按需读取的参考文档，以及输出中使用的模板、图标、字体等资产。

#### 渐进式披露

Skills 使用三级加载系统：

1. **Metadata** (`name + description`) - 始终在上下文中，理想长度约 100 词。
2. **`SKILL.md` body** - 当 skill 触发时进入上下文，理想长度小于 500 行。
3. **Bundled resources** - 按需使用，容量不限；脚本可以在不完整读入上下文的情况下执行。

这些词数和行数只是近似建议。如果任务确实需要，可以适当更长，但要保持结构清楚。

**关键限制:**

- 将 `SKILL.md` 控制在 500 行以内；如果接近这个限制，应增加一层结构，把细节放进 bundled resources，并在正文中清楚指向模型下一步应读取或执行的位置。
- 在 `SKILL.md` 中清楚引用 reference files，并说明什么时候读取它们。
- 对大型 reference files（超过 300 行），提供目录或定位说明。

**组织架构**：当一个 skill 支持多个领域或框架时，按变体组织：

```text
cloud-deploy/
├── SKILL.md (workflow + selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

只读取与当前任务相关的 reference file。

#### Principle of Lack of Surprise

这一点不言自明，但仍然必须明确：skill 不能包含恶意软件、利用代码，或任何可能损害系统安全的内容。只要用户理解了 skill 的描述，skill 的内容就不应该让用户感到意外。

不要配合创建误导性 skill，也不要创建旨在协助未授权访问、数据外传或其他恶意行为的 skill。类似“roleplay as an XYZ”的角色扮演 skill 可以接受，前提是它与用户描述的意图一致，并且不越过安全边界。

#### Writing Patterns

优先在说明中使用祈使句，让模型清楚知道要做什么。

**Defining output formats** - 可以这样定义输出格式：

```markdown
## 文本结构
**总是**使用以下模板:
# [题目]
## 执行摘要
## 主要发现
## 建议
```

**示例参考** - 示例很有用。可以这样写示例（如果示例中包含 `输入` 和 `输出`，也可以根据实际情况调整格式）：

```markdown
## 提交信息格式
示例1:
输入:添加了使用JWT令牌的用户身份验证
输出:feat(auth):实现基于JWT的身份验证
```

### 写作风格

尽量解释某条规则为什么重要，而不是只堆叠强硬的 `MUST`。利用模型的 theory of mind，让 skill 具有泛化能力，而不是只覆盖几个狭窄示例。先写一个草稿，再用新鲜视角审视它并改进。

不要滥用全大写 `MUST`、`NEVER`、`ALWAYS`。当某条规则确实是机器接口、安全边界或固定格式时可以强制；普通行为偏好应尽量解释原因，让模型理解任务结构，而不是机械服从。

### 结构化输入与脚本校验

当 skill 包含 bundled script，并且脚本读取 JSON、YAML、CSV、Markdown frontmatter、表格或其他结构化输入时，必须定义并校验输入契约。

至少检查：

- 必填字段是否存在；
- 字段类型是否正确；
- 列表类字段是否允许为空；
- 文件路径字段是否指向真实可读文件；
- 输出路径是否可写；
- 用户要求的关键约束是否能在输入中表达。

如果输入不满足契约，应明确失败并给出可操作错误信息。不要静默生成空文档、空表格、空报告或看似成功但内容缺失的文件。

对于可以程序化检查的契约，优先在 `scripts/` 中实现校验函数；对于只能由模型判断的契约，在 `SKILL.md` 中写清楚检查步骤。

---

## 测试用例

写完 skill 草稿后，设计 2-3 个真实测试提示词，像真实用户会说的话，而不是抽象标签。

先展示给用户确认：

> 我准备用这几个测试用例试一下。它们覆盖了主要场景，你看是否符合预期，或者要不要补充更刁钻的例子？

将测试用例保存到 `evals/evals.json`。先不要写 `assertions`，先只写 prompt 和预期输出说明；`assertions` 可以在测试运行时再补。

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

完整 schema 参考 `references/schemas.md`。

测试用例不要只覆盖成功路径。对于包含脚本、外部网络、文件读写、第三方依赖或复杂输入格式的 skill，至少加入一个失败/降级场景，例如：

- 输入 JSON 缺少必填字段；
- 用户提供的文件不存在；
- `web_search` / `web_fetch` 不可用或返回空；
- 第三方依赖未安装；
- 输出路径不可写；
- 用户要求超出当前工具能力。

这类测试的目的不是让 skill 永远不失败，而是确认它能清楚失败、合理降级，并告诉用户下一步怎么做。

如果 skill 依赖实时网络、外部 API、第三方服务或时间敏感数据，将测试分成两层：

1. **mock/offline eval**：使用固定样例输入，验证脚本、格式、schema、文件输出和降级逻辑。它应该稳定可重复。
2. **real-world smoke test**：少量调用真实网络/API，确认当前环境链路可用。它用于发现现实问题，但不要把实时结果波动当成唯一评分依据。

benchmark 时优先让 mock/offline eval 承担可重复评分，让 real-world smoke test 作为补充观察。

---

## 运行和评估测试用例

这是一个连续流程。开始后尽量不要中途停下，也不要使用 `/skill-test` 或其他测试 skill 替代本流程。

把结果放到 `<skill-name>-workspace/`，该目录作为 skill 目录的 sibling。workspace 内按迭代组织：

```text
<skill-name>-workspace/
└── iteration-1/
    ├── eval-0/
    ├── eval-1/
    └── eval-2/
```

不要提前创建所有目录，按需要创建。

### Step 1: 同一轮启动 with-skill 和 baseline

对每个测试用例，尽量在同一轮启动两个运行：

- `with_skill`：加载当前 skill。
- baseline：根据场景选择无 skill 或旧版本 skill。

这能让结果更可比，避免时间和环境差异过大。

**With-skill run:**

```text
Execute this task:
- Skill path: <path-to-skill>
- Task: <eval prompt>
- Input files: <eval files if any, or "none">
- Save outputs to: <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Outputs to save: <what the user cares about — e.g., "the .docx file", "the final CSV">
```

**Baseline run:**

- 创建新 skill：baseline 不使用 skill，输出到 `without_skill/outputs/`。
- 改进已有 skill：先快照旧版本：`cp -r <skill-path> <workspace>/skill-snapshot/`，baseline 使用快照，输出到 `old_skill/outputs/`。

为每个测试用例写 `eval_metadata.json`。`assertions` 可以先为空。

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": []
}
```

目录名要有描述性，不要只叫 `eval-0`。如果本轮修改了测试 prompt，要为新的 eval 目录重新写 metadata，不要假设上一轮自动继承。

### Step 2: 运行期间起草 assertions

不要单纯等待测试完成。运行期间起草定量 `assertions`，并向用户解释它们检查什么。

好的 assertion 应该：

- 可客观验证；
- 名称或文本清楚；
- 能区分 skill 版本好坏；
- 避免把主观审美硬塞成伪定量指标。

把 assertions 更新进 `eval_metadata.json` 和 `evals/evals.json`。

### Step 3: 捕获 timing 数据

当 subagent 完成时，如果通知里包含 `total_tokens` 和 `duration_ms`，立即把数据保存到对应 run 目录的 `timing.json`：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

这类数据通常只在任务完成通知中出现，不一定能事后恢复。收到就保存，不要等到最后批量处理。

### Step 4: 评分、聚合和打开 viewer

所有运行完成后：

1. **评分**：读取 `agents/grader.md`，用 grader subagent 或内联方式评估每个 assertion。结果保存为 `grading.json`。`grading.json` 的 expectations 数组必须使用字段 `text`、`passed`、`evidence`，不要改成 `name`、`met`、`details` 等其他字段。

2. **聚合 benchmark**：在本 skill 目录下运行：

```bash
python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
```

这会生成 `benchmark.json` 和 `benchmark.md`，包含 pass rate、耗时和 token 统计，以及 mean ± stddev 和 delta。

如果手动生成 `benchmark.json`，先看 `references/schemas.md`，字段必须符合 viewer 预期。

3. **分析结果**：读取 benchmark，找出聚合统计可能掩盖的问题，例如：所有版本都通过的无区分度 assertion、高方差 eval、时间/token 成本异常等。必要时读取 `agents/analyzer.md`。

4. **打开 viewer**：

```bash
nohup python <skill-creator-path>/eval-viewer/generate_review.py \
  <workspace>/iteration-N \
  --skill-name "my-skill" \
  --benchmark <workspace>/iteration-N/benchmark.json \
  > /dev/null 2>&1 &
VIEWER_PID=$!
```

从第二轮开始，增加：

```bash
--previous-workspace <workspace>/iteration-<N-1>
```

在 Windows 或 Hanako 环境中，如果 `nohup`、后台进程或浏览器打开方式不可用，使用当前平台可用方式启动 viewer；命令参数和目录结构仍保持一致。

5. **告诉用户如何查看**：说明 viewer 里通常有两个 tab：`Outputs` 看每个测试用例的输出并写反馈，`Benchmark` 看定量对比。

### 用户在 viewer 中看到什么

`Outputs` tab 通常显示：

- `Prompt`：测试任务。
- `Output`：生成的文件或文本，能内联渲染的尽量内联。
- `Previous Output`：第二轮以后显示上一轮结果。
- `Formal Grades`：如果已评分，显示 assertion 通过/失败。
- `Feedback`：用户输入反馈，会自动保存。
- `Previous Feedback`：第二轮以后显示上一轮反馈。

`Benchmark` tab 显示 pass rate、耗时、token 使用量、各 eval 分解，以及分析观察。

### Step 5: 读取反馈

用户说看完后，读取 `feedback.json`：

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "the chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

空反馈通常表示用户认为可以。优先处理有具体意见的测试用例。

查看完反馈后关闭 viewer server：

```bash
kill $VIEWER_PID 2>/dev/null
```

在 Windows 或当前环境中，如果 `kill` 不适用，使用平台可用方式关闭对应进程。

---

## 改进 skill

改进的核心是把用户反馈转化为通用规则，而不是只对少数测试用例打补丁。

### 思考方式

1. **从反馈中抽象规律**

用户只评估了少数样例，但 skill 未来会面对更多变体。避免过拟合。与其写一堆狭窄的硬编码规则，不如解释任务背后的判断标准和工作策略。

2. **保持 prompt 精简**

读测试 transcript，不只看最终输出。如果模型在无效步骤上浪费大量时间，删除或改写导致浪费的说明。

3. **解释原因**

模型通常能理解意图。比起机械堆 `ALWAYS` 和 `NEVER`，更好的写法是解释为什么某个步骤重要。只有协议字段、安全边界和固定格式需要强制语气。

4. **寻找重复劳动**

如果多个测试用例都让模型临时写相似脚本，例如 `create_docx.py`、`build_chart.py`，说明这些脚本应该变成 bundled resource。把稳定、重复、确定性的部分放进 `scripts/`，让未来调用少走弯路。

### 迭代循环

改完 skill 后：

1. 应用修改。
2. 在 `iteration-<N+1>/` 中重新跑测试，包括 baseline。
3. 打开 reviewer，并传入 `--previous-workspace <workspace>/iteration-<N>`。
4. 等用户看完并反馈。
5. 读取新 `feedback.json`，继续改进。

停止条件：

- 用户说满意；
- 反馈基本为空；
- 新一轮没有明显改进；
- 继续复杂化 skill 的收益小于维护成本。

---

## 高级：盲评比较

如果用户问“新版真的比旧版好吗”，可以做 blind comparison。

读取：

- `agents/comparator.md`
- `agents/analyzer.md`

基本方法是：把两个版本的输出交给独立评审，不告诉它哪个是新版，要求它判断哪个更好并说明理由。然后分析胜出原因。

这一步可选，通常只有在版本差异微妙、用户需要更严谨比较时才做。

---

## Description 优化

`SKILL.md` frontmatter 里的 `description` 是 skill 触发的核心。创建或改进 skill 后，可以主动询问用户是否要优化触发准确率。

### Step 1: 生成触发评估查询

创建 20 条 eval queries，混合 `should_trigger: true` 和 `should_trigger: false`：

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

要求：

- 像真实用户会输入的话；
- 有具体上下文、文件名、需求、边界；
- 包含正式、随意、简写、错别字、长短不同的表达；
- 重点覆盖边界情况，而不是只写一眼能看出的正负例。

坏例子：

```json
"Format this data"
```

好例子：

```json
"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"
```

`should-trigger` 建议 8-10 条，覆盖不同说法和少见用法。

`should-not-trigger` 建议 8-10 条，重点写近邻误触发场景，比如关键词相似但实际应该由另一个 skill 处理。

不要让负例太容易，例如给 PDF skill 写“写一个 fibonacci 函数”没有价值。

### Step 2: 让用户审查 eval set

使用 HTML 模板呈现给用户：

1. 读取 `assets/eval_review.html`。
2. 替换占位符：
   - `__EVAL_DATA_PLACEHOLDER__` → JSON array，不要加额外引号，因为它是 JS 变量赋值的一部分。
   - `__SKILL_NAME_PLACEHOLDER__` → skill 名称。
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → 当前 description。
3. 写入临时 HTML 文件，例如 `/tmp/eval_review_<skill-name>.html`。
4. 用当前环境可用方式打开它。
5. 用户可以编辑 queries、切换 should-trigger、增删条目，然后点击 `Export Eval Set`。
6. 导出的文件通常在 `~/Downloads/eval_set.json`，如有多个，取最近的版本，例如 `eval_set (1).json`。

这一步很重要。坏 eval queries 会诱导出坏 description。

### Step 3: 运行优化循环

告诉用户：

> 这会花一点时间，我会在后台跑优化循环，并定期看进度。

保存 eval set 到 workspace，然后运行：

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id-powering-this-session> \
  --max-iterations 5 \
  --verbose
```

使用当前会话的模型 ID，这样触发测试更接近用户实际体验。

运行期间定期查看输出，告诉用户当前迭代和分数。

该流程会把 eval set 分为 60% train 和 40% held-out test，对当前 description 做多次触发率评估，再提出改写并重复评估。最终选择 test score 最好的 `best_description`，降低过拟合风险。

### Step 4: 应用结果

从 JSON 输出中取 `best_description`，更新 `SKILL.md` frontmatter。向用户展示 before/after 和分数变化。

---

## Hanako/OpenHanako 环境适配

原 `skill-creator` 有些内容来自 Claude Code 语境。使用本中文安全版时，按当前环境能力适配：

- 如果有 subagent，优先用 subagent 并行跑 with-skill 和 baseline。
- 如果没有 subagent，自己逐个运行测试，跳过严格 baseline，重点做定性验证。
- 如果本地浏览器、`nohup`、`open`、`claude -p` 等不可用，不要硬跑；改用当前平台可用工具，或在对话中呈现结果。
- 如果用户要求交付文件，且当前环境有 `stage_files`，用 `stage_files` 呈现真实存在的本地文件。
- 修改已有 skill 时，先定位真实源文件。不要编辑 `sessions/.skill-snapshots`、`session-files` 或旧会话快照。

---

## 没有 subagents 或浏览器时

核心流程仍然是：草稿 → 测试 → 用户反馈 → 改进 → 重复。

### 运行测试

没有 subagents 时，读取目标 skill 的 `SKILL.md`，自己按说明完成测试 prompt。逐个执行。因为你既写 skill 又执行测试，独立性较弱，但仍能做 sanity check。

可以跳过 baseline，专注验证 skill 是否能产出用户可接受的结果。

### 呈现结果

如果不能打开 browser reviewer，就在对话里直接展示：

- 测试 prompt；
- 输出摘要；
- 生成文件路径或已 stage 的文件；
- 你观察到的问题；
- 请用户反馈“哪里需要改”。

### benchmark

没有 subagents 或 baseline 时，定量 benchmark 的意义有限。不要强行制造伪数据，优先做定性反馈。

### description 优化

如果没有 `claude` CLI 或对应优化脚本不可用，跳过自动优化循环。可以手动生成正负触发样例，人工改写 `description` 并让用户确认。

---

## 打包与呈现

如果可用，运行：

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

生成 `.skill` 后，如果用户需要安装包或要求“发给我/给我文件”，使用可用的文件交付工具呈现该文件。只交付真实存在的本地路径。

---

## 参考文件

需要时读取这些资源：

- `agents/grader.md`：如何根据 assertions 评估输出。
- `agents/comparator.md`：如何进行盲评 A/B 比较。
- `agents/analyzer.md`：如何分析 benchmark 结果。
- `references/schemas.md`：`evals.json`、`grading.json`、`benchmark.json` 等结构。

---

## 快速检查清单

创建或改造 skill 完成前，检查：

- `name` 是否稳定、唯一、英文 kebab-case。
- `description` 是否明确写了何时触发，并覆盖中文和英文常见表达。
- 正文是否能指导模型实际完成任务，而不是只描述愿景。
- 是否把长文档、脚本、模板放到了合适的 `references/`、`scripts/`、`assets/`。
- 是否有 2-3 个真实测试 prompt。
- 是否至少有一个失败/降级测试用例。
- 如果依赖实时网络/API，是否区分 mock/offline eval 和 real-world smoke test。
- JSON 字段名、路径、命令、占位符是否保持原样。
- 如果包含 bundled script，是否有清楚的输入契约和失败提示。
- 如果读取结构化输入，是否校验必填字段、类型和空列表。
- 示例命令是否避免写死本机绝对路径。
- 是否说明与相邻 skill 的主次关系，避免触发竞争。
- 如果是中文化改造，是否遵守“中文化说明，保留接口”。
- 是否与原 skill 隔离，避免直接覆盖用户可能还要保留的版本。

祝你把 skill 打磨成一把顺手、稳定、能反复复用的小工具。
