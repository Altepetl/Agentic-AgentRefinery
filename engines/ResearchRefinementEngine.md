# Iterative refinement algorithm for LLM-generated research results

## 1. Objective

Design a process capable of taking the results of **two or more independent research runs performed by LLMs**, comparing them, detecting matches and contradictions, evaluating the evidence, ranking the conclusions, and producing a refined result.

The goal is not simply to combine two responses.

The goal is:

> **Progressively increase the quality, consistency, coverage, and reliability of the result through multiple independent research runs and iterative refinement.**

The algorithm must be able to evolve from:

```text
Research A
       ↓
Result A
```

and:

```text
Research B
       ↓
Result B
```

towards:

```text
Result A + Result B
            ↓
      Semantic fusion
            ↓
       Comparison
            ↓
    Evidence / consensus
            ↓
     Contradictions
            ↓
        Ranking
            ↓
      LLM Synthesis
            ↓
    Refined result
```

And subsequently allow for:

```text
Refined result
        ↓
Detect uncertainties
        ↓
Generate new research
        ↓
New result
        ↓
Re-run the process
```

---

# 2. General architecture

The proposed architecture is:

```text
                           ┌───────────────────────┐
                           │      Research A       │
                           │        LLM #1         │
                           └───────────┬───────────┘
                                       │
                                       ▼
                                Results A
                                       │
                                       │
                           ┌───────────▼───────────┐
                           │      Research B       │
                           │        LLM #2         │
                           └───────────┬───────────┘
                                       │
                                       ▼
                                Results B
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │  1. NORMALIZATION         │
                         │                           │
                         │ Convert results to a     │
                         │ common structure          │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │  2. EMBEDDINGS / MATCHING │
                         │                           │
                         │ Detect semantic           │
                         │ equivalences               │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │  3. CLUSTERING             │
                         │                           │
                         │ Group related             │
                         │ conclusions                 │
                         └─────────────┬─────────────┘
                                       │
                         ┌─────────────┴─────────────┐
                         ▼                           ▼
                   Matches                       Differences
                         │                           │
                         ▼                           ▼
             ┌──────────────────┐       ┌─────────────────────┐
             │ 4. EVIDENCE      │       │ 5. CONTRADICTION    │
             │    SCORING       │       │    DETECTION        │
             └────────┬─────────┘       └──────────┬──────────┘
                      │                            │
                      └─────────────┬──────────────┘
                                    ▼
                         ┌───────────────────────────┐
                         │  6. RANKING / RRF          │
                         │                           │
                         │ Prioritize the most       │
                         │ reliable results           │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                         ┌───────────────────────────┐
                         │  7. LLM SYNTHESIS / JUDGE  │
                         │                           │
                         │ Generate the refined      │
                         │ result                     │
                         └─────────────┬─────────────┘
                                       │
                                       ▼
                              REFINED RESULT
                                       │
                                       ▼
                          Does uncertainty exist?
                              /             \
                            YES              NO
                            │                 │
                            ▼                 ▼
                    New research           FINAL
                            │
                            └───────────┐
                                        │
                                        ▼
                                NEXT ITERATION
```

---

# 3. Recommended data model

Before executing the seven steps, all results must be converted to a common structure.

A research run might produce:

```json
{
  "research_id": "research-001",
  "model": "model-a",
  "timestamp": "2026-07-28T12:00:00Z",
  "question": "Original question",
  "results": [
    {
      "id": "result-001",
      "type": "finding",
      "claim": "The system must use a decoupled architecture.",
      "explanation": "Decoupling makes it easier for components to evolve independently.",
      "evidence": [
        "Source 1",
        "Source 2"
      ],
      "confidence": 0.82,
      "source_rank": 1
    }
  ]
}
```

The second LLM might produce:

```json
{
  "research_id": "research-002",
  "model": "model-b",
  "timestamp": "2026-07-28T12:05:00Z",
  "question": "Original question",
  "results": [
    {
      "id": "result-002",
      "type": "finding",
      "claim": "Components must minimize their dependencies.",
      "explanation": "This allows modules to be modified without affecting the rest of the system.",
      "evidence": [
        "Source 3"
      ],
      "confidence": 0.89,
      "source_rank": 2
    }
  ]
}
```

The common structure is important because it allows subsequent algorithms to work on comparable data.

---

# 4. Step 1 — Embeddings and Semantic Matching

## Objective

Determine when two different results express essentially the same idea.

We must not compare text strings alone.

For example:

```text
Result A:
"The system must reduce coupling between components."

Result B:
"The architecture must minimize dependencies between modules."
```

Although the phrasing is different, they may semantically represent the same conclusion.

---

## Process

Each `claim` is converted into a vector:

```text
Claim A
   ↓
Embedding Model
   ↓
[0.21, -0.18, 0.72, ...]
```

and:

```text
Claim B
   ↓
Embedding Model
   ↓
[0.19, -0.15, 0.69, ...]
```

A similarity is then computed, for example via **Cosine Similarity**.

```text
similarity(A, B) = cosine(vectorA, vectorB)
```

An initial policy could be:

```text
>= 0.90  → practically equivalent
0.80-0.89 → probably related
0.70-0.79 → possible relationship
< 0.70  → probably different
```

These values must be calibrated with real data from the system.

---

## Instructions for the LLM

The LLM participating in this stage must follow these rules:

```text
Your task is to identify semantic equivalences between results from
independent research runs.

Do not compare only words.

Determine whether two statements express:

1. The same conclusion.
2. Compatible but different conclusions.
3. Partially related conclusions.
4. Contradictory conclusions.
5. Completely independent conclusions.

Do not declare two results equivalent just because they use
similar words.

Analyze the full meaning of each statement.

Always keep the original results and do not remove information
during this stage.

For each pair of results, return:

- result_a
- result_b
- relationship
- semantic_similarity
- explanation

relationship must be one of:

EQUIVALENT
RELATED
PARTIALLY_RELATED
CONTRADICTORY
INDEPENDENT
```

---

# 5. Step 2 — Semantic Clustering

## Objective

Group results that speak to the same concept.

After semantic matching we may have:

```text
Cluster 1
├── A1: decoupling
├── B3: dependency reduction
└── A7: module independence

Cluster 2
├── A2: scalability
└── B5: horizontal growth

Cluster 3
├── A4: security
├── B1: authentication
└── B8: authorization
```

This allows working with **concepts** instead of treating each result individually.

---

## Possible algorithms

Depending on the dataset size:

### K-Means

Suitable when you approximately know the number of groups.

### DBSCAN

Suitable when you want to automatically detect groups and outliers.

### Hierarchical Clustering

Very useful when you want to preserve a hierarchical structure:

```text
Architecture
├── Coupling
│   ├── Dependencies
│   └── Modularity
└── Scalability
    ├── Horizontal
    └── Vertical
```

For semantic research results, **Hierarchical Clustering or DBSCAN** tend to be good initial options.

---

## Instructions for the LLM

```text
Group the semantically related results.

Do not group results just because they belong to the same topic.

A cluster must represent a common idea, concept, conclusion, or
phenomenon.

For each cluster:

1. Identify the central concept.
2. List all related results.
3. Identify which results come from each research run.
4. Determine whether the results are equivalent, complementary, or
   partially related.
5. Identify isolated results that cannot be associated with any
   cluster.

Do not remove results.

If two results seem similar but contain important differences,
keep them within the same cluster and record the differences.

Return:

- cluster_id
- central_concept
- members
- investigations
- similarities
- differences
- unresolved_questions
```

---

# 6. Step 3 — Evidence Scoring

## Objective

Determine how strong each conclusion is.

Not all results from an LLM should carry the same weight.

We can evaluate factors such as:

```text
E = f(
    consensus,
    source_quality,
    independence,
    confidence,
    original_position,
    evidence,
    consistency
)
```

A conceptual score could be:

```text
Evidence Score =
    25% consensus
  + 20% evidence quality
  + 20% independence of research runs
  + 15% model confidence
  + 10% consistency
  + 10% relevance
```

The weights are only an example.

---

## Consensus

If:

```text
Research A → conclusion X
Research B → conclusion X
Research C → conclusion X
```

consensus increases.

But there is an important rule:

> **Three LLMs repeating the same information does not necessarily equal three independent pieces of evidence.**

If all three models obtained the information from the same source, the real consensus is lower.

---

## Instructions for the LLM

```text
Evaluate the strength of each conclusion.

Do not confuse repetition with independent evidence.

For each conclusion analyze:

1. How many research runs found it?
2. Are the research runs independent?
3. What evidence supports the conclusion?
4. Are the sources reliable?
5. Is the conclusion directly supported or is it an inference?
6. Does contrary evidence exist?
7. What level of uncertainty exists?
8. Is the conclusion relevant to the original question?

Assign separate scores:

consensus_score
evidence_quality_score
independence_score
reasoning_quality_score
contradiction_penalty
uncertainty_score
relevance_score

Then compute an overall evidence_score.

Never increase the score solely because a statement was
repeated by several models.

Distinguish between:

DIRECT_EVIDENCE
INDIRECT_EVIDENCE
INFERENCE
SPECULATION
UNSUPPORTED_CLAIM

Briefly explain why you assigned each score.
```

---

# 7. Step 4 — Contradiction Detection

## Objective

Find situations where the research runs do not agree.

Example:

```text
Research A:
"Redis must be used as persistent storage."

Research B:
"Redis must not be used as persistent storage."
```

This must not be resolved automatically by selecting A or B.

It must be marked:

```text
CONTRADICTION
```

and passed to a resolution phase.

---

## Types of contradiction

It is recommended to distinguish:

```text
DIRECT_CONTRADICTION

PARTIAL_CONTRADICTION

CONTEXT_DEPENDENT_CONTRADICTION

TEMPORAL_CONTRADICTION

ASSUMPTION_CONFLICT

EVIDENCE_CONFLICT
```

For example:

```text
A:
"X is better for small systems."

B:
"Y is better for large systems."
```

There is no real contradiction here.

The context is different.

---

## Instructions for the LLM

```text
Compare the conclusions of the research runs and detect
contradictions.

Do not treat a difference in wording as a contradiction.

Before marking CONTRADICTORY:

1. Identify exactly what A claims.
2. Identify exactly what B claims.
3. Identify the conditions under which each claim is valid.
4. Identify implicit assumptions.
5. Determine whether both claims can be true under different
   conditions.

Classify the relationship as:

CONSISTENT
COMPLEMENTARY
PARTIALLY_CONFLICTING
CONTEXT_DEPENDENT
DIRECTLY_CONTRADICTORY
INSUFFICIENT_INFORMATION

Never resolve a contradiction by inventing information.

When a contradiction exists, return:

- claim_a
- claim_b
- contradiction_type
- conflicting_assumption
- evidence_a
- evidence_b
- missing_information
- recommended_research
```

---

# 8. Step 5 — Ranking and RRF

## Objective

Combine the rankings produced by different research runs.

Suppose:

```text
Research A:

1. X
2. Y
3. Z
4. W
```

and:

```text
Research B:

1. Y
2. X
3. W
4. Z
```

An appropriate technique is:

**Reciprocal Rank Fusion (RRF)**.

The formula:

```text
RRF(d) = Σ 1 / (k + rank_i(d))
```

where:

```text
d     = result
rank  = position of the result in a research run
k     = constant
i     = research run
```

The result that consistently appears well positioned across several research runs obtains a higher score.

---

## Hybrid ranking

In a real system it is not advisable to use RRF alone.

We can combine:

```text
Final Score =
      RRF Score
    + Evidence Score
    + Consensus Score
    + Relevance Score
    - Contradiction Penalty
    - Uncertainty Penalty
```

This produces a much richer ranking.

---

## Instructions for the LLM

```text
Build a global ranking of the conclusions.

Consider:

- original ranking of each research run
- available evidence
- consensus
- independence of sources
- relevance with respect to the question
- contradictions
- uncertainty

Do not favor a conclusion just because it appears in the
first position of a research run.

A conclusion should rise in the ranking when there is convergence
between independent research runs and solid evidence.

A conclusion should fall when:

- contradictory evidence exists
- it depends on unproven assumptions
- it has low relevance
- it is speculative
- it lacks sufficient evidence

Return:

- rank
- claim
- source_investigations
- rrf_score
- evidence_score
- consensus_score
- contradiction_penalty
- uncertainty
- final_score
- explanation
```

---

# 9. Step 6 — LLM Judge / Synthesis

## Objective

This stage uses an LLM to transform the processed results into a coherent conclusion.

The LLM must not simply summarize.

It must perform an **evidence-based synthesis**.

Input:

```text
Original question
        +
Grouped results
        +
Evidence scores
        +
Contradictions
        +
Ranking
        +
Uncertainties
```

Output:

```text
Refined result
```

---

## Fundamental rule

The LLM must distinguish:

```text
FACT
INFERENCE
HYPOTHESIS
UNCERTAINTY
CONTRADICTION
```

It must not turn a hypothesis into a fact just because several models mentioned it.

---

## Instructions for the LLM

```text
Act as a judge and synthesizer of research results.

Your goal is not to automatically pick the answer with the
highest frequency.

Your goal is to produce the conclusion most supported by the
available evidence.

Analyze:

1. The original question.
2. The results of all research runs.
3. The semantic equivalences.
4. The clusters.
5. The evidence.
6. The contradictions.
7. The scores.
8. The uncertainties.
9. The global ranking.

For each important conclusion determine whether it is:

FACT
STRONG_CONCLUSION
LIKELY
POSSIBLE
SPECULATIVE
UNRESOLVED
CONTRADICTED

Rules:

- Do not invent evidence.
- Do not invent sources.
- Do not hide contradictions.
- Do not remove results that contradict the majority.
- Do not confuse consensus with truth.
- Do not turn inferences into facts.
- Explain the main uncertainties.
- Prioritize independent evidence.
- If there is not enough information, say so explicitly.

Produce:

1. Main answer.
2. Main conclusions.
3. Evidence supporting them.
4. Contradictory conclusions.
5. Uncertainties.
6. Assumptions.
7. What additional information would be needed to increase
   confidence.
```

---

# 10. Step 7 — Iterative Research / Active Refinement

## Objective

This is the component that turns the system from a simple aggregation of results into an **adaptive research system**.

After obtaining the refined result, the system must ask itself:

> Which part of the answer still shows uncertainty?

For example:

```text
Refined result:

"The decoupled architecture appears to be the best option."

Confidence = 0.76

Uncertainty:
"There is uncertainty about the latency impact."
```

The system then generates a new research run:

```text
Original question
        ↓
Refined result
        ↓
Detect uncertainty
        ↓
Generate specific question
        ↓
Research C
        ↓
Result C
        ↓
Merge A + B + C
```

---

## Targeted research

Instead of doing:

```text
"Research everything again."
```

one should do:

```text
"Specifically research the latency impact
of the decoupled architecture under conditions X."
```

This reduces:

* cost
* tokens
* time
* noise
* redundant information

and increases useful information.

---

## Instructions for the LLM

```text
Analyze the refined result and identify the areas where
significant uncertainty exists.

Do not generate new research if the existing information
is already sufficiently solid.

Look only for uncertainties that could change the conclusion.

Classify each uncertainty according to:

IMPACT:
LOW
MEDIUM
HIGH
CRITICAL

If an uncertainty could significantly change the answer,
generate specific research to resolve it.

The new research must:

1. Be specific.
2. Directly target the uncertainty.
3. Avoid repeating previous research.
4. Seek independent evidence.
5. Attempt to confirm and refute the current hypothesis.
6. Produce results that can be integrated back
   into the pipeline.

Return:

- uncertainty
- impact
- why_it_matters
- research_question
- expected_information
- recommended_sources
- falsification_strategy
```

---

# 11. Complete refinement loop

The complete system becomes:

```text
                  ┌─────────────────────┐
                  │   Original question │
                  └──────────┬──────────┘
                             │
                ┌────────────┴────────────┐
                ▼                         ▼
          Research A                 Research B
             LLM #1                    LLM #2
                │                         │
                ▼                         ▼
          Results A                  Results B
                │                         │
                └────────────┬────────────┘
                             ▼
                   1. Normalization
                             │
                             ▼
                   2. Semantic Matching
                             │
                             ▼
                     3. Clustering
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                 Similar          Different
                    │                 │
                    ▼                 ▼
              4. Evidence       5. Contradiction
                  Score             Detection
                    │                 │
                    └────────┬────────┘
                             ▼
                       6. Ranking
                        / RRF
                             │
                             ▼
                     7. LLM Judge
                       / Synthesis
                             │
                             ▼
                    Refined result
                             │
                             ▼
                   Evaluate confidence
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
             Low confidence    Sufficient confidence
                    │                 │
                    ▼                 ▼
             New research           FINAL
                    │
                    ▼
             Research C
                    │
                    ▼
             Repeat pipeline
```

---

# 12. Structure of the refined result

The final result should retain information about how the conclusion was reached.

A recommended structure:

```json
{
  "question": "Original question",

  "iteration": 2,

  "conclusion": {
    "text": "Refined conclusion",
    "confidence": 0.87,
    "classification": "STRONG_CONCLUSION"
  },

  "supporting_findings": [
    {
      "claim": "Conclusion from research A",
      "evidence_score": 0.91,
      "sources": ["A"]
    },
    {
      "claim": "Equivalent conclusion from research B",
      "evidence_score": 0.86,
      "sources": ["B"]
    }
  ],

  "clusters": [],

  "contradictions": [],

  "uncertainties": [
    {
      "question": "What is the latency impact?",
      "impact": "MEDIUM"
    }
  ],

  "researches": [
    "research-001",
    "research-002"
  ],

  "next_research": {
    "required": true,
    "question": "Research the latency impact..."
  }
}
```

---

# 13. Confidence metric

It is advisable to maintain a confidence metric separate from the ranking.

For example:

```text
Confidence =
    Evidence Quality
  × Independence
  × Consistency
  × Relevance
  × Coverage
```

It is not necessary to use exactly this formula.

What matters is that confidence does not depend exclusively on the number of LLMs that gave the same answer.

A model may produce:

```text
Confidence = 0.95
```

but if:

```text
Evidence Quality = 0.20
```

the overall conclusion should not have high confidence.

---

# 14. Independence control

This point is fundamental.

Suppose:

```text
LLM A → Source X → conclusion Y

LLM B → Source X → conclusion Y

LLM C → Source X → conclusion Y
```

We do not actually have three pieces of evidence.

We have:

```text
Source X
   ↓
Y
```

Therefore:

```text
Consensus ≠ Independent Evidence
```

The system must track the origin of each piece of evidence.

A recommended structure:

```json
{
  "claim": "Conclusion X",

  "evidence": [
    {
      "source": "source-001",
      "type": "primary",
      "independence_group": "group-001"
    },
    {
      "source": "source-002",
      "type": "secondary",
      "independence_group": "group-002"
    }
  ]
}
```

This allows distinguishing:

```text
3 LLMs
↓
1 source
↓
1 piece of evidence
```

from:

```text
3 LLMs
↓
3 independent sources
↓
3 pieces of evidence
```

---

# 15. Strategy against consensus bias

The algorithm must avoid an important trap:

> The majority is not necessarily right.

Example:

```text
LLM A → X
LLM B → X
LLM C → X
LLM D → Y
```

We must not automatically conclude:

```text
X = true
Y = false
```

The system must ask:

```text
Why did three models conclude X?

Did they use the same sources?

Does primary evidence exist?

And does Y have stronger evidence even if it is a minority?

Are the conditions of X and Y different?
```

That is why the ranking must combine:

```text
consensus
+
evidence
+
independence
+
relevance
+
contradictions
```

---

# 16. Stopping condition strategy

The algorithm must know when to stop.

It is not advisable to research indefinitely.

A termination condition could be:

```text
STOP if:

confidence >= 0.90

AND

no critical contradictions exist

AND

no HIGH/CRITICAL uncertainties exist

AND

the last iteration did not produce relevant new evidence

AND

the expected new research would not significantly change
the conclusion.
```

The following can also be used:

```text
MAX_ITERATIONS = 5
```

as a safety limit.

---

# 17. Summarized algorithm

The complete algorithm can be expressed as:

```text
INPUT:

Question Q
Research runs R1...Rn

FOR each iteration:

    1. Normalize(R1...Rn)

    2. GenerateEmbeddings(results)

    3. SemanticMatch(results)

    4. Cluster(results)

    5. EvaluateEvidence(results)

    6. DetectContradictions(results)

    7. AggregateRankings(RRF)

    8. LLMJudge(
           question,
           clusters,
           evidence,
           contradictions,
           rankings
       )

    9. CalculateConfidence()

    10. DetectUncertainty()

    IF confidence >= threshold
       AND no critical contradictions
       AND no high-impact uncertainty:

        RETURN final_result

    ELSE:

        research_question =
            GenerateTargetedResearch(
                unresolved_uncertainties
            )

        new_research =
            ExecuteResearch(research_question)

        Add new_research to dataset

        CONTINUE
```

---

# 18. Implementation recommendation

For a first version, it is not necessary to implement an excessively complex system.

The initial version can be:

```text
LLM Research A
       +
LLM Research B
       ↓
Normalize JSON
       ↓
Embeddings
       ↓
Cosine Similarity
       ↓
Clustering
       ↓
LLM Evidence Judge
       ↓
LLM Contradiction Judge
       ↓
Weighted Ranking / RRF
       ↓
LLM Synthesis
       ↓
Confidence
       ↓
Targeted Research
```

It can then evolve into:

```text
                  Research Orchestrator
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       LLM #1          LLM #2          LLM #3
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                   Evidence Store
                         │
                  Semantic Index
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Matching       Clustering    Contradictions
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    Rank Engine
                         │
                         ▼
                   Synthesis LLM
                         │
                         ▼
                  Confidence Engine
                         │
                ┌────────┴────────┐
                ▼                 ▼
             Stable          Uncertain
                │                 │
                ▼                 ▼
              FINAL        Research Planner
                                  │
                                  ▼
                            New Research
                                  │
                                  └───────────↺
```

---

# 19. Fundamental principle of the system

The system must not treat the research runs as answers that should simply be averaged.

It must treat them as **independent sources of evidence that can confirm, complement, refute, or challenge a hypothesis**.

The core logic must be:

```text
             RESEARCH
                   ↓
              OBSERVATIONS
                   ↓
              CONCLUSIONS
                   ↓
          SEMANTIC MATCHING
                   ↓
               CLUSTERS
                   ↓
        ┌──────────┴───────────┐
        │                      │
     CONSENSUS             CONTRADICTION
        │                      │
        └──────────┬───────────┘
                   ↓
              EVIDENCE
                   ↓
              RANKING
                   ↓
             SYNTHESIS
                   ↓
             CONFIDENCE
                   ↓
           UNCERTAINTY
                   ↓
        NEW RESEARCH
                   ↓
                 LOOP
```

The most important characteristic of this design is that **the result of a single research run is not considered definitive**.

Each research run is a new piece of evidence.

The final result is the conclusion that remains after subjecting the hypotheses to:

```text
        multiple research runs
                 +
        semantic comparison
                 +
             consensus
                 +
          independent evidence
                 +
          contradiction detection
                 +
             ranking
                 +
              synthesis
                 +
        additional research
```

Therefore, the system can be seen as a process of **iterative knowledge refinement**:

```text
Knowledge₀
    ↓
Research
    ↓
Knowledge₁
    ↓
Research driven by uncertainty
    ↓
Knowledge₂
    ↓
Research
    ↓
Knowledge₃
    ↓
...
    ↓
Knowledgeₙ
```

Where each iteration seeks that:

```text
Knowledge(n+1) > Knowledge(n)
```

not necessarily in the amount of information, but in:

* precision
* evidence
* coverage
* consistency
* independence
* reduction of uncertainty
* ability to explain contradictions
* confidence in the conclusion

The ultimate goal is not to produce **more text**, but to produce **a progressively better-supported conclusion**.
