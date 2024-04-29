---
title: Analiza eksploracyjna - dostepne modele, wiek, stan, wyposażenie
---

```sql avg_price
select CONCAT(ROUND(avg(price), 2), ' PLN') as avg_price from offers_details
```

```sql avg_milage
select CONCAT(ROUND(avg(milage), 2), ' km') as avg_milage from offers_details
```

```sql avg_price_model
SELECT lower(brand) as brand, lower(model) as model, CONCAT(ROUND(AVG(price), 2), ' PLN') AS avg_price
FROM offers_base as ob left join offers_details as od on ob.clasfieds_id = od.clasfieds_id where lower(brand) in ( 'opel',
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
GROUP BY brand, model
ORDER BY AVG(price) DESC
```

```sql avg_milage_model
SELECT lower(brand) as brand, lower(model) as model, CONCAT(ROUND(AVG(milage), 2), ' km') AS avg_milage
FROM offers_base as ob left join offers_details as od on ob.clasfieds_id = od.clasfieds_id where lower(brand) in ( 'opel',
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
GROUP BY brand, model
ORDER BY AVG(milage) DESC
```

<BigValue 
  data={avg_price} 
  value=avg_price
  title="Przeciętna cena"
/>

<BigValue 
  data={avg_milage} 
  value=avg_milage
  title="Przeciętny przebieg"
/>

<DataTable data={avg_price_model} search=true>
    <Column id=brand />
    <Column id=model title="Średnia wartość modelu" />
    <Column id=avg_price />
</DataTable>

<DataTable data={avg_milage_model} search=true>
    <Column id=brand />
    <Column id=model title="Średnia wartość modelu" />
    <Column id=avg_milage />
</DataTable>


```sql colors_query
select color, count(*) as ct from offers_details group by color
```

<BarChart 
    data={colors_query} 
    x=color
    y=ct 
    xAxisTitle=Color
/>

----

```sql car_body_query
select car_body, count(*) as ct from offers_details group by car_body
```

<BarChart 
    data={car_body_query} 
    x=car_body
    y=ct 
    xAxisTitle=car_body
/>


----

```sql transmission_query
select transmission, count(*) as ct from offers_details group by transmission
```

<BarChart 
    data={transmission_query} 
    x=transmission
    y=ct 
    xAxisTitle=transmission
/>

----


```sql country_region
select country_origin, count(*) as ct from offers_details group by country_origin
```

<BarChart 
    data={country_region} 
    x=country_origin
    y=ct 
    xAxisTitle=country
/>

----

```sql righthanddrive_query
select righthanddrive, count(*) as ct from offers_details group by righthanddrive
```

<BarChart 
    data={righthanddrive_query} 
    x=righthanddrive
    y=ct 
    xAxisTitle=righthanddrive
/>

----

```sql petrol_query
select petrol, count(*) as ct from offers_details group by petrol
```

<BarChart 
    data={petrol_query} 
    x=petrol
    y=ct 
    xAxisTitle=petrol
/>

----
