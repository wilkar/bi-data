---
title: Model RandomForestClassifier
---


## Wstęp

Poniższy artykuł prezentuje szczegółową analizę modelu `RandomForestClassifier` zbudowanego przy użyciu [omawianych wcześniej](/d_analiza_modeli_ml/) dwóch podejść do oznaczania danych.

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
   "accuracy":0.8224912165129556,
   "precision":0.8314873417721519,
   "recall":0.2581358221785583,
   "f1-score":0.3939649517383563,
   "build_time_seconds":2748
}
```

![Confusion Matrix](/assets/confusion_matrix_RandomForestClassifier-description.png)

![Precision Recall](/assets/precision_recall_curve_RandomForestClassifier-description.png)

### Model zbudowany przy użyciu danych oznaczanych za pomocą numeru VIN

```json
{
   "accuracy":0.9056049626701801,
   "precision":0.9801866710332405,
   "recall":0.6433791917454859,
   "f1-score":0.77684770618389464,
   "build_time_seconds":1347
}
```

![Confusion Matrix](/assets/confusion_matrix_RandomForestClassifier-vin.png)


![Precision Recall](/assets/precision_recall_curve_RandomForestClassifier-vin.png)

## Interpretacja

Precision-Recall Curve

- Precision-Recall dla modelu z danymi opisowymi wykazuje średnią precyzję (AP = 0.63), która wskazuje na pewne wyzwania w różnicowaniu klas w danych tekstowych. Krzywa ta obniża się stopniowo, więc precyzja maleje w miarę dążenia do wykrycia większej liczby podejrzanych ofert.
- W przypadku modelu na danych VIN, krzywa PR wykazuje bardzo wysoką średnią precyzję (AP = 0.96). Świadczy to o skuteczności modelu w klasyfikacji i jego zdolności do utrzymania wysokiej precyzji nawet przy wyższych wartościach recall. Jest to sytuacja idealna, zwłaszcza w kontekstach wymagających precyzyjnego wykrywania nadużyć.

Confusion Matrix

- Confusion Matrix dla `RandomForestClassifier` z danych opartych na opisie ogłoszeń wskazuje na solidną zdolność modelu do identyfikacji niepodejrzanych ofert (TN), ale zwraca uwagę na stosunkowo dużą liczbę fałszywie negatywnych wyników (FN). Precyzja jest umiarkowana, ale czułość jest niska, co wskazuje na to, że model nie jest w stanie skutecznie wykryć wszystkich przypadków podejrzanych ofert.
- Dla danych VIN, macierz pomyłek pokazuje lepsze wyniki, z bardzo wysoką liczbą prawdziwie negatywnych wyników (TN) i względnie niską liczbą fałszywie negatywnych wyników (FN). Wysoka precyzja i umiarkowana czułość wskazują, że model jest bardziej kompetentny w wykrywaniu podejrzanych ofert przy jednoczesnym minimalizowaniu fałszywie pozytywnych wyników.
