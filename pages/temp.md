

```sql brand_list
select DISTINCT lower(brand) as brand from offers_base where lower(brand) in ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
                     'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda', 'suzuki',
                     'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini', 'land rover',
                     'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')
```

```sql popular_in_region
WITH ModelCounts AS (
    SELECT
        ol.region,
        lower(ob.brand),
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
        lower(ob.brand),
        od.model
),
     RankedModels AS (
         SELECT
             region,
             lower(brand),
             model,
             model_count,
             ROW_NUMBER() OVER (PARTITION BY region ORDER BY model_count DESC) AS rank
         FROM
             ModelCounts
     )
SELECT
    region,
    lower(brand),
    model,
    model_count
FROM
    RankedModels
WHERE
        rank = 1;
```
<DataTable data={popular_in_region}/>


```sql most_brands_in_region
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

<DataTable data={most_brands_in_region}/>


```sql top_models
select count(*) as ct, model from offers_details group by model order by ct desc limit 10
```

<DataTable data={top_models}/>


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

```sql investigation
select ROUND((usa.ct::decimal / (usa.ct::decimal + al.ct::decimal)) * 100, 2) as percent, usa.ct as count_north_america, al.ct as total, usa.region
from (select count(*) as ct, region
      from offers_location as ol
               left join offers_details as od on ol.clasfieds_id = od.clasfieds_id
      where country_origin in ('Stany Zjednoczone', 'Kanada')
      group by region) as usa
         left join
     (select count(*) as ct, region
      from offers_location as ol
               left join offers_details as od on ol.clasfieds_id = od.clasfieds_id
      group by region) as al on usa.region = al.region

```

<DataTable data={investigation}>
  <Column id=region/>
    <Column id=count_north_america/>
    <Column id=total/>
  <Column id=percent fmt='0"%"'/>
</DataTable>


```sql no_vin_region
SELECT
    region,
    COUNT(*) AS total_vins,
    SUM(CASE WHEN is_valid_vin THEN 1 ELSE 0 END) AS valid_vins,
    SUM(CASE WHEN is_valid_vin = false THEN 1 ELSE 0 END) AS invalid_vins,
    ROUND(SUM(CASE WHEN is_valid_vin = false THEN 1 ELSE 0 END)::numeric / COUNT(*) * 100, 2) AS invalid_vin_percentage
FROM
    offers_location AS ol
LEFT JOIN
    (SELECT
         clasfieds_id,
         vin,
         CASE
             WHEN vin IS NULL OR
                  vin = 'None' OR
                  LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                  vin ~ '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww'
                 THEN false
             ELSE true
         END AS is_valid_vin
     FROM
         offers_base) AS vin
ON ol.clasfieds_id = vin.clasfieds_id
GROUP BY
    region;


```

<DataTable data={no_vin_region}>
  <Column id=region/>
  <Column id=invalid_vin_percentage fmt='0"%"'/>
  <Column id=invalid_vins/>
  <Column id=valid_vins/>
</DataTable>



```sql invalid_vin_brands
SELECT
    lower(brand) AS brand,
    COUNT(*) AS total_vins,
    SUM(CASE WHEN is_valid_vin THEN 1 ELSE 0 END) AS valid_vins,
    SUM(CASE WHEN NOT is_valid_vin THEN 1 ELSE 0 END) AS invalid_vins,
    ROUND(SUM(CASE WHEN NOT is_valid_vin THEN 1 ELSE 0 END)::numeric / COUNT(*) * 100, 2) AS invalid_vin_percentage
FROM
    (SELECT
         clasfieds_id,
         brand,
         CASE
             WHEN vin IS NULL OR
                  vin = 'None' OR
                  LENGTH(vin) NOT BETWEEN 16 AND 17 OR
                  vin ~ '1234567|11111111|22222222|33333333|44444444|55555555|66666666|77777777|88888888|99999999|00000000|zapytaj|wysylam|kontakt|zadzwon|nrvin|astaz|error|xxxxx|vvvvv|zzzzz|yyyyy|wwwww'
                 THEN false
             ELSE true
         END AS is_valid_vin
     FROM
         offers_base) AS derived_table
WHERE
    lower(brand) IN ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
                     'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda', 'suzuki',
                     'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini', 'land rover',
                     'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes')
GROUP BY
    lower(brand);
```

<DataTable data={invalid_vin_brands}>
  <Column id=brand/>
  <Column id=invalid_vin_percentage fmt='0"%"'/>
  <Column id=invalid_vins/>
  <Column id=valid_vins/>
</DataTable>


```sql created_time
SELECT
    COUNT(*) AS ct,
    CAST(strptime(created_time, '%Y-%m-%d %H:%M:%S') AS DATE) AS dt
FROM
    offers_base
WHERE created_time > '2023-01-01'
GROUP BY
    CAST(strptime(created_time, '%Y-%m-%d %H:%M:%S') AS DATE);
```
<CalendarHeatmap
    data={created_time}
    date=dt
    value=ct
    title="New offers daily"
    yearLabel=true
/>

```sql median_price_by_region
select region, min(price), max(price), median(price), avg(price)
from offers_base as ob
         left join offers_details od on ob.clasfieds_id = od.clasfieds_id
         left outer join offers_location ol on ob.clasfieds_id = ol.clasfieds_id
group by region
```

```sql region_list
select DISTINCT(region) as region from offers_location
```


<Dropdown data={region_list} name="region" value="region">
    <DropdownOption value="%" valueLabel="Regions"/>
</Dropdown>


<Dropdown data={brand_list} name="brand" value="brand">
    <DropdownOption value="%" valueLabel="Brands"/>
</Dropdown>

```sql test_scattered_plot
select lower(brand) as brand, milage, price from offers_base as ob left join offers_details as od on ob.clasfieds_id = od.clasfieds_id left join offers_location as ol on ob.clasfieds_id = ol.clasfieds_id
where  lower(brand) IN ('opel', 'ford', 'renault', 'audi', 'peugeot', 'skoda', 'toyota', 'volkswagen', 'bmw', 'volvo',
                     'hyundai', 'kia', 'mercedes-benz', 'nissan', 'citroën', 'fiat', 'seat', 'mazda', 'honda', 'suzuki',
                     'jeep', 'mitsubishi', 'dacia', 'porsche', 'chevrolet', 'lexus', 'alfa romeo', 'mini', 'land rover',
                     'dodge', 'jaguar', 'subaru', 'chrysler', 'mercedes') and region like '${inputs.region.value}' and brand like '${inputs.brand.value}'
```






<ScatterPlot 
    data={test_scattered_plot} 
    x=milage 
    y=price 
    series=brand 
    xAxisTitle=true 
    yAxisTitle=true
/>


 ```sql similarity
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
GROUP BY agreement
```
