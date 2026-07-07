# DOMAUDIT: Signature-Construction Analysis for EIP-712 Replay Risk

DOMAUDIT is the artifact for our ICSE submission on **signature-construction correctness** in smart contracts. The tool analyzes verified Solidity source code and checks whether EIP-712 / permit-style authorization logic correctly binds signatures to the intended authority context: domain, chain, verifier, freshness state, and logical protocol subdomain.

The repository contains the analyzer implementation, scripts used for result aggregation, generated evaluation data, manual-validation samples, and figures used in the paper.

## Repository Structure

```text
DOMAUDIT/
├── python/
│   ├── domain_separator_analyzer_corrected.py
│   └── helper scripts for aggregation / sampling
├── JSON/
│   └── final_results/
│       ├── domain_taxonomy_all_chains_corrected_20260618_153701.json
│       ├── domain_taxonomy_all_chains_corrected_summary_20260618_153701.json
│       ├── t1_rerun_corrected.json
│       ├── t1_rerun_corrected_summary.json
│       ├── domain_taxonomy_all_chains.json
│       └── domain_taxonomy_all_chains_summary.json
├── CSV/
│   └── final_tables/
│       ├── generated .csv / .tsv / .tex tables
├── data/
│   └── manual_validation/
│       ├── manual_precision_recall_200_final.tar.gz
│       └── manual_recall_200_final.tar.gz
├── Allium_trends_figures/
│   └── adoption and trend figures
├── final_all_chain_results/
│   └── paper-ready result exports
└── README.md
```

## What DOMAUDIT Checks

DOMAUDIT analyzes Solidity source code for signature-domain construction errors. The analyzer extracts EIP-712 / permit evidence such as:

* local `permit()` implementations,
* `DOMAIN_SEPARATOR` definitions,
* EIP-712 domain typehashes,
* chain-id usage,
* verifying-contract binding,
* salt and logical-domain indicators,
* cached or stored domain separators,
* proxy / upgradeable patterns,
* ERC-5267 metadata exposure.

The analyzer maps detected violations into five authority-boundary classes:

| Class | Meaning                      | Rules  |
| ----- | ---------------------------- | ------ |
| C1    | Domain-separation violations | R1     |
| C2    | Chain-binding violations     | R2, R3 |
| C3    | Verifier-binding violations  | R4, R5 |
| C4    | Domain-freshness violations  | R6     |
| C5    | Logical-domain violations    | R7, R8 |

A finding is a **replay-risk observation**, not an automatic exploit claim. Exploitability depends on whether another verifier reconstructs the same digest and accepts its local checks.

## Final Paper Numbers

The final evaluation uses 148,517 verified Solidity files across four EVM ecosystems.

### Source-Code Corpus

| Blockchain | Verified Files | Analyzed Files | Local permit() | EIP-712 Evidence | Replay-Risk Candidates | Candidate Rate |
| ---------- | -------------: | -------------: | -------------: | ---------------: | ---------------------: | -------------: |
| Ethereum   |         36,993 |         36,993 |         27,822 |           35,501 |                  5,737 |         15.51% |
| BNB Chain  |         52,139 |         52,139 |         46,042 |           51,097 |                 10,307 |         19.77% |
| Avalanche  |          6,043 |          6,043 |          5,197 |            5,844 |                  2,032 |         33.63% |
| Polygon    |         53,342 |         53,342 |         43,170 |           50,441 |                  8,402 |         15.75% |
| **Total**  |    **148,517** |    **148,517** |    **122,231** |      **142,883** |             **26,478** |     **17.83%** |

`Replay-Risk Candidates` are raw files assigned at least one violation class before primary-rule attribution.

### Primary Rule-Level Observations

After primary-rule attribution, DOMAUDIT reports 11,447 replay-risk observations.

| Class / Rule |  Ethereum | BNB Chain | Avalanche |   Polygon |      Total |
| ------------ | --------: | --------: | --------: | --------: | ---------: |
| C1/R1        |       559 |       679 |       746 |       427 |      2,411 |
| C2/R2        |        32 |        44 |         4 |       424 |        504 |
| C2/R3        |       146 |       161 |        13 |        56 |        376 |
| C3/R4        |        30 |         8 |         0 |        18 |         56 |
| C3/R5        |       196 |       110 |        17 |       450 |        773 |
| C4/R6        |     1,448 |     3,706 |       325 |     1,526 |      7,005 |
| C5/R7        |        10 |        23 |         8 |        13 |         54 |
| C5/R8        |        59 |        38 |        35 |       136 |        268 |
| **Total**    | **2,480** | **4,769** | **1,148** | **3,050** | **11,447** |

## Installation

DOMAUDIT requires Python 3.9 or newer.

```bash
git clone https://github.com/BatchTransfer/DOMAUDIT.git
cd DOMAUDIT

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Minimal dependencies:

```text
tqdm
pandas
```

If `requirements.txt` is not yet created, create it with:

```text
pandas
tqdm
```

## Running the Analyzer

To analyze a folder of Solidity files:

```bash
python3 python/domain_separator_analyzer_corrected.py \
  --input-dir path/to/solidity/files \
  --output-json JSON/results.json \
  --output-summary JSON/summary.json
```

Example:

```bash
python3 python/domain_separator_analyzer_corrected.py \
  --input-dir data/example_contracts \
  --output-json JSON/example_results.json \
  --output-summary JSON/example_summary.json
```

The output JSON contains per-file evidence, including:

* `has_permit`
* `has_domain_separator`
* `has_eip712_domain_typehash`
* `uses_dynamic_chainid`
* `uses_address_this`
* `risk_category`
* `taxonomy_ids`
* `critical_issues`
* `warnings`

## Reproducing the Paper Tables

The final paper tables are generated from the result JSON files in:

```text
JSON/final_results/
```

Important files:

```text
domain_taxonomy_all_chains_corrected_20260618_153701.json
domain_taxonomy_all_chains_corrected_summary_20260618_153701.json
t1_rerun_corrected.json
t1_rerun_corrected_summary.json
domain_taxonomy_all_chains.json
domain_taxonomy_all_chains_summary.json
```

The final table combines:

* C1 from `t1_rerun_corrected.json`,
* C2--C4 from `domain_taxonomy_all_chains_corrected_20260618_153701.json`,
* C5 from `domain_taxonomy_all_chains.json`.

This reflects the final corrected analysis used in the paper.

## Manual Validation

The repository includes two manual-validation datasets.

### Precision Dataset

```text
data/manual_validation/manual_precision_recall_200_final.tar.gz
```

This contains 200 flagged findings, stratified across the five replay-risk classes:

| Class     | Reviewed |
| --------- | -------: |
| C1        |       40 |
| C2        |       40 |
| C3        |       40 |
| C4        |       40 |
| C5        |       40 |
| **Total** |  **200** |

Observed precision:

| Class     |      TP |     FP | Precision |
| --------- | ------: | -----: | --------: |
| C1        |      33 |      7 |     82.5% |
| C2        |      36 |      4 |     90.0% |
| C3        |      30 |     10 |     75.0% |
| C4        |      37 |      3 |     92.5% |
| C5        |      28 |     12 |     70.0% |
| **Total** | **164** | **36** | **82.0%** |

### Recall Dataset

```text
data/manual_validation/manual_recall_200_final.tar.gz
```

This contains 200 non-flagged signature-related candidates. We manually inspected them and found no missed violations, giving observed sampled recall of 100% on this set.

## Artifact Contents

The artifact includes:

1. Analyzer source code.
2. Final evaluation JSON files.
3. Aggregated tables and result exports.
4. Manual precision and recall validation samples.
5. Adoption figures and trend data.
6. Controlled replay demonstration artifacts.

## Controlled Replay Demonstration

The repository contains the proof-of-concept artifact showing that a valid permit signature for one contract can be accepted by another contract that reconstructs the same signing domain.

The demonstration records:

* source and target contract addresses,
* transaction hashes,
* domain separators,
* signature values,
* replay transaction,
* nonce and allowance state changes.

This experiment demonstrates that signer recovery only proves who signed; domain construction determines where the signature is valid.

## Notes on Reproducibility

The full raw verified-source corpus is large. This repository includes the final analysis outputs and validation samples needed to reproduce the paper tables. If reviewers need the full source corpus, it can be regenerated using the scripts and source manifests, or provided as a compressed artifact separately.

Do not commit API keys or `.env` files.

## Citation

If you use this artifact, please cite the ICSE paper associated with DOMAUDIT.

```bibtex
@inproceedings{domaudit2026,
  title     = {Signature-Construction Correctness for Replay-Resistant Smart Contracts},
  author    = {Anonymous},
  booktitle = {Proceedings of the International Conference on Software Engineering},
  year      = {2026}
}
```

## License

This artifact is released for academic review and reproducibility. Add the final license file before making the repository public.
