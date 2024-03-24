---
title: Porównanie wyników przetestowanych modeli uczenia maszynowego
---

## Wykorzystane modele

        - LogisticRegression,
        - RandomForestClassifier,
        - GradientBoostingClassifier,
        - AdaBoostClassifier,
        - ExtraTreesClassifier,
        - KNeighborsClassifier

```ml_res
select * from ml_results
```

<DataTable data={ml_res} rowNumbers=true>
  <Column id="Classifier" title="Classifier" />
  <Column id="Accuracy" title="Accuracy" contentType="colorscale" scaleColor="#a85ab8" align="center" />
  <Column id="Precision" title="Precision" contentType="colorscale" scaleColor="#e3af05" align="center" />
  <Column id="Recall" title="Recall" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="F1-score" title="F1-score" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="Build Time (seconds)" title="Build Time" contentType="colorscale" scaleColor="#c43957" align="center" />
</DataTable>

## Interpretacja
Z wyników w tabelce jednoznacznie wynika, że RandomForestClassifier jest modelem, który wykazuje najlepszą skutecznośc. Drugi w kolejności jest LogisticRegression. Z kolei najsłabiej wypada LogisticRegression. Jednocześnie wszystkie modele prezentują przeciętną zdolność do prawidłowego identyfikowania prawdziwie podejrzanych ofert i wymagają dalszych prac.
