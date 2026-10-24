# Laboratorium 1: Sieci semantyczne i ontologie

Dane: ontologia uczelni w Turtle (gotowa w szablonie, przestrzeń nazw `http://example.org/uczelnia#`), ontologia uczelni budowana w WebProtégé, publiczny punkt SPARQL Wikidata (`https://query.wikidata.org/sparql`)
Środowisko: Google Colab (CPU), biblioteki rdflib, owlrl, owlready2, networkx; reasoner HermiT wymaga Javy; WebProtégé (`https://webprotege.stanford.edu`) w przeglądarce

## Cel

Laboratorium prowadzi od sieci semantycznej z dziedziczeniem do standardów Semantic Web. Najpierw zobaczycie, jak sieć is-a oszczędza zapis i gdzie zawodzi przy wyjątkach i wielodziedziczeniu. Potem ta sama wiedza o uczelni trafia do grafu RDF, który odpytujecie w SPARQL i rozszerzacie wnioskowaniem w RDFS, OWL 2 RL i pełnym OWL 2 DL. Na każdym etapie sprawdzacie nie tylko, co formalizm wywnioskuje, ale też czego nie wywnioskuje i dlaczego.

W drugiej części budujecie fragment tej samej ontologii w WebProtégé, edytorze ontologii OWL działającym w przeglądarce i przeznaczonym do pracy zespołowej. Eksport z WebProtégé wczytujecie potem w notebooku i sprawdzacie reasonerem, bo sam edytor wniosków nie wyprowadza. Ostatnie zadanie porównuje własny graf z Wikidata, gdzie wiedza ma kwalifikatory i rangi.

Ontologia uczelni z tego laboratorium wróci w laboratorium 3 (reguły) i 6 (prywatność), a struktura dziedziczenia z zadania 1 w laboratorium 2 (ramy).

## Przed zajęciami

Przeczytajcie `lab01_wprowadzenie.md`. Załóżcie konto w WebProtégé (`https://webprotege.stanford.edu`, rejestracja przez odnośnik do zakładania konta na stronie logowania). Rejestracja wymaga adresu e-mail, dlatego zróbcie to przed zajęciami. Jedna osoba z pary może założyć projekt, a druga dostanie do niego dostęp w scenariuszu W4.

Otwórzcie `lab01_szablon.ipynb` w Google Colab i uruchomcie pierwszą komórkę: instaluje rdflib, owlrl i owlready2, co trwa kilkanaście sekund. Druga komórka zawiera importy i stałą `RANDOM_STATE = 42` (używaną tylko przy rozmieszczaniu węzłów na rysunkach). Dane uczelni, aksjomaty OWL, deklaracje klas, przykładowy eksport z WebProtégé, funkcja pomocnicza do Wikidata i dane zapasowe są w szablonie jako gotowe komórki. Plik `uczelnia_webprotege_przyklad.owl` możecie wgrać do Colaba, ale nie musicie: jego treść jest też w notebooku.

Zadania 5 i 7 potrzebują odpowiednio Javy i internetu. Komórka z początku zadania 5 sprawdza Javę i w razie braku ją instaluje. Jeśli Wikidata nie odpowiada, funkcja `zapytaj_wikidata` po kilku próbach zwraca dane zapasowe wbudowane w notebook, a kolumna `zrodlo` pokazuje, skąd pochodzi wynik.

## Zadanie 1: Sieć semantyczna i dziedziczenie

Sieć w szablonie to graf `nx.DiGraph`. Węzły mają słownik `cechy`, krawędzie atrybut `rel` o wartości `is-a` albo `part-of`. Są w niej klasy (zwierzę, ptak, ryba, kanarek, struś, pingwin, rekin, łosoś), dwa egzemplarze (Tweety is-a kanarek, Pingu is-a pingwin) i część (skrzydło part-of ptak). Rysunek sieci jest gotowy.

1. Napiszcie `znajdz_ceche(siec, wezel, cecha)`, która przeszukuje sieć wszerz w górę, wyłącznie po krawędziach is-a, i zwraca wartości cechy z najbliższych węzłów, w których jest zapisana wprost, razem ze źródłem i odległością. Na jej podstawie napiszcie `odpowiedz(...)`, która zwraca wartość, napis `"konflikt"` (gdy na tej samej odległości są różne wartości) albo `"nie wiadomo"`.
2. Zbudujcie tabelę odpowiedzi dla par: Tweety (oddycha, lata, kolor), Pingu (lata, plywa, liczba_nog), rekin (lata, kolor), skrzydło (lata, oddycha). Skąd pochodzi każda odpowiedź i z jakiej odległości?
3. Policzcie, ile cech zapisano wprost, a na ile par (węzeł, cecha) sieć potrafi odpowiedzieć dzięki dziedziczeniu.
4. Zbudujcie diament Nixona: Nixon is-a kwakier, Nixon is-a republikanin, kwakier ma `pacyfista=True`, republikanin `pacyfista=False`. Co odpowiada sieć? Na kopii głównej sieci dodajcie krawędź nadmiarową Pingu is-a ptak (prawdziwą, ale wynikającą z innych) i zapytajcie, czy Pingu lata.
5. Poprawcie rozstrzyganie konfliktów. Zamiast najkrótszej ścieżki odrzućcie trafienia z węzłów, pod którymi w hierarchii leży inne trafienie (klasa bardziej szczegółowa wygrywa). Pomocna będzie funkcja `jest_podklasa(siec, a, b)`. Sprawdźcie Pingu i Nixona.

Dlaczego skrzydło nie dziedziczy niczego po ptaku? Dlaczego krawędź, która jest prawdziwa, zepsuła odpowiedź dla Pingu, i które konflikty poprawka rozwiązuje, a które zostają? Co by się stało z wnioskiem „Tweety lata”, gdybyśmy dowiedzieli się, że Tweety jest pingwinem?

## Zadanie 2: RDF i Turtle w rdflib

Komórka `UCZELNIA_TTL` zawiera fakty o uczelni: wydziały, kursy z punktami ECTS i prerekwizytami, pracowników, studentów z zapisami i zaliczeniami oraz prace dyplomowe. Komórka `PREFIKSY` zawiera prefiksy do zapytań SPARQL.

1. Wczytajcie `UCZELNIA_TTL` do `rdflib.Graph` i powiążcie prefiks `u:`. Wypiszcie liczbę trójek, liczbę różnych podmiotów, liczbę trójek z literałem w obiekcie oraz tabelę liczby trójek dla każdego predykatu.
2. Ile zasobów ma każdy typ (`rdf:type`)? Wypiszcie osoby (zasoby z `u:imie`), które nie mają żadnego typu. Zapamiętajcie je, wrócą w zadaniu 4.
3. Sprawdźcie, czy w grafie jest trójka `(u:p01, u:imie, Literal("Anna"))` oraz ta sama z `Literal("Anna", lang="pl")`.
4. Dodajcie programowo studentkę u:s13: Zofia Wróbel, rok studiów 1, urodzona 2003, zapisana na u:Programowanie i u:Logika. Liczby zapiszcie jako literały typu `xsd:integer` (domyślne dla `Literal(1)`). Zserializujcie jej trójki do Turtle i N-Triples. Porównajcie długość obu serializacji całego grafu.
5. Uruchomcie gotową komórkę, która buduje `nx.DiGraph` z trójek `u:wymagaKursu` i `u:prowadzi`, sprawdza acykliczność, wypisuje najdłuższy łańcuch prerekwizytów i rysuje graf.

Czym graf RDF różni się od sieci z zadania 1, a czym jest do niej podobny? Dlaczego dwa literały „Anna” nie są równe? Która serializacja nadaje się do czytania, a która do przetwarzania wiersz po wierszu?

## Zadanie 3: SPARQL

Funkcja `tabela(graf, zapytanie)` w szablonie dokleja prefiksy i zwraca wynik jako DataFrame z IRI skróconymi do postaci `u:...`. Każde zapytanie piszcie w osobnej komórce. Zapytanie 1 jest gotowe jako przykład.

1. Studenci (`rdf:type u:Student`) z imieniem, nazwiskiem i rokiem studiów, posortowani po nazwisku. Ilu ich jest i czy wszyscy studenci z danych trafili do wyniku?
2. `FILTER`: kursy Wydziału Informatyki za co najmniej 5 ECTS. W osobnym zapytaniu osoby urodzone przed 1980 rokiem.
3. `OPTIONAL` i `GROUP_CONCAT`: wszyscy pracownicy (`rdf:type u:Pracownik`) z nazwami prowadzonych kursów, łącznie z tymi, którzy nic nie prowadzą.
4. `GROUP BY`: liczba zapisanych na każdy kurs, także na kursy bez zapisanych. Suma ECTS zaliczonych kursów na studenta, tylko dla studentów z co najmniej 15 ECTS (`HAVING`).
5. Ścieżki właściwości: wszystkie prerekwizyty u:ReprezentacjaWiedzy, także pośrednie (`u:wymagaKursu+`). Potem studenci zapisani na kurs, któremu brakuje zaliczonego prerekwizytu (dowolnego poziomu), z nazwą brakującego kursu (`FILTER NOT EXISTS`).
6. `CONSTRUCT`: zbudujcie graf z trójkami `?s u:uczonyPrzez ?p` dla studenta zapisanego na kurs prowadzony przez ?p. Ile trójek powstało? Ilu studentów uczy każdy prowadzący?

Dlaczego w wyniku zapytania 1 nie ma u:s10, choć jest zapisana na dwa kursy? Którzy pracownicy brakują w zapytaniu 3 i dlaczego? Czym różni się „brak zaliczenia prerekwizytu” w zapytaniu 5 od „braku informacji o zaliczeniu”?

## Zadanie 4: RDFS i OWL 2 RL w owlrl

1. Uzupełnijcie `SCHEMAT_TTL`: Student i Pracownik są podklasami Osoby, Wykladowca podklasą Pracownika, KursObowiazkowy i KursObieralny podklasami Kursu. Dodajcie `rdfs:domain` i `rdfs:range` dla właściwości obiektowych (`u:prowadzi`, `u:zapisanyNa`, `u:zaliczyl`, `u:wymagaKursu`, `u:promotor`, `u:autor`), a dla `u:naWydziale` tylko zakres, bo na wydziale są i kursy, i osoby oraz `rdfs:domain` dla `u:rokStudiow` i `u:ects`.
2. Napiszcie funkcję `domknij(*grafy, semantyka)`, która łączy grafy w nowy graf i wywołuje `owlrl.DeductiveClosure(semantyka).expand(g)`. Policzcie domknięcie `owlrl.RDFS_Semantics` dla danych i schematu: trójki przed i po, ile nowych, ile nowych `rdf:type`, ile trójek `rdf:type rdfs:Resource`, ile trójek z literałem jako podmiotem. Zmierzcie czas.
3. Jakie typy mają po domknięciu RDFS osoby u:p05 i u:s10 bez typu z zadania 2? Ilu jest teraz pracowników, studentów i osób? Czy ktokolwiek jest wykładowcą?
4. Komórka `OWL_TTL` zawiera gotowe aksjomaty OWL: przechodniość `u:wymagaKursu`, równoważność Wykladowca z ograniczeniem `owl:someValuesFrom`, rozłączność Student i Pracownik oraz odwrotną właściwość `u:prowadzonyPrzez`. Policzcie domknięcie `owlrl.OWLRL_Semantics` dla danych, schematu i aksjomatów. Zestawcie oba domknięcia w jednej tabeli. Kto jest teraz wykładowcą? Jakie prerekwizyty ma u:ReprezentacjaWiedzy? Kto prowadzi Sztuczną inteligencję według `u:prowadzonyPrzez`?
5. Do kopii grafu dodajcie trójkę `u:s01 u:prowadzi u:Logika` (s01 jest studentem). Jakie typy ma s01 po domknięciu RDFS? Co pokaże domknięcie OWL 2 RL? owlrl zapisuje wykryte sprzeczności jako trójki z predykatem `<http://www.daml.org/2002/03/agents/agent-ont#error>`.

Skąd się biorą trójki z literałem jako podmiotem i trójki `owl:sameAs` łączące zasób z samym sobą, i czy są dla was użyteczne? Dlaczego RDFS nie zgłasza błędu w punkcie 5? Który wniosek z punktu 4 nie był możliwy w RDFS?

## Zadanie 5: OWL w owlready2 i reasoner HermiT

Pierwsza komórka sprawdza Javę. Komórka `DEKLARACJE_TTL` deklaruje klasy i właściwości OWL (owlready2 potrzebuje jawnych deklaracji), a funkcja `wczytaj_ontologie(...)` tworzy z grafów rdflib nowy, niezależny świat owlready2. Uwaga: nie używajcie `from owlready2 import *`, bo nadpisuje między innymi nazwy z rdflib.

1. Wczytajcie ontologię (`swiat, onto = wczytaj_ontologie(graf, schemat, deklaracje)`). W funkcji `dodaj_aksjomaty(onto)` zapiszcie w składni owlready2: Wykladowca równoważny `Pracownik & prowadzi.some(Kurs)`, przechodniość `wymagaKursu` i rozłączność Student i Pracownik (`AllDisjoint`).
2. Uruchomcie `sync_reasoner_hermit(swiat, infer_property_values=True)` i zmierzcie czas. Porównajcie instancje Wykladowca z wynikiem OWL 2 RL z zadania 4. Wypiszcie `p05.is_a`, `s10.is_a` i prerekwizyty ReprezentacjaWiedzy.
3. Świat otwarty: zdefiniujcie klasę StudentBezZaliczen równoważną `Student & zaliczyl.max(0, Kurs)` i uruchomcie HermiT. Którzy studenci nie mają w danych żadnego zaliczenia, a kogo reasoner zaliczył do tej klasy? Zamknijcie świat dla jednego studenta (`close_world(onto.s08)`), uruchomcie reasoner ponownie i porównajcie. Jakie aksjomaty dodało `close_world`?
4. Na świeżym świecie (`wczytaj_ontologie` i `dodaj_aksjomaty`) zdefiniujcie klasę StudentPracownik jako podklasę Student i Pracownik. Uruchomcie HermiT i wypiszcie `swiat2.inconsistent_classes()`. Następnie dodajcie `s01 prowadzi Logika` i uruchomcie HermiT w bloku `try/except OwlReadyInconsistentOntologyError`.

Skoro HermiT i owlrl dały tych samych wykładowców, po co pełny reasoner? Podajcie aksjomat, którego wniosków owlrl nie wyprowadzi. Dlaczego bez `close_world` nikt nie jest StudentBezZaliczen? Czym różni się sytuacja z punktu 4 przed dodaniem trójki i po nim?

## Zadanie 6: Ontologia z WebProtégé

Scenariusze W1 do W4 wykonujecie w przeglądarce, w parach. Budujecie w nich tę samą ontologię co w notebooku: przestrzeń nazw `http://example.org/uczelnia#`, prefiks `u:`, te same nazwy klas i właściwości. Nie musicie wpisywać wszystkich osób i kursów z `UCZELNIA_TTL`. Wystarczy fragment opisany w krokach, taki jak w pliku `uczelnia_webprotege_przyklad.owl`.

Opis interfejsu odpowiada wersji WebProtégé z kodu źródłowego projektu (`https://github.com/protegeproject/webprotege`). Położenie przycisków na webprotege.stanford.edu może się nieco różnić. Jeśli nie widzicie opisanego elementu, szukajcie go po nazwie panelu lub sekcji, a nie po położeniu na ekranie.

WebProtégé nie uruchamia reasonera. Pokazuje tylko aksjomaty zapisane wprost: nie wyliczy, że ktoś jest wykładowcą, i nie ostrzeże, że ontologia jest niespójna. Dlatego klasyfikację i spójność sprawdzacie w notebooku po eksporcie (scenariusz W4 i punkty 6.1 do 6.4).

### Scenariusz W1: projekt i hierarchia klas

Cel: założyć projekt, ustawić czytelne IRI nowych encji i zbudować hierarchię klas z etykietami w dwóch językach.

Kroki:

1. Zalogujcie się i na liście projektów utwórzcie nowy projekt (Create New Project) o nazwie `uczelnia-NAZWISKO`. Język możecie zostawić domyślny.
2. Otwórzcie projekt i w menu Project wybierzcie Project Settings. W sekcji New Entity Settings ustawcie nadawanie IRI na podstawie podanej nazwy (Supplied name) zamiast domyślnego identyfikatora UUID, a jako prefiks IRI wpiszcie `http://example.org/uczelnia#`. W tych samych ustawieniach (New Entity Language Settings) ustawcie język etykiet na `pl`. Zapiszcie ustawienia.
3. W zakładce z hierarchią klas (Classes) utwórzcie pod `owl:Thing` klasy Osoba, Kurs, Wydzial i PracaDyplomowa. Pod Osoba utwórzcie Student i Pracownik, pod Pracownik klasę Wykladowca, a pod Kurs klasy KursObowiazkowy i KursObieralny. Nazwy wpisujcie bez polskich znaków i spacji, tak jak w notebooku.
4. Dla każdej klasy w sekcji Annotations dodajcie `rdfs:label` po polsku (np. „Wykładowca”, Language `pl`) i po angielsku („Lecturer”, Language `en`). Dla Osoba, Student, Pracownik, Wykladowca i Kurs dodajcie też `rdfs:comment` w obu językach, jedno zdanie definicji.
5. Wybierzcie dowolną klasę i sprawdźcie jej IRI w edytorze encji.

Oczekiwany rezultat: 9 klas w hierarchii jak w notebooku, wszystkie z IRI w postaci `http://example.org/uczelnia#Nazwa`, z etykietami pl i en.

Pytania kontrolne:

1. Jakie IRI dostałaby klasa Student, gdybyście pominęli krok 2? Czym grozi zmiana tego ustawienia w połowie pracy?
2. Czym różni się nazwa w IRI od `rdfs:label` i która z nich jest widoczna w drzewie klas?
3. Co oznacza, że klasa Wykladowca jest pod Pracownik? Zapiszcie to jedną trójką RDF.

### Scenariusz W2: właściwości i osobniki

Cel: dodać właściwości obiektowe i danych z dziedziną, zakresem i cechami oraz kilku osobników.

Kroki:

1. W zakładce właściwości (Properties) utwórzcie właściwości obiektowe `prowadzi`, `prowadzonyPrzez`, `zapisanyNa`, `zaliczyl`, `wymagaKursu`, `naWydziale`, `promotor`, `autor`. Dla każdej ustawcie dziedzinę (Domain) i zakres (Range) zgodnie ze schematem z zadania 4 (np. `prowadzi`: Domain Pracownik, Range Kurs; `naWydziale` tylko Range Wydzial).
2. Zaznaczcie `prowadzonyPrzez` jako właściwość odwrotną do `prowadzi`, a `wymagaKursu` jako przechodnią. Jeśli formularz właściwości nie ma odpowiedniego pola, zróbcie to w scenariuszu W3 w edytorze opisu encji (wpis `InverseOf: prowadzi`, `Characteristics: Transitive`).
3. Utwórzcie właściwości danych `imie`, `nazwisko` (Domain Osoba, Range `xsd:string`), `ects` (Domain Kurs, Range `xsd:integer`) i `rokStudiow` (Domain Student, Range `xsd:integer`). W sekcji Characteristics zaznaczcie Functional dla `ects` i `rokStudiow`.
4. W zakładce osobników (Individuals) utwórzcie: WI (Types: Wydzial), kursy Programowanie, Algorytmy, Logika, SztucznaInteligencja (KursObowiazkowy) i ReprezentacjaWiedzy (KursObieralny) z wartościami `ects` i `naWydziale WI`, prerekwizyty (`Algorytmy wymagaKursu Programowanie`, `SztucznaInteligencja wymagaKursu Algorytmy` i `Logika`, `ReprezentacjaWiedzy wymagaKursu SztucznaInteligencja`). Wartości właściwości wpisujcie w sekcji Relationships.
5. Dodajcie osoby: p01 (Pracownik, Anna Kowalska, prowadzi SztucznaInteligencja i ReprezentacjaWiedzy), p04 (Pracownik, Barbara Wójcik), p05 bez typu (Tomasz Kamiński, prowadzi Logika), s01 i s04 (Student, rok 1, zapisani na ReprezentacjaWiedzy; s01 zaliczył Programowanie, Algorytmy, Logika i SztucznaInteligencja, s04 tylko Programowanie) oraz pracę pd01 (PracaDyplomowa, autor s01, promotor p01).

Oczekiwany rezultat: 8 właściwości obiektowych, 4 właściwości danych, 12 osobników. p05 ma tylko wartość `prowadzi` i żadnego typu.

Pytania kontrolne:

1. Jaki typ będzie miał p05 według reasonera, choć go nie podaliście? Które dwa elementy ontologii za to odpowiadają?
2. Co oznacza, że `ects` jest funkcyjna? Czy wpisanie kursowi dwóch różnych wartości `ects` będzie błędem? Od czego to zależy?
3. Dlaczego prerekwizyt pośredni (Programowanie dla ReprezentacjaWiedzy) nie pojawia się przy kursie w WebProtégé?

### Scenariusz W3: aksjomaty klas

Cel: zapisać aksjomaty, których nie da się wyrazić w RDFS: rozłączność, klasę zdefiniowaną i ograniczenia liczności.

Kroki:

1. Jeśli w zakładce z klasami nie ma widoku z opisem encji w składni Manchester, dodajcie go: Add view, widok OWL Entity Description Editor. W razie trudności dodajcie nową zakładkę (Add tab) i umieśćcie widok w niej.
2. Dla klasy Student dopiszcie rozłączność z Pracownik (`DisjointWith: Pracownik`), a dla KursObowiazkowy rozłączność z KursObieralny.
3. Dla klasy Wykladowca zapiszcie równoważność `EquivalentTo: Pracownik and (prowadzi some Kurs)`. Klasa przestaje być tylko nazwą: staje się warunkiem, który reasoner może sprawdzić na osobnikach.
4. Dodajcie ograniczenia liczności jako nadklasy (SubClassOf): Kurs `naWydziale exactly 1 Wydzial`, PracaDyplomowa `promotor exactly 1 Pracownik` i `autor min 1 Student`.
5. Uzupełnijcie brakujące aksjomaty z W2 (odwrotność, przechodniość), jeśli nie udało się ich ustawić w formularzu.

Oczekiwany rezultat: dwie pary klas rozłącznych, klasa zdefiniowana Wykladowca, trzy ograniczenia liczności. Drzewo klas wygląda tak samo jak po W1, a żaden osobnik nie ma zapisanego typu Wykladowca.

Pytania kontrolne:

1. Dlaczego WebProtégé nie pokazuje p01 i p05 jako instancji Wykladowca, choć spełniają definicję?
2. Czym różni się `SubClassOf: Pracownik and (prowadzi some Kurs)` od `EquivalentTo` z kroku 3? Jakie wnioski daje każda z wersji?
3. Gdyby p01 był też zapisany jako Student, co powinien zgłosić reasoner i dlaczego edytor tego nie zrobi?

### Scenariusz W4: praca zespołowa i eksport

Cel: poznać funkcje pracy zespołowej i przenieść ontologię do notebooka.

Kroki:

1. Udostępnijcie projekt drugiej osobie z pary (Share, opcja udostępnienia wskazanym osobom) z prawem edycji. Druga osoba otwiera projekt ze swojego konta.
2. Przy klasie Wykladowca rozpocznijcie wątek dyskusji w panelu komentarzy (Comments, Start new thread), np. pytanie, czy doktorant prowadzący ćwiczenia jest wykładowcą. Druga osoba odpowiada w wątku. Włączcie obserwowanie klasy (Watch).
3. Niech druga osoba zmieni etykietę jednej klasy. Otwórzcie historię zmian projektu (History) i znajdźcie tę zmianę z autorem i datą. Sprawdźcie, gdzie jest opcja wycofania zmian z danej rewizji, ale jej nie używajcie.
4. Na liście projektów pobierzcie projekt (Download). W oknie wyboru formatu wybierzcie RDF/XML albo Turtle. WebProtégé zapisuje plik jako archiwum `.zip` z katalogiem, w którym leży plik ontologii.
5. Wgrajcie do Colaba archiwum `.zip` albo wypakowany plik pod nazwą `uczelnia_webprotege.owl` (RDF/XML) lub `uczelnia_webprotege.ttl` (Turtle) i wykonajcie punkty 6.1 do 6.4 w notebooku.

Oczekiwany rezultat: projekt widoczny dla dwóch kont, wątek z dwoma wpisami, zmiana w historii z nazwiskiem autora, plik eksportu wczytany w notebooku.

Pytania kontrolne:

1. Które z funkcji z tego scenariusza są potrzebne, gdy ontologię rozwija kilkanaście osób, a których brakuje przy pracy na pliku `.ttl` w repozytorium Git?
2. Jakie IRI ma ontologia w eksporcie i czym różni się od IRI klas?
3. Dlaczego nie wszystkie formaty z okna pobierania nadają się do wczytania przez rdflib?

### Sprawdzenie eksportu w notebooku

Komórka wczytująca szuka kolejno: `uczelnia_webprotege.owl` lub `.ttl`, archiwum `*ontologies*.zip`, pliku `uczelnia_webprotege_przyklad.owl`, a w ostateczności używa przykładu z komórki `PRZYKLAD_WEBPROTEGE_TTL`. Wypisuje, który plik wczytała. Punkty 6.1 i 6.2 są gotowe.

1. (6.1) Ile klas, właściwości obiektowych, właściwości danych i osobników ma eksport? Czy wszystkie encje mają IRI w przestrzeni `u:`? Które klasy nie mają etykiety lub komentarza w obu językach?
2. (6.2) Jakie aksjomaty zapisane w WebProtégé są w pliku (przechodniość, funkcyjność, odwrotność, rozłączność, równoważność, ograniczenia liczności)?
3. (6.3) Wczytajcie eksport do nowego świata owlready2 funkcją `swiat_z_eksportu(wp)` i uruchomcie HermiT. Wypiszcie klasy niespełnialne, wykładowców zapisanych wprost i wywnioskowanych, `p05.is_a`, prerekwizyty ReprezentacjaWiedzy i prowadzących SztucznaInteligencja według `prowadzonyPrzez`.
4. (6.4) Na świeżym świecie dodajcie osobnika x01 typu Student i Pracownik i uruchomcie HermiT w `try/except`. Na kolejnym świeżym świecie dodajcie pracy pd01 drugiego promotora p04. Czy ontologia jest niespójna? Co się zmienia po `AllDifferent([p01, p04])`?

Które wnioski reasonera były niewidoczne w WebProtégé? Dlaczego ograniczenie „dokładnie jeden promotor” nie zgłasza błędu przy dwóch promotorach, dopóki nie dodacie `AllDifferent`?

## Zadanie 7: Wikidata

Funkcja `zapytaj_wikidata(zapytanie, zapas)` wysyła zapytanie z nagłówkiem User-Agent, a przy błędzie lub limicie zapytań zwraca dane zapasowe. Zapytania są celowo małe (`LIMIT`, konkretne identyfikatory), bo publiczny punkt dostępowy ma limit czasu i ogranicza liczbę zapytań. Nie wysyłajcie ich w pętli.

1. Wykonajcie gotowe zapytanie `Q_UCZELNIE` (uniwersytety w Polsce z miastem i rokiem założenia; Q3918 to uniwersytet, Q36 to Polska). Zamieńcie kolumnę rok na liczby całkowite z brakami (`Int64`). Ile wierszy nie ma roku założenia, ile nie ma nazwy (nazwa równa identyfikatorowi), czy encje się powtarzają?
2. Wykonajcie `Q_LUDNOSC` (ludność Warszawy, Q270, przez węzły deklaracji `p:`, `ps:`, `pq:`). Potem zapytajcie o to samo przez `wdt:P1082` i porównajcie liczbę zwróconych wartości.

Które uczelnie pominęło zapytanie, które wymaga `wdt:P31 wd:Q3918` dokładnie, a nie przez `wdt:P31/wdt:P279*`? Dlaczego `wdt:P1082` zwraca jedną wartość i jakiej informacji nie da się zapisać zwykłą trójką `rdf:type` czy `wdt:`? Pamiętajcie, że Wikidata zmienia się z dnia na dzień, więc wasze liczby mogą się różnić od wyników kolegów sprzed tygodnia.

## Zadanie 8: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Czym różni się dziedziczenie w sieci semantycznej z wyjątkami od wnioskowania w RDFS i dlaczego to pierwsze jest niemonotoniczne?
2. Dlaczego zapytanie o `?p a u:Pracownik` nie zwraca u:p05 przed wnioskowaniem, a zwraca po domknięciu RDFS?
3. Czego nie da się wyrazić w RDFS, a da się w OWL? Podajcie dwa przykłady z tego laboratorium.
4. Dlaczego HermiT nie zaliczył nikogo do klasy StudentBezZaliczen, choć część studentów nie ma w danych żadnego zaliczenia, a SPARQL z `FILTER NOT EXISTS` takich studentów znajduje?
5. Czym różni się klasa niespełnialna od niespójnej ontologii?
6. Czym różni się `wdt:P31` od `rdf:type`, a `wdt:P279` od `rdfs:subClassOf`?
7. Dlaczego w WebProtégé u:p05 nie jest wykładowcą, a w notebooku po eksporcie jest? Co z tego wynika dla podziału pracy między edytorem a reasonerem?

## Do oddania

1. Notebook `lab01_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, rysunkami i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
2. Plik eksportu z WebProtégé (archiwum `.zip` albo `uczelnia_webprotege.owl` lub `.ttl`). Jeśli w zadaniu 6 notebook użył pliku przykładowego, a nie waszego eksportu, napiszcie to we wnioskach.
3. Jeśli w zadaniu 7 część wyników pochodzi z danych zapasowych, zaznaczcie to we wnioskach.
