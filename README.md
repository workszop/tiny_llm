# Jak uczy się model językowy

Lekcja w przeglądarce: pretraining, SFT i RL na małym modelu trenowanym na żywo, bez serwera.
Strona: https://workszop.github.io/tiny_llm/

## Co jest w środku

- `index.html` - cała aplikacja w jednym pliku (HTML, CSS, JS), działa też z `file://`.
- `fonts/` - czcionki Outfit, IBM Plex Sans i Roboto Mono (woff2, licencja OFL), serwowane lokalnie, bez zapytań do Google.
- `models/` (ignorowany przez git) - lokalna konwersja SmolLM2-135M-SFT-Only do ONNX, opisana niżej.

Model z lekcji to sieć Bengio i in. (2003): okno 8 tokenów, wektory słów o 24 wymiarach, 128 neuronów ukrytych, softmax na słowniku.
Korpus, demonstracje SFT i symulowani oceniający są zbudowane w kodzie z list w sekcji stałych.

## Widoki

Klawisze 1-8 przełączają główne widoki (model, pretraining, SFT, RL, porównanie, dane, prawda, quiz), 9, 0 i - strony z pojedynczym krokiem treningu, = panel SmolLM2.
Skróty jednoklawiszowe można wyłączyć w oknie pomocy (?).

Wytrenowane wagi i stan RL są zapisywane w IndexedDB przeglądarki i wracają po odświeżeniu, dopóki ziarno losowości jest to samo.
Zmiana ziarna kasuje zapis.

## Walidacja

- Pretraining: korpus powtarza linie wielokrotnie, więc podział na trening i walidację idzie po unikalnych liniach. Wszystkie kopie linii walidacyjnej są poza treningiem.
- SFT: walidacja to demonstracje dla promptów wyłączonych z SFT (`sft: false` w stałych), na których model nigdy nie trenuje.

## Testy

- `index.html?probe=1` - zestaw sprawdzeń DOM i logiki, wynik w tabeli i w `data-probe-status`.
- `index.html?probe=seeds` - pełny trening dla kilku ziaren z metrykami jakości.
- `index.html?probe=smollm` - test panelu SmolLM2 z prawdziwymi modelami.

Headless: uruchom `python3 -m http.server`, otwórz adres z parametrem w Playwright lub Chrome i odczytaj `#probe[data-probe-status]`.

## Panel SmolLM2

Trzy warianty SmolLM2-135M (base, SFT-only, Instruct) działają w przeglądarce przez transformers.js.
Base i Instruct to kopie publicznych konwersji ONNX (`onnx-community`). Wariant SFT-only nie ma publicznej konwersji, więc został przekonwertowany lokalnie do ONNX (pliki `model.onnx`, `model_int8.onnx`, `model_uint8.onnx` w układzie takim jak w `onnx-community/SmolLM2-135M-ONNX`) i wgrany na Hugging Face jako `andrzej/smollm2-135M-SFT-Only-ONNX`. Adres mirrora jest w stałej `SMOL_SFT_URL`; lokalna kopia leży w `models/`, poza gitem.

Typ danych wybrany po testach w Chrome: fp32 na WebGPU, int8 na WASM. Wariant q4 był wolny i gubił odpowiedź, fp16 psuł się na zintegrowanej karcie Intel.

## Źródła

- Bengio i in. 2003, A Neural Probabilistic Language Model, JMLR 3.
- Brown i in. 2020, Language Models are Few-Shot Learners.
- Ouyang i in. 2022, Training language models to follow instructions with human feedback.
- Bai i in. 2022, Constitutional AI.
- DeepSeek-AI 2025, DeepSeek-R1.

## Licencja

MIT (plik `LICENSE`). Czcionki w `fonts/` na licencji SIL Open Font License 1.1.
