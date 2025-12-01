# HR_Monitor
Tytuł modelu

Wearable Bioelectric Measurement System (HR Monitor) - AADL Model

Dane studenta

Imię i Nazwisko: Wojciech Wyżgoł E-mail: wojtekwyzgol@student.agh.edu.pl

Opis modelowanego systemu

Opis ogólny: System stanowi noszony monitor funkcji życiowych (EKG), oparty na mikrokontrolerze STM32U3 (Cortex-M33) pracującym pod kontrolą systemu czasu rzeczywistego Azure RTOS (ThreadX). Głównym zadaniem systemu jest akwizycja sygnału bioelektrycznego z front-endu analogowego ADS1194, przetwarzanie cyfrowe sygnału (filtracja, detekcja zespołów QRS), zapis danych na kartę microSD (FileX) oraz transmisja wyników przez moduł Bluetooth Low Energy (BlueNRG-M0A).

Opis dla użytkownika: Urządzenie pobiera sygnał EKG z elektrod podłączonych do ciała pacjenta. Dane są pobierane z częstotliwością 500 Hz. System w czasie rzeczywistym oblicza tętno (HR). Surowe dane EKG są zapisywane na karcie pamięci w celu późniejszej analizy lekarskiej, a obliczone tętno oraz status urządzenia są wysyłane bezprzewodowo do aplikacji mobilnej.

Spis komponentów AADL z komentarzem

Devices (Urządzenia sprzętowe):

ADS1194_Sensor: Reprezentuje analogowy front-end EKG. Generuje przerwanie DRDY (Data Ready).

MicroSD_Card: Pamięć masowa do logowania danych.

BLE_Module: Moduł radiowy do transmisji danych.

Threads (Wątki oprogramowania - ThreadX):

Acquisition_Thread: Wątek o najwyższym priorytecie, wyzwalany sygnałem DRDY. Obsługuje komunikację SPI z ADS1194.

Processing_Thread: Wątek algorytmiczny. Przetwarza surowe dane, filtruje szumy i wykrywa piki R (tętno).

Storage_Thread: Wątek zapisu. Buforuje dane i zapisuje bloki na kartę SD (operacja czasochłonna).

Communication_Thread: Wątek obsługi BLE/UART. Wysyła przetworzone wyniki.

Processor:

STM32U3_MCU: Reprezentuje jednostkę obliczeniową z schedulerem wywłaszczającym (RM - Rate Monotonic / Preemptive).

Buses (Magistrale):

SPI_Bus: Współdzielona magistrala komunikacyjna łącząca procesor z ADS1194, SD i BLE.

Proponowane metody analizy modelu (dostępne w OSATE)

Flow Latency Analysis (Analiza opóźnień przepływu):

Cel: Sprawdzenie, ile czasu mija od momentu akwizycji próbki przez sensor ADS1194 do momentu wysłania jej przez BLE (ścieżka krytyczna: Sensor -> CPU -> BLE).

Wynik: Model definiuje ścieżki przepływu (flow path). Analiza wykaże sumaryczne opóźnienie (latency) uwzględniając czasy przetwarzania wątków i transmisji magistrali.

Scheduling Analysis (Analiza harmonogramowania):

Cel: Weryfikacja, czy procesor STM32U3 jest w stanie obsłużyć wszystkie wątki w wyznaczonym czasie (deadline) przy zadanym obciążeniu (Utilization).

Wynik: Sprawdzenie, czy przy częstotliwości próbkowania 500 Hz (okres 2ms) wątek akwizycji i przetwarzania nie "zagłodzą" wątków niższego priorytetu (SD/BLE).
