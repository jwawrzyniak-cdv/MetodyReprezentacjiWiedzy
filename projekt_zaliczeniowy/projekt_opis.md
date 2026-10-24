# Projekt zespołowy: system oparty na wiedzy

## Założenia

Projekt jest jedyną podstawą zaliczenia kursu "Metody reprezentacji wiedzy". Zgodnie z sylabusem modułu "Istota sztucznej inteligencji" ocena z kursu stanowi 50% oceny z modułu, a temat projektu ustala się z prowadzącym.

Zadanie polega na zaprojektowaniu i zbudowaniu małego systemu, który przechowuje wiedzę o wybranej dziedzinie w jawnej, symbolicznej postaci, wyciąga z niej wnioski i odpowiada na pytania użytkownika. Na laboratoriach poszczególne formalizmy ćwiczyliście osobno: ontologie i SPARQL, ramy, reguły, sieci bayesowskie, grafy wiedzy, prywatność i uczciwość. W projekcie łączycie co najmniej dwa z nich w jeden działający system i sprawdzacie, jak dobrze odpowiada on na pytania, dla których powstał.

Zespoły liczą od 3 do 4 osób. Skład zgłaszacie razem z tematem. Każda osoba odpowiada za wyraźnie opisaną część pracy, ale na obronie musi umieć wyjaśnić całość, w tym fragmenty napisane przez innych.

Środowisko pracy to Google Colab. Wynikiem jest notebook (albo kilka notebooków), który przechodzi od początku do końca po wybraniu "Uruchom wszystko", bez ręcznego wgrywania plików. Ontologię i dane można trzymać w publicznym repozytorium i pobierać w kodzie albo wbudować w notebook.

## Harmonogram

Terminy podaje prowadzący na pierwszych zajęciach. Projekt ma trzy punkty kontrolne.

| Etap | Co oddajecie | Moment |
|---|---|---|
| 1. Temat | Skład zespołu, dziedzina, kilkanaście pytań kompetencyjnych, źródła danych z licencjami, wstępny podział ról (jedna do dwóch stron) | po laboratorium 2 |
| 2. Stan pośredni | Notebook z ontologią lub grafem wiedzy, danymi, zapytaniami SPARQL odpowiadającymi na część pytań kompetencyjnych i pierwszym mechanizmem wnioskowania | po laboratorium 4 |
| 3. Wersja końcowa | Notebook, prezentacja zespołu (12 do 15 minut, plus pytania) i odpowiedzi na pytania o kod | ostatnie zajęcia |

Etapy 1 i 2 nie dostają osobnej oceny, ale ich brak obniża ocenę końcową (patrz kryteria). Ich celem jest wczesne wykrycie problemów: dziedzina zbyt szeroka, brak danych, pytania, na które nie da się odpowiedzieć z przyjętej reprezentacji.

## Pytania kompetencyjne

Pracę zaczynacie od listy pytań, na które system ma umieć odpowiedzieć (competency questions). Są one jednocześnie specyfikacją i testem akceptacyjnym. Dobre pytanie jest konkretne i sprawdzalne, na przykład "Które dzieła epoki romantyzmu doczekały się ekranizacji po 1990 roku?", a nie "Co system wie o literaturze?". Część pytań powinna wymagać wnioskowania, czyli odpowiedzi, której nie ma zapisanej wprost w danych.

## Wymagania

### Na ocenę 3,0

1. Dziedzina uzgodniona z prowadzącym i co najmniej 12 pytań kompetencyjnych, z których co najmniej 4 wymagają wnioskowania.
2. Ontologia w OWL albo graf wiedzy w RDF ze schematem RDFS: co najmniej 15 klas, 10 właściwości (obiektowych i danych łącznie) i 100 instancji. Klasy mają etykiety (`rdfs:label`) i krótkie opisy (`rdfs:comment`).
3. Dane z publicznego źródła (na przykład Wikidata, OpenML, dane.gov.pl) albo opracowane samodzielnie z opisem pochodzenia. Licencja danych podana w notebooku.
4. Zapytania SPARQL odpowiadające na pytania kompetencyjne, każde opatrzone pytaniem w języku naturalnym i wynikiem.
5. Jeden mechanizm wnioskowania użyty świadomie: reasoner OWL (HermiT albo profil OWL 2 RL przez owlrl) lub system regułowy. Pokazane fakty, które pojawiły się dopiero dzięki wnioskowaniu.
6. Tabela pokrycia: które pytania kompetencyjne system obsługuje, a które nie i dlaczego.
7. Wnioski i ograniczenia przyjętej reprezentacji.

### Na ocenę 4,0 (dodatkowo)

8. Drugi formalizm reprezentacji, połączony z pierwszym, a nie postawiony obok niego: system regułowy nad ontologią, ramy z wartościami domyślnymi, sieć bayesowska zasilana faktami z grafu albo skrypty opisujące typowe przebiegi zdarzeń w dziedzinie. Uzasadnienie, dlaczego ta część wiedzy wymaga innego formalizmu.
9. Integracja z zewnętrznym źródłem wiedzy (na przykład łączenie encji z Wikidata przez `owl:sameAs` albo identyfikatory), z oceną, ile encji udało się połączyć i ile powiązań było błędnych.
10. Objaśnienia: dla co najmniej trzech odpowiedzi system pokazuje, skąd je wziął (ścieżka w grafie, łańcuch reguł, wpływ obserwacji na prawdopodobieństwo).
11. Testy automatyczne: każde pytanie kompetencyjne ma oczekiwaną odpowiedź zapisaną w kodzie, a jedna komórka sprawdza wszystkie naraz.

### Na ocenę 5,0 (dodatkowo)

12. Aplikacja wiedzy: odpowiadanie na pytania zadawane po polsku (szablony, wyszukiwanie semantyczne albo model językowy z kontekstem z grafu), rekomendacja albo wspomaganie decyzji, z oceną jakości na przygotowanym zestawie pytań.
13. Porównanie z rozwiązaniem bez jawnej reprezentacji wiedzy (na przykład ten sam zestaw pytań zadany samemu modelowi językowemu albo prosty model uczony z danych) i omówienie, w czym każde podejście jest lepsze.
14. Analiza prywatności i uczciwości systemu: jakie dane osobowe lub wrażliwe przetwarza albo pozwala wywnioskować, kto powinien mieć dostęp do jakiej części grafu, czy decyzje systemu mogą różnicować grupy osób. Przynajmniej jeden problem sprawdzony liczbami lub zapytaniem, nie tylko opisany.
15. Analiza jakości ontologii: spójność sprawdzona reasonerem, wykryte i poprawione błędy modelowania, uwagi o założeniu świata otwartego w kontekście waszych pytań.
16. Jednostronicowa notatka dla odbiorcy nietechnicznego (po polsku, bez żargonu): co system robi, czego nie umie, komu i do czego może służyć.

Ocena celująca (5,5) wymaga według sylabusa pisemnego uzasadnienia prowadzącego. Może ją dostać projekt, który wyraźnie wykracza poza wymagania na 5,0, na przykład własnym algorytmem wnioskowania, nietrywialnym porównaniem kilku formalizmów na tych samych pytaniach albo wdrożeniem z prawdziwymi użytkownikami.

## Propozycje tematów

Każdy temat trzeba zawęzić do dziedziny, którą zespół zdoła opisać w kilka tygodni. Źródła w opisach są punktem wyjścia, ich dostępność i licencję sprawdzacie w etapie 1.

### Diagnoza awarii sprzętu lub usług IT

Ontologia komponentów i zależności między nimi, system regułowy z objaśnieniami ("dlaczego pytasz", "jak do tego doszedłeś"), sieć bayesowska do diagnozy przy niepewnych objawach. Dobrze nadaje się do porównania reguł z rachunkiem prawdopodobieństwa na tych samych przypadkach.

### Graf wiedzy o polskiej kulturze

Pisarze, utwory, epoki, nagrody, ekranizacje z Wikidata. Odpowiadanie na pytania po polsku, wnioskowanie o epokach i powiązaniach, wykrywanie braków i niespójności w danych źródłowych.

### Przepisy jako reguły

Wąski fragment przepisów, na przykład regulamin studiów w części o warunkach zaliczenia semestru, wybrana ulga podatkowa albo prawa pasażera. Reguły z odwołaniem do konkretnych paragrafów, wyjątki i wartości domyślne, objaśnienia decyzji, przypadki graniczne. Tekst przepisów pobieracie ze źródła urzędowego i cytujecie z datą wersji.

### Zabytki i trasy zwiedzania

Obiekty zabytkowe opisane w Wikidata, ich typy, style architektoniczne, położenie. Reguły układania tras (czas, dostępność, odległości), rekomendacje, skrypt zwiedzania jako typowy przebieg wizyty.

### Ryzyko kredytowe z audytem uczciwości

Zbiór German Credit z OpenML (identyfikator 31). Ontologia pojęć kredytowych, jawne reguły decyzyjne (jak w karcie scoringowej), sieć bayesowska uczona z danych, audyt uczciwości reguł i sieci względem wieku i płci, objaśnienia decyzji odmownych.

### Żywienie i alergeny

Ontologia produktów, składników i alergenów, reguły zgodności z dietą (wegetariańska, bezglutenowa, ograniczenia alergiczne), rekomendacje posiłków z objaśnieniem. Dane o składnikach można opracować samodzielnie albo pozyskać z otwartej bazy, jeśli licencja na to pozwala.

### Temat własny

Dziedzina z waszej pracy zawodowej lub zainteresowań. Do etapu 1 dołączcie pytania kompetencyjne i opis źródeł danych. Danych firmowych i osobowych nie wolno używać bez zgody i anonimizacji, a do notebooka oddawanego na uczelnię nie trafiają w ogóle.

## Struktura notebooka

Minimalny układ (gotowy szkielet jest w `projekt_szablon.ipynb`):

1. Zespół, dziedzina, cel systemu
2. Pytania kompetencyjne
3. Źródła danych i licencje
4. Ontologia: klasy, właściwości, aksjomaty, uzasadnienie decyzji modelowych
5. Dane i budowa grafu wiedzy
6. Zapytania SPARQL
7. Wnioskowanie
8. Drugi formalizm i jego połączenie z grafem
9. Aplikacja: odpowiadanie na pytania, rekomendacja lub decyzja
10. Ocena: testy pytań kompetencyjnych, pokrycie, porównanie z podejściem bez jawnej wiedzy
11. Prywatność, przejrzystość, uczciwość
12. Wnioski, ograniczenia, dalsze kroki
13. Podział pracy w zespole

Każda sekcja zawiera komentarz opisowy. Sam kod i wyniki bez słowa wyjaśnienia nie wystarczą.

## Kryteria oceny

| Kryterium | Waga | Co oceniamy |
|---|---|---|
| Analiza dziedziny i pytania kompetencyjne | 10% | Zakres dziedziny, jakość i sprawdzalność pytań, źródła i licencje |
| Ontologia i graf wiedzy | 20% | Poprawność modelowania, spójność, sensowne aksjomaty, nazewnictwo, etykiety |
| Wnioskowanie | 20% | Poprawne użycie reasonera lub reguł, wyprowadzone fakty, świadomość ograniczeń (świat otwarty, monotoniczność) |
| Integracja formalizmów i aplikacja | 15% | Połączenie dwóch reprezentacji, odpowiadanie na pytania lub decyzje |
| Ocena systemu | 15% | Testy, pokrycie pytań, porównanie z innym podejściem, uczciwie opisane porażki |
| Prywatność, przejrzystość, uczciwość | 10% | Konkretna analiza dla waszego systemu, poparta zapytaniem lub liczbami |
| Prezentacja, obrona i praca zespołowa | 10% | Jasność wypowiedzi, odpowiedzi na pytania o kod, czytelny podział pracy |

Każde kryterium dostaje ocenę od 2 do 5, a ocenę końcową wyznacza średnia ważona, przeliczona według skali z sylabusa (od 3,00 do 3,25 dostateczna, od 3,26 do 3,75 dostateczna plus, od 3,76 do 4,25 dobra, od 4,26 do 4,75 dobra plus, od 4,76 do 5,0 bardzo dobra). Wymagania z poprzedniej sekcji wyznaczają górny pułap: projekt, który nie spełnia wymagań na 4,0, nie dostanie więcej niż 3,5, a niespełniający wymagań na 5,0 nie więcej niż 4,5. Brak etapu 1 albo 2 obniża ocenę końcową o pół stopnia za każdy brakujący etap. Prowadzący może zróżnicować oceny w zespole, jeśli obrona pokaże wyraźnie nierówny wkład.

## Format oddania

1. Notebook lub notebooki `.ipynb` z zapisanymi wynikami, wgrane na platformę kursu, z linkiem do kopii w Colabie z uprawnieniem do odczytu.
2. Ontologia w osobnym pliku (Turtle albo RDF/XML), wczytywana przez notebook z publicznego adresu albo wbudowana w kod.
3. Instalacja dodatkowych bibliotek w pierwszej komórce.
4. Prezentacja: do 10 slajdów albo bezpośrednio notebook, 12 do 15 minut dla całego zespołu. Głos zabiera każda osoba.
5. Notatka z podziałem pracy: kto za które sekcje odpowiadał.

## Praktyczne wskazówki

Zacznijcie od pytań kompetencyjnych i małego grafu, który odpowiada na trzy z nich. Ontologia zaprojektowana "na zapas", bez pytań, rośnie szybko i niczemu nie służy.

Zanim dodacie aksjomat, zastanówcie się, jakie wnioski z niego wynikną. Reasoner wyprowadzi wszystko, co logicznie wynika z ontologii, również rzeczy, których się nie spodziewaliście. Sprawdzajcie spójność po każdej większej zmianie.

Zapisujcie, co nie zadziałało. Pytanie, na które system nie umie odpowiedzieć, z wyjaśnieniem, której konstrukcji zabrakło w reprezentacji, jest wartościowym wynikiem.
