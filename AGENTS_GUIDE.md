# Antigravity Multi-Agent Collaboration Guide

This guide contains the **exact instructions and prompt templates** for each of the 3 team members to paste into their respective **Google Antigravity IDE** sessions.

---

## Team Setup & Git Branches

| Member | Focus Area | Branch Name | Working Files |
| :--- | :--- | :--- | :--- |
| **Person 1** | Preprocessing & Normalization | `feature/preprocessing` | `src/preprocessing/normalize.py`<br>`tests/test_preprocessing.py` |
| **Person 2** | Blocking & Candidate Generation | `feature/blocking` | `src/blocking/blocker.py`<br>`tests/test_blocking.py` |
| **Person 3** | Matching Model & Evaluation | `feature/matching` | `src/matching/matcher.py`<br>`src/evaluation/metrics.py`<br>`tests/test_matching.py` |

---

## Prompt Template: Person 1 (Preprocessing & Normalization)

> **Instructions for Person 1**:
> 1. Switch to your branch: `git checkout -b feature/preprocessing`
> 2. Open Antigravity and copy-paste the prompt below into the chat:

```markdown
Hello Antigravity. I am Person 1 on a 3-person team competing in the Amazon ML Challenge 2026 for Business Entity Resolution.

My sole responsibility is Stage 1: Preprocessing & Normalization.
Read docs/pipeline_contract.md carefully. My code must strictly adhere to the contract schema for Stage 1.

Key Requirements:
1. Input: Raw TSVs (entity_id, business_name, business_address, country) located in dataset/ or tests/data/raw/.
2. Output: normalized_source1.tsv, normalized_source2.tsv, normalized_source3.tsv.
   Columns: entity_id, business_name_clean, business_address_clean, country, postal_code, name_tokens.
3. Clean business names: lowercase, strip punctuation, expand/normalize abbreviations (pvt ltd -> private limited, inc -> incorporated, corp -> corporation, etc.), handle special characters.
4. Clean addresses: lowercase, standardize road/street/ave/suite/apt, extract postal/PIN codes (US ZIP codes, Indian 6-digit PIN codes, French postal codes).
5. Extract space-separated name_tokens excluding generic stop words and legal terms.
6. Open set country rule: Never hardcode or drop countries; must support US, India, and France.
7. Validation: Use src/schemas/contracts.py to validate your output files.
8. Implement src/preprocessing/normalize.py with CLI arguments:
   python -m src.preprocessing.normalize --input-dir tests/data/raw --output-dir tests/data/normalized
```

---

## Prompt Template: Person 2 (Blocking & Candidate Generation)

> **Instructions for Person 2**:
> 1. Switch to your branch: `git checkout -b feature/blocking`
> 2. Open Antigravity and copy-paste the prompt below into the chat:

```markdown
Hello Antigravity. I am Person 2 on a 3-person team competing in the Amazon ML Challenge 2026 for Business Entity Resolution.

My sole responsibility is Stage 2: Blocking & Candidate Generation.
Read docs/pipeline_contract.md carefully. My code must strictly adhere to the contract schema for Stage 2.

Key Requirements:
1. Input: Normalized files produced by Stage 1 (tests/data/normalized/normalized_source*.tsv).
   Schema: entity_id, business_name_clean, business_address_clean, country, postal_code, name_tokens.
2. Output: output/candidate_pairs.tsv.
   Schema: source1_entity_id\tcandidate_entity_ids (tab-separated, comma-separated candidate IDs).
3. Blocking Strategy:
   - Country blocking: Only match within same country (India to India, US to US, France to France).
   - Multi-key indexing:
     * Postal code exact match + first name token
     * Shared name tokens / n-gram TF-IDF similarity
     * Phonetic / Soundex key matching
   - Aim for high recall (>95% of true matches retained) while keeping candidate set small (avg 20-50 candidates per S1 entity).
4. Formatting rules (strictly enforced by validate_submission.py):
   - Every Source 1 entity in the evaluated set must have exactly one row.
   - Candidate IDs must ONLY have prefix S2- or S3- (no S1- self matches).
   - No duplicate IDs within a row.
   - Comma-separated with NO spaces (e.g. S2-00047,S3-00812).
   - Singletons must have an empty candidate_entity_ids.
5. Provide both a file generator and streaming iterator:
   - CLI: python -m src.blocking.blocker --normalized-dir tests/data/normalized --output output/candidate_pairs.tsv
   - Python helper: iterate_candidate_pairs(candidates_path) yielding (s1_id, cand_id).
6. Validate using:
   python -m src.schemas.contracts --file output/candidate_pairs.tsv --type candidate
```

---

## Prompt Template: Person 3 (Matching Model & Leaderboard Submission)

> **Instructions for Person 3**:
> 1. Switch to your branch: `git checkout -b feature/matching`
> 2. Open Antigravity and copy-paste the prompt below into the chat:

```markdown
Hello Antigravity. I am Person 3 on a 3-person team competing in the Amazon ML Challenge 2026 for Business Entity Resolution.

My sole responsibility is Stage 3: Matching Model, Evaluation & Leaderboard Submission.
Read docs/pipeline_contract.md carefully. My code must strictly adhere to the Stage 3 contract and official challenge rules.

Key Requirements:
1. Input:
   - Candidates from Stage 2: output/candidate_pairs.tsv (or tests/data/blocking/candidate_pairs_sample.tsv)
   - Normalized records from Stage 1: normalized_source*.tsv
   - Ground truth for training: dataset/train/train_ground_truth.tsv
2. Feature Engineering on Candidate Pairs:
   - Name similarities: Token Jaccard, character n-gram cosine, Levenshtein ratio, token containment.
   - Address similarities: Address token overlap, postal code match/mismatch flag, house/building number match.
   - Exact match indicators.
3. Classification & Optimization:
   - Train a fast binary classifier (e.g., LightGBM / XGBoost / LogisticRegression).
   - Optimize decision threshold for Macro F_0.5 (Precision is weighted 2x over Recall: beta = 0.5).
   - False positives are heavily penalized! Prefer predicting fewer high-confidence matches over risky merges.
4. Output: output/matching_results.tsv.
   - Schema: source1_entity_id\tmatched_entity_ids (tab-separated, comma-separated matched IDs).
   - Singletons must have empty matched_entity_ids.
   - SUBSET RULE: Every matched ID MUST appear in candidate_pairs.tsv for that entity.
5. Validation:
   - Offline F_0.5 score: python -m src.evaluation.metrics --ground-truth tests/data/raw/train_ground_truth_sample.tsv --predictions output/matching_results.tsv
   - Challenge validator: python 6ab10eb3b23ba_student_resource/student_resource/utils/validate_submission.py --matching output/matching_results.tsv --candidate output/candidate_pairs.tsv --test-dir tests/data/raw
```

---

## Git Pull Request & Integration Checklist

When any teammate opens a Pull Request into `main`:

- [ ] **Contract Integrity**: Does the code change any column name or delimiter in `src/schemas/contracts.py`? (If yes, reject until agreed by all).
- [ ] **Tests Pass**: Run `python -m unittest discover tests` on sample fixtures.
- [ ] **No Big Files**: Ensure no `*.tsv` files larger than 5MB or raw datasets are staged in git.
- [ ] **Challenge Validator**: Ensure `utils/validate_submission.py` outputs `PASS`.
