---
title: Model AdaBoostClassifier
---


## Wyniki

```json
    {
        "AdaBoostClassifier accuracy": 0.8080234475178604
        "AdaBoostClassifier precision": 0.5448275862068965
        "AdaBoostClassifier recall": 0.09278834860230209
        "AdaBoostClassifier F1-score": 0.1585708550782818
        "AdaBoostClassifier build time": 139 seconds
    }
```


![Confusion Matrix](/confusion-matrix-ada-boost-classifier.png)
Legenda:
    - True-Negative (Lewy górny róg, TN): Liczba poprawnie sklasyfikowanych ofert jako "niepodejrzane".
    - False-Positivve (Prawy górny róg, FP): Liczba ofert błędnie zakwalifikowanych jako "podejrzane", które faktycznie są poprawne.
    - False-Negative (Lewy dolny róg, FN): Liczba ofert błędnie zakwalifikowanych jako "niepodejrzane", które są podejrzane.
    - False-Positive (Prawy dolny róg, TP): Liczba poprawnie sklasyfikowanych ofert jako "podejrzane".

![Precision Recall](/precision-recall-ada-boost-classfier.png)
Legenda:
    - Precision (precyzja): Proporcja poprawnie zidentyfikowanych pozytywnych przypadków (TP) do wszystkich przypadków zidentyfikowanych jako pozytywne (TP + FP).
    - Recall (czułość): Proporcja poprawnie zidentyfikowanych pozytywnych przypadków (TP) do wszystkich faktycznie pozytywnych przypadków (TP + FN).
    - Area Under Curve (AUC): Ogólnie, im większe pole pod krzywą, tym lepsza zdolność modelu do klasyfikacji.
    - Średnia precyzja (Average Precision - AP): Średnia wartość precyzji dla różnych progów, obliczona jako ważona średnia zmian recall.

Wzory:
    - Dokładność (Accuracy) -  (TP + TN) / (TP + TN + FP + FN)
    - Precision (Precision) -  (TP) / (TP + FP) 
    - Czułość (Recall) - (TP) / (TP + FN)
## Interpretacja

Confusion matrix:
    - Średnia precyzja (Average precision) o wartości 0.53% wskazuje przeciętną wydajność modelu
    - Dokładność (Accuracy) modelu 80% sugeruje, że model podejmuje poprawną decyzję w większości przypadków.
    - Precision (Precision) modelu 54.4% sugeruje, że model przewiduje ofertę jako "podejrzaną", jest ona faktycznie podejrzana tylko w połowie przypadków.
    - Czułość (Recall) modelu 9% pokazuje, że model identyfikuje tylko około jedną na dziesięć faktycznie "podejrzanych" ofert. Jest to najgorszy wynik spośród badanych modeli.
    - F1-score na poziomie 15.8% wskazuje niską równowagę pomiędzy czułością i precyzją. Sugeruje to, że model nie jest efektywny w identyfikowaniu poejrzanych przypadków.
    - Model został zbudowany w 139 sekund, co pozwala na szybkie zmiany i eksperymentownanie potencjalnie dając szansę na poprawę wyników.
