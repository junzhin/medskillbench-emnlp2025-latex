# CLAUDE.md — EMNLP 2025 论文工程

## 当前活跃版本

**ver2/ (V005 bench-only)** — 2026-05-20 起为主线

## 版本对照

| 目录 | 版本 | 标题 | 说明 |
|------|------|------|------|
| 根目录 (`./`) | V002 原版 | MedSkill-E²: Evaluating and Evolving... | 含 GEPA, 历史基线, 不再编辑 |
| `ver1/` | V002 冻结备份 | 同上 | 2026-05-20 快照, 只读 |
| **`ver2/`** | **V005 bench-only** | **Med-SkillBench: Evaluating Medical Agent Skills at Scale** | **当前主线**, 全面去 GEPA |

**编辑规则**: 只编辑 `ver2/` 内文件。根目录和 `ver1/` 冻结不动。

## ver2 论文结构

```
ver2/
├── main.tex                  # 主文件, 标题改为 Med-SkillBench
├── sections/
│   ├── 00_abstract.tex       # 去 GEPA, 以 RBU gap 发现收尾
│   ├── 01_introduction.tex   # 1 gap + 2 contributions + 2 RQs
│   ├── 02_related_work.tex   # 3 subsections (去 §2.4 prompt evolution)
│   ├── 03_method.tex         # §3.1 Med-SkillBench only + Algorithm 1 (评测流程伪代码)
│   ├── 04_experiments.tex    # 评测 + RBU 分析 + benchmark matching + discussion
│   ├── 05_conclusion.tex     # bench 贡献总结
│   ├── 06_limitations.tex    # 4 项 (去 GEPA cost)
│   ├── 07_ethical.tex        # 4 项 (去 GEPA deployment review)
│   └── 99_appendix.tex       # 5 appendix sections (去 GEPA effectiveness)
├── custom.bib
├── acl.sty / acl_natbib.bst
└── math_commands.tex
```

## 自定义 LaTeX 宏 (ver2)

| 宏 | 展开 | 说明 |
|----|------|------|
| `\medskillbench` | Med-SkillBench | 主名称 (ver2 唯一品牌) |
| `\rbu` | RBU | Read-But-Not-Use |
| `\skillmd` | SKILL.md | 技能描述文件 |

**已删除**: `\medskille` (MedSkill-E²), `\gepamed` (GEPA-Med) — ver2 不存在

## Tables 状态 (ver2)

| 表号 | 内容 | 状态 |
|------|------|------|
| Table 1 | 8 维评分总览 (n=1,462) | ✅ V2 真实数据 |
| Table 2 | tool_type 分层 (5 行) | ✅ V2 真实数据 |
| Table 3 | 12 benchmark 匹配率 | ✅ Step1 R1 数据 |
| **Table 4 (待新增)** | **With/Without Skill 对比主表** | **❌ 等 Step 2** |

## Figures 状态 (ver2)

| 图号 | 内容 | 状态 |
|------|------|------|
| Fig 1 | Teaser (RBU gap) | placeholder, 需手绘 |
| Fig 2 | Pipeline (评测全流程) | placeholder, 需手绘 |
| Fig 3 | tool_type bar chart | 数据就绪, 需生成 |
| Fig 4 | Benchmark match rate | 数据就绪, 需生成 |

## Algorithm 状态 (ver2)

| 编号 | 内容 | 状态 |
|------|------|------|
| Algorithm 1 | Med-SkillBench Evaluation Pipeline | ✅ 已写入 03_method.tex |

## Pipeline 依赖 (论文数据来源)

```
Step 0 ✅ → Table 1, 2 (skill 评测)
Step 1 R1 ✅ → Table 3 (benchmark matching)
Step 1 R2 ⏳ (等API) → R3 → Step 2
Step 2 ⏳ → Table 4 (with/without skill 对比) ← 核心缺口
```

详细 pipeline 分析见: `V005_20260520_bench_only_situation_analysis.md`

## 关键规则

- **不在 WSL 中自动编译 LaTeX** (会 crash VSCode server)
- Limitations 节**必须存在**, 缺失 = desk reject
- `\author{Anonymous ACL submission}` — 匿名审稿
- 交叉引用: 用 `\Cref{}` (capitalize, noabbrev), 检查 label 一致性
- ver2 中无任何 GEPA/gepamed/MedSkill-E² 引用 (已验证 grep clean)
