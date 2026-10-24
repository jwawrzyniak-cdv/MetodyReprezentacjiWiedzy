# Metody reprezentacji wiedzy: laboratoria w Google Colab

Materiały do laboratoriów z przedmiotu "Metody reprezentacji wiedzy": sześć laboratoriów i projekt zaliczeniowy. Notebooki uruchamiacie w Google Colab, na darmowym środowisku. Część pierwszego laboratorium robicie w przeglądarce, w edytorze ontologii WebProtégé.

## Plan zajęć

| Lab | Temat | Główne narzędzia |
|---|---|---|
| 1 | Sieci semantyczne i ontologie | WebProtégé, RDF, SPARQL, OWL, Wikidata |
| 2 | Ramy, skrypty i sieci powiązań | ramy ze slotami i demonami, skrypty Schanka, rozprzestrzenianie aktywacji |
| 3 | Logika symboliczna i systemy regułowe | unifikacja, wnioskowanie w przód i wstecz, system ekspercki |
| 4 | Probabilistyczna reprezentacja wiedzy | sieci bayesowskie, zbiory przybliżone, teoria Dempstera-Shafera |
| 5 | Grafy wiedzy w zastosowaniach | Wikidata, ekstrakcja wiedzy z tekstu, wyszukiwanie semantyczne, model językowy |
| 6 | Prywatność, odpowiedzialność, przejrzystość | k-anonimowość, kontrola dostępu, objaśnienia, audyt uczciwości, RODO, AI Act |

Wszystkie laboratoria korzystają z tej samej dziedziny: uczelni ze studentami, prowadzącymi, kursami i prerekwizytami. Ontologia zbudowana w laboratorium 1 wraca w kolejnych zajęciach.

## Zasady zaliczenia

Podstawą zaliczenia jest projekt zespołowy na temat ustalony z prowadzącym. Ocena z projektu stanowi 50% oceny z modułu "Istota sztucznej inteligencji". Projekt ma trzy punkty kontrolne: temat po laboratorium 2, stan pośredni po laboratorium 4 i wersja końcowa na ostatnich zajęciach. Wymagania, kryteria oceny i propozycje tematów są w pliku `projekt_zaliczeniowy/projekt_opis.md`, a szablon notebooka w `projekt_zaliczeniowy/projekt_szablon.ipynb`.

Zadania z laboratoriów są przygotowaniem do projektu. Niektóre zadania dodatkowe (`labNN_rozszerzone.md`) są oznaczone jako zalążek projektu i można je rozwinąć w projekt zespołu.

## Jak pracować z materiałami

Przed każdymi zajęciami przeczytajcie `labNN_wprowadzenie.md`. Na zajęciach pracujecie według `labNN_instrukcja.md` w notebooku `labNN_szablon.ipynb`. Plik `labNN_rozszerzone.md` zawiera dodatkowe zadania do wyboru.

Aby otworzyć notebook w Colabie, wybierzcie Plik, Prześlij notebook albo otwórzcie go z GitHuba. Zapiszcie własną kopię na Dysku Google, żeby nie stracić zmian. Biblioteki, których nie ma w Colabie domyślnie, instalują się komórką `%pip install` na początku notebooka. Reasoner HermiT wymaga Javy, a notebooki, które go używają, mają komórkę sprawdzającą Javę i instalującą ją w razie potrzeby.

Laboratoria 1 i 5 pobierają dane z Wikidata, a laboratorium 5 także modele z Hugging Face, więc potrzebne jest połączenie z internetem. Gdy Wikidata nie odpowiada, notebook użyje danych zapasowych zapisanych w szablonie.

Przed pierwszymi zajęciami załóżcie bezpłatne konto w WebProtégé: https://webprotege.stanford.edu. Rejestracja wymaga potwierdzenia adresu e-mail.

## Literatura

Literatura podstawowa:

1. S. J. Russell, P. Norvig, Artificial Intelligence: A Modern Approach, Pearson Education Limited, Londyn 2016.
2. N. Buduma, N. Locascio, Fundamentals of Deep Learning: Designing Next-Generation Machine Intelligence Algorithms, O'Reilly Media, 2017.
3. L. Rutkowski, Metody i techniki sztucznej inteligencji, PWN, Warszawa 2005.
4. A. Mrózek, L. Płonka, Analiza danych metodą zbiorów przybliżonych, Akademicka Oficyna PLJ, Kraków 1999.

Literatura uzupełniająca:

1. I. Goodfellow, Y. Bengio, A. Courville, Deep Learning, MIT Press, Cambridge 2016.
2. B. Ramsundar, R. B. Zadeh, TensorFlow for Deep Learning: From Linear Regression to Reinforcement Learning, O'Reilly Media, 2018.
3. A. Géron, Hands-On Machine Learning with Scikit-Learn and TensorFlow: Concepts, Tools, and Techniques to Build Intelligent Systems, O'Reilly Media, 2017.

Źródła do poszczególnych tematów są wymienione na końcu plików `labNN_wprowadzenie.md`.
