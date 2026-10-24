# Laboratorium 2: Ramy, skrypty i sieci powiązań

Dane: baza wiedzy o uczelni, skrypty i sieć powiązań wbudowane w notebook; angielski WordNet pobierany przez `nltk.download("wordnet")`
Środowisko: Google Colab (CPU wystarczy), Python, networkx, rdflib, owlrl, nltk

## Cel

Laboratorium pokazuje trzy formalizmy wiedzy stereotypowej i ich granice. Zbudujecie w czystym Pythonie mały system ram ze slotami, fasetami, dziedziczeniem i demonami, sprawdzicie, gdzie wielokrotne dziedziczenie wartości domyślnych prowadzi do konfliktu, i porównacie to z OWL, który wartości domyślnych nie ma. Potem dopasujecie krótkie historie do skryptów Schanka, uzupełnicie zdarzenia, o których historia milczy, i wykryjecie odstępstwa. Na końcu zaimplementujecie rozprzestrzenianie aktywacji w polskiej sieci powiązań i porównacie skojarzenia z podobieństwem w WordNecie.

## Przed zajęciami

Przeczytajcie `lab02_wprowadzenie.md`. Otwórzcie `lab02_szablon.ipynb` w Google Colab. Pierwsza komórka instaluje rdflib i owlrl, reszta bibliotek jest w Colabie. Komórka importów pobiera WordNet (kilkanaście sekund przy pierwszym uruchomieniu).

Szablon zawiera gotowe dane: szkielet klasy `Frame`, bazę wiedzy o uczelni, ramy ptaków i diamentu Nixona, tekst Turtle o ptakach, trzy skrypty z pięcioma historiami, listę krawędzi sieci powiązań i pary pojęć do porównania z WordNetem. Fragmenty do uzupełnienia oznaczone są komentarzem `# UZUPEŁNIJ`. Zadania są zależne: zadania 2 i 3 korzystają z metod napisanych w zadaniu 1.

## Zadanie 1: System ram

Klasa `Frame` przechowuje sloty jako słownik faset: `value`, `default`, `type`, `card`, `if_needed`, `if_added`. Metody `define`, `own`, `is_a`, `linearize` (kolejność przodków: `"dfs"` albo `"mro"`, czyli linearyzacja C3) i `put` (zapis z kontrolą typu, liczności i demonem if-added) są gotowe.

1. Uzupełnijcie `local_answer(frame, slot)`: odpowiedź jednej ramy z łańcucha przodków, w kolejności własna wartość, demon if-needed wywołany dla ramy, od której zaczęło się wyszukiwanie (`self`), wartość domyślna. Demon zwracający `None` oznacza brak odpowiedzi. Zwróćcie parę (wartość, nazwa fasety) albo `(MISSING, None)`.
2. Uzupełnijcie `lookup(slot, strategy)`: przejście po `self.linearize(strategy)` i zwrócenie pierwszej odpowiedzi jako trójki (wartość, rama źródłowa, faseta).
3. Uruchomcie komórkę z bazą wiedzy. Wypiszcie kolejność MRO i DFS dla Marka (StudentPracownik).
4. Zbudujcie tabelę dla instancji ania, piotr, marek, nowak i slotów imie, wiek, tryb, zajecia, rok_studiow, limit_ksiazek, parking: wartość, rama, z której pochodzi, i faseta. Dla czytelności pokażcie też wersję szeroką (instancja w kolumnach).
5. Sprawdźcie fasety `type` i `card`: rok urodzenia jako tekst, kurs zaliczony podany jako napis zamiast ramy, osoba jako zaliczony kurs, drugi i trzeci kierunek studiów, instancja jako rodzic ramy. Co zostało przyjęte, a co odrzucone?

Skąd Piotr ma tryb niestacjonarny, a skąd Ania stacjonarny i czym te dwa źródła się różnią? Dlaczego rama klasy Student nie ma wieku, a ma tryb?

## Zadanie 2: Wartości domyślne, wyjątki i wielokrotne dziedziczenie

1. Wpiszcie Ani wyjątek: zajęcia według indywidualnego harmonogramu. Porównajcie `lookup("zajecia")` przed i po zmianie oraz z wynikiem dla Piotra.
2. Uzupełnijcie `candidates(frame, slot)` (wszystkie ramy z łańcucha, które same coś mówią o slocie) i `lookup_skeptical(frame, slot)`: kandydat odpada, jeśli inny kandydat jest jego potomkiem; jeśli pozostali się zgadzają, zwróćcie wartość, w przeciwnym razie zgłoście wyjątek `Conflict`.
3. Zbudujcie tabelę strategii dfs, mro i sceptycznej dla par (tux, lata), (marek, limit_ksiazek), (marek, parking), (nixon, pacyfista), (nixon_odwr, pacyfista).
4. Pokażcie niemonotoniczność: utwórzcie instancję kos ramy Ptak, odczytajcie `lata`, a potem przepnijcie ją pod ramę Pingwin i odczytajcie ponownie.
5. Wczytajcie `PTAKI_TTL` do rdflib, policzcie domknięcie owlrl (`owlrl.DeductiveClosure(owlrl.OWLRL_Semantics).expand(g)`) i zapytaniem SPARQL sprawdźcie, jakie wartości `:lata` ma kos. Powtórzcie po dodaniu `WYJATEK_TTL`, w którym kos jest pingwinem. Napiszcie zapytanie, które wykrywa dwie różne wartości właściwości funkcyjnej.

Która strategia daje dla parkingu Marka wynik zgodny z intuicją i dlaczego DFS się myli? Czy wynik dla limitu Marka i dla Nixona wynika z wiedzy, czy z kolejności wpisania rodziców? Czy owlrl sam zgłosił sprzeczność? Dlaczego OWL nie może wyrazić reguły "ptak lata, chyba że wiadomo inaczej"?

## Zadanie 3: Demony

1. Dopiszcie do ramy Student demona if-needed dla slotu `ects`: suma punktów ECTS zaliczonych kursów. Wypiszcie wiek i ECTS Ani, Piotra i Marka. Zmieńcie Ani rok urodzenia i odczytajcie wiek ponownie.
2. Dopiszcie demona if-added dla slotu `zaliczyl`, który odrzuca kurs, jeśli student nie zaliczył wszystkich kursów ze slotu `wymaga` tego kursu (demon zgłasza wyjątek, a `put` wycofuje zmianę). Sprawdźcie kolejno: Piotr i Sztuczna inteligencja, Piotr i Algorytmy, Ania i Sztuczna inteligencja, Ania i Metody reprezentacji wiedzy, Marek i Bazy danych. Po każdej próbie wypiszcie wynik i ECTS.
3. Zdefiniujcie ramę PracaDyplomowa ze slotem `promotor` (typ: Pracownik) i dopiszcie do ramy Pracownik slot `prace`. Napiszcie dwa demony if-added utrzymujące relację odwrotną: wpisanie promotora dopisuje pracę do jego prac i odwrotnie. Przypiszcie promotora przy `Frame.check_changes = False`, a potem `True`, i porównajcie liczbę wywołań demonów (`Frame.demon_calls`).

Jakie są zalety i koszty liczenia ECTS przy każdym odczycie? Czy kolejność wpisywania zaliczeń ma znaczenie? Co się stało bez sprawdzania zmian i jak ten sam problem rozwiązuje OWL?

## Zadanie 4: Skrypty Schanka

Szablon zawiera trzy skrypty: `EGZAMIN` (egzamin ustny), `DZIEKANAT` (odbiór zaświadczenia) i `RESTAURACJA`. Każdy ma role (zmienne zaczynające się od `?`), rekwizyty, warunki wejścia, wyniki, sceny ze zdarzeniami (wykonawca, czynność, obiekt) i listę zdarzeń opcjonalnych. Słownik `STORIES` zawiera pięć historii. Funkcje `unify` i `instantiate` są gotowe.

1. Uzupełnijcie `match_story(script, story)`. Dla każdego zdarzenia historii szukajcie pierwszego pasującego zdarzenia skryptu za bieżącą pozycją, z zachowaniem dotychczasowych wiązań ról. Zdarzenia skryptu przeskoczone po drodze dostają status "domyślne", "pominięte (opcjonalne)" albo, jeśli wcześniej było odstępstwo, "niepewne (po odstępstwie)". Zdarzenie historii bez dopasowania jest odstępstwem: odróżnijcie "zła kolejność" (pasuje do zdarzenia wcześniejszego) od "brak w skrypcie".
2. Pokażcie wynik dla historii "Ania na egzaminie", "Ola i ściąga" i "Tomek bez kolejki": wiązania ról, status każdego zdarzenia historii i tabelę zdarzeń skryptu.
3. Uzupełnijcie `answer(script, story, question)`, gdzie pytanie jest zdarzeniem. Odpowiedź to "tak (podano wprost)", "tak (wniosek ze skryptu)" albo "nie wiadomo (...)" z powodem. Zadajcie co najmniej pytania: czy Ania okazała legitymację, czy dr Nowak wpisał ocenę do protokołu (u Ani i u Oli), czy zadał pytanie dodatkowe, czy Kasia odebrała zaświadczenie, czy pani Ewa podbiła pieczątkę, czy Jan zapłacił i poprosił o rachunek, czy Jan zjadł schabowego.
4. Napiszcie `score(script, story)` (na przykład odsetek dopasowanych zdarzeń historii) i `choose_script(story)`. Zbudujcie tabelę wyników dla wszystkich historii i skryptów.

Jakie wnioski o historii Ani wyprowadza skrypt, choć historia ich nie zawiera? Dlaczego dla Oli wpis do protokołu powinien być niepewny i co by się stało, gdyby dopasowanie ignorowało odstępstwa? Czy odpowiedź "nie wiadomo" na pytanie o schabowego jest poprawna i czego brakuje, żeby odpowiedzieć "nie"? Co rozstrzyga o wyborze skryptu, gdy historie mają wspólne czynności?

## Zadanie 5: Sieć powiązań i rozprzestrzenianie aktywacji

Tekst `EDGES_TXT` zawiera polską sieć powiązań w formacie `pojęcie;pojęcie;waga`.

1. Zbudujcie nieskierowany graf ważony w networkx. Podajcie liczbę pojęć i krawędzi, spójność, średnicę i pojęcia o najwyższym stopniu.
2. Uzupełnijcie `spread(G, sources, decay, threshold, max_steps, fan_out)` według pseudokodu z wprowadzenia: źródła mają aktywację 1, każdy węzeł o aktywacji co najmniej równej progowi wysyła sygnał raz, sygnał do sąsiada to `a(i) * w(i, j) * decay` (przy `fan_out=True` dodatkowo dzielony przez stopień węzła `i`), aktywację obcinamy do 1.
3. Policzcie aktywację ze źródeł `["lekarz"]`, `["szpital"]` i `["lekarz", "szpital"]` przy domyślnych parametrach. Zbudujcie tabelę 12 najbardziej aktywnych pojęć (bez źródeł) z kolumną torowania: aktywacja przy dwóch źródłach minus większa z aktywacji przy jednym źródle.
4. Zbadajcie wpływ parametrów dla źródła "lekarz": decay 0,3, 0,4, 0,6 i 0,8 przy progach 0,05, 0,1 i 0,2 bez normalizacji oraz `fan_out=True` przy decay 1,0 i progach 0,01, 0,02 i 0,05. Dla każdego ustawienia podajcie liczbę pojęć z aktywacją co najmniej równą progowi, aktywację pojęć pielęgniarka, kucharz i kelner oraz trzy najbardziej aktywne pojęcia.
5. Narysujcie sieć z kolorem węzła równym aktywacji, obok siebie dla źródła "lekarz" i dla źródeł "lekarz" i "szpital".
6. Dla par z listy `PAIRS` policzcie średnią aktywację (b ze źródła a i a ze źródła b), `path_similarity` i `wup_similarity` podanych synsetów WordNetu oraz najniższy wspólny hiperonim (`lowest_common_hypernyms`). Policzcie korelację Spearmana aktywacji z każdą z miar.

Które pojęcia zyskują najwięcej przy dwóch źródłach i dlaczego właśnie one? Przy jakich parametrach aktywacja przestaje cokolwiek różnicować? Dlaczego lekarz i szpital są w sieci silnie związani, a w WordNecie słabo, i jaki rodzaj wiedzy mierzy każda z tych miar?

## Zadanie 6: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Czym różni się faseta `value` od fasety `default` i co z tego wynika dla wyjątków?
2. Dlaczego przeszukiwanie w głąb daje dla parkingu Marka inną odpowiedź niż linearyzacja C3?
3. Co oznacza, że wnioskowanie w ramach jest niemonotoniczne, i dlaczego OWL takiego wnioskowania nie dopuszcza?
4. Jakie ryzyko niosą demony if-added w porównaniu z aksjomatem owl:inverseOf?
5. Jakie trzy rzeczy daje dopasowanie historii do skryptu i dlaczego wykrywanie odstępstw jest konieczne?
6. Dlaczego bezwzględna wartość aktywacji nie ma interpretacji, a ranking pojęć ją ma?

## Do oddania

Notebook `lab02_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, wykresem sieci i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
