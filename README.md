# Act as a pathology resident,you have to present a seminar on the topic iron deficiency anemia,create a slide presentation with detailed content and suitable images


多智能体 PPT 幻灯片生成工作流，支持跨平台运行。

LLM 生成 + Gemini 审查，输出 SVG 1280×720 Bento Grid 布局的演示幻灯片。

## 效果展示

同一提示词、同一工作流，不同模型 × 不同宿主的对比效果：

> `/ppt-agent:ppt 帮我收集一下新一代小米su7的发布会资料然后做一套PPT`

---

### GPT-5.4 · OpenCode

**这是 OpenCode 中 GPT-5.4 的效果** | 深色科技蓝橙配色 | 12页 | 平均质量分 8.53/10

| | | |
|:---:|:---:|:---:|
| ![封面](docs/images/gpt54/slide-01.svg) | ![为什么是新一代](docs/images/gpt54/slide-02.svg) | ![价格版本](docs/images/gpt54/slide-03.svg) |
| ![设计焕新](docs/images/gpt54/slide-04.svg) | ![座舱焕新](docs/images/gpt54/slide-05.svg) | ![三电与性能](docs/images/gpt54/slide-06.svg) |
| ![补能与底盘](docs/images/gpt54/slide-07.svg) | ![智能座舱](docs/images/gpt54/slide-08.svg) | ![智能驾驶](docs/images/gpt54/slide-09.svg) |
| ![安全体系](docs/images/gpt54/slide-10.svg) | ![竞品对比](docs/images/gpt54/slide-11.svg) | ![结尾](docs/images/gpt54/slide-12.svg) |

---

### MiniMax M2.5 · OpenCode

**当前运行效果** | 深蓝商务橙配色 | 14页 | 平均质量分 8.5/10

| | | |
|:---:|:---:|:---:|
| ![封面](docs/images/minimax/slide-01.svg) | ![车型概览](docs/images/minimax/slide-02.svg) | ![三款配置](docs/images/minimax/slide-03.svg) |
| ![设计美学](docs/images/minimax/slide-04.svg) | ![九色系列](docs/images/minimax/slide-05.svg) | ![内饰豪华](docs/images/minimax/slide-06.svg) |
| ![内饰设计](docs/images/minimax/slide-07.svg) | ![安全升级](docs/images/minimax/slide-08.svg) | ![安全配置](docs/images/minimax/slide-09.svg) |
| ![智能科技](docs/images/minimax/slide-10.svg) | ![智能驾驶](docs/images/minimax/slide-11.svg) | ![三电升级](docs/images/minimax/slide-12.svg) |
| ![底盘升级](docs/images/minimax/slide-13.svg) | ![价格发布](docs/images/minimax/slide-14.svg) | |

---

### MiMo V2 Pro · OpenCode

| | | |
|:---:|:---:|:---:|
| ![封面](docs/images/slide-01.svg) | ![市场势能](docs/images/slide-02.svg) | ![设计焕新](docs/images/slide-03.svg) |
| ![座舱焕新](docs/images/slide-04.svg) | ![性能表现](docs/images/slide-05.svg) | ![超长续航](docs/images/slide-06.svg) |
| ![极速充电](docs/images/slide-07.svg) | ![HAD辅助驾驶](docs/images/slide-08.svg) | ![XLA认知大模型](docs/images/slide-09.svg) |
| ![蛟龙底盘](docs/images/slide-10.svg) | ![全方位防护](docs/images/slide-11.svg) | ![价格发布](docs/images/slide-12.svg) |
| ![核心优势](docs/images/slide-13.svg) | ![结尾](docs/images/slide-14.svg) | |

**小米品牌橙 #FF6900** | 深色科技风格 | 平均质量分 8.34/10

---

### Claude Opus · Claude Code

| | | |
|:---:|:---:|:---:|
| ![封面](docs/images/claude/slide-01.png) | ![产品定位](docs/images/claude/slide-02.png) | ![外观设计](docs/images/claude/slide-03.png) |
| ![性能参数](docs/images/claude/slide-04.png) | ![标配亮点](docs/images/claude/slide-05.png) | ![智能座舱](docs/images/claude/slide-06.png) |
| ![智能驾驶](docs/images/claude/slide-07.png) | ![安全体系](docs/images/claude/slide-08.png) | ![续航充电](docs/images/claude/slide-09.png) |
| ![产品矩阵](docs/images/claude/slide-10.png) | ![生态布局](docs/images/claude/slide-11.png) | ![结尾](docs/images/claude/slide-12.png) |

---

## 安装

```bash
claude plugin marketplace add zengwenliang416/ppt-agent
claude plugin install ppt-agent
```

## 使用

```
/ppt-agent:ppt <主题或需求描述>
```

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--style` | business | 风格预设；实际可用值以 `skills/_shared/index.json` 中 `domain=style` 的注册表为准 |
| `--brand-colors` | 无 | 品牌色彩 YAML 文件路径 |
| `--pages` | 10-15 | 目标页数范围 |
| `--run-id` | 自动生成 | 恢复已有运行目录，按已有产物断点续跑 |

当前内置 style 注册表包含 17 个风格：`business`、`tech`、`creative`、`minimal`、`blueprint`、`bold-editorial`、`chalkboard`、`editorial-infographic`、`fantasy-animation`、`intuition-machine`、`notion`、`pixel-art`、`scientific`、`sketch-notes`、`vector-illustration`、`vintage`、`watercolor`。

### 品牌色彩自定义

```yaml
brand:
  primary: "#FF6900"     # 主品牌色
  secondary: "#000000"   # 辅助品牌色
  logo_text: "Mi"        # 品牌标识（2-3字符）
```

使用方式：
```
/ppt-agent:ppt --brand-colors=brand.yaml 小米SU7发布会
```

## 工作流程

1. **初始化** — 解析参数，创建运行目录
2. **需求调研** — 背景搜索 + 用户确认需求（Hard Stop）
3. **素材收集** — 按章节并行深度搜索
4. **大纲规划** — 金字塔原理结构化大纲 + 用户审批（Hard Stop）
5. **规划草稿** — 每页生成简版 SVG 草稿
6. **设计稿 + 审查** — Bento Grid SVG 生成 + Gemini 质量审查循环
7. **交付** — 最终 SVG 文件 + 交互式 HTML 预览页（Hard Stop）

说明：当 Gemini 不可用时，审查阶段会降级为**技术校验模式**，继续检查 XML、`viewBox`、字号下限、安全边距、对比度与样式 token 合规性，但不再生成审美优化建议。

## 智能体

| 智能体 | 职责 |
|--------|------|
| `research-core` | 需求调研 + 素材收集 |
| `content-core` | 大纲规划 + 规划草稿 |
| `slide-core` | 设计 SVG 生成（Bento Grid 布局） |
| `review-core` | Gemini 驱动的 SVG 质量审查 |

## 输出目录

```
openspec/changes/<run_id>/
├── input.md                 # 输入参数
├── proposal.md              # 变更提案
├── tasks.md                 # 任务清单
├── research-context.md      # 调研上下文
├── requirements.md          # 需求文档
├── materials.md             # 素材汇总
├── style.yaml               # 样式配置
├── outline.json             # 结构化大纲
├── outline-preview.md       # 大纲预览
├── draft-manifest.json      # 草稿清单
├── drafts/slide-{nn}.svg    # 规划草稿
├── slides/slide-{nn}.svg    # 设计稿
├── slide-status.json        # 逐页进度（支持中断恢复）
├── reviews/review-{nn}.md   # 审查报告
├── review-manifest.json     # 审查汇总（Phase 6→7 质量门控）
└── output/
    ├── slide-{nn}.svg       # 最终 SVG
    ├── index.html           # 交互式预览页
    └── speaker-notes.md     # 演讲者备注
```

## 质量标准

| 指标 | 最低要求 | 优秀标准 |
|------|----------|----------|
| 加权总分 | ≥ 7.0 | ≥ 8.5 |
| 布局评分 | ≥ 6 | ≥ 8 |
| 可读性 | ≥ 6 | ≥ 8 |
| 修复轮次 | ≤ 2 | 0 |

补充说明：以上加权评分门槛适用于 Gemini 可用时的完整审查链路；若进入技术校验模式，则以硬规则通过/失败作为放行标准，不生成审美分数。

## 跨平台适配路线图

PPT Agent 当前已验证多个平台和模型：

| 平台 | 模型 | 状态 | 备注 |
|------|------|------|------|
| [OpenCode](https://opencode.ai) | GPT-5.4 | ✅ 已验证 | 新一代小米 SU7 12页效果 |
| [OpenCode](https://opencode.ai) | MiniMax M2.5 | ✅ 已验证 | 当前运行效果 |
| [OpenCode](https://opencode.ai) | MiMo V2 Pro | ✅ 已验证 | 小米品牌橙配色 |
| [Claude Code](https://claude.ai) | Claude Opus | ✅ 已验证 | 经典商务风格 |

### 长期方向

- **MCP Server 化**：将核心工作流封装为 MCP Server，暴露 `ppt/generate`、`ppt/outline`、`ppt/review` 等标准 tools。所有支持 MCP 的宿主（Claude Code / Codex / OpenCode / Droid / Cursor / Zed 等）均可直接接入，一次开发全平台通用。
- **核心解耦**：将 7 阶段工作流逻辑与宿主特有协议（Task / SendMessage / AskUserQuestion）分离为平台无关的 `core/` 层，新增平台只需编写轻量 adapter。
- **Headless 模式**：支持无交互批量生成，适用于 CI/CD pipeline 和 API 调用场景。

## 许可证

MIT
