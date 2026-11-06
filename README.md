# Jak uczy się model językowy

Lekcja w przeglądarce: pretraining, SFT i RL w symulacji ze scenariuszami, bez serwera. Prawdziwy model jest tylko w panelu SmolLM2.
Strona: https://workszop.github.io/tiny_llm/

## Co jest w środku

- `index.html` - cała aplikacja w jednym pliku (HTML, CSS, JS), działa też z `file://`.
- `fonts/` - czcionki Outfit, IBM Plex Sans i Roboto Mono (woff2, licencja OFL), serwowane lokalnie, bez zapytań do Google.
- `models/` (ignorowany przez git) - lokalna konwersja SmolLM2-135M-SFT-Only do ONNX, opisana niżej.

Trening jest symulowany: każdy etap ma scenariusz (krzywa straty lub nagrody, próbki w trakcie treningu, zestaw typowych odpowiedzi po etapie).
Każdy etap uczy jednego pojęcia: pretraining to następne słowo (korpus nad treningiem), SFT to pytanie i odpowiedź (tabela par nad treningiem), RL to preferowana odpowiedź (pary z wyborem oceniającego nad treningiem). Po SFT odpowiedzi są poprawne, ale bywają niepełne, a na szkodliwe pytania (narkotyki, kradzież) model pomaga; RL uczy pełnych odpowiedzi i odmowy. Treść szkodliwych odpowiedzi jest w lekcji zamaskowana (███).
Widok Porównanie ma trzy prompty: „Wymień trzy owoce.”, „Jak ukraść rower?” i „Napisz jedno zdanie o psie.” (oraz własny prompt). Po pretrainingu model ciągnie dalej tekst, który zna z korpusu: kolejne pytania z arkusza lub forum albo akapit na ten temat. Nie odpowiada na pytanie.
Jeden zestaw scenariuszy (`outcomes()` w kodzie) zasila widok Porównanie, laboratorium następnego tokenu, pary RL i metryki, więc wszystkie widoki się zgadzają.
Strony „Krok” pokazują jeden krok treningu na wybranych przykładach, w trzech momentach treningu (początek, środek, koniec), z liczbami dobranymi tak, by mechanizm był widoczny.
Korpus, demonstracje SFT i prompty są prawdziwe i zbudowane w kodzie z list w sekcji stałych. Widok „Co jest prawdziwe” rozdziela to, co prawdziwe, od tego, co symulowane.


## Widoki

Klawisze 1-8 przełączają główne widoki (model, pretraining, SFT, RL, porównanie, dane, prawda, quiz), 9, 0 i - strony z pojedynczym krokiem treningu, = panel SmolLM2.
Skróty jednoklawiszowe można wyłączyć w oknie pomocy (?).

Ukończone etapy, wybór oceniającego i własne oceny par są zapisywane w localStorage i wracają po odświeżeniu. Reset etapu kasuje późniejsze etapy.

## Testy

- `index.html?probe=1` - zestaw sprawdzeń DOM i logiki symulacji (scenariusze, przebieg etapów, pauza i reset, laboratorium tokenów, strony Krok, oceniający, zapis stanu), wynik w tabeli i w `data-probe-status`.
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
