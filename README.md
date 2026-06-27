# ArtinSkills

OpenCode Agent Skills 仓库。用于扩展 AI Agent 的专业领域能力。

## 目录结构

```
skills/
├── custom/                     # 自定义 Skills（本仓库维护）
│   └── html-report/            # HTML 报告生成
│
├── opencode/                   # OpenCode 内置 Skills
│   ├── reference: https://github.com/anomalyco/opencode/tree/dev/.opencode/skills
│   ├── android-native-dev      # Android 原生开发
│   ├── ios-application-dev     # iOS 应用开发
│   ├── flutter-dev             # Flutter 跨平台开发
│   ├── react-native-dev        # React Native 跨平台开发
│   ├── frontend-dev            # 全栈前端开发
│   ├── fullstack-dev           # 全栈后端架构
│   ├── shader-dev              # GLSL 着色器开发
│   ├── embedded-widget-dev     # 嵌入式 Widget 开发
│   ├── gif-sticker-maker       # GIF 表情包生成
│   ├── vision-analysis         # 图片分析与 OCR
│   └── find-skills             # Skill 发现与安装
│
├── opencode-docs/              # OpenCode 文档处理 Skills
│   ├── reference: https://github.com/anomalyco/opencode
│   ├── docling                 # 文档处理与转换
│   ├── minimax-pdf             # PDF 生成与设计
│   ├── minimax-docx            # Word 文档创建与编辑
│   ├── minimax-xlsx            # Excel 电子表格处理
│   ├── pptx-generator          # PowerPoint 演示文稿生成
│   └── pdf-extraction          # PDF 文本与表格提取
│
├── opencode-ui/                # OpenCode UI 组件 Skills
│   ├── reference: https://github.com/anomalyco/opencode
│   ├── shadcn                  # shadcn/ui 组件管理
│   ├── shadcn-vue              # shadcn-vue 组件管理
│   ├── frontend-design         # 前端视觉设计指导
│   └── make-interfaces-feel-better  # 界面打磨优化
│
└── multimodal/                 # 多模态 AI Skills
    ├── reference: https://github.com/anomalyco/opencode
    └── minimax-multimodal-toolkit   # MiniMax 多模态工具集
```

## 安装方式

将 Skill 目录复制到以下位置之一：

- **全局安装**：`%USERPROFILE%\.config\opencode\skills\{skill-name}\`
- **项目级安装**：项目根目录 `.agents/skills/{skill-name}/`

## 自定义 Skill

创建新目录，添加 `SKILL.md`，包含以下内容：

- **Triggers**：触发条件描述
- **Instructions**：详细的工作流程和约束
- **References**：参考模板或代码片段（可选）

## 来源说明

| 前缀 | 来源 | 说明 |
|------|------|------|
| `custom/` | 本仓库 | 自定义开发的 Skills |
| `opencode/` | [anomalyco/opencode](https://github.com/anomalyco/opencode) | OpenCode 内置及社区贡献 |
| `opencode-docs/` | [anomalyco/opencode](https://github.com/anomalyco/opencode) | 文档处理相关 |
| `opencode-ui/` | [anomalyco/opencode](https://github.com/anomalyco/opencode) | UI 组件相关 |
| `multimodal/` | [anomalyco/opencode](https://github.com/anomalyco/opencode) | 多模态 AI 相关 |
