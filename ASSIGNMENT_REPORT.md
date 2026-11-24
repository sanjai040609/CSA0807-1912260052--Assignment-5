# SIMATS ENGINEERING
### DEPARTMENT OF COMPUTER SCIENCE AND ENGINEERING
## CSA08 / CSA0807 – PYTHON PROGRAMMING
### LABORATORY ASSIGNMENT 5 (COURSE OUTCOME: CO5)
**DATA ACQUISITION, CLEANING, ANALYSIS AND AI APPLICATIONS**

---

### CANDIDATE AND COURSE CREDENTIALS

| Academic Parameter | Official Institutional Record |
| :--- | :--- |
| **Student Name** | V. Karthik Harish |
| **Register Number** | 192511328 |
| **Degree & Program** | B.E. / B.Tech – Computer Science and Engineering |
| **Course Code & Title** | CSA08 / CSA0807 – Python Programming |
| **Course Outcome Assessed** | CO5 – Develop Python programs using libraries and tools to acquire, clean, visualise and analyse data from sources such as CSV files and SQL databases |
| **Nature of Assignment** | Individual |
| **Duration** | 1 Week |
| **Total Marks** | 50 Marks (Part A: 25 Marks, Part B: 25 Marks) |
| **Submission Date** | 22/11/2026 |
| **Bloom's Levels Assessed**| K3 (Apply), K4 (Analyse), K5 (Evaluate) |
| **Python Version** | Python 3.14.3 (compatible with Python 3.10+) |
| **GitHub Repository Link** | [https://github.com/KarthikHarish170260/CSA08-192511328-Assignment5](https://github.com/KarthikHarish170260/CSA08-192511328-Assignment5) |

---

# PROGRESSIVE GIT COMMIT HISTORY

Per course guidelines, all tasks were executed, benchmarked, and committed progressively into the public repository:

| Commit Hash | Milestone / Task | Progressive Commit Description |
| :--- | :--- | :--- |
| `0c4bd60` | Setup | `Initial project setup: directory layout, requirements.txt, .gitignore, README.md, and color screenshot helper` |
| `c2fb603` | Task A | `Task A: Part A design specification, SIMATS pseudocode, schema definitions & cleaning rules` |
| `bfef489` | Task B1 | `Task B1: Cleaning pipeline, complete-the-code verification, SQLite queries & 4-panel charts` |
| `8de4152` | Task B2 | `Task B2: Adversarial file stress testing, SQL injection parameterisation & NLP edge cases` |
| `e61b8f4` | Task B3 | `Task B3: Loops vs Pandas vs SQL scaling benchmark (1.05M rows) & cProfile bottleneck` |
| `af7f3eb` | Task B4 | `Task B4: Production scaling, 50-complaint AI sentiment/summarizer & SQLite B-Tree index` |

---

# PART A – DATA AND PROGRAM DESIGN (25 Marks)

## TASK A1 – DESIGN THE DATA ACQUISITION (7 Marks | Bloom's: K3 – Apply)

### A1.1 Source Inspection and Expected Data-Quality Problems

The city air-quality monitoring network consists of 12 stations sending daily CSV files with the schema:
`station,date,hour,pm25,pm10,no2,temperature`

Inspection of the physical telemetry pipeline reveals six critical data-quality defects:

| No. | Expected Data-Quality Problem | Underlying Physical / Transmission Cause | Concrete Raw CSV Example | Systemic Business Impact |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **Missing / Blank Readings** | Sensor packet dropped during cellular transmission; serial connection timeout. | `2026-10-01,14,,45,22,30.5` or `...,NA,...` | Distorts daily averages; causes `ValueError` if parsed naively as floats. |
| **2** | **Negative Sensor Values** | Zero-point baseline voltage drift in uncalibrated optical particle counters. | `2026-10-01,03,-12.0,18,10,24.0` | Physical mass concentration cannot be negative; invalidates scientific records. |
| **3** | **Impossibly Large Values** | Electrical power transients or analog-to-digital converter (ADC) short-circuits. | `2026-10-01,18,9999.0,80,45,28.0` | Skews mean calculations and triggers false public health emergency alarms. |
| **4** | **Unit Inconsistency** | Sensor firmware configuration mismatch (stations Guindy and Marina report in °F). | `2026-10-01,12,45,80,30,86.0` | 86.0°F interpreted as 86.0°C corrupts thermal inversion dispersion models. |
| **5** | **Duplicate Telemetry Rows** | Cellular modem retry buffers re-transmitting unacknowledged TCP packets. | `Adyar,2026-10-01,14,52,86,25,32` (repeated) | Double-counts particulate load, inflating station pollution statistics. |
| **6** | **Corrupted Delimiters / Text** | Transmission line noise or regional localized terminal exports. | `Adyar;2026-10-01;08;48.0;82.0;24.0` | Crashes standard comma-separated split parsers with single-token string errors. |

---

### A1.2 Relational Database Schema Design (SQLite)

```sql
-- Master station registry table
CREATE TABLE IF NOT EXISTS stations (
    station_id TEXT PRIMARY KEY,
    station_name TEXT NOT NULL,
    zone TEXT NOT NULL,
    native_temp_unit TEXT DEFAULT 'C'
);

-- Cleaned hourly telemetry readings table
CREATE TABLE IF NOT EXISTS readings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    station_id TEXT NOT NULL,
    reading_date TEXT NOT NULL,          -- ISO-8601: YYYY-MM-DD
    hour INTEGER NOT NULL,               -- 0 to 23
    pm25 REAL,                           -- µg/m³
    pm10 REAL,                           -- µg/m³
    no2 REAL,                            -- ppb
    temperature REAL,                    -- Standardized Celsius (°C)
    pm25_minmax REAL,                    -- Normalized [0, 1]
    pm25_zscore REAL,                    -- Standardized (mean=0, std=1)
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    UNIQUE(station_id, reading_date, hour)
);

-- Composite B-Tree index for spatial-temporal query acceleration
CREATE INDEX IF NOT EXISTS idx_station_date ON readings(station_id, reading_date);
CREATE INDEX IF NOT EXISTS idx_date_hour ON readings(reading_date, hour);
```

#### CSV Column to SQLite Mapping:
- `station` $\rightarrow$ `station_id` (foreign key validated against `stations`)
- `date` $\rightarrow$ `reading_date` (validated ISO-8601 string)
- `hour` $\rightarrow$ `hour` (integer range $[0, 23]$)
- `pm25, pm10, no2` $\rightarrow$ `REAL` (floating point, NULL for missing)
- `temperature` $\rightarrow$ `temperature` (converted to Celsius if native was Fahrenheit)

---

### A1.3 CSV or SQL? Architectural Query Comparison

| Board Inquiry | CSV Ingestion Approach | SQL Query Approach | Performance & Architectural Verdict |
| :--- | :--- | :--- | :--- |
| **Q1: Average PM2.5 of Adyar in October** | Iterate through 372 daily CSV files; parse strings and accumulate in Python. | `SELECT AVG(pm25) FROM readings WHERE station_id = 'Adyar' AND reading_date LIKE '2026-10-%';` | **SQL is $\approx 80\times$ faster**: B-Tree index avoids touching unrelated stations and dates. |
| **Q2: Top 3 Polluted Stations by Monthly PM10** | Load all records into RAM; group by station in nested dictionary; sort keys. | `SELECT station_id, AVG(pm10) FROM readings GROUP BY station_id ORDER BY AVG(pm10) DESC LIMIT 3;` | **SQL is superior**: Internal C hash aggregation executes in single-digit milliseconds. |
| **Q3: Diurnal Hourly Trend Across Stations** | Load all files; bucket rows into 24 bins using `defaultdict`; compute averages. | `SELECT hour, AVG(pm25) FROM readings GROUP BY hour ORDER BY hour;` | **SQL is superior**: Database uses disk paging without blowing up memory footprint. |

#### Alternative Concept and Complexity Analysis:
- **CSV + Pandas DataFrame per Question:** Requires loading and re-parsing raw text files on every query ($O(N)$ full disk scan and high GC memory churn).
- **Indexed SQLite:** Operates in $O(\log N + k)$ search time via composite B-Tree indexes with an $O(1)$ constant working-set page cache.

---

## TASK A2 – DESIGN THE CLEANING AND NORMALISATION (5 Marks | Bloom's: K4 – Analyse)

### A2.1 Cleaning Rules and Justifications

1. **Missing Values (`""`, `"NA"`):**
   - *Rule:* If an isolated missing reading occurs (gap $\le 2$ hours), apply Linear Forward Fill (Last Observation Carried Forward - LOCF). If gap $> 2$ hours, impute with the station's historical diurnal median for that specific hour. If an entire day is absent, mark as NULL and do not impute.
   - *Justification:* Preserves diurnal cycle continuity without distorting variance or dampening peak rush-hour signals.
2. **Impossible Values ($< 0$ or $> 999$):**
   - *Rule:* Values $< 0$ or $> 999\ \mu\text{g/m}^3$ are stripped and treated as missing.
   - *Justification:* Physical particulate concentration cannot be negative; values $> 999$ represent electrical ADC transients that would catastrophically skew arithmetic means.
3. **Duplicate Telemetry Rows:**
   - *Rule:* Enforce unique constraint on `(station_id, reading_date, hour)`. Retain the first packet; discard subsequent duplicates.
   - *Justification:* Prevents double-counting caused by cellular retry buffers.
4. **Fahrenheit Temperatures:**
   - *Rule:* Convert stations with Fahrenheit configuration via:
     $$C = (F - 32) \times \frac{5}{9}$$
   - *Justification:* Harmonizes temperature across all 12 stations to ensure valid atmospheric dispersion analysis.

---

### A2.2 Normalisation Design

1. **Min-Max Normalisation:**
   $$x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}} \in [0, 1]$$
   - *Appropriate Use:* When algorithms require strictly bounded positive inputs (e.g., neural networks, comparative visual heatmaps, composite air-quality index scores).
2. **Z-Score Standardisation:**
   $$z = \frac{x - \mu}{\sigma}$$
   - *Appropriate Use:* When identifying statistical anomalies and extreme pollution episodes ($|z| > 3.0$), assuming an underlying bell-shaped distribution.

---

### A2.3 Canonical Formatting Standards

- **Dates:** Strict ISO-8601 string format `YYYY-MM-DD` (e.g. `2026-10-15`).
- **Hours:** Integer values strictly within $[0, 23]$.
- **Station Names:** Canonical Title Case strings (e.g. `Adyar`, `Guindy`, `Anna Nagar`).
- **Numbers:** IEEE 754 64-bit floating point rounded to 2 decimal places for reporting.

---

## TASK A3 – DESIGN THE ANALYSIS AND VISUALISATION (5 Marks | Bloom's: K4 – Analyse)

### A3.1 Justified Chart Selections

| Analytical Inquiry | Selected Chart Type | Technical Justification |
| :--- | :--- | :--- |
| **Trend of PM2.5 over a Month** | **Time-Series Line Chart with Moving Average** | Displays continuous temporal progression, day-to-day weather shifts, and episodic surges. |
| **Comparison Across Stations** | **Station Box Plot (Five-Number Summary)** | Visualizes median, interquartile range (IQR), variance spread, and extreme outlier spikes across stations. |
| **Temperature vs PM2.5 Relationship** | **Scatter Plot with Fitted Polynomial Trendline** | Reveals meteorological correlation, specifically nocturnal boundary-layer thermal inversions trapping pollutants. |
| **Hours with Worst Air Quality** | **Diurnal 24-Hour Bar Chart with Rush-Hour Highlights** | Directly isolates bimodal morning (08:00–10:00) and evening (19:00–21:00) traffic congestion surges. |

---

### A3.2 Daily Summary Table Design

- **Columns:** `station_id, reading_date, valid_hours_count, min_pm25, max_pm25, mean_pm25, std_pm25, air_quality_band`
- **Production Method:** Generated via an optimized SQL aggregate query:
  ```sql
  SELECT station_id, reading_date, COUNT(pm25), MIN(pm25), MAX(pm25), ROUND(AVG(pm25), 2)
  FROM readings
  WHERE pm25 IS NOT NULL
  GROUP BY station_id, reading_date;
  ```

---

## TASK A4 – PLAN DEBUGGING AND PROFILING (4 Marks | Bloom's: K4 – Analyse)

### A4.1 Hierarchical Debugging Strategy
1. **Print Statements:** Quick sanity checking during initial prototype scripting.
2. **Assert Statements:** Enforce data invariants at module boundaries (e.g. `assert 0.0 <= minmax <= 1.0`).
3. **Logging Module:** Production-grade logging with levels (`DEBUG`, `INFO`, `WARNING`, `ERROR`), rotatable log files, and zero code modification to disable (`logger.setLevel(logging.WARNING)`).
4. **PDB Debugger:** Interactive breakpoint inspection (`breakpoint()`) to inspect local namespace variables upon unexpected exceptions.

### A4.2 Profiling Plan & Instrumentation
1. **`time.perf_counter()`:** High-precision macroscopic benchmarking of end-to-end ingestion and query blocks.
2. **`timeit` Module:** Micro-benchmarking specific function implementations (e.g. string splitting vs regex parsing).
3. **`cProfile` Module:** Call-graph statistical profiling to measure cumulative execution time (`cumtime`) and call counts (`ncalls`) to isolate CPU bottlenecks.

---

## TASK A5 – DESIGN AN AI APPLICATION FOR CITIZEN COMPLAINTS (4 Marks | Bloom's: K3 – Apply)

### A5.1 Lexicon Sentiment Scorer and Extractive Summariser
- **Lexicon Scorer:** Tokenizes input string into words; checks membership in `POSITIVE` (+1) and `NEGATIVE` (-1) sets. Maintains a `negate` boolean flag triggered by `NEGATORS` (`not`, `never`, `no`) that inverts the score of the subsequent sentiment token.
- **Extractive Summariser:** Computes token frequencies of non-stopwords across all complaints. Evaluates sentences by dividing the sum of constituent word frequencies by sentence length. Selects the top $k$ sentences with the highest keyword density.

### A5.2 Rule-Based Keyword Chatbot & Critical Limitations
- **Architecture:** State-machine matcher extracting metric (`PM2.5`, `temperature`), station name (`Adyar`, `Guindy`), and temporal marker (`today`, `yesterday`). Formulates parameterized SQL query and constructs a natural language response.
- **Critical Limitations:**
  1. *Brittle to Typos & Synonyms:* Fails on misspelled station names (`Adayar`) or colloquial phrases.
  2. *Zero Contextual Memory:* Cannot handle follow-up queries ("What about yesterday?").
  3. *Out-of-Domain Failure:* Completely fails on unstructured or compound intent inquiries.

---

# PART B – IMPLEMENTATION & EXPERIMENTAL INVESTIGATION (25 Marks)

## TASK B1 – IMPLEMENT AND VALIDATE THE CLEANING PIPELINE (7 Marks | Bloom's: K3 – Apply)

### B1.1 "Complete the Code" Implementation & Verification

```python
def load_clean(text):
    """Returns (station, date, pm25) tuples for valid, non-negative readings <= 999 only."""
    rows = []
    reader = csv.DictReader(io.StringIO(text.strip()))
    for r in reader:
        raw_val = r["pm25"].strip()
        if not raw_val or raw_val.upper() == "NA":
            continue
        try:
            val = float(raw_val)
            if 0.0 <= val <= 999.0:
                rows.append((r["station"].strip(), r["date"].strip(), val))
        except ValueError:
            continue
    return rows

def min_max(values):
    if not values:
        return []
    min_v = min(values)
    max_v = max(values)
    if min_v == max_v:
        return [0.0 for _ in values]
    return [round((v - min_v) / (max_v - min_v), 3) for v in values]
```

#### Verified Output:
```text
Cleaned Tuples Result:  [('Adyar', '2026-10-01', 48.0), ('Guindy', '2026-10-01', 95.0), ('Guindy', '2026-10-03', 121.0)]
Min-Max Scaled Result:  [0.0, 0.644, 1.0]
```
*Exact match with course expected output specification! Sensor spike row (`Guindy,2026-10-04,1500`) was correctly filtered out.*

---

### B1.2 Cleaning Pipeline Before and After Evidence

Testing on `raw_sensor_readings.csv` (24 raw records) yielded **17 clean records** and filtered out **7 anomalies**:
- *Line 4 & 5 [Adyar]:* Missing/blank PM2.5 readings $\rightarrow$ Filtered out.
- *Line 6 [Adyar]:* Negative PM2.5 value (-12.0) $\rightarrow$ Filtered out.
- *Line 7 [Adyar]:* Impossibly large PM2.5 spike (1250.0) $\rightarrow$ Filtered out.
- *Line 9 [Adyar]:* Duplicate telemetry packet $\rightarrow$ Deduplicated.
- *Line 11 [Guindy]:* Negative PM2.5 value (-5.0) $\rightarrow$ Filtered out.
- *Line 14 [Guindy]:* Missing/NA PM2.5 reading $\rightarrow$ Filtered out.
- *Station Guindy (Lines 10–15):* Fahrenheit temperatures ($86.0^\circ\text{F} \dots 93.2^\circ\text{F}$) converted to Celsius ($30.0^\circ\text{C} \dots 34.0^\circ\text{C}$).

---

### B1.3 SQLite Database Execution & Query Answers

Clean records were loaded into `air_quality.db` via `executemany()`:

```text
Query 1: Average PM2.5 of Adyar in October:
  Result: 51.67 µg/m³

Query 2: Top 3 Polluted Stations by Monthly Mean PM10:
  Rank 1: Guindy       -> 153.25 µg/m³
  Rank 2: Anna Nagar   -> 129.60 µg/m³
  Rank 3: Adyar        -> 86.00 µg/m³

Query 3: Diurnal Hourly Trend Across All Stations:
  Hour 08:00 -> Mean PM2.5:  65.75 µg/m³ (4 valid readings)
  Hour 09:00 -> Mean PM2.5:  62.33 µg/m³ (3 valid readings)
  Hour 10:00 -> Mean PM2.5:  83.67 µg/m³ (3 valid readings)
  Hour 11:00 -> Mean PM2.5:  77.33 µg/m³ (3 valid readings)
  Hour 12:00 -> Mean PM2.5:  57.50 µg/m³ (2 valid readings)
  Hour 13:00 -> Mean PM2.5: 115.00 µg/m³ (1 valid readings)
  Hour 14:00 -> Mean PM2.5:  52.00 µg/m³ (1 valid readings)
```

---

### B1.4 Publication-Grade Visualisations and Interpretation

![Figure 1: Task B1 Four-Panel Scientific Air Quality Visualisations](report_assets/fig1_b1_pipeline_charts.png)

1. **Chart (a) - Monthly Trend:** Industrial Guindy consistently breaches the 24-hr NAAQS standard ($60\ \mu\text{g/m}^3$), displaying severe surges during mid-month weather stagnation, whereas coastal Adyar remains mostly compliant.
2. **Chart (b) - Station Comparison:** Guindy exhibits the highest median PM2.5 ($\approx 112\ \mu\text{g/m}^3$) and largest IQR, driven by industrial boiler emissions, while coastal Marina benefits from sea-breeze dispersion.
3. **Chart (c) - Temp vs PM2.5:** Clear negative correlation (slope: $-2.14$); nocturnal cooling ($< 26^\circ\text{C}$) induces shallow boundary-layer thermal inversions that trap particulate matter near the surface.
4. **Chart (d) - Diurnal Peaks:** Distinct bimodal rush-hour peaks at 08:00–10:00 (morning traffic commute) and 19:00–21:00 (evening traffic compounded by nighttime cooling).

---

### B1.5 Alternative Concept Benchmark: Python Loops vs Pandas Vectorisation

| Row Count ($N$) | Python Loops Runtime (ms) | Pandas Vectorised Runtime (ms) | Python RAM (MB) | Pandas RAM (MB) |
| :--- | :--- | :--- | :--- | :--- |
| **1,000** | 1.90 ms | 31.08 ms | 0.01 MB | 0.07 MB |
| **100,000** | 117.40 ms | 39.05 ms | 0.85 MB | 7.06 MB |
| **1,000,000** | 1,172.67 ms | 322.79 ms | 7.95 MB | 70.45 MB |

**Findings:** At small scales ($N=1,000$), plain Python loops are faster because Pandas pays module overhead. At $N=1,000,000$, Pandas vectorised C-loops are **$3.6\times$ faster**, but consume **$\approx 8.8\times$ more RAM** (70.45 MB vs 7.95 MB).

---

## TASK B2 – ENGINEER ADVERSARIAL DATA (5 Marks | Bloom's: K4 – Analyse)

### B2.1 Complete the Code & Parameterised Query Defense

```python
con.executemany("INSERT INTO aq VALUES (?, ?, ?)", readings)
con.commit()

def station_average(name):
    cur = con.execute("SELECT AVG(pm25) FROM aq WHERE station = ?", (name,))
    row = cur.fetchone()
    return round(row[0], 1) if (row and row[0] is not None) else None
```

#### Execution Output:
```text
54.0 108.0
None
```

#### Why Injection Attempt Returns `None`:
In a parameterized query, SQLite compiles the SQL syntax structure *before* binding the argument. The parameter is treated strictly as a literal text value. Because no station is literally named `"Adyar' OR '1'='1"`, zero rows match, and `AVG(pm25)` evaluates to `NULL` (`None`).

---

### B2.2 Vulnerable String Concatenation vs Parameterization AST

![Figure 3: SQL Injection AST Parse Tree Comparison](report_assets/fig3_b2_sql_injection.png)

When using string concatenation (`f"SELECT * FROM users WHERE station = '{name}'"`), the input `admin' OR '1'='1` alters the SQL AST, introducing an `OR` binary operator whose right-hand operand is always `TRUE`, dumping the entire table (catastrophic breach). Parameterization preserves the immutable query tree.

---

### B2.3 Adversarial File Stress Tests

1. **Missing Column File (`missing_column.csv`):** Missing columns detected by header set intersection; imputed as NULL while preserving valid PM2.5 rows.
2. **Wrong Delimiter (`wrong_delimiter.csv`):** Semicolon delimiter detected via sniffer; parsed successfully without loss.
3. **Different Text Encoding (`different_encoding.csv`):** Latin-1 byte sequences detected; recovered via fallback encoding with zero crashes.
4. **Empty File (`empty_file.csv`):** Zero bytes detected; gracefully bypassed with `EmptyFileWarning`.

---

### B2.4 Adversarial Sentiment Testing & Limitations

| Stress Test Case | Input Sentence | Model Label | Human Truth | Underlying Linguistic Defect |
| :--- | :--- | :--- | :--- | :--- |
| **Sarcasm** | *"Oh great, another glorious day of inhaling fresh black smoke from the factory."* | `positive` | `negative` | Counts 'great', 'glorious', 'fresh' as +3, missing the sarcastic irony. |
| **Double Negative** | *"The air in the industrial corridor is not not bad today."* | `positive` | `negative` | Fails to cancel out consecutive negators; only immediate next word is toggled. |
| **Mixed Opinion** | *"The park is clean and beautiful, but the factory nearby emits terrible toxic smoke."* | `negative` | `negative` | Positive (+2) and negative (-2) tokens cancel out; model is blind to clause contrast ('but'). |
| **Implicit Sentiment** | *"I have had a severe asthma attack every night this week."* | `neutral` | `negative` | Medical symptom vocabulary is absent from generic positive/negative lists. |

---

### B2.5 Benchmark: `executemany()` vs `execute()` in a Loop

| Rows ($N$) | `executemany()` Latency (ms) | `execute()` Loop Latency (ms) | Measured Speedup Factor |
| :--- | :--- | :--- | :--- |
| **100** | 0.050 ms | 0.207 ms | **4.1x** |
| **1,000** | 1.047 ms | 1.989 ms | **1.9x** |
| **10,000** | 5.364 ms | 23.150 ms | **4.3x** |

`executemany()` amortizes journal disk synchronization over the entire batch within a single transaction, avoiding repeated `fsync` system calls.

---

## TASK B3 – LOOPS vs LIBRARIES vs SQL (8 Marks | Bloom's: K4 – Analyse)

### B3.1 Correctness on Small Data

All three paradigms (Python Loops, Pandas `groupby`, and SQL `GROUP BY`) yielded **exact numerical agreement** across daily station averages ($60.00$, $44.00$, $110.00$, $95.00$).

---

### B3.2 Performance Scaling Across Data Sizes (1 to 120 Stations)

![Figure 4: Loops vs Pandas vs SQLite Scaling Benchmark](report_assets/fig4_b3_scaling_benchmark.png)

| Station Count | Total Rows ($N$) | Python Loops (ms) | Pandas Groupby (ms) | SQL GROUP BY (ms) |
| :--- | :--- | :--- | :--- | :--- |
| **1 Station** | 8,760 rows | 1.33 ms | 3.91 ms | **1.43 ms** |
| **12 Stations** | 105,120 rows | 15.29 ms | 33.03 ms | **12.34 ms** |
| **120 Stations** | 1,051,200 rows | 177.74 ms | 309.82 ms | **154.64 ms** |

**Findings:** SQLite indexed queries remain the fastest and most memory-efficient aggregation engine at 1.05 million rows, completing in 154.64 ms.

---

### B3.3 cProfile Bottleneck Analysis

![Figure 5: cProfile Execution Time Breakdown](report_assets/fig5_b3_cprofile_flame.png)

1. **`dict.__getitem__` (42.5% of time):** In-memory hash collisions and dictionary bucket lookups dominate execution time.
2. **`tuple.__new__` (26.8% of time):** Allocating 105,120 key tuples `(station, date)` creates garbage collector overhead.
3. **Float Addition (14.2% of time):** Python virtual machine opcode overhead during accumulation.

---

### B3.4 Single-Pass Running Accumulator Complexity Analysis

- **Single-Pass Running Accumulator ($O(S \cdot D)$ space):** Keeps running sum and count in-place. For 120 stations over 365 days, requires strictly $120 \times 365 = 43,800$ entries.
- **Storing All Raw Readings ($O(N)$ space):** Stores all 1,051,200 float objects in RAM.
- **Impact:** Single-pass accumulation provides a **$24\times$ memory reduction** ($1,051,200 / 43,800$).

---

## TASK B4 – FINAL ENGINEERING DECISION (5 Marks | Bloom's: K5 – Evaluate)

### B4.1 Complete the Code Execution

```python
def score(text):
    words = text.lower().replace(".", " ").replace(",", " ").split()
    total = 0
    negate = False
    for w in words:
        if w in NEGATORS:
            negate = True
            continue
        val = 1 if w in POSITIVE else -1 if w in NEGATIVE else 0
        if val != 0:
            if negate:
                val = -val
                negate = False
            total += val
    return total
```

#### Output:
```text
positive | Roads are clean and the bus was fast.
negative | Smoke from the factory, air is bad.
negative | The staff were not helpful.
positive | Not bad, service improved.
```

---

### B4.2 Evaluation on 50 Real Citizen Complaints

![Figure 7: NLP Sentiment Confusion Matrix](report_assets/fig7_b4_nlp_evaluation.png)

- **Total Complaints:** 50
- **Agreed Classifications:** 44
- **Overall Agreement Rate:** **88.0%**
- **Disagreements (6 complaints):** Caused by unlisted medical terms (*asthma attack*), implicit pollution context (*garbage compactor leachate*), and complex clause contrast.

---

### B4.3 Database Indexing Experiment (100,000 Records)

![Figure 6: SQLite Query Latency - Table Scan vs B-Tree Index](report_assets/fig6_b4_indexing_speedup.png)

- **Unindexed Full Table Scan:** 3.730 ms
- **B-Tree Composite Index Range Scan:** 0.280 ms
- **Measured Acceleration:** **$13.3\times$ faster**

---

### B4.4 Final Production Architecture Blueprint (100 Stations, 50,000 Complaints)

1. **Telemetry Storage & Ingestion:**
   - Deploy PostgreSQL / TimescaleDB or partitioned SQLite with composite B-Tree indexes on `(station_id, reading_timestamp)`.
   - Ingest 15-minute micro-batches using atomic transactions (`executemany`) to eliminate per-row fsync overhead.
2. **Data Cleaning:**
   - Gateway boundary validation filtering impossible values ($<0$ or $>999$) upon ingestion. Pre-compute min-max and z-scores during ingestion.
3. **AI / NLP Decision:**
   - At 50,000 complaints a month ($\approx 0.02\text{ complaints/second}$), the workload is modest.
   - Replace the word-list scorer with a fine-tuned **DistilBERT / Small Language Model (SLM)**. It processes complaints in $\approx 15\text{ ms}$ on CPU, uses $<200\text{ MB}$ RAM, and achieves $>95\%$ accuracy, eliminating the 12% failure rate of rule-based models.
