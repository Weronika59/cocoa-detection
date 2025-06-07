# Cocoa Project

Projekt został wykonany w ramach inicjatywy Studenckiego Koła Naukowego ATLAS. 
Problematyka projektu obejmuje detekcję orzechów oraz drzew kakaowca w celu estymacji plonów. 

Detekcja ta może stanowić wsparcie rolników w:
- szacowaniu zbiorów (dojrzałe i niedojrzałe owoce) w danym sezonie na dużym obszarze,
- lepszej prognozie ilości plonów (zautomatyzowanie procesu liczenia owoców),
- estymacji ponoszonych strat (określanie liczby zepsutych owoców),
- poprawie jakości upraw (informacje o liczbie występujących zepsutych owoców mogą stanowić podstawę do wdrożenia dodatkowych środków ochrony roślin),
- lepszej logistyce przewozu zbiorów (poznanie planowanej wielkości zbiorów pozwala podjąć lepszą decyzję nt. przygotowania floty transportowej).

## Zbiór danych
Zbiór danych wykorzystany w projekcie został pobrany ze strony Huggingface: https://huggingface.co/datasets/KaraAgroAI/Drone-based-Agricultural-Dataset-for-Crop-Yield-Estimation.
Składa się on ze zdjęć wykonanych w Ghanie (Afryka) przez zespół KaraAgro AI oraz za pomocą dronów. 
Rozdzielczość zdjęć to 16000 x 13000 px.

Zbiór obejmuje 5 klas: 
- cocoa-pod-immature (małe niedojrzałe orzechy kakaowca), 
- cocoa-pod-mature-unripe (duże niedojrzałe orzechy kakaowca), 
- cocoa-pod-riped (dojrzałe orzechy kakaowca), 
- cocoa-pod-spoilt (zepsute orzechy kakaowca), 
- cocoa-tree (drzewa kakaowca).

## Przygotowanie danych (Preprocessing)
Zbiór został zweryfikowany i doopisany ręcznie przy użyciu platformy Roboflow. Zweryfikowano m.in. poprawność etykiet klas przypisanych do istniejących Bounding-Boxów (BB), wielkość BB (1 BB powinien odpowiadać 1 rozpoznawanemu obiektowi) oraz jakość zdjęć. 
Przetworzony zbiór zawiera łącznie 4061 zdjęć podzielonych na zbiór treningowy, walidacyjny oraz testowy. Jest on dostępny do pobrania w formacie zip z Dysku Google pod linkiem: https://drive.google.com/file/d/1HvsWypW6k_v_yFN-9bGc0SNudhxcstAk/view?usp=sharing.

## Wybór modelu YOLO
Do przeprowadzenia detekcji został wybrany model YOLOv8m dostępny na stronie Ultralytics: https://docs.ultralytics.com/models/yolov8/#performance-metrics.
Jest to model najbardziej optymalny ze wszystkich dostępnych pod względem balansu pomiędzy szybkością uczenia a uzyskiwaną metryką dokładności detekcji mAP50-95 na zbiorze walidacyjnym.

## Przepływ pracy (Workflow)
Kolejne etapy pracy nad projektem obejmowały:
1. Trening
2. Tuning hiperparametrów
3. Walidację
4. Trening
5. Walidacja (Ewaluacja wyników)
6. Estymację

### Tuning
Do tuningu zostały wybrane hiperparametry odpowiadające m.in. za szybkość uczenia (np. lr0 - początkowa szybkość uczenia) oraz augumentację pomocną przy detekcji kakaowców różniących się między sobą kolorem (np. hsv_h - odcienie kolorów, hsv_s - saturacja, hsv_v - jasność).

W trakcie prowadzenia tuningu w /runs/detect automatycznie tworzone są foldery train, w których podczas kolejnych iteracji tuningu zapisywane są najlepsze wagi, ostatnie wagi, a także wykresy z walidacji (tu foldery train - train8). Przykładowo, aby uzyskać dostęp do wag z 8 iteracji tuningu należy wybrać: /runs/detect/train8/weights/best.pt. 

Najlepsze wagi uzyskane po tuningu znajdują się w: /runs/detect/tuning1/weights/best.pt

### Trening po tuningu
Po zakończeniu tuningu został przeprowadzony trening na najlepszych uzyskanych wagach. 
Automatycznie zapisywane wyniki ewaluacji tego modelu znajdują się w folderze /runs/detect/train22.

### Ewaluacja
Wszystkie wyniki ewaluacji uzyskane podczas procesu doboru najlepszego modelu znajdują się w folderach val-val7. Zobaczyć tam można m.in. macierze pomyłek oraz krzywe F1-Confidence, Precision-Confidence, Recall-Confidence i Precision-Recall. Wykresy te są otrzymywane w wyniku walidacji sprawdzanego modelu na zbiorze walidacyjnym (nie testowym). 
Folder val7 zawiera wykresy i rysunki uzyskane dla najlepszego modelu. Pokrywają się one z uzyskanymi automatycznie wykresami w folderze train22.
W folderze val4 znajdują się wyniki ewaluacji najlepszego modelu na zbiorze testowym.

Uzyskane wykresy oraz macierze pomyłek świadczą o dobrym dopasowaniu modelu do danych. 
Warto zauważyć, że metryki mAP (mean Average Precision) uzyskane na zbiorze testowym są lepsze od metryk uzyskanych na zbiorze walidacyjnym.

### Estymacja
W folderze /runs/predict znajdują się wyniki predykcji przeprowadzonej na zdjęciach ze zbioru testowego. 
Zliczanie BB każdej z klas po predykcji przeprowadzone zostało na końcu pliku tuning.ipynb. 

Od strony biznesowej warto zaznaczyć, że wg [Horticulture :: Plantation Crops :: Cocoa](https://agritech.tnau.ac.in/horticulture/horti_plantation%20crops_cocoa.html) w Ghanie na 1 kg sprzedawanych suchych ziaren kakaowca przypada ok. 30-36 orzechów. Wg innej strony (https://cargohandbook.com/index.php/Cocoa_Beans) orzech kakaowca zawiera zazwyczaj 20-50 nasion, a 880 wysuszonych nasion waży 1 kg. Dlatego też 1 kg suchych nasion można otrzymać z ok. 18-44 orzechów (880/20 = 44, 880/50=17.6), co daje średnio (18+44)/2 = 31 orzechów/kg. Ta sama strona podaje też, że waga suchych nasion kakaowca otrzymanych z 1 orzecha wynosi średnio 40 g - stąd średnio 1000 g / 40 g = 25 orzechów jest potrzebnych do uzyskania 1 kg suchych nasion.
Zatem na potrzeby estymacji dla uproszczenia zakładamy, że 1 kg suchych nasion kakaowca otrzymamy z ok. 30 orzechów.

Po zsumowaniu liczby orzechów niedojrzałych i dojrzałych wykrytych na zdjęciach ze zbioru testowego oraz podzieleniu ich przez średnią liczbę orzechów na kilogram otrzymaliśmy, że zebralibyśmy łącznie ok. 65,8 kg suchych nasion. Wg Business Insider na stan z 06.06.2025 r. 1 kg suchych orzechów kakaowca kosztuje 6,60 USD (https://markets.businessinsider.com/commodities/cocoa-price), co przy obecnym kursie 3,76 zł/USD daje nam ok. 1633 zł.

## Dalsze kierunki rozwoju
W każdym projekcie znajduje się przestrzeń na wprowadzenie ulepszeń. W tym projekcie można rozważyć takie działania jak:
- zmniejszenie underfittingu poprzez przeprowadzenie treningu na większej liczbie epok lub poprzez powiększenie zbioru danych o nowe zdjęcia z poprawnymi opisami,
- dokonanie upsamplingu klas mniejszościowych i zobaczenie, jak to wpłynęłoby na model,
- przeprowadzenie tuningu na większej liczbie epok i iteracji, aby móc sprawdzić i odnaleźć jeszcze lepsze wartości hiperparametrów,
- przeprowadzenie tuningu z wykorzystaniem hiperparametrów poprzednio nie uwzględnianych, aby odnaleźć ich najlepszą kombinację,
- przeprowadzenie głębszego tuningu (na większej liczbie epok) na zawężonych przedziałach wartości dla hiperparametrów,
- zastosowanie walidacji krzyżowej, żeby uzyskiwane w wynikach metryki były bardziej wiarygodne i stabilne,
- rozszerzenie projektu na inne wersje modelu YOLO (m.in. YOLOv11, najnowsze YOLOv12) i porównanie uzyskiwanych wyników.

Należy jednak zaznaczyć, że do przeprowadzenia tych kroków potrzeba byłoby sporo mocy obliczeniowej, ponieważ ze względu na specyfikę problemu uczenie modelu do wykrywania orzechów kakaowca zajmuje dużo czasu.
