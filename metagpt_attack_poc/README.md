# MetaGPT Memory Poisoning Attack - Proof of Concept

This POC demonstrates three attack primitives against MetaGPT's memory-based agent system:

1. **Schema-Spoofing**: Inject forged ReAct-style traces via documentation.
2. **Rubric-Mimicry (JudgeJacking)**: Fool success validation with synthetic markers.
3. **Memory Graft**: Poison the agent’s retrieval store so “successful” but unsafe patterns persist and surface on future tasks (implemented by `exp4`).

## Attack Novelty

**Persistent compromise via memory graft** — Unlike one-shot prompt injections, these attacks:
- Poison the agent's memory system (experience/RAG stores).
- Persist across sessions via on-disk storage.
- Propagate through retrieval on semantically similar tasks.
- Cause lasting behavior drift without repeated injection.

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
│   └── exp4_rag_vector_drift.py   # Memory Graft attack (current)
├── monitors/               # Monitoring utilities
│   └── memory_inspector.py
├── results/                # Experiment outputs
└── README.md
```

## Memory Graft Attack (current, exp4)

**Goal:** Poison MetaGPT’s retrieval store so clean, semantically similar queries pull attacker-crafted “successes,” biasing the agent toward unsafe shortcuts.

**What it does:**
- Loads 100 benign + 10 poisoned experiences from `payloads/experience_seeds.json`.
- Builds a retrieval index (BM25; adds FAISS if embeddings are configured) and persists it to `results/rag_poison_store`.
- Prompts DataInterpreter to read `payloads/rag_poisoned_notes.md` (which contains the runnable poisoning block).
- Runs a suite of DI-style queries and computes Poisoned Retrieval Rate (PRR), logging per-query poisoned hits and agent retrieval outputs.
- Saves a timestamped JSON report under `results/`.

**Why it matters:** Retrieval is treated as prior “wins.” Seeding the store with poisoned “successes” makes drift persistent across sessions and agents sharing the store.

**How to run (summary):**
```bash
# Ensure embedding config if you want FAISS; BM25 is used otherwise.
python3 metagpt_attack_poc/experiments/exp4_rag_vector_drift.py
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

# Memory Graft (current)
python experiments/exp4_rag_vector_drift.py
```
