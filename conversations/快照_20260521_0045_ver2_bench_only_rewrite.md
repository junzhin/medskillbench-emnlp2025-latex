# 上下文记忆快照

> 时间：2026-05-21 00:45 | 会话主题：ver2 bench-only 论文重写 | 模型：Opus 4.6 (1M)

---

## 1. 核心目标与全局约束

**核心任务**：将 EMNLP 2025 论文从双组件版（Med-SkillBench + GEPA-Med）转为纯 bench 论文（Med-SkillBench only），提升学术性和医学领域特异性，补充 with/without skill 对比实验框架（数据待填）。

**全局约束**：
- 目标会议：EMNLP 2025 (ACL ARR submission)，8 页正文 long paper，匿名审稿
- LaTeX 模板：`\usepackage[review]{acl}`，`\author{Anonymous ACL submission}`
- **绝对不暴露开发历史**：v1→v2 judge 迭代、pilot 数据、内部资源困境、API 阻塞等一律不出现在论文中
- **不在 WSL 中自动编译 LaTeX**（会 crash VSCode server）
- 工作目录：`/mnt/d/coding_files/Linux_D_Projects/20260310_medskillbench/05_WRITING_LATEX/EMNLP_2025/`
- 只编辑 `ver2/` 内文件；根目录和 `ver1/` 冻结不动
- 代码库（只读参考）：`/mnt/d/coding_files/Linux_D_Projects/20260310_medskillbench_codes/GMAI-Claw/skills_benchmarks_matching_ver2/`

---

## 2. 已落地成果（历史沉淀）

### 2.1 版本结构

| 目录 | 版本 | 说明 |
|------|------|------|
| `EMNLP_2025/`（根） | V002 原版 | 含 GEPA，不再编辑 |
| `EMNLP_2025/ver1/` | V002 冻结备份 | 2026-05-20 快照 |
| **`EMNLP_2025/ver2/`** | **V005 bench-only** | **当前主线** |
| `EMNLP_2025/CLAUDE.md` | 局部指南 | ver2 写作规范 |
| `EMNLP_2025/V005_20260520_bench_only_situation_analysis.md` | 情况分析 | pipeline 全貌 + 缺口 |

### 2.2 GEPA 清理（已完成，41 处引用全删）

- 标题：`MedSkill-E²: Evaluating and Evolving...` → `Med-SkillBench: Evaluating Medical Agent Skills at Scale`
- 宏：删 `\gepamed`、`\medskille`；只保留 `\medskillbench`、`\rbu`、`\skillmd`
- 所有 section 中 GEPA 引用清零（grep 验证通过）
- Contribution: 3→2→**3**（重新加了 C3 ablation）
- RQ: 4→2→**3**（重新加了 RQ3 ablation）

### 2.3 当前论文结构（ver2 最终状态）

```
ver2/main.tex  (标题: Med-SkillBench: Evaluating Medical Agent Skills at Scale)
├── 00_abstract.tex      — 9行: bench + RBU gap + SGI + ablation(TODO) + 医学domain
├── 01_introduction.tex  — 59行: medical motivation + C1(bench) + C2(RBU) + C3(ablation) + 3 RQs
├── 02_related_work.tex  — 70行: 3 subsections (skills/medical bench/tool optimization) + comparison table
├── 03_method.tex        — 171行:
│   ├── §3.1 Med-SkillBench (benchmark design)
│   │   ├── Skill Collection & Quality Filtering (1462→917 Tier-A)
│   │   ├── Isolated Evaluation Protocol
│   │   ├── Task Construction (role separation)
│   │   ├── 8-Dim Binary Scoring + Pass/Gold定义
│   │   ├── RBU Gap (Eq.3)
│   │   ├── **SGI (Eq.4)** — Specification Groundability Index, core/aux加权分解 [新增]
│   │   ├── **Medical Domain Taxonomy** — reasoning-execution duality, 8域分布 [新增]
│   │   ├── Fig 2 (pipeline placeholder) + **Fig 3 (domain dist placeholder)** [新增]
│   │   └── Algorithm 1 (evaluation pipeline pseudocode)
│   └── §3.2 Skill-Augmented QA Evaluation Protocol [新增]
│       ├── Task Selection (3-round matching pipeline)
│       ├── Dual-Mode Execution (with/without skill)
│       ├── Answer Evaluation (exact match + LLM judge)
│       ├── Uplift Metric (Eq.5)
│       └── Statistical Testing (McNemar + bootstrap CI)
├── 04_experiments.tex   — 311行:
│   ├── §4.1 Setup (models, environment, protocol, conditions)
│   ├── §4.2 Main Results (Table 1: 8-dim总览, n=1462) ✅有数据
│   ├── §4.3 RBU Gap Analysis (Table 2: tool_type分层) ✅有数据
│   ├── **§4.4 Domain-Stratified Groundability** (Table: SGI per domain) [新增, TODO]
│   ├── §4.5 Benchmark Integration (Table 3: 12 benchmark匹配) ✅有数据
│   ├── **§4.6 Skill-Augmented QA Results** (Table 4: model×benchmark主表) [新增, TODO]
│   ├── **§4.7 Uplift by Skill Quality** (Table 5: tool_type uplift) [新增, TODO]
│   └── §4.8 Discussion (3段: closing-the-loop + complementarity + implications)
├── 05_conclusion.tex    — 15行: bench + RBU + ablation(TODO) + paradigm
├── 06_limitations.tex   — 23行: 4项 (synthetic tasks, binary scoring, LLM judge bias, single-model)
├── 07_ethical.tex       — 20行: 4项
└── 99_appendix.tex      — 235行: 6节 (eval dims, read-first prompt, domain dist, benchmark详表, eval stability, **ablation details**)
```

### 2.4 表格/图清单

| 编号 | 内容 | 状态 |
|------|------|------|
| Table 1 | 8维总览 (n=1,462) | ✅ 有真实数据 |
| Table 2 | tool_type分层 (5行) | ✅ 有真实数据 |
| Table 3 | 12 benchmark匹配率 | ✅ 有真实数据 |
| Table (SGI) | Domain-stratified SGI | ❌ TODO (可从Step0算) |
| **Table 4** | **Skill-Augmented QA主表** | **❌ TODO (等Step2)** |
| **Table 5** | **Uplift by tool_type** | **❌ TODO (等Step2)** |
| Fig 1 | Teaser (RBU gap) | placeholder |
| Fig 2 | Pipeline (评测流程) | placeholder |
| **Fig 3** | **Domain distribution** | **placeholder [新增]** |
| Fig 4 | tool_type bar chart | placeholder |
| Fig 5 | Benchmark match rate | placeholder |

### 2.5 形式化指标（方法贡献）

1. **Pass/Gold** (Eq.1-2): 基于3核心/8全维的二元状态判定
2. **$\Delta_{\textsc{RBU}}$** (Eq.3): $\bar{y}_1 - \bar{y}_3$，invocation vs completion gap
3. **SGI** (Eq.4): $w_c \cdot \frac{1}{3}\sum_{i=1}^{3}y_i + (1-w_c)\cdot\frac{1}{5}\sum_{i=4}^{8}y_i$, $w_c=0.6$
4. **$\Delta_{\text{uplift}}$** (Eq.5): $\text{Acc}_{\text{with}} - \text{Acc}_{\text{without}}$

### 2.6 关键数据资产（只读）

| 资产 | 路径 | 状态 |
|------|------|------|
| Skills Catalog (917 Tier-A) | `...codes/.../data_assets/skills_catalog_v2_tierA_v2.json` | frozen, md5 verified |
| Domain Classification | `...codes/.../data_assets/skills_domain_v2_tierA_v2.json` | frozen |
| Static Quality | `...codes/.../data_assets/skill_static_quality.json` | frozen |
| Step 0 Output (1462 skills) | `...codes/.../output/step0/step0__full_v2_l20_c16_t120__gpt-5_1/` | ✅ complete |
| Step 1 R1 Output | `...codes/.../output/step1/r1__full_20260505_191956Z__glm-5_1/` | ✅ complete |

### 2.7 已删除/清理的内容

- Judge校准 v2→v3 段落（experiments + appendix 整节删除）
- 所有 `\gepamed` 引用（41处）
- `\medskille` 宏（replaced by `\medskillbench`）
- Batch stability 中暴露内部 judge 迭代的语句
- Related work §2.4 (Prompt Evolution) 整节
- Method §3.2 (GEPA-Med) 整节 + Algorithm + Figure
- Experiments GEPA效果/消融/跨模型迁移节
- Conclusion/Limitations/Ethical 中 GEPA 段落
- Appendix GEPA Detailed Effectiveness 节

---

## 3. 当前断点（进行中事项）

### 刚完成的动作
用户提出论文 motivation 太弱、太像技术报告、医学特性不够、缺理论指标。本轮修改：
1. 删 judge 校准 v2→v3
2. 新增 SGI (Specification Groundability Index) 形式化指标
3. 新增 Medical Domain Taxonomy 段（reasoning-execution duality, 8域, safety boundaries）
4. 新增 Fig 3 domain distribution placeholder
5. 新增 §4.4 Domain-Stratified Groundability Analysis + Table (SGI per domain)
6. 强化 intro/abstract 医学 motivation

### 用户尚未编译验证
修改后未编译。用户需手动编译确认排版无误。

### 潜在下一步问题
- SGI 的 Table (domain-stratified) 数据可以从 Step 0 scores 计算得出，尚未填入
- 用户可能对 SGI 公式的 $w_c=0.6$ 权重有意见（目前是初稿值）
- 用户可能还要进一步加强医学叙事（如 specific medical failure case examples）
- 论文是否需要更多 medical-specific related work

---

## 4. 避坑档案（受阻/废弃方案）

### 4.1 绝不暴露的内部细节
| 禁区 | 原因 |
|------|------|
| Judge v2→v3 校准（pass rate 18.5%→64.4%） | 暴露评测不成熟，显得低端 |
| V1 pilot 数据（42 skills × 60 tasks） | 已被 V2 取代，规模太小 |
| API 端点阻塞/成本问题 | 资源限制不能暴露 |
| GEPA 实验未跑完的原因 | 内部资源分配问题 |
| Batch 1 评分偏低（judge prompt微调） | 内部迭代细节 |
| "先只投 bench" 的决策过程 | 内部策略 |

### 4.2 废弃方案
| 方案 | 废弃原因 |
|------|---------|
| 双组件论文 (bench + GEPA-Med) | GEPA 实验受 API/并发/环境阻塞 |
| 只 2 contributions 的 bench 论文 | 用户反馈太弱，像技术报告 |
| 暴露 judge 演化历史作为贡献 | 用户明确拒绝："都是内部消化的" |
| `\medskille` (MedSkill-E²) 作为品牌 | E² 代表 Evaluate+Evolve，去 GEPA 后不适用 |

---

## 5. 待办事项树（全局视野）

### P0 (阻塞论文投稿)
- [ ] **恢复 API 端点** → 跑 R2 → R3 → Step 2 → 填 Table 4/5 数据
  - 依赖：外部 (boyuerichdata/yunwu/glm_inf)
  - 预计：恢复后 6-8h compute
- [ ] **计算 SGI per domain** → 填 Table (domain-stratified SGI)
  - 依赖：Step 0 scores (已有) + domain mapping (已有)
  - 预计：写个脚本 30min
- [ ] **生成真实图表** → 替换所有 placeholder figures
  - Fig 1 (RBU gap teaser): 手绘或 tikz
  - Fig 2 (pipeline): 手绘或 tikz
  - Fig 3 (domain distribution): 从 domain json 生成
  - Fig 4 (tool_type bar): 从 Step 0 数据生成
  - Fig 5 (benchmark match): 从 R1 数据生成

### P1 (论文质量)
- [ ] 进一步强化医学叙事 — 可能需要 specific failure examples
- [ ] SGI 权重 $w_c$ 的敏感性分析或理论依据
- [ ] Related work 可能需补充 medical skill/agent 相关近期工作
- [ ] Limitations 补一条 subset sample size for ablation
- [ ] 用户可能还要人为调研完善理论部分

### P2 (锦上添花)
- [ ] 多模型扩展 (DeepSeek V3.2, GPT-4.1-mini)
- [ ] Global mode Step 2 (255K pairs, 远期)
- [ ] 更新 `CLAUDE.md` 和 `V005_situation_analysis` 反映最新状态
- [ ] Storyline + log 落盘（会话结束时）

---

## 6. 唤醒后的下一步行动（破局点）

### 第 1 步：确认 API 状态 + 算 SGI
```bash
# 测试 API
cd /mnt/d/coding_files/Linux_D_Projects/20260310_medskillbench_codes/GMAI-Claw/skills_benchmarks_matching_ver2/
python scripts/smoke_real_api.py

# 计算 SGI per domain (可离线做，不需要 API)
# 读 Step 0 scores + domain mapping → 按 domain 聚合 → 输出 table
```

### 第 2 步：填 SGI Table + 替换 placeholder 图
- 用 Step 0 output (`scores/`) + domain json 计算每个 domain 的 core/aux/SGI
- 填入 `04_experiments.tex` Table (domain-stratified SGI)
- 用 matplotlib/tikz 生成 Fig 3 (domain distribution) 和 Fig 4 (tool_type bar)

### 关键文件路径速查
```
论文主线:     ver2/sections/*.tex
数据资产:     ...codes/.../data_assets/
Step 0 输出:  ...codes/.../output/step0/step0__full_v2_l20_c16_t120__gpt-5_1/
Step 1 输出:  ...codes/.../output/step1/r1__full_20260505_191956Z__glm-5_1/
情况分析:     EMNLP_2025/V005_20260520_bench_only_situation_analysis.md
局部指南:     EMNLP_2025/CLAUDE.md
```
