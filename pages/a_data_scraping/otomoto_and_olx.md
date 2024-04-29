---
title: Scraping ofert Otomoto i OLX
---

## Wstęp

W trakcie analizy struktur portali OLX i OTOMOTO zidentyfikowano różnice w używanych API: OLX wykorzystuje REST API, a OTOMOTO - GraphQL API. REST API okazało się prostsze do zastosowania w scrapingu danych ze względu na łatwiejsze przygotowanie odpowiedniego zapytania.


### OLX

Zaimplementowany scraper charakteryzuje się następującymi właściwościami:

- Przetwarzanie danych wyłącznie z kategorii motoryzacyjnych.
- Ograniczenie działania do maksymalnie 25 podstron na każdą kategorię.
- Parsowanie odpowiedzi w formacie JSON do predefiniowanych struktur danych.
- Wykorzystanie iteratora Pythonowego (yield), co pozwala na przetwarzenie wyników na bieżąco.

Poniżej przykladowa oferta w formacie zwracanym przez REST API OLX:
```json
{
"id": 909345897,
"url": "https://www.olx.pl/d/oferta/bmw-seria-5-automat-panorama-m-pakiet-CID5-IDZxvpj.html",
"title": "BMW Seria 5 Automat, Panorama, M-Pakiet",
"last_refresh_time": "2024-04-27T23:01:00+02:00",
"created_time": "2024-04-04T00:02:43+02:00",
"valid_to_time": "2024-05-03T23:01:53+02:00",
"pushup_time": null,
"description": "Sprzedam BMW 520d G31 M-Pakiet, Touring <br />\n<br />\nSilnik: 2.0 190 ps - dynamiczny i ekonomiczny <br />\nRok produkcji 2017r.<br />\nAuto z oryginalnym przebiegiem 251 681 km<br />\n<br />\nAuto w bardzo dobrym stanie, na bieżąco serwisowane, w ciągłej eksploatacji do jazdy prywatnej <br />\n<br />\nDuże, przestronne idealne na dłuższe wyprawy oraz jazdę po mieście. Niskie spalanie,a zarazem auto jest dynamiczne <br />\n<br />\nWyposażenie:<br />\n- Światła w technologii Led <br />\n- Nawigacja<br />\n- Automatyczna skrzynia biegów<br />\n- Rozpoznawanie znaków <br />\n- Skórzana tapicerka<br />\n- 4x elektryczne szyby<br />\n- Elektryczne lusterka<br />\n- Podgrzewane lusterka<br />\n- Podgrzewane i elektryczne fotele <br />\n- Pamięć foteli <br />\n- Panorama<br />\n- Auto Hold<br />\n- Klimatyzacja 3-strefowa <br />\n- Wielofunkcyjna kierownica z manetkami od zmiany biegów <br />\n- Czujniki parkowania przód/tył<br />\n- Kamera cofania <br />\n- Tempomat/ogranicznik prędkości<br />\n- Czujnik zmierzchu/deszczu<br />\n- Coming/leaving home<br />\n- Isofix<br />\n- USB<br />\n- Apple car play/ Android auto <br />\n- Bluetooth<br />\n- Przyciemniane tylne szyby<br />\n- Felgi 19\"<br />\n- Oraz wiele innych<br />\n<br />\nAuto serdecznie polecam, technicznie i wizualnie bez zarzutów, silnik pracuje równo, zawieszenie nie stuka, układ kierowniczy precyzyjny. W samochodzie wszystko działa. Pojazd można sprawdzić na dowolnej stacji diagnostycznej. <br />\n<br />\nZapraszam na oględziny oraz jazdę próbną.",
"promotion": {
"highlighted": false,
"urgent": false,
"top_ad": false,
"options": [],
"b2c_ad_page": false,
"premium_ad_page": false
},
"params": [
{
"key": "price",
"name": "Cena",
"type": "price",
"value": {
"value": 93000,
"type": "price",
"arranged": false,
"budget": false,
"currency": "PLN",
"negotiable": false,
"converted_value": null,
"previous_value": null,
"converted_previous_value": null,
"converted_currency": null,
"label": "93 000 zł",
"previous_label": null
}
},
{
"key": "vin",
"name": "Numer VIN",
"type": "input",
"value": {
"key": "WBAJM71090B055323",
"label": "WBAJM71090B055323 "
}
},
{
"key": "model",
"name": "Model",
"type": "select",
"value": {
"key": "5-os-sorozat",
"label": "Seria 5"
}
},
{
"key": "enginesize",
"name": "Poj. silnika",
"type": "input",
"value": {
"key": "1995",
"label": "1 995 cm³"
}
},
{
"key": "year",
"name": "Rok produkcji",
"type": "input",
"value": {
"key": "2017",
"label": "2017 "
}
},
{
"key": "enginepower",
"name": "Moc silnika",
"type": "input",
"value": {
"key": "190",
"label": "190 KM"
}
},
{
"key": "petrol",
"name": "Paliwo",
"type": "select",
"value": {
"key": "diesel",
"label": "Diesel"
}
},
{
"key": "car_body",
"name": "Typ nadwozia",
"type": "select",
"value": {
"key": "estate-car",
"label": "Kombi"
}
},
{
"key": "milage",
"name": "Przebieg",
"type": "input",
"value": {
"key": "251681",
"label": "251 681 km"
}
},
{
"key": "color",
"name": "Kolor",
"type": "select",
"value": {
"key": "szary",
"label": "Szary"
}
},
{
"key": "condition",
"name": "Stan techniczny",
"type": "select",
"value": {
"key": "notdamaged",
"label": "Nieuszkodzony"
}
},
{
"key": "transmission",
"name": "Skrzynia biegów",
"type": "select",
"value": {
"key": "automatic",
"label": "Automatyczna"
}
},
{
"key": "country_origin",
"name": "Kraj pochodzenia",
"type": "select",
"value": {
"key": "d",
"label": "Niemcy"
}
},
{
"key": "drive",
"name": "Napęd",
"type": "select",
"value": {
"key": "rear-wheel",
"label": "Na tylne koła"
}
}
],
"key_params": [
"year",
"milage"
],
"business": false,
"user": {
"id": 23063449,
"created": "2013-05-01T08:32:22+02:00",
"other_ads_enabled": true,
"name": "Rafał",
"logo": null,
"logo_ad_page": null,
"social_network_account_type": null,
"photo": null,
"banner_mobile": "",
"banner_desktop": "",
"company_name": "",
"about": "",
"b2c_business_page": false,
"is_online": true,
"last_seen": "2024-04-27T23:05:15+02:00",
"seller_type": null,
"uuid": "0afdea7a-f009-4f41-b8b5-ad22f3b75f44"
},
"status": "active",
"contact": {
"name": "Rafał",
"phone": true,
"chat": true,
"negotiation": false,
"courier": false
},
"map": {
"zoom": 12,
"lat": 51.7954,
"lon": 15.65819,
"radius": 0,
"show_detailed": false
},
"location": {
"city": {
"id": 89441,
"name": "Lubieszów",
"normalized_name": "lubieszow"
},
"region": {
"id": 9,
"name": "Lubuskie",
"normalized_name": "lubuskie"
}
},
"photos": [
{
"id": 7592251390,
"filename": "mc53jffj0kdb1-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/mc53jffj0kdb1-PL/image;s={width}x{height}"
},
{
"id": 7592251479,
"filename": "x5dua5q9eoxf3-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/x5dua5q9eoxf3-PL/image;s={width}x{height}"
},
{
"id": 7592251631,
"filename": "3iaj79x5febg2-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/3iaj79x5febg2-PL/image;s={width}x{height}"
},
{
"id": 7592251711,
"filename": "ufujekqge3fv-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/ufujekqge3fv-PL/image;s={width}x{height}"
},
{
"id": 7592251761,
"filename": "f73x0ybrwxw1-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/f73x0ybrwxw1-PL/image;s={width}x{height}"
},
{
"id": 7592251858,
"filename": "mxcb4hnh40op3-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/mxcb4hnh40op3-PL/image;s={width}x{height}"
},
{
"id": 7592251950,
"filename": "augilygnuqgb2-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/augilygnuqgb2-PL/image;s={width}x{height}"
},
{
"id": 7592252001,
"filename": "ur4wqzoqp2ku2-PL",
"rotation": 0,
"width": 1813,
"height": 1360,
"link": "https://ireland.apollo.olxcdn.com:443/v1/files/ur4wqzoqp2ku2-PL/image;s={width}x{height}"
}
],
"partner": {
"code": "otomoto_pl_form"
},
"external_url": "https://www.otomoto.pl/osobowe/oferta/bmw-seria-5-automat-panorama-m-pakiet-ID6GkrKT.html",
"category": {
"id": 183,
"type": "automotive"
},
"delivery": {
"rock": {
"offer_id": null,
"active": false,
"mode": "NotEligible"
}
},
"safedeal": {
"weight": 0,
"weight_grams": 0,
"status": "unactive",
"safedeal_blocked": false,
"allowed_quantity": []
},
"shop": {
"subdomain": null
},
"offer_type": "offer"
}
```

Oraz modele danych do których mapowane są oryginalne dane

```python
from dataclasses import dataclass, field
from datetime import datetime


@dataclass(frozen=True, kw_only=True)
class RawOfferLocation:
    id: int
    region: str | None
    city: str | None


@dataclass(frozen=True, kw_only=True)
class RawOfferParameters:
    id: int
    model: str | None
    price: int | None
    engine_size: str | None
    manufactured_year: str | None
    engine_power: str | None
    petrol: str | None
    car_body: str | None
    milage: int | None
    color: str | None
    condition: str | None
    transmission: str | None
    drive: str | None
    country_origin: str | None
    righthanddrive: str | None
    vin: str | None


@dataclass(frozen=True, kw_only=True)
class RawOffer:
    brand: str
    id: int
    link: str
    title: str
    created_time: datetime | None
    description: str
    image_links: list[str] | None
    parameters: list[RawOfferParameters] = field(default_factory=list)
    location: list[RawOfferLocation] = field(default_factory=list)
    vin: str | None
    scraped_time: datetime | None


@dataclass(frozen=True, kw_only=True)
class SuspiciousOffer:
    suspicious_clasfieds_id: int
    is_suspicious: bool

```

A także sam scraper, który jest implementacją klasy bazowej
```python
import logging
from datetime import datetime, timezone
from typing import Iterator

import requests
from dacite import from_dict

from src.config import log_init
from src.config.olx_config import (OLX_API_LIMIT, OLX_API_OFFSET,
                                   OLX_API_PAGINATION_LIMIT, OLX_API_URL,
                                   OLX_CATEGORIES)
from src.models.raw_offer import RawOffer, RawOfferLocation, RawOfferParameters
from src.raw_offer_producer.base import BaseRawOfferProducer

log_init.setup_logging()

logger = logging.getLogger(__name__)


class OlxRawOfferProducer(BaseRawOfferProducer):
    def __init__(
        self,
        olx_categories: dict = OLX_CATEGORIES,
        olx_api_url: str = OLX_API_URL,
        olx_api_limit: int = OLX_API_LIMIT,
        olx_api_pagination_limit: int = OLX_API_PAGINATION_LIMIT,
        olx_api_offset: int = OLX_API_OFFSET,
    ):
        self.olx_categories = olx_categories
        self.olx_api_url = olx_api_url
        self.olx_api_limit = olx_api_limit
        self.olx_api_pagination_limit = olx_api_pagination_limit
        self.olx_api_offset = olx_api_offset

    # TODO: handle duplicates
    def get_offers(self) -> Iterator[RawOffer]:
        for category in self.olx_categories.items():
            for offer in self._get_all_offers_from_category(category):
                yield self._map_offers(offer)

    def _olx_api_url_builder(self, page: int, category_id: int) -> str:
        offset = page * self.olx_api_offset
        url = f"{self.olx_api_url}?offset={offset}&limit={self.olx_api_limit}&category_id={category_id}&sort_by=created_at:desc"
        return url

    def _get_response(self, url: str) -> dict:
        data = requests.get(url)
        data.raise_for_status()
        return data.json()

    def _get_offers_for_category_with_page(self, page, category_id) -> list:
        category_url = self._olx_api_url_builder(page, category_id)
        olx_json = self._get_response(category_url)
        return olx_json["data"]

    def _get_all_offers_from_category(self, category: tuple) -> list:
        all_category_offers: list = []
        pagination_limit = self.olx_api_pagination_limit
        for page in range(pagination_limit):
            offers = self._get_offers_for_category_with_page(page, category[0])
            for offer in offers:
                all_category_offers.append(offer)
        logger.info(f"Found {len(all_category_offers)} in category_id: {category[1]}")
        return all_category_offers

    def _get_location(self, offer) -> RawOfferLocation:
        region = offer.get("location", {}).get("region", {}).get("name", "")
        city = offer.get("location", {}).get("city", {}).get("name", "")
        id = offer.get("id", "")
        return RawOfferLocation(id=id, region=region, city=city)

    def _get_images_list(self, offer) -> list[str]:
        return [image.get("link", "") for image in offer.get("photos", [])]

    def _map_offers(self, offer: dict) -> RawOffer:
        brand = offer.get("title", "").split()[0]
        parameters = self._get_offer_parameters(offer)
        return RawOffer(
            brand=brand,
            id=offer.get("id", ""),
            link=offer.get("url", ""),
            created_time=offer.get("created_time"),
            description=offer.get("description", "")
            .replace("\n", " ")
            .replace("\r", ""),
            title=offer.get("title", ""),
            image_links=self._get_images_list(offer),
            parameters=[parameters],
            location=[self._get_location(offer)],
            vin=str(parameters.vin).strip(),
            scraped_time=datetime.now(timezone.utc),
        )

    def _get_offer_parameters(self, offer: dict) -> RawOfferParameters:
        params = {
            param.get("key", ""): param.get("value", {}).get("label", "")
            for param in offer.get("params", [])
        }
        offer_id = offer.get("id", 0)

        if "price" in params:
            params["price"] = self.parse_price_to_int(params["price"])

        if "milage" in params:
            params["milage"] = self.parse_milage_to_int(params["milage"])

        params_defaults = {
            key: params.get(key, None)
            for key in RawOfferParameters.__annotations__.keys()
        }
        params_defaults["id"] = offer_id

        return from_dict(data_class=RawOfferParameters, data=params_defaults)

    def parse_price_to_int(self, price_str: str) -> int | None:
        price_str_cleaned = (
            price_str.replace("zł", "").replace(" ", "").replace(",", "")
        )
        try:
            return int(price_str_cleaned)
        except ValueError:
            return None

    def parse_milage_to_int(self, milage_str: str) -> int | None:
        if milage_str is None:
            return None
        milage_str_cleaned = (
            milage_str.replace("km", "").replace(" ", "").replace(",", "")
        )
        try:
            return int(milage_str_cleaned)
        except ValueError:
            return None
```
Struktury danych, do których mapowane są oryginalne dane, oraz implementacja scrapera zostały zaprojektowane tak, aby maksymalnie efektywnie przetwarzać zgromadzone informacje. Przetwarzane są tylko informacje opisujace charakterystykę obiektu sprzedaży, wszystkie pozostałe informacje są pomijane z uwagi na ich niską wartość lub wrażliwy charakter jak np. dane osobowe sprzedającego.

### OTOMOTO

OTOMOTO korzysta z bardziej złożonego GraphQL API, co wymaga dodatkowych uwierzytelnień i jest trudniejsze w implementacji stabilnego scrapingu danych. Jednakże, dzięki analizie struktury OLX, odkryto, że wiele ogłoszeń OTOMOTO jest integrowanych z OLX. Pozwoliło to na wykorzystanie istniejącego scrapera OLX z niewielkimi modyfikacjami, takimi jak dodanie identyfikatora użytkownika OTOMOTO do konstrukcji URL w funkcji _olx_api_url_builder:

```python
def _olx_api_url_builder(self, page: int, category_id: int) -> str:
    offset = page * self.olx_api_offset
    url = f"{self.olx_api_url}?offset={offset}&limit={self.olx_api_limit}&category_id={category_id}&sort_by=created_at:desc&query=&user_id=23063449&owner_type="

    return url
```

Dzięki tej metodzie udało się efektywnie łączyć dane z obu portali, co znacząco usprawniło proces gromadzenia danych dla dalszych etapów pracy, w tym oznaczania danych oraz budowania modeli ML. Szczegółowe analizy są dostępne w dedykowanych sekcjach dokumentacji projektu.

Zebrane dane zostałyt wykorzystane w kolejnych krokach, czyli [oznaczaniu danych](/pages/data_labeling/index.md) oraz budowaniu[modeli ML](/pages/analiza_modeli_ml/index.md). Z kolei szczegółowa analiza eksploracyjna widoczna jest w [osobnej sekcji](/pages/analiza_eksploracyjna/index.md)
