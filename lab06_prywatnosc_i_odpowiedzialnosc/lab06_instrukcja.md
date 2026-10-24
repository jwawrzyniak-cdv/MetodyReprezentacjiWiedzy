# Laboratorium 6: Prywatność, przejrzystość i uczciwość w systemach opartych na wiedzy

Dane: graf uczelni ze wspólnego słownictwa kursu rozszerzony o quasi-identyfikatory i fikcyjne dane wrażliwe (Turtle wbudowany w notebook), zbiór Adult z OpenML (id 1590), syntetyczna populacja 3000 studentów z regulaminem stypendium
Środowisko: Google Colab (CPU wystarczy), biblioteki rdflib, owlrl, pgmpy, scikit-learn

## Cel

Ostatnie laboratorium odwraca perspektywę. Do tej pory wnioskowanie było narzędziem, dziś zobaczycie, że ten sam mechanizm ujawnia dane, których nikt jawnie nie opublikował, i że usunięcie trójki z grafu nie jest jej ukryciem. Zmierzycie, ile kosztuje anonimizacja tabeli metodą k-anonimowości i dlaczego sama k-anonimowość nie chroni atrybutu wrażliwego. Zbudujecie widoki grafu dla różnych ról i sprawdzicie je na domknięciu, a nie na samym widoku. W drugiej części objaśnicie tę samą decyzję stypendialną regułami i siecią bayesowską, przeprowadzicie audyt uczciwości regulaminu, który nie używa płci, i znajdziecie zmienne zastępcze odpowiedzialne za nierówne skutki. Na koniec przypiszecie zbudowane systemy do pojęć RODO i AI Act.

Wszystkie dane osobowe w grafie uczelni i w populacji studentów są fikcyjne.

## Przed zajęciami

Przeczytajcie `lab06_wprowadzenie.md`, zwłaszcza części o k-anonimowości, kontroli dostępu na domknięciu i definicjach uczciwości. Otwórzcie `lab06_szablon.ipynb` w Google Colab. Pierwsza komórka instaluje `rdflib`, `owlrl` i `pgmpy` (kilkanaście sekund), scikit-learn jest w Colabie domyślnie. Gotowe są komórki z grafem uczelni (`TBOX_TTL`, `ABOX_TTL`), listą uczestników biegu, hierarchiami generalizacji dla Adult, generatorem populacji z regulaminem `REGULY` i strukturą sieci bayesowskiej `KRAWEDZIE`. Resztę piszecie sami w komórkach oznaczonych komentarzem.

Do zadania 6 potrzebny będzie dostęp do tekstów RODO (rozporządzenie (UE) 2016/679) i AI Act (rozporządzenie (UE) 2024/1689) w serwisie EUR-Lex. Korzystajcie z wersji polskiej i z wersji skonsolidowanej AI Act, bo akt był zmieniany w 2026 r.

## Zadanie 1: Wnioskowanie jako zagrożenie dla prywatności

1. Przeczytajcie TBox. Znajdźcie dwa aksjomaty, z których może wynikać przynależność do klasy `u:OsobaZOrzeczeniem`. Napiszcie funkcję `domknij(graf)` zwracającą kopię grafu z domknięciem OWL 2 RL i sprawdźcie, ile trójek przybyło.
2. Zbudujcie publikację dla badaczy: pseudonimy (skrót SHA-256 z solą) zamiast IRI studentów, bez imion, nazwisk, stypendium i bez jawnego typu `u:OsobaZOrzeczeniem`. Zapytajcie SPARQL o osoby z orzeczeniem przed domknięciem i po nim. Porównajcie wynik z danymi dziekanatu.
3. Dla każdego wywnioskowanego pseudonimu wypiszcie trójki, które posłużyły za przesłanki.
4. Wyciągnijcie z publikacji tabelę quasi-identyfikatorów (płeć, rok urodzenia, kod pocztowy, kierunek) i połączcie ją z listą uczestników biegu `bieg`. Ilu kandydatów z publikacji ma każda osoba z listy i czego atakujący się o niej dowiaduje?
5. Dla każdego niepustego podzbioru czterech quasi-identyfikatorów policzcie, ilu studentów jest w nim unikalnych, i podajcie k.

Dlaczego jedna z osób z orzeczeniem nie została wywnioskowana i co to mówi o sile tej ochrony? Czy dwóch kandydatów do jednej osoby z listy to dla niej dobra wiadomość, a dla drugiej osoby? Czy ukrycie TBox rozwiązałoby problem?

## Zadanie 2: k-anonimowość i l-różnorodność na zbiorze Adult

Quasi-identyfikatory: `age`, `sex`, `race`, `marital-status`, `native-country`. Atrybut wrażliwy: `income`. Hierarchie generalizacji są w szablonie.

1. Wczytajcie Adult (`fetch_openml("adult", version=2, as_frame=True)`), usuńcie wiersze z brakami, podzielcie dane 70/30 ze stratyfikacją po `income`. Policzcie k dla danych treningowych i liczbę wierszy unikalnych względem QI.
2. Przejdźcie po całej kracie generalizacji. Dla każdego węzła zapamiętajcie rozmiar klasy równoważności każdego wiersza i liczbę różnych wartości `income` w klasie. Napiszcie funkcję straty (średnia względna wysokość generalizacji, wiersz usunięty liczy się jako pełna generalizacja) i funkcję `najlepszy(k, l, max_supr=0.01)`.
3. Dla k w [2, 5, 10, 25, 50, 100] zbudujcie tabelę: węzeł, strata, liczba usuniętych wierszy, liczba wierszy w klasach jednorodnych względem `income` oraz zbalansowana trafność drzewa decyzyjnego (`max_depth=8`) uczonego na danych po anonimizacji, raz z dodatkowymi cechami spoza QI, raz na samych QI. Porównajcie z modelem na danych oryginalnych.
4. Przeprowadźcie atak homogeniczności przy k = 5: wypiszcie największe klasy jednorodne i pokażcie konkretną ofiarę.
5. Powtórzcie wyszukiwanie z warunkiem 2-różnorodności dla k w [2, 5, 10, 25].
6. Narysujcie stratę i udział wierszy w klasach jednorodnych w funkcji k.

Która z dwóch miar użyteczności (strata, trafność modelu) lepiej opisuje koszt anonimizacji i od czego to zależy? Dlaczego ofiara jest bezbronna, choć ukrywa się w grupie kilkuset osób? Czy distinct 2-różnorodność wystarcza przy atrybucie, w którym jedna wartość ma 75% udziału?

## Zadanie 3: Kontrola dostępu do grafu wiedzy

Polityka jest opisana w notebooku: student widzi katalog kursów, listy zapisanych i wszystkie swoje dane, prowadzący widzi katalog oraz studentów swoich kursów (z informacją o asystencie), dziekanat widzi wszystko. Fakty wrażliwe zna poza dziekanatem tylko osoba, której dotyczą. TBox jest publiczny.

1. Zapiszcie politykę dla ról student i prowadzący jako zapytania `CONSTRUCT` z parametrem `?ja` przekazywanym przez `initBindings`.
2. Napiszcie funkcję `wycieki(widok, ja)`, która zwraca fakty wrażliwe o innych osobach jawnie obecne w widoku oraz obecne po domknięciu widoku razem z TBox. Zbudujcie tabelę dla użytkowników s01, s03, s12, p1, p2, p3 i p4.
3. Napiszcie funkcję `napraw(widok, ja)`, która usuwa jak najmniej trójek ABox tak, żeby domknięcie nie zawierało wycieków. Zacznijcie od zachłannego usuwania trójek o osobach, których dotyczy wyciek, a potem próbujcie przywrócić każdą z usuniętych. Wypiszcie usunięte trójki.

Dlaczego kontrola "nie pokazuj trójek wrażliwych" jest formalnie spełniona, a mimo to nieskuteczna? Co tracą użytkownicy po naprawie? Czy dla prowadzącego p4 naprawa jest właściwym rozwiązaniem, czy raczej trzeba zmienić politykę?

## Zadanie 4: Objaśnianie decyzji

Regulamin stypendium za wyniki ma trzy reguły sprawdzane po kolei: R1 (urlop dziekański w roku oceny, odmowa), R2 (średnia co najmniej 4,6 i ECTS w terminie, przyznanie), R3 (średnia co najmniej 4,2, ECTS w terminie i udział w hackathonie, przyznanie). W pozostałych przypadkach odmowa. Zmienna `wyroznienie` oznacza, że student faktycznie należy do najlepszych, ale system jej nie widzi.

1. Napiszcie silnik `wnioskuj(r, reguly)`, który zwraca decyzję i ślad warunków, oraz funkcję `dlaczego(r)`, która przy odmowie wypisuje dla każdej reguły przyznającej niespełnione warunki albo regułę, która rozstrzygnęła wcześniej. Sprawdźcie zgodność silnika z wektorową wersją regulaminu na całej populacji.
2. Napiszcie `kontrfakty(r, niezmienne, max_zmian=2)`, która szuka najmniejszych zbiorów zmian odwracających odmowę: średnia rośnie co 0,05, cechy logiczne się przełączają. Zastosujcie obie funkcje do studentki o indeksie 76, najpierw z urlopem jako cechą niezmienialną, potem bez tego ograniczenia. Powtórzcie dla osoby bez urlopu, z odmową, faktycznie wyróżniającej się i ze średnią co najmniej 4,3.
3. Nauczcie sieć bayesowską o zadanej strukturze (estymator `DiscreteBayesianEstimator`, `prior_type="BDeu"`, `equivalent_sample_size=10`). Napiszcie `p_wyr(dowody)`, która liczy P(Wyroznienie = tak | dowody) przez wymnożenie czynników CPT. Dla studentki 76 policzcie posterior przy wszystkich dowodach i po usunięciu każdego dowodu z osobna, a wkład dowodu wyraźcie zmianą log-szans.
4. Policzcie wartości Shapleya dla wkładów pięciu dowodów (średnia po 120 kolejnościach). Porównajcie ich sumę z różnicą logit(posterior) i logit(prior) oraz z sumą wkładów z punktu 3.

Co studentka 76 może zrobić, żeby dostać stypendium, i co to mówi o regule R1? Dlaczego usunięcie dowodu `Urlop` nie zmienia posterioru, choć według regulaminu to właśnie urlop rozstrzyga? Które objaśnienie sieci (usuwanie dowodów czy wartości Shapleya) pokazalibyście studentce i dlaczego?

## Zadanie 5: Audyt uczciwości regulaminu

1. Napiszcie wektorową wersję regulaminu z parametrami (progi, blokada urlopu, wymagana aktywność) i funkcję `audyt(dec)`: liczba stypendiów, stopy selekcji w grupach K i M, różnica parytetu, disparate impact, TPR w grupach i różnica equal opportunity, precyzja i czułość względem `wyroznienie`. Czy regulamin spełnia regułę 80%?
2. Poszukajcie zmiennych zastępczych: porównajcie średnie cech w grupach płci i policzcie AUC regresji logistycznej przewidującej płeć z każdej cechy osobno i ze wszystkich razem (5-krotna walidacja). Sprawdźcie, jak udział w hackathonie zależy od płci w obrębie kierunku.
3. Zaproponujcie poprawki: usunięcie R1, zastąpienie warunku "hackathon" warunkiem "hackathon lub koło naukowe", obie zmiany razem. Potem dobierzcie progi (siatka co 0,05) tak, żeby liczba stypendiów różniła się od regulaminowej najwyżej o 10%, a precyzja była możliwie wysoka. Zestawcie audyty wszystkich wariantów w jednej tabeli i narysujcie stopy przyznań i TPR w grupach.

Skoro regulamin nie używa płci, skąd bierze się nierówność? Która zmienna zastępcza działa bezpośrednio, a która przez kierunek studiów? Ile kosztuje poprawka i kto o niej powinien zdecydować?

## Zadanie 6: RODO i AI Act

Wypełnijcie tabelę w notebooku dla systemów z zadań 1 do 5 (graf uczelni z publikacją, system stypendialny). W kolumnie "przepis" podawajcie tylko te jednostki redakcyjne (artykuł, ustęp, litera, punkt załącznika, motyw), które sami odnaleźliście w tekście aktu. Jeśli odpowiedź nie jest jednoznaczna, napiszcie, od czego zależy.

Pytania z tabeli: czy pseudonimizowany graf to dane osobowe, czy informacja o orzeczeniu należy do szczególnych kategorii danych, czy system stypendialny dokonuje profilowania, czy decyzja jest oparta wyłącznie na zautomatyzowanym przetwarzaniu i jakie prawa ma student, czy regulamin z trzech reguł jest systemem AI, do jakiej kategorii ryzyka należy system wspomagający decyzje w edukacji i jakie obowiązki AI Act odpowiadają zadaniom 2 do 5.

## Zadanie 7: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Dlaczego usunięcie faktu wrażliwego z grafu nie wystarcza, gdy publikowany jest TBox, i jaki warunek musi spełniać widok, żeby był bezpieczny?
2. Czym różnią się pseudonimizacja i anonimizacja i dlaczego skrót SHA-256 numeru albumu nie jest anonimizacją?
3. Na czym polega atak homogeniczności i jak chroni przed nim l-różnorodność?
4. Co gwarantuje prywatność różnicowa, a czego nie gwarantuje k-anonimowość?
5. Czym różni się odpowiedź na pytanie "dlaczego nie" od kontrfaktu i kiedy kontrfakt nie istnieje?
6. Dlaczego regulamin, który nie używa atrybutu chronionego, może naruszać regułę 80%?
7. Dlaczego przy różnych częstościach klasy pozytywnej w grupach nie da się mieć jednocześnie równej precyzji i równych stóp błędów?

## Do oddania

Notebook `lab06_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, wykresami z zadań 2 i 5, wypełnioną tabelą z zadania 6 i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
