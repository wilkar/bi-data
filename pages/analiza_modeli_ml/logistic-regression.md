---
title: Model LogisticRegression
---


## Wyniki

```json
    {
        "LogisticRegression accuracy": 0.8280820663125115
        "LogisticRegression precision": 0.6762438682550805
        "LogisticRegression recall": 0.22668545924359879
        "LogisticRegression F1-score": 0.33954961294862773
        "LogisticRegression build time": 21 seconds
    }
```


![Confusion Matrix](confusion-matrix-logistics-regression.png)
Legenda:
    - True-Negative (Lewy górny róg, TN): Liczba poprawnie sklasyfikowanych ofert jako "niepodejrzane".
    - False-Positivve (Prawy górny róg, FP): Liczba ofert błędnie zakwalifikowanych jako "podejrzane", które faktycznie są poprawne.
    - False-Negative (Lewy dolny róg, FN): Liczba ofert błędnie zakwalifikowanych jako "niepodejrzane", które są podejrzane.
    - False-Positive (Prawy dolny róg, TP): Liczba poprawnie sklasyfikowanych ofert jako "podejrzane".

![Precision Recall](precision-recall-logistics-regression.png)
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
    - Dokładność (Accuracy) modelu (17,117 + 965) / (17,117 + 965 + 462 + 3,292) = 82.8% sugeruje, że model podejmuje poprawną decyzję w większości przypadków.
    - Precision (Precision) modelu (965) / (965+462) = 67.6% sugeruje, że model przewiduje ofertę jako "podejrzaną", jest ona faktycznie podejrzana w dwóch na trzy przypadki.
    - Czułość (Recall) modelu (965) / (965 + 3292) = 22.7% pokazuje, że model identyfikuje tylko około jedną na pięć faktycznie "podejrzanych" ofert.
    - F1-score na poziomie 33.95% wskazuje niską równowagę pomiędzy czułością i precyzją. Sugeruje to, że model nie jest efektywny w identyfikowaniu poejrzanych przypadków.
    - Model został zbudowany w 21 sekund, co pozwala na szybkie zmiany i eksperymentownanie potencjalnie dając szansę na poprawę wyników.
