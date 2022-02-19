<style>
.ui-infobar, #doc.markdown-body { max-width: 1300px; }
</style>

Wersja online: https://hackmd.io/@Grzmot/B11V72j15

# Raport końcowy - projekt na algorytmy ewolucyjne

## Podstawowe dane

### Nazwa projektu: "Zrób sobie własną mozaikę!"
### Autor: Mateusz Orda

## Ogólny opis zagadnienia

Dla danego przez użytkownika obrazu chcemy wykonać mozaikę przybliżającą ten obraz. Dodatkowo chcemy, aby mozaika ta składała się z niewielkiej liczby różnych płytek.

Naszym celem będzie znalezienie $\textbf{palety}$, czyli niewielkiego zbioru kolorów opisującego dozwolone rodzaje płytki. Dla takiej palety mozaika powstanie przez rozmieszczenie w pewien optymalny sposób płytek (płytki danego rodzaju mogą się powtarzać wiele razy).

W części badawczej chcemy sprawdzić, czy zastosowanie strategii ewolucyjnej do rowiązania tego problemu daje jakościowe wyniki (zarówno pod względem poprawy wartości funkcji celu czy tempa zbieżności algorytmu, jak i wizualnej oceny uzyskanego wyniku).

## Bardziej ścisła definicja problemu

### Kilka definicji

Kolorem nazwiemy trójkę liczb całkokowitych $(r, g, b)$, gdzie $r, g, b \in [0, 255]$ oznaczają współrzędne RGB. Przez paletę $N$-elementową rozumieć będziemy (multi)zbiór $N$ kolorów.

Obrazkiem o wymiarach $w \times h$ nazwiemy macierz $O$ o wymiarach $w \times h$ złożoną z kolorów (w sensie powyższej definicji). Wygodnie jest myśleć o takim obrazku jak o trójwymiarowej strukturze o wymiarach $w \times h \times 3$, wtedy przez $O_{i, j, k}$ rozumieć będziemy $k$-tą współrzędną koloru znajdującego się w komórce macierzy o współrzędnych $(i, j)$. Dla danych dwóch obrazków o takich samych wymiarach definiujemy ich odległość w następujący sposób:

$|O - O'| = \sum_{i = 1}^w \sum_{j = 1}^h \sum_{k = 0}^2 (O_{i, j, k} - O'_{i, j, k})^2$

Innymi słowy, odległość to suma kwadratów różnic odpowiadających sobie wartości występujących w kolorach leżących w odpowiadających sobie polach obu macierzy.

### Sformułowanie problemu

Dany jest obrazek $O$ oraz dodatnia liczba całkowita $K$. Chcemy znaleźć taką paletę $K$-elementową $p$ (spośród wszystkich możliwych), która zminimalizuje wartość następującej funkcji celu:

$F(p) = |O - G(p)|$,

gdzie $G(p)$ to optymalna mozaika dla palety $p$.

$G$ wyznaczamy w następujący sposób:

- Wariant 1: dla danej liczby $b$ ($b$ powinno być dzielnikiem zarówno $w$ jak i $h$) dzielimy obrazek na rozłączne kwadraciki o boku $b$. Wartością $G(p)$ będzie inny obrazek, ktory powstanie z nałożenia na każdy taki kwadracik optymalnego koloru z palety (tzn. takiego, który minimalizuje odległość między oryginalnym kwadracikiem w $O$, a kwadracikiem złożonym wyłącznie z tego koloru)

- Warant 2: podobnie, tylko tym razem mamy dany na wejściu dodatkowo inny obrazek o wymiarach $b \times b$, który nazwiemy wzorem płytki. Ponownie chcemy podzielić obrazek na rozłączne kwadraciki o boku $b$, a następnie dla każdego kwadracika wybrać optymalny kolor z palety. Czynimy to w analogiczny sposób -- jedyna różnica jest taka, że zamiast kwadracika złożonego wyłącznie z danego koloru $k$, będziemy roważać wzór płytki z kolorami uśrednionymi względem $k$ (definicja uśrednienia poniżej).

### Uśrednianie

Dla danego obrazka możemy zdefiniować obrazek uśredniony względem koloru $k = (r_k, g_k, b_k)$, który powstaje przez liniowe przeskalowanie jego kolorów w taki sposób, aby zachować proporcje pomiędzy wartościami każdej z trzech współrzędnych tych kolorów, i jednocześnie aby średnie arytmetyczne tych wartości wyniosły odpowiednio $r_k, b_k, g_k$.

Może się tak zdarzyć, że po uśrednieniu niektóre współrzędne nie będą liczbami całkowitymi z przedziału $[0, 255]$. Musimy je wtedy zaokrąglić (na przykład w dół, do najbliższej pasującej liczby). Ostatecznie więc uśrednianie może nieco zaburzyć proporcje pomiędzy wartościami kolorów (ale w wielu przypadkach nie będzie to duży problem, albo wręcz będzie to zaleta).

Takie uśrednianie chcemy interpretować jako "przesunięcie" całego obrazka w stronę danego koloru, przy jednoczesnym zachowaniu struktury obrazka.

## Algorytm

### Dobór algorytmu i jego składowych

Głównym celem tego projektu jest przetestowanie zastosowania algorytmów ewolucyjnych do rozwiązania przedstawionego problemu, zatem przy wyborze algorytmu skupimy się wyłącznie na tego typu podejściach.

Początkowym wyzwaniem jest reprezentacja osobnika. Pojedyncza współrzędna koloru zapisuje się na $8$ bitach, cały kolor możemy więc zakodować na za pomocą $24$ bitów. Osobnik ma mieć zakodowane pewne $K$ kolorów, co prowadzi do pierwszego pomysłu:  chromosom będzie wektorem binarnym długości $24K$, wtedy pojedynczy gen będzie odpowiadać za pewien bit pewnej współrzędnej pewnego koloru. W takiej formie wygodnie jest użyć jakiegoś algorytmu estymowania rozkładu, na przykład PBIL.

Takie podejście wydaje się jednak nie do końca obiecujące. Jednym problemem jest mocna zależność między bitami kodującymi daną współrzędną pewnego koloru. Innym problemem są nie mniej silne zależności pomiędzy samymi kolorami. Ostatecznie taki pomysł został szybko porzucony.

W tym momencie warto spróbować poczynić kilka obserwacji specyficznych dla rozważanego problemu.

- wygodniej jest reprezentować osobnika bezpośrednio, pamiętając ciąg kolorów w oryginalnej postaci (trójki liczb); dzięki temu pozbywamy się problemu zależności bitów kodujących współrzędną koloru

- może się tak zdarzyć, że istnieje wiele rozwiązań bliskich optymalnemu, które jednak leżą stosunkowo daleko od siebie (na przykład jeden odcień niebieskiego i dwa żołtego będą tak samo dobre jak dwa żółtego i jeden niebieskiego); zamiast rozkładu prawdopodobieństwa lepiej myśleć o utrzymywaniu kolejnych populacji, wtedy unikniemy sytuacji, gdzie dalekie od siebie silne osobniki na zmianę oddalają aktualny rozkład od pozostałych silnych osobników

- z drugiej strony, skoro dwa silne osobniki mogą być silne dzięki zupełnie innym kolorom, to należałoby jakoś umieć "połączyć ich silne strony" w sytuacjach gdy to ma jednak sens

- kluczowa będzie możliwość efektywnej mutacji, zwłaszcza w początkowej fazie: chcemy znaleźć potencjalnie różne ścieżki, które generują potencjalnie podobnie silne osobniki

- z drugiej strony wraz z czasem spodziewamy się, że nasz obrazek będzie przybliżony coraz lepiej; chcielibyśmy wtedy zmniejszyć zarówno szansę, jak i agresywność mutacji (aby zmienić charakter przeszukiwania na bardziej lokalne)

Prowadzi to do pomysłu zastosowania strategii ewolucyjnej. 

Będziemy reprezentować osobnika przez ciąg kolejnych kolorów ($i$-tym genem będzie więc $i$-ty kolor, a genomem jego wartość). W tym momencie warto sobie zadać pytanie dlaczego ciąg, a nie (multi)zbiór. Oczywiście kolejność kolorów nie ma znaczenia, po prostu eksperytentalne utożsamienie ze sobą osbników różniących się jedynie permutacją posiadanych kolorów nie przyniosło istotnej poprawy.

Operator krzyżowania nie jest oczywisty. Eksperymentalnie sprawdziłem kilka pomysłów, najlepiej sprawdził się ten opisany poniżej. Istotne wydaje się, że uwzględnia on możliwość "podzielenia się" przez dwa mocne osobniki ich osobnymi mocnymi genomami.

Operator mutacji zmienia pojedyncze współrzędne kolorów o pewną wartość, przy każdym kolorze co najwyżej jedną z nich. Dzieje się to niezależnie dla każdego koloru. Jednocześnie prawdopodobieństwo i agresja mutacji dwukrotnie maleją po pewnym czasie (wszystkie progi i wartości współczynników zostały ustalone eksperymentalnie, szczegóły realizacji również poniżej). Operator ten miał początkowo uwzględniać dalej występujące pewne zależności między genami, ale prowadziło to do zbyt skomplikowanych pomysłów, i ostatecznie porzuciłem tę opcję. Być może tutaj jest pewien potencjał do próby usprawnienia algorytmu.

Sposób wyboru par rodziców również nie był tutaj dla mnie jasny, więc w podobny sposób sprawdziłem eksperymentalnie kilka opcji i wybrałem najlepszą. Podobnie z wyborem nowej populacji, tutaj jednak ekesprymenty potwierdziły pewne intiucje. Przykładowo, zostawianie wyłącznie najlepszych dzieci działało gorzej niż najepszych spośród rodziców i dzieci. Wydaje się to spodziewane -- druga opcja zabezpiecza nas przed sytuacją, w której dany rodzic sparował się wyłącznie z oddalonymi od siebie osobnikami, i jenocześnie nie wystąpiła sytuacja, w której wymiana osobnych silnych genów daje korzyści.

### Ostateczny algorytm

Algorytm użyty do ewolucji bazuje na idei $ES(\mu + \lambda)$. Standardowo przez pewną liczbę kroków będziemy ewoluować kolejne populacje, a na końcu wybierzemy najlepszego osobnika z ostatecznej populacji.

Niech $n$ oznacza rozmiar starej populacji. Nowa populacja powstaje w następujący sposób:

1. Krzyżowanie: losujemy pewną permutację $p_1, ..., p_n$. Następnie tworzymy $n$ dzieci $ch_1, ..., ch_n$ w następujący sposób: u dziecka $ch_i$ kolor $j$-ty to z kolor $j$-ty osobnika $i$ lub $p_i$ (obie opcje z prawdopodobieństwem $\frac{1}{2}$, niezależnie od innych kolorów) ze starej poulacji.

2. Mutacja: u każdego dziecka $ch_i$ każdy kolor mutujemy niezależnie od innych, z prawdopodobieństwem $p$ zmieniając jego losową współrzędną o losową liczbę z przedziału $[-a, a]$ (i zaokrąglając do najbliższej liczby całkowitej w $[0, 255]$). Wartość $p$ zależy od tego, ile iteracji wykonaliśmy (po kolejnych progach skokowo maleje), analogicznie jest z wartością $a$.

3. Aktualizacja populacji: łączymy ze sobą starą populację oraz utworzone  dzieci i z takiego multizbioru osobników wybieramy $m$ najlepszych pod względem wartości funkcji celu. Jest to nowa populacja.

## Implementacja

Raczej standardowa -- w języku Python, w postaci notebooka jupyterowego. Składa się z funkcji pomocniczych działających bezpośrednio na obrazkach, klas reprezentujących płytkę oraz osobnika, głównej funkcji realizującej powyższą strategię ewolucyjną, oraz funkcji umożliwiających testowanie i odczyt/zapis danych z/do plików.

Główne dwa notebooki (WZO-color-tiles-final-3-kot.ipynb, WZO-pic-tiles-final-3-kot_kot.ipynb) odpowiadające odpowiednio wariantom 1 i 2 (kolorowe płytki/płytki z uśrednionych zdjęć), mają ustawione przykładowe parametry startowe. Oprócz tego dołączony został notebook użyty do wygenerowania wykresów (Wykresy.ipynb).

Projekt zawiera również następujące pliki: 
- README
- raport
- pliki txt, w których zapisane zostały dane, na podstawie których wygenerowano wykresy
- pliki jpg, które zawierają użyte w eksperymentach obrazki, a także zapisane co 10 iteracji obrazki odpowiadające najsilniejszym osobnikom
- pliki ipynb, które odpowiadają kopiom głównych notebooków -- różnią się jedynie ustawianymi na końcu parametrami (powstały one już po przetestowaniu i zdebugowaniu oryginalnych notebooków, w celu łatwego uruchamiania ich równolegle; autor zdaje sobie sprawę, że nie jest to najbardziej eleganckie spośród możliwych rozwiązań)


## Część badawcza

Dane testowe składają się z trzech zdjęć:

- "kot"

![](https://i.imgur.com/yZ8UHfX.jpg)


w wariantach: kot średni ($448 \times 448$), kot mały ($28 \times 28$)

- "kwiat"

![](https://i.imgur.com/U4ijJIY.jpg)

w wariantach: kwiat średni ($504 \times 504$), kwiat mały ($28 \times 28$)

- "plaża"

![](https://i.imgur.com/ad0LHhw.jpg)


w wariantach: plaża średnia ($308 \times 224$), plaża duża ($812 \times 560$)

Wszystkie te zdjęcia pochodzą z Wikimedia Commons (a dokładniej ich odpowiedniki, które zostały przycięte oraz przeskalowane) i są udostępnione na licencji CC.

W części badawczej tego projektu skupiłem się przede wszystkim na zbadaniu $\textbf{jakości}$ wyników uzyskanych w takim podejściu na wybranych przykładach.

Pojedynczy test polegał na wybraniu wariantu, danych oraz parametrów. Następnie wykonałem pięciokrotnie opisany powyżej algorytm, za każdym razem notując fitness najlepszego osobnika po kolejnych krokach. Ostatecznie interesuje nas średnia z tych wyników, obrazują ją poniższe wykresy (w celu zwiększenia czytelności różnic pomiędzy 8 oraz 15 kolorami dane zostały pokazane w skali logarytmicznej).

Oprócz tego wykonałem jedno wywołanie, w którym co $10$ kroków zapisałem pośrednio uzyskiwane obrazki odpowiadające aktualnemu najlepszemu osobnikowi. Poza realizacją części technicznej projektu, mają one na celu umożliwić człowiekowi dodatkową (subiektywną) ocenę jakości/estetyki uzyskanych obrazków.

Parametry wspólne dla wszystkich wywołań:

Początkowa wielkość populacji $m_{init} = 30$
Wielkość populacji po kolejnych krokach $m = 15$
Liczba kroków: $301$

Parametry mutacji:

$a_1 = 50, a_2 = 20, a_3 = 10$

$K$ | $p_1$ | $p_2$ | $p_3$ |
----|-------|-------|-------|
$3$ |$0.5$  |$0.25$ |$0.1$  |
$8$ |$0.25$ |$0.125$|$0.05$ |
$15$|$0.2$  |$0.1$  |$0.04$ |

gdzie $(p, a)$ są równe odpowiednio $(p_1, a_1)$, $(p_2, a_2)$, $(p_3, a_3)$ przez pierwsze $0.2$ kroków, $0.2$ -- $0.5$ kroków, pozostałe kroki.

Powyższe parametry zostały dobrane eksperymentalnie na podstawie małej próbki, ostatecznie zdecydowałem się zostawić takie same parametry dla różnych wariantów i obrazków wejściowych, aby wygodniej było porównywać uzyskane wyniki.

Wykonałem następujące eksperymenty:

W ramach wariantu pierwszego (jedno zdjęcie, mozaika składa się z jednolitych kwadracików o danym kolorze)

obrazek       |$K$ |$b$ |
--------------|----|----|
kot średni    | 3  | 14 |
kot średni    | 8  | 14 |
kot średni    | 15 | 14 |
plaża średnia | 3  | 7  |
plaża średnia | 8  | 7  |
plaża średnia | 15 | 7  |
kwiatek średni| 3  | 28 |
kwiatek średni| 8  | 28 |
kwiatek średni| 15 | 28 |

W ramach wariantu drugiego (dwa zdjęcia -- obrazek i wzór, mozaika składa się z uśrednionych względem kolorów kopii wzoru)

obrazek       |wzór        |$K$ |
--------------|------------|----|
kot średni    |kot mały    | 3  |
kot średni    |kot mały    | 8  |
kot średni    |kot mały    | 15 |
plaża duża    |kot mały    | 3  |
plaża duża    |kot mały    | 8  |
plaża duża    |kot mały    | 15 |
kot średni    |kwiatek mały| 3  |
kot średni    |kwiatek mały| 8  |
kot średni    |kwiatek mały| 15 |
plaża duża    |kwiatek mały| 3  |
plaża duża    |kwiatek mały| 8  |
plaża duża    |kwiatek mały| 15 |

Wszystkie powyższe parametry zostały dobrane po serii ekeperymentów, ideą było z jednej strony pewne zróżnicowanie danych testowych, a z drugiej dobranie wartości dających pewien potencjał do porównania wyników w postaci obrazków.

## Wyniki i wnioski

obrazek | wykres
--------|-------
![](https://i.imgur.com/gd2SU0U.jpg) | ![](https://i.imgur.com/2SEuHdc.png)


K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/34stG1e.jpg) | ![](https://i.imgur.com/xOtpRbo.jpg)|![](https://i.imgur.com/w1lragC.jpg)



obrazek | wykres
--------|-------
![](https://i.imgur.com/1gviXS8.jpg) |![](https://i.imgur.com/KlnXn4m.png)

K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/MBMvIjm.jpg) | ![](https://i.imgur.com/HIUjZyZ.jpg) |![](https://i.imgur.com/TzL2HS0.jpg)


obrazek | wykres
--------|-------
![](https://i.imgur.com/SJIbOET.jpg) |![](https://i.imgur.com/gzlBtyk.png)


K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/olQDCL3.jpg) | ![](https://i.imgur.com/VNzfqQX.jpg) |![](https://i.imgur.com/V2v8npS.jpg)

obrazek | wzór | wykres
--------|------|-------
![](https://i.imgur.com/ESE9E4A.jpg) |![](https://i.imgur.com/9YVIHns.jpg) |![](https://i.imgur.com/gUzyk6y.png)

K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/v3hH7gB.jpg) |![](https://i.imgur.com/RSMDrUW.jpg) |![](https://i.imgur.com/KpgZDzf.jpg)


obrazek | wzór | wykres
--------|------|-------
![](https://i.imgur.com/IzyXdtS.jpg) |![](https://i.imgur.com/9YVIHns.jpg) |![](https://i.imgur.com/36DuVr9.png)



K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/SqvAnMD.jpg) |![](https://i.imgur.com/dBE605S.jpg) | ![](https://i.imgur.com/yWoMqU0.jpg)

obrazek | wzór | wykres
--------|------|-------
![](https://i.imgur.com/WQQtdOe.jpg)| ![](https://i.imgur.com/6cSrU5h.jpg)| ![](https://i.imgur.com/BZDdFkk.png)


K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/7hKlReF.jpg) |![](https://i.imgur.com/haPKiUC.jpg) |![](https://i.imgur.com/cRhd2Gs.jpg)


obrazek | wzór | wykres
--------|------|-------
![](https://i.imgur.com/CNPdScM.jpg)| ![](https://i.imgur.com/WR8NGqf.jpg)| ![](https://i.imgur.com/D7EL9fC.png)



K = 3                                |K = 8                                |K = 15
-------------------------------------|-------------------------------------|-----------
![](https://i.imgur.com/qVHJerG.jpg)|![](https://i.imgur.com/nAsQ0S9.jpg)|![](https://i.imgur.com/v1vEhkj.jpg)





W każdym z przykładów kształt wykresów jest podobny. Algorytm dosyć szybko stablizuje się (szybciej niż mogłyby sugerować momenty, w których maleją współczynniki odpowiadające za szansę i agresywność mutacji). Można dostrzec także istotną poprawę podczas początkowych iteracji. Widać wyraźny przeskok pomiędzy 3 a 8 dozwolonymi kolorami -- zarówno na wykresach, jak i ostatecznie uzyskanych mozaikach. Różnica między 8 a 15 kolorami jest bardziej subtelna, lecz również zauważalna.

Wyniki -- zarówno te matematyczne, jak i ich wizualne odpowiedniki -- wydają się sugerować, że algorytm działa w miarę sensownie (a przynajmniej zdaniem autora projektu).


## Dalsze możliwości

- zestawienie metody z innymi, porównanie wyników
- dokładniejsza analiza szerszej gamy przykładów, próba uogólnienia uzyskanych tu wniosków
- skupienie się oprócz jakości również na wydajności (pomiary czasu, dokładniejsze szacowanie tempa zbieżności w różnych przypadkach, porównanie z innymi metodami)