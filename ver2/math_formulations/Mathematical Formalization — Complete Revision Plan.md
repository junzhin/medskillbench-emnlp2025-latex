# Med-SkillBench: Mathematical Formalization — Complete Revision Plan (v2)

> **Purpose**: This document provides (1) a diagnostic of every mathematical weakness in the current draft, (2) a fully rigorous rewrite of the formal framework, and (3) a concrete plan for integrating the formalism with experiments and the paper's core thesis. All LaTeX is publication-ready.
>
> **v2 Changelog**: Advisor-level audit identified and corrected 6 substantive errors and 5 structural issues from v1. All changes are marked with **[v2 FIX]** tags for traceability. A dedicated errata section (Part VII) documents each fix with rationale.

---

## Part I — Diagnostic: What Is Wrong with the Current Math

### 1.1 The Equations Are Bookkeeping, Not Theory

The current draft contains five numbered equations. Every one of them is a trivial arithmetic identity:

| Eq. | Content | Problem |
|-----|---------|---------|
| (1) | PASS = indicator of $y_1 \wedge y_2 \wedge y_3$ | Definition masquerading as a result. No reader learns anything. |
| (2) | GOLD = indicator of $\sum y_i = 8$ | Same — just "all eight are 1." |
| (3) | $\Delta_{\text{RBU}} = \bar{y}_1 - \bar{y}_3$ | A subtraction. No decomposition, no causal structure, no connection to *why* the gap exists. |
| (4) | SGI = weighted average, $w_c = 0.6$ | Ad-hoc weight with zero justification. Why 0.6? Why linear? Why this partition? |
| (5) | $\Delta_{\text{uplift}} = \text{Acc}_{\text{with}} - \text{Acc}_{\text{without}}$ | Another subtraction. No conditioning on skill quality, no formal link to RBU. |

**Verdict**: The paper claims a "formal" benchmark but the math is at the level of a lab notebook. For ACL, we need the formalism to *do work* — to define objects precisely, to decompose phenomena into interpretable parts, and to generate testable predictions that connect Step 0 → Step 1 → Step 2.

### 1.2 Missing Formal Structures

1. **No object definitions.** What *is* a skill mathematically? What is a trajectory? What is the action space? These are never defined.
2. **No execution model.** There is no formal model of how an agent consumes a skill specification and produces a trajectory. Without this, "read-but-not-use" is an informal observation, not a formally decomposable phenomenon.
3. **No decomposition of the gap.** The RBU gap is a single scalar. But logically it decomposes into at least two stages: (a) semantic grounding failure and (b) execution completion failure. The formalism should expose this.
4. **No connection theorem.** The paper claims Steps 0–2 "close the loop." But there is no formal statement connecting RBU gap magnitude to downstream uplift. This is the paper's central claim and it has no mathematical backing.
5. **No statistical formalization.** McNemar's test is mentioned in prose but never formalized. Confidence intervals are described but never written as equations.

### 1.3 What the Formalism Must Achieve

The math in this paper must:

- **Define** the skill execution problem as a structured prediction task with clear input/output spaces
- **Decompose** the RBU gap into mechanistically interpretable components
- **Justify** SGI weighting from first principles (or replace it)
- **Connect** skill-level diagnostics (Step 0) to downstream QA performance (Step 2) via a formal proposition
- **Formalize** the statistical testing framework
- **Stratify** cleanly by tool_type within the formal framework

---

## Part II — Complete Rewrite: Formal Mathematical Framework

Below is the **publication-ready** mathematical framework. Section numbers reference where each block would appear in the paper.

---

### §3.0 — Problem Formulation

> **Narrative purpose**: Define the skill-grounded execution problem before describing the benchmark design.

#### Definition 1 (Skill Specification).

A *skill specification* is a tuple $s = (\texttt{desc}, \texttt{params}, \texttt{iface}, \texttt{dom})$ where:

- $\texttt{desc} \in \Sigma^*$ is a natural-language description document (the SKILL.md content),
- $\texttt{params} = \{(p_j, T_j, C_j)\}_{j=1}^{m}$ is a parameter schema, with each parameter $p_j$ having type $T_j$ and constraint set $C_j$,
- $\texttt{iface} \in \mathcal{I} = \{\texttt{cli}, \texttt{python}, \texttt{R}, \texttt{mixed}, \texttt{none}\}$ is the tool interface type,
- $\texttt{dom} \in \mathcal{D}$ is the medical domain label from a fixed taxonomy $\mathcal{D}$.

We write $\mathcal{S}$ for the full skill catalog and $\mathcal{S}_A \subseteq \mathcal{S}$ for the Tier-A subset surviving quality filtering.

#### Definition 2 (Agent Execution).

**[v2 FIX: Agent modeled as stochastic, not deterministic.]**

An *agent* $\mathcal{M}$ is a stochastic function that, given a task description $x \in \mathcal{X}$ and a skill specification $s \in \mathcal{S}$, samples a bounded execution trajectory:

$$
\tau \sim \mathcal{M}(x, s), \quad \tau = (a_1, o_1, a_2, o_2, \ldots, a_T, o_T), \quad T \leq T_{\max},
$$

where $a_t \in \mathcal{A}$ is an agent action (tool call, code execution, reasoning step, file read) and $o_t \in \mathcal{O}$ is the environment observation returned after $a_t$. The trajectory is bounded by $T_{\max} = 40$ iterations and a wall-clock timeout $\Delta t_{\max} = 300\text{s}$.

**Remark (Deterministic Approximation).** In practice, all experiments use temperature 0 (greedy decoding), making $\mathcal{M}$ effectively deterministic for a given $(x, s)$. We retain the stochastic formulation for generality, but all reported metrics are point estimates from single runs. Future multi-run evaluations should report variance.

#### Definition 3 (Evaluation Function).

The *evaluation function* $E$ maps a (trajectory, skill, task) triple to an eight-dimensional binary quality vector:

$$
E: (\mathcal{A} \times \mathcal{O})^{\leq T_{\max}} \times \mathcal{S} \times \mathcal{X} \;\longrightarrow\; \{0,1\}^8, \quad E(\tau, s, x) = \mathbf{y} = (y_1, \ldots, y_8).
$$

The eight dimensions are partitioned into **core fidelity** $\mathbf{y}^{\text{core}} = (y_1, y_2, y_3)$ and **auxiliary quality** $\mathbf{y}^{\text{aux}} = (y_4, \ldots, y_8)$, where:

| Index | Dimension | Interpretation |
|-------|-----------|----------------|
| $y_1$ | `skill_invoked` | Agent reads/references the specification |
| $y_2$ | `skill_used_correctly` | Actions conform to specification semantics |
| $y_3$ | `task_completion` | Final output satisfies the task objective |
| $y_4$ | `tool_execution_success` | $\geq 1$ tool call returns verified output |
| $y_5$ | `output_quality` | Domain-appropriate quality of final output |
| $y_6$ | `reasoning_quality` | Coherent, non-circular reasoning chain |
| $y_7$ | `efficiency` | Reasonable iteration count, no retry loops |
| $y_8$ | `trajectory_quality` | Purposeful progression and error recovery |

**[v2 FIX: Dependency chain revised — soft tendency, not hard logical entailment.]**

**Remark (Soft Dependency Structure).** The core dimensions encode a *soft* dependency tendency: correct usage ($y_2 = 1$) typically presupposes invocation ($y_1 = 1$), and task completion ($y_3 = 1$) typically presupposes correct usage ($y_2 = 1$). However, this is not a strict logical entailment. Empirically, $\bar{y}_3 > \bar{y}_2$ is possible (and observed: $\bar{y}_3 = 0.743 > \bar{y}_2 = 0.720$ in Table 2), because an agent can complete a task via parametric knowledge even when it fails to follow the skill specification correctly. This is precisely the phenomenon MED-SKILLBENCH is designed to detect: successful task completion that bypasses the skill specification should not be counted as successful skill grounding. The RBU gap captures this discrepancy.

---

### §3.1.1 — Outcome Tiers

> **Narrative purpose**: Replace the current ad-hoc PASS/GOLD definitions with a principled tier system.

#### Definition 4 (Outcome Tiers).

Given evaluation vector $\mathbf{y} = E(\tau, s, x)$, define:

$$
\text{Pass}(s) = \mathbf{1}\!\left[\,y_1 = 1 \;\wedge\; y_2 = 1 \;\wedge\; y_3 = 1\,\right], \tag{1}
$$

$$
\text{Gold}(s) = \mathbf{1}\!\left[\,\|\mathbf{y}\|_1 = 8\,\right], \tag{2}
$$

$$
\text{Silver}(s) = \text{Pass}(s) \cdot \left(1 - \text{Gold}(s)\right). \tag{3}
$$

**[v2 FIX: Set-theoretic statement corrected.]**

**Interpretation.** Gold requires full execution quality across all dimensions. Silver indicates core fidelity without full auxiliary quality — the skill was correctly invoked and the task completed, but execution artifacts (tool failures, inefficiency, trajectory issues) remain. Denoting the sets of skills achieving each tier as $\mathbb{G}$, $\mathbb{S}$, and $\mathbb{P}$ respectively, we have $\mathbb{G} \subset \mathbb{P}$, $\mathbb{S} \subset \mathbb{P}$, and $\mathbb{G} \cap \mathbb{S} = \varnothing$, with $\mathbb{P} = \mathbb{G} \;\dot{\cup}\; \mathbb{S} \;\dot{\cup}\; \mathbb{F}$ forming a disjoint partition where $\mathbb{F}$ denotes core-failure skills.

---

### §3.1.2 — The Read-But-Not-Use Gap: Formal Decomposition

> **Narrative purpose**: This is the paper's central construct. The current version is a single scalar. We decompose it into two mechanistically distinct stages.

#### Definition 5 (Aggregate RBU Gap).

For a skill set $\mathcal{S}$ evaluated by agent $\mathcal{M}$, the *aggregate RBU gap* is:

$$
\Delta_{\text{RBU}}(\mathcal{S}, \mathcal{M}) = \bar{y}_1 - \bar{y}_3, \quad \text{where}\;\; \bar{y}_k = \frac{1}{|\mathcal{S}|}\sum_{s \in \mathcal{S}} y_k^{(s)}. \tag{4}
$$

**[v2 FIX: Renamed from "Three-Stage" to "Two-Stage" — the decomposition has exactly two terms, not three. The v1 title was a counting error.]**

#### Proposition 1 (Two-Stage Decomposition of the RBU Gap).

The RBU gap decomposes additively into two mechanistically distinct failure stages:

$$
\Delta_{\text{RBU}} = \underbrace{(\bar{y}_1 - \bar{y}_2)}_{\delta_{\text{sem}}} + \underbrace{(\bar{y}_2 - \bar{y}_3)}_{\delta_{\text{exec}}}, \tag{5}
$$

where:

- $\delta_{\text{sem}} = \bar{y}_1 - \bar{y}_2$ is the **semantic grounding gap**: the agent reads the specification but fails to translate it into specification-conformant actions. This measures *comprehension-to-action* fidelity.
- $\delta_{\text{exec}} = \bar{y}_2 - \bar{y}_3$ is the **execution completion gap**: the agent acts in accordance with the specification but fails to produce a complete output. This measures *action-to-outcome* fidelity.

**Proof.** Follows from telescoping: $\bar{y}_1 - \bar{y}_3 = (\bar{y}_1 - \bar{y}_2) + (\bar{y}_2 - \bar{y}_3)$. $\square$

**[v2 FIX: Added explicit handling of negative $\delta_{\text{exec}}$ case, which occurs in the real data.]**

**Remark (Sign Interpretation).** Both $\delta_{\text{sem}}$ and $\delta_{\text{exec}}$ can take negative values. In the overall corpus, $\delta_{\text{exec}} = 0.720 - 0.743 = -0.023 < 0$, meaning task completion slightly *exceeds* correct skill usage. This arises when agents complete tasks by falling back to parametric knowledge rather than following the skill — they "pass the exam without reading the textbook." Far from being a pathology, this negative $\delta_{\text{exec}}$ is a key diagnostic signal: it indicates that the task is solvable without the skill, which is important context for interpreting uplift in Step 2. The semantic grounding gap $\delta_{\text{sem}} = 0.962 - 0.720 = 0.242$ accounts for more than 100% of the aggregate RBU gap, confirming that the dominant bottleneck is specification-to-action translation, not action-to-outcome execution.

**Remark (Diagnostic Utility).** Consider two skill categories with identical $\Delta_{\text{RBU}} = 0.22$:

- **Category A** (guide-style): $\delta_{\text{sem}} = 0.18, \; \delta_{\text{exec}} = 0.04$. The bottleneck is specification comprehension — the agent cannot translate prose instructions into actions.
- **Category B** (tool-based): $\delta_{\text{sem}} = 0.04, \; \delta_{\text{exec}} = 0.18$. The bottleneck is execution — the agent acts correctly but tool calls fail due to environment issues.

The aggregate $\Delta_{\text{RBU}}$ cannot distinguish these; the decomposition can, and it directly prescribes different interventions (improve descriptions vs. improve tool environments).

---

### §3.1.3 — Conditioned RBU Gap by Interface Type

> **Narrative purpose**: Formalize the tool_type stratification as conditional expectations rather than just "group and compare."

#### Definition 6 (Interface-Conditioned RBU Gap).

For interface type $\iota \in \mathcal{I}$, let $\mathcal{S}_\iota = \{s \in \mathcal{S} : s.\texttt{iface} = \iota\}$. The *interface-conditioned* RBU gap and its decomposition are:

$$
\Delta_{\text{RBU}}^{(\iota)} = \bar{y}_1^{(\iota)} - \bar{y}_3^{(\iota)} = \delta_{\text{sem}}^{(\iota)} + \delta_{\text{exec}}^{(\iota)}, \tag{6}
$$

where all means are taken over $\mathcal{S}_\iota$.

**Hypothesis 1 (Interface Grounding Hypothesis).** *Structured tool interfaces reduce the RBU gap primarily by shrinking the semantic grounding component $\delta_{\text{sem}}$:*

$$
\delta_{\text{sem}}^{(\texttt{none})} \;\gg\; \delta_{\text{sem}}^{(\iota)}, \quad \forall\, \iota \in \{\texttt{cli}, \texttt{python}, \texttt{R}, \texttt{mixed}\}. \tag{7}
$$

*Mechanistic rationale*: When a skill specifies a concrete invocation template (e.g., `blastp -query input.fasta -db nr`), the mapping from description to action is near-deterministic. Guide-style skills require the agent to synthesize multi-step action plans from unconstrained prose, introducing a combinatorially larger space of possible (mis)interpretations.

---

### §3.1.4 — Specification Groundability Index (SGI)

> **Narrative purpose**: Replace the current ad-hoc weighted average with a principled construction.

#### Definition 7 (Specification Groundability Index).

The SGI of a skill $s$ under agent $\mathcal{M}$ is a weighted aggregate of the evaluation vector $\mathbf{y} = E(\tau, s, x)$:

$$
\text{SGI}(s, \mathcal{M}) = w_c \cdot \underbrace{\frac{1}{3}\sum_{i=1}^{3} y_i}_{\text{core fidelity } F_c(s)} + (1 - w_c) \cdot \underbrace{\frac{1}{5}\sum_{i=4}^{8} y_i}_{\text{auxiliary quality } Q_a(s)}, \tag{8}
$$

where $w_c \in (0, 1)$ controls the emphasis on core fidelity relative to auxiliary quality.

**[v2 FIX: CRITICAL — The v1 derivation of $w_c = 0.6$ contained an algebra error. The correct bound is $w_c > 3/4$, not $w_c > 3/5$. Full corrected derivation below.]**

#### Justification of $w_c$ via the Dominance Principle.

We set $w_c$ based on the following **dominance principle**: a skill that fails on any core dimension (invocation, correctness, completion) is clinically unusable regardless of auxiliary quality, so core fidelity must dominate. Formally, we require that the *worst-case* core-failing skill always scores below the *worst-case* core-passing skill:

$$
\max_{\mathbf{y}: F_c < 1} \text{SGI}(\mathbf{y}) \;<\; \min_{\mathbf{y}: F_c = 1} \text{SGI}(\mathbf{y}). \tag{9}
$$

The left-hand maximum is achieved by $s^{\text{core-fail}}$: fails exactly one core dimension ($F_c = 2/3$) and passes all auxiliary ($Q_a = 1$). The right-hand minimum is achieved by $s^{\text{aux-fail}}$: passes all core ($F_c = 1$) and fails all auxiliary ($Q_a = 0$).

$$
\text{SGI}(s^{\text{core-fail}}) = w_c \cdot \frac{2}{3} + (1 - w_c) \cdot 1 = 1 - \frac{w_c}{3},
$$

$$
\text{SGI}(s^{\text{aux-fail}}) = w_c \cdot 1 + (1 - w_c) \cdot 0 = w_c.
$$

Dominance requires:

$$
1 - \frac{w_c}{3} < w_c \;\implies\; 1 < \frac{4w_c}{3} \;\implies\; w_c > \frac{3}{4} = 0.75. \tag{10}
$$

**Recommendation**: Set $w_c = 0.75$, the tightest value satisfying the strict dominance constraint, making it the *least opinionated* choice that guarantees any core-failing skill always ranks below any core-passing skill.

At $w_c = 0.75$: $\text{SGI}(s^{\text{core-fail}}) = 0.75$ and $\text{SGI}(s^{\text{aux-fail}}) = 0.75$. For strict inequality, use $w_c = 0.76$ (or any $w_c > 0.75$). Practically, we recommend $w_c = 0.75$ with the convention that ties are broken in favor of core-passing skills.

**Alternative**: If the original $w_c = 0.6$ is preferred for continuity with preliminary results, it must be justified on different grounds — e.g., "balancing interpretability" — and the dominance property must be explicitly disclaimed. Under $w_c = 0.6$, a core-failing skill with perfect auxiliary quality ($\text{SGI} = 0.8$) outscores a core-passing skill with zero auxiliary quality ($\text{SGI} = 0.6$), which is counterintuitive for a safety-critical domain.

We verify robustness to $w_c \in \{0.6, 0.7, 0.75, 0.8\}$ in sensitivity analysis (Appendix G).

#### Proposition 2 (SGI Decomposition by Domain).

For domain $d \in \mathcal{D}$, the mean SGI decomposes as:

$$
\overline{\text{SGI}}_d = w_c \cdot \bar{F}_{c,d} + (1-w_c) \cdot \bar{Q}_{a,d}. \tag{11}
$$

This decomposition exposes cross-domain quality patterns: computational domains (bioinformatics) are predicted to show $\bar{F}_{c,d} > \bar{Q}_{a,d}$ (high core fidelity, tool execution failures drag down auxiliary), while clinical reasoning domains show $\bar{Q}_{a,d} > \bar{F}_{c,d}$ (high reasoning quality, but weaker specification adherence).

---

### §3.2 — Skill-Augmented QA: Formal Framework

> **Narrative purpose**: Give the ablation study formal structure and connect it to the RBU gap.

**[v2 FIX: Matching function μ changed from single-skill to multi-skill, consistent with the paper's data showing one question can match multiple skills.]**

#### Definition 8 (Skill-Augmented Accuracy).

Let $\mathcal{Q} = \{(x_i, g_i)\}_{i=1}^{N}$ be a set of medical QA items with gold answers $g_i$. Let $\mu: \mathcal{Q} \to 2^{\mathcal{S}}$ be a skill-matching function assigning each question to a (possibly empty) set of relevant skills. For evaluation, we expand to *skill–question pairs*:

$$
\mathcal{P}_\mu = \{(x_i, s, g_i) : s \in \mu(x_i),\; \mu(x_i) \neq \varnothing\}.
$$

Define accuracy under the with-skill condition:

$$
\text{Acc}_{\text{with}}(\mathcal{P}_\mu, \mathcal{M}) = \frac{1}{|\mathcal{P}_\mu|}\sum_{(x_i, s, g_i) \in \mathcal{P}_\mu} \mathbf{1}\!\left[\,\mathcal{M}(x_i, s) = g_i\,\right], \tag{12}
$$

and accuracy under the without-skill baseline:

$$
\text{Acc}_{\text{w/o}}(\mathcal{P}_\mu, \mathcal{M}) = \frac{1}{|\mathcal{P}_\mu|}\sum_{(x_i, s, g_i) \in \mathcal{P}_\mu} \mathbf{1}\!\left[\,\mathcal{M}(x_i, \varnothing) = g_i\,\right]. \tag{13}
$$

**Remark (Multi-Match Handling).** When $|\mu(x_i)| > 1$, the same question $x_i$ appears in multiple pairs, each with a different skill. The without-skill prediction $\mathcal{M}(x_i, \varnothing)$ is computed once and reused across all pairs sharing the same $x_i$, avoiding redundant computation while correctly counting each skill's contribution.

#### Definition 9 (Skill Uplift).

$$
\Delta_{\text{uplift}}(\mathcal{P}_\mu, \mathcal{M}) = \text{Acc}_{\text{with}} - \text{Acc}_{\text{w/o}}. \tag{14}
$$

#### Definition 10 (Per-Pair Uplift Indicator).

For each pair $(x_i, s, g_i) \in \mathcal{P}_\mu$, define the signed indicator:

$$
u_{i,s} = \mathbf{1}[\mathcal{M}(x_i, s) = g_i] - \mathbf{1}[\mathcal{M}(x_i, \varnothing) = g_i] \;\in\; \{-1, 0, +1\}. \tag{15}
$$

Then $\Delta_{\text{uplift}} = \bar{u}$. The *net benefit ratio* is $\rho = n_{+1} / (n_{+1} + n_{-1})$, where $n_v = |\{(i,s) : u_{i,s} = v\}|$. A ratio $\rho > 0.5$ indicates that skills help more pairs than they hurt.

---

### §3.2.1 — Connecting RBU Gap to Downstream Uplift

> **Narrative purpose**: This is the "closing the loop" theorem — the paper's strongest theoretical contribution. Currently this claim is made informally; here we formalize it.

#### Proposition 3 (Groundability-Modulated Uplift).

Assume that skill execution quality mediates the downstream uplift. Specifically, partition matched pairs by the interface type of their matched skill: $\mathcal{P}_\mu^{(\iota)} = \{(x_i, s, g_i) \in \mathcal{P}_\mu : s.\texttt{iface} = \iota\}$. Then:

$$
\Delta_{\text{uplift}}^{(\iota)} = \text{Acc}_{\text{with}}^{(\iota)} - \text{Acc}_{\text{w/o}}^{(\iota)}, \quad \iota \in \mathcal{I}. \tag{16}
$$

**Prediction**: If the RBU gap reflects genuine specification grounding failure, then interface types with *smaller* RBU gaps should exhibit *larger* uplift:

$$
\Delta_{\text{RBU}}^{(\iota_1)} < \Delta_{\text{RBU}}^{(\iota_2)} \;\implies\; \Delta_{\text{uplift}}^{(\iota_1)} \geq \Delta_{\text{uplift}}^{(\iota_2)}. \tag{17}
$$

**[v2 FIX: Added caveat about low statistical power of 5-point correlation; proposed per-skill alternative.]**

**Testable form (Aggregate)**: Define the *groundability-uplift correlation*:

$$
r_{GU} = \text{Corr}\!\left(\{-\Delta_{\text{RBU}}^{(\iota)}\}_{\iota \in \mathcal{I}}, \;\{\Delta_{\text{uplift}}^{(\iota)}\}_{\iota \in \mathcal{I}}\right). \tag{18}
$$

A significantly positive $r_{GU}$ validates the "closing the loop" claim: skills that ground better (lower RBU) yield larger downstream benefit.

**Important caveat**: With $|\mathcal{I}| = 5$ interface types, this correlation is computed over only 5 data points, yielding negligible statistical power ($n = 5$ requires $|r| > 0.878$ for $p < 0.05$). We therefore recommend the following complementary test with higher power:

**Testable form (Per-Skill)**: For each skill $s$ with at least $k \geq 5$ matched QA pairs, compute the per-skill uplift $\Delta_{\text{uplift}}^{(s)}$ and regress it on the per-skill SGI:

$$
\Delta_{\text{uplift}}^{(s)} = \beta_0 + \beta_1 \cdot \text{SGI}(s) + \epsilon_s. \tag{18b}
$$

A significantly positive $\hat{\beta}_1$ provides a higher-powered test of the same hypothesis, using individual skills as the unit of analysis rather than aggregated interface types. Report both $r_{GU}$ and $\hat{\beta}_1$; the latter is the primary test.

---

### §3.2.2 — Statistical Testing Framework

> **Narrative purpose**: Formalize the statistical machinery instead of hand-waving at "McNemar's test."

#### McNemar's Test for Paired Binary Outcomes.

For each pair $(x_i, s, g_i) \in \mathcal{P}_\mu$, let $c_{i,s}^{+} = \mathbf{1}[\mathcal{M}(x_i, s) = g_i]$ and $c_{i,s}^{-} = \mathbf{1}[\mathcal{M}(x_i, \varnothing) = g_i]$. Define the discordant counts:

$$
b = \sum_{(i,s)} \mathbf{1}[c_{i,s}^{+} = 1 \wedge c_{i,s}^{-} = 0], \quad c = \sum_{(i,s)} \mathbf{1}[c_{i,s}^{+} = 0 \wedge c_{i,s}^{-} = 1]. \tag{19}
$$

Under $H_0: \Pr(c^+ = 1, c^- = 0) = \Pr(c^+ = 0, c^- = 1)$, the McNemar statistic is:

$$
\chi^2_{\text{McN}} = \frac{(b - c)^2}{b + c} \;\sim\; \chi^2(1). \tag{20}
$$

We reject $H_0$ at level $\alpha = 0.05$ if $\chi^2_{\text{McN}} > 3.841$.

**Remark (Multi-Match Dependence).** When $|\mu(x_i)| > 1$, multiple pairs share the same without-skill outcome $c_{i,s}^{-}$, introducing within-question dependence. We address this via clustered bootstrap (clustering by question $x_i$) rather than assuming independence across pairs. The McNemar test is applied at the pair level as a primary analysis, with the clustered bootstrap as a robustness check.

#### Bootstrap Confidence Intervals.

For $\Delta_{\text{uplift}}$, we construct bootstrap 95\% CIs by resampling the paired vector $\{(c_{i,s}^+, c_{i,s}^-)\}$ with replacement $B = 10{,}000$ times, clustering by question index $i$:

$$
\text{CI}_{95\%}(\Delta_{\text{uplift}}) = \left[\hat{\Delta}^{*(0.025)}, \;\hat{\Delta}^{*(0.975)}\right], \tag{21}
$$

where $\hat{\Delta}^{*(\alpha)}$ is the $\alpha$-quantile of the bootstrap distribution.

---

### §3.1.5 — Domain-Stratified Analysis Framework

> **Narrative purpose**: Formalize how domain stratification interacts with SGI and RBU.

**[v2 FIX: Domain quality profile range corrected — $\delta$ components can be negative.]**

#### Definition 11 (Domain Quality Profile).

For domain $d \in \mathcal{D}$, the *quality profile* is the vector:

$$
\mathbf{q}_d = \left(\bar{y}_1^{(d)}, \ldots, \bar{y}_8^{(d)}, \;\Delta_{\text{RBU}}^{(d)}, \;\delta_{\text{sem}}^{(d)}, \;\delta_{\text{exec}}^{(d)}, \;\overline{\text{SGI}}_d\right) \in [0,1]^8 \times [-1,1]^3 \times [0,1]. \tag{22}
$$

This profile enables direct cross-domain comparison. We predict two archetypal patterns:

- **Computational archetype** (bioinformatics, cheminformatics): $\bar{y}_4^{(d)}$ high, $\delta_{\text{sem}}^{(d)}$ low, $\delta_{\text{exec}}^{(d)}$ moderate → bottleneck is tool environment, not specification quality.
- **Reasoning archetype** (clinical medicine, public health): $\bar{y}_6^{(d)}$ high, $\delta_{\text{sem}}^{(d)}$ high, $\bar{y}_4^{(d)}$ low (often N/A) → bottleneck is specification underspecification.

---

## Part III — Integration Plan: How the Math Connects to Everything

### 3.1 Connecting Formalism to Experiments

| Formal Construct | Where It Appears | What It Proves |
|---|---|---|
| Def. 1–3 (skill, trajectory, eval) | §3.0 setup | Establishes the problem as structured prediction, not informal benchmarking |
| Eq. 4 (aggregate RBU) | §4.2, Table 2 | Quantifies the overall gap (0.220) |
| Eq. 5 (two-stage decomposition) | §4.3, new Table | **Key new result**: decomposes the 0.220 gap into $\delta_{\text{sem}} = 0.242$ and $\delta_{\text{exec}} = -0.023$ — dominant bottleneck is semantic grounding |
| Eq. 6–7 (interface-conditioned) | §4.3, Table 3 | Validates Hypothesis 1: guide-style $\delta_{\text{sem}} \gg$ tool-based $\delta_{\text{sem}}$ |
| Eq. 8–10 (SGI + justification) | §4.4, Table 4 | Principled weighting via dominance; enables domain comparison |
| Eq. 14–15 (uplift + per-pair) | §4.6, Table 6 | Quantifies downstream benefit |
| Eq. 17–18b (groundability-uplift) | §4.7, new analysis | **Closes the loop**: shows SGI predicts uplift (per-skill regression as primary test) |
| Eq. 19–21 (statistical tests) | §4.6–4.7 | Provides inferential validity with multi-match dependence handling |

### 3.2 Connecting Formalism to the Core Thesis

The paper's central argument is a three-step evidence chain:

> **Step 0 (Diagnosis)**: Medical agent skills exhibit a systematic specification grounding failure, measurable as the RBU gap $\Delta_{\text{RBU}}$ and decomposable into $\delta_{\text{sem}}$ (dominant) and $\delta_{\text{exec}}$.
>
> **Step 1 (Relevance)**: These skills can be matched to real medical QA questions via the matching function $\mu$.
>
> **Step 2 (Utility)**: Skill quality, as diagnosed by SGI and decomposed by tool_type, directly predicts the downstream accuracy uplift $\Delta_{\text{uplift}}$.

The formalism makes this chain **falsifiable**: if the per-skill regression coefficient $\hat{\beta}_1$ (Eq. 18b) is not significantly positive, the loop does not close. The paper must report $\hat{\beta}_1$, its standard error, and $p$-value.

### 3.3 Notation Consistency Checklist

| Symbol | Meaning | First Use |
|--------|---------|-----------|
| $s$ | Skill specification | Def. 1 |
| $\mathcal{S}, \mathcal{S}_A$ | Full/Tier-A skill catalog | Def. 1 |
| $\mathcal{I}$ | Set of interface types | Def. 1 |
| $\mathcal{D}$ | Medical domain taxonomy | Def. 1 |
| $x, \mathcal{X}$ | Task / task space | Def. 2 |
| $\mathcal{M}$ | Agent (executor model) | Def. 2 |
| $\tau$ | Execution trajectory | Def. 2 |
| $a_t, o_t$ | Action, observation at step $t$ | Def. 2 |
| $\mathcal{A}, \mathcal{O}$ | Action / observation spaces | Def. 2 |
| $T_{\max}$ | Max iterations (= 40) | Def. 2 |
| $E$ | Evaluation function | Def. 3 |
| $\mathbf{y}, y_k$ | Evaluation vector / $k$-th dimension | Def. 3 |
| $F_c, Q_a$ | Core fidelity / auxiliary quality | Def. 7 |
| $\Delta_{\text{RBU}}$ | Aggregate RBU gap | Def. 5 |
| $\delta_{\text{sem}}, \delta_{\text{exec}}$ | Semantic / execution gap components | Prop. 1 |
| $\text{SGI}$ | Specification Groundability Index | Def. 7 |
| $w_c$ | Core fidelity weight in SGI | Def. 7 |
| $\mu$ | Skill-matching function ($\mathcal{Q} \to 2^{\mathcal{S}}$) | Def. 8 |
| $\mathcal{P}_\mu$ | Set of matched skill–question pairs | Def. 8 |
| $\Delta_{\text{uplift}}$ | Skill uplift | Def. 9 |
| $u_{i,s}$ | Per-pair uplift indicator | Def. 10 |
| $r_{GU}$ | Groundability-uplift correlation | Eq. 18 |
| $\hat{\beta}_1$ | Per-skill SGI→uplift regression coeff. | Eq. 18b |
| $b, c$ | McNemar discordant counts | Eq. 19 |
| $\mathbf{q}_d$ | Domain quality profile | Def. 11 |

---

## Part IV — Required New Experimental Results

The formalism generates specific empirical demands that must be filled before submission:

### 4.1 Must Compute (Blocking)

1. **Two-stage decomposition table** (new Table 3b): Report $\delta_{\text{sem}}^{(\iota)}$ and $\delta_{\text{exec}}^{(\iota)}$ for each tool_type. This is the data needed to validate Hypothesis 1. Computation: for each skill, you already have $y_1, y_2, y_3$. Just compute means by interface type. The overall values are already computable: $\delta_{\text{sem}} = 0.962 - 0.720 = 0.242$, $\delta_{\text{exec}} = 0.720 - 0.743 = -0.023$.

2. **SGI by domain** (Table 4, currently all TODO): Compute $\bar{F}_{c,d}$ and $\bar{Q}_{a,d}$ for each domain from existing Step 0 data. No new experiments needed. **Note**: Use $w_c = 0.75$ (corrected from 0.6).

3. **Per-skill SGI → uplift regression** $\hat{\beta}_1$ (Eq. 18b): Requires Step 2 (ablation) results per skill. This is the "closing the loop" test. Higher-powered than $r_{GU}$.

### 4.2 Should Compute (Strengthening)

4. **SGI sensitivity analysis** ($w_c \in \{0.6, 0.7, 0.75, 0.8\}$): Verify that domain rankings and regression conclusions are stable. Put in Appendix G.

5. **Per-question uplift distribution** ($n_{+1}, n_0, n_{-1}$): Already planned in Appendix F but not computed. Report the net benefit ratio $\rho$.

6. **Domain quality profile vectors** $\mathbf{q}_d$: Radar charts comparing computational vs. reasoning archetypes.

### 4.3 Nice to Have (Polish)

7. **Formal variance/stability analysis**: Report $\text{Var}(\text{SGI}_d)$ within each domain to assess heterogeneity.
8. **Multi-model RBU decomposition**: If backup Claude runs exist, report whether $\delta_{\text{sem}} / \delta_{\text{exec}}$ ratio is stable across models.

---

## Part V — Revised Section 3 Outline (For Direct Paper Integration)

Below is the recommended structure for Section 3, with equation numbering:

```
§3 Method
  §3.0 Problem Formulation
    - Def 1: Skill Specification (tuple)
    - Def 2: Agent Execution (stochastic trajectory)
    - Def 3: Evaluation Function (8-dim binary)
    - Remark: soft dependency structure (y3 > y2 is possible and informative)

  §3.1 MED-SKILLBENCH: Skill-Level Execution Benchmark
    §3.1.1 Skill Collection and Quality Filtering
      (prose, no equations — same as current)
    §3.1.2 Isolated Evaluation Protocol
      (prose — same as current)
    §3.1.3 Outcome Tiers
      - Eq 1: PASS
      - Eq 2: GOLD
      - Eq 3: SILVER
      - Set partition: G ⊔ S ⊔ F = P (disjoint)
    §3.1.4 The Read-But-Not-Use Gap
      - Eq 4: aggregate RBU
      - Eq 5: two-stage decomposition (δ_sem + δ_exec)
      - Remark: sign interpretation (δ_exec < 0 means parametric bypass)
      - Hypothesis 1: interface grounding (NEW)
      - Eq 6–7: conditioned RBU (NEW)
    §3.1.5 Specification Groundability Index
      - Eq 8: SGI definition
      - Eq 9–10: w_c ≥ 0.75 via dominance principle (CORRECTED)
      - Proposition 2: domain decomposition (NEW)
    §3.1.6 Domain-Stratified Analysis
      - Def 11: domain quality profile (range corrected)

  §3.2 Skill-Augmented QA Evaluation
    §3.2.1 Task Selection and Matching
      (prose — same as current; μ now maps to 2^S)
    §3.2.2 Dual-Mode Execution
      - Eq 12–13: Acc_with, Acc_w/o (over pairs P_μ)
      - Eq 14: uplift
      - Eq 15: per-pair indicator u_{i,s} (NEW)
    §3.2.3 Connecting Diagnostics to Utility
      - Eq 16: interface-conditioned uplift (NEW)
      - Eq 17: groundability-uplift prediction (NEW)
      - Eq 18: r_GU correlation + caveat on power (NEW)
      - Eq 18b: per-skill SGI regression (primary test) (NEW)
    §3.2.4 Statistical Testing
      - Eq 19–20: McNemar formalization (NEW)
      - Remark: multi-match dependence → clustered bootstrap
      - Eq 21: Bootstrap CI (NEW)
```

Total equations: ~20 (up from 5). Each one does real work.

**Note on venue calibration**: ACL benchmark papers typically carry 8–15 equations. At ~20, this is on the heavier side. Consider moving the full statistical testing framework (§3.2.4: Eq. 19–21) to an appendix, reducing the main text to ~17 equations.

---

## Part VI — Summary of Changes

| Aspect | Current State | After Revision |
|--------|--------------|----------------|
| Equations | 5 (all arithmetic identities) | ~20 (definitions + decompositions + predictions + tests) |
| Object definitions | None | 11 formal definitions |
| Decomposition of RBU | None | Two-stage with sign interpretation and per-interface conditioning |
| SGI justification | "we set $w_c = 0.6$" (algebra error) | Dominance principle → $w_c \geq 0.75$ (corrected) |
| Connection Step 0 → Step 2 | Informal prose claim | Per-skill regression (Eq. 18b) as primary test + aggregate correlation (Eq. 18) |
| Statistical framework | "we use McNemar's test" | Full formalization with multi-match dependence handling |
| Notation table | None | Complete 27-symbol table |
| Falsifiability | No formal predictions | Hypothesis 1 + Proposition 3 are falsifiable |

---

## Part VII — v2 Errata: All Issues Found and Fixed

This section documents every issue identified during the advisor-level audit of v1.

### Errata 1 — CRITICAL: SGI weight derivation algebra error

**Location**: §3.1.4, Eq. (10)

**v1 claim**: $w_c \cdot \frac{2}{3} + (1-w_c) \cdot 1 < w_c \cdot 1 + (1-w_c) \cdot 0 \implies w_c > \frac{3}{5} = 0.6$.

**Correct derivation**: Expanding the left side gives $1 - w_c/3$; the right side gives $w_c$. Requiring $1 - w_c/3 < w_c$ yields $1 < 4w_c/3$, hence $w_c > 3/4 = 0.75$.

**Impact**: The v1 value $w_c = 0.6$ does NOT satisfy the dominance principle it claims to derive from. At $w_c = 0.6$: $\text{SGI}(s^{\text{core-fail}}) = 0.6 \cdot 2/3 + 0.4 \cdot 1 = 0.80 > 0.60 = \text{SGI}(s^{\text{aux-fail}})$. The core-failing skill outscores the core-passing skill — the exact opposite of the intended guarantee. Corrected to $w_c \geq 0.75$.

### Errata 2 — CRITICAL: "Three-Stage" label with only two stages

**Location**: Proposition 1 title and §3.1.2 header.

**Issue**: The title says "Three-Stage Decomposition" but the equation contains exactly two terms ($\delta_{\text{sem}}$ and $\delta_{\text{exec}}$). This is a simple counting error.

**Fix**: Renamed to "Two-Stage Decomposition."

### Errata 3 — CRITICAL: Dependency chain contradicted by empirical data

**Location**: Remark after Definition 3.

**v1 claim**: "This yields a natural partial order $y_1 \succeq y_2 \succeq y_3$."

**Issue**: Table 2 shows $\bar{y}_3 = 0.743 > \bar{y}_2 = 0.720$, violating the claimed ordering. An agent can complete a task (y3=1) without correctly following the skill (y2=0) by relying on parametric knowledge. This is not a data error — it is the core phenomenon MED-SKILLBENCH aims to detect.

**Fix**: Rewritten as "soft dependency tendency" with explicit acknowledgment of the empirical violation and its diagnostic significance.

### Errata 4 — MODERATE: Set-theoretic tier ordering error

**Location**: Interpretation paragraph after Definition 4.

**v1 claim**: "$\textsc{Gold} \subset \textsc{Silver} \subset \textsc{Pass}$."

**Issue**: Gold and Silver are disjoint by definition (Silver = Pass ∧ ¬Gold). The v1 statement is set-theoretically false.

**Fix**: Rewritten with correct partition: $\mathbb{P} = \mathbb{G} \;\dot{\cup}\; \mathbb{S} \;\dot{\cup}\; \mathbb{F}$, with $\mathbb{G} \subset \mathbb{P}$ and $\mathbb{S} \subset \mathbb{P}$ but $\mathbb{G} \cap \mathbb{S} = \varnothing$.

### Errata 5 — MODERATE: Matching function μ inconsistent with paper data

**Location**: Definition 8.

**v1 claim**: $\mu: \mathcal{Q} \to \mathcal{S} \cup \{\varnothing\}$ (single skill per question).

**Issue**: The paper reports "a single question may be matched to multiple skills, yielding approximately 3,898 total skill–question pairs across the 2,596 matched questions." The v1 formalization contradicts the paper's own experimental design.

**Fix**: Changed to $\mu: \mathcal{Q} \to 2^{\mathcal{S}}$ (maps to a set of skills). All downstream definitions updated to operate over pairs $\mathcal{P}_\mu$ rather than individual questions. Added remark on multi-match handling for the without-skill baseline.

### Errata 6 — MODERATE: Correlation r_GU has negligible statistical power

**Location**: Eq. (18), Proposition 3.

**Issue**: $r_{GU}$ is computed over $|\mathcal{I}| = 5$ data points. With $n = 5$, the critical value for significance at $\alpha = 0.05$ is $|r| > 0.878$. This is an extremely weak test that will almost certainly be non-significant even if the underlying relationship is real.

**Fix**: Retained $r_{GU}$ as a secondary descriptive statistic but added Eq. (18b): per-skill SGI → uplift linear regression as the primary higher-powered test. Added explicit caveat about the 5-point limitation.

### Errata 7 — MODERATE: Agent modeled as deterministic

**Location**: Definition 2.

**Issue**: LLM agents are inherently stochastic (even at temperature 0, there can be non-determinism from batching, numerical precision, etc.). Modeling $\mathcal{M}$ as a deterministic function is technically imprecise and precludes future multi-run analysis.

**Fix**: Changed to $\tau \sim \mathcal{M}(x, s)$ (stochastic). Added remark noting that temperature-0 experiments are effectively deterministic single draws, and that multi-run variance analysis is a planned extension.

### Errata 8 — MODERATE: Negative δ_exec not addressed

**Location**: Proposition 1 and subsequent discussion.

**Issue**: The v1 text implicitly assumes $\delta_{\text{sem}} \geq 0$ and $\delta_{\text{exec}} \geq 0$ in its narrative ("the bottleneck is execution — the agent acts correctly but tool calls fail"). But empirically $\delta_{\text{exec}} = 0.720 - 0.743 = -0.023 < 0$. The interpretation framework must handle negative values.

**Fix**: Added "Sign Interpretation" remark explaining that negative $\delta_{\text{exec}}$ indicates parametric bypass and is itself a key diagnostic signal. Corrected the domain quality profile range from $[0,1]^{12}$ to $[0,1]^8 \times [-1,1]^3 \times [0,1]$.

### Errata 9 — MINOR: Section numbering out of order

**Location**: §3.1.5 appears after §3.2.2 in the document flow.

**Issue**: Domain-stratified analysis (§3.1.5) is placed after the QA evaluation framework (§3.2), breaking the logical section hierarchy.

**Fix**: In the revised outline (Part V), §3.1.6 correctly follows §3.1.5 before §3.2 begins.

### Errata 10 — MINOR: Multi-match dependence in McNemar's test

**Location**: §3.2.2, Eq. (19–20).

**Issue**: McNemar's test assumes independent pairs. When $|\mu(x_i)| > 1$, pairs sharing the same question have correlated without-skill outcomes, violating independence.

**Fix**: Added remark on clustered bootstrap as robustness check, clustering by question index $i$.

### Errata 11 — ADVISORY: Equation count for ACL venue

**Issue**: 19–20 equations may be heavy for an ACL benchmark paper (typical range: 8–15). Reviewers may perceive over-formalization.

**Recommendation**: Move statistical testing framework (Eq. 19–21) to appendix. This reduces main-text equations to ~17, within acceptable range. The formalism that remains in the main text all serves the paper's narrative directly.

---

*End of document. Execute in order: (1) decide on $w_c$ (0.75 recommended, 0.6 requires different justification), (2) compute $\delta_{\text{sem}}^{(\iota)}$ and $\delta_{\text{exec}}^{(\iota)}$ from existing data, (3) integrate formalism into §3, (4) run Step 2 and compute $\hat{\beta}_1$, (5) sensitivity analysis for Appendix G.*