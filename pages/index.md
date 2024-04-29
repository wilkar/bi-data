---
title: Strona startowa
---

### Wprowadzenie


## Projekt: Uczenie maszynowe w obróbce danych z motoryzacyjnych portali e-commerce

Raport ten przedstawia studium przypadku projektowania i implementacji aplikacji służącej do wykrywania fałszywych lub nieuczciwych ofert na motoryzacyjnych portalach ogłoszeniowych.

Celem projektu jest stworzenie rozwiązania umożliwiającego automatyczne wykrywanie oszustw z wysokim stopniem prawdopodobieństwa.

Projekt charakteryzuje się samowystarczalnością, w ramach której znajdują się moduły zbierające, agregujące i ewaluujące dane, a rezultatem jest współczynnik prawdopodobieństwa sygnalizujący, czy dana oferta może być oszustwem.

### Spis treści
- [Data Scraping](/a_data_scraping/)
  - [Bezwypadkowe](/a_data_scraping/bezwypadkowe)
  - [Otomoto and OLX](/a_data_scraping/otomoto_and_olx)
- [Data Labeling](/b_data_labeling/)
  - [Labeling by Description](/b_data_labeling/labeling_by_description)
  - [Labeling by VIN](/b_data_labeling/labeling_by_vin)
- [Exploratory Analysis](/c_analiza_eksploracyjna/)
  - [Exploratory Analysis Overview](/c_analiza_eksploracyjna/exploratory_analysis)
  - [Exploratory Analysis: Car Properties](/c_analiza_eksploracyjna/exploratory_analysis_car_properties)
  - [Exploratory Analysis: Labeling Data](/c_analiza_eksploracyjna/exploratory_analysis_labeling_data)
  - [Exploratory Analysis: Region](/c_analiza_eksploracyjna/exploratory_analysis_region)
- [ML Models Analysis](/d_analiza_modeli_ml/)
  - [Ada-Boost Classifier](/d_analiza_modeli_ml/ada-boost-classfier)
  - [Extra Trees Classifier](/d_analiza_modeli_ml/extra-trees-classifier)
  - [Gradient Boosting Classifier](/d_analiza_modeli_ml/gradient-boosting-classifier)
  - [K-Neighbors Classifier](/d_analiza_modeli_ml/k-neighbors-classifier)
  - [Logistic Regression](/d_analiza_modeli_ml/logistic-regression)
  - [Random Forest Classifier](/d_analiza_modeli_ml/random-forest-classifier)
- [Practical Usage](/e_practical_usage/)
  - [Test](/e_practical_usage/test)
