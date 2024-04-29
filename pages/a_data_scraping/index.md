---
title: Scraping danych
---

## Wstęp

Pierwszym krokiem niezbędnym w realizacji projektu było pozyskanie niezbędnych danych. Jako że dostęp do bezpłatnych baz danych, zawierających niezbędne informacje, jest ograniczony, konieczne stało się zastosowanie techniki scrapingowej.


### Data scraping a kwestie prawne

### Dane osobowe

Definicja danych osobowych przygowoana przez Komisję Europejską:

    Dane osobowe stanowią wszelkie informacje dotyczące zidentyfikowanej lub możliwej do zidentyfikowania żyjącej osoby fizycznej. Poszczególne informacje, które w połączeniu ze sobą mogą prowadzić do zidentyfikowania tożsamości danej osoby, także stanowią dane osobowe.

Na terenie Unii Europejskiej stosuje się Rozporządzenie Ogólne o Ochronie Danych (RODO), które zapewnia osobom fizycznym kontrolę nad ich danymi osobowymi oraz reguluje sposób ich przechowywania i przetwarzania przez różne organizacje.

W kontekście analizowanego projektu, dane dotyczą wyłącznie przedmiotów sprzedaży. Informacje o sprzedawcach, takie jak imiona czy numery kontaktowe, mogą być czasami uwzględnione w opisach ogłoszeń. To z kolei może umożliwić identyfikację osób na podstawie danych zamieszczonych na zdjęciach, np. numerów tablic rejestracyjnych sprzedawanych aut.
Aby w pełni sprostać wymaganiom dyrektywy RODO należałoby wdrożyć odpowieni mechanizm wycinający dane osobowe z opisu ogłoszenia na etapie scrapingu, ale przed zapisem danych do bazy danych.

### Własność intelektualna

Tematych praw autorskich to temat niezwykle obszerny i trudny do rozpatrywania zwłaszcza w kontekście scrapingu danych. Najczęściej właściciele serwisów ogłoseniowych są tez właścicielami praw autorskich treści tam zamieszczanych. Użytkownik w momencie akceptacji regulaminu i publikacji ogłoszenia najczęsciej przekazuje prawa autorskie.

Chociaż scraping jest niezbędny dla działania wyszukiwarek takich jak Google, umożliwiając indeksowanie i wyświetlanie odpowiednich wyników wyszukiwania, także otwiera możliwości dla firm trzecich do zbierania danych już przetworzonych i moderowanych przez serwisy ogłoszeniowe. Scraping danych może być traktowany jako działanie na granicy prawa, zwłaszcza gdy dane są później odsprzedawane.

W przypadku analizowanego problemu dochodzi do zbierania i agregowania danych, co jest niezgodne z regulaminem. Jednak dane nie są przekazywane do podmiotów trzecich, a ich wykorzystanie ogaranicza się jedynie do przygotowania pracy inżynierskiej.

![Fragment regulaminu OLX](/static/assets/olx_regulamin.png)
Screen z regulaminu [OLX](https://pomoc.olx.pl/olxplhelp/s/article/aktualny-regulamin-V32-olx#0.1_r13)


### Zgodność z polityką scrapingu

Wiele serwisów internetowych stosuje plik robots.txt, który określa zasady dotyczące scrapingowania danych. Reguły te mogą obejmować zarówno częstotliwość zapytań, jak i dostęp do konkretnych podstron serwisu. Przestrzeganie tych zasad minimalizuje ryzyko takich konsekwencji jak zablokowanie dostępu do strony.

Pod adresem wykorzystywanym przez scraper (OLX_API_URL = "https://www.olx.pl/api/v1/offers/"), znajdują się reguły Allow, co oznacza zgodność z polityką serwisu OLX dotyczącą scrapingowania danych.

![Screenshot pliku robotz.txt](/static/assets/olx_robots_txt.png)


### Spis treści bieżącej sekcji

1. [Scraping forum internetowego bez-wypadkowe.net](/data_scraping/bezwypadkowe)
1. [Scraping ofert Otomoto i OLX](/data_scraping/otomoto_and_olx)
