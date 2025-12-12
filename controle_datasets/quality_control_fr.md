# Oïdium – Modèle de prévision du risque (Mistral 7B / 8B / 14B)

## 1. Objectif du projet

Ce projet vise à développer un **modèle de langage spécialisé dans la prévision du risque d’oïdium de la vigne**, à partir de variables météorologiques et agronomiques clés, en s’appuyant sur des modèles **Mistral 7B, 8B et 14B**.

Contrairement à des modèles plus classiques (mildiou, botrytis), l’oïdium repose sur une **logique conditionnelle plus subtile**, intégrant notamment la durée d’humidité relative élevée, l’arbitrage pluie / assèchement (ETP), et le stade phénologique.

---

## 2. Variables utilisées

Les entrées du modèle sont volontairement limitées mais fortement informatives :

- Stade phénologique
- Température moyenne (24 h)
- **Durée d’humidité relative ≥ seuil sur 24 h** (et non uniquement nocturne)
- Pluie sur 24 h (booléen)
- Inoculum (proxy normalisé [0–1])
- ETP (mm/j)

Le modèle apprend à raisonner sur des **cas non triviaux**, par exemple :
- pluie récente suivie d’une forte ETP,
- nuit sèche mais humidité diurne liée à l’irrigation,
- situations frontières autour du seuil de 6 h d’humidité.

---

## 3. Structure du dataset

Les datasets sont au format **JSONL**, compatibles avec l’entraînement SFT (Mistral Instruct) :

```json
{
  "messages": [
    {"role": "user", "content": "... variables agronomiques ..."},
    {"role": "assistant", "content": "Risque: faible|moyen|élevé | Action: ..."}
  ]
}
```

Deux jeux de données sont fournis :

- **Train** : 1500 lignes
- **Évaluation** : 100 lignes

---

## 4. Répartition des classes (contrôle qualité)

### Dataset d’entraînement (1500 lignes)

| Classe de risque | Nombre | Pourcentage |
|------------------|--------|-------------|
| Faible           | 450    | 30 %        |
| Moyen            | 600    | 40 %        |
| Élevé            | 450    | 30 %        |

### Dataset d’évaluation (100 lignes)

| Classe de risque | Nombre | Pourcentage |
|------------------|--------|-------------|
| Faible           | 30     | 30 %        |
| Moyen            | 40     | 40 %        |
| Élevé            | 30     | 30 %        |

Cette répartition volontairement équilibrée (30 / 40 / 30) permet de **favoriser l’apprentissage des cas intermédiaires**, majoritaires en conditions réelles.

---

## 5. Vérification des doublons

Un contrôle exhaustif a été réalisé :

- ❌ aucun doublon strict dans le dataset d’entraînement
- ❌ aucun doublon strict dans le dataset d’évaluation
- ❌ aucune ligne strictement identique entre train et eval

👉 aucune fuite d’information possible entre les phases d’apprentissage et d’évaluation.

---

## 6. Cas frontières et logique conditionnelle

Le dataset d’entraînement contient **112 cas explicitement identifiés comme frontières**, combinant notamment :

- durée d’humidité proche du seuil critique (~6 h),
- ETP élevée (≥ 4 mm/j),
- inoculum variable,
- pluie présente ou absente.

Ces cas sont essentiels pour apprendre au modèle à :
- ne pas surévaluer le risque par simple prudence,
- arbitrer correctement entre humidité favorable et conditions d’assèchement,
- dépasser une logique de reconnaissance de patterns simples.

---

## 7. Résultats préliminaires (Playground)

Des tests comparatifs en playground montrent une hiérarchie claire :

**7B < 8B < 14B**

Contrairement aux projets Botrytis et Mildiou, où 7B pouvait surpasser 8B, l’oïdium met davantage en jeu des **capacités de raisonnement conditionnel**, ce qui avantage les modèles plus récents et plus expressifs.

Ce projet permet ainsi d’évaluer :
- l’impact du **SFT LoRA sur 7B**,
- le gain d’un **SFT plus doux (AI Studio)** sur 8B et 14B,
- la transférabilité du modèle dans des **workflows automatisés (n8n)**.

---

## 8. Statut du projet

- Dataset : **validé (qualité, équilibre, absence de doublons)**
- Prochaine étape : entraînements comparatifs (7B SFT LoRA vs 8B / 14B base et SFT)
- Objectif final : intégration opérationnelle dans un pipeline décisionnel agronomique

---

> Projet conçu dans une logique de **science appliquée**, combinant agronomie, météorologie et modèles de langage open-source.
