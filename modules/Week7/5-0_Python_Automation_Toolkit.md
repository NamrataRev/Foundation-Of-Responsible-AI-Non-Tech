# Python Automation Toolkit

## What is Automation?

Every week, the university's exam office receives result files from three departments. A staff member opens each file, copies the data into a spreadsheet, calculates grades, checks for missing marks, formats the report, and emails it to the head of department. The whole process takes three to four hours. Every semester. The same steps. The same order. The same chance of a copy-paste mistake.

Automation means writing a program that does all of this instead — consistently, instantly, and without mistakes — so the staff member runs one script and gets the finished report in seconds.

This is not about replacing people. It is about freeing them from repetitive, mechanical work so they can focus on decisions that actually need human judgment — like investigating why one department's average dropped by 12 points, or following up with a student whose marks are missing.

**Automation is the right tool when a task is:**
- Repetitive — the same steps run every week or every semester
- Rule-based — the logic does not change between runs
- High volume — too many files, rows, or records to handle manually
- Error-prone — a human doing it by hand introduces mistakes that a program would not

---

## What is the Python Automation Toolkit?

The Python Automation Toolkit is a set of five Python libraries that together cover the most common automation tasks in real engineering work:

| Tool | What it does | In this pipeline |
|---|---|---|
| **os / pathlib** | Navigate folders, find files, route them by type | Finds `cs_results.csv`, `management_results.xlsx`, `external_board_results.pdf` |
| **pandas** | Read, clean, transform, and combine tabular data | Reads CSVs, cleans missing marks, calculates grades, combines all departments |
| **openpyxl** | Read and write Excel files with precise control | Reads the management Excel file, writes the final formatted report |
| **pdfplumber / pypdf** | Extract text and tables from PDF files | Extracts the external board's results table from a PDF |
| **requests** | Fetch live data from APIs over the internet | Fetches the national average from the grading API to add context to results |

These five tools are not random. They are the five most common inputs and outputs in real data automation work:

```text
Input formats a real system receives:
  CSV files      → pandas
  Excel files    → openpyxl → pandas
  PDF files      → pdfplumber → pandas
  API / internet → requests → pandas

Output a real system produces:
  Formatted Excel report → openpyxl
  Summary CSV → pandas
```

---

## Why Python for Automation?

Python is the dominant language for automation work for three reasons:

**Readable syntax** — automation scripts are read and maintained by different people over time. Python code reads close to plain English, which means a script written today can be understood and modified by someone else next semester.

**Library ecosystem** — pandas, openpyxl, pdfplumber, and requests are all free, open-source, actively maintained, and supported by enormous communities. You do not build any of this from scratch.

**Orchestration** — Python's role in automation is not to compute complex algorithms. It is to be the coordinator — open this file, pass it to this library, take the result, pass it to the next step. Python is the glue that connects tools into a pipeline.

This connects directly to what you learned in the Python foundations module: Python as the orchestration layer. The same idea scales from calling an AI model to automating an entire university's results processing system.

---

## The Scenario — Student Result Processing System

Every file in this series uses one running scenario. Understanding it fully before the first file begins makes everything that follows easier to follow.

The university has a `results/` folder. Every semester, three departments drop their result files into it:

```text
results/
├── cs_results.csv              ← CS department, plain CSV
├── management_results.xlsx     ← Management department, Excel with merged title cell
├── external_board_results.pdf  ← External exam board, PDF with a table
└── notes.txt                   ← Ignored — not a results file
```

The automation pipeline must:
1. Open the folder and identify every file
2. Route each file to the right reader based on its type
3. Extract the student data from each file
4. Clean the data — handle missing marks, fix data types
5. Combine all three departments into one DataFrame
6. Calculate grades and percentages
7. Fetch the national average from an external API
8. Write a formatted Excel report with bold headers and highlighted failing students

A staff member triggers this by running one Python script. The output is a formatted Excel report ready to send.

---

## The Full Pipeline — All Five Tools Connected

```mermaid
flowchart TD
    A[results/ folder] --> B[5-1 Folder Traversal\nos and pathlib\nFind files, identify types]
    B --> C[cs_results.csv]
    B --> D[management_results.xlsx]
    B --> E[external_board_results.pdf]
    C --> F[5-2 Pandas\nRead CSV, clean, calculate grades]
    D --> G[5-3 openpyxl\nRead Excel, skip merged title]
    G --> F
    E --> H[5-4 pdfplumber\nExtract table from PDF]
    H --> F
    F --> I[Combined DataFrame\nAll 11 students, all departments]
    I --> J[5-5 requests\nFetch national average from API]
    J --> K[5-3 openpyxl\nWrite formatted Excel report]
    K --> L[final_report.xlsx]
```

Each file in this series is one step in this pipeline. Understanding where each tool fits before you learn it makes the learning faster — you always know why you are learning something and where it connects.

---

## How to Read This Series

Each file covers one tool. Every file:
- Opens by connecting to the previous file's output
- Introduces the tool and what problem it solves
- Applies every concept to the student result processing scenario
- Ends with a "Where This Fits in the Pipeline" section showing the full flow

**The recommended order is the pipeline order:**

| File | Tool | What you will build |
|---|---|---|
| `5-1_Folder_Traversal.md` | os, pathlib, shutil | A script that opens `results/`, finds all files, and routes each to the right reader |
| `5-2_Pandas.md` | pandas | A script that reads, cleans, and calculates grades for the CS CSV |
| `5-3_Excel_Handling.md` | openpyxl | A script that reads the management Excel file and writes a formatted output |
| `5-4_PDF_Extraction.md` | pdfplumber / pypdf | A script that extracts the external board's table from a PDF |
| `5-5_HTTP_Requests.md` | requests | A script that fetches the national average and combines all three departments |

By the end of File 5-5, you will have written every piece of the pipeline. The final section of that file shows all five connected into a single working script.

---

## What You Will Be Able to Do After This Series

By the end of this series you will be able to:

- Write a Python script that processes an entire folder of mixed-format files automatically
- Read, clean, and combine data from CSV, Excel, and PDF sources using the right tool for each
- Calculate derived values — grades, percentages, pass rates — programmatically across any number of students
- Fetch live data from an external API and integrate it into a local dataset
- Write a professionally formatted Excel report with bold headers, highlighted rows, and multiple sheets
- Recognise when a task is a good candidate for automation and design the pipeline for it

These are not theoretical skills. Every step in this pipeline is a task that appears in real data engineering, operations automation, and AI-assisted workflows.

---

## Before You Start — Setup

All five tools need to be installed. Run this once before beginning File 5-1:

```bash
pip install pandas openpyxl pdfplumber pypdf requests
```

`os`, `pathlib`, and `shutil` are built into Python — no installation needed.

**Verify the installation:**

```python
import pandas as pd
import openpyxl
import pdfplumber
from pypdf import PdfReader
import requests

print("pandas:", pd.__version__)
print("openpyxl:", openpyxl.__version__)
print("requests:", requests.__version__)
print("All tools ready")
```

**Output**

```text
pandas: 2.2.2
openpyxl: 3.1.2
requests: 2.31.0
All tools ready
```

If any import fails, re-run the `pip install` command for that specific library.

---

## Reference Links

- 📎 [pandas Official Documentation](https://pandas.pydata.org/docs/)
- 📎 [openpyxl Official Documentation](https://openpyxl.readthedocs.io/en/stable/)
- 📎 [pdfplumber GitHub](https://github.com/jsvine/pdfplumber)
- 📎 [pypdf Documentation](https://pypdf.readthedocs.io/en/stable/)
- 📎 [requests Documentation](https://requests.readthedocs.io/en/latest/)
- 📎 [Real Python — Automating Everyday Tasks with Python](https://realpython.com/python-automation/)
