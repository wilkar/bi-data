---
title: Data Labeling
---

## Oznaczanie Danych dla Wykrywania Oszustw w Ofertach Ogłoszeniowych

Oznaczanie danych jako podejrzanych jest kluczowym krokiem w procesie wykrywania oszustw na portalach ogłoszeniowych. Zadanie to napotyka na liczne wyzwania, takie jak brak ujednoliconej bazy danych z przykładami oszustw oraz brak narzędzi automatyzujących selekcję podejrzanych ofert. Poniżej przedstawiono dwa podejścia do identyfikacji ofert o wysokim ryzyku oszustwa, które wykorzystują różne źródła danych i metody analizy.

### Metodologia

Tworzenie poprawnie działającego modelu ML zależy od jakości i poprawności danych użytych do jego treningu. Zbalansowany i dobrze oznakowany zbiór danych jest fundamentem dla wiarygodnych wyników.

Wyzwania w przygotowaniu odpowiedniego zbioru danych:

- Brak jednolitych baz danych z przykładami oszustw.
- Brak bezpłatnych narzędzi wspierających selekcję podejrzanych ofert.
- Trudność w zebraniu danych odpowiedniej jakości.

### Metody oznaczania (labelingu) danych

#### Analiza Numerów VIN

Eksploracyjna analiza danych wykazała, że duża grupa ofert nie posiada poprawnych numberów VIN. Podstawą tej metody było założenie, że ogłoszeniodawczy nie ujawniający numery VIN chcą zataić ukryte wady lub niewygodne informacje.

Sprawdzono to za pomocą prostego zapytania SQL:

    ```sql
        select
            count(*) as ct,
            vin
        from offers_base
        group by vin
        order by ct desc
    ```

Poniżej znajduje się 10 najczęściej powtarzających się wyników

| ct    | vin               |
| ----- | ----------------- |
| 42629 | None              |
| 7191  | xxxxxxxxxxxxxxxxx |
| 6321  | XXXXXXXXXXXXXXXXX |
| 1592  | 11111111111111111 |
| 814   | Xxxxxxxxxxxxxxxxx |
| 140   | vvvvvvvvvvvvvvvvv |
| 137   | ZAPYTAJSPRZEDAWCE |
| 126   | 12345678901234567 |
| 96    | zzzzzzzzzzzzzzzzz |
| 92    | YYYYYYYYYYYYYYYYY |


Poniższy diagram prezentuje logikę budowania zestawu danych z podejrzanymi ofertami.

![Data Labelling by VIN](/assets/data_labelling_by_vin.png)

Wyniki zostały zapisane w tabeli `suspicious_offers` z następującym wynikiem:

    ```csv
    ct,suspicious
    135777,false
    45977,true
    ```

---

#### Analiza Opisów Ogłoszeń

Druga metoda oznaczania podejrzanych ogłoszeń przewidywała użycie potwierdzonych danych. Pomocne tutaj okazało się formum bez-wypadkowe.net, gdzie użytkownicy umiejszczają opisy oszustw wraz ze zdjęciami oraz numerami VIN.

Początkowo wykonano scraping forum, zbierając dane o uszkodzonych lub powypadkowych samochodach. Następnie, korzystając z analizy tekstu, zidentyfikowano ogłoszenia o podobnych opisach.

Diagram scrapingu forum internetowego:
![Bez-wypadkowe.net scraping](/assets/labelling_data_scraping.png)

Wynikiem pobierania danych było 613 numerów VIN co do których można było mieć pewność, że wskazują na auta uszkodzone lub powypadkowe.

```sql
select
    count(*) as ct
from labeling_data
```

```txt
ct
613
```

Co więcej po połączeniu tabeli zebranymi z forum internetowego wraz z tabelą zawierającą wszystkie ogłoszenia okazało się, że otrzymano tylko 67 wyników. Czyli w całkowitym zbiorze ponad 200000 ofert mamy tylko 67 potwierdzonych przypadków oszustw. Zestaw danych w takiej formie nie był dobrym metriałem do przeprowadzenia eksperytmentu ze względu na zbyt duże dysproporcje w zestawie danych.

```sql
select
    count(*)
from offers_base as ob
        inner join labeling_data ld on ob.vin = ld.vin
```

```txt
ct
67
```

Aby zbalansować zestaw danych podjęta została próba posłużenia się opisem ogłoszenia z 67 wytypowanych ofert w celu odnalezienia ogłoszeń posiadających podobny opis. Kierując się założeniem, że ogłoszeniodawcy wystawiający oferty posługują się podobnym stylem tworzenia opisów ofert.
W tym celu zbudowano osobny moduł, które diagram sekwencyjny znajduje się poniżej.

![Data Labelling by Description](/assets/data_labelling_by_description.png)


Wyniki zostały zapisane w tabeli `suspicious_offers_v2` z następującym wynikiem:

```csv
ct, is_suspicious
141938,false
40218,true

```

### Implikacje

Oba podejścia dostarczyły bardzo zbliżony zestaw danych, gdzie około 40000 zostało oznaczone jako potencjalnie podejrzane. Dokonując też szybkiej analizy obu zestawów danych można zauważyć, że oba zestawy danych znacząco się od siebie różnią w klasyfikacji podejrzanych ofert.

```sql
SELECT
    agreement,
    COUNT(*) AS count
FROM (
         SELECT
             CASE
                 WHEN so.is_suspicious = so2.is_suspicious THEN 'Agree'
                 ELSE 'Disagree'
                 END AS agreement
         FROM suspicious_offers so
                  JOIN suspicious_offers_v2 so2 ON so.suspicious_clasfieds_id = so2.suspicious_clasfieds_id
     ) sub
GROUP BY agreement;
```

### Kolejne kroki

Tak zbudowane zbiory danych zostały wykorzystane do zbudowania modeli ML, które są omówione w sekcji [Analiza modeli ML](/d_analiza_modeli_ml/)

### Spis treści bieżącej sekcji

1. [Oznaczanie Danych przy pomocy VIN](/b_data_labeling/labeling_by_description)
1. [Oznaczanie Danych przy pomocy opisu ogłoszenia](/b_data_labeling/labeling_by_vin)
