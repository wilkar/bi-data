---
title: Oznaczanie Danych przy pomocy VIN
---

## Skrypt oznaczający oraz wyniki

```python
import asyncio
import logging
import re

from sqlalchemy import Row

from src.config import log_init
from src.models.raw_offer import SuspiciousOffer
from src.repositories.helpers import get_engine
from src.repositories.offer.sql_alchemy import SqlAlchemyOfferRepository

log_init.setup_logging()

logger = logging.getLogger(__name__)


async def _is_suspicious(offer: Row) -> bool:
    vin = offer.vin

    if vin is None:
        return True

    if re.match(r"^([a-zA-Z0-9])\1{16}$", vin):
        return True

    if vin.lower() == vin or vin.upper() == vin:
        if re.match(r"^([a-zA-Z0-9])\1+$", vin):
            return True

    placeholder_patterns = [
        "zapytaj",
        "wysylam",
        "kontakt",
        "zadzwon",
        "nrvin",
        "astaz",
        "error",
        "xxxx",
        "vvvv",
        "zzzz",
        "yyyy",
        "www",
    ]

    if any(re.search(pattern, vin.lower()) for pattern in placeholder_patterns):
        return True

    if re.search("[^a-hj-npr-z0-9]", vin.lower()):
        return True

    if re.match(r"^123456789|abcdefg|012345|987654|abcdef", vin.lower()):
        return True

    if re.search(r"tel|phone|contact|zadzwon", vin.lower()):
        return True

    if re.match(r"^(wauzzz|vf|wba)[a-z0-9]*[x]{5,}", vin.lower()):
        return True

    return False


async def process_offers():
    engine = get_engine()
    scraped_offer_repository = SqlAlchemyOfferRepository(engine)

    all_offers = await scraped_offer_repository.select_all_offers()
    for offer in all_offers:
        suspicious_indicator = await _is_suspicious(offer)
        suspicious_offer = SuspiciousOffer(
            suspicious_clasfieds_id=offer.clasfieds_id,
            is_suspicious=suspicious_indicator,
        )
        await scraped_offer_repository.add_suspicious_offer(suspicious_offer)


asyncio.run(process_offers())
```

Wyniki działania skryptu
```sql sus_offers
select * from suspicious_offers
```

Wizualizacja wytypowanych podejrzanych ofert ze względu na producenta i lokalizację sprzedaży zawężona do 10 najpopularnijeszych wyników.

```sql sus_brand
select lower(ob.brand) as brand, count(*) as ct
from offers_base as ob
         left join suspicious_offers as so on ob.clasfieds_id = so.suspicious_clasfieds_id
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
         left join suspicious_offers as so on ol.clasfieds_id = so.suspicious_clasfieds_id
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
