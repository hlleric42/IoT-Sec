# Detekcja anomalii w sieciach IoT (IoT-23)

![testy](https://github.com/TWOJ_LOGIN/iot-anomaly-detection/actions/workflows/testy.yml/badge.svg)

Projekt studencki: porównanie metod wykrywania ataków w ruchu sieciowym urządzeń IoT
(heurystyki, statystyka, uczenie maszynowe) z naciskiem na **wyjaśnialność** –
dlaczego detektor uznał dany przepływ za atak.

Dane: [IoT-23](https://www.stratosphereips.org/datasets-iot23) (Stratosphere Lab, CTU Praga) –
23 scenariusze, ok. 325 mln przepływów z logów Zeek (`conn.log.labeled`).

## Co już jest

| Etap | Plik | Opis |
|---|---|---|
| 1. Dane | `konwertuj.py` | Konwersja ~23 GB logów tekstowych do Parquet (czytanie kawałkami, stałe zużycie RAM, 1 plik na scenariusz) |
| 2. Heurystyka | `heurystyka.py` | 6 reguł eksperckich, ocena (precision, recall, F1, FPR, czułość wg typu ataku), powód każdego alarmu |
| Testy | `tests/` | Testy jednostkowe reguł, uruchamiane automatycznie w GitHub Actions |

W planach: metody statystyczne, modele ML (z wyjaśnieniami SHAP), porównanie kosztu obliczeniowego.

## Reguły heurystyczne

| Reguła | Co wykrywa | Warunek |
|---|---|---|
| R1 | ataki na usługi IoT | TCP na port 23, 2323, 7547, 37215, 52869, 5555 |
| R2 | skan poziomy | ≥ 25 różnych hostów na jednym porcie w 5 min (próg z `scan.zeek`) |
| R3 | skan pionowy | ≥ 15 różnych portów jednego hosta w 5 min (próg z `scan.zeek`) |
| R4 | flood / DDoS | ≥ 1000 pakietów bez odpowiedzi albo ≥ 1000 połączeń do celu w 5 min |
| R5 | beaconing C&C | prawie równe odstępy między połączeniami (CV < 0,1) |
| R6 | uporczywe próby C&C | ≥ 10 nieudanych połączeń do tego samego IP:port w 5 min |

Progi są ustalone z góry, a nie dopasowane do danych testowych.
Każdy alarm ma kolumnę `powod`, np. `R2_skan_poziomy+R1_porty_IoT`.

## Uruchomienie

```bash
git clone https://github.com/TWOJ_LOGIN/iot-anomaly-detection.git
cd iot-anomaly-detection
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# dane: pobierz wersję "lighter" IoT-23 i rozpakuj do katalogu IoTScenarios/
python konwertuj.py              # raz: logi -> dane_parquet/
python heurystyka.py             # wyniki -> wyniki_heurystyka*.csv
pytest -v                        # testy
```

Dane nie są w repozytorium (rozmiar oraz ruch prawdziwego malware) – pobiera się je ze strony autorów.

## Technologie

Python, pandas, PyArrow / Parquet, pytest, GitHub Actions (CI).

## Źródło danych

Garcia S., Parmisano A., Erquiaga M. J. (2020). *IoT-23: A labeled dataset with malicious
and benign IoT network traffic* (v1.0.0). Zenodo. https://doi.org/10.5281/zenodo.4743746
