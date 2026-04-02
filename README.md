[![License: CC BY-NC-ND 4.0](https://img.shields.io/badge/License-CC%20BY--NC--ND%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/)

# LLM Benchmarking on Business Problems via Agent-Based Simulation

## Overview

This repository is a compact public summary of a broader **benchmarking approach focused on realistic business data analysis tasks** modeled with **agent-based simulations**.

The supply-chain problem presented here is one example of that approach. In this example, some producers occasionally deliver low-quality product batches, and the analytical task is to identify which producers were actually responsible for downstream quality problems using only operational data and resulting customer behavior. This is a data analysis task that, in a real business setting, would typically be performed by a data analyst or data scientist.

The problem was deliberately selected as a medium-difficulty case, meaning that current SOTA models score around 60% on average. That makes it useful for evaluation because different model families and reasoning settings already show interesting and practically important differences in score, speed, and stability, **while the underlying agent-based simulation approach can support substantially harder problem variants as well**.

Medium-difficulty business problems are especially valuable for understanding the real business value of LLMs. If model providers want to build trust in AI for operational use, they need to demonstrate not only occasional success, but **reliable and highly repeatable performance on meaningful business tasks**.

## Data Snapshot

Each experiment contains both structured and semi-structured business records rather than a single clean analytical table.

A typical dataset includes:
- `sales_log.csv`
- `raw_orders/`
- `raw_supplier_receipts/`
- `raw_wholesaler_receipts/`
- `raw_wholesaler_store_receipts/`

The raw records intentionally mix formats such as CSV, JSON, and TXT files in table-like, key-value, and text-template styles.

For example, `experiments/experiment 4/dataset` contains 14,602 files in total:
- 3,650 store order files,
- 3,650 supplier receipt files,
- 3,650 wholesaler receipt files,
- 3,650 wholesaler-to-store receipt files,
- plus a consolidated `sales_log.csv`.

## Method

We evaluated models in a single simulated environment with one shared ground truth and repeated each setup across multiple independent trajectories.

### Simulation task

Each simulated day follows the same basic flow:

1. Customers visit stores and try to buy products.
2. Stores sell available stock and reorder when inventory is low.
3. The wholesaler gathers supply from producers and distributes it to stores.
4. Customers who receive products below their quality threshold reduce future demand.

Because quality problems appear mainly through their downstream effects, the task cannot be solved by reading one label or one table. Models must infer causality from traces across store orders, wholesaler deliveries, store sales, and later demand changes.

### Experimental design

We ran four benchmark settings:

| Experiment | Input setting | Purpose |
| --- | --- | --- |
| 1 | Raw data only with a short prompt | Reference condition |
| 2 | Raw data only with a more directive prompt | Tests prompt sensitivity |
| 3 | Raw data plus an additional redundant table | Tests robustness to extra information |
| 4 | Raw data only with a more explicit, non-ambiguous prompt | Tests whether clearer task specification improves reliability |

Each model was evaluated on 5 trajectories per experiment, for 20 trajectories in total. Failed or stalled runs were retried once; if no valid output was produced, the trajectory received a score of `0`.

The evaluation metric was Jaccard score, comparing each model's predicted set of problematic producers against the ground-truth set.

All runs used the same evaluation harness and default model temperature settings. Most models were tested in both low- and high-reasoning variants.

## Results

### Overall leaderboard

Across 20 trajectories, the strongest average Jaccard score was achieved by `openai/gpt-5.4` in the `xhigh` reasoning setting, followed by `google/gemini-3.1-pro-preview` in `high` mode and `openai/gpt-5.2` in `xhigh` mode.

| Rank | Model | Variant | Average score |
| --- | --- | --- | --- |
| 1 | `openai/gpt-5.4` | `xhigh` | 0.6758 |
| 2 | `google/gemini-3.1-pro-preview` | `high` | 0.6426 |
| 3 | `openai/gpt-5.2` | `xhigh` | 0.5998 |
| 4 | `google/gemini-3-flash-preview` | `high` | 0.4111 |
| 5 | `openai/gpt-5.4` | `low` | 0.3936 |
| 6 | `openai/gpt-5.4-mini` | `xhigh` | 0.3910 |
| 7 | `google/gemini-3-flash-preview` | `low` | 0.3763 |
| 8 | `mistral/mistral-large-latest` | `default` | 0.2234 |
| 9 | `openai/gpt-5.2` | `low` | 0.1760 |
| 10 | `google/gemini-3.1-pro-preview` | `low` | 0.1163 |
| 11 | `openai/gpt-5.4-mini` | `low` | 0.0021 |

![General score leaderboard](charts/general_score_leaderboard.svg)

### Main findings

- Performance varied substantially across experiments and trajectories, even though the underlying ground truth was fixed.
- Prompting mattered, but not monotonically: experiment 2 was the weakest overall setting, while experiment 4 recovered much of that loss by making the task specification more explicit.
- Redundant information was not neutral: adding an extra table changed performance and stability rather than simply leaving results unchanged.
- Strong best-case runs show that the signal in the data was detectable, but many models still failed to extract it consistently.
- Higher reasoning settings were often important, especially for stronger OpenAI and Google configurations.

### Consistency

Average score alone did not tell the whole story. Some configurations were much more stable than others across repeated runs.

The most stable high-performing configuration in this benchmark remained `google/gemini-3.1-pro-preview` with `high` reasoning effort. `openai/gpt-5.4` with `xhigh` reasoning achieved the highest mean score, but Gemini 3.1 Pro high combined near-top quality with materially lower variance.

![Coefficient of variation leaderboard](charts/coefficient_of_variation_leaderboard.svg)

### Score vs time

The score-time comparison is shown below as one chart per provider. Each point is one experiment-level mean for a given model-variant configuration. Colors distinguish experiments, while marker shapes distinguish model families within a provider. Labels use compact names such as `gpt-5.2 (xhigh)` to keep the plots readable.

#### OpenAI

![OpenAI mean score vs mean time by experiment](charts/score_vs_time_openai.svg)

#### Google

![Google mean score vs mean time by experiment](charts/score_vs_time_google.svg)

#### Mistral

![Mistral mean score vs mean time by experiment](charts/score_vs_time_mistral.svg)

### Pareto frontier

Looking at mean score versus mean time across all 20 trajectories per configuration, the Pareto frontier highlights the non-dominated speed-quality trade-offs.

The frontier includes `openai/gpt-5.4-mini (low)`, `google/gemini-3.1-pro-preview (low)`, `openai/gpt-5.2 (low)`, `google/gemini-3-flash-preview (low)`, `openai/gpt-5.4 (low)`, `google/gemini-3-flash-preview (high)`, `google/gemini-3.1-pro-preview (high)`, and `openai/gpt-5.4 (xhigh)`.

The frontier shows that `openai/gpt-5.2 (xhigh)` and `openai/gpt-5.4-mini (xhigh)` improve quality relative to their low-effort counterparts, but they are not efficient frontier choices once runtime is taken into account.

![Pareto frontier from mean score and mean time](charts/pareto_frontier_score_vs_time.svg)

## Interpretation

This benchmark suggests that current LLMs can sometimes solve a structured root-cause analysis problem in simulated business data, but reliability remains a major weakness.

The simulation was intentionally configured to contain many repeated signal patterns over 365 simulated days. In other words, the main challenge was not the absence of evidence, but whether a model could discover and follow an effective analytical strategy. That makes the benchmark a useful test of consistency, causal reasoning, and robustness to prompt and input changes.

## Scope and limitations

- The benchmark uses one simulated environment and one ground truth.
- Results should be interpreted as a controlled comparative study, not as a universal ranking for all analytical tasks.
- The task focuses on delayed causal attribution in supply-chain data, not on general business intelligence performance.

For detailed information about the benchmark design, simulation parameters, and scoring methodology, see the [simulation description](simulation_description.md).

## License
This dataset is licensed under the [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/).
You may share the dataset in its original, unmodified form, provided that proper attribution is given.
You may not modify, transform, or build upon the dataset.
Commercial use of the dataset is not permitted.
For commercial licensing inquiries, please reach out to contact@deepsense.ai
