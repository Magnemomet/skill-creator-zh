# skill-creator-zh

`skill-creator-zh` 是 `skill-creator` 的中文安全化与能力等价性增强版本，用于创建新 skill、改进已有 skill、评估 skill 表现、运行 eval/benchmark、优化 skill description，并验证翻译或本地化后的 skill 是否保持原能力。

本仓库保留原 `skill-creator` 的核心工作流和文件结构，同时将正文说明中文化，并额外强调工程可迁移性、结构化输入校验、失败路径测试与相邻 skill 的触发关系。

## 原始 skill 标注

本项目基于原始 skill：

```text
skill-creator
```

原始 skill 的核心定位：

> Create new skills, modify and improve existing skills, and measure skill performance.

`skill-creator-zh` 的目标不是另起一套不同工作流，而是在保持原版能力结构、测试流程和工程严谨性的前提下，提供中文说明、安全本地化约束，以及翻译/本地化后能力一致性评估指引。

## 核心能力

- 创建新 skill；
- 修改、优化和重构已有 skill；
- 将英文 skill 安全中文化或本地化；
- 检查翻译后的 skill 是否保持原版能力；
- 设计 evals、assertions 和 benchmark；
- 比较不同 skill 版本；
- 优化 frontmatter `description` 的触发准确率；
- 打包和交付 skill。

## 目录结构

```text
skill-creator-zh/
├── SKILL.md
├── README.md
├── LICENSE.txt
├── assets/
│   └── eval_review.html
├── eval-viewer/
│   ├── generate_review.py
│   └── viewer.html
├── references/
│   └── schemas.md
└── scripts/
    ├── aggregate_benchmark.py
    ├── generate_report.py
    ├── improve_description.py
    ├── package_skill.py
    ├── quick_validate.py
    ├── run_eval.py
    ├── run_loop.py
    └── utils.py
```

## 安装方式

将本目录放入 Hanako 的 skills 目录，例如：

```text
C:\Users\<you>\.hanako\skills\skill-creator-zh
```

新会话或刷新 skill 加载后，应能看到：

```text
skill-creator-zh
```

## 使用示例

```text
帮我创建一个能把网页文章整理成 Markdown 笔记的 skill。
```

```text
把这个英文 skill 安全中文化，但不要破坏 JSON 字段、命令和路径约定。
```

```text
评估这个翻译后的 skill 是否和原版制作能力一致。
```

```text
给这个 skill 设计 evals，包含成功路径和失败/降级路径。
```

## 关键原则

### 1. 中文化说明，保留接口

可以翻译自然语言说明，但不要翻译：

- `SKILL.md`、`evals/evals.json`、`grading.json`、`benchmark.json` 等文件名；
- `skill_name`、`evals`、`prompt`、`assertions`、`text`、`passed`、`evidence` 等 JSON 字段；
- `with_skill/`、`without_skill/`、`old_skill/` 等目录名；
- 命令、脚本参数、占位符和代码块。

### 2. 路径可迁移性

可复用 skill 不应写死当前机器的绝对路径。优先使用：

```text
<skill-dir>/scripts/...
<workspace>/...
scripts/example.py
```

只有在记录临时测试结果或用户明确指定路径时，才使用绝对路径。

### 3. 结构化输入校验

如果 bundled script 读取 JSON、YAML、CSV、表格或其他结构化输入，应校验：

- 必填字段；
- 字段类型；
- 空列表是否允许；
- 输入文件是否存在；
- 输出路径是否可写。

不要静默生成空文档、空报告或看似成功但内容缺失的文件。

### 4. 测试覆盖失败路径

除了正常成功路径，evals 至少应覆盖一个失败/降级场景，例如：

- 输入缺字段；
- 文件不存在；
- 依赖缺失；
- 网络不可用；
- 输出路径不可写；
- 用户要求超出当前能力。

### 5. 区分 mock/offline eval 与 real-world smoke test

依赖实时网络、外部 API 或时间敏感数据的 skill，应分层测试：

- `mock/offline eval`：稳定、可重复，适合 benchmark；
- `real-world smoke test`：验证当前真实链路可用，作为补充观察。

### 6. 说明与相邻 skill 的主次关系

如果新 skill 与已有 skill 的能力有交叉，要说明主次关系。主 skill 由用户的主要目标决定，其他 skill 可作为辅助能力使用。

例如，用户说“收集 AI 新闻并写成 Word 日报”时，主目标是 AI 新闻日报，Word 只是输出格式。此时日报 skill 应作为主流程，`office-documents` 只用于读取、验证或编辑 `.docx`。

## 与原版相比的增强点

- 增加中文安全化说明；
- 增加翻译/本地化后能力一致性评估触发；
- 补强路径可迁移性要求；
- 补强结构化输入与脚本校验要求；
- 补强失败/降级测试要求；
- 补充 `mock/offline eval` 与 `real-world smoke test` 分层；
- 补充与其他 skill 的主次关系说明。

## License

本仓库随附 `LICENSE.txt`，采用 Apache License 2.0。

请在再分发或修改时保留原始 `skill-creator` 标注，并明确说明自己的修改。
