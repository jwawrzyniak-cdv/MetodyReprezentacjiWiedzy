# Laboratorium 6: zadania dodatkowe

Pięć zadań dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne i można je wykonywać w dowolnej kolejności, więc prowadzący może przydzielić różnym osobom różne zadania. Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie. Zadanie D nadaje się na zalążek projektu zaliczeniowego.

## Wspólne przygotowanie

Na początek notebooka skopiujcie z laboratorium komórkę instalacyjną, importy oraz komórkę z funkcją `populacja` i regulaminem `REGULY` (zadanie 4). Zadanie C potrzebuje też modułu `hmac` z biblioteki standardowej. Sprawdźcie, że wszystko działa:

```python
%pip install -q rdflib owlrl pgmpy

import time, hashlib, hmac, itertools
import numpy as np
import pandas as pd
from rdflib import Graph, Namespace, Literal, RDF

RANDOM_STATE = 42
U = Namespace("http://example.org/uczelnia#")

# tu wklejcie funkcję populacja() i listę REGULY z notebooka głównego
pop = populacja()
print(pop.shape, int(pop.urlop.sum()))
```

Powinno się wypisać `(3000, 8) 389`.

## Zadanie A: Prywatność różnicowa dla zapytań agregujących do grafu

Analityk może zadawać grafowi populacji wyłącznie zapytania `COUNT` z grupowaniem, a odpowiedzi są zaszumiane mechanizmem Laplace'a.

1. Zbudujcie graf RDF populacji: dla każdej osoby typ `u:Student` i właściwości `u:plec`, `u:kierunek`, `u:urlop`. Zapytaniem SPARQL z `GROUP BY` policzcie liczbę osób z urlopem w każdej kombinacji kierunku i płci.
2. Uzasadnijcie, że czułość L1 takiego histogramu wynosi 1. Napiszcie funkcję `laplace(wartosci, eps, czulosc)` i pokażcie histogram dokładny oraz zaszumiony dla epsilon 0,1 i 1.
3. Dla epsilon w [0,05, 0,1, 0,5, 1, 2, 5] oszacujcie (2000 powtórzeń) średni błąd bezwzględny komórki i błąd względny najmniejszej komórki. Porównajcie z teorią.
4. Przyjmijcie łączny budżet epsilon = 1 i rozłóżcie go równo na q zapytań (q w [1, 5, 20, 100]). Jak rośnie błąd jednej odpowiedzi?

Dla których komórek histogram jest użyteczny przy epsilon = 0,1, a dla których nie? Dlaczego w grafie wiedzy budżet wyczerpuje się szybciej niż przy jednorazowej publikacji tabeli? Jak należałoby zmienić mechanizm, gdyby jedna osoba mogła mieć w grafie kilka trójek `u:urlop` (np. po jednej na każdy rok)?

## Zadanie B: Atak różnicowy i atak rekonstrukcji

1. W grafie z zadania A (zbudujcie go, jeśli robicie tylko B) zadajcie dwa zapytania: liczba osób z urlopem na zarządzaniu oraz ta sama liczba z warunkiem `FILTER(?s != ?cel)` dla osoby s0076. Co wynika z różnicy?
2. Dodajcie do obu odpowiedzi szum Laplace'a z epsilon w [0,1, 0,5, 1, 5] na zapytanie i oszacujcie trafność ataku (2000 powtórzeń).
3. Atak rekonstrukcji w duchu wyniku Dinura i Nissima: weźcie pierwsze 200 osób i kolumnę `urlop`. Wylosujcie 400 podzbiorów (każda osoba trafia do podzbioru z prawdopodobieństwem 1/2) i dla każdego zwróćcie sumę z szumem. Odtwórzcie kolumnę metodą najmniejszych kwadratów i zaokrągleniem do 0 lub 1.
4. Zmierzcie odsetek poprawnie odtworzonych wartości dla szumu jednostajnego o amplitudzie 0, 1, 2, 5, 10, 20 i 40 oraz dla szumu Laplace'a przy łącznym epsilon 1, 10 i 100 rozłożonym na 400 zapytań. Porównajcie z trafnością zgadywania "nikt nie ma urlopu".

Przy jakiej amplitudzie szumu rekonstrukcja przestaje działać i jak to się ma do pierwiastka z liczby osób? Dlaczego polityka "odpowiadamy tylko na agregaty po co najmniej 100 osobach" nie chroni przed atakiem z punktu 1?

## Zadanie C: Pseudonimizacja a anonimizacja

1. Nadajcie osobom z populacji losowe, różne sześciocyfrowe numery albumu (100000 do 999999), rok urodzenia (1998 do 2004) i kod pocztowy z 30 wartości (`np.random.default_rng(RANDOM_STATE)`). Zbudujcie publikację, w której numer albumu zastąpiono skrótem SHA-256, a obok są płeć, rok, kod, kierunek i urlop.
2. Zbudujcie słownik skrótów wszystkich możliwych numerów albumu, zmierzcie czas i odsetek odwróconych pseudonimów.
3. Zastąpcie SHA-256 przez HMAC-SHA-256 z tajnym kluczem i powtórzcie atak.
4. Policzcie odsetek osób unikalnych i medianę rozmiaru klasy równoważności dla quasi-identyfikatorów (płeć, rok), (płeć, rok, kod) oraz (płeć, rok, kod, kierunek).

Czy publikacja z HMAC zawiera dane osobowe w rozumieniu RODO i kto może ją odwrócić? Przed czym HMAC nie chroni wcale? Jak zmieni się sytuacja, gdy dziekanat co semestr publikuje nową tabelę z tymi samymi pseudonimami?

## Zadanie D: Karta systemu dla systemu decyzyjnego (zalążek projektu)

Karta modelu (model card) albo karta systemu opisuje przeznaczenie systemu, dane, logikę, wyniki ewaluacji z podziałem na grupy, znane ograniczenia i sposób nadzoru. Na zajęciach wszystkie te informacje policzyliście, ale są rozproszone po notebooku.

1. Napiszcie słownik `KARTA` z polami: nazwa i wersja, przeznaczenie, zastosowania poza zakresem, dane wejściowe, logika (generowana z listy `REGULY`), ewaluacja (liczba przyznań, precyzja, czułość), audyt uczciwości (stopy przyznań, disparate impact, TPR w grupach, znane zmienne zastępcze), znane ograniczenia, nadzór człowieka, kwalifikacja prawna z odwołaniem do zadania 6. Pola liczbowe liczcie w kodzie, nie wpisujcie ich ręcznie.
2. Wyświetlcie kartę jako tabelę Markdown (`IPython.display.Markdown`).
3. Zmieńcie regulamin na jeden z wariantów z zadania 5 i wygenerujcie kartę ponownie. Które pola zmieniły się automatycznie, a które trzeba przepisać ręcznie?
4. Przygotujcie analogiczną kartę dla systemu ekspertowego z laboratorium 3 (diagnostyka sieci). Których pól nie da się tam wypełnić i dlaczego?

Które pola karty są najtrudniejsze do wypełnienia i kto w organizacji powinien za nie odpowiadać? Jak włączyć generowanie karty do procesu zmian w bazie reguł, tak by karta nie rozjeżdżała się z kodem? W projekcie zespołowym karta może objąć system zbudowany przez zespół i zawierać audyt uczciwości oraz ocenę ryzyka według AI Act.

## Zadanie E: Twierdzenie o niemożności na zbiorze Adult

1. Wczytajcie Adult, podzielcie 70/30 ze stratyfikacją i nauczcie regresję logistyczną na wszystkich cechach poza `sex` i `fnlwgt` (kodowanie one-hot i standaryzacja w pipeline).
2. Dla progu 0,5 policzcie w grupach kobiet i mężczyzn: częstość klasy pozytywnej p, PPV, FPR i FNR. Sprawdźcie tożsamość Chouldechovej `FPR = p / (1 - p) · (1 - PPV) / PPV · (1 - FNR)`.
3. Sprawdźcie kalibrację w grupach: odsetek pozytywnych w przedziałach przewidywanego prawdopodobieństwa (0 do 0,1, 0,1 do 0,3 itd.).
4. Dobierzcie osobny próg dla kobiet (siatka co 0,005) tak, żeby FPR był równy FPR mężczyzn. Jak zmieniły się PPV i FNR?

Model nie widzi płci, a mimo to odsetek decyzji pozytywnych bardzo różni się między grupami. Czy to jest dyskryminacja, czy odzwierciedlenie różnych częstości w danych? Którą definicję uczciwości wybralibyście dla systemu wspomagającego przyznawanie kredytów, a którą dla systemu kierującego na dodatkowe badania lekarskie? Czy osobny próg dla grupy jest dopuszczalny, skoro wymaga użycia atrybutu chronionego?

## Do oddania

Notebook z rozwiązaniami wybranych zadań (zapisane wyniki, tabele i odpowiedzi na pytania).
