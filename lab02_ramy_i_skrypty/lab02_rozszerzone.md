# Laboratorium 2: zadania dodatkowe

Pięć zadań alternatywnych dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne i można wykonać dowolne z nich, w dowolnej kolejności. Zadania A i E nadają się na zalążek projektu zespołowego (A: baza wiedzy łącząca OWL z wnioskowaniem domyślnym, E: sieć powiązań budowana automatycznie z tekstów). Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie.

## Wspólne przygotowanie

```python
%pip install -q rdflib owlrl owlready2
```

Zadania C i E korzystają z funkcji `spread` z zadania 5 laboratorium, a zadanie D z klasy `Frame` z zadania 1. Skopiujcie ich działające wersje do nowego notebooka. Zadanie E pobiera teksty z polskiej Wikipedii i wymaga internetu.

## Zadanie A: Ramy w OWL i co się traci

Na zajęciach OWL pokazał, że nie radzi sobie z wyjątkiem dla pingwina. Sprawdźcie, czy da się w nim zapisać bazę ram z wartościami domyślnymi, i ustalcie, która część systemu ram ginie przy tłumaczeniu.

1. W owlready2 zdefiniujcie funkcyjną właściwość danych `limitKsiazek` oraz klasy Osoba, Student, Pracownik i StudentPracownik (podklasa obu). Wartości domyślne 3, 5 i 15 zapiszcie jako ograniczenia `limitKsiazek.value(...)` dołączone do `is_a` klas. Uruchomcie HermiT (`sync_reasoner`) i wypiszcie klasy niespełnialne (`default_world.inconsistent_classes()`). Dodajcie jedną instancję Studenta i uruchomcie reasoner ponownie. Przed użyciem HermiT sprawdźcie, czy jest Java (`shutil.which("java")`), a w Colabie w razie potrzeby zainstalujcie ją: `!apt-get -qq install -y openjdk-17-jre-headless`.
2. Tę samą ontologię zapiszcie w Turtle (ograniczenia `owl:hasValue`) z czterema osobami, po jednej na klasę, policzcie domknięcie owlrl i wypiszcie wszystkie wartości `u:limitKsiazek` każdej osoby. Czy owlrl zgłasza błąd? Skąd biorą się wartości, które nie wynikają z żadnej klasy danej osoby?
3. Zapiszcie wartości domyślne inaczej: jako trójki `u:Student u:domyslnyLimit 5` przy klasach (bez semantyki OWL). Napiszcie zapytanie SPARQL, które dla każdej osoby zwraca jej własną wartość `u:limitKsiazek`, a jeśli jej nie ma, wartość domyślną z najbardziej specyficznej klasy (`FILTER NOT EXISTS` z `rdfs:subClassOf+`). Dodajcie osobę z własną wartością.
4. Dopiszcie trójkę `u:StudentPracownik u:domyslnyLimit 10` i porównajcie wyniki zapytania przed i po. Dlaczego to zapytanie jest niemonotoniczne, choć dane się tylko powiększyły?
5. Zapiszcie demon if-needed (wiek z roku urodzenia) jako `BIND` w zapytaniu. Co z demonem if-added?

Sporządźcie tabelę: element systemu ram (wartość domyślna, wyjątek, wielokrotne dziedziczenie z konfliktem, if-needed, if-added, faseta type, faseta card) i jego odpowiednik w OWL, SPARQL lub SHACL albo brak odpowiednika.

## Zadanie B: Skrypt z rozgałęzieniami jako automat

Skrypt z zajęć jest liniowy: każdy egzamin kończy się oceną i wpisem do protokołu. W rzeczywistości student może egzaminu nie zdać albo się nie stawić.

1. Zapiszcie skrypt egzaminu jako automat skończony w `nx.DiGraph`: stany jako węzły, zdarzenia (wykonawca, czynność, obiekt) jako atrybut krawędzi. Uwzględnijcie co najmniej trzy zakończenia: egzamin zdany, egzamin niezdany z zapisem na poprawkę, nieobecność (z usprawiedliwieniem albo bez).
2. Wyliczcie wszystkie ścieżki od stanu początkowego do stanów końcowych (`nx.all_simple_paths`) i wypiszcie je jako ciągi zdarzeń.
3. Napiszcie funkcję, która dla historii znajduje ścieżki zgodne (historia jest podciągiem ścieżki) i zwraca: liczbę zgodnych ścieżek, możliwe zakończenia, zdarzenia pewne (występują na każdej zgodnej ścieżce, a nie ma ich w historii) i zdarzenia tylko możliwe.
4. Przetestujcie co najmniej cztery historie: "wszedł, odpowiedział, wyszedł", historię z oceną niedostateczną, nieobecność i historię, która nie pasuje do żadnej ścieżki.
5. Narysujcie automat.

Które wnioski skryptu liniowego przestają być pewne? Kiedy historia pozwala wywnioskować zdarzenia, które nastąpiły po jej ostatnim zdarzeniu? Jak rośnie liczba ścieżek z liczbą rozgałęzień i co zrobić, gdy automat ma cykle (kolejne terminy poprawkowe)?

## Zadanie C: Rozprzestrzenianie aktywacji a ujednoznacznianie sensu słów

Słowa wieloznaczne (zamek, pióro, klucz, rakieta) można ujednoznaczniać przez rozprzestrzenianie aktywacji: kontekst aktywuje sieć, a wygrywa ten sens, do którego dotarło najwięcej aktywacji.

1. Zbudujcie sieć, w której każdy sens jest osobnym węzłem (`zamek#budowla`, `zamek#drzwi`, `zamek#błyskawiczny`, `pióro#ptak`, `pióro#pisanie`, `klucz#drzwi`, `klucz#muzyka`, `klucz#narzędzie`, `rakieta#kosmos`, `rakieta#tenis`), połączonym z pojęciami ze swojego pola znaczeniowego, a te z dalszymi pojęciami. Około 70 do 80 krawędzi wystarczy.
2. Przygotujcie co najmniej 20 zdań testowych w postaci: słowo, lista słów kontekstu w formie podstawowej, poprawny sens. Część kontekstów powinna łączyć się z sensem tylko pośrednio (przez inny węzeł), część powinna zawierać słowa spoza sieci.
3. Porównajcie trzy metody: zawsze pierwszy sens, sąsiedztwo bezpośrednie (suma wag krawędzi od słów kontekstu do sensu) i rozprzestrzenianie aktywacji ze słów kontekstu. Podajcie trafność i tabelę z predykcjami.
4. Sprawdźcie wrażliwość na parametry: siatka decay i progu.

Na których zdaniach metody się różnią i dlaczego? Czy wysoka trafność na waszym zbiorze cokolwiek mówi o skuteczności na prawdziwych tekstach? Kto ułożył sieć, a kto zdania testowe?

## Zadanie D: Rama jako klasa Pythona, dataclass i prototyp

Python ma wbudowany system ram: klasy z atrybutami klasy i dziedziczeniem C3. JavaScript ma inny model, dziedziczenie prototypowe, w którym każdy obiekt ma jeden prototyp, a odczyt właściwości idzie po łańcuchu prototypów.

1. Zapiszcie hierarchię Osoba, Student, Pracownik, StudentPracownik jako zwykłe klasy Pythona (atrybut klasy jako wartość domyślna, `property` jako if-needed). Sprawdźcie `__mro__`, limit i parking Marka oraz diament Nixona w obu kolejnościach rodziców. Spróbujcie zdefiniować klasę z rodzicami w kolejności (Osoba, Student).
2. Zrealizujcie if-added przez `__setattr__`. Porównajcie zwykłą klasę z `@dataclass`: zmieńcie wartość domyślną trybu studiów po utworzeniu instancji i sprawdźcie, co widzi istniejąca instancja.
3. Napiszcie klasę `Proto` z jednym prototypem i wyszukiwaniem po łańcuchu (`__getattr__`), na wzór JavaScriptu. Pokażcie zmianę prototypu w czasie działania (kos staje się pingwinem). Porównajcie z `collections.ChainMap`.
4. Zmierzcie (`timeit`) czas odczytu slotu zdefiniowanego w korzeniu łańcucha o głębokości 1, 5, 10, 20 i 50 dla klasy Pythona, `Proto` i `Frame` z zajęć.
5. Zbudujcie tabelę cech: wartość domyślna, if-needed, if-added, wielokrotne dziedziczenie, zmiana rodzica w locie, fasety jako dane, możliwość inspekcji opisu slotu.

Co tracimy, a co zyskujemy, zastępując system ram klasami języka programowania? Skąd biorą się różnice w czasie odczytu?

## Zadanie E: Sieć powiązań zbudowana z korpusu

Sieć z zajęć ułożył człowiek. Sprawdźcie, ile z niej da się odtworzyć automatycznie ze współwystępowania słów w tekście.

1. Pobierzcie przez API Wikipedii (`https://pl.wikipedia.org/w/api.php`, `action=query`, `prop=extracts`, `explaintext=1`) około 15 do 20 artykułów odpowiadających pojęciom z sieci (Lekarz, Szpital, Pielęgniarka, Uczelnia, Restauracja, Autobus itd.). Ustawcie nagłówek `User-Agent` z opisem, róbcie przerwę między zapytaniami, a po odpowiedzi 429 odczekajcie tyle, ile podaje nagłówek `Retry-After`. Przygotujcie mały tekst zapasowy na wypadek braku sieci.
2. Podzielcie tekst na zdania i rozpoznajcie w nich pojęcia z sieci. Bez lematyzatora wystarczy prosta reguła: forma to temat (pojęcie albo pojęcie bez ostatniej litery) plus jedna z typowych końcówek. Wypiszcie pojęcia, których nie znaleziono.
3. Policzcie współwystępowanie par pojęć w oknie dwóch kolejnych zdań i dodatnią punktową informację wzajemną (PPMI): `PMI(a, b) = log(n(a, b) * N / (n(a) * n(b)))`, gdzie N to liczba okien. Zbudujcie graf z krawędziami o PMI > 0 i co najmniej dwóch współwystąpieniach, z wagami znormalizowanymi do przedziału (0, 1].
4. Porównajcie krawędzie z siecią ręczną: ile krawędzi jest wspólnych, jakie jest pokrycie sieci ręcznej i precyzja względem niej. Porównajcie sąsiadów pojęcia "lekarz" w obu sieciach.
5. Uruchomcie rozprzestrzenianie aktywacji w obu sieciach z kilku źródeł i porównajcie rankingi (czołówka, korelacja Spearmana).

Które braki wynikają z korpusu, a które z prymitywnej normalizacji fleksji? Czy krawędzie spoza sieci ręcznej to szum, czy powiązania, których autor sieci nie przewidział? Jak dobór artykułów wpływa na wynik?

## Do oddania

Osobny notebook z sekcjami dla wykonanych zadań. Każda sekcja kończy się krótkimi wnioskami odpowiadającymi na pytania z treści zadania. Notebook powinien przechodzić "Uruchom wszystko" bez błędów.
