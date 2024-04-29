---
title: Model LogisticRegression
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `LogisticRegression` zbudowanego przy użyciu [omawianych wcześniej](/analiza_modeli_ml/index.md) dwóch podejść do oznaczania danych.

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
   "accuracy":0.815052700922266,
   "precision":0.6741879494173072,
   "recall":0.3339064226943387,
   "f1-score":0.3939649517383563,
   "build_time_seconds":35
}
```

![Confusion Matrix](/assets/confusion_matrix_LogisticRegression-description.png)

![Precision Recall](/assets/precision_recall_curve_LogisticRegression-description.png)

### Model zbudowany przy użyciu danych oznaczanych za pomocą numeru VIN

```json
{
   "accuracy":0.9601449275362319,
   "precision":0.9917334669338678,
   "recall":0.8510318142734308,
   "f1-score":0.9160111059694586,
   "build_time_seconds":41
}
```

![Confusion Matrix](/assets/confusion_matrix_LogisticRegression-vin.png)


![Precision Recall](/assets/precision_recall_curve_LogisticRegression-vin.png)

## Interpretacja

Precision-Recall Curve

- Precision-Recall dla modelu opartego na opisach ogłoszeń pokazuje średnią precyzję (AP = 0.59). Krzywa mimo że jest płaska w przy niskim poziomie recall, szybko spada, gdy recall zaczyna rosnąć. Model nie jest w stanie utrzymać wysokiej precyzji dla wszystkich danych.
- Dla danych VIN, krzywa Precision-Recall wskazuje na znacznie wyższą średnią precyzję (AP = 0.96). Model bardzo dobrze radzi sobie ze zróżnicowanymi ofertami i poprawnie wykrywa podejrzane oferty, co jest wymagane przy wykrywaniu nadużyć.

Confusion Matrix

- Confusion Matrix dla modelu `LogisticRegression` zbudowanym na danych pochodzących z opisów ogłoszeń wykazuje wysoką liczbę prawdziwie negatywnych wyników (TN). Znaczna liczba fałszywie negatywnych wyników (FN) sugeruje, że model przegapia część podejrzanych ofert. Precyzja modelu jest umiarkowana, a recall jest niski, co oznacza, że model ma problemy z wykryciem wszystkich faktycznie podejrzanych ofert.
- Dla modelu zbudowanego na danych VIN, macierz pomyłek prezentuje znacznie lepsze wyniki z wysoką liczbą prawdziwie negatywnych i pozytywnych wyników oraz bardzo niską liczbą błędów pierwszego i drugiego rodzaju, co oznacza, że model dobrze rozpoznaje zarówno podejrzane, jak i niepodejrzane oferty.
