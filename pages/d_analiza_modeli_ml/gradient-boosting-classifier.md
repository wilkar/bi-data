---
title: Model GradientBoostingClassifier
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `GradientBoostingClassifier` zbudowanego przy użyciu [omawianych wcześniej](/d_analiza_modeli_ml/) dwóch podejść do oznaczania danych.

## Legenda

Confusion Matrix:
    - True-Negative (Lewy górny róg, TN): Liczba poprawnie sklasyfikowanych ofert jako "niepodejrzane".
    - False-Positivve (Prawy górny róg, FP): Liczba ofert błędnie zakwalifikowanych jako "podejrzane", które faktycznie są poprawne.
    - False-Negative (Lewy dolny róg, FN): Liczba ofert błędnie zakwalifikowanych jako "niepodejrzane", które są podejrzane.
    - True-Positive (Prawy dolny róg, TP): Liczba poprawnie sklasyfikowanych ofert jako "podejrzane".

Precision Recall:
    - Precision (positive predictive value): Określa jaka część wyników wskazanych przez klasyfikator jest faktycznie dodatnia.
    - Recall (true positive rate): Określa jaką część dodatnich wyników wykrył klasyfikator.
    - Area Under Curve: Im większe pole pod wykresem tym skuteczniejszy model.
    - Average precision: Średnia wartość precyzji dla różnych progów wyliczona jako ważona średnia zmian recall

Wzory:
    - Dokładność (Accuracy) -  (TP + TN) / (TP + TN + FP + FN)
    - Precision (Precision) -  (TP) / (TP + FP)
    - Czułość (Recall) - (TP) / (TP + FN)


### Model zbudowany przy użyciu danych oznaczanych za pomocą opisu ogłoszeń
```json
{
   "accuracy":0.7985013175230566,
   "precision":0.7186477644492911,
   "recall":0.16185680952965736,
   "f1-score":0.2642076776586148,
   "build_time_seconds":1722
}
```

![Confusion Matrix](/assets/confusion_matrix_GradientBoostingClassifier-description.png)

![Precision Recall](/assets/precision_recall_curve_GradientBoostingClassifier-description.png)

### Model zbudowany przy użyciu danych oznaczanych za pomocą numeru VIN

```json
{
   "accuracy":0.9601998243302591,
   "precision":0.9992372234935164,
   "recall":0.8447979363714532,
   "f1-score":0.9155503785672685,
   "build_time_seconds":914
}
```

![Confusion Matrix](/assets/confusion_matrix_GradientBoostingClassifier-vin.png)


![Precision Recall](/assets/precision_recall_curve_GradientBoostingClassifier-vin.png)

## Interpretacja

Precision-Recall Curve

- Precision-Recall dla modelu z danymi opisu wykazuje średnią precyzję (AP = 0.53), która jest znacząco niższa niż model oparty o numery VIN. Spadek krzywej pokazuje, że model jest mniej skuteczny przy większej czułości.

- Precision-Recall dla modelu z danymi VIN ma bardzo wysoką średnią precyzję (AP = 0.95). Wskazuje to bardzo dobrą skuteczność modelu w klasyfikacji podejrzanych ofert. Model jest w stanie utrzymać wysoką precyzję przez szerszy zakres czułości, co jest pożądanym stanem w przypadku wykrywania oszustw.

Confusion Matrix

- Confusion Matrix dla `GradientBoostingClassifier` z danych opartych na opisie ogłoszeń pokazuje zadowalającą zdolność do rozpoznawania ofert niepodejrzanych (TN), ale także znaczną liczbę fałszywie negatywnych wyników (FN), co oznacza, że niektóre podejrzane oferty nie zostały wykryte. Wysoka precyzja wskazuje, że kiedy model klasyfikuje oferty jako podejrzane, jest to często trafne, jednak niska czułość wskazuje na to, że wiele podejrzanych ofert pozostaje niezidentyfikowanych.
- Dla danych opartych na numerze VIN, Confusion Matrix demonstruje znacznie lepsze wyniki, z wysoką liczbą poprawnie sklasyfikowanych ofert. Niska liczba błędów sugeruje, że model ten jest efektywny w rozróżnianiu pomiędzy ofertami podejrzanymi a niepodejrzanymi.
