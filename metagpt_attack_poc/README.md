# MetaGPT Memory Poisoning Attack - Proof of Concept

This POC demonstrates three novel attack primitives against MetaGPT's memory-based agent system:

1. **Schema-Spoofing**: Injecting fake format markers into tool outputs
2. **Rubric-Mimicry (JudgeJacking)**: Bypassing success validation with synthetic markers
3. **Vector-Drift Poisoning (Memory Graft)**: Persisting malicious patterns through memory retrieval
4. **RAG Vector-Drift Poisoning**: Poisoning MetaGPT’s RAG vector store so retrieval surfaces malicious “successes”

## Attack Novelty

**Persistent compromise via memory graft** - Unlike traditional prompt injection (one-shot), these attacks:
- Poison the agent's memory system
- Persist across sessions via Redis/storage
- Propagate through vector retrieval
- Cause permanent behavior drift without repeated injection

## Architecture

```
Tool Output → LLM Judge/Self-Reflection → "Success" Memory Write → Future Retrieval → Behavior Drift
     ↑                    ↑                         ↑                      ↑
  Attack 1           Attack 2                  Attack 3              Attack Result
```

## Directory Structure

```
metagpt_attack_poc/
├── payloads/               # Attack payloads
│   ├── poisoned_readme.md
│   ├── rag_poisoned_notes.md
│   ├── fake_success_script.py
│   └── experience_seeds.json
├── experiments/            # POC test scripts
│   ├── exp1_schema_spoof.py
│   ├── exp2_judge_jack.py
│   ├── exp3_memory_graft.py
│   └── exp4_rag_vector_drift.py
├── monitors/              # Monitoring utilities
│   ├── memory_inspector.py
│   └── retrieval_logger.py
├── results/               # Experiment outputs
└── README.md

## RAG Vector-Drift Poisoning (New)

**Goal:** Poison MetaGPT’s RAG vector store so semantically similar clean queries retrieve attacker-crafted “successes” and bias the agent’s behavior.

**What it does:**
- Loads an expanded seed set (100 benign + 10 poisoned experiences) from `payloads/experience_seeds.json` (and writes augmented seeds for inspection).
- Builds a RAG index (FAISS top-3 if embedding is configured; otherwise BM25 top-3).
- Issues 10 diverse semantic queries and computes Poisoned Retrieval Rate (PRR).
- Persists a poisoned RAG store (`results/rag_poison_store`) and a JSON report.

**Why it matters:** Retrieval is often trusted as prior “wins.” Seeding the vector store with poisoned “successes” makes drift persistent across sessions and agents that consume the shared store.

**How to run (summary):**
```bash
# Ensure embedding config if you want FAISS; BM25 is used otherwise.
python3 metagpt_attack_poc/experiments/exp4_rag_vector_drift.py
```
```

## Quick Start

### Setup
```bash
# From MetaGPT root directory
cd metagpt_attack_poc
python -m pip install -r requirements.txt
```

### Run Experiments

```bash
# Experiment 1: Schema-Spoofing
python experiments/exp1_schema_spoof.py

# Experiment 2: JudgeJacking
python experiments/exp2_judge_jack.py

# Experiment 3: Memory Graft
python experiments/exp3_memory_graft.py
```
