# 1. Tytuł modelu
**Wearable Bioelectric Measurement System (HR Monitor) - AADL Model**

---

# 2. Dane studenta
* **Imię i Nazwisko:** Wojciech Wyżgoł
* **E-mail:** wojtekwyzgol@student.agh.edu.pl

---

# 3. Opis modelowanego systemu

### A. Opis ogólny
Projekt przedstawia model architektury systemu wbudowanego typu *wearable* (urządzenie noszone) przeznaczonego do ciągłego monitorowania sygnału EKG. System bazuje na energooszczędnym mikrokontrolerze **STM32U3** (rdzeń Cortex-M33) oraz systemie operacyjnym czasu rzeczywistego **Azure RTOS (ThreadX)**. Warstwa sprzętowa obejmuje precyzyjny front-end analogowy ADS1194, moduł komunikacji Bluetooth Low Energy (BlueNRG-M0A) oraz pamięć masową microSD. Model AADL uwzględnia aspekty fizyczne (waga, pobór mocy) oraz czasowe (opóźnienia, harmonogramowanie) niezbędne do walidacji projektu przed etapem prototypowania.

### B. Opis dla użytkownika
Urządzenie jest osobistym monitorem pracy serca. Użytkownik zakłada urządzenie (elektrody) na klatkę piersiową. System automatycznie:
1.  Pobiera sygnał elektryczny serca z wysoką częstotliwością (500 Hz).
2.  Przetwarza sygnał w czasie rzeczywistym, wyznaczając tętno (HR).
3.  Zapisuje pełny przebieg EKG na karcie pamięci w celu późniejszej analizy medycznej.
4.  Wysyła bieżący puls bezprzewodowo (Bluetooth) do aplikacji na telefonie użytkownika.

---

# 4. Spis komponentów AADL z komentarzem

Model wykorzystuje standardowe komponenty AADL wzbogacone o właściwości `properties` w celu przeprowadzenia analiz fizycznych.

### A. Sprzęt (Hardware & Devices)

| Komponent | Typ AADL | Opis i Funkcja | Właściwości fizyczne |
| :--- | :--- | :--- | :--- |
| **STM32U3** | Processor | Główna jednostka obliczeniowa (MCU). Zarządza magistralą i wykonuje wątki. | Waga: 1g, Pobór mocy: 50mW |
| **ADS1194_Sensor** | Device | Front-end analogowy EKG. Źródło danych (Flow Source). | Waga: 5g, Pobór mocy: 15mW |
| **MicroSD_Card** | Device | Pamięć masowa do logowania danych. | Waga: 2g, Pobór mocy: 100mW |
| **BLE_Module** | Device | Moduł radiowy BlueNRG-M0A. Ujście danych (Flow Sink). | Waga: 3g, Pobór mocy: 20mW |
| **Battery** | Device | Bateria LiPo 500mAh. Źródło zasilania. | **Waga: 25g**, Pojemność: 2.5W |
| **Enclosure_PCB** | Device | Obudowa urządzenia i laminat. Element pasywny (masa). | **Waga: 40g** |
| **SRAM / Flash** | Memory | Pamięci operacyjna i programu. | 256KB RAM / 2MB Flash |

### B. Oprogramowanie (Software - Threads)

| Wątek | Priorytet | Opis zadania | Rozmiar kodu |
| :--- | :--- | :--- | :--- |
| **Acquisition_Thread** | 10 (Max) | Obsługa przerwań od sensora i magistrali SPI. Krytyczny czasowo. | 4 KB |
| **Processing_Thread** | 8 | Cyfrowa obróbka sygnału (filtracja, detekcja QRS). | 16 KB |
| **Communication_Thread** | 6 | Obsługa stosu BLE i wysyłanie wyników. | 8 KB |
| **Storage_Thread** | 5 (Min) | Obsługa systemu plików FileX i zapis na kartę SD. | 32 KB |

---

# 5. Model - rysunek

*(miejsce na zdjęcie)*

![Diagram Systemu AADL](ścieżka_do_obrazka/diagram_modelu.png)

---

# 6. Przeprowadzenie Analizy (OSATE)

Poniżej przedstawiono wyniki analiz przeprowadzonych przy użyciu wtyczek analitycznych środowiska OSATE na podstawie zdefiniowanych właściwości (`properties`).

### A. Analiza Wagowa (Weight / Mass Analysis)
* **Cel:** Weryfikacja, czy masa urządzenia nie przekracza założonego limitu.
* **Założony limit:** 0.100 kg (100g).
* **Wynik:** Całkowita masa systemu wynosi **0.076 kg (76g)**.
* **Wniosek:** Urządzenie spełnia wymagania wagowe z zapasem 24g. Najcięższymi elementami są obudowa i bateria.

### B. Analiza Energetyczna (Electrical Power Analysis)
* **Cel:** Oszacowanie maksymalnego poboru mocy w trybie aktywnym.
* **Wynik:** Sumaryczne zapotrzebowanie na moc wynosi **0.186 W** (186 mW).
* **Wniosek:** Przy baterii o pojemności ok. 1.85 Wh (3.7V * 500mAh), urządzenie może pracować teoretycznie przez ok. 10 godzin w trybie pełnego obciążenia.

### C. Analiza Opóźnień (Flow Latency Analysis)
* **Cel:** Sprawdzenie czasu latencji "End-to-End" dla krytycznej ścieżki danych (od pobrania próbki EKG do wysłania informacji o tętnie).
* **Ścieżka:** `ADS1194 -> STM32 (Acquisition -> Processing -> Communication) -> BLE Module`.
* **Wynik:** Całkowite opóźnienie mieści się w przedziale **8ms - 20ms**.
* **Wniosek:** System spełnia wymagania czasu rzeczywistego dla monitoringu pacjenta (opóźnienie jest niezauważalne dla użytkownika).

### D. Analiza Budżetu Zasobów (Resource Budget / Memory)
* **Cel:** Weryfikacja, czy oprogramowanie zmieści się w pamięci mikrokontrolera.
* **Zasoby:** Flash: 2 MB, RAM: 256 KB.
* **Wymagania:** Kod: ~60 KB, Stos: ~12 KB.
* **Wniosek:** Wykorzystanie pamięci jest na niskim poziomie (<10%), co pozwala na dalszą rozbudowę funkcjonalności w przyszłości.

---

# 7. Inne informacje zależne od tematu
Projekt kładzie szczególny nacisk na optymalizację wagi i rozmiarów, co jest kluczowe dla urządzeń medycznych noszonych przez pacjenta. Wykorzystanie systemu operacyjnego ThreadX pozwala na łatwą skalowalność projektu i separację zadań krytycznych (akwizycja EKG) od zadań tła (zapis na kartę SD).

---

# 8. Literatura
1.  **SAE AS5506C:** Architecture Analysis & Design Language (AADL) Standard.
2.  **OSATE Documentation:** Open Source AADL Tool Environment User Guide.
3.  **STMicroelectronics:** STM32U3 Series Datasheet & Reference Manual.
4.  **Texas Instruments:** ADS1194 Datasheet (Low-Power, 4-Channel, 16-Bit Analog Front-End for Biopotential Measurements).
5.  **Microsoft:** Azure RTOS ThreadX & FileX User Guides.
