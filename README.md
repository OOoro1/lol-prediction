# Prédiction de matchs pro de League of Legends

Prédire le vainqueur d'un match professionnel **avant qu'il commence**, et mesurer
si les probabilités annoncées sont fiables (calibration), pas seulement si le
côté prédit est le bon.

*English summary: see the end of this file.*

## Résultats principaux

| Modèle | Log loss (test 2025) |
|---|---|
| Prédiction constante (~52 % bleu) | ~0,692 |
| Elo (K=30, bonus côté bleu=25) | 0,6252 |
| Elo + ajustement par ligue (matchs internationaux) | À COMPLÉTER (K_lg=30) |

- Sur 3 496 parties de test, l'Elo bat nettement la prédiction constante.
- Sa courbe de calibration suit la diagonale : quand il annonce 80 %, la victoire
  arrive environ 79 % du temps.
- Une régression logistique (Elo, forme récente, repos) ne bat pas l'Elo de façon
  démontrable : différence de log loss de -0,0009, IC 95 % [-0,0039 ; +0,0020]
  (bootstrap, 3 301 parties).
- L'ajustement par ligue améliore surtout les matchs internationaux
  (À COMPLÉTER avec K_lg=30). Sur l'ensemble de 2025, l'IC 95 % de la différence
  frôle 0 : gain probable mais non démontré.

## Données

- Source : Oracle's Elixir, exports de matchs 2023, 2024 et 2025 (à télécharger
  soi-même dans `data/raw/`, non versionnés). Vérifier leurs conditions d'usage.
- 10 232 parties du 14/01/2023 au 09/11/2025, après filtrage.
- Périmètre : ligues majeures (LPL, LCK, LEC, LCS, LTA N/S, LLA, CBLOL, LCP, PCS,
  LJL, VCS, LCO) et compétitions internationales (MSI, WLDs, FST, EWC).
  Ligues de développement et académies exclues (ex. PCL).
- Taux de victoire du côté bleu : 52,9 %.

## Méthode

1. **Table une ligne par partie** construite à partir des lignes « équipe » du fichier.
2. **Elo** : chaque équipe démarre à 1500. La probabilité est calculée avant la
   partie, puis les notes sont mises à jour avec le résultat (pas de fuite de données).
3. **Validation temporelle** : réglage des hyperparamètres (K, bonus, K_lg) sur 2024,
   mesure finale une seule fois sur 2025. Aucun mélange aléatoire des dates.
4. **Métriques** : log loss, score de Brier, courbe de calibration ; accuracy en
   dernier (63-64 % pour l'Elo, contre 52 % pour « le bleu gagne »).
5. **Comparaison** des modèles sur les mêmes parties, avec bootstrap (1 000 tirages)
   pour l'incertitude.
6. **Ajustement par ligue** : sur les matchs internationaux, un décalage par région
   est appris, sans modifier les notes des équipes.

## Limites

- Les équipes de ligues différentes ne se rencontrent qu'en compétition
  internationale : l'Elo compare mal les régions ailleurs.
- Seulement 244 parties internationales en 2025 : résultats bruités.
- Les équipes qui changent de roster ne sont pas gérées (l'Elo suit l'équipe, pas les joueurs).
- La région des équipes est une simplification (dictionnaire manuel).
- Le retrait de la PCL et le choix du périmètre sont des décisions de ma part,
  pas testées par comparaison.

## Reproduire

- Créer l'environnement : `python -m venv .venv`, l'activer, puis
  `pip install -r requirements.txt`.
- Télécharger les CSV Oracle's Elixir 2023-2025 dans `data/raw/`.
- Exécuter `notebooks/02_multi_annees.ipynb` de haut en bas.

## Prochaines étapes

- Gradient boosting (LightGBM) et interprétation avec SHAP.
- Variables de draft (prédiction après le draft).
- Démo Streamlit.

## English summary

Pre-match win probability model for pro League of Legends (2023-2025 data).
An Elo baseline tuned on 2024 and evaluated once on 2025 reaches a log loss of
0.625 (constant baseline: 0.692) with well-calibrated probabilities. A logistic
regression with recent form and rest days shows no significant improvement
(bootstrap 95 % CI includes 0). League-level offsets improve predictions on
international matches, where cross-region comparison is the main weakness of Elo.
