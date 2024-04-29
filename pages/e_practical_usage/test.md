---
title: Kierunki rozwoju
---
### Kierunki rozwoju

Prezentowany projekt miał charakter przekrojowy na kilku płaszczyznach
- działania aplikacji od pozyskiwania danych po praktyczne zastosowanie
- podejścia do oznaczania podejrzanych danych
- wykorzystania modeli uczenia maszynowego

Istnieje wiele obszarów, które należy usprawnić w celu poprawy działania poszczególnych komponentów oraz całości aplikacji.

#### Scraping danych
Możliwe usprawnienia:
- refaktor ułatwiający dodawania kolejnych scraperów danych
- refaktor bazy danych w celu dodania większej ilości danych opisujących ofertę
- implementacja pełnego paralelizmu, aby przyspieszyć proces zbierania danych
- współpraca z dostawcami danych np. portalami ogłoszeniowymi w celu całkowitego wyelimnowania scrapingu z aplikacji

#### Oznaczanie danych
Możliwe usprawnienia:
- stworzenie interfejsu użytkownika, który umożliwiłby ręczne oznaczanie podejrzanych ofert
- zastosowanie zaawansowanych technik NLP jak BERT lub GPT, które mogłby lepiej radzić sobie z wyłapywaniem podejrzanych ofert

#### Budowanie modelu ML
Możliwe usprawnienia:
- Zastosowanie hiperparametrów oraz technik automatycznego doboru parametrów jak Grid Search lub Bayesian Optimization może poprawić dokładność modeli.
- zastosowanie technik balansowania jak oversampling lub undesampling zbiorów danych może zwiększyć skuteczność modeli.
- użycie modelu `ensemble`, czyli połączenia kilku rożnych modeli może zwiększyć dokładność i stabliność działania modeli.

#### Ukierunkowanie aplikacji na użytkownika oraz produktyzacja
Możliwe usprawnienia:
- Tworzenie API pozwalajacego na interakcję z modelami ML oraz danymi, a także testowanie wybranych ofert ułatwiloby korzystanie z aplikacji dla osób nie znających kodu źródłowego.
- Dodanie testów jednostkowych oraz testów E2E (End-2-End) zapewniłoby stabilność aplikacji.
- Dodanie środowiska CI/CD (Continuous Integration / Continuous Deployment) dodałoby kolejną warstwę ułątwiającą zarządzanie zmianami oraz monitorowanie działania aplikacji.
- Deployment w środowisku "produkcyjnym" na serwerze dopasowanym do zadanego zastosowania przyspieszyłby działanie aplikacji.
