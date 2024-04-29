---
title: Model KNeighborsClassifier
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `KNeighborsClassifier` zbudowanego przy użyciu [omawianych wcześniej](/analiza_modeli_ml/index.md) dwóch podejść do oznaczania danych.

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
   "accuracy":0.7936155028546333,
   "precision":0.5664678312739667,
   "recall":0.326538130909984,
   "f1-score":0.4142712471761315,
   "build_time_seconds":1356
}
```

![Confusion Matrix](/assets/confusion_matrix_KNeighborsClassifier-description.png)

![Precision Recall](/assets/precision_recall_curve_KNeighborsClassifier-description.png)

### Model zbudowany przy użyciu danych oznaczanych za pomocą numeru VIN

```json
{
   "accuracy":0.8159036012296882,
   "precision":0.6400301951903375,
   "recall":0.6389621575065941,
   "f1-score":0.6389621575065941,
   "build_time_seconds":611
}
```

![Confusion Matrix](/assets/confusion_matrix_KNeighborsClassifier-vin.png)


![Precision Recall](/assets/precision_recall_curve_KNeighborsClassifier-vin.png)

## Interpretacja

Precision-Recall Curve

- Precision-Recall dla modelu opartego na opisie ogłoszeń wykazuje średnią precyzję (AP = 0.45). Spadek krzywej jest stosunkowo stały, jest typowe dla modeli KNN w przypadku rozróżniania złożonych wzorców w danych tekstowych.
- Precision-Recall dla modelu z danymi VIN pokazuje średnią precyzję na poziomie 0.67. Widoczne są większe stopnie schodkowości w krzywej, co jest charakterystyczne dla modeli KNN. Może to również, że model ma trudności z ogólnym rozróżnianiem między klasami w zbiorze danych.

Confusion Matrix

- Confusion Matrix dla `KNeighborsClassifier` z danych opartych na opisie ogłoszeń pokazuje, że model ma przeciętną zdolność do prawidłowej klasyfikacji klas, co wskazuje na stosunkowo równomierną rozkład błędów między fałszywie negatywne (FN) a fałszywie pozytywne (FP) wyniki. Wskazuje to, że model może mieć problemy z rozróżnieniem między klasami.
- Dla danych VIN Confusion Matrix pokazuje lepszą dokładności i zwiększoną precyzję do około oraz znaczący wzrost czułości. Jednak wysoka wartość recall w połączeniu z umiarkowaną precyzją może jednak prowadzić do stosunkowo dużej liczby fałszywie pozytywnych wyników w klasyfikowaniu ofert.
