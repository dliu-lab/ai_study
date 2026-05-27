# MEMO Study Notes

Source paper: `/Users/dliu520/Downloads/Memo——2605.15156v2.pdf`

Paper PDF: [https://arxiv.org/pdf/2605.15156](https://arxiv.org/pdf/2605.15156)

Supplementary video: [MEMO: Memory as a Model (May 2026)](https://www.youtube.com/watch?v=CwH8K6fWtgg) by AI Paper Slop, uploaded May 20, 2026.

Paper: **MEMO: Memory as a Model**  
arXiv: `2605.15156v2`, dated May 20, 2026 in the PDF metadata.

## How to Use These Notes

Read in this order:

1. `01_full_analysis.md` - the paper in plain English, with the main thesis and contribution map.
2. `02_method_deep_dive.md` - the technical details of the data synthesis pipeline, MEMORY model training, and inference protocol.
3. `03_experiments_and_ablations.md` - benchmark setup, results, ablations, and what the numbers actually mean.
4. `04_critique_limitations_questions.md` - strengths, weaknesses, suspicious points, and research questions.
5. `05_flashcards_and_quiz.md` - active recall material for studying.
6. `06_youtube_analysis_notes.md` - notes from the linked YouTube explanation, with paper cross-checks.

## One-Sentence Summary

MEMO stores new knowledge not in a vector database or in the main LLM itself, but in a separate fine-tuned **MEMORY model** that a frozen **EXECUTIVE model** queries through a structured multi-turn protocol.

## Core Mental Model

Think of MEMO as replacing:

- "Search documents, paste chunks into the prompt"  
  with
- "Train a small expert model on a corpus, then ask that expert targeted questions."

The EXECUTIVE model does the reasoning. The MEMORY model supplies compact, corpus-derived answers. The target corpus is not retrieved at inference time.

## Key Terms

- **EXECUTIVE model**: the main frozen LLM that answers the user.
- **MEMORY model**: a smaller model fine-tuned to internalize a target corpus.
- **GENERATOR model**: an LLM used offline to synthesize the reflection QA dataset.
- **Reflection QA pairs**: generated question-answer pairs that expose facts, entities, and cross-document relationships from the corpus.
- **Entity surfacing**: training the MEMORY model to identify an entity from descriptions and relationships.
- **Cross-document synthesis**: generated examples that combine evidence across multiple documents.
- **Structured multi-turn protocol**: inference-time process where the EXECUTIVE model decomposes a query, identifies entities, gathers facts, and synthesizes the final answer.

## The Big Trade-Off

MEMO buys stronger cross-document reasoning, black-box compatibility, and inference cost independent of corpus size, but pays a large upfront cost in data generation and MEMORY model training.
