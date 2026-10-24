# Laboratorium 3: Logika symboliczna i systemy regułowe

Dane: fragment ontologii uczelni z laboratorium 1 (Turtle wbudowany w notebook) i baza 24 reguł diagnostycznych sieci komputerowej
Środowisko: Google Colab (CPU wystarczy), Python bez Prologa, biblioteki rdflib i owlrl

## Cel

Laboratorium pokazuje, jak działa wnioskowanie logiczne od środka. Zaczynamy od semantyki: co znaczy, że formuła wynika z bazy, i ile kosztuje sprawdzenie tego w logice zdań. Potem budujecie w czystym Pythonie unifikację, silnik wnioskowania w przód i silnik wnioskowania wstecz, które pracują na faktach o uczelni znanych z laboratorium 1. Na tych samych danych zobaczycie, czym negacja jako porażka różni się od negacji w logice klasycznej i w OWL, dlaczego potrzebna jest stratyfikacja i jak reguły zapisać w SPARQL. Na koniec piszecie mały system ekspercki z objaśnieniami "dlaczego" i "jak" i sprawdzacie, jak strategia rozwiązywania konfliktów zmienia jego odpowiedzi.

## Przed zajęciami

Przeczytajcie `lab03_wprowadzenie.md`, zwłaszcza części o unifikacji i stratyfikacji. Otwórzcie `lab03_szablon.ipynb` w Google Colab. Pierwsza komórka instaluje `rdflib` i `owlrl` (kilka sekund), pozostałe biblioteki są w Colabie domyślnie. Komórki z danymi (Turtle uczelni, baza `KB_EWA`, generatory formuł, testy unifikacji, reguły uczelni i baza reguł diagnostycznych) są gotowe. Funkcje piszecie sami w komórkach oznaczonych komentarzem.

Wszystkie silniki piszemy od zera, bez Prologa i bez bibliotek regułowych. Chodzi o to, żebyście zobaczyli, skąd biorą się ograniczenia, o których mówi teoria. Notacja jest wspólna dla całego notebooka: zmienna to napis zaczynający się od `?`, atom to krotka `(predykat, arg1, arg2)`, a fakt to atom bez zmiennych, np. `("zaliczyl", "anna", "WDP")`.

## Zadanie 1: Logika zdań i predykatów

1. Zapiszcie w logice predykatów sześć zdań z notebooka (sekcja 1a), używając predykatów ze słownictwa uczelni: `Student`, `Pracownik`, `Wykladowca`, `Kurs`, `KursObieralny`, `PracaDyplomowa`, `prowadzi`, `zaliczyl`, `zapisanyNa`, `wymagaKursu`, `promotor`. Przy każdej formule zaznaczcie, czy jest klauzulą Horna. Zdanie 5 ("dokładnie jeden promotor") wymaga równości.
2. Sprawdźcie zdania 3 i 4 na faktach `FACTS`, czyli w jednym konkretnym modelu. Dla zdania 3 wypiszcie wszystkie kontrprzykłady, dla zdania 4 kurs, który jest świadkiem.
3. Napiszcie `pl_true(f, model)`, `symbols(f)` i `tt_entails(kb, alpha)`, która przegląda wszystkie modele i zwraca, czy `kb |= alpha`, ile modeli sprawdziła i kontrmodel, jeśli istnieje. Sprawdźcie zapytania z listy `QUERIES` dla bazy `KB_EWA` i policzcie jej modele.
4. Zmierzcie czas i liczbę modeli sprawdzanych przez `tt_entails` na bazie łańcuchowej `chain_kb(n)` dla n od 4 do 18.
5. Napiszcie DPLL z propagacją jednostkową i sprawdźcie nim to samo wynikanie (`chain_cnf(n)`) dla n do 1000. Potem uruchomcie go na zasadzie szufladkowej `pigeonhole(n)` dla n od 2 do 7 i narysujcie oba wzrosty na skali logarytmicznej.

Czym różni się sprawdzenie zdania 3 na danych od pytania, czy zdanie 3 wynika z bazy? Dlaczego zapytanie `zal_PO` nie wynika z `KB_EWA`, choć Ewa zaliczyła AiSD, i co mówi o tym kontrmodel? Dlaczego DPLL rozwiązuje łańcuch z 1000 symboli od razu, a przy zasadzie szufladkowej dla 7 przegródek potrzebuje kilku tysięcy wywołań?

## Zadanie 2: Unifikacja

1. Napiszcie `unify(x, y, theta=None, occurs_check=True)`, która zwraca najogólniejszy unifikator jako słownik albo `None`. Potrzebne będą funkcje pomocnicze: `walk` (podążanie za łańcuchem wiązań), `occurs` i `subst` (zastosowanie podstawienia do termu).
2. Uruchomcie ją na parach z `UNIFY_TESTS` i zbudujcie tabelę wyników. Sprawdźcie parę `?x` i `f(?x)` z occurs check i bez niego, a potem zastosujcie podstawienie otrzymane bez occurs check do `?x`.
3. Zmierzcie koszt occurs check na parze z sekcji 2b dla n od 10 do 18: czas z testem i bez, liczbę wiązań i rozmiar termu `subst("?xn", theta)`.

Co dokładnie się psuje, gdy pominiemy occurs check? Skąd bierze się wykładniczy wzrost czasu w 2b, skoro liczba wiązań rośnie liniowo? Dlaczego mimo to Prolog domyślnie tego testu nie wykonuje?

## Zadanie 3: Wnioskowanie w przód

1. Napiszcie `match_body` (generator podstawień spełniających ciało reguły) i `forward_chain(rules, facts)`, który powtarza rundy do punktu stałego i zapisuje dla każdej rundy liczbę odpaleń i nowych faktów. Uruchomcie go na regułach Horna `HORN` (R1 do R8). Wypiszcie wywnioskowane prerekwizyty MRW, listę wykładowców i pary `promotorStudenta`.
2. Uruchomcie ten sam silnik na wszystkich regułach `RULES`, czyli także na R9 do R11 z negacją jako porażką. Porównajcie wynik z oczekiwanym: ile faktów `mozeSieZapisac` powstało i ile z nich przeczy faktom `brakujePrereq`?
3. Zbudujcie graf zależności predykatów, przypiszcie predykatom warstwy (`stratify`) i napiszcie `stratified_chain`, który liczy punkt stały warstwa po warstwie. Ile jest teraz faktów `mozeSieZapisac` i kto narusza wymagania (jest zapisany na kurs bez prerekwizytów)?
4. Dopiszcie do bazy fakt `zaliczyl(hubert, Log)`, a osobno `wymagaKursu(MRW, UM)`, i dla każdego przypadku wypiszcie wnioski, które zniknęły, i te, które się pojawiły.
5. Zamieńcie w R9 `wymagaPosrednio` na `wymagaKursu` i porównajcie wyniki.

Dlaczego przebieg bez warstw daje fałszywe wnioski, choć każda reguła z osobna jest poprawna? Co w punkcie 4 pokazuje niemonotoniczność i dlaczego w logice klasycznej nie mogłaby wystąpić? Która wersja R9 lepiej oddaje regulamin studiów i jakie założenie o danych kryje się za każdą z nich?

## Zadanie 4: Wnioskowanie wstecz

1. Napiszcie klasę `Prover` z generatorem `prove(goal, theta, depth)`, który działa jak Prolog: najpierw fakty, potem reguły w kolejności z listy, literały ciała od lewej, przemianowanie zmiennych reguły przy każdym użyciu, negacja jako porażka i wyjątek po przekroczeniu głębokości. Generator zwraca podstawienie i drzewo dowodu. Liczcie użycia reguł i dopasowania faktów.
2. Zadajcie zapytanie `wymagaPosrednio(MRW, ?x)`. Ile jest odpowiedzi, a ile różnych odpowiedzi? Wypiszcie drzewo dowodu jednej z nich.
3. Udowodnijcie `mozeSieZapisac(dawid, MRW)` i znajdźcie wszystkie naruszenia. Wypiszcie drzewa dowodu.
4. Dla zapytań z komentarza w sekcji 4b porównajcie liczbę użyć reguł z liczbą odpaleń w pełnym przebiegu w przód z zadania 3.
5. Zastąpcie R7 wersją lewostronnie rekurencyjną `R7L` i sprawdźcie oba kierunki wnioskowania.

Kiedy wnioskowanie wstecz wykonuje mniej pracy niż w przód, a kiedy więcej? Skąd biorą się powtórzone odpowiedzi w punkcie 2? Dlaczego R7L nie przeszkadza we wnioskowaniu w przód, a wstecz prowadzi do nieskończonej rekursji, skoro jest logicznie równoważna R7?

## Zadanie 5: Reguły nad grafem RDF

1. Zapiszcie reguły R1 do R11 jako zapytania SPARQL CONSTRUCT, pogrupowane w te same warstwy co w zadaniu 3. Negację wyraźcie przez `FILTER NOT EXISTS`. Napiszcie funkcję, która w każdej warstwie wykonuje zapytania w pętli i dopisuje wyniki do kopii grafu, aż liczba trójek przestanie rosnąć.
2. Zamieńcie wynikowy graf z powrotem na fakty i porównajcie z wynikiem `stratified_chain` dla każdego wywnioskowanego predykatu.
3. Policzcie pary osiągalne ścieżką `u:wymagaKursu+` jednym zapytaniem SELECT. Potem w kopii grafu oznaczcie `u:wymagaKursu` jako `owl:TransitiveProperty`, uruchomcie `owlrl` i porównajcie domknięcie z `wymagaPosrednio`.

Które reguły uczelni da się wyrazić w OWL 2 RL, a których nie, i dlaczego? Ile rund potrzebowała każda warstwa i od czego to zależy? Po co w SPARQL reguła rekurencyjna, skoro ścieżka `wymagaKursu+` daje ten sam wynik jednym zapytaniem?

## Zadanie 6: System ekspercki z objaśnieniami

1. Przeczytajcie bazę `ES_RULES` (24 reguły), listę pytań `ASKABLE` i scenariusze `SCENARIOS`. Narysujcie na kartce, które reguły się nakładają.
2. Napiszcie silnik produkcyjny `es_forward` z pamięcią roboczą ze znacznikami czasu, refrakcją i trzema strategiami rozwiązywania konfliktów: kolejność, specyficzność, świeżość. Zbudujcie tabelę scenariusz x strategia (pierwsza diagnoza i liczba cykli) i dopiszcie kolumnę ze wszystkimi diagnozami.
3. Dla scenariusza S1 wypiszcie przebieg cykli dla każdej strategii: odpaloną regułę i zbiór konfliktowy.
4. Napiszcie klasę `Consultation`, która prowadzi konsultację wstecz od celu `diagnoza`, pyta o atrybuty z `ASKABLE` (odpowiedź bierze ze scenariusza) i zapamiętuje, dlaczego zadała każde pytanie. Uruchomcie ją dla S6 z kolejnością reguł według specyficzności i wypiszcie objaśnienie "dlaczego" dla pytania o błąd certyfikatu oraz objaśnienie "jak" dla diagnozy.
5. Dla wszystkich scenariuszy porównajcie liczbę zadanych pytań i diagnozę przy kolejności z listy i według specyficzności.

W których scenariuszach strategia zmienia diagnozę i która odpowiedź jest dla użytkownika trafniejsza? Dlaczego świeżość w S1 wybiera DHCP, choć dioda karty nie świeci? Ile pytań oszczędza konsultacja wstecz w porównaniu z kompletem obserwacji i co się dzieje, gdy użytkownik odpowiada "nie wiem"?

## Zadanie 7: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Czym różni się wynikanie `KB |= a` od wyprowadzalności `KB |- a` i co znaczy, że procedura wnioskowania jest poprawna i zupełna?
2. Dlaczego wynikanie w logice zdań jest rozstrzygalne, a w logice pierwszego rzędu tylko półrozstrzygalne?
3. Do czego służy occurs check i dlaczego Prolog domyślnie go nie wykonuje?
4. Dlaczego wnioskowanie w przód na regułach bez symboli funkcyjnych zawsze się zatrzymuje i co jest jego wynikiem?
5. Co łączy negację jako porażkę, założenie świata zamkniętego i niemonotoniczność wnioskowania?
6. Kiedy lepiej wnioskować wstecz, a kiedy w przód?
7. Dlaczego reasoner OWL nie wywnioskuje `mozeSieZapisac`, choć ma do dyspozycji te same fakty co nasz silnik?

## Do oddania

Notebook `lab03_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, wykresem z zadania 1, formalizacją zdań i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
