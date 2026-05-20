# 上下文记忆快照

> 时间：2026-05-21 02:30 | 会话主题：数学形式化重写完成 + push | 模型：Opus 4.6 (1M)
> 前序快照：快照_20260521_0045_ver2_bench_only_rewrite.md

---

## 1. 核心目标与全局约束

**核心任务**：将 EMNLP 2025 bench-only 论文 (Med-SkillBench) 的数学形式化从 5 个记账级等式升级到 20+ 个出版级定义/命题/假设，基于两份规划文档 (v1 + v2 errata revision) 执行。

**全局约束**：
- 权威数学源：`ver2/math_formulations/Mathematical Formalization — Complete Revision Plan.md` (v2 revision)
- v2 优先于 v1（修复 3 CRITICAL + 5 MODERATE errata）
- 只编辑 `ver2/` 内文件；ver1/ 冻结
- 绝不暴露开发历史（judge v2→v3、pilot 数据、API 阻塞）
- 不在 WSL 自动编译 LaTeX
- Git repo：`/mnt/d/coding_files/Linux_D_Projects/20260310_medskillbench/05_WRITING_LATEX/EMNLP_2025/`
- Remote：`github.com:junzhin/medskillbench-emnlp2025-latex.git`，branch `main`

---

## 2. 已落地成果（历史沉淀）

### 2.1 已 push 到 remote (commit `bb4327f`)

```
git log --oneline -1
bb4327f feat: ver2 bench-only paper with full mathematical formalization
```

43 files changed, 10,939 insertions. 包含 ver1/ (frozen backup) + ver2/ (bench-only 主线) + 支撑文档。

### 2.2 论文结构 (ver2 最终状态)

| 文件 | 行数 | 关键内容 |
|------|------|---------|
| 00_abstract.tex | 9 | bench + RBU gap (δ_sem=0.242, δ_exec=-0.023) + SGI + ablation(TODO) |
| 01_introduction.tex | 60 | medical motivation + C1(bench) + C2(RBU+decomposition) + C3(ablation) + 3 RQs |
| 02_related_work.tex | 70 | 3 subsections + comparison table (无 GEPA) |
| **03_method.tex** | **350** | **完整形式化重写** — 见下方详表 |
| **04_experiments.tex** | **346** | two-stage decomposition表 + SGI w_c=0.75 + groundability-uplift test |
| 05_conclusion.tex | 16 | bench + RBU decomposition + ablation(TODO) |
| 06_limitations.tex | 23 | 4 项 |
| 07_ethical.tex | 20 | 4 项 |
| **99_appendix.tex** | **305** | 7 节 (eval dims, read-first, domain, benchmark详表, stability, **SGI sensitivity**, **statistical details**) |
| **总计** | **1199** | |

### 2.3 数学形式化完整清单 (03_method.tex)

#### §3.0 Problem Formulation (NEW)
| 构件 | Label | 内容 |
|------|-------|------|
| Definition 1 | def:skill | Skill Specification 4-tuple (desc, params, iface, dom) |
| Definition 2 | def:agent | Agent Execution — **stochastic** τ ~ M(x,s) + 温度0备注 |
| Eq trajectory | eq:trajectory | τ = (a₁,o₁,...,aT,oT), T ≤ T_max=40 |
| Definition 3 | def:eval | Evaluation Function E → {0,1}^8 |
| Eq eval_fn | eq:eval_fn | E: (A×O)^≤T × S × X → {0,1}^8 |
| Table | — | 8 维定义表 (inline) |
| Remark | — | **Soft dependency** — y3 > y2 可能且有意义 |

#### §3.1 Med-SkillBench
| 构件 | Label | 内容 |
|------|-------|------|
| Eq 1-3 | eq:pass, eq:gold, eq:silver | Pass/Gold/**Silver** + **disjoint partition** G⊔S⊔F |
| Eq 4 | eq:rbu | Aggregate RBU gap |
| **Prop 1** | prop:decomp, eq:decomp | **Two-Stage Decomposition** Δ_RBU = δ_sem + δ_exec |
| Remark | — | **Sign Interpretation**: δ_exec = -0.023 = parametric bypass |
| Remark | — | Diagnostic Utility (same RBU, different decomposition) |
| Eq 6 | eq:conditioned_rbu | Interface-Conditioned RBU Δ^(ι) |
| **Hyp 1** | eq:interface_hyp | **Interface Grounding**: δ_sem^(none) >> δ_sem^(tool) |
| Eq 8 | eq:sgi | SGI = w_c·F_c + (1-w_c)·Q_a |
| Eq 9 | eq:dominance | Dominance condition |
| Eq 10 | eq:wc_derivation | **w_c > 3/4 = 0.75** (修正自 0.6) |
| **Prop 2** | prop:sgi_domain, eq:sgi_domain | SGI domain decomposition |

#### §3.2 Skill-Augmented QA
| 构件 | Label | 内容 |
|------|-------|------|
| μ定义 | — | **μ: Q → 2^S** (multi-skill) + P_μ pairs |
| Eq 12-13 | eq:acc_with, eq:acc_wo | Acc over pairs P_μ |
| Eq 14 | eq:uplift | Uplift = Acc_with - Acc_w/o |
| Eq 15 | eq:per_pair | Per-pair indicator u_{i,s} ∈ {-1,0,+1} |
| Eq 16 | eq:cond_uplift | Interface-conditioned uplift |
| **Prop 3** | prop:groundability, eq:uplift_prediction | Smaller RBU → larger uplift |
| r_GU | — | Aggregate correlation (5-point caveat) |
| **Eq 18b** | eq:per_skill_reg | Per-skill SGI→uplift regression β₁ (**primary test**) |
| McNemar | — | Simplified prose + ref to app:statistical_details |

#### Appendix 新增
| Section | Label | 内容 |
|---------|-------|------|
| SGI Sensitivity | app:sgi_sensitivity | w_c ∈ {0.6, 0.7, 0.75, 0.8} 表 (TODO) |
| Statistical Details | app:statistical_details | McNemar (eq:mcnemar_counts, eq:mcnemar_stat) + clustered bootstrap (eq:bootstrap_ci) |

### 2.4 v2 Errata 修正状态 (全部已应用)

| Errata | Severity | 修正 | 状态 |
|--------|----------|------|------|
| 1. SGI w_c 代数错 | CRITICAL | 0.6→0.75 | ✅ |
| 2. "Three-Stage" 计数 | CRITICAL | →"Two-Stage" | ✅ |
| 3. y1≥y2≥y3 硬序 | CRITICAL | →soft dependency | ✅ |
| 4. Gold⊂Silver⊂Pass | MODERATE | →disjoint G⊔S⊔F | ✅ |
| 5. μ single→multi | MODERATE | →μ: Q→2^S | ✅ |
| 6. r_GU low power | MODERATE | +per-skill regression β₁ | ✅ |
| 7. Agent deterministic | MODERATE | →stochastic+remark | ✅ |
| 8. δ_exec 负值 | MODERATE | +sign interpretation | ✅ |
| 9. Section ordering | MINOR | 已按 Part V outline | ✅ |
| 10. Multi-match dependence | MINOR | +clustered bootstrap | ✅ |
| 11. Equation count advisory | ADVISORY | McNemar moved to appendix (~17 main) | ✅ |

### 2.5 Label 修复记录

| 问题 | 修复 |
|------|------|
| experiments/appendix 引用 `eq:uplift_regression` 但 method 定义为 `eq:per_skill_reg` | 统一为 `eq:per_skill_reg` |

---

## 3. 当前断点（进行中事项）

### 刚完成的动作
- 3 并行 subagent 执行数学形式化重写 (Agent A 超时但实际完成; B, C 正常完成)
- Label mismatch 修复
- Git commit + push to remote

### 已知遗留问题（已输出到 terminal）

**CRITICAL（阻塞投稿）**:
1. Step 2 ablation 数据全空 (Table 4, 5, appendix全量表) — 等 API + R2→R3→Step2
2. SGI per-domain 数据未算 — 可离线从 Step 0 scores 计算
3. δ_sem^(ι)/δ_exec^(ι) per-tool_type 未算 — 同上可离线
4. 所有 Figure 仍 placeholder

**MODERATE（影响质量）**:
5. "three-stage evidence chain" (04_experiments:337) vs "two-stage decomposition" — 语义正确（指 Step0/1/2 三步流程），但可能混淆。建议改 "three-step"
6. Appendix G SGI sensitivity 表全空
7. `\theoremstyle` numbering: definition/proposition 共享 theorem counter

**LOW**:
8. Related work §2.3 仍有 EasyTool/Play2Prompt/Trace-Free+ 背景
9. Abstract 可能偏长 (~200 词+公式)

---

## 4. 避坑档案（受阻/废弃方案）

| 方案 | 废弃原因 |
|------|---------|
| w_c = 0.6 | v2 errata 证明代数错误：core-fail skill (SGI=0.80) > core-pass skill (SGI=0.60) |
| "Three-Stage Decomposition" | 只有两项 (δ_sem + δ_exec)，计数错误 |
| y1 ≥ y2 ≥ y3 硬序 | 实证数据 y3=0.743 > y2=0.720 矛盾 |
| Gold ⊂ Silver ⊂ Pass | Gold ∩ Silver = ∅，集合论错误 |
| μ: Q → S∪{∅} | 论文数据 "一题匹配多 skill"，single-skill mapping 矛盾 |
| r_GU 作 primary test | n=5 太小 (需 \|r\|>0.878)，改 per-skill regression β₁ |
| Agent deterministic | LLM 本质随机，改 stochastic + 温度0备注 |
| Judge v2→v3 暴露 | 用户明确拒绝："内部消化的，不能上论文" |
| GEPA-Med 作为贡献 | API/并发/环境阻塞，决定先只出 bench |

---

## 5. 待办事项树（全局视野）

### P0 CRITICAL（阻塞投稿）
- [ ] **恢复 API 端点** → R2 → R3 → Step 2 → 填 Table 4/5
  - 依赖：外部 (boyuerichdata/yunwu/glm_inf)
  - 链：API → R2(1-3h) → R3(<1min) → Step2(2-4h)
- [ ] **计算 SGI per domain** → 填 domain-stratified SGI 表
  - 依赖：无（Step 0 scores + domain json 已有）
  - 脚本：读 scores/ + skills_domain_v2_tierA_v2.json → 按 domain 聚合 F_c, Q_a, SGI
- [ ] **计算 δ_sem/δ_exec per tool_type** → 填 decomposition 表
  - 依赖：无（Step 0 y1/y2/y3 已有）
- [ ] **生成真实图表** → 替换 5 个 placeholder
  - Fig 1 (RBU gap teaser)
  - Fig 2 (pipeline)
  - Fig 3 (domain distribution)
  - Fig 4 (tool_type bar)
  - Fig 5 (benchmark match rate)

### P1 HIGH（论文质量）
- [ ] 修复 "three-stage evidence chain" → "three-step" (04_experiments:337)
- [ ] 计算 SGI sensitivity (w_c ∈ {0.6,0.7,0.75,0.8}) → 填 Appendix G
- [ ] 检查 theorem/definition counter 是否需要独立
- [ ] 用户手动编译验证 LaTeX

### P2 MEDIUM
- [ ] Per-skill SGI→uplift regression β₁ → 需 Step 2 数据
- [ ] Net benefit ratio ρ → 需 Step 2 数据
- [ ] Multi-model 扩展 (DeepSeek, GPT-4.1-mini)
- [ ] 更新 CLAUDE.md 反映最新方程编号

### P3 LOW
- [ ] Abstract 长度优化
- [ ] Related work §2.3 是否精简
- [ ] Domain quality profile radar charts

---

## 6. 唤醒后的下一步行动（破局点）

### 第 1 步：离线计算可填数据（不需要 API）
```bash
cd /mnt/d/coding_files/Linux_D_Projects/20260310_medskillbench_codes/GMAI-Claw/skills_benchmarks_matching_ver2/

# 1a. δ_sem/δ_exec per tool_type
# 读 output/step0/.../scores/ 中每个 skill 的 y1,y2,y3
# 读 data_assets/skills_catalog_v2_tierA_v2.json 获取 tool_type
# 按 tool_type 分组计算 mean(y1)-mean(y2)=δ_sem, mean(y2)-mean(y3)=δ_exec

# 1b. SGI per domain (w_c=0.75)
# 读 scores/ 获取 y1-y8
# 读 skills_domain_v2_tierA_v2.json 获取 domain
# F_c = mean(y1,y2,y3)/3, Q_a = mean(y4..y8)/5
# SGI = 0.75*F_c + 0.25*Q_a
# 按 domain 聚合

# 1c. SGI sensitivity
# 同上但 w_c ∈ {0.6, 0.7, 0.75, 0.8}
```

### 第 2 步：填入 LaTeX 表格
```
ver2/sections/04_experiments.tex:
  - Table (decomposition): 填 δ_sem^(ι), δ_exec^(ι) per tool_type
  - Table (domain SGI): 填 Core, Aux, SGI per domain
ver2/sections/99_appendix.tex:
  - Table (SGI sensitivity): 填 4 个 w_c 的 mean SGI + separation + rank τ
```

### 关键文件路径速查
```
论文主线:          ver2/sections/*.tex
数学规划(权威):    ver2/math_formulations/Mathematical Formalization — Complete Revision Plan.md
Step 0 scores:     ...codes/.../output/step0/step0__full_v2_l20_c16_t120__gpt-5_1/scores/
Skill catalog:     ...codes/.../data_assets/skills_catalog_v2_tierA_v2.json
Domain mapping:    ...codes/.../data_assets/skills_domain_v2_tierA_v2.json
Git remote:        github.com:junzhin/medskillbench-emnlp2025-latex.git (main)
Latest commit:     bb4327f
```
