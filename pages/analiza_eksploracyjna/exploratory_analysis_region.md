---
title: Analiza eksploracyjna - miejsca publkowania ogłoszeń
---


```sql city_list
select city, region from offers_location where region like '${inputs.region.value}'
```

```sql region_list
select DISTINCT(region) as region from offers_location
```

<Dropdown data={region_list} name="region" value="region">
    <DropdownOption value="%" valueLabel="Regions"/>
</Dropdown>
<Dropdown data={city_list} name="city" value="city" where={`region like '${inputs.region.value}'`}>
<DropdownOption value="%" valueLabel="Cities"/>
</Dropdown>


```sql regions
select region, count(*) as ct from offers_location where region like '${inputs.region.value}' group by region order by ct desc

```

```sql cities
select city, region, count(*) as ct from offers_location where region like '${inputs.region.value}' group by region, city
```

<BarChart 
 data={regions}
 x=region 
 y=ct
 swapXY=true
/>

<DataTable 
 data={cities}
 search=true
/>
