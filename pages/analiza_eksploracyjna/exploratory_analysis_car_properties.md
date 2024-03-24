---
title: Analiza eksploracyjna - dostepne modele, wiek, stan, wyposażenie
---

```sql avg_price
select CONCAT(ROUND(avg(milage), 2), ' PLN') as avg_price from offers_details
```

```sql avg_milage
select CONCAT(ROUND(avg(milage), 2), ' km') as avg_milage from offers_details
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
