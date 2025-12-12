<img src="https://upload.wikimedia.org/wikipedia/en/c/c3/Flag_of_France.svg" width="100px" height="auto" />

![Mistral](https://img.shields.io/badge/Model-Mistral%207B-blue)
![LoRA](https://img.shields.io/badge/Fine--Tuning-LoRA-green)
![SFT](https://img.shields.io/badge/Training-SFT-orange)
![Agronomy](https://img.shields.io/badge/Domain-Agronomy%20%2F%20Viticulture-darkgreen)
![HuggingFace](https://img.shields.io/badge/Platform-Hugging%20Face-yellow)
![License](https://img.shields.io/badge/License-Apache%202.0-lightgrey)

# Modèle de prévision du risque d’oïdium de la vigne (Mistral 7B / 8B)

## Présentation générale

Ce dépôt présente un **projet complet de modélisation du risque d’oïdium de la vigne (*Erysiphe necator*)** à l’aide de modèles de langage open‑source de la famille **Mistral**.

L’objectif n’est pas uniquement de produire un classifieur de risque, mais de démontrer qu’un **LLM correctement cadré, entraîné et évalué** peut apprendre une **logique agronomique conditionnelle**, proche du raisonnement expert, et être intégré dans des **workflows décisionnels réels** (API, automatisation, alertes).

Deux approches complémentaires sont explorées et comparées :
- **Mistral 7B fine‑tuné par LoRA (SFT en Colab)**
- **Mistral 8B fine‑tuné par SFT léger via Mistral AI Studio**

Les deux atteignent un **niveau de décision agronomique comparable (~90 % de validité)**, avec des profils différents mais cohérents.

---

## Principes agronomiques du modèle

Le modèle repose sur des règles issues de la littérature agronomique et de l’expertise terrain, en particulier :

- Fenêtre thermique favorable à l’oïdium (≈ 20–25 °C)
- Rôle central de l’**humidité relative élevée maintenue dans le temps** (≥ 70–80 % pendant ≥ 6 h)
- Sensibilité accrue des **jeunes feuilles et de la floraison**
- Importance de la **pression d’inoculum** (proxy basé sur les saisons précédentes)
- Effet modérateur de l’**ETP** (assèchement), notamment après pluie

Un point clé du projet est d’avoir privilégié des **cas frontières réalistes** (pluie + forte ETP, HR limite, inoculum intermédiaire), afin d’éviter un modèle purement binaire ou alarmiste.

---

## Structure du dépôt

L’arborescence du projet est organisée pour refléter clairement les différentes étapes : génération des données, entraînement, contrôle qualité et tests d’inférence.

```
├── README.md                         # Documentation complète en français
├── README_En.md                      # Documentation anglaise (résumé du projet)
│
├── code_prompt/
│   ├── Lyra_oidium_7b.py             # Script Python d’entraînement SFT LoRA (Colab)
│   └── system_prompt.txt             # Prompt système définissant le cadre agronomique et le format strict
│
├── controle_datasets/
│   └── quality_control_fr.md         # Compte rendu détaillé du contrôle qualité des datasets
│
├── datasets/
│   ├── dataset_no_system_prompt/     # Datasets utilisés pour le SFT LoRA (Colab)
│   │   ├── Oidium_Mistral7B_train_1500.jsonl
│   │   └── Oidium_Mistral7B_eval_100.jsonl
│   │
│   └── dataset_system_prompt/        # Datasets avec prompt système injecté (AI Studio)
│       ├── Oidium_Mistral7B_train_1500_with_system.jsonl
│       └── Oidium_Mistral7B_eval_100_with_system.jsonl
│
├── test_models/
│   ├── playground_7B_8B_14B_noTrain.txt   # Tests d’inférence sans entraînement (playground)
│   ├── test_inference_7b_SFT_colab.txt    # Tests après SFT LoRA sur Mistral 7B
│   └── Tests_SFT_8B_AIstudio.xlsx          # Résultats comparatifs après SFT 8B (AI Studio)
```

---

## Datasets et contrôle qualité

Les datasets ont été **générés synthétiquement**, mais selon des règles strictes :

- **1500 lignes d’entraînement**, 100 lignes d’évaluation
- Répartition volontairement équilibrée :
  - 30 % risque faible
  - 40 % risque moyen
  - 30 % risque élevé
- **Aucun doublon strict** (train / eval / croisé)
- **112 cas frontières** explicitement intégrés

L’ensemble des vérifications (doublons, répartition, cohérence agronomique) est documenté dans :
`controle_datasets/quality_control_fr.md`.

---

## Entraînement des modèles

### Mistral 7B – SFT LoRA (Colab)

- Entraînement réalisé sous Google Colab
- Approche LoRA ciblée
- Dataset sans prompt système (prompt injecté à l’inférence)
- Convergence rapide et stable
- Gain fort par rapport au 7B de base

Cette approche démontre qu’un **modèle plus petit peut rivaliser avec des modèles plus grands** si les règles métier sont correctement apprises.

### Mistral 8B – SFT léger (Mistral AI Studio)

- Entraînement via AI Studio (SFT non agressif)
- Dataset avec **prompt système injecté**
- 2 epochs, learning rate 1e‑5
- Amélioration nette de la stabilité du format et de la décision

Le SFT 8B est particulièrement adapté à un **usage API / workflow automatisé**.

---

## Tests d’inférence et résultats

Les tests sont volontairement qualitatifs et ciblés :

- Cas faibles évidents
- Cas moyens réalistes
- Cas frontières (pluie + ETP forte, HR limite)
- Cas élevés clairs

### Résultats synthétiques

- **≈ 90 % des décisions sont agronomiquement valides**
- 0 décision biologiquement incohérente
- Les rares écarts correspondent à des **choix prudents défendables**
- Le SFT apporte surtout :
  - une **décision explicite** (faible / moyen / élevé)
  - un **format strict et exploitable**

Les fichiers de tests détaillés sont disponibles dans `test_models/`.

---

## Comparaison 7B LoRA vs 8B SFT

Les deux modèles atteignent un niveau comparable, avec des profils distincts :

- **7B LoRA** : apprentissage fort des règles, démonstration "dataset > taille"
- **8B SFT** : meilleure homogénéité, robustesse et intégration API

Ce projet montre que la **qualité du cadrage et du dataset** est plus déterminante que la taille brute du modèle.

---

## Cas d’usage

- Aide à la décision agronomique
- Surveillance du risque phytosanitaire
- Intégration dans des workflows automatisés (n8n, API)
- Démonstrateur pédagogique IA & agronomie

⚠️ Ce modèle est un **outil d’aide à la décision** et ne remplace pas une expertise humaine locale.

---

## 🔗 Modèle Hugging Face

Le modèle fine-tuné est disponible publiquement sur Hugging Face :

👉 **https://huggingface.co/jeromex1/lyra_Oidium_mistral7B_LoRA**

## Licence et statut

- Licence : Apache‑2.0
- Statut : projet validé, reproductible et documenté
- Modèles déployés et testés via API

---

Pour une version synthétique en anglais, voir `README_En.md`.

