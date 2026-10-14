# lab-report

面向 AI agent 的 skill，实现大学物理实验的**手写记录纸照片 → 电子原始数据 → 计算 → 报告 PDF** 链路自动化。

做 UCAS 基础物理实验时需要，总结成 skill 自用。Codex 与 Claude Code 都能用。

## 工作流程

| 步骤 | 做什么 |
|---|---|
| **1 转录** | 照片切块放大后逐个读数，认不清直接问用户 |
| **2 核对** | 确认前不进入计算 |
| **3 处理** | 生成 `处理.py`：零点修正、逐差法、不确定度传递、作图法／最小二乘法，并出图 |
| **4 报告** | 按课程模板写成 LaTeX，编译成 PDF |
| **5 审阅** | 开 subagent 攻击**AI味文风** |


## 目录

```
lab-data-workflow/
  SKILL.md               完整流程、读数纪律、踩过的坑、排查表
  scripts/prep_photo.py  照片预处理：总览／转正／切半／裁切放大／补光
  scripts/csv_tables.py  带分段标记的原始数据 CSV 读写
  scripts/to_utf8.py     GBK → UTF-8 BOM（解决 Excel 中文 CSV 乱码）
  templates/report.tex   LaTeX 报告模板（课程官方格式：报告头、三线表、图、公式编号）
  templates/report.typ   Typst 备选模板（不套课程模板时用）
```

## 安装

Codex 只扫 `.agents/skills`，Claude Code 只扫 `.claude/skills`，两个目录默认互不可见。按你实际用的 agent 装到对应位置：

```bash
git clone https://github.com/Helios-YQH/lab-report.git

# Codex
mkdir -p <你的项目>/.agents/skills
cp -r lab-report/lab-data-workflow <你的项目>/.agents/skills/

# Claude Code
mkdir -p <你的项目>/.claude/skills
cp -r lab-report/lab-data-workflow <你的项目>/.claude/skills/
```

### 两个 agent 共用一份（推荐）

正文、脚本、模板只留一份在 `.agents/skills/`，再给 Claude Code 放一个入口文件指过去，避免两份正文各自漂移。

```bash
mkdir -p <你的项目>/.agents/skills <你的项目>/.claude/skills/lab-data-workflow
cp -r lab-report/lab-data-workflow <你的项目>/.agents/skills/
```

然后把下面这段写进 `<你的项目>/.claude/skills/lab-data-workflow/SKILL.md`：

```markdown
---
name: lab-data-workflow
description: 物理实验数据全流程——手写记录纸照片转录成原始数据 CSV、逐项核对、写处理脚本算结果出图、生成实验报告与 PDF。当用户说"录入实验数据""处理实验数据""写实验报告""实验数据出图/出 PDF""把手写的记下来""数据誊录"时使用。
---

正文在 `.agents/skills/lab-data-workflow/SKILL.md`，先完整读取它再执行。
```

之后在项目里说"录入实验数据""处理实验数据""写实验报告"就会自动加载。显式调用：Codex 用 `$lab-data-workflow`，Claude Code 用 `/skills`。Codex 会自动发现新增的 skill，没生效就重开一个对话。

也可以直接让 AI agent 按上面的结构自己装。

## 依赖

Python 3（Pillow、numpy、matplotlib）、XeLaTeX（TeX Live／MiKTeX；字体默认用系统 `SimHei` + `Palatino Linotype`）。
备选的 Typst 路线另需 [Typst](https://typst.app)。

## 说明

自用为主，欢迎其他同学取用和提建议。后面还会继续优化。
