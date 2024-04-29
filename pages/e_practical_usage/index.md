---
title: Praktyczne użycie model ML do wykrywania podejrzanych ofert
---


### Wstęp

Bazując na analizie wydajności zbudowanych modeli przeprowadzono test na dwóch próbkach danych:
- 10 ogłoszeń potwierdzonych jako oszustwa. Wytypowane za pomocą danych zebranych z forum bez-wypadkowe.net. 

```sql fraulent_offers
select ob.clasfieds_id,
       title,
       description,
       price,
       milage,
       model,
       condition,
       country_origin,
       vin
from offers_base as ob
         left join offers_details od on ob.clasfieds_id = od.clasfieds_id
where vin in (select vin from labeling_data)
order by ob.clasfieds_id asc
limit 10
```
<DataTable data={fraulent_offers}/>


- 10 ogłoszeń wytypowanych jako uczciwe. Wytypowane za pomocą kilku kryteriów, które mają na celu zmnimalizowanie ryzyka. Wybrane zostały tylko oferty aut nowych ofertowanych przez autoryzowanych dilerów.

```sql non_fraulent_offers
select ob.clasfieds_id,
       title,
       description,
       price,
       milage,
       model,
       condition,
       country_origin,
       vin
from offers_base as ob
         left join offers_details od on ob.clasfieds_id = od.clasfieds_id
where vin not in (select vin from labeling_data)
  and milage < 100
  and vin != 'None'
  and vin is not null
  and description like '%Autoryzowany%'
order by ob.clasfieds_id asc
limit 10
```

<DataTable data={non_fraulent_offers}/>


Na potrzeby ewaluacji stworzono osobny moduł, który korzystał z zapisanych modeli w formacie `joblib`, aby uniknąć czasochłonnego tworzenia modeli od nowa.

```python
from joblib import load  # type: ignore
from enum import StrEnum
from src.services.helpers.data_normalizer import clean_text


class mode(StrEnum):
    vin = "vin"
    manual = "description"


class ml_model(StrEnum):
    logistic_regression = "LogisticRegression"
    random_forest_classifier = "RandomForestClassifier"
    gradient_boosting_classifier = "GradientBoostingClassifier"
    ada_boost_classifier = "AdaBoostClassifier"
    extra_tree_classifier = "ExtraTreesClassifier"
    k_neighbors_classifier = "KNeighborsClassifier"


class SuspiciousnessPredictor:
    def __init__(self, offer: dict, ml_model: ml_model, mode: mode):
        self.offer = offer
        self.ml_model = ml_model
        self.mode = mode

    def predict_suspiciousness(self) -> dict:
        text_input = f'{self.offer["title"]} {self.offer["description"]} {self.offer["price"]} {self.offer["milage"]} {self.offer["model"]} {self.offer["condition"]} {self.offer["country_origin"]} {self.offer["vin"]}'
        cleaned_text_input = clean_text(text_input)
        predictions = {}
        model_pipeline = load(
            f"ml_models/{self.ml_model.value}_{self.mode.value}.joblib"
        )

        if hasattr(model_pipeline, "predict_proba"):
            proba = model_pipeline.predict_proba([cleaned_text_input])[0][1]
        else:
            decision_function = model_pipeline.decision_function([text_input])
            proba = decision_function[0]

        predictions[self.ml_model.value] = proba
        return predictions
```
![Diagram klasy SuspiciousnessPredictor](/assets/suspiciousness_predictor.png)

Powyższy skrypt został następnie wywołany w pętli przy pomocy następującego kodu

```python
import asyncio
import logging

from src.config import log_init
from src.services.suspiciousness_predictor import (
    SuspiciousnessPredictor,
    ml_model,
    mode,
)
from src.evaluate_offers_library import fraulent_offers, non_fraulent_offers

log_init.setup_logging()

logger = logging.getLogger(__name__)


async def process(offers: list[dict], ml_model: ml_model, mode: mode):
    for offer in offers:
        evaluator = SuspiciousnessPredictor(offer, ml_model, mode)
        prediction = evaluator.predict_suspiciousness()
        logger.info(f'Prediction for offer {offer["clasfieds_id"]}: {prediction}')


async def main():
    logger.info("Running...")

    logger.info("Processing fraulent offers with description based model")
    await process(fraulent_offers, ml_model.extra_tree_classifier, mode.manual)
    logger.info("Processing non-fraulent offers with description based model")
    await process(non_fraulent_offers, ml_model.extra_tree_classifier, mode.manual)

    logger.info("Processing fraulent offers with VIN based model")
    await process(fraulent_offers, ml_model.gradient_boosting_classifier, mode.vin)
    logger.info("Processing non-fraulent offers with VIN based model")
    await process(non_fraulent_offers, ml_model.gradient_boosting_classifier, mode.vin)

    logger.info("Done!")


if __name__ == "__main__":
    asyncio.run(main())
```

Diagram ewaluacji testowych ofert
![Diagram klasy SuspiciousnessPredictor](/assets/suspiciousness_predictor_entrypoint.png)


### Wyniki

```json
2024-04-29 02:41:49,820 [INFO] __main__: Running...
2024-04-29 02:41:49,821 [INFO] __main__: Processing fraulent offers with description based model LogisticRegression
2024-04-29 02:41:51,568 [INFO] __main__: Prediction for offer 750272695: 0.30749747234600366
2024-04-29 02:41:52,225 [INFO] __main__: Prediction for offer 776180845: 0.7597147537411466
2024-04-29 02:41:52,924 [INFO] __main__: Prediction for offer 799371168: 0.3074693053634447
2024-04-29 02:41:53,640 [INFO] __main__: Prediction for offer 808237164: 0.07568809616128931
2024-04-29 02:41:54,278 [INFO] __main__: Prediction for offer 812355233: 0.30722701019408105
2024-04-29 02:41:54,925 [INFO] __main__: Prediction for offer 823023618: 0.4834634769175886
2024-04-29 02:41:55,585 [INFO] __main__: Prediction for offer 825055838: 0.0922780014379148
2024-04-29 02:41:56,230 [INFO] __main__: Prediction for offer 826513938: 0.15025007352231837
2024-04-29 02:41:56,862 [INFO] __main__: Prediction for offer 828979997: 0.31112813163886127
2024-04-29 02:41:57,501 [INFO] __main__: Prediction for offer 838380606: 0.13409890709943273
2024-04-29 02:41:57,501 [INFO] __main__: Processing non-fraulent offers with description based model LogisticRegression
2024-04-29 02:41:58,197 [INFO] __main__: Prediction for offer 731031843: 0.06179374948330504
2024-04-29 02:41:58,850 [INFO] __main__: Prediction for offer 744052926: 0.02240348556803191
2024-04-29 02:41:59,502 [INFO] __main__: Prediction for offer 761674517: 0.1217896336282346
2024-04-29 02:42:00,148 [INFO] __main__: Prediction for offer 766504604: 0.15753120095586257
2024-04-29 02:42:00,775 [INFO] __main__: Prediction for offer 772361212: 0.010623675288435417
2024-04-29 02:42:01,403 [INFO] __main__: Prediction for offer 774371057: 0.020376515510747394
2024-04-29 02:42:02,059 [INFO] __main__: Prediction for offer 774780676: 0.05080677893895534
2024-04-29 02:42:02,760 [INFO] __main__: Prediction for offer 779932740: 0.02981975306037116
2024-04-29 02:42:03,472 [INFO] __main__: Prediction for offer 787041095: 0.08161215384673495
2024-04-29 02:42:04,084 [INFO] __main__: Prediction for offer 789922338: 0.08570330947926609
2024-04-29 02:42:04,084 [INFO] __main__: Processing fraulent offers with VIN based model LogisticRegression
2024-04-29 02:42:04,703 [INFO] __main__: Prediction for offer 750272695: 0.19224703391452258
2024-04-29 02:42:05,325 [INFO] __main__: Prediction for offer 776180845: 0.023929677899052065
2024-04-29 02:42:05,976 [INFO] __main__: Prediction for offer 799371168: 0.04638028766252684
2024-04-29 02:42:06,556 [INFO] __main__: Prediction for offer 808237164: 0.09063466668201996
2024-04-29 02:42:07,145 [INFO] __main__: Prediction for offer 812355233: 0.06520650902146752
2024-04-29 02:42:07,731 [INFO] __main__: Prediction for offer 823023618: 0.0451267434726657
2024-04-29 02:42:08,378 [INFO] __main__: Prediction for offer 825055838: 0.05201453784641795
2024-04-29 02:42:08,974 [INFO] __main__: Prediction for offer 826513938: 0.14146557921222713
2024-04-29 02:42:09,566 [INFO] __main__: Prediction for offer 828979997: 0.06575601739188446
2024-04-29 02:42:10,176 [INFO] __main__: Prediction for offer 838380606: 0.06060405503940159
2024-04-29 02:42:10,177 [INFO] __main__: Processing non-fraulent offers with VIN based model LogisticRegression
2024-04-29 02:42:10,833 [INFO] __main__: Prediction for offer 731031843: 0.10480517530771935
2024-04-29 02:42:11,462 [INFO] __main__: Prediction for offer 744052926: 0.029853509759856407
2024-04-29 02:42:12,079 [INFO] __main__: Prediction for offer 761674517: 0.07124374639676577
2024-04-29 02:42:12,692 [INFO] __main__: Prediction for offer 766504604: 0.08295861072559298
2024-04-29 02:42:13,291 [INFO] __main__: Prediction for offer 772361212: 0.07139509538152863
2024-04-29 02:42:13,890 [INFO] __main__: Prediction for offer 774371057: 0.07936816109596909
2024-04-29 02:42:14,506 [INFO] __main__: Prediction for offer 774780676: 0.21036951516359628
2024-04-29 02:42:15,176 [INFO] __main__: Prediction for offer 779932740: 0.14226771991150644
2024-04-29 02:42:15,856 [INFO] __main__: Prediction for offer 787041095: 0.154626381266816
2024-04-29 02:42:16,443 [INFO] __main__: Prediction for offer 789922338: 0.05295848967431575
2024-04-29 02:42:16,443 [INFO] __main__: Processing fraulent offers with description based model RandomForestClassifier
2024-04-29 02:42:18,350 [INFO] __main__: Prediction for offer 750272695: 0.75
2024-04-29 02:42:20,006 [INFO] __main__: Prediction for offer 776180845: 0.81
2024-04-29 02:42:21,672 [INFO] __main__: Prediction for offer 799371168: 0.74
2024-04-29 02:42:23,298 [INFO] __main__: Prediction for offer 808237164: 0.64
2024-04-29 02:42:24,905 [INFO] __main__: Prediction for offer 812355233: 0.94
2024-04-29 02:42:26,558 [INFO] __main__: Prediction for offer 823023618: 0.29
2024-04-29 02:42:28,271 [INFO] __main__: Prediction for offer 825055838: 0.69
2024-04-29 02:42:29,894 [INFO] __main__: Prediction for offer 826513938: 0.82
2024-04-29 02:42:31,495 [INFO] __main__: Prediction for offer 828979997: 0.65
2024-04-29 02:42:33,116 [INFO] __main__: Prediction for offer 838380606: 0.64
2024-04-29 02:42:33,116 [INFO] __main__: Processing non-fraulent offers with description based model RandomForestClassifier
2024-04-29 02:42:34,824 [INFO] __main__: Prediction for offer 731031843: 0.02
2024-04-29 02:42:36,459 [INFO] __main__: Prediction for offer 744052926: 0.0
2024-04-29 02:42:38,135 [INFO] __main__: Prediction for offer 761674517: 0.12
2024-04-29 02:42:39,774 [INFO] __main__: Prediction for offer 766504604: 0.02
2024-04-29 02:42:41,448 [INFO] __main__: Prediction for offer 772361212: 0.02
2024-04-29 02:42:43,119 [INFO] __main__: Prediction for offer 774371057: 0.02
2024-04-29 02:42:44,818 [INFO] __main__: Prediction for offer 774780676: 0.04
2024-04-29 02:42:46,576 [INFO] __main__: Prediction for offer 779932740: 0.01
2024-04-29 02:42:48,341 [INFO] __main__: Prediction for offer 787041095: 0.0
2024-04-29 02:42:49,987 [INFO] __main__: Prediction for offer 789922338: 0.08
2024-04-29 02:42:49,987 [INFO] __main__: Processing fraulent offers with VIN based model RandomForestClassifier
2024-04-29 02:42:51,716 [INFO] __main__: Prediction for offer 750272695: 0.02
2024-04-29 02:42:53,254 [INFO] __main__: Prediction for offer 776180845: 0.04
2024-04-29 02:42:55,007 [INFO] __main__: Prediction for offer 799371168: 0.11
2024-04-29 02:42:56,604 [INFO] __main__: Prediction for offer 808237164: 0.02
2024-04-29 02:42:58,310 [INFO] __main__: Prediction for offer 812355233: 0.06
2024-04-29 02:42:59,961 [INFO] __main__: Prediction for offer 823023618: 0.03
2024-04-29 02:43:01,602 [INFO] __main__: Prediction for offer 825055838: 0.03
2024-04-29 02:43:03,554 [INFO] __main__: Prediction for offer 826513938: 0.06
2024-04-29 02:43:05,422 [INFO] __main__: Prediction for offer 828979997: 0.05
2024-04-29 02:43:07,077 [INFO] __main__: Prediction for offer 838380606: 0.05
2024-04-29 02:43:07,077 [INFO] __main__: Processing non-fraulent offers with VIN based model RandomForestClassifier
2024-04-29 02:43:09,197 [INFO] __main__: Prediction for offer 731031843: 0.13
2024-04-29 02:43:11,360 [INFO] __main__: Prediction for offer 744052926: 0.08
2024-04-29 02:43:13,059 [INFO] __main__: Prediction for offer 761674517: 0.16
2024-04-29 02:43:14,763 [INFO] __main__: Prediction for offer 766504604: 0.04
2024-04-29 02:43:16,462 [INFO] __main__: Prediction for offer 772361212: 0.05
2024-04-29 02:43:18,138 [INFO] __main__: Prediction for offer 774371057: 0.0
2024-04-29 02:43:19,896 [INFO] __main__: Prediction for offer 774780676: 0.07
2024-04-29 02:43:22,559 [INFO] __main__: Prediction for offer 779932740: 0.02
2024-04-29 02:43:24,453 [INFO] __main__: Prediction for offer 787041095: 0.18
2024-04-29 02:43:26,284 [INFO] __main__: Prediction for offer 789922338: 0.01
2024-04-29 02:43:26,284 [INFO] __main__: Processing fraulent offers with description based model GradientBoostingClassifier
2024-04-29 02:43:26,989 [INFO] __main__: Prediction for offer 750272695: 0.16526466470039852
2024-04-29 02:43:27,683 [INFO] __main__: Prediction for offer 776180845: 0.7648485635333309
2024-04-29 02:43:28,458 [INFO] __main__: Prediction for offer 799371168: 0.2950440727717766
2024-04-29 02:43:29,204 [INFO] __main__: Prediction for offer 808237164: 0.14195494399639524
2024-04-29 02:43:30,211 [INFO] __main__: Prediction for offer 812355233: 0.1602247835658731
2024-04-29 02:43:31,255 [INFO] __main__: Prediction for offer 823023618: 0.6298786376392574
2024-04-29 02:43:32,013 [INFO] __main__: Prediction for offer 825055838: 0.10379508176618812
2024-04-29 02:43:32,731 [INFO] __main__: Prediction for offer 826513938: 0.12872459407229817
2024-04-29 02:43:33,435 [INFO] __main__: Prediction for offer 828979997: 0.29950993841146356
2024-04-29 02:43:34,159 [INFO] __main__: Prediction for offer 838380606: 0.16363434484243183
2024-04-29 02:43:34,159 [INFO] __main__: Processing non-fraulent offers with description based model GradientBoostingClassifier
2024-04-29 02:43:34,928 [INFO] __main__: Prediction for offer 731031843: 0.08423506523284743
2024-04-29 02:43:35,862 [INFO] __main__: Prediction for offer 744052926: 0.11078827243347508
2024-04-29 02:43:36,941 [INFO] __main__: Prediction for offer 761674517: 0.14820898700971413
2024-04-29 02:43:37,679 [INFO] __main__: Prediction for offer 766504604: 0.1006605367240474
2024-04-29 02:43:38,381 [INFO] __main__: Prediction for offer 772361212: 0.09245183432636507
2024-04-29 02:43:39,080 [INFO] __main__: Prediction for offer 774371057: 0.15708350447056438
2024-04-29 02:43:39,873 [INFO] __main__: Prediction for offer 774780676: 0.17322097166490874
2024-04-29 02:43:40,671 [INFO] __main__: Prediction for offer 779932740: 0.10219524591711772
2024-04-29 02:43:41,612 [INFO] __main__: Prediction for offer 787041095: 0.09248175986565313
2024-04-29 02:43:42,602 [INFO] __main__: Prediction for offer 789922338: 0.19579785643507294
2024-04-29 02:43:42,602 [INFO] __main__: Processing fraulent offers with VIN based model GradientBoostingClassifier
2024-04-29 02:43:43,465 [INFO] __main__: Prediction for offer 750272695: 0.042615331749632106
2024-04-29 02:43:44,167 [INFO] __main__: Prediction for offer 776180845: 0.05559570158248471
2024-04-29 02:43:44,907 [INFO] __main__: Prediction for offer 799371168: 0.0679754808703607
2024-04-29 02:43:45,615 [INFO] __main__: Prediction for offer 808237164: 0.052201824918097384
2024-04-29 02:43:46,732 [INFO] __main__: Prediction for offer 812355233: 0.061650292041329574
2024-04-29 02:43:47,667 [INFO] __main__: Prediction for offer 823023618: 0.060458302575886434
2024-04-29 02:43:48,394 [INFO] __main__: Prediction for offer 825055838: 0.061650292041329574
2024-04-29 02:43:49,122 [INFO] __main__: Prediction for offer 826513938: 0.052201824918097384
2024-04-29 02:43:49,842 [INFO] __main__: Prediction for offer 828979997: 0.052201824918097384
2024-04-29 02:43:50,656 [INFO] __main__: Prediction for offer 838380606: 0.05503361932280227
2024-04-29 02:43:50,656 [INFO] __main__: Processing non-fraulent offers with VIN based model GradientBoostingClassifier
2024-04-29 02:43:51,562 [INFO] __main__: Prediction for offer 731031843: 0.04200364865405286
2024-04-29 02:43:52,386 [INFO] __main__: Prediction for offer 744052926: 0.04400059002284685
2024-04-29 02:43:53,795 [INFO] __main__: Prediction for offer 761674517: 0.06452322422390637
2024-04-29 02:43:54,832 [INFO] __main__: Prediction for offer 766504604: 0.052201824918097384
2024-04-29 02:43:55,877 [INFO] __main__: Prediction for offer 772361212: 0.03974809179710073
2024-04-29 02:43:56,712 [INFO] __main__: Prediction for offer 774371057: 0.04236848900025063
2024-04-29 02:43:57,490 [INFO] __main__: Prediction for offer 774780676: 0.03974809179710073
2024-04-29 02:43:58,301 [INFO] __main__: Prediction for offer 779932740: 0.05227921073268551
2024-04-29 02:43:59,136 [INFO] __main__: Prediction for offer 787041095: 0.03974809179710073
2024-04-29 02:43:59,879 [INFO] __main__: Prediction for offer 789922338: 0.03974809179710073
2024-04-29 02:43:59,879 [INFO] __main__: Processing fraulent offers with description based model AdaBoostClassifier
2024-04-29 02:44:00,677 [INFO] __main__: Prediction for offer 750272695: 0.4902561806979777
2024-04-29 02:44:01,453 [INFO] __main__: Prediction for offer 776180845: 0.5093578073370354
2024-04-29 02:44:02,274 [INFO] __main__: Prediction for offer 799371168: 0.4985012688245876
2024-04-29 02:44:03,026 [INFO] __main__: Prediction for offer 808237164: 0.49061087016783
2024-04-29 02:44:03,837 [INFO] __main__: Prediction for offer 812355233: 0.49059658440775405
2024-04-29 02:44:04,627 [INFO] __main__: Prediction for offer 823023618: 0.5048234285813631
2024-04-29 02:44:05,398 [INFO] __main__: Prediction for offer 825055838: 0.48844385255456474
2024-04-29 02:44:06,163 [INFO] __main__: Prediction for offer 826513938: 0.48730041186647827
2024-04-29 02:44:06,926 [INFO] __main__: Prediction for offer 828979997: 0.4958722605138432
2024-04-29 02:44:07,706 [INFO] __main__: Prediction for offer 838380606: 0.4935013006813073
2024-04-29 02:44:07,706 [INFO] __main__: Processing non-fraulent offers with description based model AdaBoostClassifier
2024-04-29 02:44:08,629 [INFO] __main__: Prediction for offer 731031843: 0.48744609206435385
2024-04-29 02:44:09,424 [INFO] __main__: Prediction for offer 744052926: 0.47871147361860733
2024-04-29 02:44:10,225 [INFO] __main__: Prediction for offer 761674517: 0.48989318974838114
2024-04-29 02:44:11,020 [INFO] __main__: Prediction for offer 766504604: 0.4821396735754995
2024-04-29 02:44:11,796 [INFO] __main__: Prediction for offer 772361212: 0.48601659979073414
2024-04-29 02:44:12,548 [INFO] __main__: Prediction for offer 774371057: 0.48469239638543016
2024-04-29 02:44:13,342 [INFO] __main__: Prediction for offer 774780676: 0.4955300925313566
2024-04-29 02:44:14,173 [INFO] __main__: Prediction for offer 779932740: 0.489098852793245
2024-04-29 02:44:15,010 [INFO] __main__: Prediction for offer 787041095: 0.48780025067656396
2024-04-29 02:44:15,748 [INFO] __main__: Prediction for offer 789922338: 0.4953044098592165
2024-04-29 02:44:15,748 [INFO] __main__: Processing fraulent offers with VIN based model AdaBoostClassifier
2024-04-29 02:44:16,492 [INFO] __main__: Prediction for offer 750272695: 0.47963627424892014
2024-04-29 02:44:17,230 [INFO] __main__: Prediction for offer 776180845: 0.4816898477251461
2024-04-29 02:44:18,027 [INFO] __main__: Prediction for offer 799371168: 0.4856311262765703
2024-04-29 02:44:18,768 [INFO] __main__: Prediction for offer 808237164: 0.48256225110005513
2024-04-29 02:44:19,497 [INFO] __main__: Prediction for offer 812355233: 0.4871169920773313
2024-04-29 02:44:20,217 [INFO] __main__: Prediction for offer 823023618: 0.4826222379275473
2024-04-29 02:44:20,931 [INFO] __main__: Prediction for offer 825055838: 0.47797362607121785
2024-04-29 02:44:21,652 [INFO] __main__: Prediction for offer 826513938: 0.4842511663518224
2024-04-29 02:44:22,359 [INFO] __main__: Prediction for offer 828979997: 0.4863155256079987
2024-04-29 02:44:23,128 [INFO] __main__: Prediction for offer 838380606: 0.4825477718195245
2024-04-29 02:44:23,129 [INFO] __main__: Processing non-fraulent offers with VIN based model AdaBoostClassifier
2024-04-29 02:44:23,956 [INFO] __main__: Prediction for offer 731031843: 0.4729426429349293
2024-04-29 02:44:24,698 [INFO] __main__: Prediction for offer 744052926: 0.4734073733269511
2024-04-29 02:44:25,444 [INFO] __main__: Prediction for offer 761674517: 0.46857551319799784
2024-04-29 02:44:26,177 [INFO] __main__: Prediction for offer 766504604: 0.48043935479750116
2024-04-29 02:44:27,199 [INFO] __main__: Prediction for offer 772361212: 0.4821422640811978
2024-04-29 02:44:28,479 [INFO] __main__: Prediction for offer 774371057: 0.47455270634947627
2024-04-29 02:44:29,913 [INFO] __main__: Prediction for offer 774780676: 0.47012339934085107
2024-04-29 02:44:30,965 [INFO] __main__: Prediction for offer 779932740: 0.4859932978705979
2024-04-29 02:44:31,868 [INFO] __main__: Prediction for offer 787041095: 0.4775523684004523
2024-04-29 02:44:32,620 [INFO] __main__: Prediction for offer 789922338: 0.4776267751777889
2024-04-29 02:44:32,620 [INFO] __main__: Processing fraulent offers with description based model ExtraTreesClassifier
2024-04-29 02:44:38,628 [INFO] __main__: Prediction for offer 750272695: 1.0
2024-04-29 02:44:43,017 [INFO] __main__: Prediction for offer 776180845: 1.0
2024-04-29 02:44:46,314 [INFO] __main__: Prediction for offer 799371168: 1.0
2024-04-29 02:44:49,478 [INFO] __main__: Prediction for offer 808237164: 1.0
2024-04-29 02:44:52,646 [INFO] __main__: Prediction for offer 812355233: 1.0
2024-04-29 02:44:55,972 [INFO] __main__: Prediction for offer 823023618: 0.43
2024-04-29 02:44:59,143 [INFO] __main__: Prediction for offer 825055838: 1.0
2024-04-29 02:45:02,430 [INFO] __main__: Prediction for offer 826513938: 1.0
2024-04-29 02:45:06,567 [INFO] __main__: Prediction for offer 828979997: 1.0
2024-04-29 02:45:11,365 [INFO] __main__: Prediction for offer 838380606: 1.0
2024-04-29 02:45:11,365 [INFO] __main__: Processing non-fraulent offers with description based model ExtraTreesClassifier
2024-04-29 02:45:15,964 [INFO] __main__: Prediction for offer 731031843: 0.0
2024-04-29 02:45:19,322 [INFO] __main__: Prediction for offer 744052926: 0.0
2024-04-29 02:45:22,627 [INFO] __main__: Prediction for offer 761674517: 0.09
2024-04-29 02:45:25,874 [INFO] __main__: Prediction for offer 766504604: 0.0
2024-04-29 02:45:29,098 [INFO] __main__: Prediction for offer 772361212: 0.0
2024-04-29 02:45:32,364 [INFO] __main__: Prediction for offer 774371057: 0.0
2024-04-29 02:45:35,531 [INFO] __main__: Prediction for offer 774780676: 0.0
2024-04-29 02:45:38,633 [INFO] __main__: Prediction for offer 779932740: 0.0
2024-04-29 02:45:41,817 [INFO] __main__: Prediction for offer 787041095: 0.0
2024-04-29 02:45:44,844 [INFO] __main__: Prediction for offer 789922338: 0.0
2024-04-29 02:45:44,845 [INFO] __main__: Processing fraulent offers with VIN based model ExtraTreesClassifier
2024-04-29 02:45:48,075 [INFO] __main__: Prediction for offer 750272695: 0.0
2024-04-29 02:45:50,887 [INFO] __main__: Prediction for offer 776180845: 0.0
2024-04-29 02:45:53,621 [INFO] __main__: Prediction for offer 799371168: 0.05
2024-04-29 02:45:56,339 [INFO] __main__: Prediction for offer 808237164: 0.0
2024-04-29 02:45:59,132 [INFO] __main__: Prediction for offer 812355233: 0.09
2024-04-29 02:46:01,749 [INFO] __main__: Prediction for offer 823023618: 0.0
2024-04-29 02:46:04,486 [INFO] __main__: Prediction for offer 825055838: 0.0
2024-04-29 02:46:07,200 [INFO] __main__: Prediction for offer 826513938: 0.0
2024-04-29 02:46:10,152 [INFO] __main__: Prediction for offer 828979997: 0.0
2024-04-29 02:46:12,830 [INFO] __main__: Prediction for offer 838380606: 0.0
2024-04-29 02:46:12,830 [INFO] __main__: Processing non-fraulent offers with VIN based model ExtraTreesClassifier
2024-04-29 02:46:15,537 [INFO] __main__: Prediction for offer 731031843: 0.0
2024-04-29 02:46:18,222 [INFO] __main__: Prediction for offer 744052926: 0.02
2024-04-29 02:46:20,905 [INFO] __main__: Prediction for offer 761674517: 0.22
2024-04-29 02:46:23,516 [INFO] __main__: Prediction for offer 766504604: 0.0
2024-04-29 02:46:26,110 [INFO] __main__: Prediction for offer 772361212: 0.0
2024-04-29 02:46:28,752 [INFO] __main__: Prediction for offer 774371057: 0.0
2024-04-29 02:46:31,810 [INFO] __main__: Prediction for offer 774780676: 0.0
2024-04-29 02:46:34,506 [INFO] __main__: Prediction for offer 779932740: 0.0
2024-04-29 02:46:37,232 [INFO] __main__: Prediction for offer 787041095: 0.0
2024-04-29 02:46:39,808 [INFO] __main__: Prediction for offer 789922338: 0.0
2024-04-29 02:46:39,808 [INFO] __main__: Processing fraulent offers with description based model KNeighborsClassifier
2024-04-29 02:46:41,292 [INFO] __main__: Prediction for offer 750272695: 0.2
2024-04-29 02:46:42,633 [INFO] __main__: Prediction for offer 776180845: 0.2
2024-04-29 02:46:44,025 [INFO] __main__: Prediction for offer 799371168: 0.2
2024-04-29 02:46:45,406 [INFO] __main__: Prediction for offer 808237164: 0.2
2024-04-29 02:46:46,756 [INFO] __main__: Prediction for offer 812355233: 1.0
2024-04-29 02:46:48,143 [INFO] __main__: Prediction for offer 823023618: 0.0
2024-04-29 02:46:49,523 [INFO] __main__: Prediction for offer 825055838: 0.6
2024-04-29 02:46:50,916 [INFO] __main__: Prediction for offer 826513938: 0.6
2024-04-29 02:46:52,280 [INFO] __main__: Prediction for offer 828979997: 0.6
2024-04-29 02:46:53,805 [INFO] __main__: Prediction for offer 838380606: 0.2
2024-04-29 02:46:53,806 [INFO] __main__: Processing non-fraulent offers with description based model KNeighborsClassifier
2024-04-29 02:46:55,241 [INFO] __main__: Prediction for offer 731031843: 0.0
2024-04-29 02:46:56,610 [INFO] __main__: Prediction for offer 744052926: 0.0
2024-04-29 02:46:57,989 [INFO] __main__: Prediction for offer 761674517: 0.2
2024-04-29 02:46:59,358 [INFO] __main__: Prediction for offer 766504604: 0.4
2024-04-29 02:47:00,732 [INFO] __main__: Prediction for offer 772361212: 0.0
2024-04-29 02:47:02,098 [INFO] __main__: Prediction for offer 774371057: 0.0
2024-04-29 02:47:03,468 [INFO] __main__: Prediction for offer 774780676: 0.0
2024-04-29 02:47:04,887 [INFO] __main__: Prediction for offer 779932740: 0.0
2024-04-29 02:47:06,367 [INFO] __main__: Prediction for offer 787041095: 0.0
2024-04-29 02:47:07,713 [INFO] __main__: Prediction for offer 789922338: 0.2
2024-04-29 02:47:07,713 [INFO] __main__: Processing fraulent offers with VIN based model KNeighborsClassifier
2024-04-29 02:47:09,185 [INFO] __main__: Prediction for offer 750272695: 0.4
2024-04-29 02:47:10,579 [INFO] __main__: Prediction for offer 776180845: 0.8
2024-04-29 02:47:11,986 [INFO] __main__: Prediction for offer 799371168: 0.0
2024-04-29 02:47:13,323 [INFO] __main__: Prediction for offer 808237164: 0.0
2024-04-29 02:47:14,663 [INFO] __main__: Prediction for offer 812355233: 0.2
2024-04-29 02:47:16,008 [INFO] __main__: Prediction for offer 823023618: 0.4
2024-04-29 02:47:17,372 [INFO] __main__: Prediction for offer 825055838: 0.2
2024-04-29 02:47:18,697 [INFO] __main__: Prediction for offer 826513938: 0.0
2024-04-29 02:47:20,059 [INFO] __main__: Prediction for offer 828979997: 0.4
2024-04-29 02:47:21,449 [INFO] __main__: Prediction for offer 838380606: 0.0
2024-04-29 02:47:21,449 [INFO] __main__: Processing non-fraulent offers with VIN based model KNeighborsClassifier
2024-04-29 02:47:22,902 [INFO] __main__: Prediction for offer 731031843: 0.6
2024-04-29 02:47:24,278 [INFO] __main__: Prediction for offer 744052926: 0.0
2024-04-29 02:47:25,684 [INFO] __main__: Prediction for offer 761674517: 0.4
2024-04-29 02:47:27,036 [INFO] __main__: Prediction for offer 766504604: 0.2
2024-04-29 02:47:28,389 [INFO] __main__: Prediction for offer 772361212: 0.2
2024-04-29 02:47:29,792 [INFO] __main__: Prediction for offer 774371057: 0.0
2024-04-29 02:47:31,184 [INFO] __main__: Prediction for offer 774780676: 0.6
2024-04-29 02:47:32,625 [INFO] __main__: Prediction for offer 779932740: 0.0
2024-04-29 02:47:34,121 [INFO] __main__: Prediction for offer 787041095: 0.2
2024-04-29 02:47:35,518 [INFO] __main__: Prediction for offer 789922338: 0.0
2024-04-29 02:47:35,519 [INFO] __main__: Done!
```

Wyniki realnego zastosowania znacząco odbiegają od statystyk zebranych podczas budowy modelu. W praktyce tylko `ExtraTreesClassifier` oparty na danych oznaczonych przy pomocy opisów ogłoszeń zwrócił wyniki adekwatne dla faktycznego stanu ofert. Pozostałe modele, niezależnie od metody oznaczania podejrzanych ofert, zwracały wyniki, które znacząco odbiegają od stanu faktycznego. 
