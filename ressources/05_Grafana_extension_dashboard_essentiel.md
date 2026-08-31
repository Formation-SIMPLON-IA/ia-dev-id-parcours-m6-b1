# Étendre un dashboard Grafana existant — Mini-cours

> Brief associé : M6-B1
> Durée de lecture : ~20 min
> Pré-requis : Grafana provisionné en M5 (mini-cours M5 `04`)

## Pourquoi cette techno ?

Vous avez déjà un dashboard de prod en M5. En M6, on ne crée **pas** un nouveau
dashboard from scratch : on **étend l'existant** avec des panels de suivi de
dérive. C'est le réflexe pro — capitaliser sur l'outillage en place plutôt que
multiplier les tableaux de bord orphelins. Le dashboard reste **provisionné**
(versionné en JSON, chargé au démarrage), pas bricolé à la main.

## Concepts clés

- **Provisioning** : le dashboard vit en JSON dans le repo (`grafana/dashboards/`)
  et est chargé automatiquement — un panel ajouté à la main dans l'UI est perdu
  au prochain `compose down` s'il n'est pas exporté.
- **Étendre = ajouter des panels** au JSON (ou un nouveau JSON provisionné par le
  même provider), pas refaire l'UI.
- **Métriques offline vs live** : le PSI se calcule en **batch** (notebook/script)
  ; pour l'afficher en continu, on **pousse** une gauge (`pyrenex_feature_psi`)
  vers Prometheus. Le live (distribution des probas) utilise les métriques déjà
  exposées par le service `model` (M5).
- ⚠️ **Gauge vs Histogram — la requête n'est pas la même.** `pyrenex_feature_psi`
  est une **gauge** : on l'interroge directement. `pyrenex_prediction_proba` est
  un **Histogram** (M5, mini-cours 02) : il n'existe pas tel quel dans
  Prometheus, il génère les séries `_bucket` / `_sum` / `_count`. Pour un
  quantile, il faut passer par `histogram_quantile()` **sur les buckets** :

  ```promql
  # ✅ médiane des probabilités prédites sur 5 min
  histogram_quantile(0.5, sum by (le) (rate(pyrenex_prediction_proba_bucket[5m])))

  # ❌ ne fonctionne pas : la métrique brute n'est pas une série interrogeable
  histogram_quantile(0.5, pyrenex_prediction_proba)
  ```

  Le `sum by (le)` agrège les buckets sur toutes les instances ; le `rate()`
  est nécessaire parce que les buckets sont des **compteurs cumulatifs**.
- **Seuils colorés** : configurez les `thresholds` du panel PSI (vert < 0.1,
  orange < 0.25, rouge ≥ 0.25) pour une lecture immédiate.
- **uid de datasource** : référencez la datasource Prometheus par son `uid`
  (comme en M5) pour que le provisioning soit reproductible.

## Exemple minimal qui tourne

```json
// panel ajouté au dashboard, lecture d'une gauge PSI poussée par un batch
{
  "title": "PSI par feature", "type": "timeseries",
  "datasource": { "type": "prometheus", "uid": "prometheus" },
  "fieldConfig": { "defaults": { "thresholds": { "steps": [
    { "color": "green", "value": null }, { "color": "orange", "value": 0.1 },
    { "color": "red", "value": 0.25 } ] } } },
  "targets": [ { "expr": "pyrenex_feature_psi", "legendFormat": "{{feature}}" } ]
}
```

## Exercice guidé

À partir du dashboard M5 :
1. Ajoutez **3 panels**, chacun répondant à une question opérationnelle :

| Panel | Question à laquelle il répond | Source |
|---|---|---|
| PSI par feature | *Les entrées changent-elles ?* | gauge `pyrenex_feature_psi` (poussée en batch) |
| F1 macro sur 12 semaines | *La performance se dégrade-t-elle ?* | calcul offline (labels requis, en différé) |
| Distribution des probabilités | *Les prédictions évoluent-elles ?* | `histogram_quantile()` sur `pyrenex_prediction_proba_bucket` (M5) |

   Un panel qui n'est relié à **aucune question** et à **aucune action du
   runbook** n'a pas sa place sur le dashboard.
2. Versionnez le JSON dans `grafana/dashboards/pyrenex_drift.json`.
3. `docker compose up` doit le charger **sans import manuel**.

## Pièges fréquents

| Piège | Conséquence |
|---|---|
| Créer un nouveau dashboard from scratch | Hors-sujet : on **étend** l'existant |
| Bricoler dans l'UI sans exporter le JSON | Perdu au redémarrage |
| Afficher le PSI sans le pousser à Prometheus | Panel « No data » (métrique offline) |
| `histogram_quantile()` sur la métrique sans `_bucket` | Panel vide / erreur PromQL |
| Oublier `rate()` sur les buckets | Quantile faux (buckets = compteurs cumulatifs) |
| `uid` de datasource non référencé | « Datasource not found » au provisioning |
| Panel sans question ni action associée | Dashboard décoratif, jamais consulté en astreinte |

| Symptôme | Cause probable |
|---|---|
| Panels disparaissent au restart | JSON non versionné/provisionné |
| PSI « No data » | gauge non poussée vers Prometheus |
| Quantile de proba vide ou aberrant | `_bucket` oublié, ou `rate()` manquant |
| « Datasource not found » | mauvais `uid` dans le panel |

## Pour aller plus loin

- Grafana — Provisioning : https://grafana.com/docs/grafana/latest/administration/provisioning/
- Prometheus — histogram_quantile : https://prometheus.io/docs/practices/histograms/

## Vérification (checklist apprenant)

- [ ] J'ai **étendu** le dashboard M5 (pas créé un nouveau).
- [ ] Mes 3 panels sont versionnés en JSON et provisionnés.
- [ ] Les seuils PSI sont colorés (vert/orange/rouge).
- [ ] `docker compose up` charge le dashboard sans clic.
- [ ] Je distingue métrique offline (PSI poussé) et live (proba M5).
- [ ] Je distingue **gauge** (interrogée directement) et **Histogram**
      (`_bucket` + `rate()` + `histogram_quantile()`).
- [ ] Chacun de mes panels répond à une **question** et renvoie à une **action**
      du runbook.

> 💡 **Récap** : on **étend** le dashboard M5 (pas un nouveau), on **provisionne**
> le JSON (pas l'UI), et on distingue métrique **offline** (PSI poussé en gauge) de
> métrique **live** (proba exposée par le service `model`). Le réflexe pro : capitaliser
> sur l'outillage en place plutôt que multiplier les tableaux de bord orphelins.
