---
title: Model GradientBoostingClassifier
---


## Wyniki

```json
    {
        "GradientBoostingClassifier accuracy": 0.8156255724491666
        "GradientBoostingClassifier precision": 0.8253521126760563
        "GradientBoostingClassifier recall": 0.06882781301385953
        "GradientBoostingClassifier F1-score": 0.12705984388551606
        "GradientBoostingClassifier build time": 614 seconds
    }
```


![Confusion Matrix](/static/confusion-matrix-gradient-boost-classifier.png)
Legenda:
    - True-Negative (Lewy górny róg, TN): Liczba poprawnie sklasyfikowanych ofert jako "niepodejrzane".
    - False-Positivve (Prawy górny róg, FP): Liczba ofert błędnie zakwalifikowanych jako "podejrzane", które faktycznie są poprawne.
    - False-Negative (Lewy dolny róg, FN): Liczba ofert błędnie zakwalifikowanych jako "niepodejrzane", które są podejrzane.
    - False-Positive (Prawy dolny róg, TP): Liczba poprawnie sklasyfikowanych ofert jako "podejrzane".

![Precision Recall](/static/precision-recall-gradient-boosting-classifier.png)
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
    - Średnia precyzja (Average precision) o wartości 0.47% wskazuje przeciętną wydajność modelu
    - Dokładność (Accuracy) modelu 81.5% sugeruje, że model podejmuje poprawną decyzję w większości przypadków.
    - Precision (Precision) modelu 82.5% sugeruje, że model przewiduje ofertę jako "podejrzaną", jest ona faktycznie podejrzana w dwóch na trzy przypadki.
    - Czułość (Recall) modelu 68.8% pokazuje, że model identyfikuje tylko około dwie na trzy faktycznie "podejrzanych" ofert.
    - F1-score na poziomie 12.7% wskazuje niską równowagę pomiędzy czułością i precyzją. Sugeruje to, że model jest mniej efektywny niż LogisticRegression i KNeighborsClassifier.
    - Model został zbudowany w 21 sekund, co pozwala na szybkie zmiany i eksperymentownanie potencjalnie dając szansę na poprawę wyników.
