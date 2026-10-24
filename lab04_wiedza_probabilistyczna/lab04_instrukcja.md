# Laboratorium 4: Probabilistyczna reprezentacja wiedzy

Dane: sieć bayesowska platformy e-learningowej (tablice CPD w szablonie), sieć ALARM z repozytorium bnlearn (`load_model("bnlearn/alarm")`), próbki generowane z sieci
Środowisko: Google Colab (CPU wystarczy), biblioteka pgmpy 1.1 instalowana w pierwszej komórce

## Cel

W laboratorium 3 wiedza miała postać reguł, które albo się stosują, albo nie. Teraz ta sama wiedza o awariach platformy e-learningowej zostaje zapisana jako sieć bayesowska. Zbudujecie sieć ręcznie, zadacie jej pytania w obu kierunkach (od przyczyny do skutku i od skutku do przyczyny), sprawdzicie, które niezależności wynikają z grafu, porównacie wnioskowanie dokładne z przybliżonym na większej sieci referencyjnej, nauczycie parametry i strukturę z danych, a na końcu porównacie wynik sieci ze współczynnikami pewności systemu MYCIN.

## Przed zajęciami

Przeczytajcie `lab04_wprowadzenie.md`. Otwórzcie `lab04_szablon.ipynb` w Google Colab (runtime CPU). Pierwsza komórka instaluje pgmpy (`%pip install -q pgmpy`); pozostałe biblioteki (numpy, pandas, matplotlib, networkx) są w Colabie domyślnie. Sieć ALARM pobiera się z Hugging Face Hub przy pierwszym wywołaniu `load_model`, co wymaga internetu. Komunikat o nieuwierzytelnionych zapytaniach do Hub można pominąć.

Komórka importów wycisza jedno ostrzeżenie pgmpy 1.1 o wycofaniu (`StructureScore is deprecated`), które biblioteka zgłasza przy własnych importach wewnętrznych. Używajcie aktualnych nazw: `DiscreteBayesianNetwork` (nie `BayesianNetwork`), `load_model` z `pgmpy.example_models` (nie `get_example_model`), estymatory z `pgmpy.parameter_estimator`, uczenie struktury z `pgmpy.causal_discovery`, oceny z `pgmpy.structure_score`. Stany zmiennych w sieci platformy to napisy `"nie"` i `"tak"`.

## Zadanie 1: Od reguł do niepewności

1. Szablon zawiera dziennik monitoringu platformy: liczbę godzin (spośród 2000) z danym układem zdarzeń Awaria i Alarm. Policzcie z tabeli P(Awaria), P(Awaria | Alarm), P(Alarm | Awaria) i P(nie Awaria | nie Alarm).
2. Zestawcie te liczby z regułą "JEŻELI Alarm TO Awaria" z systemu regułowego. Która z nich odpowiada regule, a która czułości monitoringu?
3. Napiszcie funkcję `posterior_positive(prior, sens, spec)`, która z reguły Bayesa liczy P(choroba | test dodatni). Dla częstości 0,001, czułości 0,99 i swoistości 0,95 (stałe w szablonie) policzcie wynik po jednym teście dodatnim i po drugim, warunkowo niezależnym teście dodatnim.
4. Sprawdźcie wynik na częstościach naturalnych: ilu chorych i ilu zdrowych na 100 000 badanych dostanie wynik dodatni?
5. Narysujcie P(choroba | test dodatni) w funkcji częstości choroby (oś logarytmiczna) dla swoistości 0,90, 0,95, 0,99 i 0,999.
6. Dla łańcucha Przeciazenie -> Awaria -> Alarm (tablice w szablonie) zbudujcie rozkład łączny jako tablicę 2 x 2 x 2 (np. `np.einsum`), wypiszcie go jako tabelę ośmiu wierszy i sprawdźcie, że sumuje się do 1.
7. Pokażcie, że Alarm zależy od Przeciazenia brzegowo, ale przy znanej Awarii jest od niego niezależny: porównajcie P(Alarm | Przeciazenie) z P(Alarm) oraz P(Alarm | Awaria, Przeciazenie) z P(Alarm | Awaria).
8. Zbudujcie tabelę liczby niezależnych parametrów rozkładu łącznego n zmiennych binarnych (2^n - 1) i górnego ograniczenia dla sieci, w której każdy węzeł ma najwyżej dwóch rodziców, dla n = 3, 7, 20, 37, 100.

Dlaczego test o czułości 99% daje tak mało trafnych wyników dodatnich? Co musiałoby się zmienić, żeby wynik dodatni był wiarygodny? Ile liczb trzeba by podać, żeby zapisać rozkład łączny siedmiu zmiennych sieci z zadania 2?

## Zadanie 2: Sieć bayesowska platformy e-learningowej

Szablon zawiera opis sieci: słownik `PARENTS` z rodzicami każdej zmiennej i słownik `P_TAK` z prawdopodobieństwami stanu `"tak"` dla kolejnych konfiguracji rodziców. Zmienne to Aktualizacja (wdrożono nową wersję), Sesja (trwa sesja egzaminacyjna), Blad (błąd oprogramowania), Przeciazenie, Awaria, Alarm (alarm monitoringu) i Zgloszenia (lawina zgłoszeń od studentów).

1. Napiszcie funkcję `make_cpd(var)`, która tworzy obiekt `TabularCPD` z argumentami `variable`, `variable_card=2`, `values` (dwa wiersze: "nie" i "tak"), `evidence`, `evidence_card` i `state_names`. Kolejność kolumn odpowiada kolejności konfiguracji rodziców w `P_TAK`, z ostatnim rodzicem zmieniającym się najszybciej.
2. Zbudujcie `DiscreteBayesianNetwork` z krawędzi wynikających z `PARENTS`, dodajcie wszystkie CPD (`add_cpds`) i wywołajcie `check_model()`. Wypiszcie CPD Awarii.
3. W kopii modelu (`model.copy()`) zepsujcie jedną kolumnę CPD Alarmu tak, żeby nie sumowała się do 1, i sprawdźcie komunikat `check_model()`.
4. Policzcie liczbę niezależnych parametrów każdej CPD i całej sieci. Porównajcie z 2^7 - 1.
5. Policzcie prawdopodobieństwo jednego pełnego stanu (np. Sesja, Przeciazenie, Awaria, Alarm i Zgloszenia na "tak", reszta na "nie") jako iloczyn wpisów CPD i porównajcie z `model.get_state_probability(...)`.
6. Narysujcie sieć (networkx, stałe pozycje węzłów) z liczbą parametrów przy każdym węźle. Wypiszcie koc Markowa Awarii (`get_markov_blanket`) i lokalne niezależności (`local_independencies`).

Skąd bierze się oszczędność parametrów? Które liczby w CPD wymagałyby konsultacji z administratorem platformy, a które można by oszacować z dziennika monitoringu?

## Zadanie 3: Wnioskowanie dokładne i d-separacja

1. Utwórzcie `VariableElimination(model)` i funkcję `p_tak(var, evidence)` zwracającą P(var = tak | evidence) (metoda `query` z `show_progress=False`).
2. Zbudujcie tabelę zapytań:
   - a priori: P(Awaria), P(Blad), P(Przeciazenie),
   - przyczynowe: P(Awaria | Aktualizacja = tak), P(Awaria | Sesja = tak),
   - diagnostyczne: P(Blad | Alarm = tak), P(Przeciazenie | Alarm = tak),
   - międzyprzyczynowe: P(Blad | Awaria = tak) oraz P(Blad | Awaria = tak, Przeciazenie = tak), P(Blad | Awaria = tak, Przeciazenie = nie), P(Blad | Awaria = tak, Sesja = tak),
   - mieszane, łączące obserwacje przyczyn i skutków.
3. Przy Alarm = tak wypiszcie rozkład łączny a posteriori pary (Sesja, Przeciazenie), wynik `map_query` dla tej pary i stany wybrane osobno z rozkładów brzegowych. Podawajcie w `map_query` jawną listę `variables`: w pgmpy 1.1 wywołanie bez tej listy ignoruje obserwacje. Dodajcie MPE, czyli `map_query` po wszystkich nieobserwowanych zmiennych.
4. Dla listy trójek (X, Y, Z) sprawdźcie `model.is_dconnected(X, Y, observed=Z)` i numerycznie największą różnicę |P(X = tak | Z) - P(X = tak | Y, Z)| po wszystkich stanach Y i Z. Uwzględnijcie co najmniej parę (Aktualizacja, Sesja) bez obserwacji oraz przy obserwowanej Awarii, Alarmie i Zgłoszeniach, trójkę (Blad, Alarm | Awaria) oraz parę (Przeciazenie, Zgloszenia) przy obserwowanej Awarii i przy obserwowanych Awarii i Sesji.
5. Wypiszcie `active_trail_nodes` dla Aktualizacji bez obserwacji i przy obserwowanym Alarmie oraz `minimal_dseparator` dla pary (Blad, Zgloszenia).

Jak zmienia się P(Blad), gdy oprócz awarii poznajemy stan przeciążenia, i dlaczego? Czy konfiguracja MAP składa się z najbardziej prawdopodobnych wartości brzegowych? Czy każda para d-odseparowana okazała się niezależna numerycznie, a każda d-połączona zależna?

## Zadanie 4: Wnioskowanie przybliżone w sieci ALARM

ALARM (Beinlich i in. 1989) to sieć do monitorowania pacjentów na oddziale intensywnej terapii, jedna ze standardowych sieci testowych.

1. Wczytajcie ją: `alarm = load_model("bnlearn/alarm")`. Podajcie liczbę węzłów i krawędzi, największą liczbę rodziców, liczbę niezależnych parametrów CPD i rząd wielkości liczby stanów rozkładu łącznego.
2. Dla obserwacji `EVIDENCE` z szablonu policzcie dokładnie P(LVFAILURE = TRUE | EVIDENCE) eliminacją zmiennych i zmierzcie czas. Policzcie też P(EVIDENCE) zapytaniem łącznym o zmienne z obserwacji.
3. Dla liczby próbek n z listy 100, 300, 1000, 3000, 10000, 30000 i pięciu ziaren (`RANDOM_STATE + r`) porównajcie dwie metody:
   - próbkowanie z odrzucaniem: `alarm.simulate(n, seed=...)`, zostawcie wiersze zgodne z `EVIDENCE` i oszacujcie P(LVFAILURE = TRUE) jako udział w zachowanych wierszach,
   - ważenie wiarygodnością: `BayesianModelSampling(alarm).likelihood_weighted_sample(evidence=[State(...)], size=n, seed=...)`, estymatorem jest średnia ważona kolumną `_weight`.

   Zapiszcie liczbę zachowanych próbek, błąd bezwzględny względem wyniku dokładnego i czas, a potem zbudujcie tabelę średnich po n.
4. Narysujcie średni błąd obu metod w funkcji n (obie osie logarytmiczne, linia odniesienia proporcjonalna do 1/pierwiastek(n)) oraz średnią liczbę próbek zachowanych przy odrzucaniu na tle n * P(EVIDENCE).

Ile próbek zostaje po odrzuceniu i jak to przekłada się na błąd? Kiedy warto sięgać po metody przybliżone, skoro eliminacja zmiennych w tej sieci trwa milisekundy?

## Zadanie 5: Uczenie parametrów i struktury

1. Dla n = 50, 200, 1000, 5000, 20000 i pięciu ziaren wygenerujcie dane `model.simulate(n, seed=...)` z sieci z zadania 2. Na prawdziwej strukturze nauczcie parametry dwoma estymatorami: `DiscreteMLE` i `DiscreteBayesianEstimator(prior_type="BDeu", equivalent_sample_size=5)`, w obu przypadkach z `state_names=model.states`.
2. Porównajcie nauczone CPD z prawdziwymi przez średnią odległość całkowitej zmienności między kolumnami CPD oraz dywergencję KL(P prawdziwy || P nauczony) rozkładów łącznych (128 stanów, zapytanie łączne o wszystkie zmienne). Policzcie, w ilu powtórzeniach KL jest nieskończona.
3. Dla jednej próbki n = 50 pokażcie tabelę liczności (Blad, Przeciazenie) x Awaria i zestawcie trzy CPD Awarii: prawdziwą, z MLE i z BDeu.
4. Prawdziwą klasę równoważności Markowa daje `model.to_pdag()`. Dla n = 500, 2000, 10000 nauczcie strukturę dwiema metodami:
   - funkcją `hill_climb(data, BIC(data))` z szablonu (zwraca DAG),
   - algorytmem PC: `PC(ci_test="chi_square", significance_level=0.01, n_jobs=1, show_progress=False).fit(data).causal_graph_` (zwraca PDAG).

   Porównajcie oba wyniki z prawdziwym CPDAG funkcją `compare_with_cpdag` (DAG zamieńcie na PDAG metodą `to_pdag()`). Dodajcie BIC nauczonej i prawdziwej struktury. PDAG z PC zamieńcie na DAG funkcją `consistent_dag`; gdy zwróci `None`, wpiszcie NaN.
5. Dla n = 10000 wypiszcie, którymi krawędziami wynik hill climbing różni się od prawdziwego CPDAG. Odwróćcie w prawdziwej sieci krawędź Aktualizacja -> Blad, a osobno Blad -> Awaria. Dla obu wariantów sprawdźcie `is_iequivalent` względem prawdziwej sieci i porównajcie BIC na tych samych danych.
6. Uruchomcie `HillClimbSearch(scoring_method="bic-d", return_type="dag", show_progress=False)` z pgmpy na tych samych danych i porównajcie wynik z funkcją `hill_climb`. Po restarcie środowiska uruchomcie tę komórkę ponownie: czy wynik jest taki sam?

Co dzieje się z MLE dla konfiguracji rodziców nieobecnych w danych? Dlaczego KL dla MLE bywa nieskończona, a dla BDeu nie? Dlaczego odwrócenie jednej krawędzi nie zmienia BIC, a drugiej tak? Czy BIC struktury nauczonej wyższy od BIC prawdziwej sieci byłby błędem?

## Zadanie 6: Współczynniki pewności MYCIN

1. Zaimplementujcie funkcje `cf_combine(a, b)` (trzy przypadki łączenia z wprowadzenia), `cf_chain(cf_rule, cf_premise)`, `cf_from_prob(post, prior)` z definicji CF przez prawdopodobieństwa i odwrotną `prob_from_cf(cf, prior)`. Sprawdźcie, że kolejność łączenia trzech CF nie zmienia wyniku.
2. Z sieci z zadania 2 wyznaczcie CF czterech reguł (prawdopodobieństwa a priori i a posteriori z funkcji `p_tak`):
   - R1: Alarm -> Awaria,
   - R2: Zgloszenia -> Awaria,
   - R3: Awaria -> Blad,
   - R4: Przeciazenie -> Blad.
3. Zbudujcie tabelę porównującą wynik z reguł (CF i odpowiadające mu prawdopodobieństwo) z wynikiem sieci dla czterech przypadków:
   - Awaria przy Alarm i Zgloszenia (R1 i R2 równolegle),
   - Blad przy Alarm (łańcuch R1, R3),
   - Blad przy Awaria i Przeciazenie (R3 i R4),
   - Awaria, gdy ten sam alarm przychodzi dwoma kanałami (R1 użyta dwukrotnie).

W którym przypadku reguły zgadzają się z siecią, a w którym nie? Dlaczego R4 ma CF równy 0? Co tracimy, a co zyskujemy, gdy każda reguła jest niezależnym modułem?

## Zadanie 7: Pytania kontrolne

Odpowiedzcie w notebooku na pytania z następnej sekcji, po jednym do trzech zdań na pytanie.

## Pytania kontrolne

1. Dlaczego reguła "JEŻELI Alarm TO Awaria" i liczba P(Awaria | Alarm) nie przenoszą tej samej wiedzy? Czego reguła nie mówi?
2. Ile niezależnych parametrów ma rozkład łączny siedmiu zmiennych binarnych, a ile sieć z zadania 2? Z czego wynika różnica?
3. Na czym polega efekt wyjaśniania (explaining away) i dlaczego obserwacja wspólnego skutku uzależnia od siebie niezależne przyczyny?
4. Dlaczego próbkowanie z odrzucaniem zawodzi przy mało prawdopodobnych obserwacjach i co zmienia ważenie wiarygodnością?
5. Dlaczego z samych danych obserwacyjnych nie da się ustalić kierunku każdej krawędzi sieci?
6. Kiedy współczynniki pewności dają wynik niezgodny z rachunkiem prawdopodobieństwa i co to mówi o modularności reguł?

## Do oddania

Notebook `lab04_szablon.ipynb` po uzupełnieniu: z zapisanymi wynikami komórek, wykresami i krótkimi wnioskami po każdym zadaniu. Przed oddaniem uruchomcie go od początku do końca (Środowisko wykonawcze, Uruchom wszystko) i sprawdźcie, że nie ma błędów.
