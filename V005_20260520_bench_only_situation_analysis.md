# V005 情况分析：Bench-Only 转向（2026-05-20）

> 版本：V005 | 日期：2026-05-20 | 前序版本：V002 (med_skillbench_gepa) / V004
> 转向原因：GEPA 实验受 API 成本/并发/环境阻塞，决定先只出 bench 论文

---

## 1. 版本变化摘要

| 项目 | V002 (原双组件) | V005 (bench-only) |
|------|----------------|-------------------|
| 论文标题 | MedSkill-E²: Evaluating and Evolving Medical Agent Skills at Scale | Med-SkillBench: Evaluating Medical Agent Skills at Scale |
| 核心组件 | Med-SkillBench (bench) + GEPA-Med (evolution) | Med-SkillBench (bench only) |
| Contributions | 3 个 (bench + RBU gap + GEPA-Med) | 2 个 (bench + RBU gap) |
| Research Questions | 4 个 (RQ1-RQ4) | 2 个 (RQ1-RQ2) |
| Method | §3.1 Bench + §3.2 GEPA-Med | §3.1 Bench only |
| Experiments | skill 评测 + matching + GEPA 效果 + 消融 + 跨模型 | skill 评测 + matching + **with/without skill 对比 (待跑)** |
| 目标会议 | EMNLP 2025 (8 页 long paper) | EMNLP 2025 (同) |

---

## 2. Pipeline 完整状态

### 2.1 已完成（可直接用于论文）

#### Step 0: Skill 评测 ✅

- **规模**: 1,462 skills (917 Tier-A), GPT-5.1 executor, v3 judge
- **核心指标**: pass_rate=64.4%, gold=20.5%, RBU gap=0.220
- **关键发现**: guide-style skills (tool_type=None, n=1044) RBU gap=0.269, tool-based skills gap=0.069-0.111
- **产出路径**: `20260310_medskillbench_codes/.../output/step0/step0__full_v2_l20_c16_t120__gpt-5_1/`
- **论文对应**: §4.2 Large-Scale Skill Evaluation Results + §4.3 RBU Gap Analysis

#### Step 1 R1: Skill-Benchmark 匹配 ✅

- **规模**: 917 Tier-A skills × 14,019 questions (12 benchmarks), glm-5.1
- **核心指标**: 2,596 匹配 (18.5%), 178 unique skills, ~3,898 skill-question 对
- **匹配分布**: Lab-Bench 46.2% (最高) → MedBullets-4op 2.7% (最低)
- **集中度问题**: top 3 skills 占 91.8% matches (agent-browser 78%)
- **产出路径**: `20260310_medskillbench_codes/.../output/step1/r1__full_20260505_191956Z__glm-5_1/`
- **论文对应**: §4.4 Benchmark Integration Results

### 2.2 阻塞中（论文核心缺失）

#### Step 1 R2: 验证 ⏳ (等 API)

- **做什么**: 对 R1 产出的 ~3,898 (Q, skill) 对做 15 维验证 (5 静态 + 6 适配 + 9 领域)
- **代码状态**: ✅ 完整 (`medskill link r2`)
- **阻塞**: API 端点 (glm_inf/boyuerichdata) 连接重置
- **预计耗时**: 1-3h (API 恢复后)

#### Step 1 R3: 评分合并 ⏳ (等 R2)

- **做什么**: 纯计算合并 R1+R2, 产出 ScoredRecord (final_pass 策略)
- **代码状态**: ✅ 完整 (`medskill link r3`)
- **预计耗时**: <1 分钟
- **0 API calls**

#### Step 2: With/Without Skill 对比实验 ⏳ (等 R3) ← **论文核心缺口**

- **做什么**: 对筛选后的 benchmark 题目，分别用"有 skill"和"没有 skill"两种模式让模型回答，对比正确率差异
- **Subset 模式**: 120 real (R3 top) + 30 synthetic (Step 0) = 150 tasks
- **模型**: gpt-5.1, claude-sonnet-4-5 (可配更多)
- **评判**: exact_match + LLM judge → correct: bool
- **统计**: McNemar 配对检验, bootstrap 95% CI, 按 benchmark/tool_type/domain 分层
- **代码状态**: ✅ scaffold 完整 (`medskill ablate subset`)
- **预计耗时**: 2-4h (API 恢复后)

### 2.3 阻塞链

```
API 端点挂了 ──→ R2 跑不了 ──→ R3 跑不了 ──→ Step 2 跑不了
                                                    ↑
                                              论文核心主表缺口
```

**恢复后总计: ~6-8h compute 即可产出 Step 2 结果**

---

## 3. 论文结构与数据对应 (ver2)

### 3.1 已有数据支撑的 section

| Section | 内容 | 数据来源 | 状态 |
|---------|------|---------|------|
| Abstract | bench 介绍 + RBU gap | Step 0 | ✅ |
| §1 Introduction | 1 gap + 2 contributions + 2 RQs | — | ✅ |
| §2 Related Work | 3 subsections + comparison table | — | ✅ |
| §3 Method | Med-SkillBench 评测协议 + Algorithm 1 (伪代码) | — | ✅ |
| §4.1 Setup | 模型、环境、评测协议 | — | ✅ |
| §4.2 Main Results | 8 维总览表 (Table 1) | Step 0 | ✅ |
| §4.3 RBU Gap Analysis | tool_type 分层表 (Table 2) | Step 0 | ✅ |
| §4.4 Benchmark Integration | 12 benchmark 匹配表 (Table 3) | Step 1 R1 | ✅ |
| §4.5 Discussion | 2 段 (skill-level eval 意义 + 与现有 bench 互补) | — | ✅ |
| Appendix A-E | 8 维定义、read-first prompt、domain 分布、匹配详表、judge 校准 | Step 0 + R1 | ✅ |

### 3.2 缺失数据的 section（需要 Step 2）

| Section | 需要什么 | 数据来源 | 状态 |
|---------|---------|---------|------|
| **§4.x (新增) With/Without Skill 对比** | 主表：model × benchmark × with/without → 正确率 + uplift | Step 2 | ❌ 未跑 |
| **§4.x (新增) Tool Type 分层 Uplift** | 按 tool_type 看 uplift 差异（guide-style vs tool-based） | Step 2 | ❌ 未跑 |
| **§4.x (新增) 统计检验** | McNemar p-value, bootstrap CI, effect size | Step 2 | ❌ 未跑 |

---

## 4. Bench-Only 论文的叙事逻辑

### 当前可讲的故事

1. **动机**: LLM agents 依赖 skills, 但现有 benchmark 只看 task-level 成功率, 不 isolate skill 执行质量
2. **贡献 1 (Med-SkillBench)**: 首个 skill-level execution benchmark, 1462 skills, 8 维评分, isolated sandbox
3. **贡献 2 (RBU Gap)**: 发现 96.2% invocation vs 74.3% completion 的 22pp gap; guide-style skills gap 最大 (0.269)
4. **Benchmark Integration**: 917 skills × 12 benchmarks matching, 验证 skills 与真实 QA 题关联性

### 缺失的关键论证

> **"有 skill 到底比没有 skill 好多少？"**

目前论文能说 skill 质量有差异 (pass rate varies by tool_type), 但没有直接证据说"给 agent 一个 skill 能提高答题正确率"。Step 2 就是要填这个证据链。

### 理想的 Step 2 主表（结构）

| Model | Condition | Lab-Bench | MedMCQA | MedQA | ... | Avg |
|-------|-----------|-----------|---------|-------|-----|-----|
| GPT-5.1 | Without Skill | ?% | ?% | ?% | | ?% |
| GPT-5.1 | With Skill | ?% | ?% | ?% | | ?% |
| GPT-5.1 | **Uplift** | **+?pp** | **+?pp** | **+?pp** | | **+?pp** |
| Claude | Without Skill | ?% | ?% | ?% | | ?% |
| Claude | With Skill | ?% | ?% | ?% | | ?% |
| Claude | **Uplift** | **+?pp** | **+?pp** | **+?pp** | | **+?pp** |

再加一张 tool_type 分层表：

| Tool Type | n | With Skill Acc | Without Skill Acc | Uplift | p-value |
|-----------|---|----------------|-------------------|--------|---------|
| CLI | ? | ?% | ?% | +?pp | ? |
| Python | ? | ?% | ?% | +?pp | ? |
| None (guide) | ? | ?% | ?% | +?pp | ? |

---

## 5. 下一步行动优先级

| 优先级 | 行动 | 阻塞 | 预计 |
|--------|------|------|------|
| **P0** | 恢复 API 端点 (boyuerichdata / yunwu / glm_inf) | 外部依赖 | 人工操作 |
| **P0** | `smoke_real_api.py` 验证连通性 | P0 API | 5min |
| **P1** | 跑 R2 (`bash scripts/run/step1_r2.sh`) | P0 API | 1-3h |
| **P1** | 跑 R3 (`bash scripts/run/step1_r3.sh`) | P1 R2 | <1min |
| **P1** | 跑 Step 2 subset (`bash scripts/run/step2_subset.sh`) | P1 R3 | 2-4h |
| **P2** | Step 2 结果写入论文 ver2 (新增 §4.x + 主表) | P1 Step2 | 2-3h 写作 |
| **P2** | 统计检验 + 可视化 | P1 Step2 | 1-2h |
| **P3** | 多模型扩展 (DeepSeek V3.2, GPT-4.1-mini) | P1 Step2 | 额外 compute |

---

## 6. 风险与备选方案

| 风险 | 影响 | 备选 |
|------|------|------|
| API 长期不可用 | Step 2 无法跑 | 用本地部署 GLM 跑 R2 (并发低但可用) |
| Step 2 uplift 不显著 | 论文故事弱 | 强调 diagnostic 价值 (RBU gap 定位问题而非解决问题) |
| Skill 集中度过高 (top 3 占 91.8%) | 泛化性质疑 | Step 2 按 skill 分层报告，排除 agent-browser 敏感性分析 |
| 150 tasks 样本量小 | 统计效力不足 | 报告 confidence interval，后续补 global mode |
| 时间紧迫 | bench 论文完成度不够 | 最小可投稿版本 = 当前 ver2 + Step 2 subset |

---

## 7. 文件版本对照

| 路径 | 版本 | 说明 |
|------|------|------|
| `05_WRITING_LATEX/EMNLP_2025/` (根目录) | V002 原版 | 含 GEPA, 未动 |
| `05_WRITING_LATEX/EMNLP_2025/ver1/` | V002 冻结备份 | 2026-05-20 备份，与根目录内容一致 |
| `05_WRITING_LATEX/EMNLP_2025/ver2/` | **V005 bench-only** | 全面去 GEPA, 改标题, 2 contributions + 2 RQs |
| 本文件 | V005 情况分析 | 记录转向决策、pipeline 状态、缺口分析 |
