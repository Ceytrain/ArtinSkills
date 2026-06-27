# ArtinSkills

自定义 OpenCode Agent Skills 仓库。

## 目录结构

```
skills/
└── custom/
    ├── html-report/            # HTML 报告生成
    └── embedded-widget-dev/    # 嵌入式 Widget 开发
```

## Skills 说明

| Skill | 说明 |
|-------|------|
| html-report | HTML 报告生成（三色、双主题、侧边栏导航、Maple Mono NF CN 字体） |
| embedded-widget-dev | 嵌入式 Widget 开发指南（ARM Cortex-M + Qt PC 端） |

## 安装方式

将 Skill 目录复制到以下位置之一：

- **全局安装**：`%USERPROFILE%\.config\opencode\skills\{skill-name}\`
- **项目级安装**：项目根目录 `.agents/skills/{skill-name}/`

## 自定义 Skill

创建新目录，添加 `SKILL.md`，包含以下内容：

- **Triggers**：触发条件描述
- **Instructions**：详细的工作流程和约束
- **References**：参考模板或代码片段（可选）
