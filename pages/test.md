```sql brand_list
select DISTINCT lower(brand) as brand from offers_base where lower(brand) in ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
                     'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda', 'suzuki',
                     'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini', 'land rover',
                     'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')
```

### Summary
```sql total_offers
select count(*) as ct from offers_base where lower(brand) like '${inputs.brand.value}'
```
```sql avg_price
select CONCAT(CAST(ROUND(AVG(price), 2) AS VARCHAR), ' zł') as avg_price from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where brand like '${inputs.brand.value}'
```
```sql median_price
select CONCAT(CAST(ROUND(MEDIAN(price), 2) AS VARCHAR), ' zł') as mdn_price from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where brand like '${inputs.brand.value}'
```
```sql avg_milage
select CONCAT(CAST(ROUND(AVG(milage), 2) AS VARCHAR), ' km') as avg_milage from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where brand like '${inputs.brand.value}'
```
```sql median_milage
select CONCAT(CAST(ROUND(MEDIAN(milage), 2) AS VARCHAR), ' km') as mdn_milage from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where brand like '${inputs.brand.value}'
```
```sql std_dev_price
select stddev(price) as std_dev from offers_details
```
```sql std_dev_milage
select stddev(milage) as std_dev from offers_details
```
```sql count_brands
select count(distinct lower(brand)) as ct_brands from offers_base where lower(brand) IN ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
                     'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda', 'suzuki',
                     'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini', 'land rover',
                     'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')
```
```sql count_models
select count(distinct lower(model)) as ct_models from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id  where brand like '${inputs.brand.value}'
```

<Dropdown data={brand_list} name="brand" value="brand">
    <DropdownOption value="%" valueLabel="Brands"/>
</Dropdown>
<br>
<BigValue 
  data={total_offers} 
  value=ct
  title="Ilość zebranych ogłoszeń"
/>
<br>
<BigValue 
  data={avg_price} 
  value=avg_price
  title="Przeciętna (średnia arytmetyczna) cena oferty"
/>
<BigValue 
  data={median_price} 
  value=mdn_price
  title="Przeciętna (mediana) cena oferty"
/>
<br>
<BigValue 
  data={avg_milage} 
  value=avg_milage
  title="Przeciętny (średnia arytmentyczna) przebieg"
/>
<BigValue 
  data={median_milage} 
  value=mdn_milage
  title="Przeciętny (mediana) przebieg"
/>
<br>
<BigValue 
  data={std_dev_milage} 
  value=std_dev
  title="Odchylenie standardowe dla przebiegu"
/>
<BigValue 
  data={std_dev_price} 
  value=std_dev
  title="Odchylenie standardowe dla ceny"
/>
<br>
<BigValue 
  data={count_brands} 
  value=ct_brands
  title="Ilość producentów"
/>
<br>
<BigValue 
  data={count_models} 
  value=ct_models
  title="Ilosć modeli"
/>

---
### Histograms
```sql milage_histogram
select milage from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where milage < 500000 and milage is not null and brand like '${inputs.brand.value}'

```

    <Histogram
    data={milage_histogram} 
    x=milage 
    fmt=num2
/>


```sql price_histogram
select price from offers_details as od left join offers_base as ob on od.clasfieds_id = ob.clasfieds_id where price is not null and price < 400000 and brand like '${inputs.brand.value}'
```

    <Histogram
    data={price_histogram} 
    x=price 
    fmt=num2
/>

---
### Regions

```sql count_by_region
select count(*) as ct, region from offers_location group by region
```
```sql brand_popular_in_region
WITH ModelCounts AS (
    SELECT
        ol.region,
        ob.brand,
        COUNT(*) AS brand_count
    FROM
        offers_base AS ob
            LEFT JOIN offers_details AS od
                      ON ob.clasfieds_id = od.clasfieds_id
            LEFT OUTER JOIN offers_location AS ol
                            ON ob.clasfieds_id = ol.clasfieds_id
    GROUP BY
        ol.region,
        ob.brand
),
     RankedModels AS (
         SELECT
             region,
             brand,
             brand_count,
             ROW_NUMBER() OVER (PARTITION BY region ORDER BY brand_count DESC) AS rank
         FROM
             ModelCounts
     )
SELECT
    region,
    brand,
    brand_count
FROM
    RankedModels
WHERE
        rank = 1;
```
```sql model_popular_in_region
WITH ModelCounts AS (
    SELECT
        ol.region,
        LOWER(ob.brand) as brand,
        od.model,
        COUNT(*) AS model_count
    FROM
        offers_base AS ob
    LEFT JOIN offers_details AS od
        ON ob.clasfieds_id = od.clasfieds_id
    LEFT OUTER JOIN offers_location AS ol
        ON ob.clasfieds_id = ol.clasfieds_id
    GROUP BY
        ol.region,
        LOWER(ob.brand),
        od.model
),
RankedModels AS (
    SELECT
        region,
        CONCAT(UPPER(SUBSTR(brand, 1, 1)), SUBSTR(brand, 2)) as brand,
        model,
        model_count,
        ROW_NUMBER() OVER (PARTITION BY region ORDER BY model_count DESC) AS rank
    FROM
        ModelCounts
)
SELECT
    region,
    brand,
    model,
    model_count
FROM
    RankedModels
WHERE
    rank = 1;
```
<BarChart 
    data={count_by_region}
    x=region 
    y=ct 
/>
<DataTable data={brand_popular_in_region}/>
<DataTable data={model_popular_in_region}/>

```sql country_origin_region

select * from (SELECT COUNT(*) as ct,
       ol.region,
       CASE
           WHEN country_origin = 'Polska' THEN 'Polska'
           WHEN country_origin IN
                ('Austria', 'Belgia', 'Bułgaria', 'Chorwacja', 'Czechy', 'Dania', 'Estonia', 'Finlandia', 'Francja',
                 'Grecja', 'Hiszpania', 'Holandia', 'Irlandia', 'Litwa', 'Luksemburg', 'Łotwa', 'Malta', 'Niemcy',
                 'Polska', 'Portugalia', 'Rumunia', 'Słowacja', 'Słowenia', 'Szwecja', 'Węgry', 'Włochy') THEN 'EU'
           ELSE 'non-EU'
           END AS origin

FROM offers_details AS od
         LEFT JOIN offers_location AS ol ON od.clasfieds_id = ol.clasfieds_id

GROUP BY origin, region)
```
<BarChart 
    data={country_origin_region} 
    x=region 
    y=ct 
    series=origin
    type=stacked100
/>

---
### VIN

```sql vin_analysis
select count(*) as ct, vin from offers_base group by vin order by ct desc
```
<DataTable data={vin_analysis}/>


```sql invalid_vins
select count(*) as invalid_vins_count from offers_base where  vin IS NULL OR
                  vin = 'None' OR
                  LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                  vin ~ '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww'
```
```sql valid_vins
select count(*) as valid_vins_count from offers_base where not ( vin IS NULL OR
                  vin = 'None' OR
                  LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                  vin ~ '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww')
```
<BigValue 
  data={invalid_vins} 
  value=invalid_vins_count
  title="Ilość ogłoszeń zawierajacych niepoprawne numery vin"
/>
<BigValue 
  data={valid_vins} 
  value=valid_vins_count
  title="Ilość ogłoszeń zawierajacych poprawne numery vin"
/>

```sql invalid_vin_region
SELECT dt.region, dt.is_valid_vin, count(*) as ct
FROM (select region,
             CASE
                 WHEN vin IS NULL OR
                      vin = 'None' OR
                      LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                      vin ~
                      '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww'
                     THEN 'invalid_vin'
                 ELSE 'valid_vin'
                 END      AS is_valid_vin
      from offers_base as ob left outer join offers_location ol on ob.clasfieds_id = ol.clasfieds_id
      WHERE lower(brand) IN
            ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
             'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda',
             'suzuki',
             'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini',
             'land rover',
             'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')) as dt
GROUP BY region, is_valid_vin;
```

<BarChart 
    data={invalid_vin_region} 
    x=region 
    y=ct 
    series=is_valid_vin
    type=stacked100
    swapXY=true
/>



```sql invalid_vin_brands
SELECT dt.brand, dt.is_valid_vin, count(*) as ct
FROM (select lower(brand) as brand,
             CASE
                 WHEN vin IS NULL OR
                      vin = 'None' OR
                      LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                      vin ~
                      '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww'
                     THEN 'invalid_vin'
                 ELSE 'valid_vin'
                 END      AS is_valid_vin
      from offers_base
      WHERE lower(brand) IN
            ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
             'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda',
             'suzuki',
             'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini',
             'land rover',
             'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')) as dt
GROUP BY brand, is_valid_vin;
```

<BarChart 
    data={invalid_vin_brands} 
    x=brand 
    y=ct 
    series=is_valid_vin
    type=stacked100
    swapXY=true
/>
 
