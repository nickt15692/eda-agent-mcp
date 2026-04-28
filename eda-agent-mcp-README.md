# EDA Agent MCP

Automated exploratory data analysis powered by **Claude AI** and the **Model Context Protocol (MCP)**. Upload any CSV file and get a professional, AI-written EDA report with charts in seconds.

---

## What it does

The agent runs a full EDA pipeline end-to-end:

1. **Computes statistics** — shape, missing values, numeric distributions, categorical summaries, correlations, outliers, duplicates
2. **Generates charts** — missing value heatmap, distribution histograms, correlation heatmap, categorical bar charts (saved as PNG)
3. **Writes a report** — sends all stats to Claude (`claude-sonnet-4-6`) which interprets the numbers, flags data quality issues, and produces a structured, business-ready markdown report
4. **Saves the output** — timestamped `.md` file alongside the embedded chart references

---

## Architecture

```
mcp_client.py        ← autonomous Claude agent (orchestrates tool calls)
    │
    └── mcp_server.py    ← MCP server exposing 3 tools
            │
            ├── agent.py         ← calls Claude API to generate the report
            ├── eda_engine.py    ← computes all statistics from the DataFrame
            └── charts.py        ← generates and saves matplotlib/seaborn charts

app.py               ← Streamlit web UI (alternative to CLI)
```

### MCP Tools exposed by the server

| Tool | Description |
|------|-------------|
| `run_full_eda_pipeline` | Full pipeline: load CSV → stats → charts → AI report → save |
| `get_quick_stats` | Raw statistics only, no report or charts |
| `generate_charts_only` | Charts only, saved to a configurable output directory |

### Report sections

Every generated report includes:

1. Dataset Overview
2. Data Quality Assessment
3. Key Statistical Insights
4. Correlation Analysis
5. Categorical Variable Analysis
6. ML Readiness Assessment
7. Recommended Next Steps

---

## Setup

**Prerequisites:** Python 3.10+, an [Anthropic API key](https://console.anthropic.com/)

```bash
# 1. Clone the repo
git clone https://github.com/nickt15692/eda-agent-mcp.git
cd eda-agent-mcp

# 2. Create a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
pip install streamlit             # only needed for the web UI

# 4. Set your API key
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
```

---

## Usage

### Option A — Streamlit web UI (recommended)

```bash
# macOS / Linux
./run.sh

# Windows
run.bat
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

**Pages:**
- **Upload & Analyze** — drag-and-drop a CSV, see a live preview, click "Run Full EDA Analysis"
- **View Report** — read the AI-generated markdown report in-browser or download it
- **View Charts** — browse all generated charts in a 2-column grid

### Option B — CLI agent

Runs the autonomous agent directly. Claude will call `get_quick_stats` then `run_full_eda_pipeline` automatically.

```bash
python mcp_client.py --file path/to/your_data.csv
```

### Option C — Pipeline function (import)

```python
from agent import run_full_pipeline

output_path = run_full_pipeline("your_data.csv")
# → saves eda_report_YYYYMMDD_HHMMSS.md
```

---

## Output

After a run you'll find:

```
eda_report_20260330_160843.md    ← AI-written report (timestamped)
charts/
  missing_values.png             ← null heatmap (if any missing data)
  distributions.png              ← histograms for all numeric columns
  correlation_heatmap.png        ← lower-triangle correlation matrix
  categorical_<col>.png          ← bar charts for up to 3 categorical columns
```

---

## Dependencies

```
anthropic        # Claude API
mcp              # Model Context Protocol
pandas
numpy
matplotlib
seaborn
python-dotenv
streamlit        # web UI only
```

---

## Project structure

```
eda-agent-mcp/
├── agent.py           # Claude API call + report generation
├── app.py             # Streamlit web UI
├── charts.py          # Chart generation (matplotlib/seaborn)
├── eda_engine.py      # Statistical analysis engine
├── mcp_client.py      # Autonomous MCP client agent
├── mcp_server.py      # MCP server with 3 tools
├── requirements.txt
├── run.sh             # macOS/Linux launcher
├── run.bat            # Windows launcher
└── .env               # ANTHROPIC_API_KEY (not committed)
```
