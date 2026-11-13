# Przewodnik po programie

Dokument po polsku. Wyjaśnia, **czym jest ten program, jak go używać i jaka fizyka
i chemia się za nim kryje**. Napisany tak, żeby dało się go czytać bez wcześniejszej
znajomości kodu.

Pozostała dokumentacja jest po angielsku: `README.md` (opis projektu),
`docs/en/CONFERENCE_THEORY.md` (teoria i twierdzenia T1–T11), `docs/en/RUNNING.md` (komendy),
`model/README.md` (mapa katalogów modelu).

---

## Spis treści

1. [O co w tym chodzi](#1-o-co-w-tym-chodzi)
2. [Jak uruchomić](#2-jak-uruchomić)
3. [Co widzisz na ekranie](#3-co-widzisz-na-ekranie)
4. [Jak używać: przewodnik krok po kroku](#4-jak-używać-przewodnik-krok-po-kroku)
5. [Nauka i wyniki w skrócie](#5-nauka-i-wyniki-w-skrócie)
6. [Słowniczek](#6-słowniczek)

---

## 1. O co w tym chodzi

### Pytanie naukowe

**Litopanspermia** to hipoteza, że życie może przenosić się między planetami
wewnątrz odłamków skalnych wyrzuconych po uderzeniu meteorytu. Nie jest to
fantazja: na Ziemi znaleziono ponad 300 meteorytów marsjańskich. Wiemy więc,
że kamienie z Marsa **fizycznie do nas docierają**. Otwarte pytanie brzmi:
czy cokolwiek żywego mogłoby ten lot przeżyć?

Ten program odpowiada na to pytanie liczbowo.

### Analogia

Wyobraź sobie, że wrzucasz butelkę z listem do oceanu. Trzy rzeczy decydują,
czy list dotrze czytelny:

1. **Czy butelka dopłynie** (dynamika: prądy, wiatr) → tu: orbity i grawitacja
2. **Czy butelka przetrwa** (fale, kamienie) → tu: erozja pyłowa
3. **Czy atrament nie wyblaknie** (słońce, woda) → tu: promieniowanie i hydroliza

Program liczy wszystkie trzy naraz, dla czternastu „butelek" jednocześnie.

### Co konkretnie robi

```
1. Uderzenie w Marsa wyrzuca 14 odłamków skalnych (1,05 mm – 33,9 mm)
2. REBOUND całkuje ich orbity przez Układ Słoneczny
3. Po drodze każdy odłamek:
     - pochłania promieniowanie kosmiczne (dawka rośnie)
     - grzeje się i stygnie (temperatura zależy od odległości od Słońca)
     - traci warstwę powierzchni (erozja pyłowa)
4. Mikroby w środku giną: od promieniowania i od hydrolizy DNA
5. Wynik: jaki ułamek populacji przeżył
```

### Główny wynik

Odpowiedź nie jest jedną liczbą, i to jest **celowe**. Kluczowy współczynnik
`c_rad` (wrażliwość mikroba na promieniowanie) jest w literaturze znany
z dokładnością do czynnika 17. To przekłada się na **43 rzędy wielkości**
rozrzutu w wyniku końcowym.

Program pokazuje więc nie liczbę, tylko **zależność**: przesuwasz suwak
`c_rad`, a cała odpowiedź przelicza się natychmiast.

---

## 2. Jak uruchomić

### Wizualizacja (to, czego użyjesz najczęściej)

```bash
cd web
npm install        # tylko za pierwszym razem
npm run dev        # otwiera http://localhost:5173
```

### Symulacja (gdy chcesz nowe dane)

```bash
cd model
python -m microbe_radiation_model --asteroids 14 --years 3000 --dt 20 --seed 42
```

Potem przeniesienie wyniku do wizualizacji:

```bash
python tools/export_simulation_to_web.py
```

> **Uwaga.** Ta komenda **nadpisze** replay konferencyjny. Plik w `web/public/data/`
> ma 151 klatek (3000 lat) i czyta go bezpośrednio większość testów. Jest test,
> który to blokuje, ale przed prezentacją lepiej po prostu tego nie uruchamiać.

### Testy

```bash
cd web && npm test        # 666 testów
cd model && python -m unittest discover tests   # 290 testów
```

---

## 3. Co widzisz na ekranie

```
┌──────────────────────────────────────────────────────────────┐
│  WYNIK: 6,0×10⁻⁴⁶ ─────────────────────── 1,0×10⁻²          │  ← pasmo odpowiedzi
│  ostrzeżenie: to NIE jest przedział ufności                  │
├──────────────────────────────────────────────────────────────┤
│  Figures  Scene  Panels  Analysis  View    [Run] [Obj] [Anal]│  ← menu
├────────────┬─────────────────────────────┬───────────────────┤
│            │                             │                   │
│  KONSOLA   │      SCENA 3D               │   PANEL ANALIZY   │
│  URUCHOM.  │      (ciemna)               │   (wykresy)       │
│            │                             │                   │
│ parametry  │   Słońce, planety,          │   9 wykresów      │
│ modelu     │   14 odłamków               │                   │
│            │                             │                   │
├────────────┴─────────────────────────────┴───────────────────┤
│  ⏮ ‹ ▶ › ⏭   klatka 1/101   t = 0 lat   [Orbital motion]    │  ← transport
└──────────────────────────────────────────────────────────────┘
```

### Zasada projektowa

**Ciemny prostokąt to obserwacja, wszystko wokół to przyrząd.** Scena nigdy nie
jest zasłaniana przez panele. Panele stoją obok niej i każdy zajmuje najwyżej
26% szerokości okna.

### Dwa podpisy w rogach sceny

- **lewy dół:** informacja o trzech skalach rozmiarów (patrz [KONFERENCJA §17.4](KONFERENCJA.md#174-trzy-skale-rozmiarów))
- **prawy dół:** pasek koloru dawki, 0–1000 Gy

---

## 4. Jak używać: przewodnik krok po kroku

### 4.1 Pierwsze uruchomienie

Po wejściu na stronę animacja rusza sama, domyślnie z przebiegiem 100 tys. lat.
Zobaczysz Słońce, kilka planet i czternaście małych odłamków. Kolor odłamka to **pochłonięta dawka** — im
jaśniejszy, tym więcej promieniowania zebrał.

### 4.2 Sterowanie czasem (dolny pasek)

| Przycisk | Co robi |
|---|---|
| ⏮ | pierwsza klatka |
| ‹ | krok wstecz |
| ▶ | odtwarzaj / pauza |
| › | krok naprzód |
| ⏭ | ostatnia klatka |
| suwak | przewijanie |
| `step/s` | ile klatek na sekundę |
| **Orbital motion** | patrz niżej |

**„Orbital motion" — co to naprawdę jest.** Klatki są zapisywane co 20 lat,
a odłamki okrążają Słońce w 1,8–3,8 roku. Między dwiema klatkami ciało robi
więc **osiem pełnych obiegów**. Gdyby pokazać same zapisane pozycje, wszystko
teleportowałoby się losowo.

Przy włączonej opcji każde ciało jest przesuwane **po własnej elipsie**.
To nie jest zgadywanie: elipsa wynika z wektora stanu (pozycja + prędkość),
więc pozycja jest **obliczana**, nie zmyślana. Sprawdzone wobec całkowania
REBOUND: błąd 0,03 AU wobec 2,5 AU, które ciało „przeskakiwałoby" bez tego.

### 4.3 Menu (pasek górny)

| Menu | Co zawiera |
|---|---|
| **Figures** | które wykresy pokazać |
| **Scene** | warstwy: ślady odłamków, ślady planet, powłoki UV, gwiazdy Gaia, gwiezdne tło |
| **Panels** | które panele otworzyć |
| **Analysis** | osobne strony + **wybór przebiegu symulacji** |
| **View** | skala interfejsu: biurko / duży ekran / projektor / duża sala |

**Wybór przebiegu** (menu Analysis) to najważniejsza rzecz do pokazania na
konferencji — patrz §5 (trzy przebiegi).

### 4.4 Konsola uruchomieniowa (panel lewy)

Piętnaście parametrów modelu. Każdy ma znak `?` z wyjaśnieniem: co to jest,
co się stanie po zmianie, jaka jest wartość domyślna i — gdzie to istotne —
opublikowany zakres wraz ze źródłem.

Najważniejsze:

| Parametr | Znaczenie | Domyślnie |
|---|---|---|
| `q_size` | jak stromo małe odłamki przeważają nad dużymi | 2,0 |
| `radius_min` / `radius_max` | zakres rozmiarów odłamków | 0,001–5,0 m |
| `bio_fraction` | jaki ułamek masy to „ładunek" biologiczny | 0,01 |
| `asteroids` | ile odłamków wyrzucić | 25 |
| `v_min` / `v_max` | prędkości wyrzutu | 5,03–20 km/s |
| `seed` | ziarno losowania (ta sama liczba = ten sam rój) | 42 |
| `years` / `dt` | jak długo i z jakim krokiem | 2,5 / 0,025 |

### 4.5 Suwak `c_rad` — najważniejszy element

W panelu analizy jest kontrolka **radiation inactivation**. To współczynnik
mówiący, jak szybko promieniowanie zabija mikroby.

Przesuń go, a **cała odpowiedź przeliczy się natychmiast**, bez uruchamiania
symulacji. Jest to możliwe, bo funkcja przeżycia faktoryzuje się dokładnie:

```
N/N₀ = exp(−c_rad · D_skumulowana − c_hyd · H_skumulowana)
```

Dawka `D` i hydroliza `H` są zapisane w pliku, więc przeglądarka podstawia nowe
`c_rad` i liczy wynik od nowa. Sprawdziłem to numerycznie: zgodność do
**1,6×10⁻¹⁵**, czyli do precyzji maszynowej.

Są też cztery gotowe organizmy do kliknięcia:

- **D. radiodurans R1** — najodporniejszy znany organizm (2,5×10⁻⁵ 1/Gy)
- **B. subtilis, przetrwalniki** — wartość domyślna (2,5×10⁻⁴)
- **B. subtilis przy 600 g/cm²** — głęboko osłonięty (4,3×10⁻⁴)
- **ostry D10 niskiego LET** — laboratoryjny, **nie stosuje się** do promieni kosmicznych

### 4.6 Tryb prezentacji

| Klawisz | Działanie |
|---|---|
| `P` | włącz / wyłącz tryb prezentacji |
| `1`–`6` | skok do rozdziału |
| `←` `→` | poprzedni / następny rozdział |
| `Esc` | wyjście |

Sześć rozdziałów:

1. **The rock** — Mars w chwili wyrzutu
2. **The swarm** — 14 odłamków, orbity się „oddychają"
3. **Dose accumulating** — kolor to dawka
4. **Size is the story** — rozmiar decyduje o osłonie
5. **The honest answer** — suwak `c_rad` na scenie, na żywo
6. **Wait a hundred times longer** — przebieg 100 tys. lat

---

## 5. Nauka i wyniki w skrócie

Pełne wyprowadzenia, tabele, weryfikacje i uzasadnienia decyzji są w
[`KONFERENCJA.md`](KONFERENCJA.md). Poniżej tylko to, co trzeba wiedzieć, żeby
rozumieć ekran.

**Łańcuch przyczynowy.** Masa gwiazdy → jasność → strumień na powierzchni skały →
tłumienie Beer-Lamberta → dawka w rdzeniu biologicznym; równolegle temperatura →
szybkość hydrolizy DNA. Oba kanały wchodzą do funkcji przeżycia jako suma
wykładników. → [§3–4](KONFERENCJA.md#3-pełny-łańcuch-fizyczny)

**Orbity.** REBOUND, całkowanie IAS15, jednostki `(AU, rok, M☉)`; Słońce, 8 planet,
do 50 gwiazd Gaia i rój z Marsa. → [§5](KONFERENCJA.md#5-dynamika-orbitalna)

**Promieniowanie kosmiczne (GCR) to główny zabójca.** Przychodzi spoza Układu
Słonecznego, więc prawie nie zależy od odległości od Słońca (`r^+0,08` wobec
`r^−1,90` dla światła). Kalibracja: 1 jednostka modelowa = **0,194 Gy/rok**
(Mileikowsky 2000). Rozpad U/Th/K w skale to tylko 0,042% dawki.
→ [§6](KONFERENCJA.md#6-promieniowanie-trzy-kanały)

**Osłona.** Fotony giną w ~3,3 cm skały, GCR potrzebuje ~53 cm. Odłamki mają
0,4–17 g/cm², więc przepuszczają 90–99,8% — **ten rój jest praktycznie bez
osłony**. Model nie ma narastania kaskady, co w tym zakresie daje błąd do ~25%
w stronę optymistyczną. → [§7](KONFERENCJA.md#7-osłona-i-jej-granice)

**Temperatura i hydroliza.** Odłamki mają 46–260 K. Hydroliza DNA (Arrhenius,
`Ea = 130 kJ/mol`, Lindahl & Nyberg 1972) waha się między odłamkami o 119 rzędów
wielkości, ale nigdy nie przekracza ~1% tempa zabijania.
→ [§8–9](KONFERENCJA.md#8-termika)

**Funkcja przeżycia.** `N/N₀ = exp(−c_rad·D − c_hyd·H)` faktoryzuje się dokładnie
(reszta 1,6×10⁻¹⁵), dlatego suwak `c_rad` działa bez nowego biegu. `c_rad` jest
znane z dokładnością do czynnika 17 i odpowiada za 94% rozrzutu wyniku;
`c_hyd = 1200` nie ma źródła. → [§10](KONFERENCJA.md#10-biologia-funkcja-przeżycia)

**Trzy przebiegi** (menu Analysis → „simulation run") to trzy różne pytania:

| Przebieg | Klatki | Wynik |
|---|---|---|
| 3000 lat (konferencyjny) | 151 | nic nie ginie, najgorszy odłamek zachowuje 77,5% |
| 100 tys. lat (**domyślny**) | 101 | 7 z 14 zniszczonych — przez erozję, nie promieniowanie |
| 1 mln lat | 201 | 13 z 14 zniszczonych, ocalały ma przeżywalność 1,5×10⁻¹⁰ |

Strona startuje z przebiegiem 100 tys. lat, bo dopiero w nim diagram fazowy
i „same dose, different fate” mają co pokazać. Przebieg 3000 lat otwiera się
z menu Analysis albo adresem `?replay=data/cosmos_visualizer_simulation.json`.

→ [§12–14](KONFERENCJA.md#12-trzy-przebiegi-trzy-odpowiedzi)

**Pozostałe tematy w KONFERENCJI:** wszystkie wykresy (§18), trzy skale rozmiarów
w scenie (§17.4), czego model nie robi (§20), testy i niezależne weryfikacje (§21),
błędy znalezione po drodze (§22), prowenancja (§23).

---

## 6. Słowniczek

| Termin | Znaczenie |
|---|---|
| **AU** | jednostka astronomiczna, 149 597 870,7 km, średnia odległość Ziemia–Słońce |
| **Gy (grej)** | jednostka dawki pochłoniętej, 1 J na kg |
| **GCR** | galactic cosmic rays, galaktyczne promieniowanie kosmiczne |
| **HZE** | ciężkie jony o wysokiej energii, ~1% GCR, najgroźniejsze biologicznie |
| **LET** | linear energy transfer, ile energii cząstka oddaje na jednostkę drogi |
| **D10** | dawka redukująca populację dziesięciokrotnie |
| **`c_rad`** | współczynnik inaktywacji radiacyjnej, ułamek ginący na grej |
| **REBOUND** | biblioteka do całkowania problemu N ciał |
| **IAS15** | całkowanie 15. rzędu, adaptacyjny krok, bez dryfu energii |
| **elementy oskulacyjne** | elipsa, po której ciało leciałoby, gdyby zniknęły wszystkie siły poza grawitacją Słońca |
| **`β` (beta)** | stosunek ciśnienia promieniowania do grawitacji |
| **Beer-Lambert** | prawo wykładniczego tłumienia w ośrodku |
| **hydroliza** | rozpad chemiczny pod wpływem wody |
| **depurynacja** | odłączenie zasady purynowej od szkieletu DNA |
| **prowenancja** | zapis pochodzenia wyniku: ziarno, kod, wersje, współczynniki |
| **replay** | plik JSON z zapisem przebiegu, klatka po klatce |

---

## Źródła

- **Mileikowsky, C. et al. (2000)**, *Icarus* 145(2):391–427,
  doi:10.1006/icar.1999.6317 — współczynniki przeżycia, dawka GCR
- **Lindahl, T. & Nyberg, B. (1972)**, *Biochemistry* 11(19):3610–3618,
  doi:10.1021/bi00769a018 — kinetyka hydrolizy DNA
- **Cresswell, A., Carter, J. & Sanderson, D. (2018)**,
  *Radiation Measurements* 120:195–201 — dawki od U/Th/K
- **Gosse, J. & Phillips, F. (2001)**, *Quaternary Science Reviews* 20:1475 —
  długość tłumienia promieni kosmicznych
- **Belbruno, E. et al. (2012)**, *Astrobiology* 12(8):754–774 — czasy
  transferu międzygwiezdnego
- **Zeitlin, C. et al. (2013)**, *Science* 340:1080 — pomiary MSL/RAD
- **Burns, J., Lamy, P. & Soter, S. (1979)**, *Icarus* 40:1–48 — ciśnienie
  promieniowania
- **Crameri, F. et al. (2020)**, *Nature Communications* 11:5444 — paleta
  barw batlow
- **Okabe, M. & Ito, K. (2008)** — paleta bezpieczna dla daltonistów
