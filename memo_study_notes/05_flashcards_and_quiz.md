# MEMO Flashcards and Quiz

## Flashcards

Q: What does MEMO stand for?  
A: Memory as a Model.

Q: What is the core idea of MEMO?  
A: Train a separate MEMORY model on a corpus-derived reflection QA dataset, then let a frozen EXECUTIVE model query it through natural language.

Q: What problem is MEMO trying to solve?  
A: Adding timely or domain-specific knowledge to LLMs without retraining the main model or relying on noisy retrieval.

Q: What are the three main model roles in MEMO?  
A: GENERATOR, MEMORY, and EXECUTIVE.

Q: What does the GENERATOR model do?  
A: Converts the target corpus into reflection QA pairs during offline data synthesis.

Q: What does the MEMORY model do?  
A: Stores corpus-derived knowledge in its parameters and answers targeted questions from the EXECUTIVE model.

Q: What does the EXECUTIVE model do?  
A: Decomposes the user query, queries the MEMORY model, tracks intermediate facts, and synthesizes the final answer.

Q: Why can MEMO work with black-box LLMs?  
A: The EXECUTIVE and MEMORY models communicate through natural language rather than internal weights, logits, or hidden states.

Q: How is MEMO different from RAG?  
A: RAG retrieves raw documents at inference time; MEMO trains a model to internalize corpus knowledge and queries that model at inference.

Q: What are reflections?  
A: Corpus-derived QA pairs that expose direct facts, inferred facts, entities, and cross-document relationships.

Q: What is Step 1 of the data synthesis pipeline?  
A: Fact extraction, including direct and indirect QA generation.

Q: What is Step 2 of the data synthesis pipeline?  
A: Consolidation of related QA pairs into multi-fact examples.

Q: What is Step 3 of the data synthesis pipeline?  
A: Verification and rewriting for self-contained QA pairs.

Q: What is Step 4 of the data synthesis pipeline?  
A: Entity surfacing, where clues and relationships are used to identify named entities.

Q: What is Step 5 of the data synthesis pipeline?  
A: Cross-document synthesis.

Q: Which synthesis step is most critical according to the ablation?  
A: Step 5, cross-document synthesis.

Q: Why does entity surfacing matter?  
A: It helps the MEMORY model identify entities from descriptions and supports Stage 2 of inference.

Q: What are the three inference stages?  
A: Grounding, entity identification, and answer seeking/synthesis.

Q: What happens in Stage 1, grounding?  
A: The EXECUTIVE decomposes the user query into clue-probing sub-questions.

Q: What happens in Stage 2, entity identification?  
A: The EXECUTIVE narrows candidate entities using follow-up questions to the MEMORY model.

Q: What happens in Stage 3, answer seeking and synthesis?  
A: The EXECUTIVE gathers supporting facts and produces the final answer.

Q: Why does structured multi-turn querying beat single-turn querying?  
A: It lets the EXECUTIVE adapt based on MEMORY responses and maintain state over candidates and facts.

Q: What is the main advantage of MEMO over latent memory methods?  
A: MEMO is cross-model transferable because its interface is natural language.

Q: What is the main advantage of MEMO over fine-tuning the EXECUTIVE model?  
A: It avoids modifying the EXECUTIVE model and reduces catastrophic forgetting risk.

Q: What is the main advantage of MEMO over RAG?  
A: It is more robust to retrieval noise and can encode cross-document relationships during training.

Q: What is the biggest practical disadvantage of MEMO?  
A: High upfront cost for data generation and MEMORY model training.

Q: Why is source citation harder in MEMO than RAG?  
A: The MEMORY model answers from parameters rather than returning raw source chunks.

Q: What does the noise ablation show?  
A: Retrieval systems degrade when distractor documents are added, while MEMO remains relatively stable.

Q: How does a stronger EXECUTIVE model affect MEMO?  
A: It improves performance, showing that MEMORY can be reused with better reasoners.

Q: What does the MEMORY size ablation show?  
A: Larger MEMORY models generally improve performance.

Q: What does the MEMORY family ablation show?  
A: MEMO works across different model families, though performance varies.

Q: What does the LoRA vs full SFT ablation show?  
A: Full SFT generally beats LoRA, and LoRA can fail badly if target modules do not match the architecture.

Q: What is model merging used for in MEMO?  
A: Continual integration of multiple corpora without full retraining on the union.

Q: What is a task vector?  
A: The difference between a fine-tuned MEMORY model's parameters and the base model's parameters.

Q: What is the main model merging trade-off?  
A: Lower cumulative compute but lower accuracy than full retraining.

## Short Quiz

1. Explain MEMO in two sentences.

2. Why is MEMO not just another RAG method?

3. What are the five steps of the reflection data pipeline?

4. Why is cross-document synthesis so important?

5. What does the MEMORY model see at inference time?

6. Why does MEMO avoid catastrophic forgetting in the EXECUTIVE model?

7. What are the three stages of inference?

8. Why might the structured entity-identification stage help BrowseComp-Plus more than NarrativeQA?

9. What evidence supports the plug-and-play claim?

10. What is the strongest criticism of MEMO?

## Answer Key

1. MEMO trains a separate MEMORY model on synthetic QA reflections from a target corpus. A frozen EXECUTIVE model queries that MEMORY model through a structured multi-turn protocol and synthesizes the final answer.

2. RAG retrieves raw text chunks at inference time. MEMO stores corpus knowledge parametrically in a MEMORY model and queries that model instead of searching the corpus.

3. Fact extraction, consolidation, verification/rewriting, entity surfacing, and cross-document synthesis.

4. It teaches the MEMORY model relationships distributed across multiple documents, which is exactly where simple retrieval often struggles.

5. Only the questions from the EXECUTIVE model and its own generated answer prefix; it does not see source documents.

6. The EXECUTIVE model's parameters are never updated.

7. Grounding, entity identification, and answer seeking/synthesis.

8. BrowseComp-Plus often depends on identifying entities from clues, while NarrativeQA requires broader narrative comprehension where entity pinning may use up interaction budget.

9. The same MEMORY model performs much better when paired with Gemini-3-Flash instead of Qwen2.5-32B-Instruct as EXECUTIVE.

10. MEMO is expensive to build and depends heavily on the quality of generated reflection QA pairs, while also making source attribution harder.

## Explain-It-Back Prompts

Use these to test whether you really understand the paper:

1. "MEMO moves memory from retrieval space into parameter space. What does that mean?"
2. "Why is natural language an important interface between MEMORY and EXECUTIVE?"
3. "What exactly is lost when Step 5 is removed?"
4. "Why does MEMO become better when the EXECUTIVE model becomes stronger?"
5. "How would you design a MEMO + RAG hybrid?"
6. "What kinds of corpora are good or bad fits for MEMO?"
7. "If the MEMORY model gives a wrong answer, where could the error have entered the pipeline?"

