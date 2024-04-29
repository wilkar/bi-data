---
title: Eksploatacyjna analiza danych
---

### Definicja
Analiza danych to proces polegający na sprawdzaniu, porządkowaniu, przekształcaniu i modelowaniu danych w celu zdobycia użytecznych informacji, wypracowania wniosków i wspierania procesu decyzyjnego. Analiza danych ma wiele aspektów i podejść, obejmujących różne techniki pod różnymi nazwami, w różnych obszarach biznesowych, naukowych i społecznych. Praktyczne podejście do definiowania danych polega na tym, że dane to liczby, znaki, obrazy lub inne metody zapisu, w formie, którą można ocenić w celu określenia lub podjęcia decyzji o konkretnym działaniu. Wiele osób uważa, że dane same w sobie nie mają znaczenia - dopiero dane przetworzone i zinterpretowane stają się informacją.

### Proces
W przypadku zestawu danych będącego przedmiotem pracy, analiza danych miała charakter iteracyjny i była powtarzana po kazdej zmianie wprowadzanej w scraperze danych. Prezentowane wykresy i opisy są ostateczną wersją zbudowaną, gdy aplikacja zbierająca dane oraz struktury danych wystarczająco się ustabilizowały. W toku analizy sprawdzano:
- Jakość danych - czy wszystkie informacje sa prawidłowo pobierane.
- Ilość danych - czy ilość pobranych ofert w bazie zgadza się z początkowymi estymacjami.
- Dystrybucję danych - czy wszystkie marki, modele, regiony są reprezentowane w zbiorze danych w ilości zgodnej zgodnie z powszechną wiedzą. Na przykład czy marki najpopularniejsze w Polsce mają też proporcjonalną reprezentację w zestawie danych.


### Możliwe usprawnienia
Czyszczenie danych zostało wykonane adhoc w warstwie wizualizacji danych. Należałoby stworzyć osobny moduł w aplikacji zbierającej dane, który przejmie tę odpowiedzialność. Należałoby też stworzyć testy jednostkowe pokrywające najpopularniejsze błędy.

### Spis treści bieżącej sekcji

1. [Analiza eksploracyjna - przegląd podstawowych informacji](/analiza_eksploracyjna/exploratory_analysis)
1. [Analiza eksploracyjna - dostepne modele, wiek, stan, wyposażenie](/analiza_eksploracyjna/exploratory_analysis_car_properties)
1. [Analiza eksploracyjna - miejsca publkowania ogłoszeń](/analiza_eksploracyjna/exploratory_analysis_region)
1. [Analiza eksploracyjna - oznaczanie podejrzanych ofert](/analiza_eksploracyjna/exploratory_analysis_labeling_data)
