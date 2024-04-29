---
title: Oznaczanie Danych przy pomocy opisu ogłoszenia
---

## Skrypt oznaczający oraz wyniki

```python
from src.repositories.offer.sql_alchemy import SqlAlchemyOfferRepository
from src.repositories.helpers import get_engine
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np
import asyncio
from src.models.raw_offer import SuspiciousOffer

# from src.services.helpers.data_normalizer import clean_text


class TestSusSelector:
    def __init__(self):
        self.engine = get_engine()
        self.scraped_offer_repository = SqlAlchemyOfferRepository(self.engine)
        self.model = SentenceTransformer("all-MiniLM-L6-v2")

    async def _get_suspicions_descriptions(self):
        suspicious_offers = (
            await self.scraped_offer_repository.select_suspicious_offers_from_labeling_data()
        )
        suspicious_data = [(offer[0], offer[2]) for offer in suspicious_offers]
        suspicious_descriptions = [data[1] for data in suspicious_data]
        return suspicious_data, self.model.encode(suspicious_descriptions)

    async def get_similar_descriptions(self, similarity_threshold=0.8):
        suspicious_data, suspicious_embeddings = (
            await self._get_suspicions_descriptions()
        )
        offers = await self.scraped_offer_repository.select_all_offers()
        offers_data = [(offer[0], offer[2]) for offer in offers]
        descriptions = [offer[1] for offer in offers_data]
        embeddings = self.model.encode(descriptions)
        suspicious_offers: list = []

        for i, (suspicious_id, suspicious_embedding) in enumerate(
            zip(suspicious_data, suspicious_embeddings)
        ):
            similarities = cosine_similarity([suspicious_embedding], embeddings)[0]
            for index, similarity_score in enumerate(similarities):
                if similarity_score > similarity_threshold:
                    suspicious_offers.append(offers_data[index][0])

        await self._populate_suspicious_offers_v2(suspicious_offers)

    async def _populate_suspicious_offers_v2(self, suspicious_offers: list):
        offers = await self.scraped_offer_repository.select_all_offers()

        for offer in offers:
            suspicious_indicator: bool = False
            if offer.clasfieds_id in suspicious_offers:
                suspicious_indicator = True
            suspicious_offer = SuspiciousOffer(
                suspicious_clasfieds_id=offer.clasfieds_id,
                is_suspicious=suspicious_indicator,
            )
            await self.scraped_offer_repository.add_suspicious_offer_v2(
                suspicious_offer
            )


if __name__ == "__main__":
    test = TestSusSelector()

    asyncio.run(test.get_similar_descriptions())
```

Wyniki działania skryptu
```sql sus_offers
select * from suspicious_offers_v2
```

Wizualizacja wytypowanych podejrzanych ofert ze względu na producenta i lokalizację sprzedaży zawężona do 10 najpopularnijeszych wyników.

```sql sus_brand
select lower(ob.brand) as brand, count(*) as ct
from offers_base as ob
         left join suspicious_offers_v2 as so on ob.clasfieds_id = so.suspicious_clasfieds_id
where so.is_suspicious is true
group by ob.brand
order by ct desc
limit 10;
```

<BarChart 
    data={sus_brand} 
    x=brand
    y=ct 
    xAxisTitle="Podejrzane oferty wg producenta"
/>


```sql sus_city
select lower(ol.city) as city, count(*) as ct
from offers_location as ol
         left join suspicious_offers_v2 as so on ol.clasfieds_id = so.suspicious_clasfieds_id
where so.is_suspicious is true
group by ol.city
order by ct desc
limit 10;
```

<BarChart 
    data={sus_city} 
    x=city
    y=ct 
    xAxisTitle="Podejrzane oferty wg miasta"
/>
