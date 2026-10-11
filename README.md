# ArgGenBench

**ArgGenBench: Benchmarking the Complex Controlled Argument Generation Capability of Large Language Models**

Bojun Jin*, Jianzhu Bao*, Yang Sun, Yice Zhang, Ruifeng Xu†

*Equal contribution. †Corresponding author.

**ACL 2026 · Long Paper**

[Paper](https://aclanthology.org/2026.acl-long.1414/) · [PDF](https://aclanthology.org/2026.acl-long.1414.pdf) · [Dataset](data/test.json) · [Citation](#citation)

## Overview

ArgGenBench evaluates whether large language models can generate persuasive arguments while following complex instructions that combine multiple argumentative requirements. It goes beyond topic and stance by jointly controlling length, style, argumentative strategy, target audience, and key points, with an optional input argument to support or refute.

The benchmark contains **803 instances across 82 topics**, each pairing an instruction with a human-verified reference argument. The paper evaluates 15 LLMs under zero-shot, supervised fine-tuning (SFT), and direct preference optimization (DPO) settings. Even the strongest evaluated model achieves only **42.7% Overall win rate** against the references, illustrating the difficulty of satisfying these requirements together.

This release provides the benchmark test set and its documentation.

## Task and Control Dimensions

Given an `instruction`, a model should produce an argument that satisfies the requested constraints and maintains logical coherence, clarity, and persuasiveness.

| Dimension | Description |
| --- | --- |
| Topic | The proposition being discussed. |
| Stance | Support or attack the topic, or the input argument when one is provided. |
| Input argument | Optional context that the generated argument should support or refute. |
| Length | A requested range of 150–200 or 250–300 words. |
| Argumentative style | Formality, emotional intensity, and politeness, such as formal, passionate, or polite. |
| Argumentative strategy | Rhetorical approaches from logos, pathos, and ethos, such as causal reasoning, storytelling, or concession and rebuttal. |
| Target audience | The intended audience to whom the argument should be tailored. |
| Key points | Content that the argument should address. |

Instructions combine the basic elements with 0–9 additional control signals: 0–2 styles, 0–3 strategies, 0–1 audience, and 0–3 key points. The paper groups instructions into easy (0–3 signals), medium (4–6), and hard (7–9) difficulty levels.

## Dataset

### Statistics

| Property | Value |
| --- | ---: |
| Total instances | 803 |
| Distinct topics | 82 |
| Stage1: generation without an input argument | 384 |
| Stage2: generation with an input argument | 419 |

The paper reports an average of 4.2 additional control signals per instruction, with 88.4% of instructions integrating multiple signals. Average instruction and reference lengths in the paper are 48.6 and 235.2 words, respectively.

### Construction

The benchmark is built through two phases:

1. **Instruction creation.** Topics are curated from Kialo. Basic elements are combined with sampled control signals, synthesized into natural language instructions using GPT-4.1, and manually checked for clarity, specificity, feasibility, and natural phrasing.
2. **Reference argument curation.** Claude-Sonnet-4, GPT-4.1, and Gemini-2.5-Pro generate candidate arguments with external knowledge from Kialo and web retrieval. Expert annotators select the strongest candidates based on instruction adherence and argument quality, then revise them where necessary. The paper reports manual revision of 41% of references.

`Stage1` and `Stage2` identify the two construction rounds: Stage1 instances do not require an input argument; Stage2 instances include one in the instruction. These rounds are distinct from the instruction-creation and reference-curation phases above.

### Format

[`data/test.json`](data/test.json) is a UTF-8 JSON array. Every record contains exactly four fields:

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Unique identifier: `Stage1-1` through `Stage1-384`, or `Stage2-1` through `Stage2-419`. Numbering starts at 1 independently in each stage. |
| `topic` | string | The debate topic. |
| `instruction` | string | The complete generation request, including the input argument when applicable. |
| `reference_argument` | string | The human-verified reference argument; named `output` in the internal data format. |

The released data have undergone further refinement and optimization following the study. Original record order is retained, with consecutive identifiers assigned within each stage; the statistics and experimental results reported in the paper refer to the paper's evaluation version.

### Repository Structure

```text
ArgGenBench/
├── README.md
└── data/
    └── test.json
```

## Quick Start

Clone the repository:

```bash
git clone https://github.com/bj-jin/ArgGenBench.git
cd ArgGenBench
```

Load and inspect the dataset using Python's standard library:

```python
import json
from collections import Counter
from pathlib import Path

dataset = json.loads(Path("data/test.json").read_text(encoding="utf-8"))

print(f"Instances: {len(dataset)}")
print(Counter(sample["id"].split("-", 1)[0] for sample in dataset))

sample = dataset[0]
print(sample["id"])
print(sample["instruction"])
print(sample["reference_argument"])

# Pass the complete instruction to your model as the generation request.
messages = [{"role": "user", "content": sample["instruction"]}]
```

Use `instruction` as the model input and retain `reference_argument` for evaluation. In Stage2, the input argument is already embedded in the instruction. No additional Python packages are required to load the dataset.

## Evaluation

The paper evaluates models using five metrics:

| Metric | What it measures |
| --- | --- |
| ROUGE-L (R-L) | Textual similarity between generated and reference arguments. |
| Length Following (LF) | The percentage of generations meeting the requested word range. |
| Instruction Following (IF) | Win/tie/lose comparison against the reference on topic, stance, and the requested control signals. |
| Argument Quality (Quality) | Win/tie/lose comparison on logical coherence, persuasiveness, clarity, and related qualities. |
| Overall | Joint assessment of IF and Quality, subject to length adherence: failing LF yields an Overall loss. |

GPT-4.1 serves as the main automatic judge. Each generated/reference pair is evaluated twice with the argument order reversed to reduce positional bias. The paper also reports evaluation with Gemini-2.5-Pro and human assessment on 100 sampled instances. Evaluation prompts and aggregation rules are provided in Appendix B.5 and Tables 25–26; the generation prompt appears in Table 27.

### Selected Results from the Paper

The following zero-shot results are from Table 1. Values are percentages; IF, Quality, and Overall columns show **win rates against the reference arguments**.

| Model | LF | IF Win | Quality Win | Overall Win |
| --- | ---: | ---: | ---: | ---: |
| Claude-Sonnet-4 (thinking) | 86.2 | 28.5 | 46.5 | 42.7 |
| Claude-Sonnet-3.7 (thinking) | 90.2 | 23.4 | 43.3 | 38.9 |
| Qwen3-32B (thinking) | 52.8 | 33.6 | 61.5 | 28.3 |
| GPT-4.1 | 91.3 | 14.3 | 21.2 | 19.7 |
| Qwen3-14B (thinking) | 73.4 | 14.7 | 24.8 | 18.7 |
| GPT-4o | 70.7 | 10.6 | 19.3 | 11.5 |

Thinking models generally perform better in the zero-shot setting. SFT and DPO improve instruction following and argument quality for the evaluated smaller models, while length adherence remains important: DPO's gains on IF and Quality do not always translate into higher Overall scores. See the paper for complete results and analyses by difficulty and control dimension.

## Research Use

ArgGenBench is intended for academic research on controlled argument generation. Please cite the paper when using the dataset and consider the ethical implications of audience-tailored persuasion, as discussed in the paper's Ethics Statement.

## Citation

```bibtex
@inproceedings{jin-etal-2026-arggenbench,
  title = "{A}rg{G}en{B}ench: Benchmarking the Complex Controlled Argument Generation Capability of Large Language Models",
  author = "Jin, Bojun and Bao, Jianzhu and Sun, Yang and Zhang, Yice and Xu, Ruifeng",
  booktitle = "Proceedings of the 64th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)",
  month = jul,
  year = "2026",
  address = "San Diego, California, United States",
  publisher = "Association for Computational Linguistics",
  pages = "30630--30662",
  doi = "10.18653/v1/2026.acl-long.1414",
  url = "https://aclanthology.org/2026.acl-long.1414/"
}
```

## Contact

For questions or feedback, please [open an issue](https://github.com/bj-jin/ArgGenBench/issues) or contact [Bojun Jin](mailto:bojunjinmail@gmail.com).
