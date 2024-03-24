---
title: Model ExtraTreesClassifier
---


## Wyniki

```json
    {
        "ExtraTreesClassifier accuracy": 0.8458508884411065
        "ExtraTreesClassifier precision": 0.8087318087318087
        "ExtraTreesClassifier recall": 0.2741367159971811
        "ExtraTreesClassifier F1-score": 0.4094736842105263
        "ExtraTreesClassifier build time": 884 seconds
    }
```


![Confusion Matrix](/static/confusion-matrix-extra-trees-classfier.png)
Legenda:
    - True-Negative (Lewy górny róg, TN): Liczba poprawnie sklasyfikowanych ofert jako "niepodejrzane".
    - False-Positivve (Prawy górny róg, FP): Liczba ofert błędnie zakwalifikowanych jako "podejrzane", które faktycznie są poprawne.
    - False-Negative (Lewy dolny róg, FN): Liczba ofert błędnie zakwalifikowanych jako "niepodejrzane", które są podejrzane.
    - False-Positive (Prawy dolny róg, TP): Liczba poprawnie sklasyfikowanych ofert jako "podejrzane".

![Precision Recall](/static/precision-recall-extra-tree-classfier.png)
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
    - Dokładność (Accuracy) modelu 84.5% sugeruje, że model podejmuje poprawną decyzję w większości przypadków.
    - Precision (Precision) modelu 80.8% sugeruje, że model przewiduje ofertę jako "podejrzaną", jest ona faktycznie podejrzana w czterech na pięć przypadki.
    - Czułość (Recall) modelu 0.27% pokazuje, że model identyfikuje tylko około jedną na pięć faktycznie "podejrzanych" ofert.
    - F1-score na poziomie 40% wskazuje średnią równowagę pomiędzy czułością i precyzją. Sugeruje to, że model nie ma przeciętny wynik w identyfikowaniu poejrzanych przypadków.
    - Model został zbudowany w 884 sekund, co jest najgorszym wynikiem spóśród badanych modeli.
