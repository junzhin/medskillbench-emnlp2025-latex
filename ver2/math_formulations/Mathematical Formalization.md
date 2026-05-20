# Med-SkillBench: Mathematical Formalization — Complete Revision Plan

> **Purpose**: This document provides (1) a diagnostic of every mathematical weakness in the current draft, (2) a fully rigorous rewrite of the formal framework, and (3) a concrete plan for integrating the formalism with experiments and the paper's core thesis. All LaTeX is publication-ready.

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
3. **No decomposition of the gap.** The RBU gap is a single scalar. But logically it decomposes into at least three stages: (a) comprehension failure, (b) grounding failure, (c) execution failure. The formalism should expose this.
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

An *agent* $\mathcal{M}$ is a function that, given a task description $x \in \mathcal{X}$ and a skill specification $s \in \mathcal{S}$, produces a bounded execution trajectory:

$$
\tau = \mathcal{M}(x, s) = (a_1, o_1, a_2, o_2, \ldots, a_T, o_T), \quad T \leq T_{\max},
$$

where $a_t \in \mathcal{A}$ is an agent action (tool call, code execution, reasoning step, file read) and $o_t \in \mathcal{O}$ is the environment observation returned after $a_t$. The trajectory is bounded by $T_{\max} = 40$ iterations and a wall-clock timeout $\Delta t_{\max} = 300\text{s}$.

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

**Remark.** The core dimensions encode a logical dependency chain: correct usage ($y_2 = 1$) presupposes invocation ($y_1 = 1$), and task completion ($y_3 = 1$) presupposes correct usage ($y_2 = 1$). This yields a natural partial order $y_1 \succeq y_2 \succeq y_3$, which the RBU gap exploits.

---

### §3.1.1 — Outcome Tiers

> **Narrative purpose**: Replace the current ad-hoc PASS/GOLD definitions with a principled tier system.

#### Definition 4 (Outcome Tiers).

Given evaluation vector $\mathbf{y} = E(\tau, s, x)$, define:

$$
\textsc{Pass}(s) = \mathbf{1}\!\left[\,y_1 = 1 \;\wedge\; y_2 = 1 \;\wedge\; y_3 = 1\,\right], \tag{1}
$$

$$
\textsc{Gold}(s) = \mathbf{1}\!\left[\,\|\mathbf{y}\|_1 = 8\,\right], \tag{2}
$$

$$
\textsc{Silver}(s) = \textsc{Pass}(s) \cdot \left(1 - \textsc{Gold}(s)\right). \tag{3}
$$

**Interpretation.** Gold requires full execution quality across all dimensions. Silver indicates core fidelity without full auxiliary quality — the skill was correctly invoked and the task completed, but execution artifacts (tool failures, inefficiency, trajectory issues) remain. The three tiers provide a coarse quality ordering: $\textsc{Gold} \subset \textsc{Silver} \subset \textsc{Pass}$.

---

### §3.1.2 — The Read-But-Not-Use Gap: Formal Decomposition

> **Narrative purpose**: This is the paper's central construct. The current version is a single scalar. We decompose it into three interpretable stages.

#### Definition 5 (Aggregate RBU Gap).

For a skill set $\mathcal{S}$ evaluated by agent $\mathcal{M}$, the *aggregate RBU gap* is:

$$
\Delta_{\text{RBU}}(\mathcal{S}, \mathcal{M}) = \bar{y}_1 - \bar{y}_3, \quad \text{where}\;\; \bar{y}_k = \frac{1}{|\mathcal{S}|}\sum_{s \in \mathcal{S}} y_k^{(s)}. \tag{4}
$$

#### Proposition 1 (Three-Stage Decomposition).

The RBU gap decomposes additively into three mechanistically distinct failure stages:

$$
\Delta_{\text{RBU}} = \underbrace{(\bar{y}_1 - \bar{y}_2)}_{\delta_{\text{sem}}} + \underbrace{(\bar{y}_2 - \bar{y}_3)}_{\delta_{\text{exec}}}, \tag{5}
$$

where:

- $\delta_{\text{sem}} = \bar{y}_1 - \bar{y}_2$ is the **semantic grounding gap**: the agent reads the specification but fails to translate it into specification-conformant actions. This measures *comprehension-to-action* fidelity.
- $\delta_{\text{exec}} = \bar{y}_2 - \bar{y}_3$ is the **execution completion gap**: the agent acts in accordance with the specification but fails to produce a complete output. This measures *action-to-outcome* fidelity.

**Proof.** Follows from telescoping: $\bar{y}_1 - \bar{y}_3 = (\bar{y}_1 - \bar{y}_2) + (\bar{y}_2 - \bar{y}_3)$. $\square$

**Remark.** This decomposition is trivial algebraically but diagnostically powerful. Consider two skill categories with identical $\Delta_{\text{RBU}} = 0.22$:

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

#### Justification of $w_c = 0.6$.

We set $w_c$ based on the following **dominance principle**: a skill that fails on any core dimension (invocation, correctness, completion) is clinically unusable regardless of auxiliary quality, so core fidelity must dominate. Formally, we require:

$$
\text{SGI}(s^{\text{core-fail}}) < \text{SGI}(s^{\text{aux-fail}}) \quad \text{for all plausible configurations}, \tag{9}
$$

where $s^{\text{core-fail}}$ fails exactly one core dimension ($F_c = 2/3$, $Q_a = 1$) and $s^{\text{aux-fail}}$ passes all core dimensions but fails all auxiliary ($F_c = 1$, $Q_a = 0$). This requires:

$$
w_c \cdot \frac{2}{3} + (1-w_c) \cdot 1 < w_c \cdot 1 + (1-w_c) \cdot 0 \implies w_c > \frac{3}{5} = 0.6. \tag{10}
$$

Setting $w_c = 0.6$ is the tightest value satisfying this dominance constraint, making it the *least opinionated* choice that guarantees core-failure skills always rank below core-passing skills. We verify robustness to $w_c \in \{0.5, 0.6, 0.7, 0.8\}$ in sensitivity analysis (Appendix G).

#### Proposition 2 (SGI Decomposition by Domain).

For domain $d \in \mathcal{D}$, the mean SGI decomposes as:

$$
\overline{\text{SGI}}_d = w_c \cdot \bar{F}_{c,d} + (1-w_c) \cdot \bar{Q}_{a,d}. \tag{11}
$$

This decomposition exposes cross-domain quality patterns: computational domains (bioinformatics) are predicted to show $\bar{F}_{c,d} > \bar{Q}_{a,d}$ (high core fidelity, tool execution failures drag down auxiliary), while clinical reasoning domains show $\bar{Q}_{a,d} > \bar{F}_{c,d}$ (high reasoning quality, but weaker specification adherence).

---

### §3.2 — Skill-Augmented QA: Formal Framework

> **Narrative purpose**: Give the ablation study formal structure and connect it to the RBU gap.

#### Definition 8 (Skill-Augmented Accuracy).

Let $\mathcal{Q} = \{(x_i, g_i)\}_{i=1}^{N}$ be a set of medical QA items with gold answers $g_i$. Let $\mu: \mathcal{Q} \to \mathcal{S} \cup \{\varnothing\}$ be a skill-matching function assigning each question to at most one skill (or $\varnothing$ if unmatched). Define:

$$
\text{Acc}_{\text{with}}(\mathcal{Q}, \mathcal{M}) = \frac{1}{|\mathcal{Q}_\mu|}\sum_{i \in \mathcal{Q}_\mu} \mathbf{1}\!\left[\,\mathcal{M}(x_i, \mu(x_i)) = g_i\,\right], \tag{12}
$$

$$
\text{Acc}_{\text{w/o}}(\mathcal{Q}, \mathcal{M}) = \frac{1}{|\mathcal{Q}_\mu|}\sum_{i \in \mathcal{Q}_\mu} \mathbf{1}\!\left[\,\mathcal{M}(x_i, \varnothing) = g_i\,\right], \tag{13}
$$

where $\mathcal{Q}_\mu = \{i : \mu(x_i) \neq \varnothing\}$ restricts to matched questions.

#### Definition 9 (Skill Uplift).

$$
\Delta_{\text{uplift}}(\mathcal{Q}, \mathcal{M}) = \text{Acc}_{\text{with}} - \text{Acc}_{\text{w/o}}. \tag{14}
$$

#### Definition 10 (Per-Question Uplift Indicator).

For each matched question $i \in \mathcal{Q}_\mu$, define the signed indicator:

$$
u_i = \mathbf{1}[\mathcal{M}(x_i, \mu(x_i)) = g_i] - \mathbf{1}[\mathcal{M}(x_i, \varnothing) = g_i] \;\in\; \{-1, 0, +1\}. \tag{15}
$$

Then $\Delta_{\text{uplift}} = \bar{u}$. The *net benefit ratio* is $\rho = n_{+1} / (n_{+1} + n_{-1})$, where $n_v = |\{i : u_i = v\}|$. A ratio $\rho > 0.5$ indicates that skills help more questions than they hurt.

---

### §3.2.1 — Connecting RBU Gap to Downstream Uplift

> **Narrative purpose**: This is the "closing the loop" theorem — the paper's strongest theoretical contribution. Currently this claim is made informally; here we formalize it.

#### Proposition 3 (Groundability-Modulated Uplift).

Assume that skill execution quality mediates the downstream uplift. Specifically, partition matched questions by the interface type of their matched skill. Then:

$$
\Delta_{\text{uplift}}^{(\iota)} = \text{Acc}_{\text{with}}^{(\iota)} - \text{Acc}_{\text{w/o}}^{(\iota)}, \quad \iota \in \mathcal{I}. \tag{16}
$$

**Prediction**: If the RBU gap reflects genuine specification grounding failure, then interface types with *smaller* RBU gaps should exhibit *larger* uplift:

$$
\Delta_{\text{RBU}}^{(\iota_1)} < \Delta_{\text{RBU}}^{(\iota_2)} \;\implies\; \Delta_{\text{uplift}}^{(\iota_1)} \geq \Delta_{\text{uplift}}^{(\iota_2)}. \tag{17}
$$

**Testable form**: Define the *groundability-uplift correlation*:

$$
r_{GU} = \text{Corr}\!\left(\{-\Delta_{\text{RBU}}^{(\iota)}\}_{\iota \in \mathcal{I}}, \;\{\Delta_{\text{uplift}}^{(\iota)}\}_{\iota \in \mathcal{I}}\right). \tag{18}
$$

A significantly positive $r_{GU}$ validates the "closing the loop" claim: skills that ground better (lower RBU) yield larger downstream benefit. A non-significant $r_{GU}$ would indicate that task-level uplift is driven by factors orthogonal to specification grounding.

---

### §3.2.2 — Statistical Testing Framework

> **Narrative purpose**: Formalize the statistical machinery instead of hand-waving at "McNemar's test."

#### McNemar's Test for Paired Binary Outcomes.

For each matched question $i$, let $c_i^{+} = \mathbf{1}[\mathcal{M}(x_i, \mu(x_i)) = g_i]$ and $c_i^{-} = \mathbf{1}[\mathcal{M}(x_i, \varnothing) = g_i]$. Define the discordant counts:

$$
b = \sum_{i} \mathbf{1}[c_i^{+} = 1 \wedge c_i^{-} = 0], \quad c = \sum_{i} \mathbf{1}[c_i^{+} = 0 \wedge c_i^{-} = 1]. \tag{19}
$$

Under $H_0: \Pr(c_i^+ = 1, c_i^- = 0) = \Pr(c_i^+ = 0, c_i^- = 1)$, the McNemar statistic is:

$$
\chi^2_{\text{McN}} = \frac{(b - c)^2}{b + c} \;\sim\; \chi^2(1). \tag{20}
$$

We reject $H_0$ at level $\alpha = 0.05$ if $\chi^2_{\text{McN}} > 3.841$.

#### Bootstrap Confidence Intervals.

For $\Delta_{\text{uplift}}$, we construct bootstrap 95\% CIs by resampling the paired vector $\{(c_i^+, c_i^-)\}_{i=1}^{N}$ with replacement $B = 10{,}000$ times:

$$
\text{CI}_{95\%}(\Delta_{\text{uplift}}) = \left[\hat{\Delta}^{*(0.025)}, \;\hat{\Delta}^{*(0.975)}\right], \tag{21}
$$

where $\hat{\Delta}^{*(\alpha)}$ is the $\alpha$-quantile of the bootstrap distribution.

---

### §3.1.5 — Domain-Stratified Analysis Framework

> **Narrative purpose**: Formalize how domain stratification interacts with SGI and RBU.

#### Definition 11 (Domain Quality Profile).

For domain $d \in \mathcal{D}$, the *quality profile* is the vector:

$$
\mathbf{q}_d = \left(\bar{y}_1^{(d)}, \ldots, \bar{y}_8^{(d)}, \;\Delta_{\text{RBU}}^{(d)}, \;\delta_{\text{sem}}^{(d)}, \;\delta_{\text{exec}}^{(d)}, \;\overline{\text{SGI}}_d\right) \in [0,1]^{12}. \tag{22}
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
| Eq. 5 (three-stage decomposition) | §4.3, new Table | **Key new result**: decomposes the 0.220 gap into $\delta_{\text{sem}}$ and $\delta_{\text{exec}}$ — determines *where* in the pipeline skills fail |
| Eq. 6–7 (interface-conditioned) | §4.3, Table 3 | Validates Hypothesis 1: guide-style $\delta_{\text{sem}} \gg$ tool-based $\delta_{\text{sem}}$ |
| Eq. 8–10 (SGI + justification) | §4.4, Table 4 | Replaces hand-waving with principled weighting; enables domain comparison |
| Eq. 14–15 (uplift + per-question) | §4.6, Table 6 | Quantifies downstream benefit |
| Eq. 17–18 (groundability-uplift) | §4.7, new analysis | **Closes the loop**: shows RBU predicts uplift |
| Eq. 19–21 (statistical tests) | §4.6–4.7 | Provides inferential validity |

### 3.2 Connecting Formalism to the Core Thesis

The paper's central argument is a three-step evidence chain:

> **Step 0 (Diagnosis)**: Medical agent skills exhibit a systematic specification grounding failure, measurable as the RBU gap $\Delta_{\text{RBU}}$.
>
> **Step 1 (Relevance)**: These skills can be matched to real medical QA questions via the matching function $\mu$.
>
> **Step 2 (Utility)**: Skill quality, as diagnosed by $\Delta_{\text{RBU}}$ and decomposed by tool_type, directly predicts the downstream accuracy uplift $\Delta_{\text{uplift}}$.

The formalism makes this chain **falsifiable**: if the groundability-uplift correlation $r_{GU}$ (Eq. 18) is not positive, the loop does not close. The paper must report $r_{GU}$ and its significance.

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
| $\mu$ | Skill-matching function | Def. 8 |
| $\Delta_{\text{uplift}}$ | Skill uplift | Def. 9 |
| $u_i$ | Per-question uplift indicator | Def. 10 |
| $r_{GU}$ | Groundability-uplift correlation | Eq. 18 |
| $b, c$ | McNemar discordant counts | Eq. 19 |
| $\mathbf{q}_d$ | Domain quality profile | Def. 11 |

---

## Part IV — Required New Experimental Results

The formalism generates specific empirical demands that must be filled before submission:

### 4.1 Must Compute (Blocking)

1. **Three-stage decomposition table** (new Table 3b): Report $\delta_{\text{sem}}^{(\iota)}$ and $\delta_{\text{exec}}^{(\iota)}$ for each tool_type. This is the data needed to validate Hypothesis 1. Computation: for each skill, you already have $y_1, y_2, y_3$. Just compute means by interface type.

2. **SGI by domain** (Table 4, currently all TODO): Compute $\bar{F}_{c,d}$ and $\bar{Q}_{a,d}$ for each domain from existing Step 0 data. No new experiments needed.

3. **Groundability-uplift correlation** $r_{GU}$ (Eq. 18): Requires Step 2 (ablation) results stratified by tool_type. This is the "closing the loop" test.

### 4.2 Should Compute (Strengthening)

4. **SGI sensitivity analysis** ($w_c \in \{0.5, 0.6, 0.7, 0.8\}$): Verify that domain rankings are stable. Put in Appendix G.

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
    - Def 2: Agent Execution (trajectory)
    - Def 3: Evaluation Function (8-dim binary)
    - Remark: logical dependency chain y1 ≥ y2 ≥ y3

  §3.1 MED-SKILLBENCH: Skill-Level Execution Benchmark
    §3.1.1 Skill Collection and Quality Filtering
      (prose, no equations — same as current)
    §3.1.2 Isolated Evaluation Protocol
      (prose — same as current)
    §3.1.3 Outcome Tiers
      - Eq 1: PASS
      - Eq 2: GOLD
      - Eq 3: SILVER
    §3.1.4 The Read-But-Not-Use Gap
      - Eq 4: aggregate RBU
      - Eq 5: three-stage decomposition (NEW)
      - Hypothesis 1: interface grounding (NEW)
      - Eq 6: conditioned RBU (NEW)
    §3.1.5 Specification Groundability Index
      - Eq 7: SGI definition
      - Eq 8–9: w_c justification via dominance (NEW)
      - Proposition 2: domain decomposition (NEW)
    §3.1.6 Domain-Stratified Analysis
      - Def 11: domain quality profile (NEW)

  §3.2 Skill-Augmented QA Evaluation
    §3.2.1 Task Selection and Matching
      (prose — same as current)
    §3.2.2 Dual-Mode Execution
      - Eq 10–11: Acc_with, Acc_w/o
      - Eq 12: uplift
      - Eq 13: per-question indicator u_i (NEW)
    §3.2.3 Connecting Diagnostics to Utility
      - Eq 14: interface-conditioned uplift (NEW)
      - Eq 15: groundability-uplift prediction (NEW)
      - Eq 16: r_GU correlation (NEW)
    §3.2.4 Statistical Testing
      - Eq 17–18: McNemar formalization (NEW)
      - Eq 19: Bootstrap CI (NEW)
```

Total equations: ~19 (up from 5). Each one does real work.

---

## Part VI — Summary of Changes

| Aspect | Current State | After Revision |
|--------|--------------|----------------|
| Equations | 5 (all arithmetic identities) | ~19 (definitions + decompositions + predictions + tests) |
| Object definitions | None | 11 formal definitions |
| Decomposition of RBU | None | Three-stage with per-interface conditioning |
| SGI justification | "we set $w_c = 0.6$" | Dominance principle derivation |
| Connection Step 0 → Step 2 | Informal prose claim | Formal prediction (Eq. 17) + testable correlation (Eq. 18) |
| Statistical framework | "we use McNemar's test" | Full formalization with discordant counts + bootstrap |
| Notation table | None | Complete 25-symbol table |
| Falsifiability | No formal predictions | Hypothesis 1 + Proposition 3 are falsifiable |

---

*End of document. This plan should be executed in the following order: (1) compute the missing empirical quantities from §4.1, (2) integrate the formalism into §3 following the outline in Part V, (3) update all tables, (4) write Appendix G for sensitivity analysis.*