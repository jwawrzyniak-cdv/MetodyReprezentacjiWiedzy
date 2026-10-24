# Laboratorium 4: zadania dodatkowe

Pięć zadań alternatywnych dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne i można wykonać dowolne z nich, w dowolnej kolejności. Zadania A, D i E wymagają tylko numpy i pandas, zadanie B korzysta z sieci platformy z laboratorium, a zadanie C ze zbioru car z OpenML. Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie.

## Wspólne przygotowanie

Zadania B i C potrzebują pgmpy oraz tego samego filtra ostrzeżenia co w laboratorium:

```python
%pip install -q pgmpy

import warnings
with warnings.catch_warnings():
    warnings.filterwarnings("ignore", message=r".*pgmpy\.estimators\.StructureScore.*",
                            category=FutureWarning)
    from pgmpy.inference import VariableElimination
from pgmpy.models import DiscreteBayesianNetwork
from pgmpy.factors.discrete import TabularCPD
from pgmpy.parameter_estimator import DiscreteBayesianEstimator
from pgmpy.structure_score import BDeu

RANDOM_STATE = 42
```

W zadaniu B sieć platformy budujcie funkcją `build_model(p_tak)`, która z tych samych słowników `PARENTS` i `P_TAK` co w laboratorium tworzy model. Parametr `p_tak` pozwala potem zmieniać pojedyncze CPD w analizie czułości.

## Zadanie A: Ukryty model Markowa stanu platformy

Stan platformy (sprawna, przeciążona, awaria) zmienia się co 5 minut i nie jest obserwowany. Widać tylko czas odpowiedzi testowego zapytania: szybko, wolno albo błąd. Ukryty model Markowa jest najprostszą dynamiczną siecią bayesowską: w każdym kroku czasu jest węzeł stanu i węzeł obserwacji, a CPD są takie same w każdym kroku.

1. Przyjmijcie rozkład początkowy (0,90; 0,08; 0,02), macierz przejść o wierszach (0,90; 0,08; 0,02), (0,30; 0,60; 0,10), (0,40; 0,10; 0,50) i macierz emisji o wierszach (0,85; 0,13; 0,02), (0,20; 0,70; 0,10), (0,05; 0,25; 0,70) (wiersz to stan, kolumny to szybko, wolno, błąd). Sekwencja obserwacji: szybko, wolno, wolno, błąd, wolno, szybko, błąd, błąd.
2. Zaimplementujcie w numpy algorytmy forward i backward. Policzcie P(obserwacje) i sprawdźcie wynik sumą po wszystkich 3^8 ścieżkach stanów.
3. Policzcie rozkłady filtracji P(X_t | o_1..o_t) i wygładzania P(X_t | o_1..o_8). Narysujcie P(awaria) z obu w funkcji czasu i wyjaśnijcie kroki, w których się różnią.
4. Zaimplementujcie algorytm Viterbiego (w logarytmach). Podajcie najbardziej prawdopodobną ścieżkę i jej prawdopodobieństwo a posteriori. Czy ścieżka składa się ze stanów najbardziej prawdopodobnych w wygładzaniu?
5. Policzcie prognozę stanu na 1, 3 i 12 kroków po ostatniej obserwacji oraz rozkład stacjonarny łańcucha.

Do czego służy każde z trzech zapytań (filtracja, wygładzanie, Viterbi) w monitoringu działającym na bieżąco, a do czego w analizie po incydencie?

## Zadanie B: Diagram wpływu i wartość informacji

Sieć platformy uzupełniamy o węzeł decyzyjny (działanie dyżurnego administratora) i węzeł użyteczności. Obserwujemy Alarm = tak i Zgloszenia = tak.

1. Przyjmijcie cztery działania: czekaj (koszt 0), wycofaj aktualizację (20, usuwa przyczynę Blad), skaluj serwery (15, usuwa przyczynę Przeciazenie), wycofaj i skaluj (35, usuwa obie). Każda nieusunięta przyczyna powoduje stratę: Blad 80, Przeciazenie 60. Użyteczność to minus suma kosztu działania i strat. Wypiszcie tablicę użyteczności dla czterech działań i czterech konfiguracji (Blad, Przeciazenie).
2. Policzcie rozkład a posteriori pary (Blad, Przeciazenie) przy obserwacjach, oczekiwaną użyteczność każdego działania i decyzję o maksymalnej oczekiwanej użyteczności (MEU). Czy jest to działanie skierowane na najbardziej prawdopodobną przyczynę?
3. Wyprowadźcie analitycznie próg P(Blad | obserwacje), przy którym decyzja zmienia się między skalowaniem a obydwoma działaniami.
4. Policzcie wartość doskonałej informacji (VPI) dla obserwacji zmiennych Przeciazenie, Blad, Sesja i Aktualizacja: oczekiwaną MEU po poznaniu zmiennej minus MEU bez niej. Podajcie, jaka decyzja zapada przy każdej wartości zmiennej.
5. Zbadajcie czułość decyzji na dwa parametry, których nikt nie mierzy dokładnie: P(Blad = tak | Aktualizacja = tak) z zakresu od 0,05 do 0,9 i P(Aktualizacja = tak) z wartości 0,02, 0,1, 0,3. Pokażcie tabelę decyzji i tabelę P(Blad | obserwacje).

Którą zmienną opłaca się sprawdzić najpierw i dlaczego VPI nigdy nie jest ujemne? Jak daleko od granicy decyzyjnej leżą przyjęte parametry i co z tego wynika dla wiarygodności zaleceń?

## Zadanie C: Naiwny Bayes a sieć z uczeniem struktury

Zbiór car (OpenML 40975, `fetch_openml("car", version=3, as_frame=True)`) opisuje 1728 samochodów sześcioma atrybutami nominalnymi i oceną w czterech klasach. Zadanie jest naturalnym punktem wyjścia do projektu zespołowego, w którym sieć bayesowska uczona z danych służy jako model diagnostyczny lub decyzyjny w wybranej dziedzinie.

1. Wczytajcie dane jako napisy i zapiszcie słownik stanów każdej zmiennej (`state_names`), żeby wszystkie foldy miały te same stany.
2. Zbudujcie naiwny klasyfikator Bayesa jako `DiscreteBayesianNetwork` z krawędziami od klasy do każdego atrybutu i nauczcie parametry estymatorem `DiscreteBayesianEstimator` z BDeu.
3. Wyznaczcie strukturę sieci przeszukiwaniem hill climbing z oceną `BDeu(data, equivalent_sample_size=5, state_names=...)`. Możecie użyć deterministycznej funkcji `hill_climb` z laboratorium.
4. Przy obserwacji wszystkich atrybutów P(klasa | x) jest proporcjonalne do iloczynu wszystkich CPD sieci, więc predykcję policzcie bezpośrednio z logarytmów CPD (szybciej niż `model.predict`).
5. W pięciokrotnej stratyfikowanej walidacji krzyżowej porównajcie klasę większościową, naiwnego Bayesa i sieć z uczeniem struktury: accuracy, F1 makro, log loss i czas uczenia struktury. Podajcie liczbę parametrów obu sieci.
6. Narysujcie sieć nauczoną na całych danych, wypiszcie rodziców i koc Markowa klasy. Sprawdźcie, które krawędzie powtarzają się we wszystkich foldach w tym samym kierunku.

Które założenie naiwnego Bayesa jest w tym zbiorze naruszone? Dlaczego kierunki części krawędzi zmieniają się między foldami, mimo że wynik klasyfikacji jest stabilny?

## Zadanie D: Zbiory przybliżone Pawlaka

Teoria zbiorów przybliżonych opisuje niepewność wynikającą z nierozróżnialności obiektów, bez rozkładów prawdopodobieństwa. Pojęcia i algorytmy omawia książka A. Mrózka i L. Płonki "Analiza danych metodą zbiorów przybliżonych" (1999). Zadanie wykonajcie w czystym Pythonie (pandas do wyświetlania tabel).

1. Zbudujcie tablicę decyzyjną 16 incydentów platformy z atrybutami warunkowymi Sesja (tak, nie), Aktualizacja (tak, nie), CPU (normalne, wysokie), Odpowiedz (norma, wolno, brak), Pora (dzien, noc) i atrybutem decyzyjnym Przyczyna (brak, blad, przeciazenie). Tablica powinna zawierać co najmniej dwie pary obiektów o identycznym opisie i różnej decyzji. Tablicę użytą w rozwiązaniu referencyjnym znajdziecie w notebooku z rozwiązaniami; możecie też zdyskretyzować fragment wybranego zbioru z OpenML.
2. Napiszcie funkcję wyznaczającą klasy abstrakcji relacji nierozróżnialności IND(B) dla dowolnego podzbioru atrybutów B. Wypiszcie klasy wieloelementowe dla wszystkich atrybutów i wskażcie klasy niespójne.
3. Dla każdej decyzji wyznaczcie dolne i górne przybliżenie, obszar brzegowy oraz dokładność przybliżenia (liczność dolnego przez liczność górnego). Policzcie jakość klasyfikacji: udział obiektów w obszarze pozytywnym (sumie dolnych przybliżeń).
4. Wyznaczcie wszystkie redukty, czyli minimalne w sensie inkluzji podzbiory atrybutów zachowujące jakość klasyfikacji, przez przegląd podzbiorów. Wyznaczcie rdzeń jako przecięcie reduktów i sprawdźcie, że usunięcie każdego atrybutu rdzenia obniża jakość klasyfikacji.
5. Z najkrótszego reduktu wygenerujcie reguły decyzyjne: pewne (z klas zawartych w dolnych przybliżeniach) i możliwe (z obszaru brzegowego), z wsparciem i pewnością.
6. Policzcie dla obiektów brzegowych funkcję przynależności przybliżonej |[x] ∩ X| / |[x]|.

Czym reguła możliwa różni się od reguły z CF lub od rozkładu warunkowego w sieci bayesowskiej? Czy redukty o różnej długości są błędem? Jakie informacje o dziedzinie musiałby dostarczyć ekspert, żeby z tej samej tablicy zbudować sieć bayesowską, a jakich nie potrzeba w podejściu przybliżonym?

## Zadanie E: Teoria Dempstera-Shafera

Rama rozważań to trzy wykluczające się przyczyny awarii: B (błąd oprogramowania), P (przeciążenie), S (awaria sieci). Każde źródło przypisuje masę podzbiorom ramy.

1. Zaimplementujcie regułę łączenia Dempstera (iloczyny mas na przecięciach, normalizacja przez 1 - K, gdzie K to masa konfliktu) oraz funkcje przekonania Bel i wiarygodności Pl. Zbiory reprezentujcie jako `frozenset`.
2. Połączcie dwa źródła: monitoring z masami {P}: 0,5, {B, P}: 0,3, cała rama: 0,2 oraz logi z masami {B}: 0,6, {B, S}: 0,2, cała rama: 0,2. Podajcie K i przedziały [Bel, Pl] dla B, P, S i {B, P}.
3. Odtwórzcie przykład Zadeha: pierwszy ekspert {B}: 0,99, {P}: 0,01, drugi {S}: 0,99, {P}: 0,01. Porównajcie wynik reguły Dempstera z regułą Yagera (masa konfliktu trafia do całej ramy zamiast normalizacji) i z rachunkiem bayesowskim przy tych samych wiarygodnościach i równym rozkładzie a priori.

Co wyraża szerokość przedziału [Bel, Pl], czego nie oddaje pojedyncze prawdopodobieństwo? Czy paradoks Zadeha świadczy przeciwko regule Dempstera, czy przeciwko założeniu o niezależności źródeł?

## Do oddania

Notebook z rozwiązaniem wybranego zadania (lub kilku), z zapisanymi wynikami i wnioskami w komórkach tekstowych. Przed oddaniem uruchomcie go od początku do końca i sprawdźcie, że nie ma błędów.
