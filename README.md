
# Credit Risk Data Preparation and Modeling in SAS

## Opis projektu

Celem projektu było przygotowanie kompletnego zbioru analitycznego na podstawie kilku powiązanych tabel zawierających informacje o klientach, rachunkach, kartach, kredytach, transakcjach oraz regionach.

Projekt składał się z dwóch głównych etapów:

1. przygotowania danych i inżynierii cech w języku SAS,
2. porównania modeli drzew decyzyjnych w SAS Enterprise Miner.

Najważniejszą częścią projektu było przekształcenie rozproszonych danych źródłowych w jedną tabelę modelową. Proces obejmował czyszczenie danych, zmianę formatów, agregację historii transakcji, łączenie tabel oraz tworzenie nowych zmiennych opisujących sytuację klienta w momencie udzielenia kredytu.

Modele osiągnęły umiarkowane wyniki i nie powinny być traktowane jako rozwiązanie gotowe do zastosowania w rzeczywistym procesie oceny ryzyka. Projekt koncentruje się przede wszystkim na pokazaniu pełnego procesu przygotowania danych do modelowania oraz świadomej oceny ograniczeń otrzymanych modeli.

## Cele projektu

Główne cele projektu obejmowały:

- połączenie danych pochodzących z wielu tabel,
- przygotowanie spójnej tabeli analitycznej,
- przekształcenie zmiennych tekstowych i dat,
- utworzenie zmiennej docelowej,
- agregację historii transakcji klienta,
- wykorzystanie wyłącznie informacji dostępnych przed datą kredytu,
- zaprojektowanie nowych zmiennych modelowych,
- przygotowanie danych do wykorzystania w SAS Enterprise Miner,
- porównanie kilku wariantów drzew decyzyjnych,
- ocenę modeli na podstawie wyników walidacyjnych.

## Wykorzystane technologie

- SAS
- SAS Studio
- SAS Enterprise Miner
- PROC SQL
- modelowanie predykcyjne
- drzewa decyzyjne
- analiza krzywej ROC
- przygotowanie danych
- inżynieria cech

## Rozwijane umiejętności

Projekt pozwolił wykorzystać i rozwinąć umiejętności związane z:

- analizą struktury relacyjnych zbiorów danych,
- łączeniem danych przy użyciu `LEFT JOIN`,
- tworzeniem tabel pośrednich,
- agregowaniem danych transakcyjnych,
- przetwarzaniem dat,
- kodowaniem zmiennych jakościowych,
- tworzeniem zmiennych binarnych i liczbowych,
- budowaniem cech na podstawie określonego momentu w czasie,
- ograniczaniem ryzyka wycieku informacji,
- przygotowaniem tabeli wejściowej do modelowania,
- porównywaniem modeli klasyfikacyjnych,
- interpretacją AUC, sensitivity oraz specificity,
- krytyczną oceną jakości modelu.

## Dane źródłowe

W projekcie wykorzystano kilka powiązanych tabel:

| Tabela | Zawartość |
|:---|:---|
| `account` | informacje o rachunkach |
| `card` | informacje o kartach |
| `client` | dane klientów |
| `disp` | powiązania klientów z rachunkami |
| `district` | dane dotyczące regionów |
| `loan` | informacje o kredytach |
| `order` | informacje o zleceniach |
| `trans` | historia transakcji |

Dane źródłowe nie zostały umieszczone w repozytorium ze względu na warunki wykorzystania oraz ich strukturę.


## Schemat procesu

Proces przygotowania danych przebiegał według następującego schematu:

```text
Tabele źródłowe
       ↓
Czyszczenie i standaryzacja danych
       ↓
Przetwarzanie dat i zmiennych jakościowych
       ↓
Agregacja historii transakcji
       ↓
Łączenie tabel za pomocą PROC SQL
       ↓
Utworzenie jednej tabeli analitycznej
       ↓
Inżynieria cech
       ↓
Przygotowanie danych do SAS Enterprise Miner
       ↓
Trening i porównanie drzew decyzyjnych
       ↓
Ocena wyników na zbiorze walidacyjnym
```

## Przygotowanie danych

### Czyszczenie i standaryzacja

Pierwszym etapem było przygotowanie poszczególnych tabel źródłowych. Proces obejmował między innymi:

- zmianę nazw zmiennych,
- konwersję wartości tekstowych,
- przekształcanie dat do formatu SAS,
- ujednolicenie wartości opisujących typy transakcji,
- utworzenie zmiennej określającej płeć klienta,
- przekształcenie statusu kredytu w binarną zmienną docelową,
- ujednolicenie częstotliwości obsługi rachunku.

Zmienna docelowa `default` została utworzona na podstawie statusu kredytu:

```text
default = 0  oznacza brak niewykonania zobowiązania
default = 1  oznacza wystąpienie niewykonania zobowiązania
```

## Integracja danych

Do połączenia tabel wykorzystano `PROC SQL` oraz złączenia `LEFT JOIN`.

Połączenia opierały się przede wszystkim na identyfikatorach:

- klienta,
- rachunku,
- kredytu,
- karty,
- dyspozycji,
- regionu.

Finalna tabela łączyła informacje dotyczące:

- klienta,
- posiadanego rachunku,
- historii transakcji,
- karty płatniczej,
- charakterystyki kredytu,
- danych ekonomicznych regionu.

Zastosowanie `LEFT JOIN` pozwoliło zachować rekordy kredytowe również wtedy, gdy część informacji w tabelach dodatkowych była niedostępna.

## Agregacja historii transakcji

Jednym z najważniejszych elementów projektu było przetworzenie historii transakcji klienta.

Utworzono między innymi:

- liczbę wszystkich transakcji przed udzieleniem kredytu,
- liczbę wpływów i wypływów,
- średnią wartość wpływów,
- średnią wartość wypływów,
- minimalne saldo przed kredytem,
- maksymalne saldo przed kredytem,
- zakres zmian salda,
- liczbę aktywnych dni,
- średnią liczbę transakcji dziennie,
- liczbę transakcji z ostatnich 90 dni,
- sumę wpływów i wypływów z ostatnich 90 dni,
- saldo dostępne w momencie udzielenia kredytu.

Podczas tworzenia agregatów wykorzystano wyłącznie transakcje zarejestrowane najpóźniej w dniu udzielenia kredytu:

```sas
transaction.date <= loan.date
```

Dodatkowe cechy krótkoterminowe obliczono na podstawie historii z ostatnich 90 dni:

```sas
transaction.date between loan.date - 90 and loan.date
```

Takie podejście ogranicza ryzyko wycieku informacji z przyszłości do danych treningowych.

## Inżynieria cech

Na podstawie połączonej tabeli utworzono zestaw nowych zmiennych modelowych.

### Cechy klienta

- wiek klienta w dniach,
- wiek klienta w latach,
- zakodowana płeć,
- wiek klienta w momencie otwarcia rachunku.

### Cechy rachunku

- wiek rachunku,
- częstotliwość obsługi rachunku,
- informacja, czy rachunek istnieje dłużej niż dwa lata,
- liczba transakcji przypadająca na dzień istnienia rachunku.

### Cechy karty

- informacja o posiadaniu karty,
- zakodowany typ karty,
- wiek karty,
- informacja, czy karta została wydana przed udzieleniem kredytu.

### Cechy transakcyjne

- liczba transakcji z ostatnich 90 dni,
- średni dzienny wpływ,
- relacja wpływów do wypływów,
- liczba transakcji dziennie,
- najmniejsze odnotowane saldo,
- największe odnotowane saldo,
- zakres zmian salda,
- całkowita liczba wcześniejszych transakcji,
- średnia wartość wcześniejszych wpływów.

### Cechy związane z kredytem

- relacja wartości kredytu do średniego wynagrodzenia,
- relacja miesięcznej raty do średniego wynagrodzenia,
- relacja raty do salda rachunku,
- czas trwania kredytu,
- miesiąc udzielenia kredytu,
- kwartał udzielenia kredytu,
- informacja o udzieleniu kredytu w drugim lub trzecim kwartale.

Finalny zbiór zawierał identyfikatory obserwacji, zmienną docelową oraz przygotowane zmienne rozpoczynające się od prefiksu `attr_`.

## Modelowanie w SAS Enterprise Miner

Po przygotowaniu tabeli analitycznej dane zostały wykorzystane w SAS Enterprise Miner.

Porównano trzy warianty drzew decyzyjnych. Modele różniły się konfiguracją procesu oraz wybranymi parametrami drzewa, w tym maksymalną głębokością.

Przykładowe parametry:

```text
Poziom istotności: 0.2
Maksymalne rozgałęzienie: 2
Maksymalna głębokość: 6 lub 10
Minimalna wielkość liścia: 5
Ziarno losowości: 12345
Kryterium dla zmiennych przedziałowych: ProbF
Kryterium dla zmiennych nominalnych: ProbChisq
```

## Wyniki modeli

Wyniki na zbiorze walidacyjnym przedstawiały się następująco:

| Model | AUC ROC |
|:---|---:|
| Drzewo 1 | 0.691 |
| Drzewo 2 | 0.623 |
| Drzewo 3 | **0.718** |

Najlepszy wynik osiągnęło drzewo numer 3 z wartością AUC równą `0.718`.

Wynik wskazuje na umiarkowaną zdolność modelu do rozróżniania klas. Model był lepszy od klasyfikatora losowego, ale jego jakość nie była wystarczająca do praktycznego zastosowania bez dalszego rozwoju i dodatkowej walidacji.

### Czułość i swoistość

| Model | Sensitivity | Specificity |
|:---|---:|---:|
| Drzewo 1 | 0.375 | 1.000 |
| Drzewo 2 | 0.250 | 0.994 |
| Drzewo 3 | 0.375 | 1.000 |

Wysoka wartość specificity oznacza, że modele bardzo dobrze rozpoznawały klasę negatywną.

Jednocześnie niska wartość sensitivity pokazuje, że modele miały trudność z wykrywaniem przypadków klasy dodatniej. Najlepsze warianty wykrywały jedynie `37,5%` obserwacji dodatnich.

images/roc_curve.png

## Interpretacja wyników

Otrzymane wyniki nie wskazują na model gotowy do rzeczywistego zastosowania. Szczególnie istotnym problemem jest niska czułość dla klasy dodatniej.

W kontekście projektu najważniejszy jest jednak pełny proces analityczny:

- integracja wielu tabel,
- przygotowanie danych w SAS,
- uwzględnienie kolejności czasowej,
- agregacja historii transakcji,
- zaprojektowanie cech modelowych,
- przygotowanie jednej tabeli analitycznej,
- przeniesienie danych do SAS Enterprise Miner,
- porównanie kilku modeli,
- świadoma interpretacja ich ograniczeń.

Słabsze wyniki modelowania stanowią także ważny element analizy. Pokazują, że sama poprawna implementacja modelu nie gwarantuje wysokiej jakości predykcji, a wyniki należy oceniać krytycznie przy użyciu różnych metryk.

## Ograniczenia projektu

Najważniejsze ograniczenia projektu:

- niewielka liczba obserwacji należących do klasy dodatniej,
- niska czułość modeli,
- ograniczona zdolność drzew do wykrywania przypadków niewykonania zobowiązania,
- brak pełnego strojenia hiperparametrów,
- porównanie ograniczone do drzew decyzyjnych,
- brak porównania różnych progów klasyfikacji,
- możliwy wpływ niezbalansowania klas,
- brak zewnętrznego zbioru testowego,
- część cech wymagałaby dodatkowej walidacji biznesowej.

Projekt ma charakter edukacyjny i nie powinien być wykorzystywany do podejmowania rzeczywistych decyzji kredytowych.

## Możliwe kierunki rozwoju

Projekt można rozwinąć poprzez:

- dokładniejszą analizę rozkładu zmiennej docelowej,
- zastosowanie wag klas,
- zmianę progu klasyfikacji,
- optymalizację sensitivity zamiast samej accuracy,
- strojenie hiperparametrów drzew,
- porównanie z regresją logistyczną,
- zastosowanie lasu losowego lub gradient boostingu,
- wykonanie selekcji zmiennych,
- analizę ważności cech,
- walidację krzyżową,
- analizę błędnie sklasyfikowanych obserwacji,
- zastosowanie dodatkowego zbioru testowego,
- sprawdzenie stabilności modelu w czasie.

## Autor

**Oliwier Laskowski**

- GitHub: [TUTAJ WSTAW LINK DO PROFILU]
- LinkedIn: [TUTAJ WSTAW LINK DO LINKEDIN]
- CV: [TUTAJ WSTAW LINK DO CV]
