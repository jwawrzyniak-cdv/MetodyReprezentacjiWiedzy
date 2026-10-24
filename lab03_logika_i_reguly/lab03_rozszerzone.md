# Laboratorium 3: zadania dodatkowe

Pięć zadań dla szybszych osób lub do pracy domowej. Zadania są od siebie niezależne i można je wykonywać w dowolnej kolejności. Wszystkie korzystają z kodu napisanego na zajęciach, a zadanie D dodatkowo z owlready2 i reasonera HermiT, który wymaga Javy. Rozwiązania zapisujcie w osobnym notebooku, jedna sekcja na zadanie. Zadanie E nadaje się na zalążek projektu zaliczeniowego.

## Wspólne przygotowanie

Na początek notebooka skopiujcie z rozwiązanego laboratorium komórki z Turtle uczelni, `ttl_to_facts`, unifikacją (`unify`, `subst`, `show`), regułami `RULES` i `HORN`, silnikiem w przód (`build_index`, `match_body`, `forward_chain`), stratyfikacją (`stratify`, `stratified_chain`) oraz z bazą `ES_RULES` i funkcją `es_forward`. Potem sprawdźcie, że wszystko działa:

```python
%pip install -q rdflib owlrl owlready2

RANDOM_STATE = 42

def preds(facts, pred):
    return sorted(f[1:] for f in facts if f[0] == pred)

correct = stratified_chain(RULES, FACTS)
print(len(correct), "faktów po wnioskowaniu, mozeSieZapisac:", len(preds(correct, "mozeSieZapisac")))
```

Powinno się wypisać `305 faktów po wnioskowaniu, mozeSieZapisac: 30`.

## Zadanie A: Uproszczony algorytm Rete

Silnik z zajęć po każdym nowym fakcie musiałby liczyć wszystko od nowa. W systemach produkcyjnych fakty napływają pojedynczo, więc opłaca się zapamiętywać dopasowania częściowe.

1. Przygotujcie bazę reguł `RULES_A`: R6, R7, R8 i dwie nowe reguły `powinienZnac(s, w) :- zapisanyNa(s, k), wymagaPosrednio(k, w)` oraz `prowadzacy(s, p) :- zapisanyNa(s, k), prowadzi(p, k)`. Wygenerujcie strumień 240 faktów `zapisanyNa` dla 80 nowych studentów (każdy zapisuje się na 3 losowe kursy, `np.random.default_rng(RANDOM_STATE)`).
2. Opakujcie `unify` funkcją, która zlicza wywołania. Liczba testów dopasowania będzie miarą kosztu niezależną od komputera.
3. Zaimplementujcie trzy warianty dodawania faktu: pełne przeliczenie naiwne, przyrostowe wnioskowanie półnaiwne (w każdej rundzie przynajmniej jeden literał ciała pasuje do faktu nowego) oraz sieć Rete z pamięcią alfa dla każdego warunku i pamięcią beta z dopasowaniami częściowymi dla każdego prefiksu ciała reguły.
4. Przepuśćcie strumień przez każdy wariant. Sprawdźcie, że końcowe zbiory faktów są identyczne, i zbudujcie tabelę: liczba testów, czas, liczba faktów. Podajcie też rozmiar pamięci alfa i beta w Rete.
5. Podzielcie strumień na 5 porcji po 48 faktów i dla każdej porcji policzcie średnią liczbę testów na fakt w Rete i w wariancie półnaiwnym oraz koszt jednego pełnego przeliczenia.

Jak koszt dodania faktu zależy od rozmiaru bazy w każdym wariancie? Czy wariant półnaiwny jest czuły na kolejność literałów w ciele reguły i jak to naprawić? Za co Rete płaci swoją szybkość i kiedy ten koszt przestaje się opłacać?

## Zadanie B: Datalog z rekursją i stratyfikowaną negacją

Pakiet pyDatalog nie działa na nowych wersjach Pythona, więc piszemy własny interpreter.

1. Napiszcie parser programu w składni zbliżonej do Prologa: zmienna zaczyna się wielką literą lub `_` (pojedyncze `_` to zmienna anonimowa), literał negatywny poprzedza `not`, porównanie zapisuje się `X != Y`, reguły i fakty kończą się kropką, a `%` zaczyna komentarz.
2. Sprawdzajcie bezpieczeństwo reguł: każda zmienna z głowy, z literału negatywnego (poza anonimowymi) i z porównania musi wystąpić w pozytywnym literale ciała.
3. Stratyfikujcie program, a program z cyklem przez negację odrzućcie z czytelnym komunikatem.
4. Liczcie model warstwa po warstwie metodą półnaiwną, z możliwością przełączenia na naiwną.
5. Uruchomcie program uczelni (domknięcie prerekwizytów, `brakujePrereq`, `mozeSieZapisac`, kursy końcowe, które nie są niczyim prerekwizytem, pary studentów ze wspólnym kursem) i porównajcie `mozeSieZapisac` z wynikiem z zajęć.
6. Sprawdźcie, że interpreter odrzuca program gry `wygrywa(X) :- ruch(X, Y), not wygrywa(Y).` oraz dwie reguły niebezpieczne: ze zmienną tylko w negacji i ze zmienną tylko w głowie.
7. Porównajcie liczbę testów dopasowania i czas ewaluacji naiwnej i półnaiwnej dla domknięcia przechodniego na łańcuchu kursów długości 10, 20, 40 i 60.

Dlaczego reguła ze zmienną tylko w negacji jest niebezpieczna, choć w zwykłym Pythonie dałoby się ją jakoś policzyć? Jaką semantykę należałoby przyjąć dla programu gry i co ona odpowiada dla `ruch(a, b). ruch(b, c).`? Jak zmienia się przewaga ewaluacji półnaiwnej z długością łańcucha i dlaczego?

## Zadanie C: Rezolucja w logice pierwszego rzędu

Zagadka (układ przesłanek jak w klasycznym przykładzie z podręcznika Russella i Norviga, nazwy zmienione): Każdy, kto kocha wszystkie zwierzęta, jest przez kogoś kochany. Nikt nie kocha tego, kto zabija zwierzę. Jacek kocha wszystkie zwierzęta. Kota Filemona zabił Jacek albo Ciekawość. Czy kota zabiła Ciekawość?

1. Przekształćcie przesłanki do postaci klauzulowej. Pierwsze zdanie po skolemizacji daje dwie klauzule z funkcjami Skolema `zw(x)` (zwierzę, którego x nie kocha) i `ktos(x)` (ktoś, kto kocha x). Porównajcie wynik z postacią podaną w notebooku rozwiązania.
2. Napiszcie rezolucję binarną z faktoryzacją: przemianowanie zmiennych, rezolwenty dwóch klauzul, usuwanie tautologii i klauzul już znanych (z dokładnością do nazw zmiennych).
3. Zaimplementujcie pętlę "given clause" i uruchomcie ją z negacją celu w dwóch wariantach: ze strategią zbioru wsparcia (rezolwujemy tylko z klauzulami pochodzącymi od celu) i bez niej. Policzcie wygenerowane rezolwenty.
4. Wypiszcie drzewo dowodu klauzuli pustej.
5. Zamiast konkretnego celu zadajcie pytanie "kto zabił kota?", dodając do zanegowanego celu literał odpowiedzi `odp(k)`. Wypiszcie klauzule odpowiedzi w kolejności znalezienia.

Jak strategia zbioru wsparcia zmienia liczbę rezolwent i dlaczego nie traci przy tym zupełności? Dlaczego tej zagadki nie da się rozwiązać silnikiem Horna z zajęć? Co oznacza klauzula odpowiedzi z dwoma literałami?

## Zadanie D: Reguły SWRL nad ontologią uczelni

1. Odtwórzcie ontologię uczelni w owlready2: klasy, właściwości z dziedziną i zakresem, osobników z `FACTS`, klasę `Wykladowca` zdefiniowaną jako `Pracownik & prowadzi.some(Kurs)`.
2. Dodajcie reguły SWRL dla `wymagaPosrednio` (dwie), `promotorStudenta` i `powinienZnac`, zapisane metodą `Imp().set_as_rule(...)`.
3. Sprawdźcie, czy jest Java (`shutil.which("java")`). W Colabie, gdyby jej brakowało, zainstalujcie ją: `!apt-get -qq install -y openjdk-17-jre-headless`. Uruchomcie `sync_reasoner(infer_property_values=True)` i zmierzcie czas.
4. Porównajcie wywnioskowane pary z wynikiem silnika w przód z zajęć.
5. Zdefiniujcie klasę studentów, którzy nie zaliczyli Logiki (`Student & Not(zaliczyl.value(Log))`). Kto do niej należy? Domknijcie wiedzę o zaliczeniach Huberta (`zaliczyl.only(OneOf([...]))` i `AllDifferent` dla kursów) i sprawdźcie ponownie.
6. Dodajcie regułę z wbudowanym predykatem: `Kurs(?k), ects(?k, ?e), greaterThanOrEqual(?e, 6) -> DuzyKurs(?k)` i uruchomcie reasoner.

Dlaczego klasa z punktu 5 jest pusta przed domknięciem? Czego trzeba było dodać, żeby Hubert do niej trafił, i dlaczego samo `only` nie wystarcza? Jak reasoner zareagował na regułę z punktu 6 i jak można to obejść?

## Zadanie E: Weryfikacja bazy reguł (zalążek projektu)

Duże bazy reguł psują się po cichu: reguły się powtarzają, przeczą sobie albo nigdy nie odpalają. Celem jest narzędzie, które wykrywa takie problemy automatycznie.

1. Zdefiniujcie dziedziny wartości dla 13 obserwacji z `ASKABLE` i dopiszcie do kopii `ES_RULES` pięć reguł z wadami: warunek z wartością spoza dziedziny, regułę subsumowaną przez istniejącą, regułę z tymi samymi warunkami co istniejąca i innym wnioskiem, regułę tworzącą cykl między atrybutami oraz regułę z atrybutem, którego nic nie wyprowadza.
2. Napiszcie funkcję `verify(rules, domains)`, która zwraca tabelę problemów: reguły martwe (warunek nieosiągalny z żadnych odpowiedzi), zbędne (subsumowane), sprzeczne i wyjątki (warunki jednej reguły zawierają się w warunkach drugiej, a wnioski różnią się wartością), cykle w grafie zależności atrybutów (`networkx.simple_cycles`) oraz obserwacje, o które żadna reguła nie pyta.
3. Uruchomcie ją na bazie z zajęć i na bazie z wadami.
4. Sprawdźcie pokrycie: dla wszystkich kombinacji odpowiedzi (iloczyn kartezjański dziedzin) uruchomcie `es_forward` bez zatrzymania po pierwszej diagnozie i policzcie kombinacje bez diagnozy, z jedną diagnozą i z kilkoma.

Które z wykrytych "wyjątków" w bazie z zajęć są błędami, a które zamierzonym doprecyzowaniem? Dlaczego test pokrycia na pełnym iloczynie dziedzin przesadza z liczbą przypadków niejednoznacznych i jak to poprawić? Jak rozszerzyć weryfikację na reguły ze zmiennymi, na przykład na reguły uczelni z zadania 3?

## Do oddania

Notebook z rozwiązaniami wybranych zadań (zapisane wyniki, tabele i odpowiedzi na pytania).
