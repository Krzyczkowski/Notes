
## Ćwiczenie 1. przedstawienie macierzy sąsiedztwa oraz incydencji
Wyznacz macierze sąsiedztwa i incydencji dla poniższego grafu G składającego się z 4 wierzchołków A, B, C i D oraz z 5-ciu krawędzi:

e_1. A−B
e_2. 𝐴−𝐶
e_3. 𝐵−𝐶
e_4. 𝐵−𝐷
e_5. 𝐶−𝐷

![[Pasted image 20240524121733.png]]

Macierz sąsiedztwa:

|     | A   | B   | C   | D   |
| --- | --- | --- | --- | --- |
| A   | 0   | 1   | 1   | 0   |
| B   | 1   | 0   | 1   | 1   |
| C   | 1   | 1   | 0   | 1   |
| D   | 0   | 1   | 1   | 0   |

Macierz incydencji:

|     | e_1 | e_2 | e_3 | e_4 | e_5 |
| --- | --- | --- | --- | --- | --- |
| A   | 1   | 1   | 0   | 0   | 0   |
| B   | 1   | 0   | 1   | 1   | 0   |
| C   | 0   | 1   | 1   | 0   | 1   |
| D   | 0   | 0   | 0   | 1   | 1   |

# Omówienie różnic
**Macierz sąsiedztwa** jest dobra w przypadku operacji sprawdzenie istnienia krawędzi oraz gdy graf jest gęsty.
**Macierz incydencji** jest bardziej oszczędna pamięciowo dla grafów rzadkich.

## Złożoności czasowe operacji obu macierzy
Gdzie **n** to liczba wierzchołków w Grafie a **m** to liczba jego krawędzi.

| Operacja                                            | Macierz sąsiedztwa | Macierz incydencji |
| --------------------------------------------------- | ------------------ | ------------------ |
| Sprawdzanie istnienia krawędzi między wierzchołkami | **O(1)**           | **O(m)**           |
| Iterowanie po sąsiadach wierzchołka                 | **O(n)**           | **O(m)**           |
| Dodanie nowej krawędzi                              | **O(1)**           | **O(1)**           |
| Usunięcie krawędzi                                  | **O(1)**           | **O(1)**           |


##  Ćwiczenie 2. 
Jeśli graf G jest bez pętli, to co możesz powiedzieć o jego **sumie** wyrazów:
a) dowolnego wiersza lub dowolnej kolumny macierzy sąsiedztwa grafu G?
b) dowolnego wiersza macierzy incydencji G?
c) dowolnej kolumny macierzy incydencji G?

Rozwiązanie: stopień wierzchołka, stopień wierzchołka, 2