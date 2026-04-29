[![License: CC BY-NC-ND 4.0](https://img.shields.io/badge/License-CC%20BY--NC--ND%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/)

# Business Utility of Large Language Models as Exploratory Data Analysis Agents

## Overview

This repository presents a benchmark for evaluating Large Language Models (LLMs) as exploratory data analysis (EDA) agents in a business-relevant setting. The benchmark is built around a controlled agent-based supply chain simulation in which the model must reconstruct the source of a quality problem from indirect operational evidence rather than from explicit labels.

The benchmark is motivated by a practical question: can an LLM act as a dependable EDA agent when success depends not only on average task performance but also on repeatability? In organizational settings, a model that occasionally produces strong analyses may still be difficult to trust if repeated runs under the same conditions vary substantially in quality. For that reason, this repository emphasizes both analytical performance and stability.

At present, the benchmark contains its first simulation family. Our intention is to extend it with additional **agent-based simulations** that will generate new datasets for the benchmark and broaden evaluation across EDA-relevant business settings.

This README is a concise research-oriented summary of the benchmark. The full write-up, methodological details, result tables, and references are available in [manuscript.md](manuscript.md).

## Benchmark Task

The benchmark is framed as a causal reconstruction problem over business-process data. In the simulated supply chain, producers deliver goods to a wholesaler, the wholesaler distributes them to stores, and customers purchase products from those stores. Some producers occasionally ship low-quality batches. These failures do not appear as direct labels. Instead, they leave delayed traces in downstream records such as sales, orders, and receipts.

The analytical task is to identify the supplier-product combinations responsible for the quality problem. Solving the task requires the model to inspect heterogeneous records, connect events across time and entities, and infer plausible causal sources of degraded performance.

Each dataset contains a mixture of structured and semi-structured artifacts, including:

- `sales_log.csv`
- `raw_orders/`
- `raw_supplier_receipts/`
- `raw_wholesaler_receipts/`
- `raw_wholesaler_store_receipts/`

The raw files intentionally combine formats such as CSV, JSON, and TXT so that the task more closely resembles messy operational analysis than a benchmark defined by a single clean table.

## Experimental Design

The benchmark evaluates 15 model-variant configurations from 8 model families across 4 experimental conditions. Each configuration is run on 5 trajectories per condition, which yields 20 trajectories per configuration and 300 scored runs in total.

The four conditions vary the analytical environment in ways intended to reflect realistic differences in how the same business problem may be presented to an analyst.

| Experiment | Data representation | Prompt | Simulation setup and signal |
| --- | --- | --- | --- |
| `Experiment 1` | Raw data | Short | Setup 1 with clear, strong signal |
| `Experiment 2` | Raw data plus redundant tabular data | Short | Setup 1 with clear, strong signal |
| `Experiment 3` | Raw data | Non-ambiguous | Setup 1 with clear, strong signal |
| `Experiment 4` | Raw data | Short | Setup 2 with weaker signal |

All runs use the same evaluation harness, OpenCode `v1.2.27`, and default model temperature settings. Failed trajectories are retried once. If the retry also fails, the trajectory receives a score of `0`. A failure is defined as either missing JSON output or a stalled run with no `stdout` or `stderr` change for 1200 seconds.

## Evaluation Framework

Each run produces a JSON prediction containing the suspected supplier-product combinations. Predictions are compared against deterministic ground truth using the Jaccard score.

The benchmark reports four main descriptors:

- `ms`: mean score, used as the estimate of average analytical quality
- `CoV`: coefficient of variation, used to capture relative instability within a condition
- `mean time`: the average runtime per trajectory
- `Business utility`: a risk-adjusted summary that combines quality and repeatability

`Business utility` is defined as:

`ms * exp(-2.25 * CoV^0.88)`

The instability parameters in this formulation are adapted from prospect theory, specifically from the loss-side parametrization used to represent nonlinear sensitivity to losses. In the present benchmark, they are not intended to model managerial behavior directly. Instead, they are used as a practical way to express the idea that reduced repeatability should lower perceived usefulness in a nonlinear rather than purely proportional way. The purpose of this metric is therefore not to replace the underlying measures, but to summarize a deployment-oriented intuition in a single quantity: a model is more useful when it is both accurate on average and sufficiently repeatable to support trust in repeated use.

In this benchmark, `Business utility` is bounded between `0` and `1`. However, the benchmark does not define a universal threshold for what level of utility should count as acceptable for deployment. That judgment is left open because acceptable utility depends on managerial risk tolerance, verification costs, and the practical demands of a specific organizational setting.

The benchmark also reports exploratory condition-sensitivity analyses based on Mann-Whitney U and Kruskal-Wallis statistics. Because each condition contains only five trajectories per configuration, these tests should be interpreted as diagnostic signals rather than confirmatory evidence.

## Main Findings

Across all 300 trajectories, the overall `ms` is `0.3562`, and the mean runtime is `783.56` seconds. At the pooled level, performance is highest in `Experiment 1` and lowest in `Experiment 4`, where the underlying signal is weaker.

The strongest configuration in the risk-adjusted ranking is `openai/gpt-5.4` with `xhigh` reasoning effort. It achieves an experiment-averaged `ms` of `0.8748` and an experiment-averaged `Business utility` of `0.6952`.

The broader result is more cautious than a mean-score leaderboard alone would imply. Across model-variants, the average `Business utility` is `0.1511` while the average `ms` is `0.3562`. In other words, once instability is taken into account, the practical picture becomes substantially less favorable. For this benchmark, repeatability is not a secondary property of model behavior. It is a central part of whether the model can be considered useful.

### Ranking by Business Utility

The table below summarizes the global ranking by experiment-averaged `Business utility`. It shows more directly than a mean-score leaderboard how the relative ordering changes once average task quality is discounted by instability.

| Rank | Model | Variant | Experiment-averaged `ms` | Experiment-averaged `CoV` | Experiment-averaged stability discount | Experiment-averaged `Business utility` |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `openai/gpt-5.4` | `xhigh` | 0.8748 | 0.1202 | 0.7665 | 0.6952 |
| 2 | `google/gemini-3.1-pro-preview` | `high` | 0.4991 | 0.2622 | 0.6031 | 0.3533 |
| 3 | `openai/gpt-5.2` | `xhigh` | 0.6501 | 0.3739 | 0.4126 | 0.2815 |
| 4 | `anthropic/claude-sonnet-4-6` | `low` | 0.4597 | 0.3434 | 0.4728 | 0.2506 |
| 5 | `anthropic/claude-haiku-4-5` | `high` | 0.3850 | 0.4361 | 0.3573 | 0.1513 |
| 6 | `anthropic/claude-haiku-4-5` | `low` | 0.3620 | 0.6022 | 0.2586 | 0.1044 |
| 7 | `openai/gpt-5.4` | `low` | 0.3730 | 0.6973 | 0.2431 | 0.1016 |
| 8 | `anthropic/claude-sonnet-4-6` | `high` | 0.3432 | 0.7240 | 0.2263 | 0.0966 |
| 9 | `openai/gpt-5.4-mini` | `xhigh` | 0.4252 | 0.8091 | 0.1934 | 0.0929 |
| 10 | `google/gemini-3-flash-preview` | `high` | 0.3419 | 0.7755 | 0.1799 | 0.0700 |
| 11 | `google/gemini-3-flash-preview` | `low` | 0.2501 | 1.1577 | 0.0926 | 0.0289 |
| 12 | `openai/gpt-5.2` | `low` | 0.1128 | 0.6526 | 0.5622 | 0.0258 |
| 13 | `mistral/mistral-large-latest` | `default` | 0.1635 | 1.1297 | 0.0839 | 0.0130 |
| 14 | `google/gemini-3.1-pro-preview` | `low` | 0.1003 | 2.0000 | 0.0159 | 0.0016 |
| 15 | `openai/gpt-5.4-mini` | `low` | 0.0027 | 0.5000 | 0.7540 | 0.0001 |

## Interpretation

The benchmark supports a measured conclusion. Current frontier LLMs can sometimes solve an EDA-like causal attribution task in realistic business data, but most evaluated configurations are still too unstable for dependable autonomous use.

The main concern is not simply whether performance changes across experimental conditions. The larger issue is that repeated runs under the same condition often produce meaningfully different analytical outcomes. This matters especially for EDA, where usefulness depends on the ability to recover meaningful structure from the same evidence in a reasonably consistent way.

At the pooled level, only the comparison between `Experiment 1` and `Experiment 4` crosses the conventional `p <= 0.05` threshold in the manuscript's exploratory condition-sensitivity analysis. That pattern suggests that environmental changes do matter, but the dominant obstacle in this benchmark remains within-condition instability.

## Repository Contents

- [manuscript.md](manuscript.md): full paper, methods, result tables, and references
- [simulation_description.md](simulation_description.md): concise description of the simulated business problem
- `experiments/`: benchmark datasets, prompts, and ground-truth answers for each condition
- `charts/`: figures used in the documentation and manuscript

## Scope and Limitations

- The current benchmark covers its first simulation family and one controlled business problem rather than the full range of EDA work.
- Additional simulation families are planned for future benchmark extensions, but they are not part of the present release.
- Results should be interpreted as benchmark evidence for this task setting, not as a universal ranking of model quality across analytical domains.
- The task focuses on delayed causal attribution in supply-chain records, not general business intelligence performance.
- Condition-sensitivity tests are exploratory because each condition includes only five trajectories per configuration.

## Roadmap

Planned next steps for the benchmark currently include:

- a Python-based evaluation script for running the benchmark,
- additional simulations and the resulting benchmark datasets.

## How to Cite

If you use this benchmark, dataset, or accompanying materials in academic work, please cite the accompanying manuscript:

`Labedzki, R., et al. (2026). Business Utility of Large Language Models as Exploratory Data Analysis Agents.`

This section will be updated with full citation details once the manuscript is available on arXiv. If you rely directly on repository artifacts, you may additionally cite the public repository once the GitHub version is released.

## Funding and Affiliation

This benchmark and the accompanying research were developed by a team from `deepsense.ai` and funded by `deepsense.ai`.

## License

This dataset is licensed under the [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/).

You may share the dataset in its original, unmodified form, provided that proper attribution is given. You may not modify, transform, or build upon the dataset. Commercial use of the dataset is not permitted. For commercial licensing inquiries, please reach out to `contact@deepsense.ai`.
