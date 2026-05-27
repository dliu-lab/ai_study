# MEMO Experiments and Ablations

## 1. Benchmarks

## BrowseComp-Plus

Task type: deep research, multi-hop, multi-document reasoning.

Setup:

- 300 questions,
- 1,775 evidence documents,
- 1,766 negative documents,
- 3,541 total documents.

This benchmark strongly rewards access to correct evidence documents. The EXECUTIVE model has near-zero no-context performance, so the knowledge is mostly unavailable from parametric memory alone.

## NarrativeQA

Task type: long-document narrative understanding over books and movie scripts.

Setup:

- 293 questions,
- 10 long source documents in the evaluated subset,
- long documents are chunked with sliding windows.

This is the hardest benchmark in the paper. Even perfect retrieval is not extremely high, which means the bottleneck is not just finding evidence but reasoning over long narrative structure.

## MuSiQue

Task type: multi-hop questions over Wikipedia paragraphs.

Setup:

- 1,000 questions,
- 2,648 evidence documents,
- 2,648 negative documents,
- 5,296 total documents.

No-context performance is higher here because some Wikipedia facts may already be in the EXECUTIVE model.

## 2. Baselines

The paper compares MEMO against:

- **BM25**: lexical retrieval.
- **NV-Embed-V2**: dense retrieval.
- **HippoRAG2**: graph-based RAG baseline.
- **Cartridges**: trained KV-cache baseline requiring white-box access.
- **Perfect Retrieval**: empirical upper bound where the EXECUTIVE model receives only evidence documents.

Perfect Retrieval is not a real deployment method in this setting; it estimates how well the EXECUTIVE model could do if retrieval were solved.

## 3. Main Results With Qwen2.5-32B-Instruct as EXECUTIVE

| Method | BrowseComp-Plus | NarrativeQA | MuSiQue |
|---|---:|---:|---:|
| Perfect Retrieval | 79.67 | 51.42 | 62.83 |
| BM25 | 1.11 | 10.24 | 20.00 |
| NV-Embed-V2 | 50.67 | 20.59 | 37.47 |
| HippoRAG2 | 56.11 | 21.39 | 42.17 |
| Cartridges | 0.00 | 3.75 | 8.57 |
| MEMO | 54.22 | 26.85 | 48.30 |

Interpretation:

- MEMO beats retrieval baselines on NarrativeQA and MuSiQue.
- MEMO is competitive on BrowseComp-Plus but slightly trails HippoRAG2 with Qwen EXECUTIVE.
- Perfect Retrieval remains much higher, so there is still large headroom.

## 4. Main Results With Gemini-3-Flash as EXECUTIVE

| Method | BrowseComp-Plus | NarrativeQA | MuSiQue |
|---|---:|---:|---:|
| Perfect Retrieval | 88.33 | 60.41 | 73.00 |
| BM25 | 27.00 | 14.33 | 23.20 |
| NV-Embed-V2 | 57.00 | 26.62 | 46.60 |
| HippoRAG2 | 66.33 | 23.21 | 57.00 |
| MEMO | 66.67 | 53.58 | 60.20 |

Interpretation:

- MEMO benefits strongly from a better EXECUTIVE model.
- The same MEMORY model paired with Gemini-3-Flash becomes especially strong on NarrativeQA.
- This supports the plug-and-play claim.

## 5. What the Main Results Actually Show

The paper does not show that MEMO always beats RAG. It shows a more specific claim:

MEMO is strong when questions require multi-step synthesis or when retrieval noise hurts retrieval systems.

On BrowseComp-Plus, raw evidence access is very valuable, so a strong RAG system can be highly competitive. On NarrativeQA and MuSiQue, synthesized memory plus structured querying performs better.

## 6. Noise Ablation

The authors vary how many negative documents are added.

With Qwen2.5-32B-Instruct as EXECUTIVE:

| Method | Dataset | No distractors | 1x evidence-count distractors | Change |
|---|---|---:|---:|---:|
| NV-Embed-V2 | BrowseComp-Plus | 56.89 | 50.67 | -6.22 |
| NV-Embed-V2 | MuSiQue | 42.30 | 37.47 | -4.83 |
| HippoRAG2 | BrowseComp-Plus | 62.33 | 56.11 | -6.22 |
| HippoRAG2 | MuSiQue | 47.33 | 42.17 | -5.16 |
| MEMO | BrowseComp-Plus | 53.67 | 54.22 | +0.55 |
| MEMO | MuSiQue | 50.07 | 48.30 | -1.77 |

Interpretation:

- Retrieval baselines degrade when irrelevant documents are added.
- MEMO is much more stable because it does not retrieve raw documents at inference time.
- However, MEMO can still learn noise during training if negative documents pollute generated reflections.

Appendix L extends retrieval noise to `2N` and shows monotonic degradation for retrieval methods.

## 7. MEMORY Model Size Ablation

The paper compares Qwen2.5-1.5B-Instruct and Qwen2.5-14B-Instruct as MEMORY models.

With Qwen2.5-32B-Instruct as EXECUTIVE:

| MEMORY model | BrowseComp-Plus | NarrativeQA | MuSiQue |
|---|---:|---:|---:|
| Qwen2.5-1.5B | 44.11 | 24.00 | 42.90 |
| Qwen2.5-14B | 54.22 | 26.85 | 48.30 |

With Gemini-3-Flash as EXECUTIVE:

| MEMORY model | BrowseComp-Plus | NarrativeQA | MuSiQue |
|---|---:|---:|---:|
| Qwen2.5-1.5B | 61.00 | 47.44 | 59.70 |
| Qwen2.5-14B | 66.67 | 53.58 | 60.20 |

Interpretation:

- Larger MEMORY models generally help.
- The gain depends on the task and EXECUTIVE model.
- On MuSiQue with Gemini, the gap is small, suggesting the EXECUTIVE model can compensate when the task has more familiar factual structure.

## 8. MEMORY Model Family Ablation

The paper compares similarly sized MEMORY models:

- Qwen2.5-1.5B-Instruct,
- Gemma3-1B-IT,
- LFM2.5-1.2B-Instruct.

Main interpretation:

- The framework is not tied to one architecture.
- Qwen is usually strongest, but Gemma and LFM remain competitive.
- LFM performs especially well on MuSiQue with Qwen EXECUTIVE.

This supports the claim that MEMO is a framework, not a one-model trick.

## 9. Data Pipeline Leave-One-Out Ablation

Appendix E removes one data synthesis step at a time and retrains Qwen2.5-1.5B MEMORY.

Key results:

| Ablation | NarrativeQA | MuSiQue | Meaning |
|---|---:|---:|---|
| Baseline, all steps | 24.00 | 42.90 | Full pipeline |
| Step 1a removed | 20.48 | 30.00 | Direct extraction matters |
| Step 1b removed | 22.98 | 37.33 | Indirect extraction matters |
| Step 2 removed | 24.69 | 37.10 | Consolidation helps MuSiQue, may hurt NarrativeQA |
| Step 3 removed | 28.90 | 41.70 | Verification helps MuSiQue but can corrupt narratives |
| Step 4 removed | 23.21 | 39.10 | Entity surfacing matters |
| Step 5 removed | 6.37 | 24.17 | Cross-document synthesis is critical |

The most important finding: **Step 5 is the backbone of MEMO**.

Without cross-document synthesis, MEMO loses much of the advantage over normal QA training.

## 10. Structured Multi-Turn Ablation

Appendix J compares inference setups:

| Evaluation setup | BrowseComp-Plus | NarrativeQA | MuSiQue |
|---|---:|---:|---:|
| Single turn | 32.56 | 24.80 | 37.57 |
| Unstructured multi-turn, 15 turns | 47.33 | 26.73 | 40.13 |
| Unstructured multi-turn, 50 turns | 48.67 | 27.19 | 40.57 |
| Structured, 7 entity + 8 answer turns | 54.22 | 26.39 | 48.30 |
| Structured, 7 entity + 15 answer turns | 51.44 | 27.76 | 47.57 |

Interpretation:

- Single-turn querying is weak.
- Unstructured multi-turn helps but plateaus.
- Structured multi-turn is best for BrowseComp-Plus and MuSiQue.
- NarrativeQA benefits less from entity identification and more from answer-seeking turns.

This suggests the protocol should be adapted to task type.

## 11. Epoch and Overfitting Discussion

The paper observes that more epochs do not consistently improve accuracy. Performance often peaks around epoch 2.

The authors attribute this to lexical overlap in the generated QA dataset. Later synthesis steps derive from earlier steps, so many examples are redundant.

Compression ratios:

- BrowseComp-Plus: 5.80x,
- MuSiQue: 7.03x,
- NarrativeQA: 5.45x.

Higher compression means lots of repeated surface material. That creates overfitting risk.

## 12. Full SFT vs LoRA

Full SFT generally beats LoRA.

Examples with Qwen EXECUTIVE:

| MEMORY model | BrowseComp LoRA | BrowseComp Full SFT | MuSiQue LoRA | MuSiQue Full SFT |
|---|---:|---:|---:|---:|
| Qwen2.5-1.5B | 29.78 | 44.11 | 31.53 | 42.90 |
| Qwen2.5-14B | 48.78 | 54.22 | 43.94 | 50.07 |
| LFM2.5-1.2B | 0.78 | 37.33 | 7.50 | 45.23 |

The LFM LoRA result is especially poor because the LoRA configuration targets standard transformer module names that do not cover key LFM-specific layers.

Lesson: parameter-efficient tuning is architecture-sensitive.

## 13. Model Merging Result

The paper tests a streaming update setting on NarrativeQA.

Setup:

- split NarrativeQA into two disjoint subsets,
- train separate MEMORY models,
- merge task vectors,
- compare against full retraining on the union.

Main reported result:

| Method | Compute | Qwen EXECUTIVE | Gemini EXECUTIVE |
|---|---:|---:|---:|
| Full retrain | about 72 GPU-hours | 26.85 | 53.58 |
| Merge-TIES, rho=0.3 | about 48 GPU-hours | 15.81 | 34.47 |

Interpretation:

- Merging saves about 33% compute for two corpora.
- Accuracy drops a lot.
- The compute savings grow with more corpora.

Important critical note: the text claims the merged model still beats every retrieval baseline on NarrativeQA, but Table 2 and Table 6 appear to conflict for Qwen EXECUTIVE. The merged Qwen result is 15.81, while NV-Embed-V2 is 20.59 and HippoRAG2 is 21.39 on NarrativeQA. The claim is true for the Gemini EXECUTIVE numbers, but not for Qwen if the tables are compared directly.

