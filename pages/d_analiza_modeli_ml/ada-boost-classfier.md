---
title: Model AdaBoostClassifier
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `AdaBoostClassifier` zbudowanego przy użyciu [omawianych wcześniej](/d_analiza_modeli_ml/) dwóch podejść do oznaczania danych.

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
   "accuracy":0.7875494071146245,
   "precision":0.5643153526970954,
   "recall":0.21711899791231734,
   "f1-score":0.31358637814827955,
   "build_time_seconds":184
}
```

![Confusion Matrix](/assets/confusion_matrix_AdaBoostClassifier-description.png)

![Precision Recall](/assets/precision_recall_curve_AdaBoostClassifier-description.png)

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

![Confusion Matrix](/assets/confusion_matrix_AdaBoostClassifier-vin.png)


![Precision Recall](/assets/precision_recall_curve_AdaBoostClassifier-vin.png)

## Interpretacja

Precision-Recall Curve

- Model `AdaBoostClassifier` używający danych oznaczonych numerami VIN wykazuje bardzo wysoką średnią precyzję. Oznacza to, że bardzo dobrze poprawnie klasyfikuje podejrzane oferty jako podejrzane i mało prawdopodobne jest oznacznie uczciwej oferty jako podejrzanej.
- Model opearty na danych z opisu ogłoszeń ma znacznie niższą średnią precyzję (AP = 0.47).Oznacza to, że ma trudności z różnicowaniem między klasami i częściej popełnia błędy pierwszego rodzaju (FP).

Confusion Matrix

- Macierz pomyłek dla modelu opartego na VIN wykazuje znaczną liczbę prawidłowo sklasyfikowanych przypadków jako podejrzane (TP) i niewiele błędów drugiego rodzaju (FN), co jest pożądaną cechą w systemach detekcji nadużyć.
- W przypadku modelu opartego na opisach ogłoszeń, znaczna liczba podejrzanych ofert nie została wykryta (FN), co oznacza, że system mógłby przepuszczać faktycznie podejrzane oferty.
