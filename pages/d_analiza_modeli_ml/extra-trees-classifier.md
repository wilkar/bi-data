---
title: Model ExtraTreesClassifier
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `ExtraTreesClassifier` zbudowanego przy użyciu [omawianych wcześniej](/d_analiza_modeli_ml/) dwóch podejść do oznaczania danych.

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
   "accuracy":0.883893280632411,
   "precision":0.962449872402479,
   "recall":0.5674978503869303,
   "f1-score":0.7139959432048681,
   "build_time_seconds":2197
}
```

![Confusion Matrix](/assets/confusion_matrix_ExtraTreesClassifier-description.png)

![Precision Recall](/assets/precision_recall_curve_ExtraTreesClassifier-description.png)

### Model zbudowany przy użyciu danych oznaczanych za pomocą numeru VIN

```json
{
   "accuracy":0.9580314009661836,
   "precision":0.9905362776025236,
   "recall":0.8437231298366294,
   "f1-score":0.9112542805734517,
   "build_time_seconds":262
}
```

![Confusion Matrix](/assets/confusion_matrix_ExtraTreesClassifier-vin.png)


![Precision Recall](/assets/precision_recall_curve_ExtraTreesClassifier-vin.png)

## Interpretacja

Precision-Recall Curve

- Precision-Recall dla modelu opartego na opisie ogłoszeń pokazuje niższą średnią precyzję (AP = 0.65). Sugeruje to na większe wyzwanie w odróżnianiu pomiędzy klasami w danych tekstowych. Jakość klasyfikacji obniża się wraz ze wzrostem czułości. Model zaczyna wtedy klasyfikować więcej ofert jako podejrzane, ale kosztem większej liczby popełnianych błędów.
- W przypadku danych VIN, Precision-Recall ma wyższą średnią precyzję (AP = 0.93), co świadczy o lepszej ogólnej wydajności modelu. Model utrzymuje wysoką precyzję nawet przy wzroście czułości, co jest idealnym scenariuszem.

Confusion Matrix

- Confusion Matrix dla modelu `ExtraTreesClassifier` używającego danych opartych na opisie ogłoszeń wykazuje wysoką liczbę prawdziwie negatywnych wyników (TN). Model dobrze radzi sobie z identyfikacją niepodejrzanych ofert, ale liczba fałszywie negatywnych wyników jest wysoka, więc model nie poradził sobie z wykryciem wszytkich podejrzanych ogłoszeń.
- Dla danych opartych na numerze VIN, Confusion Matrix wskazuje na znacznie lepsze wyniki, z bardzo wysoką liczbą prawdziwie negatywnych (TN) i pozytywnych wyników (TP). Błąd pierwszego rodzaju (FP) jest bardzo niski, co jest pożądane w rozpatrywanym problemie, ponieważ koszt błędnego oznaczenia oferty może być bardzo wysoki z perspektywy biznesowej.
