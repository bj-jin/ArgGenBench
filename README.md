# ArgGenBench

**ArgGenBench: Benchmarking the Complex Controlled Argument Generation Capability of Large Language Models**

Bojun Jin*, Jianzhu Bao*, Yang Sun, Yice Zhang, Ruifeng Xu†

*Equal contribution. †Corresponding author.

**ACL 2026 · Long Paper**

[Paper](https://aclanthology.org/2026.acl-long.1414/) · [PDF](https://aclanthology.org/2026.acl-long.1414.pdf) · [Dataset](data/test.json) · [Citation](#citation)

## Overview

ArgGenBench evaluates whether large language models can generate persuasive arguments while following complex instructions that combine multiple argumentative requirements. It goes beyond topic and stance by jointly controlling length, style, argumentative strategy, target audience, and key points, with an optional input argument to support or refute.

The benchmark contains **803 instances across 82 topics**, each pairing an instruction with a human-verified reference argument. It supports research on controlled argument generation under zero-shot, supervised fine-tuning (SFT), and direct preference optimization (DPO) settings.

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

The paper reports an average of 4.2 additional control signals per instruction, with 88.4% of instructions integrating multiple signals. Average instruction and reference lengths in the paper are 48.6 and 235.2 words, respectively.

### Construction

The benchmark is built through two phases:

1. **Instruction creation.** Topics are curated from Kialo. Basic elements are combined with sampled control signals, synthesized into natural language instructions using GPT-4.1, and manually checked for clarity, specificity, feasibility, and natural phrasing.
2. **Reference argument curation.** Claude-Sonnet-4, GPT-4.1, and Gemini-2.5-Pro generate candidate arguments with external knowledge from Kialo and web retrieval. Expert annotators select the strongest candidates based on instruction adherence and argument quality, then revise them where necessary. The paper reports manual revision of 41% of references.

`Stage1` and `Stage2` identify the two construction rounds: Stage1 instances do not require an input argument; Stage2 instances provide one in the `input_argument` field. These rounds are distinct from the instruction-creation and reference-curation phases above.

### Format

[`data/test.json`](data/test.json) is a UTF-8 JSON array. Every record contains `id`, `topic`, `instruction`, and `reference_argument`. Stage2 records additionally contain `input_argument`:

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Unique identifier in the form `Stage1-{n}` or `Stage2-{n}`. Numbering starts at 1 independently in each stage and is consecutive. |
| `topic` | string | The debate topic. |
| `instruction` | string | The generation request and its argumentative requirements. |
| `input_argument` | string | Present in Stage2 only: the full input argument to support or refute. |
| `reference_argument` | string | The human-verified reference argument; named `output` in the internal data format. |

The released data have undergone further refinement and optimization following the study. Original record order is retained, with consecutive identifiers assigned within each stage; the dataset statistics reported in the paper refer to the paper's evaluation version.

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
from pathlib import Path

dataset = json.loads(Path("data/test.json").read_text(encoding="utf-8"))

print(f"Instances: {len(dataset)}")

sample = dataset[0]
print(sample["id"])
print(sample["instruction"])
print(sample["reference_argument"])

# Include the full input argument if it is not already present in the instruction.
prompt = sample["instruction"]
input_argument = sample.get("input_argument")
if input_argument and input_argument not in prompt:
    prompt += "\n\nInput argument:\n" + input_argument
messages = [{"role": "user", "content": prompt}]
```

Use `instruction` as the generation request and retain `reference_argument` for evaluation. For Stage2, also provide the full `input_argument` if it is not already included in the instruction, as shown above. No additional Python packages are required to load the dataset.

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
