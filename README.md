# PPT Agent

基於 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的多智能體 PPT 幻燈片生成工作流。

由 Claude 生成 + Gemini 審查，輸出 SVG 1280×720 Bento Grid 佈局的簡報幻燈片。

## 效果展示

> `/ppt-agent:ppt 幫我收集一下新一代小米su7的發布會資料然後做一套PPT`

| | | |
|:---:|:---:|:---:|
| ![封面](docs/images/slide-01.png) | ![產品定位](docs/images/slide-02.png) | ![外觀設計](docs/images/slide-03.png) |
| ![性能參數](docs/images/slide-04.png) | ![標配亮點](docs/images/slide-05.png) | ![智能座艙](docs/images/slide-06.png) |
| ![智能駕駛](docs/images/slide-07.png) | ![安全體系](docs/images/slide-08.png) | ![續航充電](docs/images/slide-09.png) |
| ![產品矩陣](docs/images/slide-10.png) | ![生態佈局](docs/images/slide-11.png) | ![結尾](docs/images/slide-12.png) |

## 安裝

```bash
claude plugin marketplace add zengwenliang416/ppt-agent
claude plugin install ppt-agent
```

## 使用

```
/ppt-agent:ppt <主題或需求描述>
```

## 工作流程

1. **初始化** — 解析參數，創建運行目錄
2. **需求調研** — 背景搜索 + 用戶確認需求
3. **素材收集** — 按章節並行深度搜索
4. **大綱規劃** — 金字塔原理結構化大綱 + 用戶審批
5. **規劃草稿** — 每頁生成簡版 SVG 草稿
6. **設計稿 + 審查** — Bento Grid SVG 生成 + Gemini 質量審查循環
7. **交付** — 最終 SVG 檔案 + 互動式 HTML 預覽頁

## 智能體

| 智能體 | 職責 |
|--------|------|
| `research-core` | 需求調研 + 素材收集 |
| `content-core` | 大綱規劃 + 規劃草稿 |
| `slide-core` | 設計 SVG 生成（Bento Grid 佈局） |
| `review-core` | Gemini 驅動的 SVG 質量審查 |

## 輸出目錄

```
openspec/changes/<run_id>/
├── research-context.md      # 調研上下文
├── requirements.md          # 需求文檔
├── materials.md             # 素材彙總
├── outline.json             # 結構化大綱
├── drafts/slide-{nn}.svg    # 規劃草稿
├── slides/slide-{nn}.svg    # 設計稿
├── reviews/review-{nn}.md   # 審查報告
└── output/
    ├── slide-{nn}.svg       # 最終 SVG
    └── index.html           # 互動式預覽頁
```

## 許可證

MIT
