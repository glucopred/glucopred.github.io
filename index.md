---
layout: default
title: GlucoPred
---

# O projekcie

## Osoby zaangażowane

### Wykonawcy

* Karolina Antonik
* Tomasz Hawro
* Wojciech Korczyński
* Jan Słowik
* Joanna Waczyńska

### Opiekunowie

* prof. Przemysław Kazienko
* dr Jan Kocoń
* dr n. med. Dariusz Sowiński
* dr hab. Adam Polak

## Czemu GlucoPred?

Gluco - tworzymy narzędzie związane z glukozą

Pred - narzędzie to ma służyć predykcji poziomu glukozy

## Czemu jest to ważne?

* Co 11 dorosły na świecie ma zdiagnozowaną cukrzycę
* 1 na 4 osoby powyżej 60. roku życia w Polsce ma stwierdzoną cukrzycę
* Co 6 sekund umiera osoba z powodu cukrzycy i jej konsekwencji
* Całkowita liczba osób chorujących na cukrzycę na świecie zwiększyła się ponad 3,5-krotnie w ciągu ostatnich 35 lat

## Co chcemy osiągnąć?
Chcemy na podstawie syganłów fizjologicznych zbieranych w sposób nie inwazyjny, móc dokonywać predykcji poziomu cukru we krwi. 


---
# Dane
Dane, z których korzystamy, to dane pochodzące z opaski Empatica E4 (sygnały będące wejściem do modelu uczenia maszynowego) oraz z urządzenia do ciągłego pomiaru glukozy CGM (sygnał będący wartością predykowaną).

## Empatica

* BVP - sygnał z fotopletyzmografu (PPG)
* EDA (GSR) - odpowiedź galwaniczna skóry
* Akcelerometr - trójosiowy akcelerometr
* Temperatura - temperatura mierzona w punkcie styku ze skórą

## CGM

* Poziom glukozy - mierzony w sposób ciągły (w odstępach 5-minutowych) poziom glukozy


---

# Filtracja danych
Spora część sygnałów zbieranych przy pomocy urządzenia Empatica E4 jest wątpliwej jakości. Wynika to z wielu czynników, m.in.:
* artefakty związane ze zmienną powierzchnią styku urządzenia ze skórą. Artefakty te spowodowane są wzmożoną aktywnością fizyczną podczas dnia lub ruchów w nocy.
* artefakty związane ze zmienną oleistością skóry. Artefakty te spowodowane są możliwością wystąpienia kremu lub potu na skórze.
* artefakty związane z brakiem kontaktu urządzenia ze skórą. Artefakty te spowodowane są np. zdjęciem urządzenia.

Większość z tych artefaktów można w prosty sposób wyeliminować poprzez uwzględnienie w analizie jedynie danych pochodzących z godzin nocnych. Jednakże nawet po odrzuceniu danych z dnia, sygnały nocne posiadają wiele artefaktów, które wciąż należy odfiltrować. Proces filtracji pozostałych danych można przeprowadzić na dwa sposoby, ręcznie lub automatycznie. Filtracja ręczna ma jednak wiele wad. Największą wadą jest ilość czasu potrzebna do oceny jakości tak długich sygnałów. Obarczona jest ona również obciążeniem związanym z przekonaniami osoby, która dokonywałaby oceny jakości sygnałów. Konieczne było więc stworzenie automatycznej filtracji sygnałów, która pozbawiona będzie wyżej wspomnianych wad. Stworzenie takiego narzędzie wymagało ukończenia  astępujących kroków:

1. Stworzenie narzędzia do adnotacji danych
2. Adnotacja tysiąca 10-sekundowych fragmentów sygnałów pod kątem jakości
3. Analiza sygnałów w celu znalezienia cech, które mogłyby świadczyć o jakości
4. Stworzenie zbioru danych składającego się z cech sygnałów (input) i etykiet jakości (target)
5. Wyuczenie modelu uczenia maszynowego służącego do oceny jakości sygnału

Wyuczony w ten sposób model nazwany został przez nas "PPG Quality Model".

## PPG Quality Model
Rysunek poniżej przedstawia przykładową ocenę jakości sygnału PPG. Model jakości sygnału PPG działa na zasadzie dobrania odpowiedniego progu jakości sygnału (od -1 do 1), a następnie wycięcie z sygnału fragmentów, które spełniają zadane kryterium.
![signal_quality](https://user-images.githubusercontent.com/50373360/176017841-47ebf055-9a4d-478b-8604-ea47da4e44c3.png)


# Modele predykujące poziom glukozy na podstawie sygnałów fizjologicznych

## Podejście spersonalizowane oparte o tabelaryczne cechy wydobyte z sygnałów

### Opis

Modele sztucznej inteligencji operujące na danych medycznych często sprawdzane są w zadaniu spersonalizowanym, tzn. dopasowanym pod konkretnego użytkownika. W celu sprawdzenia wpływu personalizacji modeli na uzyskane wyniki wyodrębniliśmy ze zbioru danych pojedynczą sesję nocną zawierającą dobrej jakości sygnały. Oznacza to, że nie tylko ograniczyliśmy zbiór do pojedynczej osoby, ale również do danych pochodzących z jednej sesji nocnej. Korzystając z tych danych wyekstrahowaliśmy z sygnału PPG cechy statystyczne, cechy zmienności serca (ang. Heart Rate Variability, HRV) oraz cechy eksperckie (związane z sygnałem PPG). Cechy były wydobywane dla sygnału pociętego w interwały o długości 5 minut (jest to czas pomiędzy kolejnymi próbkami poziomu glukozy). Model, z którego skorzystaliśmy, to klasyfikator LGBM. 

### Wyniki

#### Uwzględnienie personalizacji: MAE = 11.09
![personalization_features](https://user-images.githubusercontent.com/50373360/176017437-31432c0b-81e4-4740-bb03-a8c1c97bd894.png)

#### Zastosowanie analogicznej procedury wydobycia cech dla całego dostępnego zbioru: MAE = 48.31
![no_personalization_features](https://user-images.githubusercontent.com/50373360/176017443-aabe6f5c-2b5b-42cb-b4b3-0ad9d8a11c9d.png)



## Oparte o sygnały w postaci serii czasowych
TODO @WOJTEK

### Opis

### Wyniki

## Oparte o szeregi czasowe cech sygnałów 

### Opis

Z racji, że sygnały fizjologiczne są próbkowane z różną częstotliwością zdecydowaliśmy się wypróbować podejście bazujące na wyekstrachowanych cechach charakterystycznych cechach sygnałów obliczonych dla okien 1 minutowych. Aby roższerzyć zbiór dostepnych danych zdecydowaliśmy się, zwiększyć częstotliwość próbkowania wartości glukozy za pomocą interpolacji. Dla tak przygotowanych danych wytrenowaliśmy kilknanaście modeli sieci LSTM badając podejście populacyjne jak i spersonalizowane a, także eksperymentując z różnymi długościami sekwencji sygnałów.

### Wyniki

W podejsciu populacyjnym wszystkie badane modele miały tendencje zbiegania do średniej i predyckowania stałej wartości

W podejściu spersonalizowanym, zależnie od badanego pacjenta model próbował dokonywac predykcji lub zachowywał się tak jak model 

![lstm_on_45_sec](https://user-images.githubusercontent.com/50373360/176014061-b4456bb2-8f11-4c93-a0f6-c4e2e519f971.png)


Sprawdzono podejście, w którym populację zbudowano tylko z pacjentów, dla których model próbował predykować, ale niestety taka populacja również zbiegała do średniej


![omatko](https://user-images.githubusercontent.com/50373360/176014061-b4456bb2-8f11-4c93-a0f6-c4e2e519f971.png)


---
# Osiągnięcia
W trakcie pracy nad projektem wyniki naszej pracy zostały przedstawione na plakatach docenione na dwóch konferencjach naukowych.
Pierwszą z nich była **Ogólnopolska Konferencja Matematyczna OMatKo!** która się odbyła w dniach 27-28 listopada 2021 roku. 
W jej ramach zaprezentowaliśmy nasze pierwsze wyniki na plakacie pt. „Płynie w nas słodka krew. Czy AI o tym wie?”, który zdobył nagrodę za najlepszy plakat.
Drugim wydarzeniem, na którym zaprezentowaliśmy dalsze wyniki naszych prac było **Forum AI**, które miało miejsce na Politechnice Gdańskiej w dniach 20-24 maja 2022 roku. 
W wydarzeniu brali udział studenci z kierunków związanych ze sztuczną inteligencją z uczelni z całego kraju. 
W ramach Forum AI, wystąpiliśmy z plakatem, który ponownie został doceniony, gdyż została zdobył pierwsze miejsce w konkursie na najlepszy plakat.
## OMatKo!!!
![omatko](https://user-images.githubusercontent.com/50373360/176014061-b4456bb2-8f11-4c93-a0f6-c4e2e519f971.png)


## Forum AI
![forum_ai](https://user-images.githubusercontent.com/50373360/176014526-a6028867-cc56-42b8-9ba9-a1a65e49a019.png)
