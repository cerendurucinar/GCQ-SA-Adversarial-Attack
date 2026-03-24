# Adversarial Suffix Optimization for Jailbreaking Large Language Models

This repository contains the implementation, experiments, and analysis for a research project on **adversarial suffix optimization** in large language models (LLMs), with a focus on **black-box jailbreak attacks**.

The project was carried out as part of **ETH SSRF 2025** at the **Advanced Computing Lab, ETH Zürich**, under the supervision of **Prof. Markus Püschel** and **Mohamed Hicham Leghettas**.

## Overview

Aligned large language models are designed to refuse harmful or restricted requests. However, prior work has shown that carefully optimized token suffixes appended to an input prompt can still steer these models toward unsafe completions. This repository studies that setting in the **black-box regime**, where the attacker has no access to gradients and only interacts with the model through forward queries.

The main focus of the project is a comparison between:

- **GCQ (Greedy Coordinate Query)**, a query-based adversarial suffix optimization method
- **GC-SA (Greedy Coordinate Simulated Annealing)**, a simulated annealing variant explored in this project

The core question is whether adding annealed acceptance to coordinate-wise suffix search improves exploration and optimization behavior compared to a purely greedy query-based method.

## Project Motivation

Adversarial suffix attacks are an important security concern because they demonstrate that model alignment alone may not be sufficient to prevent jailbreak behavior. Unlike gradient-based attacks, black-box attacks are especially relevant in realistic deployment settings, where only API or query access is available.

This project builds on the **Query-Based Adversarial Prompt Generation** framework and investigates whether a simulated annealing mechanism can help the search process escape poor local minima while preserving the black-box setting.

## Method Summary

### GCQ

GCQ is a black-box adversarial prompt optimization method that iteratively improves candidate suffixes using only model queries. It maintains a buffer of candidates, perturbs one suffix position at a time, evaluates the resulting loss, and keeps the best-performing candidates.

### GC-SA

GC-SA extends this idea by incorporating **simulated annealing** into the coordinate-wise update process. Instead of accepting only strictly improving updates, the algorithm can also accept worse local moves with a probability controlled by a decaying temperature schedule. This encourages broader exploration of the token search space, especially in the earlier stages of optimization.

In this way, GC-SA remains fully black-box while introducing a more exploratory update strategy than standard greedy coordinate search.

## Experimental Setup

The experiments were conducted on a dataset of **131 adversarial prompts** paired with predefined target strings.

The evaluation uses a **refusal-prefix heuristic**:
- outputs containing refusal patterns such as *"I'm sorry"* or *"As an AI"* are marked as **non-jailbroken**
- all other outputs are treated as **jailbroken**

The main implementation settings used in the experiments are:

- **400 optimization iterations**
- **suffix length = 16**
- **beam / local refinement width = 20**

## Results

Both methods achieved very similar performance in the evaluated setting:

| Method | Prompts Tested | Jailbreak Success Rate |
|--------|----------------|------------------------|
| GC-SA (proposed) | 131 | ~78% |
| GCQ | 131 | ~78% |

These results suggest that simulated annealing provides a competitive alternative to greedy coordinate query search, but does not substantially outperform GCQ in this experimental setup.

## Exploration Analysis

Beyond final success rate, this project also analyzes how optimization effort is distributed across suffix positions.

The analysis shows that **middle-to-late suffix positions**, especially positions **8–14**, are selected much more frequently than early positions. In particular, **position 10** is chosen most often on average. This suggests that later suffix tokens exert stronger influence on the transition into the desired target continuation, while earlier positions are less impactful during optimization.

This positional imbalance is one of the main analytical findings of the project and may motivate future work on adaptive or position-aware black-box search strategies.


## Notebook Guide

### `00_model_setup.ipynb`
Loads the language model and other required components for the experiments.

### `01_attack_core_and_evaluation.ipynb`
Contains the main implementation of the adversarial suffix optimization pipeline and the evaluation workflow.

### `02_heap_based_variant.ipynb`
Includes the alternative or ablation-style implementation used for comparison or efficiency-oriented experimentation.

### `03_analysis_and_plots.ipynb`
Contains post-processing, visualization, and analysis of the experimental results, including optimization dynamics across suffix positions.

## Installation

It is recommended to create a virtual environment before installing dependencies.

```bash
pip install -r requirements.txt
```

## Usage

Run the notebooks in the following order:

1. `00_model_setup.ipynb`
2. `01_attack_core_and_evaluation.ipynb`
3. `02_heap_based_variant.ipynb` *(optional, if used in your experiments)*
4. `03_analysis_and_plots.ipynb`

Depending on your local environment, some file paths may need to be updated.

## Research Context

This repository was developed as part of an **ETH SSRF 2025** research project at the **Advanced Computing Lab, ETH Zürich**.

The work was conducted under the supervision of:

- **Prof. Markus Püschel**
- **Mohamed Hicham Leghettas**

## Limitations

The evaluation in this repository is based on a refusal-prefix heuristic rather than a semantic safety classifier. In addition, jailbreak success rates are not static and may vary across model versions, checkpoints, and safety updates. Therefore, the reported results should be interpreted within the specific experimental setting used in this project.

## Future Directions

Possible future extensions include:

- evaluating on larger and more diverse prompt datasets
- using semantic or model-based safety evaluation metrics
- exploring hybrid search strategies that combine annealing with adaptive candidate generation
- designing position-aware update rules based on observed suffix influence

## References

- Zou, A., Wang, Z., Carlini, N., Nasr, M., Kolter, J. Z., and Fredrikson, M. *Universal and Transferable Adversarial Attacks on Aligned Language Models*. 2023.
- Hayase, J., Borevkovic, E., Carlini, N., Tramèr, F., and Nasr, M. *Query-Based Adversarial Prompt Generation*. 2024.
- Additional references are included in the accompanying report.
