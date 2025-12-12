# Powdery Mildew (Oidium) Risk Prediction Model – Grapevine (Mistral 7B / 8B)

## Project overview

This repository presents a **complete applied AI project for predicting grapevine powdery mildew (Oidium, *Erysiphe necator*) risk** using open-source **Mistral language models**.

The objective is not to build a simple classifier, but to demonstrate that a **well-scoped, well-trained, and well-evaluated LLM** can learn a **conditional agronomic decision logic** close to expert reasoning, and be reliably integrated into **real decision-support workflows** (API, automation, alerts).

Two complementary approaches are implemented and compared:
- **Mistral 7B fine-tuned with LoRA (SFT in Google Colab)**
- **Mistral 8B fine-tuned via lightweight SFT using Mistral AI Studio**

Both reach a **comparable agronomic decision quality (~90% validity)**, with different but complementary profiles.

---

## Agronomic principles

The model is grounded in well-established viticulture and plant pathology principles, including:

- Optimal temperature range for powdery mildew development (≈ 20–25 °C)
- Central role of **persistently high relative humidity** (≥ 70–80% for ≥ 6 hours)
- Increased susceptibility of **young leaves and flowering stage**
- Importance of **inoculum pressure** (normalized proxy based on previous seasons)
- Moderating effect of **ETP (evapotranspiration)**, especially after rainfall

A key design choice was to explicitly include **realistic borderline cases** (rainfall combined with strong drying, humidity near thresholds, intermediate inoculum levels) in order to avoid a simplistic or alarmist model.

---

## Repository structure

The repository is organized to clearly reflect each step of the project: dataset generation, training, quality control, and inference testing.

```
├── README.md                         # Full documentation (French)
├── README_En.md                      # English documentation (this file)
│
├── code_prompt/
│   ├── Lyra_oidium_7b.py             # Python script for SFT LoRA training (Colab)
│   └── system_prompt.txt             # System prompt defining agronomic logic and strict output format
│
├── controle_datasets/
│   └── quality_control_fr.md         # Detailed dataset quality control report (French)
│
├── datasets/
│   ├── dataset_no_system_prompt/     # Datasets for SFT LoRA (Colab)
│   │   ├── Oidium_Mistral7B_train_1500.jsonl
│   │   └── Oidium_Mistral7B_eval_100.jsonl
│   │
│   └── dataset_system_prompt/        # Datasets with system prompt injected (AI Studio)
│       ├── Oidium_Mistral7B_train_1500_with_system.jsonl
│       └── Oidium_Mistral7B_eval_100_with_system.jsonl
│
├── test_models/
│   ├── playground_7B_8B_14B_noTrain.txt   # Zero-training inference tests (playground)
│   ├── test_inference_7b_SFT_colab.txt    # Inference tests after 7B LoRA SFT
│   └── Tests_SFT_8B_AIstudio.xlsx          # Inference test results after 8B SFT (AI Studio)
```

---

## Datasets and quality control

The datasets are **synthetically generated**, following strict agronomic rules:

- **Training set**: 1500 samples
- **Evaluation set**: 100 samples
- Class distribution:
  - 30% low risk
  - 40% medium risk
  - 30% high risk
- **No strict duplicates** (train / eval / cross)
- **112 explicitly designed borderline cases**

All validation steps (duplicate detection, class balance, agronomic consistency) are documented in:
`controle_datasets/quality_control_fr.md`.

---

## Model training

### Mistral 7B – LoRA SFT (Google Colab)

- Training performed on Google Colab
- Targeted LoRA approach
- Dataset without embedded system prompt (prompt injected at inference)
- Fast and stable convergence
- Strong improvement over the base 7B model

This approach demonstrates that a **smaller model can compete with larger ones** when domain rules are properly learned.

### Mistral 8B – Lightweight SFT (Mistral AI Studio)

- Training performed using Mistral AI Studio
- Dataset with **system prompt embedded**
- 2 epochs, learning rate 1e-5
- Clear improvement in decision stability and output consistency

The 8B SFT model is especially suited for **API-based and automated workflows**.

---

## Inference tests and results

Inference tests are deliberately qualitative and scenario-based:

- Clear low-risk situations
- Realistic medium-risk situations
- Borderline cases (rain + high ETP, humidity near thresholds)
- Clear high-risk situations

### Summary of results

- **≈ 90% of decisions are agronomically valid**
- 0 biologically incoherent decisions observed
- Remaining discrepancies correspond to **defensible conservative choices**
- Main contribution of SFT:
  - explicit risk classification (low / medium / high)
  - strict, machine-exploitable output format

Detailed test files are available in `test_models/`.

---

## Comparison: 7B LoRA vs 8B SFT

Both models reach a comparable level of agronomic performance, with different strengths:

- **7B LoRA**: strong rule learning, clear demonstration that *dataset quality > model size*
- **8B SFT**: improved homogeneity, robustness, and production readiness

This project shows that **model framing and dataset design** are more decisive than raw parameter count.

---

## Use cases

- Agronomic decision support
- Vineyard disease risk monitoring
- Integration into automated pipelines (API, n8n)
- Educational and demonstrative project combining AI and agronomy

⚠️ This model is an **assistive decision-support tool** and does not replace local human expertise.

---

## License and status

- License: Apache-2.0
- Status: validated, reproducible, and fully documented project
- Models deployed and tested via API

---

For full details and dataset analysis, see the French documentation in `README.md`.