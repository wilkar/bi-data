---
title: Analiza eksploracyjna - przegląd podstawowych informacji
--- 

Poniżej przedstawiam analizę eksploracyjną danych zebranych w procesie data scrapingu. W wyniku analizy udało się:

- usprawnić sam proces data scrapingu poprzez lepsze dopasowanie klas danych (dataclasses) do pobieranych danych
- przygotować moduł wyszukiwania podejrzanych ogłoszeń
- usprawnić moduł czyszczenia danych, dzieki czemu udało się poprawić skuteczność modeli ML


## Analiza wstępna

```sql total_ads_scraped
select count(*) as total_offers from offers_base
```
```sql total_ads_per_brand
select lower(brand) as brand, count(*) as total_offers from offers_base group by brand order by total_offers desc limit 20
```

```sql total_ads
select count(*) as ct from offers_base
```

<BarChart
data={total_ads_per_brand}
x=brand
y=total_offers
fillCollor='green'
></BarChart>
Na wykresie widoczne są najpopularniejsze marki aut dostępne w serwisach OLX oraz OTOMOTO. Nazwa marki została sprowadzona do małych liter (lower), ponieważ nazwa marki została wzięta z tytułu ogłoszenia. Użytkownicy wpisują markę auta na różne sposoby a czasami zaczynają tytuł od innych słów, stąd ten zestaw danych zawiera dużo niepoprawnych wartości.

```sql invalid_vin_values
select vin, count(*) ct from offers_base group by vin order by ct desc
```
<DataTable data={invalid_vin_values} rows=6/>


Jak widać pole VIN zawiera 128000 unikalnych rekordów na 2000000 ofert. Prawie 400000 ofert nie posiada VIN, a około 15000 ofert posiada wprowadzone nieprawiłowe dane. Oferty z nieprawidłowym VINem są traktowane jako podejrzane i posłużyły do budowy jednego z modeli danych. Na uwagę zasługuje też, że OLX i OTOMOTO nie posiadają żadnej walidacji VINu, która mogłaby powstrzymać najbardziej oczywiste błędne wartości.


```sql brands
select DISTINCT(lower(brand)) as brand from offers_base where lower(brand) in ( 'opel',
                            'ford',
                            'renault',
                            'audi',
                            'peugeot',
                            'skoda',
                            'toyota',
                            'volkswagen',
                            'bmw',
                            'volvo',
                            'hyundai',
                            'kia',
                            'mercedes-benz',
                            'nissan',
                            'citroën',
                            'fiat',
                            'seat',
                            'mazda',
                            'honda',
                            'suzuki',
                            'jeep',
                            'mitsubishi',
                            'dacia',
                            'porsche',
                            'chevrolet',
                            'lexus',
                            'alfa romeo',
                            'mini',
                            'land rover',
                            'dodge',
                            'jaguar',
                            'subaru',
                            'chrysler',
                            'mercedes',
                            'saab',
                            'citroen',
                            'smart',
                            'infiniti',
                            'ssangYong',
                            'lancia',
                            'daihatsu',
                            'daewoo',
                            'aixam',
                            'cadillac',
                            'polonez')
```
<Dropdown data={total_ads_per_brand} name=brand value=brand>
    <DropdownOption value="%" valueLabel="Brands"/>
</Dropdown>


<Dropdown name=year>
    <DropdownOption value=% valueLabel="All Years"/>
    <DropdownOption value=2021/>
    <DropdownOption value=2022/>
    <DropdownOption value=2023/>
    <DropdownOption value=2024/>
</Dropdown>

```sql created_at_aggregated
select date_trunc('month', created_time) as month,
    count(*) as number_of_adds,
    (lower(brand)) as brand
from offers_base
where lower(brand) like '${inputs.brand.value}'
and date_part('year', created_time) like '${inputs.year.value}' and lower(brand) in ('opel',
                            'ford',
                            'renault',
                            'audi',
                            'peugeot',
                            'skoda',
                            'toyota',
                            'volkswagen',
                            'bmw',
                            'volvo',
                            'hyundai',
                            'kia',
                            'mercedes-benz',
                            'nissan',
                            'citroën',
                            'fiat',
                            'seat',
                            'mazda',
                            'honda',
                            'suzuki',
                            'jeep',
                            'mitsubishi',
                            'dacia',
                            'porsche',
                            'chevrolet',
                            'lexus',
                            'alfa romeo',
                            'mini',
                            'land rover',
                            'dodge',
                            'jaguar',
                            'subaru',
                            'chrysler',
                            'mercedes',
                            'saab',
                            'citroen',
                            'smart',
                            'infiniti',
                            'ssangYong',
                            'lancia',
                            'daihatsu',
                            'daewoo',
                            'aixam',
                            'cadillac',
                            'polonez')
group by month, lower(brand)
order by number_of_adds desc
```
<BarChart
data={created_at_aggregated}
title="Liczba ofert w czasie"
x=month
y=number_of_adds
series=brand
/>

Powyższy wykres przedstawia ilość aktualnie aktywnych ogłoszeń wraz z datą ich dodania. Sam wykres stanie się kompletny dopiero w dłuższej perspektywie czasu, gdy zebrane zostanie wystarczącjąco dużo danych. Będzie można wtedy śledzić ilość dodanych ofert oraz długość ich życia.

```sql scraped_date
SELECT DATE_TRUNC('day', scraperd_time) AS scraped_date, COUNT(*) AS ct 
FROM offers_base 
WHERE DATE_TRUNC('month', scraperd_time) >= '2024-03-01' 
GROUP BY scraped_date 
ORDER BY scraped_date DESC;
```

<BarChart
data={scraped_date}
x=scraped_date
y=ct
fillCollor='488f96'
>
  <ReferenceArea xMin="2024-03-11" xMax="2024-03-13" label="Pierwszy pełny obieg scrapera" color=red/> </BarChart>

Powyższy wykres przestawia ilość zebranych ofert w każdym obiegu scrapera. Jak widać pierwszy run pobrał prawie 40k ofert z całego OLX. OLX ogranicza paginację API do 25 stron, więc nie udało się pobrać całej bazy strony. Po tym wprowadzono zmiany w scraperze, który od tej pory zaczynał scraping od ostatnio dodanych, więc kolejne obiegi scrapera pokazują ilość nowo doddanych ofert do serwisu. Ilość ta utrzymuje się na stałym poziomie około 5k dziennie.
