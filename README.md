# M6-B1 — Analyser la performance et détecter la dérive (Pyrenex, 3 mois post-prod)

> **Repo template GitHub.** Un·e du binôme clique **« Use this template »** →
> `M6-B1-pyrenex-drift-<binome>`, ajoute l'autre en collaborateur. Vous
> diagnostiquez la dérive du modèle déployé en M5 et rendez une note à Sophie Léger.

---

## 🧭 Votre brief en un coup d'œil

**Ce README est votre document de pilotage unique** — tout ce qu'il faut faire,
dans l'ordre, avec le bon appui. Les autres supports ont chacun un rôle précis :

| Support | Rôle |
|---|---|
| **Simplonline** | Le contrat : contexte client, livrables, critères de performance |
| **Ce README** | Le pilotage : quoi faire, quand, avec quel mini-cours |
| [`ressources/`](./ressources/) | Les 5 mini-cours d'appui (index dans [`ressources/README.md`](./ressources/README.md)) |
| **Discord `fil-M6`** | Annonces + questions |

### Les 2 jours sync (binôme)

| Quand | Tâche | Durée | Appui |
|---|---|---|---|
| Mardi 9h15 | 1. Tirage binôme + harmonisation de la reprise M5 | 30 min | — |
| Mardi 9h45 | 2. Exploration des données prod (référence vs 3 mois) | 1h15 | — |
| Mardi 11h00 | 3. Détection statistique PSI / KS / Chi² + `drift_summary.md` | 1h30 | [`01_PSI_KS_Chi2`](./ressources/01_PSI_KS_Chi2_essentiel.md) |
| Mardi 12h30 | 4. 🍽️ Déjeuner | 1h | — |
| Mardi 13h30 | 5. Calibration : reliability diagram (ECE en bonus ⭐) | 1h | [`03_Calibration_modele`](./ressources/03_Calibration_modele_essentiel.md) |
| Mardi 14h30 | 6. Diagnostic data drift vs concept drift → `diagnostic.md` | 1h30 | [`02_Data_drift_vs_concept_drift`](./ressources/02_Data_drift_vs_concept_drift_essentiel.md) |
| Mardi 16h45 | 7. Mur réflexif intermédiaire | 15 min | — |
| Mercredi 9h15 | 8. Extension du dashboard Grafana M5 (3 panels drift) | 45 min | [`05_Grafana_extension_dashboard`](./ressources/05_Grafana_extension_dashboard_essentiel.md) |
| Mercredi 10h00 | 9. Note de recommandation client | 1h15 | [`04_Note_recommandation_client`](./ressources/04_Note_recommandation_client_essentiel.md) |
| Mercredi 11h30 | 10. **Tour de table binômes** — diagnostics comparés | 1h | — |
| Mercredi 12h30 | 11. Mur réflexif final M6-B1 | 30 min | — |

### ✅ Checklist livrables (avant mercredi 12h30)

- [ ] `src/drift_detection.py` + `src/calibration.py` complétés — `pytest -q` vert
- [ ] PSI / KS / Chi² sur **toutes** les features pertinentes → `drift_summary.md`
- [ ] Diagnostic **chiffré et tranché** dans `diagnostic.md` (data vs concept
      drift — croisez features, AUC, calibration, temporalité)
- [ ] Note de recommandation **lisible par Sophie Léger** (pas ML), chiffrée,
      décision tranchée
- [ ] Dashboard M5 **étendu et provisionné** (3 panels drift, pas de dashboard neuf)
- [ ] Notebook exécuté top→bottom, commits `Co-authored-by:`, **journal de bord**

## 🚀 Démarrage

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pytest -q tests            # vert dès le clone (certains tests se débloquent avec vos TODO)
jupyter notebook notebooks/M6-B1_template.ipynb
```

> Variante `uv` : `uv venv .venv && source .venv/bin/activate` puis
> `uv pip install -r requirements.txt`.
> Dépannage : `No module named pip` → vous êtes dans un venv créé par `uv`,
> utilisez `uv pip install …` (pas `pip install`).

Les **données sont fournies** dans `data/` : `reference_set.csv` (baseline),
`prod_3months.csv` (3 mois de prod), `predictions_log.csv` (logs du modèle).

## 🧭 Ce que vous construisez

| # | À faire | Fichier | Mini-cours |
|---|---|---|---|
| 1 | Détection PSI / KS / Chi² | `src/drift_detection.py` | `01` |
| 2 | Calibration (reliability diagram ; ECE en bonus ⭐) | `src/calibration.py` | `03` |
| 3 | Analyse complète | `notebooks/M6-B1_template.ipynb` | `01`,`02`,`03` |
| 4 | Diagnostic data vs concept drift | `diagnostic.md` | `02` |
| 5 | Logique de remédiation | `src/recommendations.py` | `02`, `04` |
| 6 | Note de recommandation | `note_recommandation_TEMPLATE.md` | `04` |
| 7 | Extension dashboard Grafana | `grafana/dashboards/pyrenex_drift_TEMPLATE.json` | `05` |

## ⭐ Extension (non notée, si socle bouclé) — dater la dérive

Le PSI global compare 3 mois de prod d'un bloc : il **moyenne** la dérive.
Recalculez le PSI **par fenêtres de 2 semaines** (colonne `timestamp` de
`prod_3months.csv`) pour les 3 features les plus mouvantes, et tracez la
courbe PSI × temps. Vous devez pouvoir répondre : **quand** la dérive
a-t-elle commencé, feature par feature ? Est-elle **progressive ou
brutale** — et qu'est-ce que ça change pour votre diagnostic (axe
temporalité) et pour la fenêtre d'intervention recommandée à Sophie Léger ?
Ajoutez la courbe à votre notebook et 3 lignes de lecture dans
`note_recommandation.md`.

## 📚 Ressources

Voir [`./ressources/`](./ressources/) — 5 mini-cours + `liens_officiels.md`.
