# Laboratorium 1: zadania dodatkowe

Pięć zadań alternatywnych dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne: każde ma własne dane i można je wykonać w dowolnej kolejności. Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie. Zadanie A wymaga Javy (HermiT), zadania D i E łączą się z Wikidata. Zadanie A jest pomyślane jako zalążek projektu zespołowego: ontologia, którą w nim zbudujecie, może stać się rdzeniem projektu.

## Wspólne przygotowanie

Wszystkie zadania zaczynają się od instalacji i importów. pyshacl jest potrzebny tylko w zadaniu B.

```python
%pip install -q rdflib owlrl owlready2 pyshacl

import io, shutil, subprocess, time
import networkx as nx
import pandas as pd
import requests
import owlrl
import pyshacl
from rdflib import OWL, RDF, RDFS, SKOS, XSD, Graph, Literal, Namespace, URIRef
import owlready2
from owlready2 import (AllDifferent, ConstrainedDatatype, DataProperty, FunctionalProperty, Imp,
                       ObjectProperty, OwlReadyInconsistentOntologyError, OwlReadyJavaError, Thing,
                       World, sync_reasoner_hermit)

RANDOM_STATE = 42
U = Namespace("http://example.org/uczelnia#")
WD = Namespace("http://www.wikidata.org/entity/")
WDT = Namespace("http://www.wikidata.org/prop/direct/")
```

Nie importujcie z owlready2 przez gwiazdkę, bo nadpisuje nazwy z rdflib. W zadaniach D i E przyda się funkcja `zapytaj_wikidata(zapytanie, zapas)` z notebooka głównego (zadanie 7): wysyła zapytanie z nagłówkiem User-Agent, a przy błędzie zwraca dane zapasowe. Przed uruchomieniem HermiT sprawdźcie Javę tak samo jak w zadaniu 5.

## Zadanie A: Ontologia własnej dziedziny z regułami SWRL

Wybierzcie małą dziedzinę, najlepiej związaną z tematem waszego projektu. W przykładowym rozwiązaniu jest to biblioteka: autorzy, książki z gatunkiem i rokiem wydania, czytelnicy i wypożyczenia.

1. Zbudujcie ontologię bezpośrednio w owlready2 (`World()`, `get_ontology`, klasy jako podklasy `Thing`, właściwości z `domain` i `range`). Potrzebne są co najmniej cztery klasy, trzy właściwości obiektowe, jedna właściwość odwrotna (`inverse_property`), jedna funkcyjna właściwość danych i kilkanaście egzemplarzy.
2. Zdefiniujcie klasę przez równoważność z ograniczeniem liczności, na przykład StalyCzytelnik równoważny `Czytelnik & wypozyczyl.min(3, Ksiazka)`.
3. Dopiszcie dwie reguły SWRL (`Imp().set_as_rule(...)`), które wyprowadzają nową relację z łańcucha dwóch istniejących, na przykład „czytelnik wypożyczył książkę autora, więc czyta tego autora”. Uruchomcie HermiT z `infer_property_values=True` i pokażcie wyprowadzone fakty w tabeli.
4. Sprawdźcie, kto należy do klasy z punktu 2. Jeśli kogoś brakuje, choć w danych ma wystarczająco wiele egzemplarzy, wyjaśnijcie dlaczego i naprawcie to aksjomatem `AllDifferent`.
5. Spróbujcie reguły z funkcją wbudowaną SWRL, na przykład `lessThan(?r, 1980)` dla klasy Klasyka. Co robi HermiT? Zapiszcie to samo wymaganie inaczej: jako klasę z ograniczeniem typu danych (`ConstrainedDatatype(int, max_exclusive=1980)`) albo jako zapytanie SPARQL przez `swiat.sparql(...)`. Porównajcie wyniki.

Które z waszych reguł dałoby się zapisać w samym OWL 2 jako łańcuch właściwości (property chain), a które nie? Dlaczego bez `AllDifferent` reasoner nie liczy książek tak, jak liczy człowiek? Jakie pytania projektowe ta ontologia mogłaby obsłużyć, a do czego trzeba by ją rozszerzyć?

## Zadanie B: Walidacja SHACL a wnioskowanie OWL

SHACL (Shapes Constraint Language) opisuje wymagania wobec danych: liczności, typy literałów, zakresy, klasy wartości. W odróżnieniu od OWL sprawdza dane w świecie zamkniętym.

1. Wczytajcie dane uczelni i schemat RDFS z notebooka głównego. Dodajcie dwa celowo błędne zapisy: studenta u:s14 z `u:rokStudiow "drugi"` (napis zamiast liczby) i pracę dyplomową u:pd04, której promotorem jest studentka u:s12.
2. Napiszcie kształty SHACL: student ma dokładnie jeden rok studiów, liczbę całkowitą od 1 do 5, i nazwisko; kurs ma ECTS i dokładnie jeden wydział; praca dyplomowa ma jednego promotora z klasy Pracownik i co najmniej jednego autora z klasy Student.
3. Uruchomcie `pyshacl.validate` w trzech wariantach: na samych danych, z `ont_graph=schemat` i z `ont_graph=schemat, inference="rdfs"`. Zestawcie naruszenia w jednej tabeli (węzeł, ścieżka, rodzaj ograniczenia, wartość, wariant).
4. Zapiszcie wymaganie „każdy student ma rok studiów będący liczbą całkowitą” w OWL (ograniczenie `some` z typem `xsd:integer`) i sprawdźcie HermiT-em dane bez s14 oraz z s14.

Dlaczego liczba naruszeń jest we wszystkich wariantach taka sama, a ich lista nie? Który wariant znalazł brak roku studiów u u:s10 i dlaczego tylko ten? Co stało się z błędem pd04 po wnioskowaniu RDFS? Dlaczego HermiT nie uważa braku roku studiów za błąd, a literał „drugi” już tak?

## Zadanie C: Jakość ontologii i typowe błędy modelowania

Katalog OOPS! (OntOlogy Pitfall Scanner) zbiera typowe błędy w ontologiach, oznaczone numerami P01, P02 i dalej. W tym zadaniu wykrywacie kilka z nich zapytaniami SPARQL na małej, celowo wadliwej ontologii pojazdów.

1. Zapiszcie w Turtle ontologię pojazdów z podłożonymi błędami: cykl w hierarchii (SrodekTransportu, Pojazd, Samochod i z powrotem), część zapisaną jako podklasa (Kolo subClassOf Samochod), klasy niepołączone z resztą (Silnik, Rower), klasy i właściwości bez `rdfs:label`, właściwości bez dziedziny lub zakresu, klasę o nazwie łączącej dwa pojęcia (SamochodLubMotocykl) i egzemplarze użyte bez typu.
2. Dla każdego rodzaju błędu napiszcie osobne zapytanie SPARQL: P04 (elementy niepołączone), P06 (cykle w hierarchii klas, przez `rdfs:subClassOf+`), P08 (brak etykiet), P11 (brak dziedziny lub zakresu), nazwy z „Lub” albo „I” oraz obiekty bez deklaracji typu. Zbierzcie wyniki w jednej tabeli z liczbą i listą elementów.
3. Policzcie domknięcie OWL 2 RL i sprawdźcie skutki: jakie klasy są teraz równoważne klasie Samochod, jakie typy ma kolo1, jakie obiekty są pojazdami, czy jan jest właścicielem fiata.

Które wskazania to prawdziwe błędy, a które fałszywe alarmy heurystyki? Dlaczego cykl w hierarchii nie jest dla reasonera błędem? Który z błędów ma najpoważniejsze skutki dla wniosków i dlaczego reasoner go nie zgłosi?

## Zadanie D: Łączenie grafu lokalnego z Wikidata przez owl:sameAs

1. Zbudujcie lokalny graf z trzema uczelniami (u:UJ, u:PW, u:UWr) bez dat i adresów, z typem i etykietą.
2. Pobierzcie z Wikidata dla Q189441, Q1718656 i Q315658 nazwę, miasto (P131), datę założenia (P571) i adres strony (P856). Jedno zapytanie z `VALUES` wystarczy. Zapiszcie wyniki jako trójki rdflib, a daty jako `xsd:date`. Przygotujcie dane zapasowe na wypadek awarii.
3. Połączcie encje przez `owl:sameAs` i policzcie domknięcie OWL 2 RL. Zbudujcie tabelę dla trzech lokalnych uczelni: daty założenia, miasta, liczba adresów www, liczba polskich nazw.
4. Dodajcie jedno błędne powiązanie: u:UWr sameAs encja Politechniki Warszawskiej. Co widać w tabeli dla PW i UWr? Następnie dopiszcie `owl:differentFrom` między lokalnymi uczelniami i policzcie komunikaty błędu owlrl.
5. Zamiast `owl:sameAs` użyjcie `skos:exactMatch` (z błędnym powiązaniem). Co zostało z danych w tabeli?

Dlaczego jeden błędny link psuje dwie uczelnie, a nie jedną? Dlaczego dwa adresy www Uniwersytetu Wrocławskiego nie są błędem? Kiedy w praktyce łączenia danych wybralibyście `owl:sameAs`, a kiedy `skos:exactMatch`?

## Zadanie E: Analiza hierarchii klas Wikidata

1. Napiszcie funkcję, która dla zadanego identyfikatora pobiera wszystkie krawędzie P279 osiągalne w górę (`wdt:P279*`, potem `wdt:P279`) z etykietami po polsku lub angielsku i buduje z nich `nx.DiGraph`. Pobierzcie hierarchię uniwersytetu (Q3918) i studenta (Q48282). Między zapytaniami zróbcie kilkusekundową przerwę.
2. Dla obu hierarchii policzcie: klasy, krawędzie, czy graf jest acykliczny, liczbę cykli (`nx.simple_cycles`), klasy z więcej niż jedną nadklasą, korzenie, najdłuższą ścieżkę (w grafie po kondensacji `nx.condensation`, wtedy jest określona także przy cyklu) oraz krawędzie nadmiarowe, czyli takie, które wynikają z innej ścieżki P279 (porównajcie graf z jego redukcją przechodnią).
3. Wypiszcie znalezione cykle z etykietami i kilka przykładów krawędzi nadmiarowych.
4. Znajdźcie wspólne nadklasy uniwersytetu i studenta oraz najniższe z nich. Wypiszcie bezpośrednie nadklasy studenta i ścieżkę od studenta do klasy organizm, jeśli istnieje.

Co oznaczałby znaleziony cykl, gdyby hierarchię przetłumaczyć na `rdfs:subClassOf` i uruchomić reasoner? Czy krawędzie nadmiarowe to błąd? Czy najniższe wspólne nadklasy studenta i uniwersytetu mają sens, i co to mówi o modelowaniu w Wikidata? Pamiętajcie, że Wikidata się zmienia, więc cykl może zostać usunięty, a liczby będą inne niż u kolegów.

## Do oddania

Notebook z rozwiązaniem wybranego zadania (lub zadań), wykonany od początku do końca, z zapisanymi wynikami i krótkimi wnioskami na końcu każdej sekcji. W zadaniach D i E zaznaczcie, czy wyniki pochodzą z Wikidata, czy z danych zapasowych, i podajcie datę wykonania.

## Literatura i dokumentacja

- Shapes Constraint Language (SHACL): https://www.w3.org/TR/shacl/
- SWRL, propozycja W3C: https://www.w3.org/submission/SWRL/
- owlready2, reguły SWRL: https://owlready2.readthedocs.io/en/latest/rule.html
- owlready2, wnioskowanie: https://owlready2.readthedocs.io/en/latest/reasoning.html
- pySHACL: https://github.com/RDFLib/pySHACL
- Katalog błędów OOPS!: https://oops.linkeddata.es/catalogue.jsp
- M. Poveda-Villalón, A. Gómez-Pérez, M. C. Suárez-Figueroa, OOPS! (OntOlogy Pitfall Scanner!): An On-line Tool for Ontology Evaluation, International Journal on Semantic Web and Information Systems, 2014.
- H. Halpin, P. J. Hayes, J. P. McCusker, D. L. McGuinness, H. S. Thompson, When owl:sameAs Isn't the Same: An Analysis of Identity in Linked Data, ISWC 2010.
- Rangi deklaracji w Wikidata: https://www.wikidata.org/wiki/Help:Ranking
