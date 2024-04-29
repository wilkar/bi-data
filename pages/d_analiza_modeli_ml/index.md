---
title: Porównanie wyników przetestowanych modeli ML
---

## Wstęp

Wykorzystano sześć różnych modeli wchodzących w skład biblioteki `sklearn`:
        - LogisticRegression,
        - RandomForestClassifier,
        - GradientBoostingClassifier,
        - AdaBoostClassifier,
        - ExtraTreesClassifier,
        - KNeighborsClassifier

Każdy z modeli zbudowno przy użyciu dwóch metod oznaczania danych omawianych we [wcześniejszym rozdziale](/pages/data_labeling/index.md). Tworzenie gotowego zestawu na potrzeby tworzenia modeli wyniesniono do osobnej klasy
```python
from src.repositories.helpers import get_engine
from src.repositories.offer.sql_alchemy import SqlAlchemyOfferRepository


class DataSetBuilder:
    def __init__(self):
        self.engine = get_engine()
        self.scraped_offer_repository = SqlAlchemyOfferRepository(self.engine)

    async def _prepare_dataset_with_suspicious_vin_numbers(self) -> list[dict]:
        all_offers = await self.scraped_offer_repository.select_all_offers()
        all_suspicious = (
            await self.scraped_offer_repository.select_all_suspicious_offers()
        )
        suspicious_dict = {
            offer.suspicious_clasfieds_id: offer.is_suspicious
            for offer in all_suspicious
        }

        dataset: list = []
        for offer in all_offers:
            is_suspicious = suspicious_dict.get(offer.clasfieds_id, False)
            dataset.append(
                {
                    "title": offer[1],
                    "description": offer[2],
                    "vin": offer[3],
                    "model": offer[4],
                    "price": offer[5],
                    "milage": offer[6],
                    "condition": offer[7],
                    "country_origin": offer[8],
                    "is_suspicious": is_suspicious,
                }
            )

        return dataset

    async def _prepare_dataset_with_manually_labeled_offers(self) -> list[dict]:
        all_offers = await self.scraped_offer_repository.select_all_offers()
        all_suspicious = (
            await self.scraped_offer_repository.select_all_suspicious_offers_v2()
        )
        suspicious_dict = {
            offer.suspicious_clasfieds_id: offer.is_suspicious
            for offer in all_suspicious
        }

        dataset: list = []
        for offer in all_offers:
            is_suspicious = suspicious_dict.get(offer.clasfieds_id, False)
            dataset.append(
                {
                    "title": offer[1],
                    "description": offer[2],
                    "vin": offer[3],
                    "model": offer[4],
                    "price": offer[5],
                    "milage": offer[6],
                    "condition": offer[7],
                    "country_origin": offer[8],
                    "is_suspicious": is_suspicious,
                }
            )

        return dataset
```
Klasa zawiera bliżniacze metody `_prepare_dataset_with_suspicious_vin_numbers` oraz `_prepare_dataset_with_suspicious_vin_numbers`, które przygotowują zestaw danych w ten sam sposób, ale korzystają z innego zestawu oznaczonych danych.

Następnie zbudowano klasę `ModelsGenerator`, która korzysta z dostarczonych danych i tworzy właściwe modele
```python
import asyncio
import datetime
import logging
from enum import StrEnum

import pandas as pd
from joblib import dump  # type: ignore
from pandas import Series
from sklearn.ensemble import (
    AdaBoostClassifier,  # type: ignore
    ExtraTreesClassifier,
    GradientBoostingClassifier,
    RandomForestClassifier,
)
from sklearn.feature_extraction.text import TfidfVectorizer  # type: ignore
from sklearn.linear_model import LogisticRegression  # type: ignore
from sklearn.metrics import (
    accuracy_score,
    f1_score,  # type: ignore
    precision_score,
    recall_score,
)
from sklearn.model_selection import train_test_split  # type: ignore
from sklearn.neighbors import KNeighborsClassifier  # type: ignore
from sklearn.pipeline import make_pipeline  # type: ignore

from src.config import log_init
from src.services.dataset_generator import DataSetBuilder
from src.services.helpers.data_normalizer import clean_text
from src.services.model_visualization import (
    plot_confusion_matrix,
    plot_precision_recall_curve,
)

log_init.setup_logging()

logger = logging.getLogger(__name__)

models = {
    "LogisticRegression": LogisticRegression(),
    "RandomForestClassifier": RandomForestClassifier(),
    "GradientBoostingClassifier": GradientBoostingClassifier(),
    "AdaBoostClassifier": AdaBoostClassifier(),
    "ExtraTreesClassifier": ExtraTreesClassifier(),
    "KNeighborsClassifier": KNeighborsClassifier(),
}


class mode(StrEnum):
    vin = "vin"
    manual = "description"


class ModelsGenerator:
    def __init__(self, models: dict[str, object]):
        self.models = models
        self.dataset_generator = DataSetBuilder()

    async def evaluate_models_with_manually_labeled_offers(self):
        dataset = (
            await self.dataset_generator._prepare_dataset_with_manually_labeled_offers()
        )
        X_train, X_test, y_train, y_test = await self._build_model(dataset)

        evaluation_results = {}

        for model_name, model in self.models.items():
            logger.info(f"Evaluating model {model} in {mode.manual.value}")
            await self._evaluate_model(
                model_name, model, X_train, y_train, X_test, y_test, mode.manual.value
            )

        return evaluation_results

    async def evaluate_models_with_suspicious_vin_numbers(self):
        dataset = (
            await self.dataset_generator._prepare_dataset_with_suspicious_vin_numbers()
        )
        X_train, X_test, y_train, y_test = await self._build_model(dataset)

        evaluation_results = {}

        for model_name, model in self.models.items():
            logger.info(f"Evaluating model {model} in {mode.manual.vin}")
            await self._evaluate_model(
                model_name, model, X_train, y_train, X_test, y_test, mode.vin.value
            )

        return evaluation_results

    async def _evaluate_model(
        self,
        model_name: str,
        model: object,
        X_train: Series,
        y_train: Series,
        X_test: Series,
        y_test: Series,
        mode: str,
    ):
        evaluation_results: dict = {}
        start_time = datetime.datetime.now()

        pipeline = make_pipeline(TfidfVectorizer(), model)
        pipeline.fit(X_train, y_train)

        await self._save_model(pipeline, model_name, mode)

        y_pred = pipeline.predict(X_test)

        accuracy = accuracy_score(y_test, y_pred)
        precision = precision_score(y_test, y_pred)
        recall = recall_score(y_test, y_pred)
        f1 = f1_score(y_test, y_pred)

        end_time = datetime.datetime.now()
        duration = end_time - start_time

        y_pred = pipeline.predict(X_test)
        logger.info(f"Saving model visualization {model} in {mode}")
        if hasattr(model, "predict_proba"):
            y_scores = pipeline.predict_proba(X_test)[:, 1]
            plot_precision_recall_curve(
                model_name,
                y_test,
                y_scores,
                mode,
            )
            plot_confusion_matrix(y_test, y_pred, model_name, mode)

        evaluation_results[model_name] = {
            "accuracy": accuracy,
            "precision": precision,
            "recall": recall,
            "f1-score": f1,
            "start_time": start_time,
            "end_time": end_time,
            "build_time_seconds": duration.seconds,
        }
        logger.info(f"Model summary for {model} in {mode}: {evaluation_results}")
        return evaluation_results

    async def _build_model(
        self, dataset: list
    ) -> tuple[Series, Series, Series, Series]:
        logger.info(f"Preparing dataset")
        df = pd.DataFrame(dataset)

        df["text"] = (
            df["title"].fillna("")
            + " "
            + df["description"].fillna("")
            + " "
            + df["price"].astype(str)
            + " "
            + df["milage"].astype(str)
            + " "
            + df["model"].fillna("")
            + " "
            + df["condition"].fillna("")
            + " "
            + df["country_origin"].fillna("")
            + " "
            + df["vin"].fillna("")
        )

        df["cleaned_text"] = df["text"].apply(clean_text)

        X = df["cleaned_text"]
        y = df["is_suspicious"]

        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=0.2, random_state=42
        )

        return X_train, X_test, y_train, y_test

    async def _save_model(self, model_pipeline, model_name, mode):
        model_path = f"ml_models/{model_name}_{mode}.joblib"
        dump(model_pipeline, model_path)
        logger.info(f"Saving model")
        return model_path


if __name__ == "__main__":
    test = ModelsGenerator(models)
    loop = asyncio.get_event_loop()
    tasks = [
        loop.create_task(test.evaluate_models_with_manually_labeled_offers()),
        loop.create_task(test.evaluate_models_with_suspicious_vin_numbers()),
    ]
    results = loop.run_until_complete(asyncio.gather(*tasks))

    for result in results:
        print(result)
```


### Resultat

#### Modele używające oznaczonych danych za pomocą numeru VIN

##### Log aplikacji w trakcie budowy modeli opartych o numery VIN

```json
2024-04-28 00:42:34,126 [INFO] __main__: Preparing dataset
2024-04-28 02:28:33,938 [INFO] __main__: Evaluating model LogisticRegression() in vin
2024-04-28 02:29:08,999 [INFO] __main__: Saving model
2024-04-28 02:29:21,366 [INFO] __main__: Saving model visualization LogisticRegression() in vin
2024-04-28 02:29:27,704 [INFO] __main__: Model summary for LogisticRegression() in vin: {'LogisticRegression': {'accuracy': 0.9601449275362319, 'precision': 0.9917334669338678, 'recall': 0.8510318142734308, 'f1-score': 0.9160111059694586, 'start_time': datetime.datetime(2024, 4, 28, 2, 28, 33, 943747), 'end_time': datetime.datetime(2024, 4, 28, 2, 29, 15, 802637), 'build_time_seconds': 41}}
2024-04-28 02:29:27,722 [INFO] __main__: Evaluating model RandomForestClassifier() in vin
2024-04-28 02:51:42,667 [INFO] __main__: Saving model
2024-04-28 02:52:05,622 [INFO] __main__: Saving model visualization RandomForestClassifier() in vin
2024-04-28 02:52:16,684 [INFO] __main__: Model summary for RandomForestClassifier() in vin: {'RandomForestClassifier': {'accuracy': 0.9056049626701801, 'precision': 0.9801866710332405, 'recall': 0.6433791917454859, 'f1-score': 0.7768477061838946, 'start_time': datetime.datetime(2024, 4, 28, 2, 29, 27, 723178), 'end_time': datetime.datetime(2024, 4, 28, 2, 51, 55, 82488), 'build_time_seconds': 1347}}
2024-04-28 02:52:16,704 [INFO] __main__: Evaluating model GradientBoostingClassifier() in vin
2024-04-28 03:07:25,572 [INFO] __main__: Saving model
2024-04-28 03:07:37,028 [INFO] __main__: Saving model visualization GradientBoostingClassifier() in vin
2024-04-28 03:07:42,858 [INFO] __main__: Model summary for GradientBoostingClassifier() in vin: {'GradientBoostingClassifier': {'accuracy': 0.9601998243302591, 'precision': 0.9992372234935164, 'recall': 0.8447979363714532, 'f1-score': 0.9155503785672685, 'start_time': datetime.datetime(2024, 4, 28, 2, 52, 16, 704909), 'end_time': datetime.datetime(2024, 4, 28, 3, 7, 31, 651111), 'build_time_seconds': 914}}
2024-04-28 03:07:42,879 [INFO] __main__: Evaluating model AdaBoostClassifier() in vin
2024-04-28 03:11:56,243 [INFO] __main__: Saving model
2024-04-28 03:12:15,026 [INFO] __main__: Saving model visualization AdaBoostClassifier() in vin
2024-04-28 03:12:24,756 [INFO] __main__: Model summary for AdaBoostClassifier() in vin: {'AdaBoostClassifier': {'accuracy': 0.9580314009661836, 'precision': 0.9905362776025236, 'recall': 0.8437231298366294, 'f1-score': 0.9112542805734517, 'start_time': datetime.datetime(2024, 4, 28, 3, 7, 42, 880074), 'end_time': datetime.datetime(2024, 4, 28, 3, 12, 5, 606212), 'build_time_seconds': 262}}
2024-04-28 03:12:24,777 [INFO] __main__: Evaluating model ExtraTreesClassifier() in vin
2024-04-28 03:48:48,050 [INFO] __main__: Saving model
2024-04-28 03:49:15,430 [INFO] __main__: Saving model visualization ExtraTreesClassifier() in vin
2024-04-28 03:49:28,896 [INFO] __main__: Model summary for ExtraTreesClassifier() in vin: {'ExtraTreesClassifier': {'accuracy': 0.883893280632411, 'precision': 0.962449872402479, 'recall': 0.5674978503869303, 'f1-score': 0.7139959432048681, 'start_time': datetime.datetime(2024, 4, 28, 3, 12, 24, 777252), 'end_time': datetime.datetime(2024, 4, 28, 3, 49, 2, 264904), 'build_time_seconds': 2197}}
2024-04-28 03:49:28,932 [INFO] __main__: Evaluating model KNeighborsClassifier() in vin
2024-04-28 03:49:54,750 [INFO] __main__: Saving model
2024-04-28 04:09:21,542 [INFO] __main__: Saving model visualization KNeighborsClassifier() in vin
2024-04-28 04:19:03,150 [INFO] __main__: Model summary for KNeighborsClassifier() in vin: {'KNeighborsClassifier': {'accuracy': 0.8159036012296882, 'precision': 0.6400301951903375, 'recall': 0.6378976784178848, 'f1-score': 0.6389621575065941, 'start_time': datetime.datetime(2024, 4, 28, 3, 49, 28, 932468), 'end_time': datetime.datetime(2024, 4, 28, 3, 59, 39, 978686), 'build_time_seconds': 611}}
```

```ml_res_vin
select * from ml_results_vin
```

<DataTable data={ml_res_vin} rowNumbers=true>
  <Column id="Classifier" title="Classifier" />
  <Column id="Accuracy" title="Accuracy" contentType="colorscale" scaleColor="#a85ab8" align="center" />
  <Column id="Precision" title="Precision" contentType="colorscale" scaleColor="#e3af05" align="center" />
  <Column id="Recall" title="Recall" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="F1-score" title="F1-score" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="Build_Time_Seconds" title="Build Time" contentType="colorscale" scaleColor="#c43957" align="center" />
</DataTable>

#### Modele używające oznaczonych danych za opisu ogłoszeń

##### Log applikacji w trakcie budowy modelu opartego o opisy ofert

```json
2024-04-28 08:10:55,428 [INFO] __main__: Preparing dataset
^[[1;2C2024-04-28 10:16:32,763 [INFO] __main__: Evaluating model LogisticRegression() in description
2024-04-28 10:17:03,299 [INFO] __main__: Saving model
2024-04-28 10:17:11,586 [INFO] __main__: Saving model visualization LogisticRegression() in description
2024-04-28 10:17:16,118 [INFO] __main__: Model summary for LogisticRegression() in description: {'LogisticRegression': {'accuracy': 0.8150527009222661, 'precision': 0.6741879494173072, 'recall': 0.3339064226943387, 'f1-score': 0.4466162943495401, 'start_time': datetime.datetime(2024, 4, 28, 10, 16, 32, 768281), 'end_time': datetime.datetime(2024, 4, 28, 10, 17, 7, 860668), 'build_time_seconds': 35}}
2024-04-28 10:17:16,139 [INFO] __main__: Evaluating model RandomForestClassifier() in description
2024-04-28 11:02:35,128 [INFO] __main__: Saving model
2024-04-28 11:03:32,669 [INFO] __main__: Saving model visualization RandomForestClassifier() in description
2024-04-28 11:04:01,064 [INFO] __main__: Model summary for RandomForestClassifier() in description: {'RandomForestClassifier': {'accuracy': 0.8224912165129556, 'precision': 0.8314873417721519, 'recall': 0.2581358221785583, 'f1-score': 0.3939649517383563, 'start_time': datetime.datetime(2024, 4, 28, 10, 17, 16, 139612), 'end_time': datetime.datetime(2024, 4, 28, 11, 3, 4, 861088), 'build_time_seconds': 2748}}
2024-04-28 11:04:01,116 [INFO] __main__: Evaluating model GradientBoostingClassifier() in description
2024-04-28 11:32:38,679 [INFO] __main__: Saving model
2024-04-28 11:32:47,528 [INFO] __main__: Saving model visualization GradientBoostingClassifier() in description
2024-04-28 11:32:51,550 [INFO] __main__: Model summary for GradientBoostingClassifier() in description: {'GradientBoostingClassifier': {'accuracy': 0.7985013175230566, 'precision': 0.7186477644492911, 'recall': 0.16185680952965736, 'f1-score': 0.2642076776586148, 'start_time': datetime.datetime(2024, 4, 28, 11, 4, 1, 116682), 'end_time': datetime.datetime(2024, 4, 28, 11, 32, 43, 928152), 'build_time_seconds': 1722}}
2024-04-28 11:32:51,565 [INFO] __main__: Evaluating model AdaBoostClassifier() in description
/Users/karol.wiliwis/Library/Caches/pypoetry/virtualenvs/automotive-classifieds-data-collector-fOnt4d-o-py3.12/lib/python3.12/site-packages/sklearn/ensemble/_weight_boosting.py:519: FutureWarning: The SAMME.R algorithm (the default) is deprecated and will be removed in 1.6. Use the SAMME algorithm to circumvent this warning.
  warnings.warn(
2024-04-28 11:35:48,576 [INFO] __main__: Saving model
2024-04-28 11:36:03,932 [INFO] __main__: Saving model visualization AdaBoostClassifier() in description
2024-04-28 11:36:11,363 [INFO] __main__: Model summary for AdaBoostClassifier() in description: {'AdaBoostClassifier': {'accuracy': 0.7875494071146245, 'precision': 0.5643153526970954, 'recall': 0.21711899791231734, 'f1-score': 0.31358637814827955, 'start_time': datetime.datetime(2024, 4, 28, 11, 32, 51, 565318), 'end_time': datetime.datetime(2024, 4, 28, 11, 35, 55, 801962), 'build_time_seconds': 184}}
2024-04-28 11:36:11,378 [INFO] __main__: Evaluating model ExtraTreesClassifier() in description
2024-04-28 12:39:38,021 [INFO] __main__: Saving model
2024-04-28 12:40:41,811 [INFO] __main__: Saving model visualization ExtraTreesClassifier() in description
2024-04-28 12:41:14,292 [INFO] __main__: Model summary for ExtraTreesClassifier() in description: {'ExtraTreesClassifier': {'accuracy': 0.8253732981993852, 'precision': 0.8154445625221396, 'recall': 0.2826967947930738, 'f1-score': 0.4198431515593653, 'start_time': datetime.datetime(2024, 4, 28, 11, 36, 11, 378713), 'end_time': datetime.datetime(2024, 4, 28, 12, 40, 10, 874604), 'build_time_seconds': 3839}}
2024-04-28 12:41:14,340 [INFO] __main__: Evaluating model KNeighborsClassifier() in description
2024-04-28 12:42:12,438 [INFO] __main__: Saving model
2024-04-28 13:24:43,938 [INFO] __main__: Saving model visualization KNeighborsClassifier() in description
2024-04-28 13:45:18,248 [INFO] __main__: Model summary for KNeighborsClassifier() in description: {'KNeighborsClassifier': {'accuracy': 0.7936155028546333, 'precision': 0.5664678312739667, 'recall': 0.326538130909984, 'f1-score': 0.4142712471761315, 'start_time': datetime.datetime(2024, 4, 28, 12, 41, 14, 340415), 'end_time': datetime.datetime(2024, 4, 28, 13, 3, 50, 772941), 'build_time_seconds': 1356}}
{}
```

```ml_res_description
select * from ml_results_description
```

<DataTable data={ml_res_description} rowNumbers=true>
  <Column id="Classifier" title="Classifier" />
  <Column id="Accuracy" title="Accuracy" contentType="colorscale" scaleColor="#a85ab8" align="center" />
  <Column id="Precision" title="Precision" contentType="colorscale" scaleColor="#e3af05" align="center" />
  <Column id="Recall" title="Recall" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="F1-score" title="F1-score" contentType="colorscale" scaleColor="#c43957" align="center" />
  <Column id="Build_Time_Seconds" title="Build Time" contentType="colorscale" scaleColor="#c43957" align="center" />
</DataTable>

### Analiza porównawcza

Porównując wyniki modeli dla dwóch różnych metod oznaczania danych (VIN vs. opis ogłoszeń), można zaobserwować znaczące różnice w skuteczności poszczególnych modeli:

1. Dokładność (Accuracy):
Modele trenowane na danych z VIN generalnie wykazują wyższą dokładność niż te trenowane na opisach ogłoszeń. Na przykład, LogisticRegression osiąga dokładność około 96% dla VIN, w porównaniu do 81.5% dla opisów.
1. Precyzja (Precision):
Podobnie, precyzja dla modeli na VIN jest zazwyczaj wyższa, co wskazuje na mniejszą liczbę fałszywie pozytywnych wyników. GradientBoostingClassifier w trybie VIN osiąga niemal idealną precyzję (prawie 100%).
1. Czułość (Recall):
Tutaj również modele oparte na VIN mają lepsze wyniki, co sugeruje, że lepiej identyfikują rzeczywiste przypadki.
1. Miara F1 (F1-score):
Wyniki F1 potwierdzają, że ogólna skuteczność modeli na danych VIN jest wyższa niż na danych opartych na opisach.

Modele bazujące na danych VIN wykazują znacznie lepszą wydajność we wszystkich badanych metrykach, co może być spowodowane bardziej jednoznacznym charakterem identyfikacji za pomocą VIN w porównaniu do subiektywnych opisów tekstowych, które mogą być różnie interpretowane i klasyfikowane. Wyniki te sugerują, że metoda oznaczania danych ma istotny wpływ na skuteczność modeli predykcyjnych w kontekście rozpoznawania podejrzanych ogłoszeń samochodowych.

```sql comp_exec_time
select
    mrv.Classifier,
    mrv.Build_Time_Seconds as models_VIN,
    mrd.Build_Time_Seconds as models_Description
from ml_results_vin as mrv left join ml_results_description as mrd on mrv.Classifier = mrd.Classifier
```

<LineChart
data={comp_exec_time}  
x=Classifier
yAxisTitle="Seconds"
xAxisTitle="Model name"
labels=true
showAllLabels=true
y={["models_VIN", "models_Description"]}
/>

```sql comp_exec_accuracy
select
    mrv.Classifier,
    mrv.Accuracy as Accuracy_VIN,
    mrd.Accuracy as Accuracy_Description
from ml_results_vin as mrv left join ml_results_description as mrd on mrv.Classifier = mrd.Classifier
```

<LineChart
data={comp_exec_accuracy}  
x=Classifier
yAxisTitle="Accuracy"
xAxisTitle="Model name"
labels=true
showAllLabels=true
y={["Accuracy_VIN", "Accuracy_Description"]}
/>

```sql comp_exec_precision
select
    mrv.Classifier,
    mrv.Precision as Precision_VIN,
    mrd.Precision as Precision_Description
from ml_results_vin as mrv left join ml_results_description as mrd on mrv.Classifier = mrd.Classifier
```

<LineChart
data={comp_exec_precision}  
x=Classifier
yAxisTitle="Precision"
xAxisTitle="Model name"
labels=true
showAllLabels=true
y={["Precision_VIN", "Precision_Description"]}
/>

```sql comp_exec_recall
select
    mrv.Classifier,
    mrv.Recall as Recall_VIN,
    mrd.Recall as Recall_Description
from ml_results_vin as mrv left join ml_results_description as mrd on mrv.Classifier = mrd.Classifier
```

<LineChart
data={comp_exec_recall}  
x=Classifier
yAxisTitle="Recall"
xAxisTitle="Model name"
labels=true
showAllLabels=true
y={["Recall_VIN", "Recall_Description"]}
/>

```sql comp_exec_f1
select
    mrv.Classifier,
    mrv."F1-score" as "F1-score_VIN",
    mrd."F1-score" as "F1-score_Description"
from ml_results_vin as mrv left join ml_results_description as mrd on mrv.Classifier = mrd.Classifier
```

<LineChart
data={comp_exec_f1}  
x=Classifier
yAxisTitle="F1-score"
xAxisTitle="Model name"
labels=true
showAllLabels=true
y={["F1-score_VIN", "F1-score_Description"]}
/>

### Sugerowane usprawnienia i kierunki rozwoju
1. Optymalizacja przetwarzania danych: Implementacja równoległego przetwarzania w klasie DataSetBuilder może znacznie przyspieszyć przygotowanie zestawów danych, szczególnie gdy operacje bazodanowe są kosztowne czasowo.
1. Zastosowanie redukcji wymiarowości: Użycie metod takich jak PCA (Principal Component Analysis) do redukcji wymiarów danych tekstowych po przekształceniu TF-IDF może pomóc w lepszym oddzieleniu klas i zredukować przetrenowanie.
1. Eksperymentowanie z hiperparametrami: Automatyzacja doboru hiperparametrów za pomocą narzędzi takich jak GridSearchCV lub RandomizedSearchCV mogłaby potencjalnie poprawić wyniki modeli poprzez optymalizację ich konfiguracji.
1. Zastosowanie innego sposobu wektoryzacji: Rozważenie alternatywnych metod wektoryzacji tekstu, takich jak word embeddings (np. Word2Vec, GloVe), które mogą lepiej uchwycić kontekstualne znaczenia słów.


### Spis treści bieżącej sekcji

1. [Model AdaBoostClassifier](/d_analiza_modeli_ml/ada-boost-classfier)
1. [Model ExtraTreesClassifier](/d_analiza_modeli_ml/extra-trees-classifier)
1. [Model GradientBoostingClassifier](/d_analiza_modeli_ml/gradient-boosting-classifier)
1. [Model KNeighborsClassifier](/d_analiza_modeli_ml/k-neighbors-classifier)
1. [Model LogisticRegression](/d_analiza_modeli_ml/logistic-regression)
1. [Model RandomForestClassifier](/d_analiza_modeli_ml/random-forest-classifier)
