#  Zautomatyzowany System AI do Generacji Notatek (LaTeX)

Bezobsługowe środowisko produkcyjne (AI Pipeline), które automatyzuje proces tworzenia zaawansowanych, sformatowanych notatek akademickich z trudnym żargonem matematyczno-fizycznym (m.in. notacja Diraca, skomplikowane równania kwantowe). Projekt łączy lokalne przetwarzanie z przetwarzaniem w chmurze.

## Cel Projektu
Rozwiązanie problemu czasochłonnej transkrypcji i ręcznego formatowania notatek ze studiów (Inżynieria Kwantowa, Finanse). System inteligentnie łączy dane z dwóch różnych źródeł (audio + obraz), optymalizuje zużycie lokalnych zasobów sprzętowych oraz skutecznie omija restrykcyjne limity darmowych API chmurowych (Rate Limits).

## Technologie i Sprzęt
* **Hardware:** NVIDIA GTX 1660 Ti (6GB VRAM)
* **Lokalne AI:** OpenAI Whisper (Large-v3) z 8-bitową kwantyzacją (klient: Buzz)
* **Chmura & LLM:** API Google Gemini 3 Flash, REST API, format JSON
* **Orkiestracja:** Make.com (iPaaS), Microsoft OneDrive
* **Wynik:** Gotowy do natychmiastowej kompilacji kod `.tex`

## Architektura Systemu

Rurociąg składa się z dwóch głównych faz działających w pełni asynchronicznie:

### Faza 1: Lokalna Transkrypcja (Edge ASR)
* **Optymalizacja sprzętowa:** Najpotężniejszy model Whisper (Large-v3) uruchomiony na zaledwie 6 GB pamięci VRAM dzięki zastosowaniu **8-bitowej kwantyzacji**, co całkowicie eliminuje wąskie gardło i odciąża główny procesor.
* **Działanie:** Zautomatyzowana detekcja plików audio (Webhook/Watch folder), sprzętowe ignorowanie szumu akustycznego i transkrypcja akademickiego żargonu do surowego pliku `.txt`.

### Faza 2: Przetwarzanie Chmurowe i LLM (Make.com)
Bezstanowa integracja dysku chmurowego z API Gemini,zabezpieczona przed przekroczeniem limitów (Rate Limiting).
* **Scheduling:** System działa w oparciu o harmonogram (uruchomienie 3 razy na dobę) i pobiera precyzyjnie limitowane pakiety (max **4 komplety plików na raz**). Tworzy to samorozładowującą się kolejkę, chroniącą twarde limity API (20 RPD) oraz pakiet operacji platformy Make.com.
* **Routing & Fallback Logic:** Inteligentne sterowanie przepływem danych. W przypadku braku transkrypcji audio, główna ścieżka fuzji danych zostaje zablokowana, a ruch trafia na zdefiniowaną **trasę awaryjną** (generacja z samego pliku PDF). Zapobiega to błędom API i nadpisywaniu plików.
* **Izolacja Kontekstu (Stateless API):** Użycie czystych żądań `HTTP POST` gwarantuje, że za każdym razem LLM uruchamia się z czystą pamięcią. Eliminuje to ryzyko halucynacji i mieszania się kontekstów między różnymi wykładami.
* **Parsowanie JSON i Ekstrakcja:** Moduł chmurowy automatycznie wyciąga czysty, wygenerowany kod `.tex` ze struktury JSON (pomijając standardowy format Markdown LLMów) i nadpisuje gotowy plik na dysku produkcyjnym.

##  Struktura Repozytorium
* `Notatki do latex.blueprint.json` - Wyeksportowany schemat architektury rurociągu (do bezpośredniego importu w środowisku Make.com).
* `PWR_Kryptografia_W1_input.pdf` - Przykładowe dane wejściowe: skan prezentacji z przedmiotu Kryptografia (Inżynieria Kwantowa) stanowiący wizualną bazę wzorów dla modelu.
* `PWR_Kryptografia_W1_input.txt` - Przykładowe dane wejściowe: surowa transkrypcja audio z tego samego wykładu, wygenerowana sprzętowo przez lokalny model Whisper Large-v3.
* `PWR_Kryptografia_W1_output.tex` - Wynikowy, zintegrowany przez chmurę kod LaTeX gotowy do bezpośredniej kompilacji.
* `PWR_Kryptografia_W1_opracowane.pdf` - Skompilowany dokument końcowy udowadniający poprawną składnię wygenerowanego kodu (poprawne renderowanie ułamków, notacji Diraca i układu równań).
