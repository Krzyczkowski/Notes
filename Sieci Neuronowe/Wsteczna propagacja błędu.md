
1. Jak działa wsteczna propagacja błędu i jakie są jej zastosowania?
2. W jaki sposób wsteczna propagacja błędu uwzględnia informacje dotyczące gradientów w celu poprawy precyzji modelu?
3. Jak wsteczna propagacja błędu wprowadza poprawki do parametrów modelu i jakie są główne trudności związane z jej implementacją?
4. Jak przygotować zestaw danych uczących do użycia w uczeniu wielowarstwowej sieci neuronowej z sigmoidalną funkcją aktywacji? Co to jest normalizacja danych w kontekście tego zagadnienia i dlaczego jest konieczna? Czy dane wejściowe zawsze trzeba normalizować? Odpowiedzi uzasadnij.
5. Proszę omówić zagadnienie przeuczenia sieci neuronowej. Jak się objawia i jak można mu przeciwdziałać?
6. Proszę omówić algorytm spadku gradientowego (ang. gradient descent): kroki algorytmu, zastosowanie, wady i zalety. Opisz krótko w jaki sposób można wykorzystać ten algorytm do uczenia wielowarstwowych sieci neuronowych.


# 3.0 Ogólnie o MLP i Wstecznej Propagacji Błędów
#### Problem "XOR" (alternatywy rozłącznej)

![[1.png]]
#### Przykładowe rozwiązanie dzięki MLP i wstecznej propagacji:

![[2.png]]

Przykładem problemu, który Wsteczna Propagacja Błędu jest w stanie rozwiązać to problem "XOR" (a ADALINE i perceptrony nie mogą, bo są modelami liniowymi) - to dzięki zastosowaniu modelu z wieloma warstwami nie-liniowymi. Najpierw (1974) powstała idea perceptronów wielowarstwowych, a potem (1986) wsteczna propagacja i miało to kluczowy wpływ na dalszy rozwój dziedziny uczenia maszynowego.

#### Inny przykład działania MLP:

![[3.png]]

#### MLP składa się z **6** głównych elementów (pierwotnie, według autorów):

1. **funkcja liniowa** agregująca wartości wejściowe
2. **funkcja sigmoidalna** zwana również funkcją aktywującą
3. **funkcja progowa** dla klasyfikacji
4. **funkcja identycznościowa** dla regresji
5. **funkcja błędu/kosztu** licząca błąd całościowy sieci
6. **funkcja ucząca** dla dostosowywania wag sieci, tutaj jest to **wsteczna propagacja błędu**




