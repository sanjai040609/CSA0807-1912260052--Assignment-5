# Python Programming (CSA08) - Assignment 5 (CO5)
## Data Acquisition, Cleaning, Analysis and AI Applications

**Student Name:** V. Karthik Harish  
**Register Number:** 192511328  
**Department:** Computer Science and Engineering  
**Institution:** SIMATS Engineering, Saveetha Institute of Medical and Technical Sciences  
**Course Code:** CSA08 / CSA0807  
**Course Outcome:** CO5 – Develop Python programs using libraries and tools to acquire, clean, visualise and analyse data from sources such as CSV files and SQL databases  
**Nature of Assignment:** Individual  
**Total Marks:** 50 Marks (Part A: 25 Marks, Part B: 25 Marks)  
**Submission Date:** 22/11/2026  
**GitHub Repository:** [https://github.com/KarthikHarish170260/CSA08-192511328-Assignment5](https://github.com/KarthikHarish170260/CSA08-192511328-Assignment5)

---

## 1. Environment & Dependencies

- **Python Version:** `Python 3.14.3` (tested and fully compatible with Python 3.10+)
- **Operating System:** Windows 11 (64-bit)
- **Primary Libraries:** `sqlite3`, `csv`, `statistics`, `re`, `pandas`, `numpy`, `matplotlib`, `pillow`, `python-docx`

Install dependencies using:
```bash
pip install -r requirements.txt
```

---

## 2. Directory Structure

```text
CSA08-192511328-Assignment5/
│
├── TaskA_Design/                     # PART A: Data and Program Design (25 Marks)
│   ├── task_a_design_spec.py         # Comprehensive formal design validation & SIMATS pseudocode
│   ├── output.txt                    # Formatted design specification log
│   └── output_screenshot.png         # High-resolution color terminal screenshot
│
├── TaskB1_Cleaning_Pipeline/         # Task B1: Implement & Validate Cleaning Pipeline (7 Marks)
│   ├── raw_sensor_readings.csv       # Raw sensor data containing all expected defects
│   ├── task_b1_pipeline.py           # Cleaning, normalisation, SQLite storage & visualisations
│   ├── cleaned_readings.csv          # Cleaned, validated, and normalized dataset
│   ├── air_quality.db                # SQLite database populated via executemany
│   ├── output.txt                    # Execution log & Complete-the-Code outputs
│   └── output_screenshot.png         # High-resolution color terminal screenshot
│
├── TaskB2_Adversarial_Data/          # Task B2: Engineer Adversarial Data (5 Marks)
│   ├── task_b2_adversarial.py        # Corrupted delimiters, missing columns, SQL injection & NLP edge cases
│   ├── adversarial_inputs/           # Corrupted files (missing col, semicolon delimiter, latin-1, empty)
│   ├── output.txt                    # Execution log, parameterised SQL defense & sentiment stress tests
│   └── output_screenshot.png         # High-resolution color terminal screenshot
│
├── TaskB3_Loops_Libraries_SQL/       # Task B3: Loops vs Libraries vs SQL Benchmarks (8 Marks)
│   ├── task_b3_benchmark.py          # Empirical performance scaling across 1, 12, and 120 stations
│   ├── cprofile_report.txt           # cProfile function-level execution profiling output
│   ├── output.txt                    # Execution log & speedup analysis
│   └── output_screenshot.png         # High-resolution color terminal screenshot
│
├── TaskB4_Engineering_Decision/      # Task B4: Final Production Engineering Decision (5 Marks)
│   ├── citizen_complaints_50.txt     # Realistic dataset of 50 citizen air pollution complaints
│   ├── task_b4_production.py         # Negation-aware sentiment, extractive summariser & B-tree index test
│   ├── output.txt                    # Execution log, human agreement validation & indexing speedup
│   └── output_screenshot.png         # High-resolution color terminal screenshot
│
├── report_assets/                    # 7 publication-quality full-color figures & diagrams
│   ├── fig1_b1_pipeline_charts.png   # 4-panel air quality visualisations (line, boxplot, scatter, diurnal)
│   ├── fig2_b1_normalisation.png     # Min-max vs Z-score distribution curves
│   ├── fig3_b2_sql_injection.png     # Parameterised SQL query tree vs string concatenation vulnerability
│   ├── fig4_b3_scaling_benchmark.png # Loops vs Pandas vs SQL scaling across 8,760 to 1,051,200 rows
│   ├── fig5_b3_cprofile_flame.png    # cProfile bottleneck breakdown
│   ├── fig6_b4_indexing_speedup.png  # SQLite Table Scan vs B-Tree Index query latency
│   └── fig7_b4_nlp_evaluation.png    # Sentiment confusion matrix & extractive summary scores
│
├── run_all_tasks.py                  # Master test runner verifying all tasks sequentially
├── render_screenshot_helper.py       # High-resolution color terminal screenshot generator
├── push_task_by_task.ps1             # PowerShell progressive push utility
├── push_task_by_task.bat             # Batch progressive push utility
├── ASSIGNMENT_REPORT.md              # Complete analytical technical report in Markdown
├── CSA0807_Assignment5_192511328_Report.docx # Formal institutional Word report (B&W typography + Color charts)
└── requirements.txt                  # Python package requirements
```

---

## 3. Progressive Git Commit Log

Per assignment guidelines, all tasks are committed progressively:

| Commit Hash | Milestone / Task | Description |
| :--- | :--- | :--- |
| `[commit 1]` | Base Setup | Initial project layout, requirements.txt, .gitignore, README.md, screenshot helper |
| `[commit 2]` | Task A | Part A: Data acquisition, storage design, SIMATS pseudocode, cleaning rules & AI designs |
| `[commit 3]` | Task B1 | Part B1: Cleaning pipeline, complete-the-code verification, SQLite queries & 4-panel charts |
| `[commit 4]` | Task B2 | Part B2: Adversarial file stress testing, SQL injection parameterisation & NLP edge cases |
| `[commit 5]` | Task B3 | Part B3: Loops vs Pandas vs SQL scaling benchmark (1.05M rows) & cProfile bottleneck |
| `[commit 6]` | Task B4 | Part B4: Production scaling, 50-complaint AI sentiment/summarizer & SQLite B-Tree index |
| `[commit 7]` | Assets & Suite| Master test runner, push utility, and 7 publication-grade full-color academic figures |
| `[commit 8]` | Final Report | Formal 50-mark analytical laboratory report (Word .docx and Markdown) with full derivations |

---

## 4. Execution Commands

To execute and verify the entire assignment in one unified command:
```bash
python run_all_tasks.py
```
