# Laboratorium 5: Grafy wiedzy w zastosowaniach

Dane: Wikidata (zapytania SPARQL do Wikidata Query Service), fragmenty haseł polskiej Wikipedii (CC BY-SA 4.0), dane zapasowe wbudowane w notebook
Środowisko: Google Colab (CPU wystarczy)

## Cel

Na poprzednich zajęciach budowaliście małe ontologie, ramy, reguły i sieci bayesowskie. Tym razem te same narzędzia trafiają na prawdziwe dane. Zbudujecie graf wiedzy o laureatach Nagrody Nobla urodzonych na terenie dzisiejszej Polski: część faktów pobierzecie z Wikidata, część wydobędziecie z tekstu haseł Wikipedii. Na tym grafie uruchomicie odpowiadanie na pytania z wnioskowaniem OWL 2 RL, wyszukiwanie semantyczne oparte na embeddingach i mały model językowy, który odpowiada na te same pytania bez kontekstu i z faktami z grafu. Na końcu porównacie podejście symboliczne z neuronowym pod względem trafności, halucynacji, kosztu i weryfikowalności odpowiedzi.

## Przed zajęciami

Przeczytajcie `lab05_wprowadzenie.md`. Otwórzcie `lab05_szablon.ipynb` w Google Colab (runtime CPU). Pierwsze komórki instalują `rdflib`, `owlrl` i `sentence-transformers` oraz pobierają polski model spaCy `pl_core_news_sm`. W zadaniach 4 i 5 notebook pobiera z Hugging Face dwa modele: `paraphrase-multilingual-MiniLM-L12-v2` (około 0,5 GB) i `Qwen/Qwen2.5-0.5B-Instruct` (około 1 GB). Warto uruchomić komórkę z wczytaniem modelu językowego wcześniej, żeby pobieranie nie zabierało czasu zadania 5.

Notebook wysyła zapytania do Wikidata Query Service i do API MediaWiki. Funkcja `wikidata_csv` z szablonu dołącza nagłówek User-Agent, którego wymagają zasady Wikimedia. Nie usuwajcie go i nie wysyłajcie zapytań w pętli bez przerw: cała grupa korzysta z tego samego publicznego serwisu, który ogranicza liczbę zapytań. Gdy Wikidata nie odpowiada, funkcja `with_fallback` użyje danych zapasowych zapisanych w notebooku, a w wydruku pojawi się informacja o źródle danych. Wyniki mogą się wtedy nieznacznie różnić od bieżącego stanu Wikidata.

W szablonie nagłówki odpowiadają numeracji poniżej. Komentarze w komórkach z kodem opisują, co ma się w nich znaleźć.

## Zadanie 1: Graf wiedzy z Wikidata

1. Przeczytajcie zapytanie `Q_LAUREACI`. Zwraca ono laureatów sześciu nagród (literatura, pokój, fizyka, chemia, medycyna, ekonomia), którzy urodzili się w miejscu leżącym dziś w Polsce, wraz z płcią, datą urodzenia, nagrodą i rokiem jej przyznania. Wskażcie, w którym miejscu zapytanie korzysta z węzła wypowiedzi (`p:`, `ps:`, `pq:`) i dlaczego sam `wdt:P166` by nie wystarczył.
2. Pobierzcie wynik przez `with_fallback`. Napiszcie dwa kolejne zapytania: `Q_UCZELNIE` (uczelnie `wdt:P69` znalezionych osób, z etykietami po polsku) oraz `Q_JEDNOSTKI` (łańcuch `wdt:P131` w górę od miejsc urodzenia, z etykietami). Identyfikatory osób i miejsc wstawcie do klauzuli `VALUES`. Do `Q_JEDNOSTKI` dodajcie wiersz `hint:Query hint:optimizer "None" .`, bez którego zapytanie ze ścieżką `wdt:P131*` potrafi przekroczyć limit czasu. Między zapytaniami odczekajcie sekundę.
3. Zbudujcie graf `rdflib.Graph` z prefiksami `wd`, `wdt`, `p`, `ps`, `pq` i własną przestrzenią `ex`. Dodajcie etykiety `rdfs:label` po polsku, krawędzie `wdt:P19`, `wdt:P166`, `wdt:P69`, `wdt:P131`, datę urodzenia, płeć (`ex:plec`) i węzły wypowiedzi z rokiem nagrody. Wypiszcie liczbę trójek i liczności predykatów.
4. Przenieście krawędzie między encjami do `networkx.MultiDiGraph` i policzcie liczbę węzłów i krawędzi, gęstość, liczbę słabo spójnych składowych, średni, medianowy i maksymalny stopień. Pokażcie osiem węzłów o największym stopniu i narysujcie rozkład stopni.
5. Sprawdźcie jakość danych zapytaniami SPARQL na lokalnym grafie: osoby z więcej niż jednym miejscem urodzenia, osoby z więcej niż jedną nagrodą, węzły bez polskiej etykiety, laureaci bez informacji o uczelni.

Który węzeł ma największy stopień i co to mówi o strukturze grafu? Czy osoba z dwoma miejscami urodzenia to błąd w danych? Co oznacza brak `wdt:P69` u części laureatów?

## Zadanie 2: Ekstrakcja trójek z tekstu i łączenie encji

Szablon zawiera słownik `TEXTS` z początkowymi zdaniami dziesięciu haseł polskiej Wikipedii, zbiór `GOLD` z ręcznie oznaczonymi trójkami (miejsca w mianowniku) i słownik `EL_GOLD` z poprawnymi identyfikatorami Wikidata.

1. (opcjonalnie) Jednym zapytaniem do API MediaWiki (`action=query`, `prop=revisions`) pobierzcie numery bieżących rewizji tych haseł. Do czego przydają się przy ponownym wykorzystaniu tekstu na licencji CC BY-SA?
2. Napiszcie ekstraktor czterech relacji: `urodzonyW`, `zmarlW`, `rokUrodzenia`, `laureat` (dziedzina nagrody). Wystarczą wyrażenia regularne, np. dla zapisu `(ur. 2 lipca 1923 w Prowencie, zm. ...)`. Przygotujcie dwa warianty: bez lematyzacji (nazwa miejsca tak, jak stoi w tekście) i z lematyzacją spaCy. Lematyzujcie nazwę w kontekście całego zdania, a nie jako osobny napis, i porównajcie oba sposoby na dwóch, trzech przykładach.
3. Policzcie precyzję, pełność i F1 ogółem i osobno dla każdej relacji. Wypiszcie trójki pominięte i błędne i nazwijcie przyczynę każdego błędu.
4. Poprawcie ekstraktor tak, żeby obsłużył konstrukcję współrzędną "z fizyki (1903) i chemii (1911)". Ponownie policzcie miary. Czy ręczny słownik poprawek lematów byłby dobrym rozwiązaniem pozostałych błędów?
5. Wygenerujcie kandydatów do łączenia encji jednym zapytaniem SPARQL: etykiety (tytuły haseł i wydobyte miejsca urodzenia) w `VALUES`, dopasowanie `rdfs:label` lub `skos:altLabel` po polsku, liczba linków do Wikipedii (`wikibase:sitelinks`), opis i informacja, czy kandydat jest człowiekiem (`wdt:P31 wd:Q5`). Użyjcie `with_fallback` z danymi `FALLBACK_EL`.
6. Zaimplementujcie dwie strategie wyboru: (a) kandydat o największej liczbie linków, (b) to samo po zawężeniu kontekstem (dla osób tylko ludzie, dla miejsc tylko kandydaci obecni w grafie z zadania 1). Policzcie trafność dla osób i miejsc względem `EL_GOLD` i pokażcie błędne przypadki.
7. Połączcie osoby przez sitelink: element Wikidata, który jest tematem (`schema:about`) artykułu o danym tytule w polskiej Wikipedii. Dla której osoby łączenie po etykiecie zawiodło i dlaczego?

Dlaczego miary trzeba podawać osobno dla relacji? Skąd wzrost F1 po lematyzacji i jakie błędy zostały? Dlaczego heurystyka popularności myli się przy nazwach miejscowości?

## Zadanie 3: Odpowiadanie na pytania z grafu

Lista `QUESTIONS` zawiera dwanaście pytań po polsku z odpowiedziami wzorcowymi. Pole `typ` mówi, czy wystarczy jedna z odpowiedzi (`jedna`), czy potrzebne są wszystkie (`lista`). Funkcja `score` liczy trafność i pokrycie.

1. Zapiszcie TBox w Turtle: `wdt:P131` jako `owl:TransitiveProperty`, `ex:urodzonyNaTerenie` zdefiniowane łańcuchem właściwości (`owl:propertyChainAxiom` z listą `wdt:P19`, `wdt:P131`) oraz `wdt:P19` jako podwłaściwość `ex:urodzonyNaTerenie`. Po co potrzebny jest ten ostatni aksjomat?
2. Skopiujcie graf, dołączcie TBox i uruchomcie `owlrl.DeductiveClosure(owlrl.OWLRL_Semantics).expand(...)`. Porównajcie liczbę trójek przed i po oraz liczbę krawędzi `wdt:P131`.
3. Napiszcie dwa zapytania o laureatów urodzonych w województwie dolnośląskim: ze ścieżką `wdt:P19/wdt:P131*` na grafie bez wnioskowania i z `ex:urodzonyNaTerenie` na grafie po wnioskowaniu. Sprawdźcie, że dają ten sam wynik, a zwykłe `wdt:P19` daje wynik pusty.
4. Zbudujcie system pytanie, szablon, zapytanie:
   - funkcja `resolve(fragment, węzły)` dopasowuje fragment pytania do węzła grafu po rdzeniach słów (np. pierwszych pięciu literach), co radzi sobie z odmianą ("Władysław Reymont", "w województwie dolnośląskim");
   - lista szablonów: wyrażenie regularne i funkcja odpowiedzi dla sześciu typów pytań (miejsce urodzenia, rok nagrody, uczelnia, laureaci nagrody, laureaci z obszaru, dwukrotni laureaci);
   - każda odpowiedź zwraca listę etykiet i uzasadnienie w postaci ścieżki trójek. Dla pytań o obszar znajdźcie ścieżkę najkrótszą drogą w `networkx`.
5. Policzcie trafność na `QUESTIONS`. Pokażcie uzasadnienia dla dwóch pytań i odpowiedzi na kilka pytań spoza zestawu, np. o laureatów urodzonych we Wrocławiu, w Austro-Węgrzech i w Krakowie oraz pytanie, którego żaden szablon nie obsługuje.

Co oznacza pusty wynik dla Krakowa: że nikt się tam nie urodził, czy coś innego? Jak zachowuje się system na pytaniu spoza szablonów?

## Zadanie 4: Wyszukiwanie semantyczne faktów

1. Zamieńcie trójki grafu na zdania po polsku (werbalizacja): miejsce urodzenia, nagroda z rokiem, uczelnia, położenie w jednostce administracyjnej. Uwzględnijcie rodzaj gramatyczny osoby z `ex:plec` ("urodził się", "urodziła się").
2. Zakodujcie fakty i pytania modelem `paraphrase-multilingual-MiniLM-L12-v2` z normalizacją wektorów (`normalize_embeddings=True`) na CPU. Dla porównania zbudujcie TF-IDF na n-gramach znakowych (`analyzer="char_wb"`, `ngram_range=(3, 5)`).
3. Dla k = 1, 3, 5 i 10 policzcie średnie pokrycie odpowiedzi: jaka część elementów odpowiedzi wzorcowej pojawia się w k najbliższych faktach.
4. Pokażcie cztery najbliższe fakty wraz z podobieństwem dla pytań o miejsce urodzenia, o rok nagrody i o województwo.

Dlaczego dla pytania o województwo żaden pojedynczy fakt nie zawiera odpowiedzi? W czym TF-IDF na n-gramach znakowych jest dobry, a w czym gorszy od embeddingów?

## Zadanie 5: Graf wiedzy a model językowy

1. Wczytajcie `Qwen/Qwen2.5-0.5B-Instruct` (`AutoTokenizer`, `AutoModelForCausalLM`, `dtype=torch.float32`, CPU). Napiszcie funkcję `ask(question, context=None)` z komunikatem systemowym (odpowiadaj krótko po polsku, przy kontekście tylko na jego podstawie, a gdy brak odpowiedzi w kontekście, napisz "nie wiem"), szablonem czatu `apply_chat_template`, generowaniem zachłannym (`do_sample=False`, `max_new_tokens=40`) i pomiarem czasu.
2. Zadajcie wszystkie pytania z `QUESTIONS` w czterech wariantach:
   - bez kontekstu;
   - z pięcioma najbliższymi faktami z zadania 4 (prosty RAG na grafie);
   - z surowymi ścieżkami uzasadnień z zadania 3 (np. `Fritz Haber -> [wdt:P19] Wrocław -> [wdt:P131] ...`);
   - z tymi samymi ścieżkami zapisanymi jako zdania.
   Cztery warianty po dwanaście pytań to 48 generacji, lokalnie na CPU trwa to około dwóch minut. Jeśli w Colabie trwa to wyraźnie dłużej, zmniejszcie zestaw do sześciu pytań i zaznaczcie to we wnioskach.
3. Dla każdej odpowiedzi zapiszcie trafność (funkcja `score`), czy jest odmową ("nie wiem") i czas. Halucynacją nazywamy odpowiedź nietrafną, która nie jest odmową.
4. Zbudujcie tabelę zbiorczą: trafność, średnie pokrycie, liczba halucynacji i odmów, średni czas, z dodatkowym wierszem dla systemu szablonów z zadania 3. Pokażcie wszystkie odpowiedzi obok siebie.

Które odpowiedzi modelu bez kontekstu brzmią wiarygodnie, a są błędne? Dlaczego sam kontekst z grafu nie gwarantuje poprawnej odpowiedzi? Która forma kontekstu działa najlepiej i jak to wyjaśnić? Które odpowiedzi da się zweryfikować bez sięgania do zewnętrznych źródeł?

## Zadanie 6: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Dlaczego zapytanie o laureatów z województwa dolnośląskiego bez wnioskowania daje pusty wynik? Czym różni się materializacja (owlrl) od ścieżki właściwości `wdt:P19/wdt:P131*` w zapytaniu?
2. Co oznacza brak `wdt:P69` u części laureatów w świetle założenia otwartego świata i jak należy czytać wynik zapytania z `FILTER NOT EXISTS`?
3. Dlaczego precyzję i pełność ekstrakcji trzeba podawać osobno dla relacji i skąd wziął się wzrost F1 po lematyzacji?
4. Czym różni się wyszukiwanie wektorowe od zapytania SPARQL i kiedy wyszukiwanie wektorowe zawodzi?
5. Które odpowiedzi w zadaniu 5 da się zweryfikować i dlaczego kontekst z grafu nie gwarantuje poprawnej odpowiedzi modelu?

## Do oddania

Notebook `lab05_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, wykresem rozkładu stopni, tabelami z zadań 2 do 5 i krótkimi wnioskami po każdym zadaniu. Zaznaczcie we wnioskach, czy dane pochodziły z Wikidata, czy z danych zapasowych. Przed oddaniem uruchomcie notebook od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
