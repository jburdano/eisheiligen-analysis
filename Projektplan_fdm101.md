Perfekt! Das ist eine sehr gute Ausgangslage. Hier ist dein konkretes Rezept für ein **FAIR, reproduzierbares Miniprojekt mit steiler Lernkurve**:

## Das Projekt-Setup Rezept

| **Phase** | **Dauer** | **Outputs** | **Tools** |
|-----------|----------|-----------|---------|
| **1. Struktur & Setup** | 2–3 h | Repo, Projektstruktur, Environment, GitHub Projects | git, uv, GitHub |
| **2. Daten-Pipeline & EDA** | 6–8 h | `data/`, Notebooks, erste Visualisierungen | pandas, seaborn, Jupyter |
| **3. Analyse & Methodik** | 6–8 h | Eisheiligen-Metriken, Tests, Interpretation | numpy, scipy optional |
| **4. Report & Dokumentation** | 4–6 h | README, Datendokumentation, Report (Quarto/HTML) | Markdown, Quarto |
| **5. Finalisierung & Veröffentlichung** | 2–4 h | GitHub Actions optional, LICENSE, Clean-Up | git, GitHub |

---

## Phase 1: Struktur & Setup (2–3 h)

### Schritt 1.1: GitHub-Repo erstellen & clonen

```bash
# Auf GitHub.com:
# - Neues Repo: "eisheiligen-analysis"
# - Public, README + .gitignore (Python) gleich hinzufügen
# - Lizenz: CC-BY-4.0 oder MIT (für Code+Daten)

# Lokal:
git clone https://github.com/deinname/eisheiligen-analysis.git
cd eisheiligen-analysis
```

### Schritt 1.2: Projektstruktur nach FAIR-Standard

Erstelle diese Struktur in VScodium:

```
eisheiligen-analysis/
├── README.md                 # Projekt-Übersicht, Reproduzierbarkeit
├── LICENSE                   # CC-BY-4.0 oder MIT
├── pyproject.toml            # Dependencies (uv)
├── Makefile                  # Häufige Befehle (optional, aber praktisch)
├── data/
│   ├── raw/                  # Originalrohdaten (kachelmannwetter-Export)
│   │   ├── DATA_DESCRIPTION.md  # Metadaten: Quelle, Spalten, Einheiten
│   │   └── bodentemperaturen_*.csv
│   └── processed/            # Bereinigt/transformiert
│       └── eisheiligen_prepared.csv
├── notebooks/
│   ├── 01_exploration.ipynb  # EDA
│   ├── 02_analysis.ipynb     # Hauptanalyse
│   └── 03_report.ipynb       # Report-Notebook (kann zu HTML konvertiert werden)
├── src/
│   └── eisheiligen/
│       ├── __init__.py
│       ├── data_loading.py   # CSV-Laden, Validierung
│       ├── analysis.py       # Berechnungen (letzte Bodenfrost, etc.)
│       └── plotting.py       # Visualisierungen (wiederverwendbar)
├── results/
│   ├── figures/              # PNG/SVG der Plots
│   └── statistics/           # CSV mit Ergebnissen
└── .github/
    └── workflows/            # GitHub Actions (optional)
```

### Schritt 1.3: uv + Python-Umgebung

**pyproject.toml erstellen:**

```toml
[project]
name = "eisheiligen-analysis"
version = "0.1.0"
description = "Analyse der Eisheiligen und Bodenfrost in Deutschland"
requires-python = ">=3.11"
authors = [{name = "Dein Name", email = "dein@email.de"}]
license = {text = "CC-BY-4.0"}

dependencies = [
    "pandas>=2.0",
    "numpy>=1.24",
    "matplotlib>=3.7",
    "seaborn>=0.12",
    "jupyter>=1.0",
    "scipy>=1.11",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=23.0",
    "ruff>=0.1",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

**In VScodium Terminal:**

```bash
# uv installieren (falls nicht vorhanden)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Umgebung aufsetzen
uv sync

# Aktivieren
source .venv/bin/activate  # Linux/Mac
# oder in PowerShell: .venv\Scripts\Activate.ps1
```

### Schritt 1.4: GitHub Projects Setup

1. **GitHub.com Repo → "Projects"** Tab
2. **Neues Project:** "Eisheiligen Analysis Workflow"
3. **Template:** "Table" (einfach, übersichtlich)
4. **Spalten:** Status | Priority | Assigned
5. **Issues erstellen & in Project ziehen:**
   - `Setup: Daten laden & validieren`
   - `EDA: Zeitreihen visualisieren`
   - `Analyse: Eisheiligen-Definition + Messungen`
   - `Analyse: Klimawandel-Trend prüfen`
   - `Report: Dokumentation + Visualisierungen`
   - `Finalisierung: README, Doku, Clean-Up`

**Pro-Tipp:** Label setzen (`data`, `analysis`, `documentation`) für Überblick

### Schritt 1.5: Erste Commits

```bash
git add README.md pyproject.toml data/raw/DATA_DESCRIPTION.md
git commit -m "chore: Initialize project structure and dependencies"
git push
```

---

## Phase 2: Daten-Pipeline & EDA (6–8 h)

### Schritt 2.1: Datenbeschreibung (DATA_DESCRIPTION.md)

```markdown
# Bodentemperatur-Daten

**Quelle:** [CDC DWD](https://cdc.dwd.de/portal/202209231028/mapview), Tagestiefsttemperaturen in 5 cm Bodentiefe

**Zeitraum:** [DEINE SPANNE]

**Stationen:**
- Station 1: Stadt1 (Koordinaten, Höhe m ü.M.)
- Station 2: Stadt2
- Station 3: Stadt3

**Spalten:**
- `Datum`: YYYY-MM-DD
- `Station`: Name
- `T_min_5cm`: Tagestiefsttemperatur 5 cm in °C

**Datenqualität:** Lücken? Plausibilität geprüft?

**Lizenz:** [Quelle angeben]
```

### Schritt 2.2: Daten-Loading Script (src/eisheiligen/data_loading.py)

```python
import pandas as pd
from pathlib import Path

def load_raw_data(filepath: str) -> pd.DataFrame:
    """
    Lade CSV und validiere Struktur.
    
    Args:
        filepath: Pfad zur CSV-Datei
        
    Returns:
        DataFrame mit parsed Datum
    """
    df = pd.read_csv(filepath, parse_dates=['Datum'])
    
    # Validierung
    assert df['T_min_5cm'].dtype in ['float64', 'int64'], "Temperatur sollte numerisch sein"
    assert not df['Datum'].isna().any(), "Datum darf keine NaN sein"
    
    return df.sort_values('Datum').reset_index(drop=True)

def load_all_stations(raw_data_dir: str = "data/raw") -> pd.DataFrame:
    """Lade alle CSV-Dateien aus raw-Ordner."""
    dfs = []
    for csv_file in Path(raw_data_dir).glob("*.csv"):
        print(f"Loading {csv_file.name}...")
        dfs.append(load_raw_data(csv_file))
    
    return pd.concat(dfs, ignore_index=True)
```

### Schritt 2.3: EDA-Notebook (notebooks/01_exploration.ipynb)

```python
# Zelle 1: Setup
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
import sys

sys.path.insert(0, "../src")
from eisheiligen.data_loading import load_all_stations

sns.set_theme(style="whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)

# Zelle 2: Daten laden
df = load_all_stations()
print(f"Shape: {df.shape}")
print(df.head(10))
print(df.info())
print(df.describe())

# Zelle 3: Zeitreihe pro Station
for station in df['Station'].unique():
    data = df[df['Station'] == station].sort_values('Datum')
    plt.figure(figsize=(14, 5))
    plt.plot(data['Datum'], data['T_min_5cm'], linewidth=0.5)
    plt.title(f"Bodentemperatur (5cm) - {station}")
    plt.xlabel("Datum")
    plt.ylabel("Temperatur (°C)")
    plt.grid(True, alpha=0.3)
    plt.axhline(0, color='red', linestyle='--', alpha=0.5, label='0°C (Bodenfrost-Grenze)')
    plt.legend()
    plt.tight_layout()
    plt.savefig(f"../results/figures/{station}_timeseries.png", dpi=300)
    plt.show()

# Zelle 4: Jahres-Muster (April-Mai Fokus)
df['Month'] = df['Datum'].dt.month
df['DoY'] = df['Datum'].dt.dayofyear  # Tag des Jahres
april_mai = df[(df['Month'].isin([4, 5]))].copy()

plt.figure(figsize=(12, 6))
for station in april_mai['Station'].unique():
    data = april_mai[april_mai['Station'] == station]
    plt.scatter(data['DoY'], data['T_min_5cm'], alpha=0.5, label=station)
plt.axvline(121, color='orange', linestyle='--', linewidth=2, label='Eisheilige (11.-15. Mai)')
plt.xlabel("Tag des Jahres (April=91-120, Mai=121-151)")
plt.ylabel("Tagestiefst-Bodentemperatur (°C)")
plt.title("Bodentemperatur April-Mai")
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("../results/figures/april_mai_pattern.png", dpi=300)
plt.show()

# Zelle 5: Lücken-Analyse
print(f"Fehlende Werte pro Station:")
print(df.groupby('Station')['T_min_5cm'].apply(lambda x: x.isna().sum()))
```

**Git-Commit nach Phase 2:**
```bash
git add data/raw/DATA_DESCRIPTION.md src/ notebooks/01_exploration.ipynb results/figures/
git commit -m "feat: Data loading + EDA notebook with visualization"
git push
```

---

## Phase 3: Analyse & Methodik (6–8 h)

Hier brauchst du eine **klare Definition, wie die Bauernregel gemessen wird**:

### Schritt 3.1: Analyse-Funktionen definieren (src/eisheiligen/analysis.py)

```python
import pandas as pd
import numpy as np
from scipy import stats

def find_last_frost_day(df_station: pd.DataFrame, year: int) -> dict:
    """
    Finde den letzten Tag mit Bodentemperatur < 0°C im Frühjahr.
    
    Args:
        df_station: DataFrame einer Station mit Spalten ['Datum', 'T_min_5cm']
        year: Zu analysierende Jahresjahreszahl
        
    Returns:
        dict mit 'last_frost_date', 'doy', 'is_eisheilen_period'
    """
    # Betrachte Jan-Juni
    df_year = df_station[df_station['Datum'].dt.year == year].copy()
    
    # Finde letzten Tag mit T < 0°C
    frost_days = df_year[df_year['T_min_5cm'] < 0.0].copy()
    
    if frost_days.empty:
        return {
            'year': year,
            'last_frost_date': None,
            'doy': None,
            'is_eisheilen_period': None,
            'notes': 'Kein Frost im Zeitraum'
        }
    
    last_frost = frost_days.iloc[-1]
    doy = last_frost['Datum'].dayofyear
    
    # Eisheilige: 11.-15. Mai = DoY 131-135
    eisheiligen_start = 131
    eisheiligen_end = 135
    is_in_period = eisheiligen_start <= doy <= eisheiligen_end
    
    return {
        'year': year,
        'last_frost_date': last_frost['Datum'],
        'doy': doy,
        'is_eisheilen_period': is_in_period,
        'notes': ''
    }

def analyze_all_years(df: pd.DataFrame) -> pd.DataFrame:
    """Wende Analyse auf alle Jahre und Stationen an."""
    results = []
    
    for station in df['Station'].unique():
        df_station = df[df['Station'] == station].sort_values('Datum')
        years = df_station['Datum'].dt.year.unique()
        
        for year in years:
            result = find_last_frost_day(df_station, year)
            result['Station'] = station
            results.append(result)
    
    return pd.DataFrame(results)

def test_eisheilen_hypothesis(analysis_df: pd.DataFrame) -> dict:
    """
    Teste die Hypothese: "Letzte Bodenfrost fällt typischerweise auf Eisheilige"
    
    Methoden:
    1. Anteil der Jahre, in denen Frost in Eisheiligen-Periode fällt
    2. Vergleich: Median DoY vs. Eisheilige-Zentrum (133)
    3. Trend-Test: Verschiebt sich der Frosttag mit den Jahren?
    """
    analysis_df = analysis_df.dropna()
    
    # 1. Anteil
    in_period = analysis_df['is_eisheilen_period'].sum()
    total = len(analysis_df)
    proportion = in_period / total if total > 0 else np.nan
    
    # 2. Median vs. Eisheilige
    median_doy = analysis_df['doy'].median()
    eisheilige_center = 133
    
    # 3. Trend: Lineare Regression DoY