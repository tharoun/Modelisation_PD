# Modélisation de la Probabilité de Défaut (PD)
### Pipeline complet— Approche Bâle III

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Production--ready-brightgreen)
![Framework](https://img.shields.io/badge/Framework-statsmodels%20%7C%20sklearn-orange)

---

## Table des matières

- [Présentation](#présentation)
- [Structure du pipeline](#structure-du-pipeline)
- [Arborescence du projet](#arborescence-du-projet)
- [Installation](#installation)
- [Données](#données)
- [Pipeline détaillé](#pipeline-détaillé)
- [Fonctions principales](#fonctions-principales)
- [Résultats attendus](#résultats-attendus)
- [Références](#références)
- [Auteurs](#auteurs)

---

## Présentation

Ce projet implémente un pipeline de modélisation de la **Probabilité de Défaut (PD)** conforme aux exigences réglementaires **Bâle III**. Il couvre l'ensemble des étapes de la segmentation : de l'exploration des données jusqu'à la **calibration finale par Classe Homogène de Risque (CHR)**, en passant par la sélection des variables, la modélisation par régression logistique, la construction d'une grille de score.

Deux méthodes de construction de la grille de score sont implémentées :

- **Méthode PDO** (Points to Double the Odds) - standard marché, lien direct avec la PD calibrée (Siddiqi, 2006)
- **Méthode Nexialog** - normalisation [0, 1000] sur coefficients logistiques, orientée communication opérationnelle (Nexialog Consulting, 2023)

La phase de calibrage produit les **PD calibrée par CHR** intégrant la LRA (Long Run Average), les ajustements contrats courts (test de Wilcoxon), les MoC A, B et C, et un seuil plancher réglementaire de 0,03 % (BCBS § 463).

> Le notebook est conçu pour être **pédagogique** : chaque étape est accompagnée de cellules Markdown expliquant le concept statistique, l'intérêt métier et la référence réglementaire associée.

---

## Structure du pipeline

```
Section 0   ─  Configuration & Imports
Section 1   ─  Chargement & Contrôles qualité des données
Section 2   ─  Analyse exploratoire (EDA)
                ├── 2.1  Évolution temporelle du taux de défaut
                └── 2.2  Statistiques descriptives & stabilité temporelle
Section 3   ─  Prétraitement & Feature Engineering
                ├── 3.1  Valeurs manquantes
                ├── 3.2  Détection des outliers (IQR)
                ├── 3.3  Modalités rares
                └── 3.4  Discrétisation supervisée (arbres de décision)
Section 4   ─  Sélection des variables
                ├── 4.0  WOE / IV (qualitatives & discrétisées)
                │         + AUC univariée / Mann-Whitney (continues brutes)
                ├── 4.1  Variance quasi-nulle
                └── 4.2  Corrélations — V de Cramér
Section 5   ─  Échantillonnage
                ├── 5.1  Découpage temporel train / test / hors du temps
                ├── 5.2  Représentativité — PSI
                └── 5.3  Rééquilibrage SMOTE
Section 6   ─  Modélisation — Régression Logistique
                ├── 6.1  Sélection forward BIC
                ├── 6.2  Sélection chi² (SelectKBest)
                └── 6.3  Estimation finale (statsmodels)
Section 7   ─  Évaluation & Validation
                ├── 7.1  Pouvoir discriminant (AUC, Gini, ROC)
                ├── 7.2  Validation hors du temps (OOT)
                ├── 7.3  Grille de score — Méthode PDO (0–1000)
                ├── 7.3b Grille de score — Méthode Nexialog (0–1000)
                ├── 7.4  Robustesse — K-Fold & Bootstrap
                └── 7.5  Sous-populations matérielles
Section 8   ─  Classes Homogènes de Risque (CHR)
                ├── 8.1  K-Means (méthode du coude native)
                ├── 8.2  Découpage par déciles
                ├── 8.3  Stabilité temporelle des CHR
                └── 8.4  Tests d'homogénéité (Welch) & hétérogénéité (Marascuilo)
Section 9   ─  Export intermédiaire des résultats
Section 10  ─  Calibrage de la PD
                ├── 10.0 Cadre réglementaire du calibrage
                ├── 10.1 LRA classique & LRA stable par fenêtre temporelle
                ├── 10.2 Contrats courts — Test de Wilcoxon
                ├── 10.3 Analyse de saisonnalité (OLS)
                ├── 10.4 LRA vs Taux PIT par CHR
                ├── 10.5 MoC A — Désalignements & données manquantes
                ├── 10.6 MoC B — Impact Covid-19 (EBA 2022)
                ├── 10.7 MoC C — Incertitude statistique (IC 95 %)
                └── 10.8 Estimation finale PD = max(Plancher, LRA + MoC A + MoC B + MoC C)
```

---

## Arborescence du projet

```
pd-modeling/
│
├── modelisation_PD.ipynb        # Notebook principal
├── README.md
├── requirements.txt
│
├── data/
│   └── BASE_MODELE_PD_FF.csv    # Base de modélisation (non fournie)
│
├── output/                      # Générés à l'exécution
│   ├── Base_Modele_PD_Enrichie.csv
│   ├── Base_Modele_PD_Enrichie_CHR.csv
│   ├── Base_Modele_PD_Final_Calibre.csv
│   ├── Grille_Score_0_1000.csv
│   ├── Grille_Score_0_1000.xlsx
│   ├── Grille_Score_Nexialog.csv
│   ├── Grille_Score_Nexialog.xlsx
│   ├── Scores_Individuels_Nexialog.csv
│   ├── Seuils_Discretisation.xlsx
│   ├── Taux_Indice_trim.csv
│   ├── WOE_detail.csv
│   ├── IV_synthese.csv
│   ├── Metriques_Variables_Continues.csv
│   ├── V_de_Cramer_Paires_Arbitrage.xlsx
│   ├── Performances_Comparatif.csv
│   ├── logit_results_complet.xlsx
│   ├── Calibrage_PDCalibreeParCHR.csv
│   ├── Calibrage_MoCCdetail.csv
│   ├── Calibrage_MoCAdetail.csv
│   ├── Calibrage_MoCBdetail.csv
│   ├── Calibrage_LRAClassiqueParFenetre.csv
│   ├── Calibrage_LRAStableParFenetre.csv
│   ├── Calibrage_TestWilcoxonContratsCourts.csv
│   ├── Calibrage_TestSaisonnaliteOLS.csv
│   ├── analyse_taux_defaut.png
│   ├── roc_train.png
│   ├── roc_test.png
│   ├── roc_hors_du_temps.png
│   ├── grille_score_visualisation.png
│   ├── grille_nexialog_variables.png
│   ├── nexialog_densite_conditionnelle.png
│   ├── nexialog_defaut_vingtile.png
│   ├── kmeans_elbow.png
│   ├── calibrage_lra_vs_pit.png
│   ├── calibrage_moc_c.png
│   ├── calibrage_pd_finale.png
│   ├── WOE_par_variable.png
│   ├── WOE_stabilite_temporelle.png
│   └── IV_barchart.png
│
└── references/
    └── Nexialog-scoring-ML-interpretable-2023.pdf
```

---

## Installation

### Prérequis

- Python ≥ 3.12
- uv, pip ou conda

### Cloner le dépôt

```bash
git clone https://github.com/tharoun/Modelisation_PD.git
cd Modelisation_PD
```

### Créer un environnement virtuel

```bash
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
.venv\Scripts\activate           # Windows
```

### Installer les dépendances

```bash
pip install -r requirements.txt
```

### Lancer le notebook

```bash
jupyter notebook modelisation_PD.ipynb
```

### `requirements.txt`

```
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
scipy>=1.11
scikit-learn>=1.3
statsmodels>=0.14
imbalanced-learn>=0.11
ppscore>=1.3
openpyxl>=3.1
rich>=13.0
jinja2>=3.1
gdown>=4.7
jupyter>=1.0
```
> `gdown` permet le téléchargement optionnel depuis Google Drive (cellule commentée en Section 1).

---

## Données

### Format attendu

Le fichier `BASE_MODELE_PD_FF.csv` doit être placé dans le répertoire `data/`
avec les caractéristiques suivantes :

| Propriété | Valeur attendue |
|---|---|
| Encodage | UTF-8 |
| Séparateur | `,` |
| Granularité | Contrat ou client |
| Périodicité | Trimestrielle (mois = 3, 6, 9, 12) |
| Profondeur historique | ≥ 5 ans (exigence Bâle III§ 463) |
| Variable cible | `DDefaut_NDB` — binaire (0 = sain, 1 = défaut) |
| Date d'arrêté | `ARRETE` — format `YYYYMM` (ex. `202203`) |

### Contrôles automatiques à l'exécution

Le notebook vérifie automatiquement en Section 1 :

- ✅ Périodicité strictement trimestrielle
- ✅ Profondeur historique ≥ 5 ans consécutifs
- ✅ Variable cible strictement binaire (0/1)

---

## Pipeline détaillé

### Sélection des variables — quatre niveaux complémentaires

| Méthode | Type de variable | Ce qu'elle filtre |
|---|---|---|
| **IV** (≥ 0.02) | Qualitatives & discrétisées | Absence de lien avec la cible |
| **AUC univariée + Mann-Whitney** | Continues brutes | Absence de pouvoir discriminant |
| **V de Cramér** (< 0.80) | Toutes | Redondance entre prédicteurs |
| **Forward BIC** | Toutes | Variables n'améliorant pas le modèle |

### Méthode du coude K-Means (Section 8.1)

La détection automatique du nombre optimal de clusters K utilise
la **distance perpendiculaire maximale** à la droite reliant K_min et K_max
— algorithme identique à `KElbowVisualizer` (yellowbrick), implémenté nativement
pour éviter les conflits de version avec scikit-learn.

### Grilles de score — comparaison des deux méthodes

| Dimension | Méthode PDO (Siddiqi) | Méthode Nexialog |
|---|---|---|
| Base du calcul | Coefficients → log-odds | Écarts entre coefficients |
| Paramètres | PDO=20, Score ancre=600, Odds=50:1 | Score max = 1 000 |
| Interprétabilité | Score lié à une PD via les odds | Contribution par variable visible |
| Usage recommandé | Calibration IRB, reporting réglementaire | Communication métier, scorecard papier |

### Calibrage de la PD (Section 10)

La PD finale par CHR est calculée selon la formule réglementaire :

```
PD_calibrée = max(PD_plancher, LRA + Ajustement + MoC_A + MoC_B + MoC_C)
```

| Composante | Description | Référence |
|---|---|---|
| **LRA** | Long Run Average Default Rate — moyenne des taux PIT sur la fenêtre retenue | BCBS § 461–463 |
| **Ajustement** | Correction contrats courts si test de Wilcoxon significatif | EBA GL/2017/16 § 82 |
| **MoC A** | Désalignements modélisation / application | EBA GL/2017/16 § 96 |
| **MoC B** | Impact des mesures Covid-19 (moratoires) | EBA, juin 2022 |
| **MoC C** | Incertitude statistique — borne sup. IC 95 % sur les taux PIT | EBA GL/2017/16 § 105 |
| **PD plancher** | 0,03 % — seuil réglementaire retail | BCBS (2006) § 463 |

La monotonicité croissante des PD par CHR est vérifiée automatiquement à l'issue du calibrage.

---

## Fonctions principales

### Section 1 — Contrôles qualité

| Fonction | Description |
|---|---|
| `rapport_qualite(df)` | Inventaire types, missing, unicité |
| `controle_periodicite(df)` | Vérifie la périodicité trimestrielle |
| `controle_profondeur_historique(df)` | Vérifie ≥ 5 ans (Bâle III§ 463) |
| `controle_variable_cible(df)` | Vérifie la binarité de la cible |

### Section 2 — EDA

| Fonction | Description |
|---|---|
| `calculer_stats_defaut(df)` | Taux de défaut & indices base 100 par trimestre |
| `plot_evolution_taux_defaut(df, cible, variables)` | Stabilité temporelle par modalité (usage multiple) |

### Section 3 — Prétraitement

| Fonction | Description |
|---|---|
| `preprocess_missing(df, ...)` | Indicateur NR + médiane / modalité 'NR' |
| `detection_outliers_iqr(df, ...)` | IQR avec seuil d'alerte configurable |
| `detection_modalites_rares(df, ...)` | Modalités < 5 % de la population |
| `discretiser_par_arbre(df, ...)` | Discrétisation supervisée (DecisionTree depth=2) + export Excel |

### Section 4 — Sélection

| Fonction | Description |
|---|---|
| `calculer_woe_iv(df, variables)` | WOE & IV pour variables qualitatives/discrétisées |
| `calculer_woe_iv_continu(df, variables)` | WOE & IV après binning en vingtiles (variables continues) |
| `calculer_auc_univariee(df, variables)` | AUC univariée par variable continue |
| `calculer_correlation_pbiseriale(df, ...)` | Corrélation point-bisériale |
| `calculer_mann_whitney(df, variables)` | Test non paramétrique de séparation |
| `analyser_variables_numeriques(df, ...)` | Synthèse variance / skewness / corrélation |
| `cramers_v(x, y)` | V de Cramér corrigé (Bergsma & Wicher, 2013) |
| `matrice_cramer(df, variables, seuil)` | Matrice + paires corrélées |
| `plot_cramer_heatmap(cramer_df)` | Heatmap triangulaire |

### Section 5 — Échantillonnage

| Fonction | Description |
|---|---|
| `sub_psi(e_perc, a_perc)` | Contribution élémentaire au PSI |
| `psi_quantitative(df_ref, df_test, ...)` | PSI variables quantitatives |
| `psi_categorielle(df_ref, df_test, ...)` | PSI variables qualitatives |

### Section 6 — Modélisation

| Fonction | Description |
|---|---|
| `forward_selection_bic(X, y)` | Sélection forward par critère BIC |
| `export_logit_results(model_results)` | Export coefficients + résumé modèle (Excel, 2 onglets) |

### Section 7 — Évaluation

| Fonction | Description |
|---|---|
| `evaluer_performance(y_true, y_pred_prob, nom)` | AUC, Gini, matrice de confusion, courbe ROC |
| `evaluer_sous_population(X_test, ...)` | Évaluation sur un sous-segment filtré |
| `extraire_coefficients(model_results)` | Coefficients, OR, IC 95 % |
| `construire_grille_normalisee(...)` | Grille PDO normalisée [0, 1000] |
| `extraire_coefs_binaires(model_results, ...)` | Coefficients → structure (variable, modalité) |
| `calculer_notes_nexialog(df_coef)` | Équation 16 Nexialog |
| `enrichir_grille_nexialog(df_grille, ...)` | Ajout statistiques observées |
| `calculer_score_individuel_nexialog(X, ...)` | Score = somme des notes actives |
| `plot_grille_nexialog(grille)` | Visualisation double axe note/taux de défaut |
| `plot_densite_conditionnelle(X_scored)` | Densité conditionnelle (Figure 6 Nexialog) |
| `plot_defaut_par_vingtile(X_scored)` | Taux de défaut par vingtile (Figure 7 Nexialog) |
| `verifier_monotonie_nexialog(grille)` | Contrôle note ↓ quand risque ↑ |
| `_fit_logit_select(X_tr, y_tr, X_te)` | Logit + SelectKBest pour K-Fold / Bootstrap |

### Section 8 — CHR

| Fonction | Description |
|---|---|
| `plot_elbow_kmeans(data, k_min, k_max)` | Méthode du coude native (sans yellowbrick) |
| `plot_stabilite_classes(df, cible, classe_col)` | Stabilité temporelle avec IC 90 % |
| `test_homogeneite_welch(df, variable, classe_col)` | Z-test de Welch intra-classe |
| `marascuilo_test(data, group_col, target_col)` | Test post-hoc d'hétérogénéité inter-classes |

### Section 10 — Calibrage

| Fonction | Description |
|---|---|
| `calculer_lra_classique(df, ...)` | LRA tous contrats par CHR et par fenêtre temporelle |
| `calculer_lra_stable(df, ...)` | LRA clients présents sur toute la période (exclusion contrats courts) |
| `plot_lra_vs_pit(df, ...)` | Évolution taux PIT vs LRA + IC 95 % par CHR |
| `calculer_moc_c(df, ...)` | MoC C — IC bilatéral 95 % sur taux PIT annuels |
| `plot_moc_c(df_moc_c, ...)` | Visualisation LRA + MoC C avec errorbars IC |
| `estimer_pd_finale(df_moc_c, df_moc_a, df_moc_b, df_wilcoxon, ...)` | PD calibrée finale par CHR avec vérification de monotonicité |
| `plot_pd_finale(df_pd, ...)` | Décomposition LRA / MoC A / B / C + LRA vs PD calibrée |

---

## Résultats attendus

### Performances types (dépendent des données)

| Échantillon | AUC attendu | Gini attendu |
|---|---|---|
| Train (SMOTE) | 0.75 – 0.90 | 0.50 – 0.80 |
| Test | 0.70 – 0.85 | 0.40 – 0.70 |
| Hors du temps | 0.68 – 0.83 | 0.36 – 0.66 |

> Une dégradation AUC(Test) - AUC(OOT) > 5 points est un signal d'instabilité temporelle.

### Fichiers exportés

| Fichier | Contenu |
|---|---|
| `Base_Modele_PD_Enrichie.csv` | Base avec variables discrétisées |
| `Base_Modele_PD_Enrichie_CHR.csv` | Base enrichie avec scores et clusters |
| `Base_Modele_PD_Final_Calibre.csv` | Base finale avec PD calibrée par CHR |
| `Grille_Score_0_1000.csv/xlsx` | Grille PDO par (variable, modalité) |
| `Grille_Score_Nexialog.csv/xlsx` | Grille Nexialog par (variable, modalité) |
| `Scores_Individuels_Nexialog.csv` | Score individuel Nexialog sur l'échantillon test |
| `WOE_detail.csv` | WOE par (variable, modalité) |
| `IV_synthese.csv` | IV global par variable, trié |
| `V_de_Cramer_Paires_Arbitrage.xlsx` | Paires corrélées + décision de suppression |
| `Performances_Comparatif.csv` | AUC/Gini train, test, OOT |
| `Seuils_Discretisation.xlsx` | Seuils des arbres par variable continue |
| `logit_results_complet.xlsx` | Résumé modèle + coefficients (2 onglets) |
| `Calibrage_PDCalibreeParCHR.csv` | PD calibrée finale par CHR (LRA + MoC A/B/C) |
| `Calibrage_MoCCdetail.csv` | Détail MoC C par CHR et IC 95 % |
| `Calibrage_MoCAdetail.csv` | Détail MoC A (saisie experte) |
| `Calibrage_MoCBdetail.csv` | Détail MoC B (correction Covid) |
| `Calibrage_LRAClassiqueParFenetre.csv` | LRA classique multi-fenêtres |
| `Calibrage_LRAStableParFenetre.csv` | LRA stable (contrats longs uniquement) |
| `Calibrage_TestWilcoxonContratsCourts.csv` | Résultats test de Wilcoxon |
| `Calibrage_TestSaisonnaliteOLS.csv` | Analyse de saisonnalité des taux PIT |

---

## Références

### Ouvrages de référence

| Référence | Usage dans le projet |
|---|---|
| **Siddiqi, N. (2006/2017).** *Credit Risk Scorecards.* Wiley. | Méthode PDO, scorecard scaling, WOE/IV |
| **Thomas, L.C., Edelman, D.B., Crook, J.N. (2002/2017).** *Credit Scoring and Its Applications.* SIAM. | Fondements théoriques de la régression logistique appliquée au scoring |
| **Anderson, R. (2007).** *The Credit Scoring Toolkit.* Oxford University Press. | WOE/IV comme distance de Kullback-Leibler, discrétisation, CHR |
| **Refaat, M. (2011).** *Credit Risk Scorecard: Development and Implementation Using SAS.* | Implémentation algorithmique |

### Documents réglementaires

| Document | Usage dans le projet |
|---|---|
| **BCBS (2006).** *Basel II: International Convergence — Comprehensive Version.* BIS. [lien](https://www.bis.org/publ/bcbs128.pdf) | Profondeur historique ≥ 5 ans (§ 463), exigences IRB (§ 443–460), seuil plancher PD |
| **EBA (2017).** *Guidelines on PD estimation, LGD estimation (EBA/GL/2017/16).* | Homogénéité/hétérogénéité des CHR (§§ 96–105), contrats courts (§ 82), MoC A/B/C |
| **EBA (2022).** *Report on the impact of Covid-19 on IRB models.* | MoC B — correction mesures compensatoires Covid |
| **BCE (2024).** *ECB Guide to Internal Models.* [lien](https://www.bankingsupervision.europa.eu) | Validation OOT, robustesse à l'échantillonnage |

### Working papers

| Document | Usage dans le projet |
|---|---|
| **Nexialog Consulting (2023).** *Scoring d'octroi par Machine Learning interprétable ?* | Méthode Nexialog (équation 16), discrétisation en vingtiles, CHR par CAH |
| **Bergsma & Wicher (2013).** *A consistent test of independence based on a sign covariance.* | Correction de biais du V de Cramér |

---

## Étapes non implémentées — perspectives

Les étapes suivantes sont signalées dans le notebook mais non implémentées.
Elles constituent les extensions naturelles vers un modèle IRB complet :

| Étape | Description | Référence |
|---|---|---|
| **Scorecard en points entiers** | Transformation PDO → points entiers (PDO method) | Siddiqi (2006) Ch. 8 |
| **Test de Hosmer-Lemeshow** | Calibration globale du modèle logistique | Thomas et al. (2002) |
| **Backtesting** | Comparaison PD prédites vs taux observés | EBA GL/2017/16 § 115–120 |
| **Modèles challenger** | Random Forest / XGBoost + SHAP values | Nexialog (2023) Section 4 |
| **Calcul RWA** | Exigences en fonds propres f(PD, LGD, EAD) | BCBS (2006) § 272–279 |
| **Provisions IFRS 9** | ECL = PD × LGD × EAD, staging S1/S2/S3 | IFRS 9, § 5.5 |

---

## Auteurs

**Harouna TRAORE**, Analyste quantitatif & Data scientist  
Projet développé dans un contexte de formation et de modélisation PD appliquée au risque de crédit retail.

---

*Dernière mise à jour : avril 2026*
