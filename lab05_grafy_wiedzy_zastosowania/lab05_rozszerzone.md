# Laboratorium 5: zadania dodatkowe

Pięć zadań dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne i można wykonać dowolne z nich, w dowolnej kolejności. Wszystkie korzystają z grafu laureatów z zadania 1 notebooka głównego, a zadania B i E także z dat i miejsc śmierci. Zadanie C używa własnych, syntetycznych danych. Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie. Zadanie D wymaga modelu `Qwen/Qwen2.5-0.5B-Instruct`, a samo generowanie zajmuje lokalnie na CPU około 100 s.

## Wspólne przygotowanie

Skopiujcie z notebooka głównego komórki instalacyjne, importy, funkcje `wikidata_csv` i `with_fallback`, zapytanie `Q_LAUREACI`, dane zapasowe oraz zapytania `Q_UCZELNIE` i `Q_JEDNOSTKI`. Budowę grafu zamknijcie w funkcji, żeby dało się ją wywołać ponownie na innych danych:

```python
def build_graph(laureaci, uczelnie, jednostki):
    g = Graph()
    for prefix, ns in [("wd", WD), ("wdt", WDT), ("p", P), ("ps", PS), ("pq", PQ), ("ex", EX)]:
        g.bind(prefix, ns)
    ...                      # etykiety, wdt:P19, wdt:P166, wdt:P69, wdt:P131, p:P166 / ps:P166 / pq:P585
    return g

g = build_graph(laureaci, uczelnie, jednostki)
LABEL = {s: str(o) for s, o in g.subject_objects(RDFS.label)}
lab = lambda node: LABEL.get(node, g.qname(node))
```

Do zadań B i E dopiszcie jeszcze jedno zapytanie: daty i miejsca śmierci (`wdt:P570`, `wdt:P20`) znalezionych osób oraz tytuły ich haseł w polskiej Wikipedii (`schema:about`, `schema:isPartOf <https://pl.wikipedia.org/>`, `schema:name`). Wynik zapiszcie w ramce `smierc` i, jak w notebooku głównym, przygotujcie dane zapasowe.

## Zadanie A: Osadzenia grafu (TransE) i przewidywanie brakujących krawędzi

TransE (A. Bordes i in., Translating Embeddings for Modeling Multi-relational Data, NIPS 2013, https://papers.nips.cc/paper/2013/hash/1cecc7a77928ca8133fa24680a88d2f9-Abstract.html) reprezentuje każdą encję i każdą relację wektorem i uczy się ich tak, żeby dla prawdziwej trójki (h, r, t) zachodziło w przybliżeniu h + r ≈ t. Wiarygodność trójki to ujemna odległość ||h + r - t||. Model pozwala uzupełniać graf: dla pary (h, r) ranking wszystkich encji t podpowiada brakującą krawędź.

1. Weźcie krawędzie między encjami dla relacji `wdt:P19`, `wdt:P166`, `wdt:P69` i `wdt:P131`, ponumerujcie encje i relacje. Wydzielcie zbiór testowy: około 15% krawędzi, ale tylko takich, których oba końce występują jeszcze w innych krawędziach. Dlaczego ten warunek jest potrzebny?
2. Zaimplementujcie TransE w PyTorch (`torch.nn.Embedding` dla encji i relacji, wymiar 32, wektory encji normalizowane do długości 1). Funkcja straty to strata z marginesem: `max(0, margines + d(h, r, t) - d(h', r, t'))`, gdzie (h', r, t') powstaje przez podmianę głowy albo ogona na losową encję. Wystarczy 400 epok pełnym zbiorem, 8 przykładów negatywnych na krawędź, Adam z `lr=0.01`; trening trwa kilka sekund.
3. Oceńcie przewidywanie ogona w wersji filtrowanej: pozycja prawdziwego t w rankingu, z pominięciem innych encji, które z h i r tworzą znaną trójkę, oraz samego h. Podajcie MRR, Hits@1 i Hits@10, ogółem i dla każdej relacji.
4. Porównajcie TransE z modelem popularności: dla relacji r ranking encji według liczby krawędzi r, które do nich prowadzą w zbiorze treningowym.
5. Pokażcie pięciu najlepszych kandydatów TransE dla trzech krawędzi testowych.

Czy TransE wygrywa z popularnością? Dla której relacji i dlaczego? Jak na wynik wpływa to, że `wdt:P131` jest relacją wiele do jednego, a średni stopień w grafie wynosi niecałe 3?

## Zadanie B: Wykrywanie niespójności w danych Wikidata regułami

1. Dodajcie do kopii grafu daty i miejsca śmierci (`wdt:P570`, `wdt:P20`).
2. Zapiszcie reguły jakości jako zapytania SPARQL, z których każde zwraca osobę i opis naruszenia:
   - więcej niż jedno miejsce urodzenia, więcej niż jedno miejsce śmierci (`GROUP BY` z `HAVING (COUNT(DISTINCT ?m) > 1)`);
   - nagroda przyznana przed 25. albo po 90. roku życia;
   - nagroda przyznana po śmierci;
   - data śmierci wcześniejsza niż data urodzenia;
   - węzeł bez polskiej etykiety;
   - miejscowość leżąca (`wdt:P131`) w jednostce o tej samej etykiecie.
3. Sprawdźcie w `networkx`, czy hierarchia `wdt:P131` ma cykle, i wypiszcie jej korzenie.
4. Spróbujcie tego samego z ontologią: oznaczcie `wdt:P19` jako `owl:FunctionalProperty` i uruchomcie `owlrl`. Co wywnioskował reasoner dla osób z dwoma miejscami urodzenia? Następnie dodajcie `owl:differentFrom` między wszystkimi parami różnych miejsc urodzenia i powtórzcie wnioskowanie. Naruszenia `owlrl` zapisuje jako obiekty predykatu `http://www.daml.org/2002/03/agents/agent-ont#error`.
5. Dla każdego zgłoszenia oceńcie, czy to błąd w danych, różnica szczegółowości (ulica i miasto), rzeczywista rozbieżność źródeł, czy fałszywy alarm reguły.

Dlaczego reguły w SPARQL nadają się do kontroli jakości lepiej niż aksjomaty OWL? Co ma z tym wspólnego brak założenia o unikalnych nazwach? Jak zapisalibyście te same reguły w SHACL?

## Zadanie C: Rekomendacje na grafie wiedzy

To zadanie może być zalążkiem projektu zaliczeniowego. Zbudujcie mały graf wiedzy o programie studiów: kilkanaście kursów, każdy z listą tematów i wymaganiami wstępnymi (inne kursy). Dane są syntetyczne, wymyślone tylko do ćwiczenia, i tak je opiszcie.

1. Zdefiniujcie 3 lub 4 ścieżki zainteresowań (np. sztuczna inteligencja, dane, systemy) i wygenerujcie kilkadziesiąt studentów z ustalonym ziarnem losowania. Student na swojej ścieżce wybiera kolejne kursy, których wymagania już spełnił, częściej te o pasujących tematach.
2. Zbudujcie graf `networkx`: student, ukończył, kurs; kurs, temat, temat; kurs, wymaga, kurs.
3. Zaimplementujcie trzy metody rekomendacji, każda ograniczona do kursów, których wymagania student spełnił:
   - popularność (stopień kursu w grafie);
   - profil tematów (suma wystąpień tematów kursu w kursach ukończonych);
   - personalized PageRank z restartem w węźle studenta (`nx.pagerank(..., personalization=...)`).
4. Oceńcie metody w schemacie leave-one-out: dla każdego studenta ukryjcie ostatni ukończony kurs, zbudujcie graf bez tej krawędzi i sprawdźcie pozycję ukrytego kursu. Podajcie Hits@1, Hits@3 i MRR oraz średnią liczbę kandydatów.
5. Dla jednego studenta pokażcie trzy rekomendacje z uzasadnieniem w postaci najkrótszej ścieżki w grafie.

Ile pracy wykonuje samo ograniczenie do dostępnych kursów? Czy różnice między metodami są duże w porównaniu z tym, ile waży jeden student? Na ile wynik zależy od sposobu, w jaki powstały dane?

## Zadanie D: Model językowy generuje zapytania SPARQL, graf je weryfikuje

Zamiast odpowiadać na pytanie bezpośrednio, model językowy może napisać zapytanie, które wykonamy na grafie. Odpowiedź pochodzi wtedy z danych, a błąd modelu objawia się jako błąd składni, pusty wynik albo zły wynik, a nie jako płynnie brzmiące zmyślenie.

1. Przygotujcie graf po wnioskowaniu `owlrl` z TBoxem z zadania 3 (przechodnie `wdt:P131`, `ex:urodzonyNaTerenie`).
2. Napiszcie zwięzły opis schematu dla modelu: prefiksy, znaczenie predykatów, sposób zapisu roku nagrody (`p:P166`, `ps:P166`, `pq:P585`), etykiety po polsku w `rdfs:label`. Dodajcie dwa przykłady pytań z zapytaniami, różne od pytań testowych, podane jako wcześniejsze tury rozmowy (użytkownik, asystent). Bez przykładów mały model często powtarza pytanie zamiast pisać zapytanie.
3. Dla każdego pytania z `QUESTIONS` wygenerujcie zapytanie (zachłannie, do 120 tokenów), wyciągnijcie je z odpowiedzi i sprawdźcie kolejno: czy jest zapytanie, czy składnia jest poprawna (`prepareQuery` z jawnym słownikiem prefiksów `initNs`), czy wykonuje się bez błędu, czy wynik jest niepusty, czy jest trafny według `score`.
4. Pokażcie tabelę odsetków dla tych etapów, jedno trafne zapytanie i wszystkie nietrafne.
5. Porównajcie trafność z szablonami z zadania 3 i z modelem z kontekstem z zadania 5.

Które typy pytań model obsługuje, a które nie, i jak to się ma do przykładów, które dostał? Który rodzaj błędu jest najgroźniejszy: błąd składni, pusty wynik czy niepusty, ale zły wynik? Co dałoby się sprawdzić automatycznie przed wykonaniem zapytania?

## Zadanie E: Graf z tekstu a Wikidata: pokrycie, sprzeczności, proweniencja

1. Pobierzcie przez API MediaWiki pierwsze akapity haseł wszystkich laureatów (`action=query`, `prop=extracts`, `exintro=1`, `explaintext=1`; najwyżej 20 tytułów w jednym zapytaniu, nagłówek User-Agent, przerwa między zapytaniami, obsługa odpowiedzi 429 z nagłówkiem `Retry-After`). Przygotujcie dane zapasowe i odnotujcie licencję CC BY-SA 4.0.
2. Wyrażeniami regularnymi wyciągnijcie z tekstu miejsce i rok urodzenia oraz śmierci.
3. Porównajcie fakty z tekstu z faktami z Wikidata (miejsca urodzenia i śmierci, lata z `wdt:P569` i `wdt:P570`). Nazwy miejsc w tekście są odmienione, więc porównanie musi tolerować końcówki fleksyjne, np. przez wspólny początek wyrazu. Policzcie pokrycie w obu kierunkach oraz liczbę faktów zgodnych i rozbieżnych.
4. Każdą rozbieżność przypiszcie do kategorii: nazwa historyczna, brak etykiety, różnica szczegółowości, rzeczywista sprzeczność do sprawdzenia w źródłach.
5. Zapiszcie oba źródła w jednym `rdflib.Dataset`, każde w osobnym grafie nazwanym (`ex:wikidata`, `ex:wikipedia`), i napiszcie zapytanie z klauzulą `GRAPH`, które znajduje osoby z różnymi latami śmierci w obu źródłach.

Które rozbieżności wynikają z danych, a które z metody porównania? Po co zapisywać pochodzenie faktu i jak pomaga w tym graf nazwany? Któremu źródłu ufalibyście w przypadku sprzeczności i dlaczego żadnemu bezwarunkowo?

## Do oddania

Notebook z rozwiązaniem wybranych zadań: z zapisanymi wynikami komórek, tabelami i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca i sprawdźcie, że nie ma błędów.
