# MetaGPT Attack POC - Usage Guide

This guide explains how to use the MetaGPT Memory Poisoning Attack proof-of-concept.

## Experiment Details

### Experiment 1: Schema-Spoofing Attack

**Objective**: Test if agents confuse external content (README files) with internal reasoning

**How it works**:
1. Creates a test repository with a poisoned README
2. README contains ReAct-format markers (Thought/Action/Observation)
3. Asks agent to analyze the repository
4. Monitors if agent internalizes the malicious patterns

**Expected outcome**: Agent may treat README content as valid reasoning patterns

**Runtime**: ~30-60 seconds

### Experiment 2: JudgeJacking (Rubric-Mimicry) Attack

**Objective**: Test if agents can be fooled by fake success markers

**How it works**:
1. Uses fake_success_script.py that prints success markers
2. Script does NO actual work, only outputs success indicators
3. Asks agent to run the script and validate results
4. Monitors if agent believes the fake output

**Expected outcome**: Agent validates script as successful despite no real work

**Runtime**: ~30-60 seconds

### Experiment 3: Memory Graft (Vector-Drift Poisoning) Attack

**Objective**: Test if poisoned memories persist and influence future behavior

**How it works**:
1. Injects poisoned experiences into agent memory
2. Tests retrieval with semantically similar queries
3. Monitors if poisoned content is retrieved
4. Verifies persistence across "sessions"

**Expected outcome**: Poisoned memories retrieved for similar queries, causing behavior drift

**Runtime**: ~10-20 seconds (no LLM calls, pure memory operations)

### Experiment 4: RAG Vector-Drift Poisoning (New)

**Objective**: Poison MetaGPT’s RAG vector store so semantically similar clean queries retrieve malicious “successes.”

**How it works**:
1. Loads and augments seeds to at least 100 benign + 10 poisoned experiences (see `payloads/experience_seeds.json`; augmented copy written alongside).
2. Builds a RAG index: FAISS (top-3) if an embedding key is configured; otherwise BM25 (top-3).
3. Issues 10 diverse semantic queries (CI, hotfix, config, canary, data, mobile, review bypass, etc.) and measures PRR.
4. Persists the poisoned index to `results/rag_poison_store` and saves a JSON report.

**Expected outcome**: Poisoned entries surface for semantically close queries; PRR > 0 indicates successful drift.

**Runtime**: ~10-30 seconds (no LLM calls; local indexing)

## Running the Experiments

```bash
# Experiment 1: Schema-Spoofing
python experiments/exp1_schema_spoof.py

# Experiment 2: JudgeJacking
python experiments/exp2_judge_jack.py

# Experiment 3: Memory Graft (Experience Pool)
python experiments/exp3_memory_graft.py

# Experiment 4: RAG Vector-Drift Poisoning
# Uses BM25 by default; set a valid embedding key in config for FAISS (top-3) vector poisoning.
python experiments/exp4_rag_vector_drift.py
```

> Tip: Install RAG extras first if you haven’t: `pip install -e .[rag]` (or `pip install metagpt[rag]`).

## Understanding Results

### Results Directory

All experiment outputs are saved to `metagpt_attack_poc/results/`:

```
results/
├── exp1_schema_spoof_YYYYMMDD_HHMMSS.txt
├── exp2_judgejack_YYYYMMDD_HHMMSS.txt
├── exp3_memory_graft_YYYYMMDD_HHMMSS.txt
├── exp4_rag_vector_drift_YYYYMMDD_HHMMSS.json
├── rag_poison_store/                # Persisted poisoned RAG index
├── retrieval_log_YYYYMMDD_HHMMSS.jsonl
├── retrieval_analysis_YYYYMMDD_HHMMSS.txt
└── memory_inspection.txt
```

## Payload Files

### poisoned_readme.md

A README file containing:
- ReAct-format reasoning patterns (Thought/Action/Observation)
- Dangerous practices framed as "best practices"
- Success metrics that make bad patterns look good

**Use case**: Schema-spoofing attacks on repository analysis

### fake_success_script.py

A Python script that:
- Prints realistic success markers
- Mimics pytest/unittest output
- Returns success exit code (0)
- Does NO actual work

**Use case**: JudgeJacking attacks to fool validation

### experience_seeds.json

JSON file containing:
- 6 poisoned "successful" experiences
- Dangerous patterns (skip tests, bypass validation, etc.)
- Realistic metadata and success metrics
- Semantic clusters for retrieval targeting

**Use case**: Memory graft attacks to poison experience pool
