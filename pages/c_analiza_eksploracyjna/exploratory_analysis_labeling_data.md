---
title: Analiza eksploracyjna - oznaczanie podejrzanych ofert
---

```sql label
select * from labeling_data;
```
Wykresy pokazują ilość ofert ze względu na lokalizację ogłoszenia oraz na markę. Wyniki zawężone są tylko do ofert znajdujących się na forum bez-wypadkowe.net. Biorąc pod uwagę lokalizację, najwięcej ogłoszeń jest z miast gdzie w ogóle jest publikowane dużo ofert, więc nie jest to w żaden sposób znaczące. Interesujący jest fakt, które marki znalazły się na liście najpopularniejszych w tym zestawieniu. Może to świadczyć o popularności tych marek wsród użytkowników forum. 

```sql label_brand
select lower(ob.brand) as brand, count(*) as ct
from offers_base as ob
         inner join labeling_data as ld on ob.vin = ld.vin
group by ob.brand
order by ct desc
```

<Chart data={label_brand} x=brand y=ct>
    <Bar/>
</Chart>


```sql label_city
select lower(ob.city) as city, count(*) as ct
from (select vin, city
      from offers_base as ob
               left join offers_location as ol on ob.clasfieds_id = ol.clasfieds_id) as ob
         inner join labeling_data as ld on ob.vin = ld.vin
group by ob.city
order by ct desc
```

<Chart data={label_city} x=city y=ct>
    <Bar/>
</Chart>
